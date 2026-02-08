# Native API 参考

## 模块概述

Native API 主要供系统内部组件使用，包括框架层的内部接口和服务层的系统能力接口。

**证据来源**：
- `interfaces/innerkits/observer/telephony_observer.h:1-168` - 观察者接口
- `interfaces/innerkits/observer/telephony_observer_client.h:1-83` - 客户端接口
- `services/include/telephony_state_registry_service.h:36-117` - 服务接口

---

## Inner API 接口清单

### TelephonyObserver 接口

**定义位置**: `interfaces/innerkits/observer/telephony_observer.h:29-164`

| 方法名 | 功能 | 参数 | 返回值 | 行号 |
|--------|------|------|--------|------|
| `OnCallStateUpdated` | 通话状态变化回调 | `int32_t slotId`, `int32_t callState`, `const std::u16string &phoneNumber` | void | 41-42 |
| `OnCallStateUpdatedEx` | 通话状态变化回调(扩展) | `int32_t slotId`, `int32_t callStateEx` | void | 129-130 |
| `OnCCallStateUpdated` | C 语言通话状态回调 | `int32_t slotId`, `int32_t callState`, `const std::u16string &phoneNumber` | void | 139-140 |
| `OnSignalInfoUpdated` | 信号信息变化回调 | `int32_t slotId`, `const std::vector<sptr<SignalInformation>> &vec` | void | 50-51 |
| `OnNetworkStateUpdated` | 网络状态变化回调 | `int32_t slotId`, `const sptr<NetworkState> &networkState` | void | 59-60 |
| `OnCellInfoUpdated` | 小区信息变化回调 | `int32_t slotId`, `const std::vector<sptr<CellInformation>> &vec` | void | 68-69 |
| `OnSimStateUpdated` | SIM 卡状态变化回调 | `int32_t slotId`, `CardType type`, `SimState state`, `LockReason reason` | void | 79-80 |
| `OnCellularDataConnectStateUpdated` | 数据连接状态变化回调 | `int32_t slotId`, `int32_t dataState`, `int32_t networkType` | void | 89-90 |
| `OnCellularDataFlowUpdated` | 蜂窝数据流变化回调 | `int32_t slotId`, `int32_t dataFlowType` | void | 98-99 |
| `OnCfuIndicatorUpdated` | CFU 指示器更新回调 | `int32_t slotId`, `bool cfuResult` | void | 108 |
| `OnVoiceMailMsgIndicatorUpdated` | 语音信箱消息指示器回调 | `int32_t slotId`, `bool voiceMailMsgResult` | void | 118 |
| `OnIccAccountUpdated` | ICC 账户更新回调 | 无 | void | 121 |
| `OnRemoteRequest` | IPC 请求处理 | `uint32_t code`, `MessageParcel &data`, `MessageParcel &reply`, `MessageOption &option` | int32_t | 119-120 |

**内部实现方法** (私有):

| 方法名 | 功能 | 行号 |
|--------|------|------|
| `ConvertSignalInfoList` | 信号信息列表转换 | 145 |
| `ConvertLteNrSignalInfoList` | LTE/NR 信号信息转换 | 146-147 |
| `ConvertCellInfoList` | 小区信息列表转换 | 148 |
| `OnCallStateUpdatedInner` | 通话状态更新内部处理 | 149 |
| `OnSignalInfoUpdatedInner` | 信号信息更新内部处理 | 150 |
| `OnNetworkStateUpdatedInner` | 网络状态更新内部处理 | 151 |
| `OnCellInfoUpdatedInner` | 小区信息更新内部处理 | 152 |
| `OnSimStateUpdatedInner` | SIM 状态更新内部处理 | 153 |
| `OnCellularDataConnectStateUpdatedInner` | 数据连接状态内部处理 | 154 |
| `OnCellularDataFlowUpdatedInner` | 数据流内部处理 | 155 |
| `OnCfuIndicatorUpdatedInner` | CFU 指示器内部处理 | 156 |
| `OnVoiceMailMsgIndicatorUpdatedInner` | 语音信箱内部处理 | 157 |
| `OnIccAccountUpdatedInner` | ICC 账户内部处理 | 158 |
| `OnCallStateUpdatedExInner` | 通话状态扩展内部处理 | 159 |
| `OnCCallStateUpdatedInner` | C 通话状态内部处理 | 160 |

**常量定义**:
- `CELL_NUM_MAX = 100` - 最大小区数量限制 (line 161)
- `SIGNAL_NUM_MAX = 100` - 最大信号数量限制 (line 162)

---

### TelephonyObserverClient 接口

**定义位置**: `interfaces/innerkits/observer/telephony_observer_client.h:27-82`

| 方法名 | 功能 | 参数 | 返回值 | 行号 |
|--------|------|------|--------|------|
| `AddStateObserver` | 添加状态观察者 | `const sptr<TelephonyObserverBroker> &telephonyObserver`, `int32_t slotId`, `uint32_t mask`, `bool isUpdate` | int32_t | 40-41 |
| `RemoveStateObserver` | 移除状态观察者 | `int32_t slotId`, `uint32_t mask` | int32_t | 50 |
| `GetProxy` | 获取 State Registry 代理 | 无 | `sptr<ITelephonyStateNotify>` | 57 |

**内部类**:

| 类名 | 功能 | 行号 |
|------|------|------|
| `StateRegistryDeathRecipient` | 服务死亡监听 | 60-71 |
| `OnRemoteDied` | 远程服务死亡处理 | 64-67 |

**成员变量**:
- `mutexProxy_` - 代理互斥锁 (line 76)
- `proxy_` - IPC 代理对象 (line 77)
- `deathRecipient_` - 死亡监听对象 (line 78)

---

## 服务层接口

### TelephonyStateRegistryService

**定义位置**: `services/include/telephony_state_registry_service.h:36-117`

| 方法名 | 功能 | 行号 |
|--------|------|------|
| `OnStart` | 服务启动 | 42 |
| `OnStop` | 服务停止 | 43 |
| `OnDump` | Dump 信息输出 | 44 |
| `Dump` | 调试信息输出 | 45 |
| `RegisterStateChange` | 注册状态变更监听 | 61-63 |
| `UnregisterStateChange` | 注销状态变更监听 | 64 |
| `UpdateCellularDataConnectState` | 更新蜂窝数据连接状态 | 49 |
| `UpdateCellularDataFlow` | 更新蜂窝数据流 | 50 |
| `UpdateCallState` | 更新通话状态 | 51 |
| `UpdateCallStateForSlotId` | 更新指定卡槽通话状态 | 52-53 |
| `UpdateSignalInfo` | 更新信号信息 | 54 |
| `UpdateNetworkState` | 更新网络状态 | 55 |
| `UpdateSimState` | 更新 SIM 状态 | 56 |
| `UpdateCellInfo` | 更新小区信息 | 57 |
| `UpdateCfuIndicator` | 更新 CFU 指示器 | 58 |
| `UpdateVoiceMailMsgIndicator` | 更新语音信箱指示器 | 59 |
| `UpdateIccAccount` | 更新 ICC 账户 | 60 |
| `CheckCallerIsSystemApp` | 检查调用者是否为系统应用 | 80 |
| `CheckPermission` | 检查权限 | 81 |
| `VerifySlotId` | 验证卡槽 ID | 82 |

**私有方法**:
- `Finalize` - 资源清理 (line 75)
- `UpdateData` - 更新数据 (line 76)
- `UpdateDataEx` - 扩展数据更新 (line 77)
- `GetCallIncomingNumberForSlotId` - 获取指定卡槽来电号码 (line 83)
- `PublishCommonEvent` - 发布公共事件 (line 84)
- `SendCallStateChanged` - 发送通话状态变化 (line 85)
- `SendSignalInfoChanged` - 发送信号信息变化 (line 87)
- `SendNetworkStateChanged` - 发送网络状态变化 (line 88)
- `SendSimStateChanged` - 发送 SIM 状态变化 (line 89)
- `SendCellularDataConnectStateChanged` - 发送数据连接状态变化 (line 90)
- `IsCommonEventServiceAbilityExist` - 检查公共事件服务是否存在 (line 91)

---

### TelephonyStateRegistryStub

**定义位置**: `services/include/telephony_state_registry_stub.h:29-80`

| 方法名 | 功能 | 行号 |
|--------|------|------|
| `OnRemoteRequest` | IPC 请求分发 | 34-35 |
| `OnUpdateCellInfo` | 处理小区信息更新 | 56 |
| `OnUpdateCallState` | 处理通话状态更新 | 57 |
| `OnUpdateCallStateForSlotId` | 处理指定卡槽通话状态更新 | 58 |
| `OnUpdateSignalInfo` | 处理信号信息更新 | 59 |
| `OnUpdateNetworkState` | 处理网络状态更新 | 60 |
| `OnUpdateSimState` | 处理 SIM 状态更新 | 61 |
| `OnRegisterStateChange` | 处理注册状态变更 | 62 |
| `OnUnregisterStateChange` | 处理注销状态变更 | 63 |
| `OnUpdateCellularDataConnectState` | 处理数据连接状态更新 | 64 |
| `OnUpdateCellularDataFlow` | 处理数据流更新 | 65 |
| `OnUpdateCfuIndicator` | 处理 CFU 指示器更新 | 66 |
| `OnUpdateVoiceMailMsgIndicator` | 处理语音信箱指示器更新 | 67 |
| `OnIccAccountUpdated` | 处理 ICC 账户更新 | 68 |
| `ReadData` | 读取 IPC 数据 | 44 |
| `parseSignalInfos` | 解析信号信息 | 48 |
| `ParseLteNrSignalInfos` | 解析 LTE/NR 信号信息 | 50-51 |
| `SetTimer` | 设置超时定时器 | 69 |
| `CancelTimer` | 取消定时器 | 70 |

---

## IPC 接口码

**定义位置**: 通过 `i_telephony_state_notify.h` 引入 (外部依赖)

| 接口码 | 枚举值 | 处理函数 | 说明 |
|--------|--------|----------|------|
| `CELL_INFO` | 0 | `OnUpdateCellInfo` | 小区信息更新 |
| `SIM_STATE` | 1 | `OnUpdateSimState` | SIM 状态更新 |
| `SIGNAL_INFO` | 2 | `OnUpdateSignalInfo` | 信号信息更新 |
| `NET_WORK_STATE` | 3 | `OnUpdateNetworkState` | 网络状态更新 |
| `CALL_STATE` | 4 | `OnUpdateCallState` | 通话状态更新 |
| `CALL_STATE_FOR_ID` | 5 | `OnUpdateCallStateForSlotId` | 指定卡槽通话状态更新 |
| `CELLULAR_DATA_STATE` | 6 | `OnUpdateCellularDataConnectState` | 数据连接状态更新 |
| `CELLULAR_DATA_FLOW` | 7 | `OnUpdateCellularDataFlow` | 数据流更新 |
| `ADD_OBSERVER` | 8 | `OnRegisterStateChange` | 添加观察者 |
| `REMOVE_OBSERVER` | 9 | `OnUnregisterStateChange` | 移除观察者 |
| `CFU_INDICATOR` | 10 | `OnUpdateCfuIndicator` | CFU 指示器更新 |
| `VOICE_MAIL_MSG_INDICATOR` | 11 | `OnUpdateVoiceMailMsgIndicator` | 语音信箱指示器更新 |
| `ICC_ACCOUNT_CHANGE` | 12 | `OnIccAccountUpdated` | ICC 账户变更 |

**注册映射**: `services/src/telephony_state_registry_stub.cpp:36-61`

```cpp
// 构造函数中的接口码映射
TelephonyStateRegistryStub::TelephonyStateRegistryStub() {
    memberFuncMap_[StateNotifyInterfaceCode::CELL_INFO] =
        [this](MessageParcel &data, MessageParcel &reply) { return OnUpdateCellInfo(data, reply); };
    memberFuncMap_[StateNotifyInterfaceCode::SIM_STATE] =
        [this](MessageParcel &data, MessageParcel &reply) { return OnUpdateSimState(data, reply); };
    // ... 其他映射
}
```

---

## 关键数据类型

### 观察者掩码 (Observer Mask)

**定义位置**: 通过 `telephony_observer_broker.h` 引入 (外部依赖)

| 掩码常量 | 值 | 对应事件 |
|----------|-----|----------|
| `OBSERVER_MASK_NETWORK_STATE` | 0x01 | networkStateChange |
| `OBSERVER_MASK_CALL_STATE` | 0x02 | callStateChange |
| `OBSERVER_MASK_CELL_INFO` | 0x04 | cellInfoChange |
| `OBSERVER_MASK_SIGNAL_STRENGTHS` | 0x08 | signalInfoChange |
| `OBSERVER_MASK_SIM_STATE` | 0x10 | simStateChange |
| `OBSERVER_MASK_DATA_CONNECTION_STATE` | 0x20 | cellularDataConnectionStateChange |
| `OBSERVER_MASK_DATA_FLOW` | 0x40 | cellularDataFlowChange |
| `OBSERVER_MASK_CFU_INDICATOR` | 0x80 | cfuIndicatorChange |
| `OBSERVER_MASK_VOICE_MAIL_MSG_INDICATOR` | 0x100 | voiceMailMsgIndicatorChange |
| `OBSERVER_MASK_CALL_STATE_EX` | 0x200 | callStateChangeEx |
| `OBSERVER_MASK_CCALL_STATE` | 0x400 | cCallStateChange |

**N-API 层映射**: `frameworks/js/napi/include/napi_state_registry.h:34-40`

```cpp
constexpr int32_t LISTEN_NET_WORK_STATE = TelephonyObserverBroker::OBSERVER_MASK_NETWORK_STATE;      // 0x01
constexpr int32_t LISTEN_CALL_STATE = TelephonyObserverBroker::OBSERVER_MASK_CALL_STATE;             // 0x02
constexpr int32_t LISTEN_CELL_INFO = TelephonyObserverBroker::OBSERVER_MASK_CELL_INFO;               // 0x04
constexpr int32_t LISTEN_SIGNAL_STRENGTHS = TelephonyObserverBroker::OBSERVER_MASK_SIGNAL_STRENGTHS; // 0x08
constexpr int32_t LISTEN_SIM_STATE = TelephonyObserverBroker::OBSERVER_MASK_SIM_STATE;               // 0x10
constexpr int32_t LISTEN_DATA_CONNECTION_STATE = TelephonyObserverBroker::OBSERVER_MASK_DATA_CONNECTION_STATE; // 0x20
constexpr int32_t LISTEN_CELLULAR_DATA_FLOW = TelephonyObserverBroker::OBSERVER_MASK_DATA_FLOW;      // 0x40
```

---

## 调用链示例

### 观察者注册调用链

```
1. JS 层调用
   observer.on('callStateChange', {slotId: 0}, callback)
   └─> @ohos.telephony.observer.d.ts

2. N-API 层处理
   └─> frameworks/js/napi/src/napi_state_registry.cpp:75
       NativeOn(env, data)
       ├─> 参数解析: MatchParametersWithObject (line 140)
       ├─> slotId 验证: IsValidSlotIdEx (line 62)
       └─> EventListenerManager::RegisterEventListener (line 104)

3. 事件监听管理
   └─> frameworks/js/napi/src/event_listener_manager.cpp:22
       RegisterEventListener(eventListener)
       └─> EventListenerHandler::RegisterEventListener

4. Native 客户端调用
   └─> frameworks/native/observer/src/telephony_observer_client.cpp:40
       AddStateObserver(telephonyObserver, slotId, mask, isUpdate)
       └─> GetProxy() -> IPC 调用

5. IPC 传输
   └─> Binder IPC -> telecom 进程

6. 服务端处理
   └─> services/src/telephony_state_registry_stub.cpp:52
       OnRegisterStateChange(data, reply)
       └─> services/src/telephony_state_registry_service.cpp:61
           RegisterStateChange(...)
           ├─> 权限检查: CheckPermission (line 80-81)
           ├─> 系统应用检查: CheckCallerIsSystemApp (line 80)
           └─> 添加到记录列表: stateRecords_
```

### 状态更新调用链 (以通话状态为例)

```
1. Call Manager 触发
   CallManager -> IPC -> telecom 进程

2. Stub 层接收
   └─> services/src/telephony_state_registry_stub.cpp:44
       OnUpdateCallState(data, reply)
       ├─> data.ReadInt32() -> callState (line 132)
       └─> data.ReadString16() -> phoneNumber (line 133)

3. 服务层处理
   └─> services/src/telephony_state_registry_service.cpp:189
       UpdateCallState(callState, number)
       ├─> 权限检查: CheckPermission(SET_TELEPHONY_STATE) (line 191)
       ├─> 状态存储: callState_[-1] = callState (line 197)
       └─> 遍历观察者列表 (line 202)

4. 通知观察者
   对于每个匹配的观察者:
   ├─> 检查监听掩码: IsExistStateListener(OBSERVER_MASK_CALL_STATE) (line 204)
   ├─> 权限检查: IsCanReadCallHistory() (line 207)
   │   ├─> 有权限: phoneNumber = number
   │   └─> 无权限: phoneNumber = "" (脱敏)
   └─> IPC 回调: telephonyObserver_->OnCallStateUpdated(...) (line 212)

5. 发布公共事件
   └─> SendCallStateChanged(slotId, callState) (line 226)
       └─> SendCallStateChangedAsUserMultiplePermission (line 227)
           └─> PublishCommonEvent (line 84)
```

---

## 依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                     Inner API Consumer                       │
│                 (框架层、系统服务层)                          │
├─────────────────────────────────────────────────────────────┤
│                     TelephonyObserver                        │
│              interfaces/innerkits/observer/                  │
│                    (观察者回调接口)                          │
├────────────────────────┬────────────────────────────────────┤
│   TelephonyObserverClient │      TelephonyStateRegistryStub │
│ frameworks/native/observer │          services/src/          │
│     (客户端封装)         │           (IPC Stub)              │
├────────────────────────┴────────────────────────────────────┤
│                TelephonyStateRegistryService                 │
│                   services/src/                              │
│                      (服务实现)                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 头文件包含关系

### 使用 TelephonyObserverClient

```cpp
#include "telephony_observer_client.h"  // interfaces/innerkits/observer/telephony_observer_client.h:16

void RegisterTelephonyObserver() {
    auto& client = TelephonyObserverClient::GetInstance();
    int32_t ret = client.AddStateObserver(observer, 0, mask, false);
    if (ret != 0) {
        TELEPHONY_LOGE("AddStateObserver failed, ret=%{public}d", ret);
    }
}
```

### 使用 TelephonyObserver

```cpp
#include "telephony_observer.h"  // interfaces/innerkits/observer/telephony_observer.h:16

class MyObserver : public TelephonyObserver {
public:
    void OnCallStateUpdated(int32_t slotId, int32_t callState, 
                           const std::u16string &phoneNumber) override {
        // 处理通话状态变化
        // line 41-42
    }
};
```

---

## 版本兼容性

| API 版本 | 接口状态 | 说明 |
|----------|----------|------|
| 4.0 | 稳定 | 当前版本，无破坏性变更 |
| 3.2 | 稳定 | 兼容历史版本 |
| 3.1 | 稳定 | 兼容历史版本 |

---

## 相关文档

- [目录结构](01_Directory_Structure.md)
- [架构设计](02_Architecture.md)
- [JS API](03_JS_API.md)
- [攻击面分析](05_AttackSurface.md)
- [GN 构建](05_GN_Build.md)
- [安全风险评估](07_Security_Review.md)
