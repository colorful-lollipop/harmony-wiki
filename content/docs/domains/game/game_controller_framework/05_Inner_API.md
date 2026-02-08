# 05_Inner_API - 内部 API

## 目的

本文档详细说明 GameController Framework 的内部 API，供终端设备厂商和架构师理解模块接口。

## 适用范围

- 终端设备厂商开发人员
- 框架架构师
- 维护人员

## InnerAPI 概览

InnerAPI 是 GameController Framework 提供给终端设备厂商的内部接口，用于：

1. 设备识别和管理
2. 按键映射配置同步
3. 事件广播

**证据**: `README_zh.md:6-8`

## API 清单表

| 方法 | 方向 | 权限要求 | 实现位置 | 文件:行号 |
|------|------|-----------|----------|-----------|
| IdentifyDevice | in/out | Any | frameworks/native/sa_client/include/gamecontroller_server_client.h:34 |
| SyncIdentifiedDeviceInfos | in | 系统服务 | frameworks/native/sa_client/include/gamecontroller_server_client.h:42 |
| SyncSupportKeyMappingGames | in | 系统服务 | frameworks/native/sa_client/include/gamecontroller_server_client.h:51 |
| GetGameKeyMappingConfig | in/out | Any | frameworks/native/sa_client/include/gamecontroller_server_client.h:59 |
| SetCustomGameKeyMappingConfig | in | 系统服务 | frameworks/native/sa_client/include/gamecontroller_server_client.h:67 |
| SetDefaultGameKeyMappingConfig | in | 系统服务 | frameworks/native/sa_client/include/gamecontroller_server_client.h:74 |
| BroadcastDeviceInfo | in | 系统应用或同 Bundle | frameworks/native/sa_client/include/gamecontroller_server_client.h:82 |
| BroadcastOpenTemplateConfig | in | 系统应用或同 Bundle | frameworks/native/sa_client/include/gamecontroller_server_client.h:90 |
| EnableGameKeyMapping | in | 系统服务 | frameworks/native/sa_client/include/gamecontroller_server_client.h:98 |

**证据**: `frameworks/native/sa_client/include/gamecontroller_server_client.h`

## 详细接口说明

### 1. 设备识别接口

#### IdentifyDevice

**功能**: 执行设备识别，判断设备类别（键盘、手柄、鼠标等）

**签名**:
```cpp
int32_t IdentifyDevice(const std::vector<DeviceInfo> &deviceInfos,
                   std::vector<DeviceInfo> &identifyResult);
```

**参数**:
- `deviceInfos`: 输入，待识别的设备信息列表
- `identifyResult`: 输出，识别结果

**权限**: 任意（Any）

**返回值**:
- 0: 成功
- 非0: 失败（错误码）

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:41-42`

**证据**: `frameworks/native/sa_client/include/gamecontroller_server_client.h:34`

#### SyncIdentifiedDeviceInfos

**功能**: 同步已识别的设备信息到 SA

**签名**:
```cpp
int32_t SyncIdentifiedDeviceInfos(const std::vector<IdentifiedDeviceInfo> &deviceInfos);
```

**参数**:
- `deviceInfos`: 已识别的设备信息列表

**权限**: 系统服务（System Service Only）

**权限验证**: `IsSystemServiceCall()`

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:115-118`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:42`
- `service/ipc/include/gamecontroller_server_ability.h:115-118`

### 2. 配置管理接口

#### SyncSupportKeyMappingGames

**功能**: 同步支持输入转触控的游戏列表

**签名**:
```cpp
int32_t SyncSupportKeyMappingGames(bool isSyncAll,
                                   const std::vector<GameInfo> &gameInfos);
```

**参数**:
- `isSyncAll`: true 表示全量同步，false 表示单条记录更新
- `gameInfos`: 同步的游戏信息列表

**权限**: 系统服务（System Service Only）

**权限验证**: `IsSystemServiceCall()`

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:124-127`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:51`
- `service/ipc/include/gamecontroller_server_ability.h:124-127`

#### GetGameKeyMappingConfig

**功能**: 获取游戏的按键映射配置

**签名**:
```cpp
int32_t GetGameKeyMappingConfig(const GetGameKeyMappingInfoParam &param,
                               GameKeyMappingInfo &gameKeyMappingInfo);
```

**参数**:
- `param`: 请求参数（包含游戏信息）
- `gameKeyMappingInfo`: 输出，按键映射配置信息

**权限**: 任意（Any）

**返回值**:
- 0: 成功
- 非0: 失败

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:66-67`

**证据**: `frameworks/native/sa_client/include/gamecontroller_server_client.h:59`

#### SetCustomGameKeyMappingConfig

**功能**: 设置自定义按键映射配置

**签名**:
```cpp
int32_t SetCustomGameKeyMappingConfig(const GameKeyMappingInfo &gameKeyMappingInfo);
```

**参数**:
- `gameKeyMappingInfo`: 自定义按键映射配置

**权限**: 系统服务（System Service Only）

**权限验证**: `IsSystemServiceCall()`

**配置文件**: 写入 `etc/config/custom_key_mapping.json`

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:98-100`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:67`
- `service/ipc/include/gamecontroller_server_ability.h:98-100`

#### SetDefaultGameKeyMappingConfig

**功能**: 设置默认按键映射配置

**签名**:
```cpp
int32_t SetDefaultGameKeyMappingConfig(const GameKeyMappingInfo &gameKeyMappingInfo);
```

**参数**:
- `gameKeyMappingInfo`: 默认按键映射配置

**权限**: 系统服务（System Service Only）

**权限验证**: `IsSystemServiceCall()`

**配置文件**: 写入 `etc/config/default_key_mapping.json`

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:134-137`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:74`
- `service/ipc/include/gamecontroller_server_ability.h:134-137`

### 3. 事件广播接口

#### BroadcastDeviceInfo

**功能**: 广播设备信息到应用

**签名**:
```cpp
int32_t BroadcastDeviceInfo(const GameInfo &gameInfo, const DeviceInfo &deviceInfo);
```

**参数**:
- `gameInfo`: 游戏信息
- `deviceInfo`: 设备信息

**权限**: 系统应用或同 Bundle（System App or same bundle）

**权限验证**:
- `IsSystemAppCall()`: 验证是否为系统应用
- `VerifyBundleNameIsValid()`: 验证 Bundle 名称匹配

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:143-150`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:82`
- `service/ipc/include/gamecontroller_server_ability.h:143-150`

#### BroadcastOpenTemplateConfig

**功能**: 广播打开配置页面通知

**签名**:
```cpp
int32_t BroadcastOpenTemplateConfig(const GameInfo &gameInfo, const DeviceInfo &deviceInfo);
```

**参数**:
- `gameInfo`: 游戏信息
- `deviceInfo`: 设备信息

**权限**: 系统应用或同 Bundle（System App or same bundle）

**权限验证**:
- `IsSystemAppCall()`: 验证是否为系统应用
- `VerifyBundleNameIsValid()`: 验证 Bundle 名称匹配

**触发条件**: README_zh.md:54 - 同时按下 Q、W、P 键

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:164-170`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:90`
- `service/ipc/include/gamecontroller_server_ability.h:164-170`

#### EnableGameKeyMapping

**功能**: 启用/禁用游戏的输入转触控功能

**签名**:
```cpp
int32_t EnableGameKeyMapping(const GameInfo &gameInfo, const bool isEnable);
```

**参数**:
- `gameInfo`: 游戏信息
- `isEnable`: true 表示启用，false 表示禁用

**权限**: 系统服务（System Service Only）

**权限验证**: `IsSystemServiceCall()`

**实现位置**: `service/ipc/src/gamecontroller_server_ability.cpp:207-210`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client.h:98`
- `service/ipc/include/gamecontroller_server_ability.h:207-210`

## 模块间依赖方向

### 客户端框架依赖

```
GameDevice (CAPI)
    ↓
    depends on:
        ├── MultiModalInputMonitor (设备监听）
        ├── InputEventClient (输入事件）
        └── BundleManager (Bundle 信息）
            ↓
            depends on:
                └── KeyMappingService (按键映射）
                        ↓
                    depends on:
                        ├── GameControllerServerClient (InnerAPI/IPC）
                        └── PluginManager (插件）
```

**证据**: `frameworks/native/BUILD.gn` - 依赖关系

### SA 端依赖

```
GameControllerServerAbility (SA)
    ↓
    depends on:
        ├── DeviceManager (设备识别）
        └── KeyMappingConfigManager (配置管理）
                    ↓
                depends on:
                    └── GameSupportKeyMappingManager (支持列表管理）
                            ↓
                        └── JsonUtils (文件读写）
```

**证据**: `service/BUILD.gn` - 依赖关系

## 稳定性标注

### 稳定接口（对外 InnerAPI）

以下接口为稳定接口，向后兼容保证：

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| IdentifyDevice | 稳定 | 设备识别核心接口 |
| GetGameKeyMappingConfig | 稳定 | 查询配置，变更风险低 |
| SyncIdentifiedDeviceInfos | 稳定 | 同步接口，由系统服务调用 |

**证据**: InnerAPI 定义在 `frameworks/native/sa_client/include/gamecontroller_server_client.h`

### 稳定接口（内部框架）

以下接口为内部框架接口，变更需谨慎：

| 模块 | 接口 | 稳定性 |
|------|------|--------|
| MultiModalInputMonitor | RegisterMonitorByUser/BySystem | 内部，可调整 |
| KeyMappingService | IsSupportGameKeyMapping | 依赖配置文件格式 |
| WindowInputIntercept | RegisterWindowInputIntercept | 依赖 Window Framework |
| PluginManager | 插件接口 | 扩展点，稳定 |

**证据**: 各模块头文件定义

### 不稳定接口（扩展点）

以下接口为扩展点，可能随版本变化：

| 模块 | 扩展点 | 稳定性 |
|------|--------|--------|
| PluginManager | 插件回调接口 | 低（扩展点）|

**证据**: `frameworks/native/plugin/include/plugin_event_callback.h`

## InnerAPI 调用示例

### 设备识别调用示例

```cpp
#include "gamecontroller_server_client.h"

// 1. 获取 InnerAPI 客户端（单例）
auto client = OHOS::GameController::GameControllerServerClient::GetInstance();

// 2. 准备设备信息
std::vector<OHOS::GameController::DeviceInfo> deviceInfos;
OHOS::GameController::DeviceInfo device;
device.deviceId = "device001";
device.name = "GamePad XYZ";
deviceInfos.push_back(device);

// 3. 调用设备识别
std::vector<OHOS::GameController::DeviceInfo> identifyResult;
int32_t ret = client->IdentifyDevice(deviceInfos, identifyResult);

if (ret == 0) {
    // 识别成功
    for (const auto& info : identifyResult) {
        printf("Device type: %d\n", info.deviceType);
    }
}
```

**证据**: `frameworks/native/sa_client/include/gamecontroller_server_client.h:25-35`

### 配置同步调用示例

```cpp
#include "gamecontroller_server_client.h"

auto client = OHOS::GameController::GameControllerServerClient::GetInstance();

// 1. 准备游戏列表
std::vector<OHOS::GameController::GameInfo> gameInfos;
OHOS::GameController::GameInfo gameInfo;
gameInfo.bundleName = "com.example.game1";
gameInfo.version = "1.0.0";
gameInfos.push_back(gameInfo);

// 2. 同步支持转触控的游戏列表（全量同步）
int32_t ret = client->SyncSupportKeyMappingGames(true, gameInfos);

if (ret == 0) {
    printf("Game list synchronized successfully\n");
}

// 3. 设置默认配置
OHOS::GameController::GameKeyMappingInfo keyMappingInfo;
// ... 填充 keyMappingInfo ...

ret = client->SetDefaultGameKeyMappingConfig(keyMappingInfo);

if (ret == 0) {
    printf("Default key mapping configured\n");
}
```

**证据**: `frameworks/native/sa_client/include/gamecontroller_server_client.h:51-74`

## 关键结论

1. **InnerAPI 分为三类**：设备识别、配置管理、事件广播
2. **权限分级**：
   - Any: 设备查询和一般配置
   - 系统服务：配置写入和同步
   - 系统应用或同 Bundle：事件广播
3. **依赖单向**：客户端框架 → SA，无循环依赖
4. **稳定性标注**：对外 InnerAPI 为稳定，内部框架接口谨慎变更

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [03_Architecture.md](./03_Architecture.md) - 完整架构说明
- [04_External_CAPI.md](./04_External_CAPI.md) - CAPI 接口详解

---

**版本**: 1.0 | **更新时间**: 2026-02-06
