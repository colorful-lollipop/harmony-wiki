# 对外 API

## 概述

Device Standby 部件提供两类对外 API：
1. **N-API**：供 JS 应用调用的 Node.js API
2. **Taihe (ArkTS)**：供 ArkTS 应用调用的 API

## N-API 接口

### 模块注册

| 项目 | 说明 |
|------|------|
| **注册函数** | `napi_module_register` |
| **注册位置** | `interfaces/kits/napi/src/init.cpp:84-87` |
| **模块名** | `devicestandby` |

```cpp
// interfaces/kits/napi/src/init.cpp
static napi_value DeviceStandbyInit(napi_env env, napi_value exports)
{
    DeviceStandbyFuncInit(env, exports);
    DeviceStandbyTypeInit(env, exports);
    return exports;
}

__attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&g_module);
}
```

### JS API 清单

| JS 方法 | C++ 实现 | 同步/异步 | 说明 |
|---------|----------|-----------|------|
| `isDeviceInStandby()` | `IsDeviceInStandby` | 异步 (Callback) | 查询设备是否待机 |
| `getExemptedApps(resourceTypes)` | `GetExemptionListApps` | 异步 (Callback/Promise) | 获取豁免应用列表 |
| `requestExemptionResource(request)` | `ApplyAllowResource` | 同步 | 申请豁免资源 |
| `releaseExemptionResource(request)` | `UnapplyAllowResource` | 同步 | 释放豁免资源 |
| `ResourceType` | 常量枚举 | 常量 | 资源类型枚举 |

### API 详细说明

#### isDeviceInStandby

```typescript
// 声明
function isDeviceInStandby(callback: AsyncCallback<boolean>): void;
function isDeviceInStandby(): Promise<boolean>;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| callback | AsyncCallback\<boolean\> | 否 | 回调函数（使用时必填） |

| 返回值 | 类型 | 说明 |
|--------|------|------|
| Promise\<boolean\> | Promise | 设备是否处于待机状态 |

**C++ 实现**：`interfaces/kits/napi/src/standby_napi_module.cpp:98`

```cpp
napi_value IsDeviceInStandby(napi_env env, napi_callback_info info)
{
    // 创建异步工作项
    napi_create_async_work(...);
    // 调用 StandbyServiceClient::IsDeviceInStandby()
}
```

**错误码**：
| 错误码 | 说明 |
|--------|------|
| `ERR_STANDBY_SYS_NOT_READY` | 服务未就绪 |
| 其他 | IPC 调用失败 |

#### getExemptedApps

```typescript
// 声明
function getExemptedApps(resourceTypes: number, callback: AsyncCallback<Array<ExemptedAppInfo>>): void;
function getExemptedApps(resourceTypes: number): Promise<Array<ExemptedAppInfo>>;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceTypes | number | 是 | 资源类型（ResourceType 枚举值） |
| callback | AsyncCallback\<Array\<ExemptedAppInfo\>\> | 否 | 回调函数 |

| 返回值 | 类型 | 说明 |
|--------|------|------|
| Promise\<Array\<ExemptedAppInfo\>\> | Promise | 豁免应用列表 |

**C++ 实现**：`interfaces/kits/napi/src/standby_napi_module.cpp:209`

**错误码**：
| 错误码 | 说明 |
|--------|------|
| `ERR_RESOURCE_TYPES_INVALID` | 无效的资源类型 |

#### requestExemptionResource

```typescript
// 声明
function requestExemptionResource(request: ResourceRequest): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| request | ResourceRequest | 是 | 豁免申请请求 |

**ResourceRequest 结构**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceTypes | number | 是 | 资源类型 |
| uid | number | 是 | 应用 UID |
| name | string | 是 | 应用名称 |
| duration | number | 是 | 豁免时长（秒） |
| reason | string | 是 | 申请原因 |

**C++ 实现**：`interfaces/kits/napi/src/standby_napi_module.cpp:283`

**权限要求**：`ohos.permission.DEVICE_STANDBY_EXEMPTION`

**错误码**：
| 错误码 | 说明 |
|--------|------|
| `ERR_PERMISSION_ERROR` | 权限校验失败 |
| `ERR_RESOURCE_TYPES_INVALID` | 无效的资源类型 |
| `ERR_DURATION_INVALID` | 无效的时长 |
| `ERR_STANDBY_SYS_NOT_READY` | 服务未就绪 |

#### releaseExemptionResource

```typescript
// 声明
function releaseExemptionResource(request: ResourceRequest): void;
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| request | ResourceRequest | 是 | 豁免释放请求 |

**C++ 实现**：`interfaces/kits/napi/src/standby_napi_module.cpp:294`

**权限要求**：`ohos.permission.DEVICE_STANDBY_EXEMPTION`

#### ResourceType 枚举

```typescript
// 声明
const ResourceType: {
    NETWORK: number;
    RUNNING_LOCK: number;
    TIMER: number;
    WORK_SCHEDULER: number;
    AUTO_SYNC: number;
    PUSH: number;
    FREEZE: number;
};
```

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `NETWORK` | 1 | 网络访问资源 |
| `RUNNING_LOCK` | 2 | CPU RunningLock 资源 |
| `TIMER` | 4 | 定时器任务资源 |
| `WORK_SCHEDULER` | 8 | Work 任务资源 |
| `AUTO_SYNC` | 16 | 自动同步资源 |
| `PUSH` | 32 | PushKit 资源 |
| `FREEZE` | 64 | 冻结应用资源 |

**定义位置**：`interfaces/kits/napi/src/init.cpp:47-58`

## Taihe (ArkTS) 接口

### 概述

Taihe 是 OpenHarmony 的 ArkTS 接口生成工具，通过 IDL 定义接口并自动生成 ArkTS 代码。

### IDL 定义位置

| 文件 | 说明 |
|------|------|
| `interfaces/kits/ani/idl/ohos.resourceschedule.deviceStandby.taihe` | Taihe IDL 定义 |

### Taihe Target

| Target | 类型 | 产物 |
|--------|------|------|
| `device_standby_ani` | 共享库 | `.so` / `.ani.cpp` |
| `device_standby_abc` | 静态 ABC | `.abc` |
| `device_standby_etc` | 预编译 ETC | 安装到 `/system/framework/` |

**BUILD.gn**：`interfaces/kits/ani/BUILD.gn`

### ArkTS API 清单

ArkTS API 与 N-API 基本对应：

| ArkTS 方法 | 对应 N-API | 说明 |
|------------|------------|------|
| `isDeviceInStandby()` | `isDeviceInStandby()` | 查询设备是否待机 |
| `getExemptedApps(resourceTypes)` | `getExemptedApps()` | 获取豁免应用列表 |
| `requestExemptionResource(request)` | `requestExemptionResource()` | 申请豁免资源 |
| `releaseExemptionResource(request)` | `releaseExemptionResource()` | 释放豁免资源 |

### 实现位置

| 文件 | 说明 |
|------|------|
| `interfaces/kits/ani/src/ohos.deviceStandby.impl.cpp` | Taihe 实现 |

```cpp
// interfaces/kits/ani/src/ohos.deviceStandby.impl.cpp
int32_t ret = StandbyServiceClient::GetInstance().ApplyAllowResource(resourceRequest);
```

## 使用示例

### JS 应用

```javascript
import deviceStandby from '@ohos.deviceStandby';

// 查询设备待机状态
deviceStandby.isDeviceInStandby().then((isStandby) => {
    console.log("设备是否待机:", isStandby);
});

// 获取豁免应用列表
deviceStandby.getExemptedApps(deviceStandby.ResourceType.NETWORK, (err, apps) => {
    if (err) {
        console.error("获取失败:", err);
        return;
    }
    console.log("豁免应用:", apps);
});

// 申请豁免
let request = {
    resourceTypes: deviceStandby.ResourceType.NETWORK | deviceStandby.ResourceType.TIMER,
    uid: 1000,
    name: "com.example.music",
    duration: 300,
    reason: "music_playback"
};
deviceStandby.requestExemptionResource(request);
```

### ArkTS 应用

```typescript
import deviceStandby from '@ohos.deviceStandby';

// 与 JS API 使用方式相同
let isStandby = await deviceStandby.isDeviceInStandby();
```

## 错误码定义

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | `ERR_OK` | 成功 |
| -1 | `ERR_STANDBY_SYS_NOT_READY` | 系统未就绪 |
| -2 | `ERR_RESOURCE_TYPES_INVALID` | 无效的资源类型 |
| -3 | `ERR_DURATION_INVALID` | 无效的时长 |
| -4 | `ERR_PERMISSION_ERROR` | 权限错误 |
| -5 | `ERR_STANDBY_ALLOWLIST_FULL` | 允许列表已满 |

**定义位置**：`utils/common/include/standby_service_errors.h`
