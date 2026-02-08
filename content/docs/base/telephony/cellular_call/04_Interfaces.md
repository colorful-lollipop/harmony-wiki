# 04 - Inner API 接口规范

## 目的

本文档描述 Cellular Call 模块提供的 **Inner API 接口规范**，包括 IMS Call 接口、Satellite Call 接口（条件编译）和补充业务接口。

⚠️ **重要说明**：本模块**不提供 N-API (JS API)**，所有接口均为 **Inner API**，仅供 OpenHarmony 系统内部调用（主要是 Call Manager）。

## 适用范围

- **读者对象**：系统开发者、模块集成者
- **使用场景**：
  - 了解可用的系统接口
  - 集成通话功能
  - 排查接口调用问题

---

## 4.1 接口分类总览

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Cellular Call Inner API                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    IMS Call 接口                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐  │    │
│  │  │ 基础通话    │ │ 补充业务    │ │ 视频通话                │  │    │
│  │  │ Dial/HangUp │ │ Hold/Unhold │ │ Camera/Preview/Display  │  │    │
│  │  │ Answer/     │ │ Switch/     │ │ Zoom/Direction/Pause   │  │    │
│  │  │ Reject      │ │ Conference   │ │                         │  │    │
│  │  └─────────────┘ └─────────────┘ └─────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │              Satellite Call 接口 (条件编译)                      │    │
│  │    Dial/HangUp/Answer/Reject/GetSatelliteCallsDataRequest      │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    补充业务接口                                   │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐  │    │
│  │  │ 呼叫转移    │ │ 呼叫等待    │ │ 呼叫限制                │  │    │
│  │  │ Transfer    │ │ Waiting     │ │ Restriction            │  │    │
│  │  └─────────────┘ └─────────────┘ └─────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4.2 IMS Call 接口

### 4.2.1 IMS Call 主接口定义

**接口位置**: `interfaces/innerkits/ims/ims_call_interface.h`

```cpp
class ImsCallInterface : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.Telephony.ImsCallInterface");
```

### 4.2.2 IMS Call API 清单

#### 4.2.2.1 基础通话类

| JS API | 参数 | 返回值 | 同步/异步 | C++ 实现 | 证据位置 |
|--------|------|--------|-----------|----------|----------|
| **Dial** | `ImsCallInfo`, `CLIRMode` | `int32_t` | 异步 | `Dial()` | `ims_call_interface.h:38` |
| **HangUp** | `ImsCallInfo` | `int32_t` | 异步 | `HangUp()` | `ims_call_interface.h:47` |
| **RejectWithReason** | `ImsCallInfo`, `ImsRejectReason` | `int32_t` | 异步 | `RejectWithReason()` | `ims_call_interface.h:57` |
| **Answer** | `ImsCallInfo` | `int32_t` | 异步 | `Answer()` | `ims_call_interface.h:66` |
| **HoldCall** | `slotId`, `callType`, `isRTT` | `int32_t` | 异步 | `HoldCall()` | `ims_call_interface.h:75` |
| **UnHoldCall** | `slotId`, `callType`, `isRTT` | `int32_t` | 异步 | `UnHoldCall()` | `ims_call_interface.h:84` |
| **SwitchCall** | `slotId`, `callType`, `isRTT` | `int32_t` | 异步 | `SwitchCall()` | `ims_call_interface.h:93` |
| **CombineConference** | `slotId` | `int32_t` | 异步 | `CombineConference()` | - |
| **SeparateConference** | `slotId`, `callId` | `int32_t` | 异步 | `SeparateConference()` | - |
| **InviteToConference** | `slotId`, `numberList` | `int32_t` | 异步 | `InviteToConference()` | - |
| **KickOutFromConference** | `slotId`, `callId` | `int32_t` | 异步 | `KickOutFromConference()` | - |

#### 4.2.2.2 视频通话类

| JS API | 参数 | 返回值 | 同步/异步 | C++ 实现 | 证据位置 |
|--------|------|--------|-----------|----------|----------|
| **ControlCamera** | `slotId`, `index`, `cameraId` | `int32_t` | 异步 | `ControlCamera()` | - |
| **SetPreviewWindow** | `slotId`, `index`, `surfaceId`, `Surface` | `int32_t` | 异步 | `SetPreviewWindow()` | - |
| **SetDisplayWindow** | `slotId`, `index`, `surfaceId`, `Surface` | `int32_t` | 异步 | `SetDisplayWindow()` | - |
| **SetCameraZoom** | `zoomRatio` | `int32_t` | 异步 | `SetCameraZoom()` | - |
| **SetPausePicture** | `slotId`, `index`, `path` | `int32_t` | 异步 | `SetPausePicture()` | - |
| **SetDeviceDirection** | `slotId`, `index`, `rotation` | `int32_t` | 异步 | `SetDeviceDirection()` | - |
| **RequestCameraCapabilities** | `slotId`, `index` | `int32_t` | 异步 | `RequestCameraCapabilities()` | - |

#### 4.2.2.3 DTMF 类

| JS API | 参数 | 返回值 | 同步/异步 | C++ 实现 | 证据位置 |
|--------|------|--------|-----------|----------|----------|
| **StartDtmf** | `slotId`, `callId`, `dtmfCode` | `int32_t` | 异步 | `StartDtmf()` | - |
| **SendDtmf** | `slotId`, `callId`, `dtmfCode` | `int32_t` | 异步 | `SendDtmf()` | - |
| **StopDtmf** | `slotId`, `callId` | `int32_t` | 异步 | `StopDtmf()` | - |

#### 4.2.2.4 IMS 配置类

| JS API | 参数 | 返回值 | 同步/异步 | C++ 实现 | 证据位置 |
|--------|------|--------|-----------|----------|----------|
| **SetImsConfig** | `slotId`, `ImsConfigItem`, `value` | `int32_t` | 异步 | `SetImsConfig()` | - |
| **GetImsConfig** | `slotId`, `ImsConfigItem` | `int32_t` | 异步 | `GetImsConfig()` | - |
| **SetImsFeatureValue** | `slotId`, `FeatureType`, `value` | `int32_t` | 异步 | `SetImsFeatureValue()` | - |
| **GetImsFeatureValue** | `slotId`, `FeatureType` | `int32_t` | 异步 | `GetImsFeatureValue()` | - |
| **SetVoNRState** | `slotId`, `state` | `int32_t` | 异步 | `SetVoNRState()` | - |
| **GetVoNRState** | `slotId`, `state` | `int32_t` | 异步 | `GetVoNRState()` | - |
| **SetMute** | `slotId`, `mute` | `int32_t` | 异步 | `SetMute()` | - |
| **GetMute** | `slotId` | `int32_t` | 异步 | `GetMute()` | - |

### 4.2.3 关键类型定义

#### ImsCallInfo

```cpp
struct ImsCallInfo {
    int32_t slotId;           // 卡槽 ID
    int32_t callId;           // 通话 ID
    int32_t callType;         // 通话类型 (语音/视频)
    std::string phoneNumber;  // 电话号码
};
```

#### CLIRMode

| 值 | 说明 |
|----|------|
| `CLIR_DEFAULT` | 默认 |
| `CLIR_INVOCATION` | 启用 CLIR |
| `CLIR_SUPPRESSION` | 禁用 CLIR |

#### ImsCallType

| 值 | 说明 |
|----|------|
| `TYPE_VOICE` | 语音通话 |
| `TYPE_VIDEO` | 视频通话 |
| `TYPE_CONFERENCE` | 会议通话 |

#### ImsRejectReason

| 值 | 说明 |
|----|------|
| `USER_IS_BUSY` | 用户忙 |
| `USER_DECLINE` | 用户拒绝 |

**证据位置**: `interfaces/innerkits/ims/ims_call_types.h`

---

## 4.3 Satellite Call 接口 (条件编译)

### 4.3.1 接口定义

**接口位置**: `interfaces/innerkits/satellite/satellite_call_interface.h`

```cpp
class SatelliteCallInterface : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.Telephony.SatelliteCallInterface");
```

### 4.3.2 Satellite Call API 清单

| JS API | 参数 | 返回值 | 同步/异步 | C++ 实现 | 证据位置 |
|--------|------|--------|-----------|----------|----------|
| **Dial** | `SatelliteCallInfo`, `CLIRMode` | `int32_t` | 异步 | `Dial()` | `satellite_call_interface.h:36` |
| **HangUp** | `slotId`, `index` | `int32_t` | 异步 | `HangUp()` | `satellite_call_interface.h:45` |
| **Reject** | `slotId` | `int32_t` | 异步 | `Reject()` | `satellite_call_interface.h:53` |
| **Answer** | `slotId` | `int32_t` | 异步 | `Answer()` | `satellite_call_interface.h:61` |
| **GetSatelliteCallsDataRequest** | `slotId` | `int32_t` | 异步 | `GetSatelliteCallsDataRequest()` | `satellite_call_interface.h:69` |
| **RegisterSatelliteCallCallback** | `SatelliteCallCallbackInterface` | `int32_t` | 异步 | `RegisterSatelliteCallCallback()` | `satellite_call_interface.h:77` |

### 4.3.3 条件编译说明

**启用条件**: `cellular_call_satellite = true` (在 `cellularcall.gni` 中配置)

**编译影响**: 当启用时，将额外编译以下文件：

| 文件 | 路径 |
|------|------|
| `cellular_call_connection_satellite.cpp` | `services/connection/src/` |
| `satellite_control.cpp` | `services/control/src/` |
| `satellite_call_callback_stub.cpp` | `services/satellite_service_interaction/src/` |
| `satellite_call_client.cpp` | `services/satellite_service_interaction/src/` |
| `satellite_call_proxy.cpp` | `services/satellite_service_interaction/src/` |

**证据位置**: `BUILD.gn:58-66`

---

## 4.4 Cellular Call 主接口 (IPC)

### 4.4.1 接口位置

**IPC 存根**: `services/manager/include/cellular_call_stub.h`

**主服务**: `services/manager/include/cellular_call_service.h`

### 4.4.2 Cellular Call API 清单

| API | 参数 | 返回值 | 同步/异步 | 权限检查 | 证据位置 |
|-----|------|--------|-----------|----------|----------|
| **Dial** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:102` |
| **HangUp** | `CellularCallInfo`, `CallSupplementType` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:111` |
| **Reject** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:119` |
| **Answer** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:127` |
| **HoldCall** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:135` |
| **UnHoldCall** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:143` |
| **SwitchCall** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:151` |
| **IsEmergencyPhoneNumber** | `slotId`, `phoneNum`, `enabled` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:161` |
| **SetEmergencyCallList** | `slotId`, `eccVec` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:170` |
| **CombineConference** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:178` |
| **SeparateConference** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:186` |
| **InviteToConference** | `slotId`, `numberList` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:195` |
| **KickOutFromConference** | `CellularCallInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:203` |
| **HangUpAllConnection** | - | `int32_t` | 异步 | ✅ | `cellular_call_service.h:210` |
| **HangUpAllConnection** | `slotId` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:218` |
| **SetCallTransferInfo** | `slotId`, `CallTransferInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:307` |
| **GetCallTransferInfo** | `slotId`, `CallTransferType` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:325` |
| **SetCallWaiting** | `slotId`, `activate` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:334` |
| **GetCallWaiting** | `slotId` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:342` |
| **SetCallRestriction** | `slotId`, `CallRestrictionInfo` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:360` |
| **GetCallRestriction** | `slotId`, `CallRestrictionType` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:369` |
| **SetDomainPreferenceMode** | `slotId`, `mode` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:390` |
| **GetDomainPreferenceMode** | `slotId` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:398` |
| **SetImsSwitchStatus** | `slotId`, `active` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:416` |
| **GetImsSwitchStatus** | `slotId`, `enabled` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:425` |
| **SetImsConfig** | `slotId`, `ImsConfigItem`, `value` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:435` |
| **GetImsConfig** | `slotId`, `ImsConfigItem` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:472` |
| **SendUpdateCallMediaModeRequest** | `CellularCallInfo`, `ImsCallMode` | `int32_t` | 异步 | ✅ | `cellular_call_service.h:229` |
| **... (50+ 接口)** | | | | | |

---

## 4.5 错误码定义

### 4.5.1 通用错误码

| 错误码 | 说明 | 证据位置 |
|--------|------|----------|
| `TELEPHONY_SUCCESS` | 操作成功 | - |
| `TELEPHONY_ERR_PERMISSION_ERR` | 权限错误 | `cellular_call_stub.cpp:49` |
| `TELEPHONY_ERR_DESCRIPTOR_MISMATCH` | 接口描述符不匹配 | `cellular_call_stub.cpp:40` |

### 4.5.2 通话特定错误码

详见 `call_manager_errors.h` (外部依赖)

---

## 4.6 权限与前置条件

### 4.6.1 权限要求

| 权限名 | 说明 | 检查位置 |
|--------|------|----------|
| `CONNECT_CELLULAR_CALL_SERVICE` | 连接蜂窝通话服务权限 | `cellular_call_stub.cpp:47` |

### 4.6.2 权限检查逻辑

```cpp
// 位置: services/manager/src/cellular_call_stub.cpp:45-50
auto callingUid = IPCSkeleton::GetCallingUid();
if (callingUid != FOUNDATION_UID &&
    !TelephonyPermission::CheckPermission(Permission::CONNECT_CELLULAR_CALL_SERVICE)) {
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

**免检条件**:
- 调用方 UID 为 `FOUNDATION_UID` (5523) 时免检
- 其他调用方必须检查 `CONNECT_CELLULAR_CALL_SERVICE` 权限

---

## 相关跳转

| 目标 | 链接 |
|------|------|
| 项目概览 | [01_Overview.md](./01_Overview.md) |
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| 内部 API | [05_Inner_API.md](./05_Inner_API.md) |
| 构建配置 | [06_GN_Build.md](./06_GN_Build.md) |

---

*最后更新：2026-02-06*
