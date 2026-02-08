# N-API 接口文档

## 概述

castengine_wifi_display 提供 N-API 接口供应用开发者使用，JS 模块名为 **`multimedia.SharingWfd`**。

**注册位置**: `frameworks/kitsimpl/js/wfd/native_module_ohos_wfd.cpp:46-48`

```cpp
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&g_module);
}
```

## WfdSink API

WfdSink API 用于控制被投端（接收投屏流）。

### 类信息

| 属性 | 值 |
|------|-----|
| JS 类名 | `WfdSinkImpl` |
| 创建方法 | `createSink()` (静态) |
| C++ 实现 | `WfdSinkNapi` |

**注册位置**: `frameworks/kitsimpl/js/wfd/wfd_napi_sink.cpp:51-77`

### API 清单

#### 静态方法

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `createSink()` | 无 | `Promise<WfdSinkImpl>` | 创建 WfdSink 实例 |

#### 实例方法

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `start()` | 无 | `void` | 开始接收投屏 |
| `setDiscoverable(enable: boolean)` | `enable` - 是否开启可发现 | `void` | 设置是否可被发现 |
| `stop()` | 无 | `void` | 停止接收投屏 |

### 使用示例

```javascript
import { SharingWfd } from '@ohos.multimedia.sharingwfd';

async function startSink() {
  try {
    // 创建 Sink 实例
    const sink = await SharingWfd.createSink();
    console.log('Sink created successfully');

    // 设置可发现
    await sink.setDiscoverable(true);
    console.log('Discoverable enabled');

    // 开始接收投屏
    await sink.start();
    console.log('Sink started');
  } catch (error) {
    console.error('Failed to start sink:', error);
  }
}
```

### 调用链

```
JS: createSink()
    │
    ▼
WfdSinkNapi::CreateSink()
    │
    ├──► WfdSinkFactory::CreateSink()
    │         │
    │         ▼
    │     WfdSinkImpl 实例化
    │
    └──► IWfdEventListener 事件监听设置
```

### 错误处理

**错误码映射** (`frameworks/kitsimpl/js/wfd/wfd_napi_sink.cpp:34-37`):

```cpp
const char* CODE = "code";
const char* MSG_KEY = "msg";
const char* NAME_KEY = "name";
const char* ERROR_NAME = "BusinessError";
```

## WfdSource API

WfdSource API 用于控制主投端（发送投屏流）。

### 类信息

| 属性 | 值 |
|------|-----|
| JS 类名 | `WfdSourceImpl` |
| 创建方法 | `createSource()` (静态) |
| C++ 实现 | `WfdSourceNapi` |

**注册位置**: `frameworks/kitsimpl/js/wfd/wfd_napi_source.cpp:50-75`

### API 清单

#### 静态方法

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `createSource()` | 无 | `Promise<WfdSourceImpl>` | 创建 WfdSource 实例 |

#### 实例方法

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `startDiscovery()` | 无 | `void` | 开始发现可投屏设备 |
| `stopDiscovery()` | 无 | `void` | 停止发现 |

### 使用示例

```javascript
import { SharingWfd } from '@ohos.multimedia.sharingwfd';

async function startSource() {
  try {
    // 创建 Source 实例
    const source = await SharingWfd.createSource();
    console.log('Source created successfully');

    // 开始发现设备
    await source.startDiscovery();
    console.log('Discovery started');
  } catch (error) {
    console.error('Failed to start source:', error);
  }
}
```

### 调用链

```
JS: createSource()
    │
    ▼
WfdSourceNapi::CreateSource()
    │
    ├──► WfdSourceFactory::CreateSource()
    │         │
    │         ▼
    │     WfdSourceImpl 初始化
    │         │
    │         ├──► AbilityManagerClient::GetAbilityRunningInfos()
    │         │         │
    │         │         ▼
    │         │     获取当前应用信息
    │         │
    │         ├──► DmKit::InitDeviceManager()
    │         │         │
    │         │         ▼
    │         │     初始化设备管理
    │         │
    │         └──► RpcKeyParser::GetRpcKey()
    │                   │
    │                   ▼
    │               生成 RPC Key
    │
    └──► IWfdEventListener 事件监听设置
```

## 参数校验

### WfdSinkNapi 参数校验

**文件位置**: `frameworks/kitsimpl/js/wfd/wfd_napi_sink.cpp`

```cpp
const int32_t ARGS_ONE = 1;
const int32_t ARGS_TWO = 2;
const int32_t ARGS_THREE = 3;
const int32_t ARGS_FOUR = 4;
const int32_t STRING_MAX_SIZE = 255;
```

### WfdSourceNapi 参数校验

**文件位置**: `frameworks/kitsimpl/js/wfd/wfd_napi_source.cpp`

```cpp
const int32_t ARGS_ONE = 1;
const int32_t ARGS_TWO = 2;
const int32_t STRING_MAX_SIZE = 255;
```

## 生命周期管理

### WfdSinkNapi 生命周期

**构造函数**: `frameworks/kitsimpl/js/wfd/wfd_napi_sink.cpp:39-49`

```cpp
WfdSinkNapi::WfdSinkNapi()
{
    SHARING_LOGI("ctor %{public}p.", this);
}

WfdSinkNapi::~WfdSinkNapi()
{
    SHARING_LOGI("dtor %{public}p.", this);
    CancelCallbackReference();
    nativeWfdSink_.reset();
}
```

### WfdSourceNapi 生命周期

**构造函数**: `frameworks/kitsimpl/js/wfd/wfd_napi_source.cpp:38-48`

```cpp
WfdSourceNapi::WfdSourceNapi()
{
    SHARING_LOGI("ctor %{public}p.", this);
}

WfdSourceNapi::~WfdSourceNapi()
{
    SHARING_LOGI("dtor %{public}p.", this);
    CancelCallbackReference();
    nativeWfdSource_.reset();
}
```

## 权限要求

使用 N-API 接口需要以下权限：

| 权限 | 用途 |
|------|------|
| `ohos.permission.ACCESS_CAST_ENGINE_MIRROR` | 访问投屏引擎镜像 |
| `ohos.permission.CAMERA` | 相机权限（投屏可能涉及相机） |
| `ohos.permission.MICROPHONE` | 麦克风权限（投屏可能涉及音频） |

**配置文件**: `services/etc/sharing_service.cfg:28-41`

## 相关文档

- [Inner API 接口](04_Inner_API.md)
- [SA 配置](07_SA_Configuration.md)
- [安全风险评审](08_Security_Review.md)
