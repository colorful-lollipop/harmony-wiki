# 安全风险评审

## 目的
本文档基于代码证据分析 graphic_surface 组件的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围
面向需要：
- 理解 graphic_surface 安全边界的开发者
- 进行安全审计的安全工程师
- 评估风险的架构师
- 集成 graphic_surface 的产品安全团队

## 证据范围
本文档基于以下代码证据：
- `surface/src/buffer_queue_producer.cpp` - PID 验证逻辑
- `sandbox/sandbox_utils.cpp` - 沙箱 PID 获取
- `buffer_handle/src/buffer_handle.cpp` - BufferHandle 序列化
- `utils/hebc_white_list/hebc_white_list.cpp` - HEBC 白名单
- `interfaces/inner_api/surface/ibuffer_producer.h` - IPC 接口定义
- `surface/src/buffer_queue.cpp` - Buffer 队列管理
- `bundle.json` - 组件依赖

## 安全边界与信任边界

### 信任边界图

```
┌─────────────────────────────────────────────────────────────┐
│                非信任区域（用户应用）                    │
│         (应用层 / 第三方应用)                          │
└─────────────────────┬───────────────────────────────────────┘
                      │ IPC 调用
                      ▼
┌─────────────────────────────────────────────────────────────┐
│           半信任区域（graphic_surface）                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Surface (生产者/消费者)                         │   │
│  │  - PID 验证 (GetCallingPid)                    │   │
│  │  - BufferHandle 传输                              │   │
│  │  - 连接状态检查                                   │   │
│  └───────────────────────┬─────────────────────────┘   │
└───────────────────────────┼───────────────────────────────┘
                          │ 共享内存（零拷贝）
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              信任区域（系统组件）                        │
│     (WindowManager, SurfaceFlinger, GPU HAL)            │
└─────────────────────────────────────────────────────────────┘
```

### 安全检查点

| 检查点 | 位置 | 信任假设 |
|---------|--------|-----------|
| PID 验证 | `buffer_queue_producer.cpp:CheckConnectLocked()` | 相信 GetCallingPid() 返回值 |
| 连接状态 | `buffer_queue_producer.cpp:Connect()` | 相信首次连接者的标识 |
| BufferHandle 序列化 | `buffer_handle.cpp:WriteBufferHandle()` | 相信 fd 传递的安全性 |
| HEBC 白名单 | `hebc_white_list.cpp:Check()` | 相信 /proc/self/cmdline 内容 |
| 沙箱隔离 | `sandbox_utils.cpp:GetRealPid()` | 相信跨平台 PID 获取 |

## 攻击面分析

### 1. IPC 攻击面

#### 入口点
- **IBufferProducer::Connect()** - 建立连接
- **IBufferProducer::RequestBuffer()** - 请求 Buffer
- **IBufferProducer::FlushBuffer()** - 提交 Buffer
- **IBufferProducer::CancelBuffer()** - 取消 Buffer
- **IBufferProducer::AttachBuffer()** - 附加 Buffer
- **IBufferProducer::DetachBuffer()** - 分离 Buffer
- **IBufferProducer::SetMetaData()** - 设置元数据
- 共 50+ IPC 方法

#### 风险分类
- **DoS 攻击**：频繁调用 RequestBuffer 耗尽队列
- **数据注入**：恶意 BufferHandle 传递
- **权限绕过**：PID 欺骗
- **信息泄露**：通过元数据泄露内存布局

#### 证据
```cpp
// buffer_queue_producer.cpp:135-148
GSError BufferQueueProducer::CheckConnectLocked()
{
    if (connectedPid_ == 0) {
        return SURFACE_ERROR_CONSUMER_DISCONNECTED;
    }
    if (connectedPid_ != GetCallingPid()) {  // 唯一 PID 检查
        return SURFACE_ERROR_CONSUMER_IS_CONNECTED;
    }
    return GSERROR_OK;
}
```

### 2. 共享内存攻击面

#### 入口点
- **BufferHandle.fd** - 文件描述符传递
- **SurfaceBuffer.virAddr** - 共享内存虚拟地址
- **SurfaceBuffer.extraData** - 扩展数据

#### 风险分类
- **UAF（Use-After-Free）**：Buffer 释放后继续使用
- **越界访问**：Buffer 边界检查不足
- **竞态条件**：多进程并发访问
- **信息泄露**：通过共享内存读取敏感数据

#### 证据
```cpp
// buffer_handle.cpp:WriteBufferHandle() - fd 传递
parcel.WriteFileDescriptor(handle.fd);  // 无 fd 权限验证
```

### 3. 文件系统攻击面

#### 入口点
- **HEBC 白名单配置** - `/etc/graphics_game/config/graphics_game.json`
- **进程命令行读取** - `/proc/self/cmdline`
- **Buffer 持久化** - 临时文件或缓存

#### 风险分类
- **配置注入**：修改 JSON 白名单
- **信息泄露**：通过 /proc 获取进程信息
- **路径遍历**：配置文件路径未验证

#### 证据
```cpp
// hebc_white_list.cpp:GetApplicationName()
std::ifstream procfile("/proc/self/cmdline");  // 直接读取，无验证
std::getline(procfile, name);
```

### 4. 权限攻击面

#### 入口点
- **GetCallingPid()** - 获取调用进程 ID
- **connectedPid_** - 已连接进程 ID
- **access_token 依赖** - 声明但未直接使用

#### 风险分类
- **PID 欺骗**：伪造 PID 绕过连接检查
- **权限提升**：无细粒度权限检查
- **横向移动**：同一 UID 下进程互相访问

#### 证据
```cpp
// buffer_queue_producer.cpp:1715
auto callingPid = GetCallingPid();  // 信任系统调用
```

## 可被利用点（基于证据）

### 风险 1：进程异常导致严重内存泄漏

**严重程度**：🔴 **高**

**证据**：
- `README.md:59-60` - 风险提示：
  > "由于使用了共享内存，而共享内存的管理任务在首次创建Surface的进程中，因此需要对该进程格外关注，如果发生进程异常且没有回收处理会发生严重的内存泄漏"
- `surface/include/buffer_queue_producer.h` - `connectedPid_` 跟踪进程
- `surface/src/buffer_queue.cpp` - Buffer 引用计数管理

**可利用路径**：
1. 恶意应用作为生产者连接到 Surface
2. 分配大量 Buffer（FillQueue 攻击）
3. 应用进程崩溃或被杀死（故意触发）
4. 共享内存未回收，系统内存耗尽

**影响**：
- 系统内存耗尽
- 其他应用受影响
- 可能导致系统重启

**修复建议**：
1. **死亡通知机制**（已实现）：
   ```cpp
   // buffer_queue_producer.cpp:1595-1603
   bool BufferQueueProducer::HandleDeathRecipient(sptr<IRemoteObject> token)
   {
       if (token_ != nullptr) {
           token_->RemoveDeathRecipient(producerSurfaceDeathRecipient_);
       }
       // ... 清理逻辑
   }
   ```
   - **验证**：确保所有 Buffer 在死亡时被回收
   - **增强**：在清理前强制释放所有引用

2. **Watchdog 监控**：
   ```cpp
   // 建议添加
   if (connectedPid_ == 0 && bufferQueueCache_.size() > 0) {
       BLOGE("Memory leak detected: consumer died without cleanup");
       CleanCache(true);  // 强制清理
   }
   ```

3. **共享内存配额**：
   - 限制单个进程可分配的总共享内存大小
   - 超过配额拒绝分配

---

### 风险 2：PID 欺骗绕过连接检查

**严重程度**：🟡 **中**

**证据**：
- `surface/src/buffer_queue_producer.cpp:135-148` - PID 验证逻辑：
  ```cpp
  if (connectedPid_ != GetCallingPid()) {
      return SURFACE_ERROR_CONSUMER_IS_CONNECTED;
  }
  ```
- `sandbox/sandbox_utils.cpp:GetRealPid()` - 跨平台 PID 获取：
  ```cpp
  #ifdef _WIN32
      return GetCurrentProcessId();
  #elif defined(OHOS_LITE) || defined(__APPLE__) || defined(__gnu_linux__)
      return getpid();
  #else
      return getprocpid();  // OpenHarmony 特定
  #endif
  ```

**可利用路径**：
1. 攻击者 A 作为消费者连接到 Surface
2. 攻击者 B 尝试连接（被拒绝，因为 connectedPid_ != GetCallingPid()）
3. 如果 GetCallingPid() 存在漏洞或可伪造（如内核漏洞）
4. 攻击者 B 绕过检查，接管 BufferQueue

**影响**：
- 数据泄露（读取其他应用的 Buffer）
- 数据篡改（修改其他应用的 Buffer）
- DoS（破坏其他应用的渲染流程）

**修复建议**：
1. **使用 Access Token 替代 PID**：
   ```cpp
   // 当前：PID 检查
   if (connectedPid_ != GetCallingPid()) { ... }

   // 建议：Token 检查
   uint64_t token = GetAccessTokenId();
   if (connectedToken_ != token) {
       return GSERROR_NO_PERMISSION;
   }
   ```
   - Access Token 提供更强的身份验证
   - Token 在进程创建时分配，不可伪造

2. **添加 UID/BID 检查**：
   ```cpp
   int32_t callingUid = GetCallingUid();
   int32_t callingBid = GetCallingBundleId();
   if (connectedUid_ != callingUid || connectedBid_ != callingBid) {
       return GSERROR_NO_PERMISSION;
   }
   ```

3. **使用 Binder 身份验证**：
   ```cpp
   // 当前：仅检查 PID
   auto remoteDescriptor = arguments.ReadInterfaceToken();
   // 仅验证 descriptor 匹配

   // 建议：验证调用者身份
   sptr<IRemoteObject> caller = GetCurrentCallingIdentity();
   if (!VerifyCaller(caller, connectedIdentity_)) {
       return GSERROR_NO_PERMISSION;
   }
   ```

---

### 风险 3：HEBC 白名单 JSON 配置注入

**严重程度**：🟡 **中**

**证据**：
- `utils/hebc_white_list/hebc_white_list.cpp:Init()` - 白名单加载：
  ```cpp
  std::ifstream file(configPath);  // /etc/graphics_game/config/graphics_game.json
  cJSON *root = cJSON_Parse(jsonStr.c_str());  // 无 JSON 验证
  ```
- `utils/hebc_white_list/hebc_white_list.cpp:GetApplicationName()` - 进程名读取：
  ```cpp
  std::ifstream procfile("/proc/self/cmdline");  // 直接读取
  std::getline(procfile, name);
  ```

**可利用路径**：
1. 攻击者有权限修改 `/etc/graphics_game/config/graphics_game.json`
2. 注入恶意应用名到白名单
3. 恶意应用绕过 HEBC 限制，获得 Buffer 缓存特权
4. 造成内存碎片化或性能下降

**影响**：
- 系统性能下降（内存碎片）
- 不当应用获得高性能 Buffer 优先级
- 可能的安全绕过

**修复建议**：
1. **配置文件签名验证**：
   ```cpp
   // 建议：验证配置文件签名
   if (!VerifyConfigSignature(configPath)) {
       BLOGE("Invalid config signature: %s", configPath);
       return false;
   }
   ```

2. **JSON 结构验证**：
   ```cpp
   // 建议：验证 JSON 结构
   if (!cJSON_IsArray(root) || cJSON_GetArraySize(root) > MAX_HEBC_LIST_SIZE) {
       BLOGE("Invalid config format");
       return false;
   }
   ```

3. **SELinux 上下文检查**：
   ```cpp
   // 建议：限制可读配置文件的进程
   if (!CheckSELinuxContext(getpid(), "graphics_game_config")) {
       BLOGE("Unauthorized access to HEBC config");
       return false;
   }
   ```

4. **应用签名验证**：
   ```cpp
   // 建议：检查应用签名而非仅应用名
   std::string appSignature = GetAppSignature(callingPid);
   if (!IsSignatureInWhitelist(appSignature)) {
       return false;
   }
   ```

---

### 风险 4：BufferHandle fd 传递无权限验证

**严重程度**：🟡 **中**

**证据**：
- `buffer_handle/src/buffer_handle.cpp:WriteBufferHandle()` - fd 序列化：
  ```cpp
  bool WriteBufferHandle(MessageParcel &parcel, const BufferHandle &handle)
  {
      // ... 写入元数据
      parcel.WriteBool(validFd);
      parcel.WriteFileDescriptor(handle.fd);  // 无 fd 权限验证
      // ...
  }
  ```

**可利用路径**：
1. 攻击者 A 创建一个合法 Buffer（fd1）
2. 攻击者 A 获取 fd1 的句柄
3. 攻击者 A 将 fd1 传递给攻击者 B
4. 攻击者 B 使用 fd1 读取/修改 Buffer
5. 如果 fd 未正确关闭或权限未设置，导致越权访问

**影响**：
- 跨进程数据泄露
- 数据篡改
- 间接权限提升

**修复建议**：
1. **fd 权限验证**：
   ```cpp
   // 建议：验证 fd 权限
   int flags = fcntl(handle.fd, F_GETFL);
   if (flags < 0 || (flags & O_RDWR)) {
       BLOGE("Invalid fd permissions");
       return false;
   }
   ```

2. **fd 使用范围限制**：
   ```cpp
   // 建议：记录 fd 创建者和预期使用者
   struct FdInfo {
       int32_t fd;
       int32_t creatorPid;
       int32_t allowedUserPid;
   };
   if (fdInfo.allowedUserPid != GetCallingPid()) {
       return GSERROR_NO_PERMISSION;
   }
   ```

3. **使用 SELinux fd 类别**：
   ```cpp
   // 建议：SELinux fd 类别隔离
   if (!SetFdClass(handle.fd, "surface_buffer")) {
       BLOGE("Failed to set fd class");
       return false;
   }
   ```

---

### 风险 5：Magic Number 篡改导致内存破坏检测失效

**严重程度**：🟢 **低**（但影响严重性高）

**证据**：
- `surface/src/buffer_queue_producer.cpp:174-187` - Magic Number 验证：
  ```cpp
  bool BufferQueueProducer::CheckIsAlive()
  {
      static const bool isBeta = system::GetParameter(
          "const.logsystem.versiontype", "") == "beta";
      if (magicNum_ != MAGIC_INIT) {  // MAGIC_INIT = 0x16273849
          if (isBeta) {
              raise(42);  // 触发崩溃报告
          }
          return false;
      }
      return true;
  }
  ```

**可利用路径**：
1. 攻击者通过内存破坏漏洞修改 `magicNum_`
2. 如果是 Beta 版本，触发 `raise(42)` 崩溃
3. 攻击者可能利用崩溃进行信息泄露或控制劫持
4. 如果是非 Beta 版本，仅返回 false，可能绕过某些检查

**影响**：
- 内存破坏未被检测到
- 可能的进一步利用（RCE）
- 系统崩溃（DoS）

**修复建议**：
1. **Magic Number 随机化**：
   ```cpp
   // 建议：使用随机 Magic Number
   static uint32_t GenerateMagicNumber() {
       return GetSecureRandom();  // 加密安全随机数
   }

   magicNum_ = GenerateMagicNumber();
   ```

2. **移除 Beta 条件**：
   ```cpp
   // 建议：所有版本都启用严格检查
   if (magicNum_ != expectedMagicNum_) {
       raise(42);  // 生产环境也触发
       return false;
   }
   ```

3. **添加完整性校验**：
   ```cpp
   // 建议：添加 CRC 或 HMAC
   uint8_t integrity = ComputeIntegrityCheck(this);
   if (integrity != storedIntegrity_) {
       BLOGE("Integrity check failed");
       return false;
   }
   ```

---

## 其他安全考虑

### 未发现（但需关注）的风险

#### 1. Buffer 越界访问
**检查范围**：已搜索 Buffer 边界检查代码
**结论**：依赖底层 gralloc 驱动的保护，graphic_surface 层未发现明显越界漏洞

#### 2. 竞态条件
**检查范围**：已搜索多线程同步机制
**结论**：
- 使用 `mutex_` 保护队列操作
- 使用 `condition_variable` 等待机制
- **潜在问题**：Buffer 状态机可能出现竞态（RELEASED → REQUESTED → FLUSHED）
- **建议**：增加状态转换的原子性验证

#### 3. 信息泄露（元数据）
**检查范围**：已搜索 Buffer 元数据处理
**结论**：
- 元数据通过 IPC 传递，未发现敏感信息泄露
- **建议**：审查 SetMetaData 实现中是否有调试信息泄漏

#### 4. 路径遍历
**检查范围**：已搜索文件路径处理
**结论**：HEBC 配置文件路径硬编码，无用户输入路径处理，风险较低

## 安全最佳实践建议

### 对集成者
1. **最小权限原则**：
   - 仅授予应用必要的 `access_token` 权限
   - 不要为所有应用开放 Surface 创建权限

2. **使用受信任的 Surface**：
   - 从 WindowManager 获取 Surface，而非直接创建
   - 避免从未知来源的 IBufferProducer 创建 ProducerSurface

3. **监控内存使用**：
   - 跟踪应用的 Buffer 分配量
   - 对异常行为告警

4. **验证 IPC 调用者**：
   - 检查调用者的 access_token
   - 验证 Bundle ID 和签名

### 对开发者
1. **正确处理错误码**：
   - 检查所有 API 返回的 `GSError`
   - 不要忽略 `GSERROR_NO_CONSUMER`、`GSERROR_CONSUMER_IS_CONNECTED`

2. **正确管理 Buffer 生命周期**：
   - Request 后必须 Flush 或 Cancel
   - Acquire 后必须 Release
   - 使用 Reference/Unreference 管理引用计数

3. **等待 SyncFence**：
   - 在读取 Buffer 前等待 acquireFence
   - 在释放 Buffer 前设置 releaseFence

4. **避免阻塞**：
   - 使用 noblock 模式避免死锁
   - 设置合理的超时时间

### 对系统集成者
1. **启用死亡通知**：
   - 所有 Surface 连接都注册死亡监听器
   - 进程死亡时自动清理资源

2. **限制 Buffer 队列大小**：
   - 使用 SetQueueSize 限制队列容量
   - 防止 DoS 攻击

3. **启用 HEBC 白名单验证**：
   - 仅对可信游戏应用启用 HEBC
   - 定期审查白名单内容

4. **监控和告警**：
   - 监控异常的 IPC 调用频率
   - 监控内存分配失败率
   - 对异常行为告警

## 安全测试建议

### Fuzz 测试
graphic_surface 已有 Fuzz 测试框架：
- `surface/test/fuzztest/` - 9 个 Fuzz 目标
- `sync_fence/test/fuzztest/` - SyncFence Fuzz
- `buffer_handle/test/` - BufferHandle Fuzz

**建议**：
1. 增加对 IPC 参数的 Fuzz
2. 增加 JSON 配置 Fuzz
3. 使用 AFL、Honggfuzz 等工具

### 漏洞扫描
建议使用静态分析工具：
- **Cppcheck** - 检测内存安全漏洞
- **Coverity** - 检测复杂漏洞
- **Clang Static Analyzer** - 检测未定义行为

### 动态测试
建议进行：
- **竞态检测** - ThreadSanitizer (TSan)
- **内存安全检测** - AddressSanitizer (ASan)
- **未定义行为检测** - UndefinedBehaviorSanitizer (UBSan)

## 相关跳转
- [目录结构与模块职责](01_Directory_Structure.md) - 安全相关文件位置
- [对外 API](03_External_API.md) - API 使用安全建议
- [架构说明](02_Architecture.md) - 组件交互安全边界
- [常见问题](08_Troubleshooting.md) - 安全相关调试方法

## 免责声明
本文档基于代码静态分析，不能保证发现所有安全风险。实际部署前建议：
1. 进行全面的渗透测试
2. 建立安全监控和告警机制
3. 定期更新安全补丁
4. 关注 OpenHarmony 安全公告
