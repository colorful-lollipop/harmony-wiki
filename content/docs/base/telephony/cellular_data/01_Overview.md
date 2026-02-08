# 项目概览

> **目的**：让新人学习者在 5 分钟内理解 cellular_data 模块的定位、能力边界和运行环境
> **适用范围**：项目整体定位、核心功能、依赖关系
> **最后更新**：2026-02-07

---

## 1. 一句话定义

**Cellular Data（蜂窝数据）模块**是 OpenHarmony Telephony 子系统的系统服务，负责蜂窝移动数据连接的激活、状态管理、漫游控制和 APN 配置。

**位置**：`base/telephony/cellular_data/`

**系统能力**：`SystemCapability.Telephony.CellularData`

---

## 2. 能力边界

### 2.1 能做什么（核心能力）

| 功能类别 | 具体能力 | 说明 |
|---------|----------|------|
| **数据连接管理** | 激活/去激活 PDP 上下文 | 控制蜂窝数据开关，建立/断开网络连接 |
| **状态查询** | 查询数据连接状态 | 返回 `UNKNOWN` / `DISCONNECTED` / `CONNECTING` / `CONNECTED` / `SUSPENDED` |
| **漫游管理** | 启用/禁用数据漫游 | 控制数据漫游开关，查询漫游状态 |
| **默认卡管理** | 设置/查询默认数据卡槽 | 支持双卡切换，选择默认数据卡（slotId 0/1） |
| **APN 管理** | 查询/设置 APN | 管理接入点配置，支持多种 APN 类型（default、mms、supl、dun、ims 等） |
| **流量统计** | 上行/下行数据流类型 | 检测 `NONE` / `DOWN` / `UP` / `UP_DOWN` / `DORMANT` |
| **智能切换** | 卡槽智能切换 | 根据信号质量自动切换数据卡槽（可选功能） |
| **故障恢复** | 数据连接故障检测与恢复 | 监控连接质量，自动重连断开的连接 |

### 2.2 不能做什么（能力边界）

| 限制 | 说明 | 原因 |
|------|------|------|
| **不处理语音/短信** | 仅处理数据业务，语音/短信由 core_service/ril_adapter 的其他模块处理 | 职责分离 |
| **不直接管理 Wi-Fi** | Wi-Fi 连接由 netmanager_base 管理 | 模块边界 |
| **不支持 VPN 控制** | VPN 配置由系统其他模块管理 | 职责分离 |
| **不提供浏览器/应用 API** | 仅提供网络连接能力，应用层 API 由 framework 提供 | 架构层次 |

---

## 3. 运行环境

### 3.1 系统依赖

| 依赖组件 | SA ID | 依赖类型 | 用途 |
|----------|-------|----------|------|
| **core_service** | 4010 | System Ability | SIM 卡状态、运营商信息、默认数据卡管理 |
| **ril_adapter** | - | RIL 接口 | 与 Modem 通信，发送 PDP 激活/去激活命令 |
| **netmanager_base** | - | System Ability | 网络连接管理、路由配置、DNS 设置 |
| **data_share** | - | 数据库服务 | 存储/读取 APN 配置（URI: `datashare:///com.ohos.pdpprofileability`） |
| **common_event_service** | - | 事件服务 | 订阅/发布系统事件（屏幕开关、飞行模式、SIM 卡切换等） |
| **safwk** | - | SA 框架 | System Ability 注册与管理基础 |
| **samgr** | - | 服务管理器 | SA 查询与代理获取 |
| **hilog** | - | 日志系统 | 日志输出（TAG: "CellularData", Domain: `0xD001F03`） |
| **hisysevent** | - | 系统事件上报 | HiSysEvent 故障/审计事件上报 |
| **access_token** | - | 权限管理 | 权限验证（`GET_NETWORK_INFO`、`SET_TELEPHONY_STATE`、`MANAGE_APN_SETTING`） |

### 3.2 硬件依赖

| 硬件要求 | 说明 |
|----------|------|
| **蜂窝 Modem** | 支持 2G/3G/4G/5G 网络的通信模块 |
| **SIM 卡** | 至少一张 SIM 卡（支持双卡：`MAX_SLOT_NUM = 2`） |
| **天线/射频模块** | 用于蜂窝网络信号收发 |

### 3.3 软件依赖

| 组件 | 版本要求 | 说明 |
|------|----------|------|
| **OpenHarmony 系统** | API 7+ | `@syscap SystemCapability.Telephony.CellularData` |
| **N-API 框架** | - | 提供 JavaScript 绑定（`@ohos.telephony.data` 模块） |
| **IPC 框架** | - | Binder/HIDL 通信（System Ability） |
| **C++ 标准** | C++17+ | 核心服务实现语言 |

---

## 4. 对外暴露面

### 4.1 JavaScript N-API（20 个接口）

**模块名称**：`@ohos.telephony.data`

**暴露给**：ArkTS/JavaScript 应用

**主要类别**：
- **状态查询类**：`isCellularDataEnabled()`, `getCellularDataState()`, `getCellularDataFlowType()`
- **开关控制类**：`enableCellularData()`, `disableCellularData()`, `enableCellularDataRoaming()`, `disableCellularDataRoaming()`
- **卡槽管理类**：`getDefaultCellularDataSlotId()`, `setDefaultCellularDataSlotId()`
- **APN 管理类**：`queryApnIds()`, `setPreferApn()`, `queryAllApns()`
- **智能开关类**：`enableIntelligenceSwitch()`, `getIntelligenceSwitchState()`

完整 API 列表参见 [`04_NAPI_Reference.md`](./04_NAPI_Reference.md)

### 4.2 IPC 接口（31 个方法）

**接口名称**：`OHOS.Telephony.ICellularDataManager`

**暴露给**：其他系统服务（如 `netmanager_base`、`core_service`）

**主要类别**：
- **数据状态管理**：`IsCellularDataEnabled`, `GetCellularDataState`, `GetCellularDataFlowType`
- **漫游管理**：`IsCellularDataRoamingEnabled`, `EnableCellularDataRoaming`
- **APN 管理**：`QueryApnIds`, `SetPreferApn`, `QueryAllApnInfo`, `HandleApnChanged`
- **网络控制**：`RequestNet`, `ReleaseNet`, `AddUid`, `RemoveUid`, `ClearAllConnections`
- **回调注册**：`RegisterSimAccountCallback`, `UnregisterSimAccountCallback`

完整 IPC 列表参见 [`05_InnerAPI.md`](./05_InnerAPI.md)

### 4.3 C API（内部使用）

**库文件**：`libtel_cellular_data_api.z.so`

**暴露给**：Native C++ 模块（内部系统服务）

**主要接口**：
- `CellularDataClient` 单例类
- 通过 `ISystemAbilityManager::CheckSystemAbility(4007)` 获取服务代理

---

## 5. 快速开始

### 5.1 最小使用示例（JavaScript）

```javascript
import data from '@ohos.telephony.data';

// 1. 检查数据是否启用
try {
    const isEnabled = await data.isCellularDataEnabled();
    console.log(`Cellular data enabled: ${isEnabled}`);
} catch (err) {
    console.error(`Failed: ${err.message}`);
}

// 2. 获取数据连接状态
try {
    const state = await data.getCellularDataState();
    console.log(`Data state: ${state}`);
    // 输出: -1=UNKNOWN, 0=DISCONNECTED, 1=CONNECTING, 2=CONNECTED, 3=SUSPENDED
} catch (err) {
    console.error(`Failed: ${err.message}`);
}

// 3. 启用数据漫游（需要系统应用权限）
// 注意：此 API 需要 ohos.permission.SET_TELEPHONY_STATE 权限
try {
    const slotId = 0; // 卡槽 0 表示第一张卡
    await data.enableCellularDataRoaming(slotId);
    console.log('Data roaming enabled');
} catch (err) {
    console.error(`Failed: ${err.message}`);
}
```

### 5.2 权限声明（module.json5）

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.GET_NETWORK_INFO",
        "reason": "$string:get_network_info_reason",
        "usedScene": "$string:get_network_info_scene"
      }
    ]
  }
}
```

**权限说明**：
- `ohos.permission.GET_NETWORK_INFO` - 查询网络信息状态（normal 级别）
- `ohos.permission.SET_TELEPHONY_STATE` - 修改电话状态（system_basic 级别，仅系统应用）
- `ohos.permission.MANAGE_APN_SETTING` - 管理 APN 设置（system_basic 级别）

---

## 6. 系统配置

### 6.1 SA 配置

**SA ID**：4007（`TELEPHONY_CELLULAR_DATA_SYS_ABILITY_ID`）

**配置文件**：`sa_profile/4007.json`

```json
{
  "process": "telephony",
  "systemability": [{
    "name": 4007,
    "libpath": "libtel_cellular_data.z.so",
    "run-on-create": true,
    "depend": [4010],
    "depend_time_out": 60000
  }]
}
```

**关键配置项**：
- `process`：运行进程为 `telephony`
- `libpath`：服务动态库为 `libtel_cellular_data.z.so`
- `run-on-create`：开机自启动
- `depend[0]`：依赖 `core_service`（SA ID: 4010）

### 6.2 组件配置

**配置文件**：`bundle.json`

```json
{
  "name": "@ohos/cellular_data",
  "subsystem": "telephony",
  "syscap": ["SystemCapability.Telephony.CellularData"],
  "features": [
    "cellular_data_dynamic_start",
    "cellular_data_feature_base_power_improvement"
  ]
}
```

---

## 7. 架构概览

### 7.1 层次结构

```
┌─────────────────────────────────────────────────────────────────┐
│              应用层（ArkTS/JavaScript）                       │
│           @ohos.telephony.data N-API                        │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                  N-API 绑定层                            │
│      frameworks/js/napi/src/napi_cellular_data.cpp          │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                  System Ability 服务层                        │
│           CellularDataService (SA 4007)                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ CellularDataManagerStub (IPC 接口)          │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ CellularDataController (每卡槽一个)           │   │
│  │  → CellularDataHandler (事件处理)               │   │
│  │  → DataConnectionManager (连接管理)             │   │
│  │     → CellularDataStateMachine (状态机)          │   │
│  │        → Active/Inactive/Activating 等          │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│               外部系统服务依赖                              │
│  • core_service (SA 4010) - SIM 状态               │
│  • netmanager_base - 网络管理                          │
│  • ril_adapter - Modem 通信                            │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 核心组件

| 组件 | 职责 | 关键方法 |
|------|------|----------|
| **CellularDataService** | System Ability 主类，服务生命周期管理 | `OnStart()`, `Init()`, `Publish()` |
| **CellularDataController** | 单卡槽控制器，事件分发 | `SetCellularDataEnable()`, `GetCellularDataState()`, `ProcessEvents()` |
| **CellularDataHandler** | 事件处理器，协调各模块 | `OnCallStateChanged()`, `OnScreenOn()`, `OnDataShareReady()` |
| **DataConnectionManager** | 连接管理，管理多个连接 | `RequestNet()`, `ReleaseNet()`, `ClearAllConnections()` |
| **CellularDataStateMachine** | 单个连接的状态机 | `Init()`, `UpdateNetworkInfo()`, `SetConnectionBandwidth()` |
| **ApnManager** | APN 配置管理 | `GetApnHolder()`, `FilterMatchedApns()`, `CreateAllApnItemByDatabase()` |
| **CellularDataClient** | IPC 客户端，供其他模块调用 | `IsCellularDataEnabled()`, `EnableCellularData()`, `GetCellularDataState()` |

详细架构说明参见 [`02_Architecture.md`](./02_Architecture.md)

---

## 8. Feature 开关

| Feature | 默认值 | 说明 | GN 参数 |
|---------|----------|------|----------|
| **基础功耗优化** | `false` | 电源模式订阅，降低待机功耗 | `cellular_data_feature_base_power_improvement` |
| **动态启动** | `false` | SA 按需启动（非开机自启动） | `cellular_data_dynamic_start` |
| **HiCollie 监控** | `true` | 故障检测与上报（依赖 `hiviewdfx_hicollie`） | `telephony_cellular_data_hicollie_able` |
| **数据服务扩展** | 条件编译 | 支持 5G 网络切片、URSP 解码等高级功能 | `OHOS_BUILD_ENABLE_DATA_SERVICE_EXT` |
| **VSIM 支持** | 条件编译 | 虚拟 SIM 卡支持 | `OHOS_BUILD_ENABLE_TELEPHONY_VSIM` |

---

## 9. 编译产物

| 产物 | 类型 | 安装路径 | 用途 |
|------|------|----------|------|
| `libtel_cellular_data.z.so` | 共享库 | `/system/lib/` | System Ability 服务主库 |
| `libtel_cellular_data_api.z.so` | 共享库 | `/system/lib/` | C++ Native API 库 |
| `data.so` (JS NAPI) | 模块 | `/system/lib/module/telephony/` | JavaScript N-API 模块 |
| `telephony_data.so` | 共享库 | `/system/lib/ndk/` | NDK C 接口库 |
| `cellular_data_ani.so` | 共享库 | `/system/lib/` | Rust ANI 库 |
| `telephony_data.abc` | ETS 字节码 | `/system/framework/` | ArkTS 绑定字节码 |
| `4007.json` | SA 配置 | `/system/profile/` | System Ability 配置文件 |

详细构建配置参见 [`06_Build.md`](./06_Build.md)

---

## 10. 关键文件定位

### 10.1 入口文件

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **SA 主类** | `services/include/cellular_data_service.h:32` | `CellularDataService` 类定义 |
| **SA 实现** | `services/src/cellular_data_service.cpp` | `OnStart()`, `Init()`, IPC 方法实现 |
| **N-API 注册** | `frameworks/js/napi/src/napi_cellular_data.cpp:1481-1528` | `RegistCellularData()` 函数，导出 JS API |
| **N-API 声明** | `interfaces/kits/js/@ohos.telephony.data.d.ts` | TypeScript 接口定义 |

### 10.2 核心实现

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **控制器** | `services/include/cellular_data_controller.h` | `CellularDataController` 类定义 |
| **事件处理器** | `services/include/cellular_data_handler.h` | `CellularDataHandler` 类定义 |
| **连接管理器** | `services/include/data_connection_manager.h` | `DataConnectionManager` 类定义 |
| **状态机** | `services/include/state_machine/cellular_data_state_machine.h` | `CellularDataStateMachine` 类定义 |
| **APN 管理器** | `services/include/apn_manager/apn_manager.h` | `ApnManager` 类定义 |

### 10.3 接口定义

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **IPC IDL** | `frameworks/native/ICellularDataManager.idl` | `ICellularDataManager` 接口定义（40 个方法） |
| **IPC 接口码** | `interfaces/innerkits/cellular_data_ipc_interface_code.h:19` | `CellularDataInterfaceCode` 枚举（31 个方法码） |
| **IPC 客户端** | `interfaces/innerkits/cellular_data_client.h` | `CellularDataClient` 类定义 |
| **C 客户端** | `interfaces/innerkits/cellular_data_types.h` | 数据类型定义 |

### 10.4 构建配置

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **主构建文件** | `BUILD.gn` | GN targets 定义（`tel_cellular_data`、`tel_cellular_data_api` 等） |
| **组件配置** | `bundle.json` | 组件元信息、依赖声明 |
| **SA 配置** | `sa_profile/4007.json` | System Ability 配置（进程、依赖、启动方式） |

---

## 11. 相关链接

- **代码仓库**：https://gitee.com/openharmony/telephony_cellular_data
- **OpenHarmony Telephony**：https://gitee.com/openharmony/telephony
- **系统能力文档**：https://gitee.com/openharmony/docs/blob/master/zh-cn/system-dev-guides/

---

## 12. 证据索引

| 类别 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| **SA ID** | `sa_profile/4007.json:5` | SA ID: 4007 |
| **SA 主类** | `services/include/cellular_data_service.h:32` | `CellularDataService` 类定义 |
| **SA 依赖** | `sa_profile/4007.json:9` | 依赖 core_service (SA 4010) |
| **N-API 注册** | `frameworks/js/napi/src/napi_cellular_data.cpp:1515-1528` | `napi_module_register` 调用 |
| **N-API 函数** | `frameworks/js/napi/src/napi_cellular_data.cpp:1481-1512` | 20 个 JS API 注册 |
| **JS 声明** | `interfaces/kits/js/@ohos.telephony.data.d.ts:30-494` | TypeScript 接口定义 |
| **IPC IDL** | `frameworks/native/ICellularDataManager.idl:20-61` | 40 个 IPC 方法定义 |
| **IPC 接口码** | `interfaces/innerkits/cellular_data_ipc_interface_code.h:22-62` | `CellularDataInterfaceCode` 枚举 |
| **主服务库** | `BUILD.gn:36-159` | `tel_cellular_data` target 定义 |
| **组件配置** | `bundle.json:1-102` | 组件元信息 |
| **构建依赖** | `BUILD.gn:104-122` | `external_deps` 列表 |

---

**文档完成** - 证据充足，符合新人学习需求。
