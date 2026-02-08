# 目录结构与模块职责

## 2.1 顶层目录结构

```
/base/telephony/telephony_data/
├── common/                          # 公共组件和工具
│   ├── include/                     # 公共头文件
│   └── src/                         # 公共实现
├── sim/                             # SIM 卡数据存储模块
│   ├── include/                     # SIM 头文件
│   └── src/                         # SIM 实现
├── sms_mms/                         # 短信/多媒体消息模块
│   ├── include/                     # SMS/MMS 头文件
│   └── src/                         # SMS/MMS 实现
├── pdp_profile/                     # PDP/APN 配置文件模块
│   ├── include/                     # PDP 头文件
│   └── src/                         # PDP 实现
├── opkey/                           # 运营商密钥模块
│   ├── include/                     # OpKey 头文件
│   └── src/                         # OpKey 实现
├── opkey_version/                   # 运营商密钥版本模块
│   ├── include/                     # 版本头文件
│   └── src/                         # 版本实现
├── global_params/                  # 全局参数模块
│   ├── include/                     # 全局参数头文件
│   └── src/                         # 全局参数实现
├── interfaces/                      # 对外接口定义
│   └── innerkits/                  # 内部套件 API
├── etc/                            # 配置文件目录
│   └── *.json                      # 预置数据配置
├── entry/                          # 应用入口
├── signature/                      # 签名配置
├── figures/                        # 文档资源
├── test/                           # 测试代码 (已忽略)
└── BUILD.gn                       # 根构建文件
```

---

## 2.2 模块职责详解

### 2.2.1 common - 公共组件模块

**职责**: 提供所有数据模块共享的基础设施

| 组件 | 文件 | 职责 |
|------|------|------|
| 日志封装 | `data_storage_log_wrapper.h` | 统一日志接口 |
| RDB 基础 | `rdb_base_helper.h/cpp` | 数据库初始化和基础操作 |
| 权限工具 | `permission_util.h/cpp` | 权限检查封装 |
| DataShare 存根 | `telephony_datashare_stub_impl.h/cpp` | IPC 存根实现 |
| 解析工具 | `parser_util.h/cpp` | 数据解析封装 |
| 时间工具 | `time_util.h/cpp` | 时间格式化 |
| 偏好设置 | `preferences_util.h/cpp` | SharedPreferences 访问 |
| 错误码 | `data_storage_errors.h` | 统一错误码定义 |

**代码证据**: `common/include/` 目录下包含 11 个头文件

### 2.2.2 sim - SIM 卡数据模块

**职责**: 存储和管理 SIM 卡相关信息

| 组件 | 文件 | 职责 |
|------|------|------|
| SIM Ability | `sim_ability.h/cpp` | SIM 数据 DataAbility |
| SIM 回调 | `rdb_sim_callback.h/cpp` | RDB 操作回调处理 |
| SIM 帮助类 | `rdb_sim_helper.h/cpp` | SIM 数据库操作封装 |

**数据结构**: `SimData`
- SIM_ID, ICC_ID, CARD_ID, SLOT_INDEX
- SHOW_NAME, PHONE_NUMBER 等

**代码证据**: `sim/include/sim_ability.h`, `sim/src/sim_ability.cpp`

### 2.2.3 sms_mms - 短信/多媒体消息模块

**职责**: 存储和管理 SMS/MMS 消息数据

| 组件 | 文件 | 职责 |
|------|------|------|
| SMS Ability | `sms_mms_ability.h/cpp` | SMS 数据 DataAbility |
| SMS 回调 | `rdb_sms_mms_callback.h/cpp` | RDB 操作回调处理 |
| SMS 帮助类 | `rdb_sms_mms_helper.h/cpp` | SMS 数据库操作封装 |

**数据结构**: `SmsMmsInfo`
- MSG_ID, SENDER_NUMBER, RECEIVER_NUMBER
- MSG_CONTENT, MSG_TITLE, GROUP_ID 等

**代码证据**: `sms_mms/include/sms_mms_ability.h`, `sms_mms/src/sms_mms_ability.cpp`

### 2.2.4 pdp_profile - PDP/APN 配置模块

**职责**: 存储和管理网络运营商 PDP 配置文件

| 组件 | 文件 | 职责 |
|------|------|------|
| PDP Ability | `pdp_profile_ability.h/cpp` | PDP 配置 DataAbility |
| PDP 回调 | `rdb_pdp_profile_callback.h/cpp` | RDB 操作回调处理 |
| PDP 帮助类 | `rdb_pdp_profile_helper.h/cpp` | PDP 数据库操作封装 |
| APN 加密 | `apn_encryption_util.h/cpp` | APN 数据加密处理 |
| 结果集桥接 | `pdp_result_set_bridge.h/cpp` | 结果集格式转换 |

**数据结构**: `PdpProfileData`
- PROFILE_ID, APN, AUTH_TYPE
- MCC, MNC, USER_NAME, PASSWORD 等

**代码证据**: `pdp_profile/include/pdp_profile_ability.h`, `pdp_profile/src/apn_encryption_util.cpp`

### 2.2.5 opkey - 运营商密钥模块

**职责**: 存储和管理运营商识别密钥 (OpKey)

| 组件 | 文件 | 职责 |
|------|------|------|
| OpKey Ability | `opkey_ability.h/cpp` | OpKey 数据 DataAbility |
| OpKey 回调 | `rdb_opkey_callback.h/cpp` | RDB 操作回调处理 |
| OpKey 帮助类 | `rdb_opkey_helper.h/cpp` | OpKey 数据库操作封装 |

**数据结构**: `OpKeyData`
- MCCMNC, GID1, GID2, OPERATOR_KEY

**代码证据**: `opkey/include/opkey_ability.h`, `opkey/src/opkey_ability.cpp`

### 2.2.6 opkey_version - 运营商密钥版本模块

**职责**: 管理 OpKey 版本信息

| 组件 | 文件 | 职责 |
|------|------|------|
| 版本 Ability | `opkey_version_ability.h/cpp` | 版本查询 Ability |
| 结果集桥接 | `opkey_version_result_set_bridge.h/cpp` | 结果集格式转换 |

**代码证据**: `opkey_version/include/opkey_version_ability.h`

### 2.2.7 global_params - 全局参数模块

**职责**: 存储和管理全局系统参数

| 组件 | 文件 | 职责 |
|------|------|------|
| 全局参数 Ability | `global_params_ability.h/cpp` | 全局参数 DataAbility |
| 参数回调 | `rdb_global_params_callback.h/cpp` | RDB 操作回调处理 |
| 参数帮助类 | `rdb_global_params_helper.h/cpp` | 全局参数数据库操作 |

**数据结构**: `GlobalParams`
- 紧急号码 (ECC), 号码匹配规则

**代码证据**: `global_params/include/global_params_ability.h`

### 2.2.8 interfaces/innerkits - 对外接口模块

**职责**: 定义供其他子系统使用的公开 API

| 头文件 | 描述 |
|--------|------|
| `sim_data.h` | SIM 数据结构定义 |
| `sms_mms_data.h` | SMS/MMS 数据结构定义 |
| `pdp_profile_data.h` | PDP 配置文件结构定义 |
| `opkey_data.h` | OpKey 数据结构定义 |
| `global_params_data.h` | 全局参数数据结构定义 |

**代码证据**: `interfaces/innerkits/include/*.h`

### 2.2.9 etc - 配置文件目录

**职责**: 预置的 JSON 格式系统配置

| 文件 | 描述 |
|------|------|
| `pdp_profile.json` | 默认 APN 配置 |
| `ecc_data.json` | 紧急呼叫号码配置 |
| `number_match.json` | 号码匹配规则 |
| `OpkeyInfo.json` | 运营商密钥信息 |

**安装路径**: `/system/etc/telephony/`

**代码证据**: `etc/BUILD.gn` 定义了预编译目标

```gn
ohos_prebuilt_etc("pdp_profile_default") {
    source = "pdp_profile.json"
    relative_install_dir = "telephony"
}
```

---

## 2.3 模块依赖关系

```
                    ┌─────────────────┐
                    │   Application   │
                    │    Framework    │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  DataShare    │◄───│ telephony_    │    │    Utils      │
│   Framework   │    │ data_storage │───►│  Subsystem    │
└───────────────┘    └───────┬───────┘    └───────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│Relational     │    │    Security   │    │    other       │
│    Store      │    │   Framework   │    │  Components   │
└───────────────┘    └───────────────┘    └───────────────┘
```

### 内部模块依赖

| 模块 | 被依赖 | 依赖 |
|------|--------|------|
| common | 所有模块 | Utils, Security |
| sim | - | common, RDB |
| sms_mms | - | common, RDB |
| pdp_profile | - | common, RDB, Security |
| opkey | - | common, RDB |
| global_params | - | common, RDB |

**代码证据**: `BUILD.gn` 中 `deps` 配置

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [01_Overview](01_Overview.md) |
| 架构说明 | [03_Architecture](03_Architecture.md) |
| 对外 API | [04_DataShare_API](04_DataShare_API.md) |
| 构建系统 | [06_Build](06_Build.md) |

---

*最后更新: 2024-02-06*
