# 内部模块接口

## 1. 服务层接口

### 1.1 CellularDataService

**头文件**: `services/include/cellular_data_service.h`

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
| 11 | CLEAR_CELLULAR_DATA_CONNECTIONS | 清除连接 |
| 12 | GET_APN_STATE | 获取 APN 状态 |
| 13 | GET_DATA_RECOVERY_STATE | 获取恢复状态 |
| 14 | REGISTER_SIM_ACCOUNT_CALLBACK | 注册回调 |
| 15 | UNREGISTER_SIM_ACCOUNT_CALLBACK | 注销回调 |
| 16 | GET_DATA_CONN_APN_ATTR | 获取 APN 属性 |
| 17 | GET_DATA_CONN_IP_TYPE | 获取 IP 类型 |
| 18 | IS_NEED_DO_RECOVERY | 检查是否需要恢复 |
| 19 | ENABLE_INTELLIGENCE_SWITCH | 智能开关 |
| 20 | INIT_CELLULAR_DATA_CONTROLLER | 初始化控制器 |
| 21 | GET_INTELLIGENCE_SWITCH_STATE | 获取智能开关状态 |
| 22 | ESTABLISH_ALL_APNS_IF_CONNECTABLE | 建立所有 APN |
| 23 | RELEASE_CELLULAR_DATA_CONNECTION | 释放连接 |
| 24 | GET_CELLULAR_DATA_SUPPLIER_ID | 获取供应商 ID |
| 25 | CORRECT_NET_SUPPLIER_NO_AVAILABLE | 纠正供应商 |
| 26 | GET_SUPPLIER_REGISTER_STATE | 获取注册状态 |
| 27 | GET_IF_SUPPORT_DUN_APN | 检查 DUN APN |
| 28 | GET_DEFAULT_ACT_REPORT_INFO | 获取激活报告 |
| 29 | GET_INTERNAL_ACT_REPORT_INFO | 获取内部报告 |
| 30 | QUERY_APN_IDS | 查询 APN ID |
| 31 | SET_PREFER_APN | 设置首选 APN |
| 32 | QUERY_ALL_APN_INFO | 查询所有 APN |
| 33 | SEND_URSP_DECODE_RESULT | 发送 URSP 解码 |
| 34 | SEND_UE_POLICY_SECTION_IDENTIFIER | 发送策略标识 |
| 35 | SEND_IMS_RSD_LIST | 发送 IMS RSD |
| 36 | GET_NETWORK_SLICE_ALLOWED_NSSAI | 获取切片 NSSAI |
| 37 | GET_NETWORK_SLICE_EHPLMN | 获取切片 EHPLMN |
| 38 | GET_ACTIVE_APN_NAME | 获取激活 APN |
| 39 | GET_DEFAULT_CELLULAR_DATA_SIM_ID | 获取默认 SIM ID |
| 40 | CLEAR_ALL_CONNECTIONS | 清除所有连接 |
| 41 | CHANGE_CONNECTION_FOR_DSD | DSD 连接变更 |
| 42 | STRATEGY_SWITCH | 策略开关 |
