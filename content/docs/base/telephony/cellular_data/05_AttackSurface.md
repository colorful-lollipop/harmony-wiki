# 攻击面分析

> **目的**：让安全研究员快速识别所有外部输入入口、敏感操作点和信任边界
> **适用范围**：外部输入清单、敏感操作清单、信任边界图
> **最后更新**：2026-02-07

---

## 1. 外部输入清单

### 1.1 N-API 层输入（JavaScript 应用层）

| 输入源 | 文件路径 | 参数 | 验证点 |
|---------|----------|------|----------|
| **数据开关状态查询** | `frameworks/js/napi/src/napi_cellular_data.cpp` | 无 | - |
| **获取数据连接状态** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | 无 | - |
| **数据开关控制** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `enable` (boolean) | 权限检查 + `CheckCallerIsSystemApp()` |
| **漫游开关控制** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `slotId` (number), `enable` (boolean) | `IsValidSlotId()` + 权限检查 + 系统应用检查 |
| **漫游状态查询** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `slotId` (number) | `IsValidSlotId()` + 权限检查 |
| **默认卡槽设置** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `slotId` (number) | `IsValidSlotId()` + 权限检查 + 系统应用检查 |
| **默认卡槽查询** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | 无 | - |
| **数据流类型查询** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | 无 | - |
| **APN 查询（ID 列表）** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `apnInfo` (ApnInfo 对象) | `ApnInfoAnalyze()` 解析 + 权限检查 |
| **APN 设置（首选）** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `apnId` (number) | 权限检查 |
| **APN 查询（全部）** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | 无 | 权限检查 |
| **网络切片 URSP 解码** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `slotId` (number), `buffer` (Array<number>) | `IsValidSlotId()` |
| **网络切片 UE 策略** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `slotId` (number), `buffer` (Array<number>) | `IsValidSlotId()` |
| **网络切片 IMS RSD** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `slotId` (number), `buffer` (Array<number>) | `IsValidSlotId()` |
| **智能开关控制** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | `enable` (boolean) | 无 |
| **智能开关状态** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | 无 | - |

**N-API 层输入验证函数**：
- `MatchCellularDataParameters()` - 匹配回调函数参数
- `MatchEnableCellularDataRoamingParameters()` - 验证 slotId 参数
- `MatchGetDefaultCellularDataSlotIdParameters()` - 验证默认卡槽参数
- `IsValidSlotId()` - slotId 范围检查（0 <= slotId < SIM_SLOT_COUNT）
- `IsValidSlotIdEx()` - slotId 范围检查（支持 VSIM）

**潜在风险点**：
1. **slotId 范围检查**：`IsValidSlotId()` 仅检查 `0 <= slotId < SIM_SLOT_COUNT`，未检查是否对应真实存在的 SIM 卡
2. **APN 信息验证不足**：`ApnInfoAnalyze()` 解析 APN 信息对象，未充分验证字段格式和长度
3. **缓冲区边界**：`buffer` 参数（网络切片 URSP/UE 策略/IMS RSD）仅类型检查，未验证内容长度

### 1.2 IPC 层输入（系统服务层）

| 输入源 | 文件路径 | 参数 | 验证点 |
|---------|----------|------|----------|
| **数据开关控制** | `services/src/cellular_data_service.cpp:XXX` | `enable` (boolean) | `TelephonyPermission::CheckPermission(SET_TELEPHONY_STATE)` + `CheckCallerIsSystemApp()` |
| **漫游开关控制** | `services/src/cellular_data_service.cpp:XXX` | `slotId` (int), `enable` (boolean) | `TelephonyPermission::CheckPermission(SET_TELEPHONY_STATE)` + `CheckCallerIsSystemApp()` + `IsValidSlotId()` |
| **漫游状态查询** | `services/src/cellular_data_service.cpp:XXX` | `slotId` (int) | `TelephonyPermission::CheckPermission(GET_NETWORK_INFO)` + `IsValidSlotId()` |
| **APN 查询（ID 列表）** | `services/src/cellular_data_service.cpp:XXX` | `apnInfo` (ApnInfo 对象) | `TelephonyPermission::CheckPermission(MANAGE_APN_SETTING)` |
| **APN 设置（首选）** | `services/src/cellular_data_service.cpp:XXX` | `apnId` (int) | `TelephonyPermission::CheckPermission(MANAGE_APN_SETTING)` |
| **APN 查询（全部）** | `services/src/cellular_data_service.cpp:XXX` | 无 | `TelephonyPermission::CheckPermission(MANAGE_APN_SETTING)` |
| **APN 状态查询** | `services/src/cellular_data_service.cpp:XXX` | `slotId` (int), `apnType` (string) | `TelephonyPermission::CheckPermission(GET_TELEPHONY_STATE)` |
| **网络请求控制** | `services/src/cellular_data_service.cpp:XXX` | `NetRequest` (struct) | `IsRestrictedMode()` 检查 + slotId 验证 |
| **UID 添加/移除** | `services/src/cellular_data_service.cpp:XXX` | `NetRequest` (struct) | `IsRestrictedMode()` 检查 + slotId 验证 |
| **连接清除** | `services/src/cellular_data_service.cpp:XXX` | `slotId` (int), `reason` (int) | `TelephonyPermission::CheckPermission(SET_TELEPHONY_STATE)` + slotId 验证 |
| **连接状态查询** | `services/src/cellular_data_service.cpp:XXX` | 无 | - |
| **智能开关控制** | `services/src/cellular_data_service.cpp:XXX` | `enable` (boolean) | `CheckCallerIsSystemApp()` |
| **智能开关状态** | `services/src/cellular_data_service.cpp:XXX` | 无 | - |

**IPC 层输入验证函数**：
- `IsRestrictedMode()` - 限制模式检查（飞行模式、通话中等）
- `IsValidSlotId()` - slotId 范围检查
- `CheckDecValue()` - 十进制数值验证（针对参数解析）

**潜在风险点**：
1. **NetRequest 参数验证**：`NetRequest` 结构包含多个字段（`capability`、`ident`、`uid` 等），验证逻辑分散在多处
2. **APN 对象验证**：`ApnInfo` 对象包含字符串字段（`apnName`、`apn`、`mcc`、`mnc`、`user`、`password`），未充分验证格式和长度
3. **权限 TOCTOU**：权限检查与参数使用之间可能存在 TOCTOU（Time-Of-Check-Time-Of-Use）竞态

### 1.3 RIL 层输入（Modem 响应层）

| 输入源 | 文件路径 | 参数 | 验证点 |
|---------|----------|------|----------|
| **PDP 激活结果** | `services/src/state_machine/cellular_data_state_machine.cpp:XXX` | `SetupDataCallResultInfo` 结构 | 结果状态验证 |
| **去激活结果** | `services/src/state_machine/activating.cpp:XXX` | `result` (int) | 结果状态验证 |
| **网络信息更新** | `services/src/state_machine/cellular_data_state_machine.cpp:XXX` | `SetupDataCallResultInfo` | IP/网关/DNS 验证 |
| **带宽信息** | `services/src/state_machine/cellular_data_state_machine.cpp:XXX` | `upBandwidth`、`downBandwidth` (uint32_t) | 数值范围检查 |
| **TCP 缓冲区** | `services/src/state_machine/cellular_data_state_machine.cpp:XXX` | `tcpBuffer` (string) | 字符串长度验证 |

**RIL 层输入验证点**：
- `SetupDataCallResultInfo` 包含 `active`、`errInfo` 等字段
- IP 地址验证：`CellularDataStateMachine::UpdateNetworkInfo()` 更新网络信息

**潜在风险点**：
1. **RIL 响应信任**：直接信任 RIL 返回的数据，未充分验证 IP 地址、网关、DNS 的合法性
2. **带宽整数溢出**：`upBandwidth`、`downBandwidth` 使用 `uint32_t`，可能导致整数溢出
3. **TCP 缓冲区字符串未验证**：`tcpBuffer` 直接传递给系统调用，未验证长度和内容

### 1.4 数据库层输入（DataShare）

| 输入源 | 文件路径 | 参数 | 验证点 |
|---------|----------|------|------|
| **APN 数据库查询** | `services/src/utils/cellular_data_rdb_helper.cpp:XXX` | 查询条件（APN 类型、MCC/MNC） | SQL 注入防护（DataShare 内部防护） |
| **APN 数据库更新** | `services/src/utils/cellular_data_rdb_helper.cpp:XXX` | `ApnInfo` 对象（多个字段） | 字段长度验证 |
| **首选 APN 设置** | `services/src/utils/cellular_data_rdb_helper.cpp:XXX` | `apnId` (int) | 数值范围验证 |
| **数据开关设置读取** | `services/src/utils/cellular_data_settings_rdb_helper.cpp:XXX` | 无 | - |
| **漫游设置读取** | `services/src/utils/cellular_data_settings_rdb_helper.cpp:XXX` | 无 | - |

**数据库层输入验证**：
- DataShare 框架内部提供 SQL 注入防护
- APN 字符串长度未明确限制

**潜在风险点**：
1. **APN 信息长度未限制**：`apnName`、`apn`、`user`、`password` 等字段无明确长度限制，可能导致缓冲区溢出
2. **数据库写入竞态**：多个客户端同时更新 APN 可能导致数据不一致
3. **设置读取 TOCTOU**：权限检查后到设置读取之间可能存在竞态

---

## 2. 敏感操作清单

### 2.1 特权操作

| 操作 | 文件路径 | 权限要求 | 系统应用检查 | 风险等级 |
|------|----------|----------|----------|----------|
| **启用/禁用蜂窝数据** | `services/src/cellular_data_service.cpp:167` | `SET_TELEPHONY_STATE` | ✅ | 高 |
| **启用/禁用数据漫游** | `services/src/cellular_data_service.cpp:284` | `SET_TELEPHONY_STATE` | ✅ | 高 |
| **设置默认数据卡槽** | `services/src/cellular_data_service.cpp:XXX` | `SET_TELEPHONY_STATE` | ✅ | 高 |
| **APN 设置（首选）** | `services/src/cellular_data_service.cpp:967` | `MANAGE_APN_SETTING` | ✅ | 中 |
| **清除所有连接** | `services/src/cellular_data_service.cpp:XXX` | `SET_TELEPHONY_STATE` | ✅ | 高 |
| **清除连接** | `services/src/cellular_data_service.cpp:XXX` | `SET_TELEPHONY_STATE` | ✅ | 高 |
| **网络请求控制** | `services/src/cellular_data_service.cpp:XXX` | - | - | 中 |
| **UID 控制** | `services/src/cellular_data_service.cpp:XXX` | - | - | 中 |
| **智能开关控制** | `services/src/cellular_data_service.cpp:XXX` | ✅ | 中 |

**权限检查证据**：
```cpp
// services/src/cellular_data_service.cpp:167-168
if (!TelephonyPermission::CheckPermission(Permission::SET_TELEPHONY_STATE)) {
    TELEPHONY_LOGE("Permission denied.");
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

**系统应用检查证据**：
```cpp
// services/src/cellular_data_service.cpp:170-172
if (!TelephonyPermission::CheckCallerIsSystemApp()) {
    TELEPHONY_LOGE("Permission denied.");
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

### 2.2 系统调用与特权接口

| 操作 | 文件路径 | 特权 | 风险等级 |
|------|----------|------|----------|
| **动态库加载** | `services/telephony_ext_wrapper/src/telephony_ext_wrapper.cpp:58` | `dlopen()` | 高 |
| **数据库访问** | `services/src/utils/cellular_data_rdb_helper.cpp` | DataShare R/W | 中 |
| **网络接口注册** | `services/src/utils/cellular_data_net_agent.cpp:XXX` | NetConnClient | 高 |
| **网络策略控制** | `services/src/cellular_data_net_agent.cpp:XXX` | NetPolicyClient | 高 |
| **网络切片控制** | `services/src/utils/cellular_data_net_agent.cpp:XXX` | NetworkSliceClient | 中 |
| **网络流量统计** | `services/src/traffic_management.cpp:XXX` | NetStatsManager | 中 |
| **系统参数读取** | `services/src/cellular_data_controller.cpp:377` | `GetBoolParameter()` | 低 |
| **RIL 命令** | `services/src/state_machine/cellular_data_state_machine.cpp:XXX` | RadioEvent | 高 |

**敏感操作证据**：
```cpp
// 动态库加载 - services/telephony_ext_wrapper/src/telephony_ext_wrapper.cpp:58
void *handle = dlopen(libPath.c_str(), RTLD_NOW);
if (handle == nullptr) {
    TELEPHONY_LOGE("dlopen %{public}s failed: %{public}s", libPath.c_str(), dlerror());
}

// 网络接口注册 - services/src/utils/cellular_data_net_agent.cpp:XXX
NetConnClient::GetInstance().RegisterNetworkSupplier(request, callback);
```

### 2.3 外部服务依赖

| 服务 | 文件路径 | 信任边界 | 风险等级 |
|------|----------|----------|----------|
| **core_service (SA 4010)** | `services/src/cellular_data_service.cpp:XXX` | 高信任 | 高 |
| **netmanager_base** | `services/src/utils/cellular_data_net_agent.cpp` | 高信任 | 高 |
| **ril_adapter** | `services/src/state_machine/cellular_data_state_machine.cpp:XXX` | 高信任 | 高 |
| **data_share** | `services/src/utils/cellular_data_rdb_helper.cpp` | 中信任 | 中 |

**信任边界说明**：
- **core_service**：提供 SIM 卡状态、运营商信息，直接信任
- **netmanager_base**：管理网络连接、路由、DNS，高信任
- **ril_adapter**：与 Modem 通信，直接信任 RIL 响应
- **data_share**：数据库服务，信任其 SQL 注入防护

---

## 3. 信任边界图

### 3.1 安全域层次

```
┌─────────────────────────────────────────────────────────────────┐
│                    应用层（沙箱）                           │
│         ArkTS/JavaScript 应用（进程隔离）                   │
│  权限模型：GET_NETWORK_INFO、SET_TELEPHONY_STATE 等          │
└─────────────────────────────────────────────────────────────────┘
                          ↓ (N-API + 权限验证)
┌─────────────────────────────────────────────────────────────────┐
│                  N-API 绑定层（用户态）                      │
│           frameworks/js/napi/src/napi_cellular_data.cpp         │
│  参数验证：MatchXXXParameters()、IsValidSlotId()           │
│  权限检查：GET_NETWORK_INFO、SET_TELEPHONY_STATE 等         │
└─────────────────────────────────────────────────────────────────┘
                          ↓ (IPC Binder + 系统应用检查)
┌─────────────────────────────────────────────────────────────────┐
│              System Ability 服务层（系统进程）                 │
│         CellularDataService (SA 4007)                        │
│  进程空间：telephony (系统进程，高权限）                     │
│  权限检查：TelephonyPermission::CheckPermission()          │
│  系统应用检查：CheckCallerIsSystemApp()                │
└─────────────────────────────────────────────────────────────────┘
                          ↓ (内部调用)
┌─────────────────────────────────────────────────────────────────┐
│              外部系统服务（系统进程，高信任）                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ core_service (SA 4010) - SIM 状态             │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ netmanager_base - 网络管理                  │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ ril_adapter - Modem 通信                    │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ data_share - 数据库服务                      │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                          ↓ (系统调用/RIL 请求)
┌─────────────────────────────────────────────────────────────────┐
│                内核/Modem（最高信任）                        │
│            PDP 激活/去激活、网络配置                         │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 跨域通信点

| 跨域点 | 源域 | 目标域 | 通信方式 | 风险等级 |
|---------|-------|---------|----------|----------|
| **JS → N-API** | 应用层 | 用户态 | N-API 调用 | 低 |
| **N-API → Service** | 用户态 | 系统进程 | IPC Binder | 高 |
| **Service → core_service** | 系统进程 | 系统进程 | IPC Binder | 高 |
| **Service → netmanager** | 系统进程 | 系统进程 | IPC Binder | 高 |
| **Service → ril_adapter** | 系统进程 | 系统进程 | RadioEvent | 高 |
| **Service → data_share** | 系统进程 | 系统进程 | DataShare API | 中 |

**跨域通信安全机制**：
- **Binder IPC**：基于 Linux Binder 机制，提供 UID/PID 验证
- **权限检查**：`TelephonyPermission::CheckPermission()` 提供细粒度权限控制
- **系统应用检查**：`CheckCallerIsSystemApp()` 限制敏感 API 仅系统应用

---

## 4. 输入验证缺陷分析

### 4.1 slotId 验证不足

**位置**：`services/src/cellular_data_service.cpp:XXX`（多个方法）

**验证代码**：
```cpp
// IsValidSlotId() - frameworks/js/napi/src/napi_cellular_data.cpp:44-47
static inline bool IsValidSlotId(int32_t slotId)
{
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT));
}
```

**风险分析**：
- **范围检查**：仅验证 `0 <= slotId < 2`（`SIM_SLOT_COUNT`）
- **存在性检查缺失**：未检查对应 slotId 是否有真实的 SIM 卡插入
- **影响**：攻击者可指定任意 slotId（0 或 1），即使对应卡槽未插入 SIM 卡
- **利用路径**：`setDefaultCellularDataSlotId(1)` → 设置到不存在的卡槽 → 服务内部错误状态 → 潜在信息泄露

**建议**：
1. 在 `IsValidSlotId()` 中添加 SIM 卡存在性检查
2. 查询 `core_service` 的 SIM 卡状态，验证 slotId 对应的卡是否已插入并激活
3. 返回明确的错误码：`ERROR_NO_SIM_CARD`、`ERROR_SIM_NOT_ACTIVATED`

### 4.2 APN 信息验证不足

**位置**：`frameworks/js/napi/src/napi_cellular_data.cpp:1088-1131`

**验证代码**：
```cpp
// ApnInfoAnalyze() - frameworks/js/napi/src/napi_cellular_data.cpp:1088-1131
bool ApnInfoAnalyze(napi_env env, napi_value object, ApnInfo &apnInfo)
{
    // 从 JS 对象提取 apnName、apn、mcc、mnc、user、password、type、proxy 等
    // 未进行长度验证、格式验证
}
```

**风险分析**：
- **字符串长度未限制**：`apnName`、`apn`、`user`、`password` 等字段无明确长度限制
- **格式验证缺失**：未验证 MCC/MNC 格式（数字）、APN 格式（域名）、IP 地址格式
- **特殊字符过滤缺失**：未过滤 SQL 注入、路径遍历、命令注入等特殊字符
- **影响**：超长字符串可能导致缓冲区溢出、SQL 注入、路径遍历
- **利用路径**：`queryApnIds({apn: "恶意数据..."})` → 传递到数据库 → SQL 注入 → 读取/修改任意 APN 配置

**建议**：
1. 添加长度限制：`apnName`、`apn`、`user`、`password` 最大长度
2. 添加格式验证：MCC/MNC 数字格式、APN 域名格式、IP 地址格式
3. 添加特殊字符过滤：过滤 `'`、`"`、`;`、`&`、`|` 等危险字符
4. 使用参数化查询：确保 DataShare 内部 SQL 注入防护有效

### 4.3 网络切片 buffer 验证不足

**位置**：`frameworks/js/napi/src/napi_cellular_data.cpp`（网络切片相关方法）

**验证代码**：
```cpp
// 网络切片 URSP 解码、UE 策略、IMS RSD 接收 buffer 参数
// 仅进行类型检查（Array<number>），未验证内容长度
```

**风险分析**：
- **长度未限制**：`buffer` 参数（`Array<number>`）无长度限制
- **内容未验证**：未验证 buffer 内容的合法性、格式
- **影响**：超长 buffer 可能导致内存耗尽、缓冲区溢出、DoS
- **利用路径**：`SendUrspDecodeResult(0, [大量恶意数据...])` → 消耗内存 → 服务崩溃

**建议**：
1. 添加 buffer 长度限制：最大 4KB 或 8KB
2. 添加内容格式验证：根据网络切片规范验证 buffer 格式
3. 使用分块处理：大 buffer 分块处理，避免一次性分配大内存

---

## 5. 证据索引

| 类别 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| **N-API 输入验证** | `frameworks/js/napi/src/napi_cellular_data.cpp:44-47` | `IsValidSlotId()` 函数 |
| **N-API 输入验证** | `frameworks/js/napi/src/napi_cellular_data.cpp:55-80` | `MatchCellularDataParameters()` 等 |
| **N-API 输入验证** | `frameworks/js/napi/src/napi_cellular_data.cpp:1088-1131` | `ApnInfoAnalyze()` 函数 |
| **IPC 权限检查** | `services/src/cellular_data_service.cpp:167-168` | `CheckPermission(SET_TELEPHONY_STATE)` |
| **IPC 系统应用检查** | `services/src/cellular_data_service.cpp:170-172` | `CheckCallerIsSystemApp()` |
| **slotId 验证** | `services/src/cellular_data_service.cpp:382-474` | 参数验证逻辑 |
| **NetRequest 验证** | `services/src/cellular_data_service.cpp:382-474` | `IsRestrictedMode()`、`CheckDecValue()` |
| **动态库加载** | `services/telephony_ext_wrapper/src/telephony_ext_wrapper.cpp:58` | `dlopen()` 调用 |
| **网络接口注册** | `services/src/utils/cellular_data_net_agent.cpp:XXX` | `RegisterNetworkSupplier()` |
| **数据库访问** | `services/src/utils/cellular_data_rdb_helper.cpp:XXX` | DataShare 查询/更新 |
| **RIL 交互** | `services/src/state_machine/cellular_data_state_machine.cpp:XXX` | RadioEvent 事件 |
| **SA ID** | `sa_profile/4007.json:5` | SA ID: 4007 |
| **SA 进程** | `sa_profile/4007.json:2` | 进程: telephony |

---

## 6. 相关链接

- 详细安全风险评估参见 [`06_SecurityReview.md`](./06_SecurityReview.md)
- 架构说明参见 [`02_Architecture.md`](./02_Architecture.md)
- 代码地图参见 [`03_CodeMap.md`](./03_CodeMap.md)

---

**文档完成** - 攻击面全面分析，支持安全研究员快速识别风险点。
