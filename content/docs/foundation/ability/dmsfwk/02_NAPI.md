# N-API 对外接口

## 1. 模块概览

dmsfwk 项目提供三个主要的 N-API 模块，暴露给 JS/TS 开发者使用。

## 2. ContinuationManager 模块

### 2.1 模块注册

| 属性 | 值 |
|------|-----|
| **文件** | `interfaces/kits/napi/continuation_manager/continuation_manager_module.cpp:27` |
| **模块名** | `continuation.continuationManager` |
| **注册函数** | `JsContinuationManagerInit` |

### 2.2 导出方法

| JS 方法 | C++ 函数 | 用途 |
|---------|----------|------|
| `register` | `JsContinuationManager::Register` | 注册接续 |
| `unregister` | `JsContinuationManager::Unregister` | 注销接续 |
| `on` | `JsContinuationManager::RegisterDeviceSelectionCallback` | 注册设备选择回调 |
| `off` | `JsContinuationManager::UnregisterDeviceSelectionCallback` | 注销设备选择回调 |
| `updateConnectStatus` | `JsContinuationManager::UpdateConnectStatus` | 更新连接状态 |
| `startDeviceManager` | `JsContinuationManager::StartDeviceManager` | 启动设备管理器 |
| `registerContinuation` | `JsContinuationManager::RegisterContinuation` | 注册接续（新版） |
| `unregisterContinuation` | `JsContinuationManager::UnregisterContinuation` | 注销接续（新版） |
| `updateContinuationState` | `JsContinuationManager::UpdateContinuationState` | 更新接续状态 |
| `startContinuationDeviceManager` | `JsContinuationManager::StartContinuationDeviceManager` | 启动接续设备管理器 |

### 2.3 初始化对象

| JS 对象 | C++ 函数 | 用途 |
|---------|----------|------|
| `DeviceConnectState` | `InitDeviceConnectStateObject` | 初始化设备连接状态对象 |
| `ContinuationMode` | `InitContinuationModeObject` | 初始化接续模式枚举 |

**证据来源**：`interfaces/kits/napi/continuation_manager/js_continuation_manager.h`

## 3. AbilityConnectionManager 模块

### 3.1 模块注册

| 属性 | 值 |
|------|-----|
| **文件** | `interfaces/kits/napi/ability_connection_manager/ability_connection_manager_module.cpp:27` |
| **模块名** | `distributedsched.abilityConnectionManager` |
| **注册函数** | `JsAbilityConnectionManagerInit` |

### 3.2 导出方法

| JS 方法 | C++ 函数 | 同步/异步 | 用途 |
|---------|----------|----------|------|
| `createAbilityConnectionSession` | `CreateAbilityConnectionSession` | 异步 | 创建能力连接会话 |
| `destroyAbilityConnectionSession` | `DestroyAbilityConnectionSession` | 异步 | 销毁能力连接会话 |
| `getPeerInfoById` | `GetPeerInfoById` | 异步 | 根据 ID 获取对端信息 |
| `on` | `RegisterAbilityConnectionSessionCallback` | 异步 | 注册会话回调 |
| `off` | `UnregisterAbilityConnectionSessionCallback` | 异步 | 注销会话回调 |
| `connect` | `Connect` | 异步（Promise） | 连接对端 |
| `disconnect` | `DisConnect` | 异步 | 断开连接 |
| `acceptConnect` | `AcceptConnect` | 异步（Promise） | 接受连接 |
| `reject` | `Reject` | 异步 | 拒绝连接 |
| `sendMessage` | `SendMessage` | 异步（Promise） | 发送消息 |
| `sendData` | `SendData` | 异步（Promise） | 发送数据 |
| `sendImage` | `SendImage` | 异步（Promise） | 发送图片 |
| `createStream` | `CreateStream` | 异步（Promise） | 创建音视频流 |
| `setSurfaceId` | `SetSurfaceId` | 异步 | 设置 Surface ID |
| `getSurfaceId` | `GetSurfaceId` | 异步 | 获取 Surface ID |
| `updateSurfaceParam` | `UpdateSurfaceParam` | 异步 | 更新 Surface 参数 |
| `destroyStream` | `DestroyStream` | 异步 | 销毁音视频流 |
| `startStream` | `StartStream` | 异步 | 启动音视频流 |
| `stopStream` | `StopStream` | 异步 | 停止音视频流 |

**证据来源**：`interfaces/kits/napi/ability_connection_manager/js_ability_connection_manager.h`

## 4. ContinuationStateManager 模块

### 4.1 模块注册

| 属性 | 值 |
|------|-----|
| **文件** | `interfaces/kits/napi/continuation_state_manager/js_continuation_state_manager.cpp:259` |
| **模块名** | `app.ability.continueManager` |

### 4.2 导出方法

| JS 方法 | C++ 函数 | 用途 |
|---------|----------|------|
| `on` | `ContinueStateCallbackOn` | 注册接续状态回调 |
| `off` | `ContinueStateCallbackOff` | 注销接续状态回调 |

## 5. 错误码定义

### 5.1 通用错误码

| 错误码 | 说明 | 用途 |
|--------|------|------|
| 201 | `PERMISSION_DENIED` | 权限拒绝 |
| 401 | `PARAMETER_CHECK_FAILED` | 参数检查失败 |
| 16600001 | `SYSTEM_WORK_ABNORMALLY` | 系统能力异常 |
| 16600002 | `CALLBACK_TOKEN_UNREGISTERED` | Token 或回调未注册 |
| 16600003 | `OVER_MAX_REGISTERED_TIMES` | 超过最大注册次数 |
| 16600004 | `REPEATED_REGISTRATION` | 重复注册 |

### 5.2 AbilityConnection 专用错误码

| 错误码 | 说明 |
|--------|------|
| 201 | `ERR_INVALID_PERMISSION` - 权限验证失败 |
| 202 | `ERR_NOT_SYSTEM_APP` - 调用方不是系统应用 |
| 401 | `ERR_INVALID_PARAMS` - 输入参数错误 |
| 801 | `ERR_CAPABILITY_NOT_SUPPORT` - 能力不支持 |
| 32300001 | `ERR_ONLY_SUPPORT_ONE_STREAM` - 不支持多流 |
| 32300002 | `ERR_RECEIVE_STREAM_NOT_START` - 接收端流未启动 |
| 32300003 | `ERR_BITATE_NOT_SUPPORTED` - 不支持该比特率 |
| 32300004 | `ERR_COLOR_SPACE_NOT_SUPPORTED` - 不支持该色彩空间 |

**证据来源**：`interfaces/kits/napi/include/napi_error_code.h`, `js_ability_connection_manager.h:57-74`

## 6. 参数校验

### 6.1 类型解析函数

| 函数 | 用途 |
|------|------|
| `napi_get_value_int32` | 解析 int32 参数 |
| `napi_get_value_string_utf8` | 解析字符串参数 |
| `napi_get_value_bool` | 解析布尔值参数 |

### 6.2 业务参数校验

| 校验类型 | 说明 |
|----------|------|
| `JsToServiceName` | 服务名称校验 |
| `JsToAbilityInfo` | AbilityInfo 对象校验 |
| `JsToPeerInfo` | 对端信息校验 |
| `JSToConnectOption` | 连接选项校验 |
| `JsToStreamParam` | 流参数校验 |
| `UnwrapOptions` | 连接选项解包 |
| `UnwrapStartOptions` | 启动选项解包 |

**证据来源**：`js_ability_connection_manager.h:106-158`

## 7. 异步处理模式

### 7.1 Promise + AsyncWork 模式

```cpp
// 步骤 1: 创建 Promise
napi_create_promise(env, &deferred);

// 步骤 2: 创建异步工作项
napi_create_async_work(env, nullptr, executeFunc, completeFunc, data, &asyncWork);

// 步骤 3: 队列执行
napi_queue_async_work(env, asyncWork);
```

### 7.2 ThreadSafeFunction 模式

用于连接回调等需要跨线程安全的场景：

```cpp
napi_create_threadsafe_function(env, js_func, NULL, callback, ...);
napi_call_threadsafe_function(tsfn, data, napi_tsfn_nonblocking);
```

### 7.3 异步方法列表

| 方法 | 异步模式 | 文件位置 |
|------|----------|----------|
| `Connect` | Promise + AsyncWork | `js_ability_connection_manager.cpp:964` |
| `AcceptConnect` | Promise + AsyncWork | `js_ability_connection_manager.cpp:1218` |
| `SendMessage` | Promise + AsyncWork | `js_ability_connection_manager.cpp:1291` |
| `SendData` | Promise + AsyncWork | `js_ability_connection_manager.cpp:1341` |
| `SendImage` | Promise + AsyncWork | `js_ability_connection_manager.cpp:1424` |
| `CreateStream` | Promise + AsyncWork | `js_ability_connection_manager.cpp:1511` |

**证据来源**：`js_ability_connection_manager.cpp` 异步模式分析

## 8. N-API 工具文件

| 文件 | 用途 |
|------|------|
| `include/napi_error_code.h` | N-API 错误码定义 |
| `js_ability_connection_manager.h` | AbilityConnectionManager 接口声明 |
| `js_continuation_manager.h` | ContinuationManager 接口声明 |
| `js_continuation_state_manager.h` | ContinuationStateManager 接口声明 |

## 9. JS 对象包装

### 9.1 napi_wrap 使用

| 文件 | 行号 | 用途 |
|------|------|------|
| `js_continuation_manager.cpp` | 1040 | 包装 JsContinuationManager 对象 |
| `distributed_extension_js.cpp` | 74, 176 | 包装 DistributedExtensionContext 弱指针 |
| `distributed_extension_context_js.cpp` | 212 | 包装 DistributedExtensionContextJS 对象 |

### 9.2 napi_define_properties 使用

主要用于定义模块导出方法，共 13 处使用。

## 10. 使用示例

### 10.1 设备接续注册

```typescript
import continuationManager from 'continuation.continuationManager';

const token = continuationManager.register({
  deviceType: ['phone', 'tablet'],
  continuationMode: 0,  // COLLABORATION_SINGLE
  filter: { ... }
});

continuationManager.on('deviceSelect', (deviceInfo) => {
  console.log('Selected device:', deviceInfo);
});
```

### 10.2 能力连接会话

```typescript
import abilityConnectionManager from 'distributedsched.abilityConnectionManager';

const session = await abilityConnectionManager.createAbilityConnectionSession({
  bundleName: 'com.example.remote',
  abilityName: 'EntryAbility'
});

session.on('connect', (result) => {
  console.log('Connected:', result);
});

await session.connect({ deviceId: 'remote-device-id' });
await session.sendMessage({ type: 'text', data: 'Hello!' });
```

**证据来源**：`interfaces/kits/napi/` 模块代码分析
