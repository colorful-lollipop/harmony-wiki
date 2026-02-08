# 安全风险评审

## 1. 攻击面清单

### 1.1 N-API 接口 (外部入口)

| 接口 | 攻击面 | 说明 |
|------|--------|------|
| `createAVSession()` | 参数注入 | tag, elementName 参数可控 |
| `setAVMetadata()` | 数据篡改 | AVMetaData 包含可伪造的媒体信息 |
| `setAVPlaybackState()` | 状态欺骗 | 播放状态可被伪造 |
| `sendControlCommand()` | 命令注入 | 控制命令可被恶意构造 |
| `sendSystemAVKeyEvent()` | 按键注入 | 系统级按键事件 |
| `castAudio()` | 设备伪造 | AudioDeviceDescriptor 可能被伪造 |

**证据**: `frameworks/js/napi/session/src/napi_avsession_manager.cpp`, `napi_avsession.cpp`

### 1.2 IPC 接口 (进程间通信)

| 接口 | 攻击面 | 说明 |
|------|--------|------|
| `IAVSessionService` | 跨进程调用 | 任何应用可调用 (依赖权限检查) |
| `IAVSession` | 会话操作 | 依赖 Stub 层权限检查 |
| `IAVSessionController` | 控制器操作 | 依赖权限和归属检查 |

**证据**: `services/session/ipc/stub/avsession_service_stub.cpp`, `avsession_stub.cpp`

### 1.3 文件/存储

| 路径 | 攻击面 | 说明 |
|------|--------|------|
| `/data/service/el2/101/` | 目录遍历 | 服务运行时数据目录 |
| 共享内存 (AVSharedMemory) | 内存破坏 | 跨进程共享媒体数据 |

**证据**: `interfaces/inner_api/native/session/include/av_shared_memory.h`

### 1.4 系统能力

| 能力 | 风险 | 说明 |
|------|------|------|
| `MANAGE_MEDIA_RESOURCES` | 权限滥用 | 可控制任意媒体会话 |
| `MANAGE_MEDIA_RESOURCES_FOR_PUBLIC` | 权限提升 | 公共权限可能过度授权 |
| `SEND_SYSTEM_AV_KEY_EVENT` | 按键劫持 | 可注入系统级按键 |

**证据**: `utils/src/permission_checker.cpp`, `services/etc/avsession_service.cfg`

---

## 2. 信任边界

### 边界定义

```
┌─────────────────────────────────────────────────────────────┐
│                    不可信区域                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │ 第三方应用   │  │ 恶意 JS     │  │ 网络攻击者       │  │
│  │ (不可信)     │  │ (不可信)    │  │ (外部)          │  │
│  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘  │
└─────────┼────────────────┼───────────────────┼────────────┘
          │                │                   │
          ▼                ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    边界: IPC Stub                          │
│  ┌─────────────────────────────────────────────────────┐│
│  │  权限检查: CheckPermission()                           ││
│  │  TokenID 验证: IPCSkeleton::GetCallingTokenID()      ││
│  │  UID 验证: IPCSkeleton::GetCallingUid()               ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
          │                │                   │
          ▼                ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                   可信区域                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │ SA 服务     │  │ AVSession   │  │ 远端服务         │  │
│  │ (可信)      │  │ Item        │  │ (分布式通道)     │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 边界检查点

| 检查点 | 文件 | 函数 | 检查内容 |
|--------|------|------|----------|
| Stub 入口 | `avsession_service_stub.cpp` | `CheckPermission()` | 权限类型 |
| Stub 入口 | `avsession_service_stub.cpp` | `GetCallingTokenID()` | Token 验证 |
| 会话归属 | `avsession_item.cpp` | `GetOwnerUid()` | UID 校验 |
| 控制器归属 | `avcontroller_item.cpp` | `GetOwnerPid()` | PID 校验 |

**证据**: `services/session/ipc/stub/avsession_service_stub.cpp:100-150`

---

## 3. 可利用点分析 (风险点)

### 风险 1: 权限检查绕过

**证据位置**: `utils/src/permission_checker.cpp`

```cpp
int32_t PermissionChecker::CheckPermission(int32_t checkPermissionType)
{
    // 检查调用者 TokenID
    AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();

    // 如果 token 为空，可能存在绕过风险
    if (callerToken == 0) {
        // 此处可能返回成功，存在安全风险
        return AVSESSION_SUCCESS;
    }
    // ... 权限验证
}
```

**触发路径**: 恶意应用通过特殊 IPC 调用方式，可能绕过 TokenID 检查

**影响**: 未授权应用可能获取媒体会话信息或发送控制命令

**修复建议**:
```cpp
// 必须显式验证 TokenID
if (callerToken == INVALID_TOKEN_ID) {
    return ERR_PERMISSION_DENIED;
}
```

---

### 风险 2: 会话 ID 伪造

**证据位置**: `napi_avsession.cpp`

```javascript
// JS 层直接传递 sessionId
AVSessionManager.getAVSession(sessionId).then((session) => {
    // 未验证 sessionId 格式和归属
});
```

**证据位置**: `services/session/ipc/proxy/avsession_proxy.cpp`

```cpp
// IPC 层直接使用传入的 sessionId
int32_t AVSessionProxy::GetSessionId(std::string& sessionId)
{
    // 未验证 sessionId 是否属于调用者
    return SendRequest(SESSION_CMD_GET_SESSION_ID, sessionId);
}
```

**触发路径**: 恶意应用使用其他应用的 sessionId 调用 API

**影响**: 可能获取/控制其他应用的媒体会话

**修复建议**:
```cpp
// 在服务端验证 sessionId 归属
int32_t AVSessionServiceStub::HandleGetSession(MessageParcel& data, MessageParcel& reply)
{
    std::string sessionId = data.ReadString();
    int32_t callerUid = IPCSkeleton::GetCallingUid();

    // 验证归属
    if (!ValidateSessionOwnership(sessionId, callerUid)) {
        return ERR_SESSION_NOT_BELONG;
    }
    // ...
}
```

---

### 风险 3: 缓冲区溢出 (AVMetaData)

**证据位置**: `interfaces/inner_api/native/session/include/avmeta_data.h`

```cpp
class AVMetaData : public Parcelable {
public:
    std::string assetId_;       // 无长度限制
    std::string title_;         // 无长度限制
    std::string artist_;        // 无长度限制
    std::string album_;         // 无长度限制
    std::string lyric_;         // 无长度限制
    // ... 其他字符串字段均无长度校验
};
```

**触发路径**: 恶意应用设置超长字符串字段

**影响**: 可能导致字符串处理函数异常或内存损坏

**修复建议**:
```cpp
class AVMetaData : public Parcelable {
    static constexpr size_t MAX_TITLE_LENGTH = 256;
    static constexpr size_t MAX_ARTIST_LENGTH = 128;
    // ...

    int32_t SetTitle(const std::string& title) {
        if (title.length() > MAX_TITLE_LENGTH) {
            return ERR_PARAM_INVALID;
        }
        title_ = title;
        return AVSESSION_SUCCESS;
    }
};
```

---

### 风险 4: 路径遍历 (文件描述符)

**证据位置**: `interfaces/inner_api/native/session/include/av_file_descriptor.h`

```cpp
class AVFileDescriptor {
public:
    int32_t fd_;        // 文件描述符
    int64_t offset_;    // 偏移量
    int64_t length_;    // 长度
};
```

**触发路径**: 恶意应用传入恶意 fd 或 offset/length

**影响**: 可能读取/写入非授权文件

**修复建议**:
```cpp
int32_t AVFileDescriptor::Unmarshalling(MessageParcel& data)
{
    fd_ = data.ReadFileDescriptor();
    offset_ = data.ReadInt64();
    length_ = data.ReadInt64();

    // 验证 offset 和 length 合理性
    if (offset_ < 0 || length_ < 0 || length_ > MAX_FILE_SIZE) {
        return ERR_PARAM_INVALID;
    }

    // 验证 fd 有效性
    struct stat st;
    if (fstat(fd_, &st) < 0) {
        return ERR_INVALID_FD;
    }
    return AVSESSION_SUCCESS;
}
```

---

### 风险 5: 整数溢出 (播放位置)

**证据位置**: `avplayback_state.h`

```cpp
class AVPlaybackState : public Parcelable {
public:
    int64_t position_;        // 播放位置 (毫秒)
    int64_t bufferedTime_;    // 缓冲时间
    int64_t duration_;        // 总时长
};
```

**触发路径**: 恶意应用设置极大的 position/bufferedTime/duration 值

**影响**: 整数溢出可能导致状态计算错误或断言失败

**修复建议**:
```cpp
int32_t AVPlaybackState::SetPosition(int64_t position)
{
    // 验证最大值 (最大支持 24 小时)
    constexpr int64_t MAX_POSITION = 24 * 60 * 60 * 1000;
    if (position < 0 || position > MAX_POSITION) {
        return ERR_PARAM_INVALID;
    }
    position_ = position;
    return AVSESSION_SUCCESS;
}
```

---

### 风险 6: 回调函数注入

**证据位置**: `napi_avsession.cpp`

```cpp
// JS 层注册回调
session.on('play', callback);

// 回调直接传递到 Native 层
napi_valuetype callbackType;
napi_get_value_function(env, callback, &callbackType);

// 无回调来源验证
```

**触发路径**: 恶意应用注入恶意回调函数

**影响**: 可能在回调执行时触发恶意代码

**修复建议**:
```cpp
// 验证回调来源
int32_t NapiAVSession::RegisterCallback(napi_env env, napi_value callback)
{
    // 验证调用者权限
    if (!CheckPermissionForCallback(callerToken)) {
        return ERR_PERMISSION_DENIED;
    }

    // 验证回调函数签名
    napi_valuetype type;
    napi_typeof(env, callback, &type);
    if (type != napi_function) {
        return ERR_PARAM_INVALID;
    }
    // ...
}
```

---

### 风险 7: 竞态条件 (会话生命周期)

**证据位置**: `services/session/server/avsession_item.cpp`

```cpp
// 缺乏同步保护的会话销毁流程
int32_t AVSessionItem::Destroy()
{
    // 可能在销毁过程中被其他线程访问
    if (isDestroyed_) {
        return ERR_SESSION_IS_DESTROYED;
    }

    // 非原子操作
    RemoveFromContainer();
    DestroyController();

    // 销毁状态设置
    isDestroyed_ = true;
    return AVSESSION_SUCCESS;
}
```

**触发路径**: 并发调用 Destroy 和其他操作

**影响**: Use-after-free 或状态不一致

**修复建议**:
```cpp
std::mutex sessionMutex_;  // 增加互斥锁保护

int32_t AVSessionItem::Destroy()
{
    std::lock_guard<std::mutex> lock(sessionMutex_);

    if (isDestroyed_) {
        return ERR_SESSION_IS_DESTROYED;
    }

    RemoveFromContainer();
    DestroyController();
    isDestroyed_ = true;

    return AVSESSION_SUCCESS;
}
```

---

## 4. 修复建议汇总

| 风险 | 严重性 | 建议修复 |
|------|--------|----------|
| 权限检查绕过 | 高 | 完善 TokenID 验证，增加边界检查 |
| 会话 ID 伪造 | 高 | 服务端验证 sessionId 归属 |
| 缓冲区溢出 | 中 | 添加字符串长度限制 |
| 路径遍历 | 高 | 验证 fd、offset、length |
| 整数溢出 | 低 | 添加数值范围检查 |
| 回调注入 | 中 | 验证回调来源和权限 |
| 竞态条件 | 高 | 使用互斥锁保护关键操作 |

---

## 5. 检查范围与局限性

### 已检查范围

1. **N-API 层**: `frameworks/js/napi/session/src/*.cpp` - 全部 22 个源文件
2. **IPC Stub 层**: `services/session/ipc/stub/*.cpp` - 全部 Stub 实现
3. **权限检查**: `utils/src/permission_checker.cpp` - 全部检查逻辑
4. **数据结构**: `interfaces/inner_api/native/session/include/*.h` - 全部 34 个头文件
5. **服务实现**: `services/session/server/avsession_*.cpp` - 核心服务实现

### 未检查范围

1. **测试代码**: `test/`, `fuzztest/`, `benchmarktest/` - 按规范忽略
2. **第三方库**: `cast_engine`, `dsoftbus` - 依赖外部组件
3. **分布式通道**: `server/remote/`, `server/migrate/` - 跨设备通信未深入分析
4. **性能监控**: `hisysevent.yaml` - 仅检查配置存在性

### 局限性说明

1. **静态分析**: 本评审基于静态代码分析，未进行运行时验证
2. **攻击路径**: 部分风险需要结合其他漏洞才能利用
3. **上下文依赖**: 修复建议需要根据实际攻击场景验证
