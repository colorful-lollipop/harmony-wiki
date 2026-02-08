# 内部模块接口

## 1. 服务层接口

### 1.1 CellularDataService

**头文件**: `services/include/cellular_data_service.h`
**行号**: 32

```cpp
class CellularDataService : public SystemAbility, public CellularDataManagerStub {
    DECLARE_DELAYED_REF_SINGLETON(CellularDataService)
    DECLARE_SYSTEM_ABILITY(CellularDataService)
```

**主要方法**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| OnStart | void | void | 服务启动 |
| OnStop | void | void | 服务停止 |
| IsCellularDataEnabled | bool &dataEnabled | int32_t | 检查数据开关 |
| EnableCellularData | bool enable | int32_t | 开关数据 |
| GetCellularDataState | int32_t &state | int32_t | 获取连接状态 |
| IsCellularDataRoamingEnabled | int32_t slotId, bool &dataRoamingEnabled | int32_t | 检查漫游 |
| EnableCellularDataRoaming | int32_t slotId, bool enable | int32_t | 开关漫游 |
| GetDefaultCellularDataSlotId | int32_t &slotId | int32_t | 获取默认卡槽 |
| SetDefaultCellularDataSlotId | int32_t slotId | int32_t | 设置默认卡槽 |
| GetCellularDataFlowType | int32_t &type | int32_t | 获取流类型 |
| QueryApnIds | const ApnInfo&, vector\<uint32_t\> &apnIdList | int32_t | 查询 APN |
| SetPreferApn | int32_t apnId | int32_t | 设置首选 APN |
| HandleApnChanged | int32_t slotId | int32_t | APN 变更 |
| RequestNet | const NetRequest &request | int32_t | 请求网络 |
| ReleaseNet | const NetRequest &request | int32_t | 释放网络 |
| HasInternetCapability | int32_t slotId, int32_t cid, int32_t &capability | int32_t | 检查能力 |

### 1.2 CellularDataController

**头文件**: `services/include/cellular_data_controller.h`

```cpp
class CellularDataController : public TelEventHandler {
    std::shared_ptr<CellularDataNetAgent> netAgent_;
    std::shared_ptr<CellularDataStateMachine> stateMachine_;
    std::shared_ptr<ApnManager> apnManager_;
    int32_t slotId_;
```

### 1.3 State Machine

**头文件**: `services/include/state_machine/cellular_data_state_machine.h`

```cpp
class CellularDataStateMachine : public StateMachineBase {
    void ProcessEvent(int32_t event) override;
    void EnterState(StateId state) override;
    void ExitState(StateId state) override;
```

**状态定义**:

```cpp
enum StateId {
    STATE_ID_INVALID = -1,
    STATE_ID_INACTIVE = 0,
    STATE_ID_ACTIVATING = 1,
    STATE_ID_ACTIVE = 2,
    STATE_ID_DISCONNECTING = 3,
    STATE_ID_INCALL_DATA = 4
};
```

### 1.4 ApnManager

**头文件**: `services/include/apn_manager/apn_manager.h`

```cpp
class ApnManager {
    std::vector<ApnItem> apnList_;
    int32_t defaultApnId_ = -1;
```

## 2. 框架层接口

### 2.1 CellularDataClient（客户端）

**头文件**: `interfaces/innerkits/cellular_data_client.h`

```cpp
class CellularDataClient : public DelayedRefSingleton<CellularDataClient> {
    static std::atomic<int32_t> defaultCellularDataSlotId_;
    sptr<ICellularDataManager> GetProxy();
    bool IsConnect();
```

**主要方法**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| GetInstance | void | CellularDataClient& | 获取单例 |
| GetProxy | void | sptr\<ICellularDataManager\> | 获取服务代理 |
| IsConnect | void | bool | 检查连接状态 |
| IsCellularDataEnabled | bool &dataEnabled | int32_t | 检查数据开关 |
| EnableCellularData | bool enable | int32_t | 开关数据 |
| GetCellularDataState | void | int32_t | 获取状态 |
| IsCellularDataRoamingEnabled | int32_t slotId, bool &enabled | int32_t | 检查漫游 |
| EnableCellularDataRoaming | int32_t slotId, bool enable | int32_t | 开关漫游 |
| GetDefaultCellularDataSlotId | void | int32_t | 获取默认卡槽 |
| SetDefaultCellularDataSlotId | int32_t slotId | int32_t | 设置默认卡槽 |
| QueryApnIds | const ApnInfo&, vector\<uint32_t\> &apnIdList | int32_t | 查询 APN |

### 2.2 死亡回调

```cpp
class CellularDataDeathRecipient : public IRemoteObject::DeathRecipient {
    void OnRemoteDied(const wptr<IRemoteObject>& remote) override;
};
```

### 2.3 客户端初始化

**文件**: `frameworks/native/cellular_data_client.cpp`
**行号**: 430-443

```cpp
bool CellularDataClient::IsCellularDataSysAbilityExist(sptr<IRemoteObject> &object)
{
    sptr<ISystemAbilityManager> sm = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    object = sm->CheckSystemAbility(TELEPHONY_CELLULAR_DATA_SYS_ABILITY_ID);
    return object != nullptr;
}
```

## 3. 数据类型

### 3.1 ApnInfo

**头文件**: `interfaces/innerkits/cellular_data_types.h`

```cpp
struct ApnInfo : public Parcelable {
    std::u16string apnName;
    std::u16string apn;
    std::u16string mcc;
    std::u16string mnc;
    std::u16string user;
    std::u16string type;
    std::u16string proxy;
    std::u16string mmsproxy;

    bool Marshalling(Parcel &parcel) const;
    static ApnInfo* Unmarshalling(Parcel &parcel);
};
```

### 3.2 数据状态枚举

```cpp
enum class DataConnectionStatus : int32_t {
    DATA_STATE_DISCONNECTED = 11,
    DATA_STATE_CONNECTING = 12,
    DATA_STATE_CONNECTED = 13,
    DATA_STATE_SUSPENDED = 14
};

enum class DataConnectState : int32_t {
    DATA_STATE_UNKNOWN = -1,
    DATA_STATE_DISCONNECTED = 0,
    DATA_STATE_CONNECTING = 1,
    DATA_STATE_CONNECTED = 2,
    DATA_STATE_SUSPENDED = 3
};

enum class CellDataFlowType : int32_t {
    DATA_FLOW_TYPE_NONE = 0,
    DATA_FLOW_TYPE_DOWN = 1,
    DATA_FLOW_TYPE_UP = 2,
    DATA_FLOW_TYPE_UP_DOWN = 3,
    DATA_FLOW_TYPE_DORMANT = 4
};
```

### 3.3 网络请求

```cpp
struct NetRequest {
    int32_t uid;
    int32_t netId;
    std::vector<std::string> netCaps;
    std::vector<std::string> netSpecifiers;
};
```

## 4. IPC 接口代码

**头文件**: `interfaces/innerkits/cellular_data_ipc_interface_code.h`

| Code | Method | Description |
|------|--------|-------------|
| 0 | IS_CELLULAR_DATA_ENABLED | 检查数据开关 |
| 1 | ENABLE_CELLULAR_DATA | 开启数据 |
| 2 | DISABLE_CELLULAR_DATA | 关闭数据 |
| 3 | GET_CELLULAR_DATA_STATE | 获取状态 |
| 4 | IS_CELLULAR_DATA_ROAMING_ENABLED | 检查漫游 |
| 5 | ENABLE_CELLULAR_DATA_ROAMING | 开启漫游 |
| 6 | DISABLE_CELLULAR_DATA_ROAMING | 关闭漫游 |
| 7 | GET_DEFAULT_CELLULAR_DATA_SLOT_ID | 获取默认卡槽 |
| 8 | SET_DEFAULT_CELLULAR_DATA_SLOT_ID | 设置默认卡槽 |
| 9 | GET_CELLULAR_DATA_FLOW_TYPE | 获取流类型 |
| 10 | HAS_INTERNET_CAPABILITY | 检查网络能力 |
| 30 | QUERY_APN_IDS | 查询 APN ID |
| 31 | SET_PREFER_APN | 设置首选 APN |
| 32 | QUERY_ALL_APN_INFO | 查询所有 APN |
| 39 | GET_DEFAULT_CELLULAR_DATA_SIM_ID | 获取默认 SIM ID |

---

## 5. 依赖方向

```
                    ┌─────────────────┐
                    │   应用层 (JS)   │
                    └────────┬────────┘
                             │ N-API
                    ┌────────▼────────┐
                    │ CellularDataClient │
                    └────────┬────────┘
                             │ IPC (Binder)
                    ┌────────▼────────┐
                    │ CellularDataService │
                    │   (SA ID: 4007)  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼─────┐ ┌────▼─────┐ ┌─────▼─────┐
     │   NetManager  │ │ RIL Adapter │ │ CoreService │
     └──────────────┘ └───────────┘ └───────────┘
```

---

## 6. 稳定性标注

| 组件 | 路径 | 稳定性 | 说明 |
|------|------|--------|------|
| N-API 接口 | frameworks/js/napi/ | 稳定 | 公开 API |
| Inner API | interfaces/innerkits/ | 稳定 | 系统 API |
| 服务实现 | services/src/ | 稳定 | 内部实现 |
| 状态机 | services/src/state_machine/ | 稳定 | 内部状态机 |
| APN 管理 | services/src/apn_manager/ | 稳定 | 内部模块 |
