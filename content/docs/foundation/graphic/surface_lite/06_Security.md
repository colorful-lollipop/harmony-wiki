# 安全风险评审

> surface_lite 安全分析与风险评估

---

## 1. 评估范围

### 1.1 评估对象

- **组件**: surface_lite (foundation/graphic/surface_lite)
- **版本**: 3.1 (OpenHarmony master 分支)
- **代码路径**: interfaces/, frameworks/
- **排除**: test/ 目录

### 1.2 评估方法

1. 源代码静态分析
2. 攻击面识别
3. 信任边界分析
4. 已知漏洞模式匹配

---

## 2. 攻击面分析

### 2.1 攻击面清单

| 攻击面 | 类型 | 风险等级 | 说明 |
|--------|------|----------|------|
| **IPC 接口** | 网络/进程间 | 🔴 高 | 20 个 IPC 请求码，跨进程调用 |
| **共享内存** | 内存 | 🔴 高 | Buffer 数据共享，物理/虚拟内存 |
| **Buffer 元数据** | 输入 | 🟡 中 | width/height/format/size 等参数 |
| **额外数据** | 输入 | 🟡 中 | SetInt32/SetInt64 键值对 |
| **用户数据** | 输入 | 🟢 低 | SetUserData 字符串键值对 |
| **Gralloc 接口** | 外部依赖 | 🟡 中 | 底层内存分配器 |

### 2.2 信任边界图

```
┌─────────────────────────────────────────────────────────────────┐
│                        不信任区域                                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  恶意应用    │    │  攻击者进程  │    │  外部输入   │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
└─────────┼──────────────────┼──────────────────┼─────────────────┘
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界 (IPC Handler)                       │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  SurfaceImpl::DoIpcMsg() / BufferQueueProducer::OnIpcMsg()│   │
│  │  - 请求码校验 (code < MAX_REQUEST_CODE)                   │   │
│  │  - 参数反序列化                                          │   │
│  │  ❌ 缺少调用者身份验证                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    核心信任区域                          │   │
│  │                                                         │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │ BufferQueue │  │BufferManager│  │  Gralloc    │     │   │
│  │  │  (状态管理)  │  │ (内存管理)   │  │ (硬件抽象)   │     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    内核信任区域                          │   │
│  │              (共享内存、物理内存、DMA)                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**关键发现**: IPC 入口缺少调用者身份验证 (TODO)

---

## 3. 可被利用点 (风险清单)

### 3.1 风险 #1: IPC 请求码越界访问

**位置**: `frameworks/buffer_queue_producer.cpp:373-385`

**代码**:
```cpp
int32_t BufferQueueProducer::OnIpcMsg(uint32_t code, IpcIo *data, 
                                       IpcIo *reply, MessageOption option) {
    if (data == NULL) {
        return SURFACE_ERROR_INVALID_PARAM;
    }
    
    if (code >= MAX_REQUEST_CODE) {  // 边界检查存在
        return SURFACE_ERROR_INVALID_REQUEST;
    }
    
    return g_ipcMsgHandleList[code](this, data, reply);  // 数组访问
}
```

**评估**: ✅ **已修复** - 有明确的边界检查

**风险等级**: 🟢 低

---

### 3.2 风险 #2: IPC 反序列化缺少长度校验

**位置**: `frameworks/surface_buffer_impl.cpp:125-160`

**代码**:
```cpp
void SurfaceBufferImpl::ReadFromIpcIo(IpcIo& io) {
    ReadInt32(&io, &bufferData_.handle.key);
    ReadUint64(&io, &bufferData_.handle.phyAddr);
    // ... 读取其他字段
    
    uint32_t extDataSize;
    ReadUint32(&io, &extDataSize);
    
    // ⚠️ 仅检查上限，不检查合法性
    if (extDataSize > 0 && extDataSize < MAX_USER_DATA_COUNT) {
        for (uint32_t i = 0; i < extDataSize; i++) {
            // 循环读取
        }
    }
}
```

**风险**:
- extDataSize 可由攻击者控制
- 循环次数可能导致 CPU 资源耗尽
- 每个条目读取失败不会中断循环

**触发路径**:
```
攻击者进程 → SendRequest() → OnRequestBuffer() → ReadFromIpcIo()
                                         ↓
                                    构造恶意 IPC 消息
                                    extDataSize = 999 (接近上限)
```

**影响**: DoS (拒绝服务)

**风险等级**: 🟡 中

**修复建议**:
```cpp
// 添加更严格的校验
if (extDataSize > MAX_USER_DATA_COUNT / 2) {  // 设置合理上限
    GRAPHIC_LOGW("Suspicious extDataSize: %u", extDataSize);
    return;
}
```

---

### 3.3 风险 #3: Buffer 大小整数溢出

**位置**: `frameworks/buffer_manager.cpp:172-193`

**代码**:
```cpp
SurfaceBufferImpl* BufferManager::AllocBuffer(uint32_t width, uint32_t height, 
                                               uint32_t format, uint32_t usage) {
    AllocInfo info = {0};
    info.width = width;
    info.height = height;
    // ...
    SurfaceBufferImpl* buffer = AllocBuffer(info);  // 底层计算 size
    // ...
}
```

**风险**:
- width * height * bpp 可能在底层计算中溢出
- 溢出后分配的 Buffer 小于预期
- 后续写入导致堆溢出

**触发条件**:
```cpp
// 构造极端参数
surface->SetWidthAndHeight(0x10000, 0x10000);  // 大值但不超过 MAX
// 底层: 65536 * 65536 * 4 = 溢出
```

**影响**: 堆溢出 → 代码执行

**风险等级**: 🔴 高

**修复建议**:
```cpp
// 在 SetWidthAndHeight 中添加溢出检查
void SurfaceImpl::SetWidthAndHeight(uint32_t width, uint32_t height) {
    // 现有检查...
    
    // 添加溢出检查
    uint64_t totalSize = static_cast<uint64_t>(width) * height * 4;  // 最大bpp
    if (totalSize > SURFACE_MAX_SIZE) {
        GRAPHIC_LOGE("Size overflow: %lux%lu", width, height);
        return;
    }
    
    producer_->SetWidthAndHeight(width, height);
}
```

---

### 3.4 风险 #4: 共享内存未初始化访问

**位置**: `frameworks/buffer_client_producer.cpp:56-77`

**代码**:
```cpp
SurfaceBufferImpl* BufferClientProducer::RequestBuffer(uint8_t wait) {
    // ... 发送 IPC 请求
    
    SurfaceBufferImpl* buffer = new SurfaceBufferImpl();  // 分配对象
    buffer->ReadFromIpcIo(reply);  // 填充数据
    
    BufferManager* manager = BufferManager::GetInstance();
    if (manager == nullptr) {
        delete buffer;  // 清理
        return nullptr;
    }
    
    if (!manager->MapBuffer(*buffer)) {  // 映射内存
        Cancel(buffer);
        return nullptr;
    }
    
    return buffer;
}
```

**风险**:
- 如果 ReadFromIpcIo 失败填充不完整数据
- MapBuffer 后返回的 virAddr 可能指向未初始化内存
- 生产者可能读取敏感数据残留

**影响**: 信息泄露

**风险等级**: 🟡 中

---

### 3.5 风险 #5: 竞态条件导致 Use-After-Free

**位置**: `frameworks/buffer_queue.cpp:47-62`

**代码**:
```cpp
BufferQueue::~BufferQueue() {
    pthread_mutex_lock(&lock_);  // 加锁
    freeList_.clear();
    dirtyList_.clear();
    
    for (iterBuffer = allBuffers_.begin(); ... ) {
        SurfaceBufferImpl* tmpBuffer = *iterBuffer;
        BufferManager* bufferManager = BufferManager::GetInstance();
        if (bufferManager == nullptr) {
            continue;  // ⚠️ 未释放当前 Buffer
        }
        bufferManager->FreeBuffer(&tmpBuffer);
    }
    allBuffers_.clear();
    pthread_mutex_unlock(&lock_);  // 解锁
    pthread_cond_destroy(&freeCond_);
    pthread_mutex_destroy(&lock_);
}
```

**风险**:
- 如果 BufferManager 为 null，Buffer 未释放但已clear
- 其他线程可能仍持有指针
- 后续访问导致 UAF

**影响**: Use-After-Free → 代码执行

**风险等级**: 🟡 中

**修复建议**:
```cpp
// 确保总是释放，或记录错误
if (bufferManager == nullptr) {
    GRAPHIC_LOGE("BufferManager is null during destruction");
    // 继续尝试释放或记录泄漏
}
```

---

### 3.6 风险 #6: UserData 无长度限制

**位置**: `frameworks/buffer_queue.cpp:379-394`

**代码**:
```cpp
void BufferQueue::SetUserData(const std::string& key, const std::string& value) {
    if (usrDataMap_.size() > USER_DATA_COUNT) {  // 仅限制条目数
        return;
    }
    usrDataMap_[key] = value;  // 无单条长度限制
}
```

**风险**:
- key/value 长度无限制
- 可能导致内存过度消耗
- 跨进程传输时大字符串消耗 IPC 带宽

**影响**: DoS / 资源耗尽

**风险等级**: 🟢 低

---

### 3.7 风险 #7: 缺少调用者权限校验

**位置**: 所有 IPC 处理函数

**现状**:
```cpp
// 所有 OnXXX 处理函数都没有调用者身份检查
static int32_t OnRequestBuffer(BufferQueueProducer* product, IpcIo *io, IpcIo *reply) {
    // 直接处理，不检查调用者 uid/token
}
```

**风险**:
- 任意进程可连接并操作 Surface
- 恶意应用可窃取或篡改 Buffer 数据
- 无法实施访问控制策略

**影响**: 未授权访问

**风险等级**: 🔴 高

**修复建议**:
```cpp
// TODO: 添加调用者身份验证
static int32_t OnRequestBuffer(...) {
    // 获取调用者身份信息
    pid_t callerPid = GetCallerPid();
    uid_t callerUid = GetCallerUid();
    
    // 校验权限
    if (!CheckPermission(callerUid, PERM_ACCESS_SURFACE)) {
        return SURFACE_ERROR_PERMISSION_DENIED;
    }
    
    // 处理请求
}
```

---

## 4. 已知漏洞参考

### 4.1 OpenHarmony 图形相关 CVE (2024-2025)

| CVE ID | 严重程度 | 描述 | 相关性 |
|--------|----------|------|--------|
| CVE-2024-47398 | 🔴 8.8 | 越界写入 | 中 (类似模式) |
| CVE-2024-54030 | 🟡 4.4 | Use-After-Free | 高 (风险 #5) |
| CVE-2024-45070 | 🟡 5.5 | 越界读取 | 中 (风险 #4) |
| CVE-2025-20024 | 🔴 7.8 | 整数溢出 | 高 (风险 #3) |
| CVE-2025-23234 | 🟡 5.5 | 缓冲区溢出 | 中 |
| CVE-2025-27132 | 🔴 7.8 | 越界写入 | 中 |
| CVE-2025-24298 | 🔴 Critical | Use-After-Free | 高 (风险 #5) |

---

## 5. 修复建议汇总

### 5.1 高优先级

1. **添加 IPC 调用者身份验证**
   - 位置: 所有 IPC 处理函数
   - 方案: 使用 IPC 框架提供的身份查询接口

2. **修复整数溢出风险**
   - 位置: `SurfaceImpl::SetWidthAndHeight`
   - 方案: 添加 64 位溢出检查

### 5.2 中优先级

3. **强化 IPC 反序列化校验**
   - 位置: `ReadFromIpcIo`
   - 方案: 限制循环次数，检查每条目合法性

4. **修复 UAF 风险**
   - 位置: `BufferQueue::~BufferQueue`
   - 方案: 确保异常情况下也释放资源

5. **初始化敏感内存**
   - 位置: Buffer 分配路径
   - 方案: 清零新分配的 Buffer

### 5.3 低优先级

6. **限制 UserData 长度**
   - 位置: `SetUserData`
   - 方案: 添加 key/value 长度限制

---

## 6. 检查局限性

### 6.1 未覆盖范围

1. **底层 Gralloc 实现**: 依赖厂商实现，未审计
2. **IPC 框架本身**: 假设 IPC 底层安全
3. **测试代码**: test/ 目录未分析
4. **硬件交互**: 物理内存管理由 HAL 层处理

### 6.2 假设条件

- 攻击者已具备进程间通信能力
- 系统其他组件存在被攻破可能
- 物理内存由可信的 Gralloc 实现管理

---

## 7. 安全开发建议

### 7.1 输入校验清单

- [ ] 所有 IPC 参数范围检查
- [ ] 指针非空检查
- [ ] 字符串长度限制
- [ ] 数组索引边界检查
- [ ] 整数运算溢出检查

### 7.2 内存安全清单

- [ ] 新分配内存清零
- [ ] 释放后置空指针
- [ ] 避免重复释放
- [ ] 锁保护共享状态
- [ ] 析构时确保资源释放

### 7.3 权限控制清单

- [ ] IPC 调用者身份验证
- [ ] 敏感操作权限检查
- [ ] 资源配额限制

---

*文档版本: v1.0 | 更新日期: 2026-02-06*
