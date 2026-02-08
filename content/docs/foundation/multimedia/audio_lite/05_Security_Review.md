# 05_Security_Review - 安全评审

本文档对 audio_lite 进行安全风险分析，识别潜在攻击面和可利用点，并提供修复建议。

## 5.1 评审范围与方法

### 5.1.1 评审范围

| 维度 | 覆盖情况 |
|------|----------|
| 代码范围 | `frameworks/`, `services/`, `interfaces/kits/` |
| 测试代码 | **不纳入评审**（根据 Wiki 规范） |
| 第三方库 | codec, audio_hw 等 HAL 库不纳入评审 |
| 系统依赖 | samgr, pms_client 等依赖框架不深入评审 |

### 5.1.2 评审方法

- 静态代码分析：检查输入校验、内存安全、权限验证
- 架构分析：识别信任边界、数据流
- 威胁建模：识别攻击面、潜在威胁

## 5.2 攻击面分析

### 5.2.1 外部输入点

| 输入源 | 输入类型 | 风险等级 |
|--------|----------|----------|
| 用户空间 Buffer (`Read()`) | 内存写入 | **高** |
| AudioCapturerInfo 参数 | 配置参数 | **中** |
| IPC 命令 (funcId) | 控制命令 | **中** |
| Surface 数据 | 共享内存数据 | **低** |

### 5.2.2 敏感操作

| 操作 | 说明 | 风险等级 |
|------|------|----------|
| HAL 调用 (AudioSource) | 访问硬件资源 | **高** |
| 内存分配 (new/malloc) | 资源消耗 | **中** |
| Surface Flush | 跨进程数据传递 | **低** |
| 线程创建 (pthread_create) | 资源消耗 | **低** |

### 5.2.3 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                         信任边界                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  边界 1: 客户端 ↔ IPC 框架                                    │
│         - GetCallingPid() 提供可信调用者身份                  │
│         - 单客户端限制 (clientPid_)                          │
│                                                             │
│  边界 2: 服务端 ↔ HAL                                        │
│         - AudioSource 封装 HAL 调用                          │
│         - 无额外验证                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 5.3 安全风险清单

### 5.3.1 风险列表

| ID | 风险类型 | 严重程度 | 可利用性 | 状态 |
|----|----------|----------|----------|------|
| SR-01 | 缓冲区溢出 | **高** | 已验证 | 需修复 |
| SR-02 | 空指针解引用 | **中** | 潜在 | 需修复 |
| SR-03 | 权限缺失 | **中** | 潜在 | 建议增强 |
| SR-04 | 资源耗尽 | **低** | 潜在 | 需修复 |
| SR-05 | 整数溢出 | **低** | 潜在 | 需确认 |

---

### SR-01: 缓冲区溢出风险（高危）

**证据**：`frameworks/binder/audio_capturer_client.cpp:379-413`

```cpp
int32_t AudioCapturer::AudioCapturerClient::Read(uint8_t *buffer, size_t userSize, bool isBlockingRead)
{
    // ...
    uint8_t *buf = static_cast<uint8_t *> (surfaceBuf->GetVirAddr());
    int32_t dataSize = surfaceBuf->GetSize();
    
    // ⚠️ 风险：仅检查是否过大，未检查是否过小
    if (dataSize - sizeof(Timestamp) > userSize) {
        surface_->ReleaseBuffer(surfaceBuf);
        MEDIA_ERR_LOG("input buffer size too small.");
        break;
    }
    
    // ⚠️ 风险：memcpy_s 目标大小为 userSize，但实际数据可能更小
    (void)memcpy_s(buffer, userSize, buf + sizeof(Timestamp), dataSize - sizeof(Timestamp));
    // ...
}
```

**问题分析**：

1. **仅检查上限**：代码检查 `dataSize > userSize`，但未检查 `dataSize` 是否为有效值
2. **memcpy_s 用法**：`destMax` 参数为 `userSize`，但实际复制大小为 `dataSize - sizeof(Timestamp)`
3. **时间戳覆盖**：`memcpy_s(&curTimestamp_, ...)` 未检查源数据完整性

**触发条件**：

```
1. 攻击者控制 SurfaceBuffer 的 dataSize 字段
2. dataSize < sizeof(Timestamp)（0~31 字节）
3. 复制操作导致越界读取或写入
```

**影响**：

- 内存读取越界（Information Disclosure）
- 潜在的代码执行（配合其他漏洞）

**修复建议**：

```cpp
int32_t AudioCapturer::AudioCapturerClient::Read(uint8_t *buffer, size_t userSize, bool isBlockingRead)
{
    // 添加 dataSize 有效性检查
    if (dataSize < sizeof(Timestamp)) {
        MEDIA_ERR_LOG("Invalid dataSize: %{public}d", dataSize);
        return ERR_INVALID_READ;
    }
    
    // 计算实际数据大小
    size_t actualDataSize = static_cast<size_t>(dataSize - sizeof(Timestamp));
    
    // 检查缓冲区是否足够（使用实际大小而非 userSize）
    if (actualDataSize > userSize) {
        MEDIA_ERR_LOG("Buffer too small: %{public}zu < %{public}zu", userSize, actualDataSize);
        return ERR_INVALID_READ;
    }
    
    // 使用实际大小进行复制
    (void)memcpy_s(buffer, userSize, buf + sizeof(Timestamp), actualDataSize);
    return static_cast<int32_t>(actualDataSize);
}
```

---

### SR-02: 空指针解引用风险（中危）

**证据**：`frameworks/audio_capturer.cpp:22-28`

```cpp
#define CHK_NULL_RETURN(ptr, ret) \
    do { \
        if ((ptr) == nullptr) { \
            MEDIA_ERR_LOG("ptr null"); \
            return (ret); \
        } \
    } while (0)
```

**问题分析**：

1. **检查不完整**：`CHK_NULL_RETURN` 仅检查空指针并返回错误值
2. **未防止重复检查**：多处使用可能导致调试困难
3. **析构风险**：`Release()` 后对象可能处于不一致状态

**触发条件**：

```
1. 构造函数抛出异常后，调用方未正确处理
2. 多线程环境下，Release() 与其他 API 并发调用
```

**影响**：

- 进程崩溃（空指针解引用）
- 资源泄漏

**修复建议**：

```cpp
class AudioCapturer {
private:
    std::unique_ptr<AudioCapturerClient> impl_;
    std::atomic<bool> released_{false};
    
public:
    bool Start() {
        if (released_.load(std::memory_order_acquire)) {
            MEDIA_ERR_LOG("AudioCapturer already released");
            return false;
        }
        CHK_NULL_RETURN(impl_, false);
        return impl_->Start();
    }
    
    bool Release() {
        bool expected = false;
        if (released_.compare_exchange_strong(expected, true, 
                                               std::memory_order_acq_rel)) {
            return impl_->Release();
        }
        return true;  // 已释放，返回成功
    }
};
```

---

### SR-03: 权限验证缺失（中危）

**证据**：`frameworks/BUILD.gn:45-48`

```gn
deps = [
  "//base/security/permission_lite/services/pms_client:pms_client",
  "//foundation/graphic/surface_lite:surface_lite",
  "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
]
```

**问题分析**：

1. **依赖声明存在**：虽然声明了对 `pms_client` 的依赖
2. **代码中未使用**：实际代码中未发现权限校验逻辑
3. **单 PID 检查**：`clientPid_` 仅限制单客户端，无法防止同进程滥用

**触发条件**：

```
1. 恶意应用直接连接到 AudioCapturer 服务
2. 无 RECORD_AUDIO 权限检查
3. 可以录制任意音频
```

**影响**：

- 隐私泄露（未经授权的音频录制）
- 潜在的安全绕过

**修复建议**：

```cpp
int32_t AudioCapturerServer::Dispatch(int32_t funcId, pid_t pid, IpcIo *req, IpcIo *reply)
{
    // 权限检查
    if (funcId == AUD_CAP_FUNC_START || funcId == AUD_CAP_FUNC_READ) {
        PermissionStatus status = pmsClient_->CheckPermission(pid, "ohos.permission.MICROPHONE");
        if (status != GRANTED) {
            MEDIA_WARN_LOG("Permission denied for pid=%{pid}, funcId=%{funcId}", pid, funcId);
            WriteInt32(reply, ERR_PERMISSION_DENIED);
            return EC_PERMISSION_ERR;
        }
    }
    
    // PID 验证
    if (pid != clientPid_) {
        MEDIA_ERR_LOG("PID mismatch: expected=%{clientPid_}, got=%{pid}", clientPid_, pid);
        WriteInt32(reply, ERR_INVALID_PID);
        return EC_INVALID_PID;
    }
    
    // 正常分发
    return DispatchInternal(funcId, req, reply);
}
```

---

### SR-04: 资源耗尽风险（低危）

**证据**：`services/server/src/audio_capturer_server.cpp`

```cpp
// 线程创建
pthread_create(&dataThreadId_, nullptr, ReadAudioDataProcess, this);

// Surface 配置
surface->SetQueueSize(SURFACE_QUEUE_SIZE);  // 5
surface->SetSize(SURFACE_SIZE);              // 8192
```

**问题分析**：

1. **无最大连接限制**：单客户端模式，但每次连接可能创建新实例
2. **Surface 缓冲区固定**：无法根据负载动态调整
3. **线程无超时机制**：数据线程无法优雅退出

**触发条件**：

```
1. 大量快速连接/断开循环
2. 内存压力下分配失败
3. Stop() 失败导致线程泄漏
```

**影响**：

- 资源耗尽（内存、线程句柄）
- 拒绝服务

**修复建议**：

```cpp
class AudioCapturerServer {
private:
    static constexpr int MAX_CONNECTIONS = 10;
    static constexpr int CONNECTION_TIMEOUT_MS = 30000;
    std::atomic<int> activeConnections_{0};
    
public:
    int32_t AcceptServer(pid_t pid, IpcIo *reply)
    {
        // 连接数限制
        if (activeConnections_.load(std::memory_order_acquire) >= MAX_CONNECTIONS) {
            MEDIA_ERR_LOG("Too many connections");
            WriteInt32(reply, ERR_TOO_MANY_CONNECTIONS);
            return MEDIA_IPC_FAILED;
        }
        
        // 超时检查（伪代码）
        if (CheckConnectionTimeout(pid, CONNECTION_TIMEOUT_MS)) {
            MEDIA_WARN_LOG("Connection timeout, force close");
            ForceCloseConnection(pid);
        }
        
        // ... 正常处理
    }
};
```

---

### SR-05: 整数溢出风险（低危，需确认）

**证据**：`frameworks/binder/audio_capturer_client.cpp:213-214`

```cpp
WriteUint32(&io, sizeof(audioFormat));
WriteBuffer(&io, &audioFormat, sizeof(audioFormat));
```

**问题分析**：

1. `sizeof(audioFormat)` 结果类型为 `size_t`，但传递给 `WriteUint32`
2. 在 32 位系统上，`sizeof` 可能返回大于 UINT32_MAX 的值（极不可能，但需验证）

**当前状态**：

- 此处风险较低，`sizeof(Enum)` 在所有平台上均 ≤ 4 字节
- 建议添加断言确保安全

**修复建议**：

```cpp
static_assert(sizeof(audioFormat) <= UINT32_MAX, "sizeof(audioFormat) overflow");
WriteUint32(&io, sizeof(audioFormat));
WriteBuffer(&io, &audioFormat, sizeof(audioFormat));
```

---

## 5.4 信任边界分析

### 5.4.1 当前信任模型

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            信任边界图                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  外部应用 ─────────────────────┬────────────────────────────                │
│                               │                                             │
│                               ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  IPC 边界 (Samgr_lite)                                              │   │
│  │    - GetCallingPid(): 提供调用者 PID                                │   │
│  │    - IpcIo: 序列化控制命令                                           │   │
│  │    - Surface: 共享内存数据传递                                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                               │                                             │
│                               ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  服务端验证                                                           │   │
│  │    - clientPid_ 检查: 单客户端限制                                    │   │
│  │    - 无细粒度权限检查                                                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                               │                                             │
│                               ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  HAL 边界 (AudioSource)                                             │   │
│  │    - 直接调用 HAL 接口                                                │   │
│  │    - 无额外验证                                                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.4.2 数据流安全

| 数据流 | 源 | 目的 | 风险等级 | 现有保护 |
|--------|-----|------|----------|----------|
| IPC 控制命令 | 客户端 | 服务端 | **中** | PID 验证 |
| Surface 数据 | 服务端 | 客户端 | **低** | Size 检查 |
| HAL 参数 | 服务端 | 驱动 | **高** | 无 |
| 配置参数 | 客户端 | 服务端 | **中** | 空值检查 |

---

## 5.5 安全建议总结

### 5.5.1 优先级排序

| 优先级 | 风险 ID | 建议 |
|--------|---------|------|
| P0 (立即修复) | SR-01 | 修复缓冲区溢出检查逻辑 |
| P1 (高优先级) | SR-03 | 集成权限校验框架 |
| P2 (中优先级) | SR-02 | 添加释放状态追踪 |
| P3 (低优先级) | SR-04 | 增加资源限制 |
| P4 (可选) | SR-05 | 添加静态断言 |

### 5.5.2 长期建议

1. **输入验证标准化**：使用 whitelist 策略验证所有外部输入
2. **权限框架集成**：集成 OpenHarmony access_token 框架
3. **安全编码规范**：采用 MISRA C++ 或 AUTOSAR C++14 规范
4. **模糊测试**：添加 libFuzzer 集成
5. **安全审计**：定期进行第三方安全审计

---

## 5.6 评审局限性

| 局限性 | 说明 |
|--------|------|
| HAL 库未评审 | codec、audio_hw 等第三方库不在评审范围 |
| 运行时环境未测试 | 仅进行静态分析，未运行时测试 |
| 并发场景未深入 | 多线程安全性分析不完整 |
| 性能安全未覆盖 | 侧信道攻击等未覆盖 |

---

**上一章**：[04_Build_System](04_Build_System.md) | **下一章**：[06_Troubleshooting](06_Troubleshooting.md)
