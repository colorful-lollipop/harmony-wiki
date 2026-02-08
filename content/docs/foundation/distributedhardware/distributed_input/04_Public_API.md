# 公共 API (Public API)

## 目的

本文档描述 distributed_input 模块的公共 API（Inner SDK），包括 C++ 接口、方法签名、参数说明、权限要求和错误码。

## 适用范围

本文档适用于以下场景：
- 多模输入模块开发者调用分布式输入能力
- 理解 Inner SDK 的接口设计
- 掌握 API 的使用方法和约束条件
- 进行集成开发和调试

## 关键结论

1. **纯 C++ 接口**: Inner SDK 仅提供 C++ API，无 JavaScript/N-API 层
2. **异步回调模式**: 所有异步操作使用回调机制（SPTR<ICallback>）
3. **权限保护**: 所有 API 调用都需要相应权限（详见[安全评审](08_Security_Review.md)）
4. **重载支持**: 核心方法提供多种重载以支持不同调用场景

## API 概览

DistributedInputKit 提供以下核心 API：

| 类别 | 方法 | 说明 |
|------|------|------|
| **准备/取消准备** | `PrepareRemoteInput()` / `UnprepareRemoteInput()` | 准备/取消跨设备输入 |
| **启动/停止** | `StartRemoteInput()` / `StopRemoteInput()` | 启动/停止跨设备输入 |
| **设备查询** | `IsStartDistributedInput()` | 查询设备是否在分布式输入状态 |
| **事件过滤** | `IsNeedFilterOut()` / `IsTouchEventNeedFilterOut()` | 判断事件是否需要过滤 |
| **事件监听** | `RegisterSimulationEventListener()` / `UnregisterSimulationEventListener()` | 注册/注销仿真事件监听器 |
| **会话状态** | `RegisterSessionStateCb()` / `UnregisterSessionStateCb()` | 注册/注销会话状态回调 |

## API 详细说明

### 1. 准备远程输入 (PrepareRemoteInput)

#### 1.1 单设备版本

```cpp
// SinkId 版本
static int32_t PrepareRemoteInput(
    const std::string &sinkId,
    sptr<IPrepareDInputCallback> callback
);

// SrcId + SinkId 版本（Relay 模式）
static int32_t PrepareRemoteInput(
    const std::string &srcId,
    const std::string &sinkId,
    sptr<IPrepareDInputCallback> callback
);
```

**位置**: [distributed_input_kit.h:41-47](../interfaces/inner_kits/include/distributed_input_kit.h:41-47)

**参数**:

| 参数 | 类型 | 说明 | 验证 |
|------|------|------|------|
| `sinkId` | `const std::string&` | Sink 设备 ID | 非空字符串 |
| `srcId` | `const std::string&` | Source 设备 ID | 非空字符串 |
| `callback` | `sptr<IPrepareDInputCallback>` | 异步回调接口 | 不能为 nullptr |

**返回值**:

| 返回码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 准备成功 |
| `ERR_DH_INPUT_SA_NOT_READY` | -67001 | SA 未就绪 |
| `ERR_DH_INPUT_IPC_CHECK_FAILED` | -67003 | IPC 检查失败 |
| `ERR_DH_INPUT_PERMISSION_DENIED` | -67004 | 权限不足 |
| `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | -67045 | IPC token 写入验证失败 |
| `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | -67046 | IPC token 读取验证失败 |

**权限要求**:
- 需要 `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` 权限

**回调接口**: `IPrepareDInputCallback`

```cpp
class IPrepareDInputCallback : public IRemoteBroker {
    virtual void OnResult(const std::string &devId, const int32_t &status) = 0;
    virtual ~IPrepareDInputCallback() = default;
};
```

**位置**: [i_prepare_d_input_call_back.h](../frameworks/include/i_prepare_d_input_call_back.h)

**回调参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `devId` | `const std::string&` | 目标设备 ID |
| `status` | `const int32_t&` | 操作状态码 |

**说明**:
- 异步操作，结果通过回调返回
- 准备成功后，会话状态变为 `SESSION_STATE_PREPARED`
- 准备操作建立 SoftBus 会话，但未开始事件传输

---

### 2. 取消准备远程输入 (UnprepareRemoteInput)

```cpp
// SinkId 版本
static int32_t UnprepareRemoteInput(
    const std::string &sinkId,
    sptr<IUnprepareDInputCallback> callback
);

// SrcId + SinkId 版本（Relay 模式）
static int32_t UnprepareRemoteInput(
    const std::string &srcId,
    const std::string &sinkId,
    sptr<IUnprepareDInputCallback> callback
);
```

**位置**: [distributed_input_kit.h:42-43,45-47](../interfaces/inner_kits/include/distributed_input_kit.h:42-43)

**参数**:

| 参数 | 类型 | 说明 | 验证 |
|------|------|------|------|
| `sinkId` | `const std::string&` | Sink 设备 ID | 非空字符串 |
| `srcId` | `const std::string&` | Source 设备 ID | 非空字符串 |
| `callback` | `sptr<IUnprepareDInputCallback>` | 异步回调接口 | 不能为 nullptr |

**返回值**:

| 返回码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 取消准备成功 |
| `ERR_DH_INPUT_IPC_CHECK_FAILED` | -67003 | IPC 检查失败 |
| `ERR_DH_INPUT_PERMISSION_DENIED` | -67004 | 权限不足 |
| `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | -67045 | IPC token 写入验证失败 |
| `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | -67046 | IPC token 读取验证失败 |

**权限要求**:
- 需要 `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` 权限

**回调接口**: `IUnprepareDInputCallback`

```cpp
class IUnprepareDInputCallback : public IRemoteBroker {
    virtual void OnResult(const std::string &devId, const int32_t &status) = 0;
    virtual ~IUnprepareDInputCallback() = default;
};
```

**位置**: [i_unprepare_d_input_call_back.h](../frameworks/include/i_unprepare_d_input_call_back.h)

**回调参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `devId` | `const std::string&` | 目标设备 ID |
| `status` | `const int32_t&` | 操作状态码 |

**说明**:
- 异步操作，结果通过回调返回
- 取消准备会关闭 SoftBus 会话并释放资源
- 会话状态变为 `SESSION_STATE_UNPREPARED`

---

### 3. 启动远程输入 (StartRemoteInput)

#### 3.1 按输入类型启动

```cpp
// SinkId 版本
static int32_t StartRemoteInput(
    const std::string &sinkId,
    const uint32_t &inputTypes,
    sptr<IStartDInputCallback> callback
);

// SrcId + SinkId 版本（Relay 模式）
static int32_t StartRemoteInput(
    const std::string &srcId,
    const std::string &sinkId,
    const uint32_t &inputTypes,
    sptr<IStartDInputCallback> callback
);
```

**位置**: [distributed_input_kit.h:49-51,59-61](../interfaces/inner_kits/include/distributed_input_kit.h:49-51)

**参数**:

| 参数 | 类型 | 说明 | 验证 |
|------|------|------|------|
| `sinkId` | `const std::string&` | Sink 设备 ID | 非空字符串 |
| `srcId` | `const std::string&` | Source 设备 ID | 非空字符串 |
| `inputTypes` | `const uint32_t&` | 输入类型位掩码 | 非零 |
| `callback` | `sptr<IStartDInputCallback>` | 异步回调接口 | 不能为 nullptr |

**inputTypes 位掩码**:

| 常量 | 值 | 说明 |
|------|-----|------|
| `MOUSE` | 0x1 | 鼠标输入 |
| `KEYBOARD` | 0x2 | 键盘输入 |
| `TOUCHSCREEN` | 0x4 | 触摸屏输入 |
| `JOYSTICK` | 0x8 | 摇杆输入 |

可以组合使用，例如：`MOUSE | KEYBOARD` 表示同时启用鼠标和键盘

**返回值**:

| 返回码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 启动成功 |
| `ERR_DH_INPUT_IPC_CHECK_FAILED` | -67003 | IPC 检查失败 |
| `ERR_DH_INPUT_PERMISSION_DENIED` | -67004 | 权限不足 |
| `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | -67045 | IPC token 写入验证失败 |
| `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | -67046 | IPC token 读取验证失败 |

**权限要求**:
- 需要 `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` 权限

**回调接口**: `IStartDInputCallback`

```cpp
class IStartDInputCallback : public IRemoteBroker {
    virtual void OnResult(const std::string &devId, const uint32_t &inputTypes, const int32_t &status) = 0;
    virtual ~IStartDInputCallback() = default;
};
```

**位置**: [i_start_d_input_call_back.h](../frameworks/include/i_start_d_input_call_back.h)

**回调参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `devId` | `const std::string&` | 目标设备 ID |
| `inputTypes` | `const uint32_t&` | 实际启动的输入类型 |
| `status` | `const int32_t&` | 操作状态码 |

**说明**:
- 异步操作，结果通过回调返回
- 启动成功后，Sink 侧开始采集并发送事件，Source 侧开始接收并注入事件
- 会话状态变为 `SESSION_STATE_STARTED`
- Sink 设备状态从 `THROUGH_OUT` 变为 `THROUGH_IN`

#### 3.2 按设备 ID 列表启动

```cpp
// SinkId 版本
static int32_t StartRemoteInput(
    const std::string &sinkId,
    const std::vector<std::string> &dhIds,
    sptr<IStartStopDInputsCallback> callback
);

// SrcId + SinkId 版本（Relay 模式）
static int32_t StartRemoteInput(
    const std::string &srcId,
    const std::string &sinkId,
    const std::vector<std::string> &dhIds,
    sptr<IStartStopDInputsCallback> callback
);
```

**位置**: [distributed_input_kit.h:54-56,64-67](../interfaces/inner_kits/include/distributed_input_kit.h:54-56)

**参数**:

| 参数 | 类型 | 说明 | 验证 |
|------|------|------|------|
| `sinkId` | `const std::string&` | Sink 设备 ID | 非空字符串 |
| `srcId` | `const std::string&` | Source 设备 ID | 非空字符串 |
| `dhIds` | `const std::vector<std::string>&` | 分布式硬件设备 ID 列表 | 非空列表 |
| `callback` | `sptr<IStartStopDInputsCallback>` | 异步回调接口 | 不能为 nullptr |

**返回值**: 同按输入类型启动

**权限要求**:
- 需要 `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` 权限

**回调接口**: `IStartStopDInputsCallback`

```cpp
class IStartStopDInputsCallback : public IRemoteBroker {
    virtual void OnResult(const std::string &devId, const std::vector<std::string> &dhIds, const int32_t &status) = 0;
    virtual ~IStartStopDInputsCallback() = default;
};
```

**位置**: [i_start_stop_d_inputs_call_back.h](../frameworks/include/i_start_stop_d_inputs_call_back.h)

**回调参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `devId` | `const std::string&` | 目标设备 ID |
| `dhIds` | `const std::vector<std::string>&` | 实际启动的设备 ID 列表 |
| `status` | `const int32_t&` | 操作状态码 |

---

### 4. 停止远程输入 (StopRemoteInput)

#### 4.1 按输入类型停止

```cpp
// SinkId 版本
static int32_t StopRemoteInput(
    const std::string &sinkId,
    const uint32_t &inputTypes,
    sptr<IStopDInputCallback> callback
);

// SrcId + SinkId 版本（Relay 模式）
static int32_t StopRemoteInput(
    const std::string &srcId,
    const std::string &sinkId,
    const uint32_t &inputTypes,
    sptr<IStopDInputCallback> callback
);
```

**位置**: [distributed_input_kit.h:51-53,60-62](../interfaces/inner_kits/include/distributed_input_kit.h:51-53)

**参数**: 同启动远程输入

**返回值**:

| 返回码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 停止成功 |
| `ERR_DH_INPUT_IPC_CHECK_FAILED` | -67003 | IPC 检查失败 |
| `ERR_DH_INPUT_PERMISSION_DENIED` | -67004 | 权限不足 |
| `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | -67045 | IPC token 写入验证失败 |
| `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | -67046 | IPC token 读取验证失败 |

**权限要求**:
- 需要 `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` 权限

**回调接口**: `IStopDInputCallback`

```cpp
class IStopDInputCallback : public IRemoteBroker {
    virtual void OnResult(const std::string &devId, const uint32_t &inputTypes, const int32_t &status) = 0;
    virtual ~IStopDInputCallback() = default;
};
```

**位置**: [i_stop_d_input_call_back.h](../frameworks/include/i_stop_d_input_call_back.h)

**回调参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `devId` | `const std::string&` | 目标设备 ID |
| `inputTypes` | `const uint32_t&` | 实际停止的输入类型 |
| `status` | `const int32_t&` | 操作状态码 |

**说明**:
- 停止成功后，Sink 侧停止采集并发送事件，Source 侧停止接收并注入事件
- 会话状态保持 `SESSION_STATE_PREPARED`
- Sink 设备状态从 `THROUGH_IN` 变为 `THROUGH_OUT`

#### 4.2 按设备 ID 列表停止

```cpp
// SinkId 版本
static int32_t StopRemoteInput(
    const std::string &sinkId,
    const std::vector<std::string> &dhIds,
    sptr<IStartStopDInputsCallback> callback
);

// SrcId + SinkId 版本（Relay 模式）
static int32_t StopRemoteInput(
    const std::string &srcId,
    const std::string &sinkId,
    const std::vector<std::string> &dhIds,
    sptr<IStartStopDInputsCallback> callback
);
```

**位置**: [distributed_input_kit.h:56-57,65-67](../interfaces/inner_kits/include/distributed_input_kit.h:56-57)

**参数**: 同启动远程输入

**回调接口**: `IStartStopDInputsCallback`（与启动版本相同）

**说明**:
- 停止指定设备 ID 列表的跨设备输入
- 如果停止所有设备，则相当于停止所有类型的跨设备输入

---

### 5. 查询分布式输入状态 (IsStartDistributedInput)

#### 5.1 按输入类型查询

```cpp
static DInputServerType IsStartDistributedInput(const uint32_t &inputType);
```

**位置**: [distributed_input_kit.h:72](../interfaces/inner_kits/include/distributed_input_kit.h:72)

**参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `inputType` | `const uint32_t&` | 输入类型（MOUSE、KEYBOARD、TOUCHSCREEN 等） | - |

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `NULL_SERVER_TYPE` | 0x0 | 未启动 |
| `SOURCE_SERVER_TYPE` | 0x1 | 作为 Source 运行 |
| `SINK_SERVER_TYPE` | 0x2 | 作为 Sink 运行 |

**说明**:
- 同步接口，直接返回结果
- 用于查询当前设备在分布式输入中的角色

#### 5.2 按设备 ID 查询

```cpp
static bool IsStartDistributedInput(const std::string &dhId);
```

**位置**: [distributed_input_kit.h:78](../interfaces/inner_kits/include/distributed_input_kit.h:78)

**参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `dhId` | `const std::string&` | 分布式硬件设备 ID | 非空字符串 |

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `true` | 设备正在跨设备共享 |
| `false` | 设备未在跨设备共享 |

**说明**:
- 同步接口，直接返回结果
- 用于查询特定设备是否正在跨设备输入

---

### 6. 事件过滤 (IsNeedFilterOut)

#### 6.1 业务事件过滤

```cpp
static bool IsNeedFilterOut(const std::string &sinkId, const BusinessEvent &event);
```

**位置**: [distributed_input_kit.h:69](../interfaces/inner_kits/include/distributed_input_kit.h:69)

**参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `sinkId` | `const std::string&` | Sink 设备 ID | - |
| `event` | `const BusinessEvent&` | 业务事件（类型、代码、值） | - |

**BusinessEvent 结构**（根据 [constants_dinput.h](../common/include/constants_dinput.h:132-141)）:

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | `uint32_t` | 事件类型（MOUSE、KEYBOARD、TOUCHSCREEN、RELATIVE 等） |
| `code` | `uint32_t` | 事件代码（键码、鼠标按键等） |
| `value` | `uint32_t` | 事件值（按下/释放、坐标等） |

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `true` | 事件需要过滤（不在白名单或需本地生效） |
| `false` | 事件不过滤（可以跨设备传输） |

**说明**:
- 同步接口，直接返回结果
- 用于在 Source 侧判断接收到的事件是否需要过滤
- 白名单配置文件：`/etc/distributedhardware/dinput_business_event_whitelist.cfg`

**实现位置**: [white_list_util.cpp](../common/include/white_list_util.cpp:16-334)

#### 6.2 触摸屏事件过滤

```cpp
static bool IsTouchEventNeedFilterOut(const TouchScreenEvent &event);
```

**位置**: [distributed_input_kit.h:70](../interfaces/inner_kits/include/distributed_input_kit.h:70)

**参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `event` | `const TouchScreenEvent&` | 触摸屏事件 | - |

**返回值**: 同业务事件过滤

**说明**:
- 同步接口，直接返回结果
- 专门用于触摸屏事件的过滤判断

---

### 7. 仿真事件监听器 (RegisterSimulationEventListener)

```cpp
static int32_t RegisterSimulationEventListener(sptr<ISimulationEventListener> listener);
static int32_t UnregisterSimulationEventListener(sptr<ISimulationEventListener> listener);
```

**位置**: [distributed_input_kit.h:80-81](../interfaces/inner_kits/include/distributed_input_kit.h:80-81)

**RegisterSimulationEventListener 参数**:

| 参数 | 类型 | 说明 | 验证 |
|------|------|------|
| `listener` | `sptr<ISimulationEventListener>` | 仿真事件监听器接口 | 不能为 nullptr |

**返回值**:

| 返回码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 注册成功 |
| `ERR_DH_INPUT_INVALID_PARAM` | -67002 | 参数无效 |
| `ERR_DH_INPUT_IPC_CHECK_FAILED` | -67003 | IPC 检查失败 |

**UnregisterSimulationEventListener 参数**: 同注册

**回调接口**: `ISimulationEventListener`

```cpp
class ISimulationEventListener : public IRemoteBroker {
    virtual void OnSimulationEvent(const std::string &deviceId, const std::string &dhId,
        const std::string &type, const std::string &code, const std::string &value) = 0;
    virtual ~ISimulationEventListener() = default;
};
```

**位置**: [i_simulation_event_listener.h](../frameworks/include/i_simulation_event_listener.h)

**回调参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `deviceId` | `const std::string&` | 设备 ID |
| `dhId` | `const std::string&` | 分布式硬件设备 ID |
| `type` | `const std::string&` | 事件类型 |
| `code` | `const std::string&` | 事件代码 |
| `value` | `const std::string&` | 事件值 |

**说明**:
- 异步注册，仿真事件通过回调通知
- 用于监听跨设备的仿真输入事件

---

### 8. 会话状态回调 (RegisterSessionStateCb)

```cpp
static int32_t RegisterSessionStateCb(sptr<ISessionStateCallback> callback);
static int32_t UnregisterSessionStateCb();
```

**位置**: [distributed_input_kit.h:83-84](../interfaces/inner_kits/include/distributed_input_kit.h:83-84)

**RegisterSessionStateCb 参数**:

| 参数 | 类型 | 说明 | 验证 |
|------|------|------|
| `callback` | `sptr<ISessionStateCallback>` | 会话状态回调接口 | 不能为 nullptr |

**返回值**:

| 返回码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 注册成功 |
| `ERR_DH_INPUT_INVALID_PARAM` | -67002 | 参数无效 |
| `ERR_DH_INPUT_IPC_CHECK_FAILED` | -67003 | IPC 检查失败 |

**回调接口**: `ISessionStateCallback`

```cpp
class ISessionStateCallback : public IRemoteBroker {
    virtual void OnSessionState(const std::string &deviceId, const std::string &dhId,
        const int32_t &state, const int32_t &reason) = 0;
    virtual ~ISessionStateCallback() = default;
};
```

**位置**: [i_session_state_callback.h](../frameworks/include/i_session_state_callback.h)

**回调参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `deviceId` | `const std::string&` | 设备 ID |
| `dhId` | `const std::string&` | 分布式硬件设备 ID |
| `state` | `const int32_t&` | 会话状态（0=INIT, 1=PREPARED, 2=STARTED, 3=STOPPED, 4=UNPREPARED） |
| `reason` | `const int32_t&` | 状态变化原因 |

**说明**:
- 异步注册，会话状态变化时通知
- 用于监控跨设备输入会话的生命周期

## 权限要求汇总

### API 权限矩阵

| API | 需要权限 | 证据 |
|------|---------|------|
| `PrepareRemoteInput()` | `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | [distributed_input_source_stub.cpp:139](../interfaces/ipc/src/distributed_input_source_stub.cpp:139) |
| `StartRemoteInput()` | `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | [distributed_input_source_stub.cpp:179](../interfaces/ipc/src/distributed_input_source_stub.cpp:179) |
| `StopRemoteInput()` | `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | [distributed_input_source_stub.cpp:200](../interfaces/ipc/src/distributed_input_source_stub.cpp:200) |
| `UnprepareRemoteInput()` | `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | [distributed_input_source_stub.cpp:159](../interfaces/ipc/src/distributed_input_source_stub.cpp:159) |
| `IsNeedFilterOut()` | 无权限要求（本地查询） | - |
| `RegisterSimulationEventListener()` | `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | [distributed_input_source_stub.cpp:419](../interfaces/ipc/src/distributed_input_source_stub.cpp:419) |
| `RegisterSessionStateCb()` | `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | [distributed_input_source_stub.cpp:519](../interfaces/ipc/src/distributed_input_source_stub.cpp:519) |

**权限验证位置**:

- **Source Stub**: [distributed_input_source_stub.cpp:36-52](../interfaces/ipc/src/distributed_input_source_stub.cpp:36-52) - `HasAccessDHPermission()` 方法
- **Sink Stub**: [distributed_input_sink_stub.cpp:40-47](../interfaces/ipc/src/distributed_input_sink_stub.cpp:40-47) - `HasEnableDHPermission()` 方法

## 错误码

### 核心错误码（部分）

根据 [dinput_errcode.h](../common/include/dinput_errcode.h)：

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 操作成功 |
| `ERR_DH_INPUT_INVALID_PARAM` | -67002 | 参数无效 |
| `ERR_DH_INPUT_SA_NOT_READY` | -67001 | SA 未就绪 |
| `ERR_DH_INPUT_IPC_CHECK_FAILED` | -67003 | IPC 检查失败 |
| `ERR_DH_INPUT_PERMISSION_DENIED` | -67004 | 权限不足 |
| `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | -67045 | IPC token 写入验证失败 |
| `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | -67046 | IPC token 读取验证失败 |
| `ERR_DH_INPUT_SERVER_SOURCE_TRANSPORT_PERMISSION_DENIED` | -65042 | Transport 层权限被拒绝 |
| `ERR_DH_INPUT_SRC_ENABLE_PERMISSION_CHECK_FAIL` | -67061 | Source ENABLE_DISTRIBUTED_HARDWARE 权限检查失败 |
| `ERR_DH_INPUT_SRC_ACCESS_PERMISSION_CHECK_FAIL` | -67062 | Source ACCESS_DISTRIBUTED_HARDWARE 权限检查失败 |
| `ERR_DH_INPUT_SINK_ENABLE_PERMISSION_CHECK_FAIL` | -67063 | Sink ENABLE_DISTRIBUTED_HARDWARE 权限检查失败 |

### IPC 错误码

| 错误码 | 值 | 说明 | 位置 |
|--------|-----|------|
| `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | -67045 | [distributed_input_source_stub.cpp:582](../interfaces/ipc/src/distributed_input_source_stub.cpp:582) |
| `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | -67046 | [distributed_input_source_stub.cpp:582](../interfaces/ipc/src/distributed_input_source_stub.cpp:582) |

### 权限错误码

| 错误码 | 值 | 说明 | 位置 |
|--------|-----|------|
| `ERR_DH_INPUT_SRC_ENABLE_PERMISSION_CHECK_FAIL` | -67061 | [distributed_input_source_stub.cpp:56](../interfaces/ipc/src/distributed_input_source_stub.cpp:56) |
| `ERR_DH_INPUT_SRC_ACCESS_PERMISSION_CHECK_FAIL` | -67062 | [distributed_input_source_stub.cpp:76](../interfaces/ipc/src/distributed_input_source_stub.cpp:76) |
| `ERR_DH_INPUT_SINK_ENABLE_PERMISSION_CHECK_FAIL` | -67063 | [distributed_input_sink_stub.cpp:77](../interfaces/ipc/src/distributed_input_sink_stub.cpp:77) |

## API 使用示例

### 示例 1: 准备并启动跨设备输入

```cpp
#include "distributed_input_kit.h"

using namespace OHOS::DistributedHardware::DistributedInput;

// 准备远程输入
std::string sinkId = "network_id_of_sink_device";
auto callback = sptr(new PrepareDInputCallbackImpl());
int32_t ret = DistributedInputKit::PrepareRemoteInput(sinkId, callback);
if (ret != SUCCESS) {
    // 处理错误
    return;
}

// 等待准备完成（回调中处理）

// 启动远程输入（启用鼠标和键盘）
uint32_t inputTypes = MOUSE | KEYBOARD;
auto startCallback = sptr(new StartDInputCallbackImpl());
ret = DistributedInputKit::StartRemoteInput(sinkId, inputTypes, startCallback);
if (ret != SUCCESS) {
    // 处理错误
    return;
}
```

### 示例 2: 停止跨设备输入

```cpp
// 停止远程输入
auto stopCallback = sptr(new StopDInputCallbackImpl());
int32_t ret = DistributedInputKit::StopRemoteInput(sinkId, inputTypes, stopCallback);
if (ret != SUCCESS) {
    // 处理错误
    return;
}

// 等待停止完成（回调中处理）
```

### 示例 3: 查询设备状态

```cpp
// 查询当前设备角色
DInputServerType serverType = DistributedInputKit::IsStartDistributedInput(MOUSE);
switch (serverType) {
    case SOURCE_SERVER_TYPE:
        // 当前作为 Source 运行
        break;
    case SINK_SERVER_TYPE:
        // 当前作为 Sink 运行
        break;
    case NULL_SERVER_TYPE:
        // 未启动分布式输入
        break;
}
```

### 示例 4: 注册事件监听器

```cpp
// 注册仿真事件监听器
auto listener = sptr(new SimulationEventListenerImpl());
int32_t ret = DistributedInputKit::RegisterSimulationEventListener(listener);
if (ret != SUCCESS) {
    // 处理错误
    return;
}

// 接收仿真事件（通过 ISimulationEventListener::OnSimulationEvent）
```

## 相关跳转

- [项目定位](01_Project_Positioning.md) - 功能边界和核心能力
- [内部 API](05_Inner_API.md) - 模块间接口
- [安全评审](08_Security_Review.md) - 详细的权限和安全机制
- [常见问题](09_QA_and_Debugging.md) - 调试和问题定位

---

*更新时间: 2026-02-06 15:08:55*
