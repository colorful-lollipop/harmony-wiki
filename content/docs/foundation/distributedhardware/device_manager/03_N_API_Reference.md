# N-API 参考

## 1. 模块注册

### 1.1 注册入口

**文件**：`interfaces/kits/js4.0/src/native_devicemanager_js.cpp`

**行号**：3014

```cpp
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    LOGI("RegisterModule() is called!");
    napi_module_register(&g_dmModule);
}
```

**模块信息**：

```cpp
static napi_module g_dmModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Export,      // 导出函数
    .nm_modname = "distributedDeviceManager",
    .nm_priv = ((void *)0),
    .reserved = {0}
};
```

**导出函数**：`Export` - 定义 JS 接口属性和方法

> 证据来源：`native_devicemanager_js.cpp:3001-3015`

### 1.2 事件类型常量

```cpp
const std::string DM_NAPI_EVENT_DEVICE_STATE_CHANGE = "deviceStateChange";
const std::string DM_NAPI_EVENT_DEVICE_DISCOVER_SUCCESS = "discoverSuccess";
const std::string DM_NAPI_EVENT_DEVICE_DISCOVER_FAIL = "discoverFailure";
const std::string DM_NAPI_EVENT_DEVICE_PUBLISH_SUCCESS = "publishSuccess";
const std::string DM_NAPI_EVENT_DEVICE_PUBLISH_FAIL = "publishFail";
const std::string DM_NAPI_EVENT_DEVICE_SERVICE_DIE = "serviceDie";
const std::string DM_NAPI_EVENT_REPLY_RESULT = "replyResult";
const std::string DM_NAPI_EVENT_DEVICE_NAME_CHANGE = "deviceNameChange";
```

> 证据来源：`native_devicemanager_js.cpp:57-65`

## 2. JS API 清单

### 2.1 公共接口

| JS API | 方法类型 | 说明 |
|-------|---------|------|
| `createDeviceManager(bundleName)` | 同步 | 创建设备管理器实例 |
| `releaseDeviceManager(deviceManager)` | 同步 | 释放设备管理器实例 |

**使用示例**：

```javascript
try {
    let dmClass = deviceManager.createDeviceManager("ohos.samples.jshelloworld");
} catch(err) {
    console.error("createDeviceManager errCode:" + err.code);
}
```

### 2.2 设备信息查询

| JS API | 方法类型 | 说明 |
|-------|---------|------|
| `getAvailableDeviceListSync()` | 同步 | 获取可信设备列表 |
| `getLocalDeviceNetworkId()` | 同步 | 获取本地设备网络标识 |
| `getLocalDeviceName()` | 同步 | 获取本地设备名称 |
| `getLocalDeviceType()` | 同步 | 获取本地设备类型 |
| `getLocalDeviceId()` | 同步 | 获取本地设备 ID |
| `getDeviceName(networkId)` | 同步 | 根据网络标识获取设备名称 |
| `getDeviceType(networkId)` | 同步 | 根据网络标识获取设备类型 |

### 2.3 设备发现

| JS API | 方法类型 | 说明 |
|-------|---------|------|
| `startDiscovering(discoverParam, filterOptions?)` | 异步 | 开始发现周边设备 |
| `stopDiscovering()` | 异步 | 停止发现周边设备 |

**发现参数**：

```javascript
var discoverParam = {
    'discoverTargetType': 1  // 发现目标类型
};

var filterOptions = {
    'availableStatus': 1,        // 可用状态
    'discoverDistance': 50,       // 发现距离（厘米）
    'authenticationStatus': 0,    // 认证状态
    'authorizationType': 0        // 授权类型
};
```

### 2.4 设备认证

| JS API | 方法类型 | 说明 |
|-------|---------|------|
| `bindTarget(deviceId, bindParam, callback)` | 异步 | 认证设备 |
| `unbindTarget(deviceId)` | 同步 | 解除认证 |

**认证参数**：

```javascript
let bindParam = {
    'bindType': 1,           // 认证类型：1 - 无帐号 PIN 码认证
    'targetPkgName': 'xxxx', // 目标包名
    'appName': 'xxxx',       // 应用名称
    'appOperation': 'xxxx',  // 应用操作
    'customDescription': 'xxxx' // 自定义描述
};

dmClass.bindTarget(deviceId, bindParam, (err, data) => {
    if (err) {
        console.error("bindTarget failed:", err);
    } else {
        let pinToken = data.pinTone;
    }
});
```

### 2.5 设备发布

| JS API | 方法类型 | 说明 |
|-------|---------|------|
| `publishDeviceDiscovery(publishInfo)` | 异步 | 发布设备发现 |
| `unPublishDeviceDiscovery(publishId)` | 异步 | 取消发布 |

**发布参数**：

```javascript
var publishInfo = {
    'publishId': publishId,
    'mode': 0xAA,
    'freq': 2,
    'ranging': 1
};
```

### 2.6 事件监听

#### 设备状态变更

```javascript
dmClass.on('deviceStateChange', (data) => {
    console.info("deviceStateChange:", JSON.stringify(data));
});
dmClass.off('deviceStateChange');
```

#### 发现成功

```javascript
dmClass.on('discoverSuccess', (data) => {
    console.info("discoverSuccess:", JSON.stringify(data));
});
dmClass.off('discoverSuccess');
```

#### 发现失败

```javascript
dmClass.on('discoverFailure', (data) => {
    console.info("discoverFailure reason:", data.reason);
});
dmClass.off('discoverFailure');
```

#### 服务死亡

```javascript
dmClass.on('serviceDie', () => {
    console.info("DeviceManager service died");
});
dmClass.off('serviceDie');
```

#### 设备名称变更

```javascript
dmClass.on('deviceNameChange', (data) => {
    console.info("deviceNameChange:", data.deviceName);
});
dmClass.off('deviceNameChange');
```

### 2.7 系统接口（仅系统应用）

| JS API | 方法类型 | 说明 | 权限要求 |
|-------|---------|------|---------|
| `replyUiAction(action, actionResult)` | 异步 | 回复 UI 操作 | `ohos.permission.ACCESS_SERVICE_DP` |

```javascript
dmClass.replyUiAction(0, "extra");  // 0 - 允许授权
dmClass.on('replyResult', (data) => {
    console.info("replyResult:", JSON.stringify(data));
});
```

## 3. 回调管理机制

### 3.1 回调 Map

N-API 层使用 Map 管理回调：

```cpp
std::map<std::string, DeviceManagerNapi *> g_deviceManagerMap;
std::map<std::string, std::shared_ptr<DmNapiInitCallback>> g_initCallbackMap;
std::map<std::string, std::shared_ptr<DmNapiDeviceStatusCallback>> g_deviceStatusCallbackMap;
std::map<std::string, std::shared_ptr<DmNapiDiscoveryCallback>> g_DiscoveryCallbackMap;
std::map<std::string, std::shared_ptr<DmNapiAuthenticateCallback>> g_authCallbackMap;
std::map<std::string, std::shared_ptr<DmNapiPublishCallback>> g_publishCallbackMap;
std::map<std::string, std::shared_ptr<DmNapiBindTargetCallback>> g_bindCallbackMap;
```

### 3.2 互斥锁保护

```cpp
std::mutex g_deviceManagerMapMutex;
std::mutex g_initCallbackMapMutex;
std::mutex g_deviceStatusCallbackMapMutex;
std::mutex g_discoveryCallbackMapMutex;
std::mutex g_authCallbackMapMutex;
std::mutex g_bindCallbackMapMutex;
```

> 证据来源：`native_devicemanager_js.cpp:75-95`

## 4. 异步回调实现

### 4.1 UV 队列工作

```cpp
int ret = uv_queue_work_with_qos_internal(loop, work,
    [] (uv_work_t *work) {
        // 工作线程执行
    },
    [] (uv_work_t *work, int status) {
        // 完成回调（回到 JS 线程）
    },
    uv_qos_user_initiated,
    "CallbackName");
```

**质量-of-service 级别**：`uv_qos_user_initiated` - 用户发起操作

> 证据来源：`native_devicemanager_js.cpp:45-59`

### 4.2 Promise 支持

N-API 支持 Promise 风格调用：

```javascript
// Promise 风格
dmClass.getAvailableDeviceList()
    .then((deviceList) => {
        console.info("Device list:", deviceList);
    })
    .catch((err) => {
        console.error("Failed:", err);
    });

// Callback 风格
dmClass.getAvailableDeviceList((err, deviceList) => {
    if (err) {
        console.error("Failed:", err);
    } else {
        console.info("Device list:", deviceList);
    }
});
```

## 5. 错误处理

### 5.1 错误码

| 错误码 | 说明 |
|-------|------|
| `DM_NAPI_ARGS_ZERO` | 参数数量为 0 |
| `DM_NAPI_ARGS_ONE` | 参数数量为 1 |
| `DM_NAPI_ARGS_TWO` | 参数数量为 2 |
| `DM_NAPI_ARGS_THREE` | 参数数量为 3 |
| `DM_AUTH_REQUEST_SUCCESS_STATUS` | 认证请求成功状态 (7) |

> 证据来源：`native_devicemanager_js.cpp:67-73`

### 5.2 设备数量限制

```cpp
const int32_t DM_MAX_DEVICE_SIZE = 100;          // 最大设备数
const uint32_t DM_MAX_DEVICESLIST_SIZE = 50;     // 设备列表最大长度
```

### 5.3 错误回调格式

```javascript
{
    "code": <number>,      // 错误码
    "message": "<string>"   // 错误信息
}
```

## 6. 调用链示例

### 6.1 设备发现调用链

```
JS: dmClass.startDiscovering(discoverParam)
    │
    ▼
N-API: DeviceManagerNapi::StartDiscovering()
    │
    ▼
Inner SDK: DeviceManager::StartDiscovery()
    │
    ▼
IPC: SendRequest(CMD_START_DISCOVERY)
    │
    ▼
DeviceManager Service
    │
    ├──► SoftBus Connector
    │         │
    │         ▼
    │     StartDiscovery()
    │         │
    │         ▼
    │     发现结果回调
    │
    ▼
IPC 返回
    │
    ▼
N-API: discoverSuccess 事件
    │
    ▼
JS: callback(data)
```

## 7. 权限要求

### 7.1 基础权限（公开 API）

```javascript
// 调用以下接口需申请权限
ohos.permission.DISTRIBUTED_DATASYNC
```

### 7.2 系统权限（仅系统应用）

```javascript
// 仅系统应用可调用
ohos.permission.ACCESS_SERVICE_DP
```

## 8. 头文件清单

| 头文件 | 用途 |
|-------|------|
| `native_devicemanager_js.h` | N-API 主头文件 |
| `dm_native_util.h` | Native 工具函数 |
| `dm_native_event.h` | Native 事件处理 |

> 证据来源：`interfaces/kits/js4.0/include/`
