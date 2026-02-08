# 项目概览

## 1.1 项目定位

**Telephony Data Storage**（电话数据存储）是 OpenHarmony Telephony（电话）子系统的核心数据持久化模块，负责为 SIM 卡管理、短信/多媒体消息、网络运营商配置等关键业务提供可靠的数据存储服务。

### 核心能力

| 能力 | 描述 | 代码证据 |
|------|------|----------|
| **SIM 卡数据持久化** | 存储 SIM 卡相关信息（ICCID、PhoneNumber、SlotIndex 等） | `sim/include/sim_ability.h` |
| **短信/多媒体消息存储** | 存储 SMS/MMS 消息数据（发送方、接收方、内容等） | `sms_mms/include/sms_mms_ability.h` |
| **PDP/APN 配置管理** | 存储网络运营商 PDP 配置文件（含 APN 加密） | `pdp_profile/include/pdp_profile_ability.h` |
| **运营商密钥管理** | 存储 OpKey 运营商识别密钥 | `opkey/include/opkey_ability.h` |
| **全局参数存储** | 存储紧急号码、号码匹配等全局配置 | `global_params/include/global_params_ability.h` |

### 技术定位

```
OpenHarmony 系统
    │
    ├── Telephony 子系统
    │   ├── telephony_core_service (核心服务)
    │   ├── telephony_call_manager (通话管理)
    │   ├── telephony_cellular_call (蜂窝通话)
    │   ├── telephony_sms_mms (短信服务)
    │   └── telephony_data_storage ← 本模块 (数据存储)
    │
    └── DataShare 框架 (数据共享)
```

---

## 1.2 模块边界

### 输入边界

| 来源 | 类型 | 描述 |
|------|------|------|
| **应用层** | DataShare API | 第三方应用通过 DataShareHelper 访问数据 |
| ** telephony_core_service** | IPC/System Ability | 核心服务通过 SA 调用本模块 |
| **系统配置** | JSON 文件 | 预置的 APN、紧急号码、运营商密钥配置 |

### 输出边界

| 目标 | 类型 | 描述 |
|------|------|------|
| **RDB 数据库** | 关系型存储 | SQLite3 数据库持久化 |
| **应用层** | DataShare ResultSet | 查询结果返回给调用方 |
| **系统配置** | JSON 文件 | 运行时读取预置配置 |

---

## 1.3 运行环境

### 硬件要求

| 资源 | 要求 | 说明 |
|------|------|------|
| 内存 | ≥ 64MB | 基础运行内存 |
| 存储 | ≥ 10MB | 数据库文件和配置空间 |

### 软件依赖

| 依赖项 | 版本 | 用途 |
|--------|------|------|
| **Utils 子系统** | - | 日志、基础工具类 |
| **Application Framework** | - | Ability 框架、生命周期管理 |
| **DataShare 框架** | - | 数据共享与访问接口 |
| **Relational Store** | - | RDB 关系型数据库存储 |
| **Security Framework** | - | AccessToken 权限管理 |

### 权限要求

| 权限名称 | 保护级别 | 用途 |
|----------|----------|------|
| `ohos.permission.GET_TELEPHONY_STATE` | system_basic | 查询电话相关数据 |
| `ohos.permission.SET_TELEPHONY_STATE` | system_basic | 修改电话相关数据 |
| `ohos.permission.READ_MESSAGES` | system_basic | 读取短信/多媒体消息 |

**代码证据**: `common/include/permission_util.h`

```cpp
static constexpr const char *SET_TELEPHONY_STATE = "ohos.permission.SET_TELEPHONY_STATE";
static constexpr const char *GET_TELEPHONY_STATE = "ohos.permission.GET_TELEPHONY_STATE";
static constexpr const char *READ_MESSAGES = "ohos.permission.READ_MESSAGES";
```

---

## 1.4 关键概念

### DataShare 与 DataAbility

| 概念 | 描述 |
|------|------|
| **DataShare** | OpenHarmony 数据共享框架，提供跨应用数据访问能力 |
| **DataAbility** | 数据提供方的抽象，通过 URI 标识资源，支持 CRUD 操作 |
| **DataShareHelper** | 客户端工具类，用于连接 DataAbility 并执行数据操作 |
| **URI Scheme** | 数据资源标识，如 `datashare:///com.ohos.simability` |

### RDB (Relational Database)

| 概念 | 描述 |
|------|------|
| **RdbStore** | RDB 数据库实例，提供关系型数据存储 |
| **RdbPredicates** | 查询谓词，定义数据过滤条件 |
| **DataSharePredicates** | DataShare 查询条件，支持更灵活的数据筛选 |

### 系统能力 (System Ability)

| 概念 | 描述 |
|------|------|
| **SA (System Ability)** | 系统级服务，提供系统核心功能 |
| **SystemAbilityManager** | SA 管理器，负责 SA 的注册与发现 |
| **DataShareExtAbility** | DataAbility 的扩展，支持 DataShare 协议 |

---

## 1.5 核心数据结构

### SimData (SIM 卡数据)

| 字段 | 类型 | 描述 |
|------|------|------|
| SIM_ID | int64_t | SIM 卡唯一标识 |
| ICC_ID | string | SIM 卡集成电路卡标识 |
| CARD_ID | int64_t | 卡槽标识 |
| SLOT_INDEX | int | 卡槽索引 |
| SHOW_NAME | string | 显示名称 |
| PHONE_NUMBER | string | 电话号码 |

**代码证据**: `interfaces/innerkits/include/sim_data.h`

### SmsMmsInfo (短信/多媒体消息)

| 字段 | 类型 | 描述 |
|------|------|------|
| MSG_ID | int64_t | 消息唯一标识 |
| SENDER_NUMBER | string | 发送方号码 |
| RECEIVER_NUMBER | string | 接收方号码 |
| MSG_CONTENT | string | 消息内容 |
| MSG_TITLE | string | 消息标题 |
| GROUP_ID | int | 会话组 ID |

**代码证据**: `interfaces/innerkits/include/sms_mms_data.h`

### PdpProfileData (PDP/APN 配置)

| 字段 | 类型 | 描述 |
|------|------|------|
| PROFILE_ID | int64_t | 配置文件唯一标识 |
| APN | string | 接入点名称 |
| AUTH_TYPE | int | 认证类型 |
| MCC | string | 移动国家代码 |
| MNC | string | 移动网络代码 |

**代码证据**: `interfaces/innerkits/include/pdp_profile_data.h`

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 目录结构 | [02_Directory_Structure](02_Directory_Structure.md) |
| 架构说明 | [03_Architecture](03_Architecture.md) |
| 对外 API | [04_DataShare_API](04_DataShare_API.md) |
| 安全评审 | [07_Security](07_Security.md) |

---

*最后更新: 2024-02-06*
