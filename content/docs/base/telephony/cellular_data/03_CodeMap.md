# 目录结构与代码地图

> **目的**：让新人学习者在 15 分钟内快速定位核心代码位置，理解目录组织方式
> **适用范围**：目录职责说明、核心文件定位、代码导航图
> **最后更新**：2026-02-07

---

## 1. 顶层目录结构

```
cellular_data/
├── frameworks/              # 框架层 - 对外 API 实现
│   ├── js/              # JavaScript N-API 绑定
│   ├── native/           # C++ Native API 实现
│   ├── cj/               # CJ (Cangjie) FFI 绑定
│   └── ets/              # ArkTS/ETS 实现
│       └── ani/         # ANI (Ark Native Interface)
│
├── interfaces/            # 接口层 - 接口定义与头文件
│   ├── innerkits/        # 内部接口（供其他系统服务使用）
│   └── kits/             # 对外接口（供应用使用）
│       ├── c/            # NDK C 接口
│       └── js/           # JavaScript/TypeScript 接口声明
│
├── services/             # 服务层 - System Ability 实现
│   ├── include/          # 服务头文件
│   │   ├── apn_manager/      # APN 管理器头文件
│   │   ├── state_machine/    # 状态机头文件
│   │   ├── utils/            # 工具类头文件
│   │   └── common/           # 通用定义头文件
│   │
│   ├── src/              # 服务实现
│   │   ├── apn_manager/      # APN 管理实现
│   │   ├── state_machine/    # 状态机实现
│   │   ├── utils/            # 工具类实现
│   │   ├── telephony_ext_wrapper/    # 电话扩展包装器
│   │   └── data_service_ext_wrapper/ # 数据服务扩展包装器
│   │
│   └── *.cpp            # 服务核心实现文件
│
├── sa_profile/           # System Ability 配置文件
│
├── test/                # 测试代码（忽略测试代码）
│   ├── unit_test/        # 单元测试
│   └── fuzztest/         # Fuzz 测试
│
├── figures/              # 架构图片
├── BUILD.gn             # GN 构建入口文件
├── bundle.json           # 组件配置清单
├── README.md            # 项目说明（英文）
├── README_zh.md        # 项目说明（中文）
├── OAT.xml             # OpenHarmony 兼容性配置
└── LICENSE              # Apache 2.0 许可证
```

---

## 2. 顶层目录职责

| 目录 | 职责 | 包含内容 |
|------|------|----------|
| **frameworks/** | 对外 API 框架层 | N-API 绑定、Native API、CJ FFI、ETS ANI 实现 |
| **interfaces/** | 接口定义层 | 内部 C++ 接口、NDK C 接口、TypeScript 接口声明、IPC 接口码 |
| **services/** | System Ability 服务实现 | CellularDataService 实现、状态机、APN 管理器、工具类 |
| **sa_profile/** | System Ability 配置 | SA 4007 配置文件（静态/动态） |
| **test/** | 测试代码 | 单元测试、Fuzz 测试、UI 测试 |
| **figures/** | 架构图片 | 系统架构图、数据流图 |

---

## 3. 核心文件定位

### 3.1 服务入口与初始化

| 功能 | 文件路径 | 关键类/函数 | 行号 |
|------|----------|-------------|------|
| **SA 主类定义** | `services/include/cellular_data_service.h:32` | `CellularDataService` | - |
| **SA 注册** | `services/src/cellular_data_service.cpp:37` | `MakeAndRegisterAbility` | - |
| **服务启动** | `services/src/cellular_data_service.cpp:45` | `OnStart()` | - |
| **服务初始化** | `services/src/cellular_data_service.cpp:119` | `Init()` | - |
| **SA 配置** | `sa_profile/4007.json:5` | SA ID: 4007 | - |

### 3.2 N-API 绑定入口

| 功能 | 文件路径 | 关键函数 | 行号 |
|------|----------|----------|------|
| **N-API 模块注册** | `frameworks/js/napi/src/napi_cellular_data.cpp:1515-1528` | `napi_module_register(&_cellularDataModule)` | - |
| **JS API 导出** | `frameworks/js/napi/src/napi_cellular_data.cpp:1481-1512` | `RegistCellularData()` | - |
| **N-API 头文件** | `frameworks/js/napi/include/napi_cellular_data.h` | - | - |
| **TypeScript 声明** | `interfaces/kits/js/@ohos.telephony.data.d.ts:30-494` | `data` 命名空间 | - |

### 3.3 IPC 接口定义

| 功能 | 文件路径 | 关键定义 | 行号 |
|------|----------|----------|------|
| **IPC IDL 定义** | `frameworks/native/ICellularDataManager.idl:20-61` | `ICellularDataManager` 接口 | - |
| **IPC 接口码** | `interfaces/innerkits/cellular_data_ipc_interface_code.h:22-62` | `CellularDataInterfaceCode` 枚举（31 个方法） | - |
| **IPC 客户端** | `interfaces/innerkits/cellular_data_client.h` | `CellularDataClient` 单例 | - |
| **IPC 客户实现** | `frameworks/native/cellular_data_client.cpp` | `CellularDataClient::GetInstance()` | - |

### 3.4 核心控制器

| 功能 | 文件路径 | 关键类/方法 | 行号 |
|------|----------|----------|------|
| **控制器定义** | `services/include/cellular_data_controller.h:24` | `CellularDataController` | - |
| **控制器实现** | `services/src/cellular_data_controller.cpp` | `Init()`, `SetCellularDataEnable()` | - |
| **事件处理器定义** | `services/include/cellular_data_handler.h:38` | `CellularDataHandler` | - |
| **事件处理实现** | `services/src/cellular_data_handler.cpp` | `ProcessEvent()` | - |

### 3.5 状态机实现

| 功能 | 文件路径 | 关键类 | 行号 |
|------|----------|----------|------|
| **状态机主类** | `services/include/state_machine/cellular_data_state_machine.h:37-38` | `CellularDataStateMachine` | - |
| **状态机实现** | `services/src/state_machine/cellular_data_state_machine.cpp` | `Init()`, `UpdateNetworkInfo()` | - |
| **状态类** | `services/src/state_machine/` | - | - |
|  | `inactive.cpp` | `Inactive` 状态类 | - |
|  | `activating.cpp` | `Activating` 状态类 | - |
|  | `active.cpp` | `Active` 状态类 | - |
|  | `disconnecting.cpp` | `Disconnecting` 状态类 | - |
|  | `default.cpp` | `Default` 状态类（父状态） | - |
| **通话中状态机** | `services/include/state_machine/incall_data_state_machine.h` | `IncallDataStateMachine` | - |
|  | `services/src/state_machine/incall_data_state_machine.cpp` | `IncallDataStateMachine` 实现 | - |

### 3.6 APN 管理

| 功能 | 文件路径 | 关键类/方法 | 行号 |
|------|----------|----------|------|
| **APN 管理器** | `services/include/apn_manager/apn_manager.h` | `ApnManager` | - |
| **APN 管理实现** | `services/src/apn_manager/apn_manager.cpp` | `GetApnHolder()`, `CreateAllApnItemByDatabase()` | - |
| **APN Holder** | `services/include/apn_manager/apn_holder.h` | `ApnHolder` | - |
| **APN Item** | `services/include/apn_manager/apn_item.h` | `ApnItem` | - |
| **APN 属性** | `interfaces/innerkits/apn_attribute.h` | `ApnAttribute` 结构 | - |

### 3.7 工具类

| 功能 | 文件路径 | 关键类/方法 | 行号 |
|------|----------|----------|------|
| **网络代理** | `services/include/utils/cellular_data_net_agent.h` | `CellularDataNetAgent` | - |
| **数据库助手** | `services/include/utils/cellular_data_rdb_helper.h` | `CellularDataRdbHelper` | - |
| **设置助手** | `services/include/utils/cellular_data_settings_rdb_helper.h` | `CellularDataSettingsRdbHelper` | - |
| **系统事件** | `services/include/utils/cellular_data_hisysevent.h` | `CellularDataHiSysEvent` | - |
| **通用工具** | `services/include/utils/cellular_data_utils.h` | `CellularDataUtils` | - |

### 3.8 数据类型定义

| 功能 | 文件路径 | 关键定义 | 行号 |
|------|----------|----------|------|
| **通用常量** | `services/include/common/cellular_data_constant.h:23-189` | `ApnProfileState`, `DisConnectionReason`, `ApnTypes`, `DataContextRolesId` 等 | - |
| **数据类型** | `interfaces/innerkits/cellular_data_types.h` | `ApnInfo`, `ApnActivateReportInfo` | - |
| **错误码** | `interfaces/innerkits/cellular_data_error.h` | 错误码定义 | - |

---

## 4. 代码导航图

### 4.1 按"功能"定位代码

#### 数据连接状态查询
```
应用层（JS）
  ↓
interfaces/kits/js/@ohos.telephony.data.d.ts:139-155 (getCellularDataState)
  ↓
frameworks/js/napi/src/napi_cellular_data.cpp:XXX (NativeGetCellularDataState)
  ↓
interfaces/innerkits/cellular_data_client.h:XXX (GetCellularDataState)
  ↓ (IPC Binder)
services/src/cellular_data_service.cpp:XXX (GetCellularDataState)
  ↓ (权限检查)
services/include/cellular_data_controller.h:XXX (GetCellularDataState)
  ↓
services/src/cellular_data_handler.cpp:XXX (GetCellularDataState)
  ↓
services/include/state_machine/cellular_data_state_machine.h:XXX (状态查询)
```

#### 数据开关控制
```
应用层（JS）
  ↓
interfaces/kits/js/@ohos.telephony.data.d.ts:XXX (enableCellularData)
  ↓
frameworks/js/napi/src/napi_cellular_data.cpp:XXX (NativeEnableCellularData)
  ↓ (权限检查: SET_TELEPHONY_STATE)
services/src/cellular_data_service.cpp:XXX (EnableCellularData)
  ↓
services/include/cellular_data_controller.h:XXX (SetCellularDataEnable)
  ↓
services/src/cellular_data_handler.cpp:XXX (SetCellularDataEnable)
  ↓
[事件分发] → 状态机触发连接/断开
```

#### APN 查询
```
应用层（JS）
  ↓
interfaces/kits/js/@ohos.telephony.data.d.ts:XXX (queryApnIds)
  ↓
frameworks/js/napi/src/napi_cellular_data.cpp:XXX (NativeQueryApnIds)
  ↓ (权限检查: MANAGE_APN_SETTING)
services/src/cellular_data_service.cpp:XXX (QueryApnIds)
  ↓
services/include/apn_manager/apn_manager.h:XXX (QueryApnIds)
  ↓
services/src/apn_manager/apn_manager.cpp:XXX (QueryApnIds)
  ↓
[数据库查询] data_share://.../pdp_profile
```

### 4.2 按"组件"定位代码

#### System Ability 服务
```
主入口:
  services/include/cellular_data_service.h:32 (CellularDataService 类定义)
  ↓
  services/src/cellular_data_service.cpp:37 (SA 注册)
  ↓
  services/src/cellular_data_service.cpp:45 (OnStart 服务启动)
  ↓
  services/src/cellular_data_service.cpp:119 (Init 初始化)
  ↓
  services/src/cellular_data_service.cpp:305 (InitModule 模块初始化)
```

#### 状态机
```
基类:
  services/include/state_machine/state_machine.h (StateMachine 基类)
  ↓
具体实现:
  services/include/state_machine/cellular_data_state_machine.h:37 (CellularDataStateMachine)
  ↓
  services/src/state_machine/cellular_data_state_machine.cpp (状态机实现)
  ↓
状态类:
  services/include/state_machine/active.h (Active 状态)
  services/include/state_machine/inactive.h (Inactive 状态)
  services/include/state_machine/activating.h (Activating 状态)
  services/include/state_machine/disconnecting.h (Disconnecting 状态)
  ↓
  services/src/state_machine/active.cpp (Active 实现)
  services/src/state_machine/inactive.cpp (Inactive 实现)
  services/src/state_machine/activating.cpp (Activating 实现)
  services/src/state_machine/disconnecting.cpp (Disconnecting 实现)
```

#### N-API 绑定
```
JS 接口声明:
  interfaces/kits/js/@ohos.telephony.data.d.ts (TypeScript 类型定义)
  ↓
N-API 头文件:
  frameworks/js/napi/include/napi_cellular_data.h (C++ 函数声明)
  ↓
N-API 实现:
  frameworks/js/napi/src/napi_cellular_data.cpp (C++ 实现和注册)
  ↓
注册入口:
  frameworks/js/napi/src/napi_cellular_data.cpp:1515-1528 (napi_module_register)
```

---

## 5. 数据流图（简化）

### 5.1 数据连接请求流程

```
[应用层] data.enableCellularData()
  ↓ (N-API)
[NAPI 层] NativeEnableCellularData() → AsyncWork
  ↓ (权限验证: SET_TELEPHONY_STATE)
[客户端层] CellularDataClient::EnableCellularData()
  ↓ (IPC Binder)
[服务层] CellularDataService::EnableCellularData()
  ↓ (权限检查: SET_TELEPHONY_STATE + system app)
[控制器层] CellularDataController::SetCellularDataEnable()
  ↓ [事件发送 MSG_SM_CONNECT]
[状态机层] CellularDataStateMachine::Init() → Inactive → Activating
  ↓ [RIL 请求: RADIO_RIL_SETUP_DATA_CALL]
[RIL 层] SetupDataCallResultInfo
  ↓ [事件: RADIO_RIL_SETUP_DATA_CALL]
[状态机层] Activating → Active (连接成功)
  ↓ [网络注册: RequestNet]
[NetManager 层] NetConnClient::RegisterNetworkSupplier()
  ↓ [网络可用: OnNetworkAvailable]
[状态机层] UpdateNetworkInfo() → 设置 IP/带宽/路由
```

### 5.2 数据状态查询流程

```
[应用层] data.getCellularDataState()
  ↓ (N-API)
[NAPI 层] NativeGetCellularDataState() → AsyncWork
  ↓ (无权限要求)
[客户端层] CellularDataClient::GetCellularDataState()
  ↓ (IPC Binder)
[服务层] CellularDataService::GetCellularDataState()
  ↓ (权限检查: GET_NETWORK_INFO)
[控制器层] CellularDataController::GetCellularDataState()
  ↓
[状态机层] CellularDataStateMachine::GetState() → 返回当前状态
  ↓
[NAPI 层] WrapCellularDataType() → DataConnectState 枚举
  ↓
[应用层] 返回: DATA_STATE_DISCONNECTED/CONNECTING/CONNECTED/SUSPENDED
```

### 5.3 APN 查询流程

```
[应用层] data.queryApnIds(apnInfo)
  ↓ (N-API)
[NAPI 层] NativeQueryApnIds() → AsyncWork
  ↓ (权限验证: MANAGE_APN_SETTING)
[客户端层] CellularDataClient::QueryApnIds()
  ↓ (IPC Binder)
[服务层] CellularDataService::QueryApnIds()
  ↓ (权限检查: MANAGE_APN_SETTING)
[APN 管理层] ApnManager::QueryApnIds()
  ↓
[数据库层] DataShare::Query() → APN 数据表查询
  ↓
[APN 管理层] FilterMatchedApns() → 筛选匹配的 APN
  ↓
[NAPI 层] ApnInfoConversion() → 转换为 N-API 对象
  ↓
[应用层] 返回: List<ApnInfo>
```

---

## 6. 快速定位索引

### 6.1 我要：添加新的 JS API

1. 在 `interfaces/kits/js/@ohos.telephony.data.d.ts` 添加 TypeScript 声明
2. 在 `frameworks/js/napi/src/napi_cellular_data.cpp` 实现 C++ 函数
3. 在 `RegistCellularData()` 函数中添加 `napi_property_descriptor` 条目
4. 添加权限常量定义（如需要）到文件顶部
5. 在 `interfaces/innerkits/cellular_data_ipc_interface_code.h` 添加 IPC 接口码（如需要服务端支持）

### 6.2 我要：修改数据连接逻辑

1. 定位状态：`services/src/state_machine/`（`active.cpp`、`activating.cpp` 等）
2. 修改状态转换逻辑（`StateProcess()` 方法）
3. 添加/修改事件处理（`ProcessEvent()` 方法）
4. 检查权限要求（`services/src/cellular_data_service.cpp`）
5. 测试状态转换流程

### 6.3 我要：添加新的 IPC 方法

1. 在 `frameworks/native/ICellularDataManager.idl` 添加 IDL 方法声明
2. 在 `interfaces/innerkits/cellular_data_ipc_interface_code.h` 添加接口码枚举值
3. 在 `services/include/cellular_data_service.h` 添加方法声明
4. 在 `services/src/cellular_data_service.cpp` 实现方法（包括权限检查）
5. 在 `services/include/cellular_data_controller.h` 添加控制器方法
6. 在 `services/src/cellular_data_controller.cpp` 实现控制器逻辑
7. 在 `interfaces/innerkits/cellular_data_client.h` 添加客户端方法声明
8. 在 `frameworks/native/cellular_data_client.cpp` 实现客户端调用

### 6.4 我要：修改 APN 配置

1. 查看当前 APN 配置：`services/src/utils/cellular_data_rdb_helper.cpp`
2. 查看数据库表结构：`CELLULAR_DATA_RDB_URI` 常量
3. 修改 APN 查询/更新逻辑：`services/src/apn_manager/apn_manager.cpp`
4. 测试 APN 管理流程（创建/查询/修改/删除）

### 6.5 我要：修改构建配置

1. 查看 GN targets：`BUILD.gn` 根文件
2. 查看框架层构建：`frameworks/native/BUILD.gn`、`frameworks/js/BUILD.gn` 等
3. 添加新源文件到 `sources` 列表
4. 添加新的外部依赖到 `external_deps`
5. 修改组件配置：`bundle.json`
6. 重新编译验证

---

## 7. 忽略的目录

根据规范，以下目录包含测试代码，在文档中忽略：

```
test/
├── unit_test/        # 单元测试
├── fuzztest/         # Fuzz 测试
└── cellular_data_test/  # UI 集成测试
```

如需了解测试内容，可参考：
- `test/unit_test/` - 单元测试实现
- `test/fuzztest/` - Fuzz 测试用例

---

## 8. 证据索引

| 类别 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| **SA 注册** | `services/src/cellular_data_service.cpp:37` | `MakeAndRegisterAbility` 调用 |
| **SA ID** | `sa_profile/4007.json:5` | SA ID: 4007 |
| **N-API 注册** | `frameworks/js/napi/src/napi_cellular_data.cpp:1515-1528` | `napi_module_register` 调用 |
| **JS 导出** | `frameworks/js/napi/src/napi_cellular_data.cpp:1481-1512` | 20 个 JS API 注册 |
| **IPC IDL** | `frameworks/native/ICellularDataManager.idl:20-61` | 40 个 IPC 方法定义 |
| **IPC 接口码** | `interfaces/innerkits/cellular_data_ipc_interface_code.h:22-62` | `CellularDataInterfaceCode` 枚举 |
| **状态机类** | `services/include/state_machine/cellular_data_state_machine.h:37-38` | `CellularDataStateMachine` 类定义 |
| **APN 管理器** | `services/include/apn_manager/apn_manager.h` | `ApnManager` 类定义 |
| **主构建文件** | `BUILD.gn:36-159` | `tel_cellular_data` target 定义 |
| **组件配置** | `bundle.json:1-102` | 组件元信息 |

---

**文档完成** - 目录结构和代码导航图清晰，支持新人快速定位代码。
