# contacts_data 构建文档

> 本文档详细说明 contacts_data 子系统的 GN 构建配置、编译目标和产物。

## 构建系统概述

contacts_data 子系统使用 OpenHarmony 的 GN（Generate Ninja）构建系统进行编译管理。GN 构建系统通过 .gn、.gni 和 BUILD.gn 配置文件定义编译目标、依赖关系和构建选项，最终生成 Ninja 构建文件执行实际编译。

contacts_data 的构建配置涉及三个主要层次：
- **根构建配置**：`BUILD.gn` 定义顶层编译目标和组件配置
- **模块构建配置**：`contacts/BUILD.gn`、`contactsCJ/BnIild.gn` 定义子模块编译目标
- **构建变量定义**：`contact.gni` 定义共享的构建变量和配置

### 构建入口

| 文件 | 路径 | 说明 |
|------|------|------|
| 根构建文件 | `//applications/standard/contacts_data/BUILD.gn` | 顶层构建入口 |
| contacts 构建 | `//applications/standard/contacts_data/contacts/BUILD.gn` | N-API 模块构建 |
| contactsCJ 构建 | `//applications/standard/contacts_data/contactsCJ/BUILD.gn` | Inner API 模块构建 |
| 变量定义 | `//applications/standard/contacts_data/contact.gni` | 共享构建变量 |

## 关键 Targets 清单

### contacts 模块（编译目标：`//applications/standard/contacts_data/contacts:contact`）

**目标类型**：`ohos_shared_library`（动态库）

**输出产物**：`libcontact.so`

**源文件列表**：

| 序号 | 源文件 | 路径 | 功能说明 |
|------|--------|------|----------|
| 1 | contacts_api.cpp | contacts/src/ | N-API 接口实现 |
| 2 | contacts_build.cpp | contacts/src/ | 联系人构建逻辑 |
| 3 | contacts_control.cpp | contacts/src/ | 联系人控制逻辑 |
| 4 | contacts_napi_utils.cpp | contacts/src/ | N-API 工具函数 |
| 5 | contacts_telephony_permission.cpp | contacts/src/ | 电话权限处理 |
| 6 | native_module.cpp | contacts/src/ | 原生模块注册 |
| 7 | result_convert.cpp | contacts/src/ | 结果转换工具 |

**依赖配置**：

| 依赖类型 | 依赖项 | 说明 |
|----------|--------|------|
| external_deps | ability_base:zuri | URI 处理 |
| external_deps | ability_runtime:abilitykit_native | 能力工具 |
| external_deps | ability_runtime:app_context | 应用上下文 |
| external_deps | ability_runtime:app_manager | 应用管理 |
| external_deps | ability_runtime:extensionkit_native | 扩展工具 |
| external_deps | ability_runtime:napi_base_context | N-API 基础上下文 |
| external_deps | ability_runtime:napi_common | N-API 公共接口 |
| external_deps | access_token:libaccesstoken_sdk | 访问令牌 |
| external_deps | access_token:libprivacy_sdk | 隐私 SDK |
| external_deps | c_utils:utils | C 工具库 |
| external_deps | data_share:datashare_consumer | 数据共享消费者 |
| external_deps | hilog:libhilog | 日志库 |
| external_deps | ipc:ipc_single | IPC 单例 |
| external_deps | napi:ace_napi | ACE N-API |
| external_deps | relational_store:native_dataability | 原生 DataAbility |
| external_deps | relational_store:native_rdb | 原生关系数据库 |
| external_deps | samgr:samgr_proxy | SAMGR 代理 |
| deps | :contact_abc | ABC 字节码 |
| deps | :contact_js | JS 源文件 |

**包含目录**：

| 目录 | 说明 |
|------|------|
| contacts/include | N-API 头文件目录 |

**编译宏定义**：

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| CONTACTSDATA_LOG_TAG | "ContactsApi" | 日志标签 |
| LOG_DOMAIN | 0xD001F09 | 日志域 |

**Sanitizer 配置**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| cfi | true | 启用控制流完整性检查 |
| cfi_cross_dso | true | 跨 DSO 的 CFI 检查 |
| debug | false | 关闭调试模式 |

**安装配置**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| relative_install_dir | "module" | 安装相对路径 |
| part_name | "contacts_data" | 部件名称 |
| subsystem_name | "applications" | 子系统名称 |

### contactsCJ 模块（编译目标：`//applications/standard/contacts_data/contactsCJ:contact_cj`）

**目标类型**：`ohos_shared_library`（动态库）

**输出产物**：`libcontact_cj.so`

**源文件列表**：

| 序号 | 源文件 | 路径 | 功能说明 |
|------|--------|------|----------|
| 1 | contacts.cpp | contactsCJ/src/ | 联系人核心实现 |
| 2 | contacts_control.cpp | contactsCJ/src/ | 联系人控制逻辑 |
| 3 | contacts_ffi.cpp | contactsCJ/src/ | FFI 接口实现 |
| 4 | contacts_permission.cpp | contactsCJ/src/ | 权限处理 |
| 5 | contacts_utils.cpp | contactsCJ/src/ | 工具函数 |

**依赖配置**：

| 依赖类型 | 依赖项 | 说明 |
|----------|--------|------|
| external_deps | ability_base:want | Want 处理 |
| external_deps | ability_base:zuri | URI 处理 |
| external_deps | ability_runtime:ability_connect_callback_stub | 能力连接回调 |
| external_deps | ability_runtime:ability_manager | 能力管理 |
| external_deps | ability_runtime:abilitykit_native | 能力工具 |
| external_deps | ability_runtime:app_context | 应用上下文 |
| external_deps | ability_runtime:napi_base_context | N-API 基础上下文 |
| external_deps | ability_runtime:ui_extension | UI 扩展 |
| external_deps | access_token:libaccesstoken_sdk | 访问令牌 |
| external_deps | access_token:libprivacy_sdk | 隐私 SDK |
| external_deps | ace_engine:ace_uicontent | ACE UI 内容 |
| external_deps | c_utils:utils | C 工具库 |
| external_deps | data_share:cj_data_share_predicates_ffi | CJ 数据共享谓词 FFI |
| external_deps | data_share:datashare_consumer | 数据共享消费者 |
| external_deps | hilog:libhilog | 日志库 |
| external_deps | ipc:ipc_core | IPC 核心 |
| external_deps | ipc:ipc_single | IPC 单例 |
| external_deps | napi:ace_napi | ACE N-API |
| external_deps | napi:cj_bind_ffi | CJ 绑定 FFI |
| external_deps | napi:cj_bind_native | CJ 绑定原生 |
| external_deps | samgr:samgr_proxy | SAMGR 代理 |

**包含目录**：

| 目录 | 说明 |
|------|------|
| contactsCJ/include | Inner API 头文件目录 |

**编译宏定义**：

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| CONTACTSDATA_LOG_TAG | "ContactsFfi" | 日志标签 |
| LOG_DOMAIN | 0xD001F09 | 日志域 |

**Sanitizer 配置**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| cfi | true | 启用控制流完整性检查 |
| cfi_cross_dso | true | 跨 DSO 的 CFI 检查 |
| debug | false | 关闭调试模式 |

**Inner API 标记**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| innerapi_tags | ["platformsdk"] | 标记为平台 SDK 接口 |

**安装配置**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| part_name | "contacts_data" | 部件名称 |
| subsystem_name | "applications" | 子系统名称 |

### contactsdataability 模块（编译目标：`//applications/standard/contacts_data:contactsdataability`）

**目标类型**：`ohos_shared_library`（动态库）

**输出产物**：`libcontactsdataability.so`

**源文件列表**（按模块分类）：

| 模块 | 源文件 | 功能说明 |
|------|--------|----------|
| account | account_data_collection.cpp | 账户数据集合 |
| account | account_manager.cpp | 账户管理 |
| account | account_sync.cpp | 账户同步 |
| common/utils | contacts_common_event.cpp | 公共事件 |
| common/utils | contacts_json_utils.cpp | JSON 工具 |
| common/utils | contacts_path.cpp | 路径工具 |
| common/utils | contacts_string_utils.cpp | 字符串工具 |
| common/utils | file_utils.cpp | 文件工具 |
| common/utils | merge_utils.cpp | 合并工具 |
| common/utils | predicates_convert.cpp | 谓词转换 |
| common/utils | sql_analyzer.cpp | SQL 分析器 |
| common/utils | telephony_permission.cpp | 电话权限 |
| common/utils | uri_utils.cpp | URI 工具 |
| datadisasterrecovery | database_disaster_recovery.cpp | 数据库灾难恢复 |
| merge | candidate.cpp | 候选联系人 |
| merge | candidate_status.cpp | 候选状态 |
| merge | match_candidate.cpp | 匹配候选 |
| merge | merger_contacts.cpp | 联系人合并 |
| sinicization | character_transliterate.cpp | 字符转写 |
| sinicization | construction_name.cpp | 姓名构建 |
| calllog | calllog_ability.cpp | 通话记录能力 |
| calllog | calllog_database.cpp | 通话记录数据库 |
| contacts | contacts.cpp | 联系人核心 |
| contacts | contacts_account.cpp | 联系人账户 |
| contacts | contacts_data_ability.cpp | 联系人 DataAbility |
| contacts | contacts_database.cpp | 联系人数据库 |
| contacts | contacts_datashare_stub_impl.cpp | DataShare 存根实现 |
| contacts | contacts_type.cpp | 联系人类型 |
| contacts | contacts_update_helper.cpp | 更新辅助 |
| contacts | profile_database.cpp | 配置数据库 |
| contacts | raw_contacts.cpp | 原始联系人 |
| quicksearch | contacts_search.cpp | 联系人搜索 |
| voicemail | voicemail_ability.cpp | 语音信箱能力 |
| voicemail | voicemail_database.cpp | 语音信箱数据库 |

**包含目录**（public_config）：

| 目录 | 说明 |
|------|------|
| ability/common/include | 公共能力头文件 |
| ability/common/utils/include/ | 工具头文件 |
| dataBusiness/voicemail/include | 语音信箱头文件 |
| dataBusiness/calllog/include | 通话记录头文件 |
| dataBusiness/contacts/include | 联系人头文件 |
| ability/account/include | 账户头文件 |
| dataBusiness/quicksearch/include | 快速搜索头文件 |
| ability/sinicization/include | 汉字转拼音头文件 |
| ability/merge/include | 合并头文件 |
| ability/datadisasterrecovery/include | 灾难恢复头文件 |

**依赖配置**：

| 依赖类型 | 依赖项 | 说明 |
|----------|--------|------|
| external_deps | ability_base:want | Want 处理 |
| external_deps | ability_base:zuri | URI 处理 |
| external_deps | ability_runtime:ability_connect_callback_stub | 能力连接回调 |
| external_deps | ability_runtime:abilitykit_utils | 能力工具 |
| external_deps | ability_runtime:app_manager | 应用管理 |
| external_deps | ability_runtime:app_context | 应用上下文 |
| external_deps | ability_runtime:dataobs_manager | 数据观察者管理 |
| external_deps | ability_runtime:extensionkit_native | 扩展工具 |
| external_deps | access_token:libaccesstoken_sdk | 访问令牌 SDK |
| external_deps | access_token:libprivacy_sdk | 隐私 SDK |
| external_deps | access_token:libtokenid_sdk | 令牌 ID SDK |
| external_deps | bundle_framework:appexecfwk_base | 应用框架基础 |
| external_deps | bundle_framework:appexecfwk_core | 应用框架核心 |
| external_deps | c_utils:utils | C 工具库 |
| external_deps | c_utils:utilsbase | C 工具基础库 |
| external_deps | common_event_service:cesfwk_innerkits | 公共事件服务 |
| external_deps | data_share:datashare_common | 数据共享公共库 |
| external_deps | data_share:datashare_provider | 数据共享提供者 |
| external_deps | hilog:libhilog | 日志库 |
| external_deps | ipc:ipc_napi | IPC N-API |
| external_deps | ipc:ipc_single | IPC 单例 |
| external_deps | jsoncpp:jsoncpp | JSON 处理库 |
| external_deps | napi:ace_napi | ACE N-API |
| external_deps | node:node_header_notice | Node 头文件 |
| external_deps | os_account:os_account_innerkits | 账户内部套件 |
| external_deps | relational_store:native_dataability | 原生 DataAbility |
| external_deps | relational_store:native_rdb | 原生关系数据库 |
| external_deps | relational_store:rdb_data_share_adapter | RDB DataShare 适配器 |

**编译宏定义**：

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| CONTACTSDATA_LOG_TAG | "ContactsData" | 日志标签 |
| LOG_DOMAIN | 0xD001F09 | 日志域 |

### DataAbility 应用（编译目标：`//applications/standard/contacts_data:Contacts_DataAbility`）

**目标类型**：`ohos_hap`（HAP 应用包）

**输出产物**：`Contacts_DataAbility.hap`

**依赖配置**：

| 依赖类型 | 依赖项 | 说明 |
|----------|--------|------|
| deps | :Contacts_DataAbility_js_assets | JS 资源 |
| deps | :Contacts_DataAbility_resources | 资源文件 |
| shared_libraries | :contactsdataability | SA 服务库 |

**签名配置**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| certificate_profile | signature/contactsdata.p7b | 签名文件 |
| hap_name | "Contacts_DataAbility" | HAP 名称 |
| module_install_dir | "app/com.ohos.contactsdataability" | 安装路径 |

## 编译产物清单

### 动态库产物

| 产物名 | 源 target | 预计路径 | 说明 |
|--------|-----------|----------|------|
| libcontact.so | //applications/standard/contacts_data/contacts:contact | out/.../system/lib/ | N-API 实现库 |
| libcontact_cj.so | //applications/standard/contacts_data/contactsCJ:contact_cj | out/.../system/lib/ | Inner API 实现库 |
| libcontactsdataability.so | //applications/standard/contacts_data:contactsdataability | out/.../system/lib/ | SA 服务库 |

### HAP 产物

| 产物名 | 源 target | 预计路径 | 说明 |
|--------|-----------|----------|------|
| Contacts_DataAbility.hap | //applications/standard/contacts_data:Contacts_DataAbility | out/.../ | DataAbility 应用包 |

### ABC 产物

| 产物名 | 源 target | 预计路径 | 说明 |
|--------|-----------|----------|------|
| contact.abc | contacts 模块 | out/.../ | JS 字节码 |

## 产物与 Target 映射关系

```
┌─────────────────────────────────────────────────────────────────┐
│                         编译产物映射图                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  contacts:contact ──────► libcontact.so                         │
│       │                      │                                  │
│       ├── contacts_api.cpp   │                                  │
│       ├── contacts_build.cpp │                                  │
│       ├── contacts_control.cpp                                  │
│       ├── contacts_napi_utils.cpp                               │
│       ├── contacts_telephony_permission.cpp                     │
│       ├── native_module.cpp                                     │
│       └── result_convert.cpp                                    │
│                                                                 │
│  contactsCJ:contact_cj ──► libcontact_cj.so                     │
│       │                      │                                  │
│       ├── contacts.cpp       │                                  │
│       ├── contacts_control.cpp                                  │
│       ├── contacts_ffi.cpp                                      │
│       ├── contacts_permission.cpp                               │
│       └── contacts_utils.cpp                                    │
│                                                                 │
│  :contactsdataability ──► libcontactsdataability.so             │
│       │                      │                                  │
│       ├── ability/account/  │                                  │
│       ├── ability/common/   │                                  │
│       ├── ability/merge/    │                                  │
│       ├── ability/sinicization/                                 │
│       ├── dataBusiness/calllog/                                 │
│       ├── dataBusiness/contacts/                                │
│       ├── dataBusiness/quicksearch/                             │
│       └── dataBusiness/voicemail/                               │
│                                                                 │
│  :Contacts_DataAbility ──► Contacts_DataAbility.hap             │
│       │                      │                                  │
│       ├── :Contacts_DataAbility_js_assets                       │
│       ├── :Contacts_DataAbility_resources                       │
│       └── :contactsdataability (shared_library)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 运行时加载关系

contacts_data 子系统的模块在运行时按以下顺序加载：

```
系统启动
    │
    ▼
1. 加载系统基础库 (libc.so, libpthread.so, libhilog.so 等)
    │
    ▼
2. 加载 contactsdataability.so (SA 服务)
    │
    ▼
3. 注册 System Ability (contactsDataAbility)
    │
    ▼
4. 应用启动时加载 libcontact.so (N-API)
    │
    ▼
5. 应用调用 N-API 接口
    │
    ▼
6. N-API 调用 contactsdataability 服务
    │
    ▼
7. 如需 Inner API，加载 libcontact_cj.so
```

### 动态库依赖链

```
libcontact.so
    ├── libace_napi.so
    ├── libhilog.so
    ├── libaccesstoken_sdk.so
    ├── libdatashare_consumer.so
    ├── libnative_rdb.so
    └── libsamgr_proxy.so

libcontact_cj.so
    ├── libace_napi.so
    ├── libhilog.so
    ├── libaccesstoken_sdk.so
    ├── libcj_data_share_predicates_ffi.so
    ├── libipc_core.so
    └── libability_manager.so

libcontactsdataability.so
    ├── libhilog.so
    ├── libaccesstoken_sdk.so
    ├── libdatashare_provider.so
    ├── libnative_rdb.so
    ├── libapp_manager.so
    └── libos_account_innerkits.so
```

## 构建命令

### 完整构建

```bash
# 构建整个 contacts_data 子系统
hb build -p contacts_data

# 或使用 GN 直接构建
python3 build.py --product <product_name> --build-type release
```

### 模块构建

```bash
# 仅构建 contacts (N-API)
gn build out/.../:contact

# 仅构建 contactsCJ (Inner API)
gn build out/.../:contact_cj

# 仅构建 contactsdataability (SA 服务)
gn build out/.../:contactsdataability

# 构建 DataAbility HAP
gn build out/.../:Contacts_DataAbility
```

### 清理构建

```bash
# 清理 contacts_data 构建产物
hb build -p contacts_data --clean

# 或使用 Ninja 清理
ninja -C out/.../ clean
```

## 相关文档

| 文档 | 描述 | 链接 |
|------|------|------|
| 项目概览 | 了解项目定位和能力 | [00_Overview.md](00_Overview.md) |
| 目录结构 | 了解模块划分 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | 理解整体架构 | [02_Architecture.md](02_Architecture.md) |
| API 参考 | 学习接口使用 | [03_API_Reference.md](03_API_Reference.md) |
| 安全评审 | 了解安全考量 | [05_Security_Review.md](05_Security_Review.md) |
