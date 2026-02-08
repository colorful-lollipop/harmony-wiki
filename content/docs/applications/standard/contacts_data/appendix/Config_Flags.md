# contacts_data 配置参数

> 本文档记录 contacts_data 子系统中的关键编译宏、feature flags 和运行时配置参数。

## 编译期配置

### BUILD.gn 中的宏定义

contacts_data 子系统使用 BUILD.gn 文件中的 `defines` 配置来定义编译期宏。这些宏影响代码的编译行为和日志输出。

#### 日志相关宏

| 宏名称 | 定义位置 | 值 | 用途 |
|--------|----------|-----|------|
| CONTACTSDATA_LOG_TAG | contacts/BUILD.gn | "ContactsApi" | N-API 层日志标签 |
| CONTACTSDATA_LOG_TAG | contactsCJ/BUILD.gn | "ContactsFfi" | Inner API 层日志标签 |
| CONTACTSDATA_LOG_TAG | BUILD.gn | "ContactsData" | SA 服务日志标签 |
| LOG_DOMAIN | BUILD.gn | 0xD001F09 | 日志域标识 |

**使用示例**：

```cpp
// 使用日志宏输出日志
CONTACTS_LOGI("Add contact success, id=%{public}d", contactId);
CONTACTS_LOGE("Failed to query contacts, error=%{public}d", errorCode);
CONTACTS_LOGW("Permission check skipped for system app");
```

#### 编译器安全选项

| 配置项 | 定义位置 | 值 | 用途 |
|--------|----------|-----|------|
| sanitize.cfi | contacts/BUILD.gn | true | 启用控制流完整性检查 |
| sanitize.cfi_cross_dso | contacts/BUILD.gn | true | 跨 DSO 的 CFI 检查 |
| sanitize.debug | contacts/BUILD.gn | false | 关闭调试模式 |
| sanitize.cfi | contactsCJ/BUILD.gn | true | 启用控制流完整性检查 |
| sanitize.cfi_cross_dso | contactsCJ/BUILD.gn | true | 跨 DSO 的 CFI 检查 |
| sanitize.debug | contactsCJ/BUILD.gn | false | 关闭调试模式 |

### contact.gni 中的变量

`contact.gni` 文件定义了 contacts_data 子系统的共享构建变量。

```gni
# 根路径变量
CONTACT_ROOT = "//applications/standard/contacts_data"
```

### 组件配置 (bundle.json)

bundle.json 文件定义了 contacts_data 子系统的组件级配置。

#### 系统能力

| 配置项 | 值 | 说明 |
|--------|-----|------|
| name | "contacts_data" | 组件名称 |
| subsystem | "applications" | 所属子系统 |
| syscap | ["SystemCapability.Applications.ContactsData"] | 系统能力标识 |

#### 资源占用

| 配置项 | 值 | 说明 |
|--------|-----|------|
| rom | "400KB" | ROM 占用估算 |
| ram | "7MB" | RAM 占用估算 |

#### 依赖组件

| 组件名称 | 用途 |
|----------|------|
| ability_base | 基础能力框架 |
| ability_runtime | 运行时支持 |
| access_token | 权限管理 |
| bundle_framework | 应用框架 |
| common_event_service | 公共事件服务 |
| c_utils | C 工具库 |
| data_share | 数据共享 |
| eventhandler | 事件处理 |
| hilog | 日志系统 |
| ipc | 进程间通信 |
| napi | Node.js API |
| os_account | 账户管理 |
| preferences | 首选项存储 |
| relational_store | 关系型数据库 |
| safwk | 系统能力框架 |
| samgr | 服务管理框架 |
| ace_engine | ACE 引擎 |
| jsoncpp | JSON 处理 |
| node | Node 运行时 |

## 运行时配置

### DataShare URI 配置

contacts_data 子系统定义了多个 DataShare URI 用于访问不同的数据表。

#### 联系人数据 URI

| URI | 数据表 | 用途 |
|-----|--------|------|
| datashare:///com.ohos.contactsdataability/contacts/contact | contact | 联系人视图 |
| datashare:///com.ohos.contactsdataability/contacts/raw_contact | raw_contact | 原始联系人 |
| datashare:///com.ohos.contactsdataability/contacts/contact_data | contact_data | 联系人详情 |
| datashare:///com.ohos.contactsdataability/contacts/groups | groups | 联系人分组 |
| datashare:///com.ohos.contactsdataability/contacts/photo_files | photo_files | 头像文件 |
| datashare:///com.ohos.contactsdataability/contacts/contact_blocklist | blocklist | 黑名单 |
| datashare:///com.ohos.contactsdataability/contacts/search_contact | search_contact | 搜索索引 |

#### 通话记录 URI

| URI | 数据表 | 用途 |
|-----|--------|------|
| datashare:///com.ohos.calllogability/calls/calllog | calllog | 通话记录 |

#### 语音信箱 URI

| URI | 数据表 | 用途 |
|-----|--------|------|
| datashare:///com.ohos.voicemailability/calls/voicemail | voicemail | 语音信箱 |

### 权限配置

contacts_data 子系统涉及以下权限：

| 权限名称 | 权限级别 | 用途 |
|----------|----------|------|
| ohos.permission.READ_CONTACTS | normal | 读取联系人 |
| ohos.permission.WRITE_CONTACTS | normal | 写入联系人 |
| ohos.permission.GET_TELEPHONY_STATE | system_basic | 获取电话状态 |
| ohos.permission.READ_CALL_LOG | normal | 读取通话记录 |
| ohos.permission.WRITE_CALL_LOG | normal | 写入通话记录 |

### 账户配置

contacts_data 子系统支持多账户场景，通过账户 ID 实现数据隔离。

| 配置项 | 说明 |
|--------|------|
| 主账户 ID | 0 或当前活跃用户 ID |
| 配置账户 | 支持多用户数据隔离 |
| 同步策略 | 账户切换时自动同步/隔离数据 |

## Feature Flags

### 联系人合并功能

| Feature Flag | 默认值 | 用途 |
|--------------|--------|------|
| 手动合并 | 启用 | 允许用户手动选择合并重复联系人 |
| 自动合并 | 启用 | 自动合并高度相似的联系人 |
| 合并预览 | 启用 | 显示合并预览界面 |

### 快速搜索功能

| Feature Flag | 默认值 | 用途 |
|--------------|--------|------|
| 拼音搜索 | 启用 | 支持按拼音首字母搜索 |
| 模糊搜索 | 启用 | 支持模糊匹配搜索 |
| 索引构建 | 启用 | 构建搜索索引加速查询 |

### 数据恢复功能

| Feature Flag | 默认值 | 用途 |
|--------------|--------|------|
| 自动恢复 | 启用 | 检测数据库损坏并自动恢复 |
| 备份同步 | 启用 | 自动同步联系人备份 |
| 损坏检测 | 启用 | 定期检测数据库完整性 |

## 数据库配置

### 数据库文件路径

| 路径 | 说明 |
|------|------|
| /data/contacts/contacts.db | 联系人数据库 |
| /data/contacts/calllog.db | 通话记录数据库 |
| /data/contacts/voicemail.db | 语音信箱数据库 |
| /data/contacts/search_index.db | 搜索索引数据库 |

### 数据库版本

| 数据库 | 当前版本 | 说明 |
|--------|----------|------|
| contacts.db | [待确认] | 联系人数据库版本 |
| calllog.db | [待确认] | 通话记录数据库版本 |
| voicemail.db | [待确认] | 语音信箱数据库版本 |

### 数据库表结构配置

#### raw_contact 表

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INTEGER | 主键，自增 |
| account_id | INTEGER | 账户 ID |
| display_name | TEXT | 显示名称 |
| photo_file_id | INTEGER | 头像文件 ID |
| last_time_contacted | INTEGER | 最后联系时间 |
| starred | INTEGER | 是否收藏 |
| is_deleted | INTEGER | 是否已删除 |
| create_time | TEXT | 创建时间 |
| modify_time | TEXT | 修改时间 |

#### contact_data 表

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INTEGER | 主键，自增 |
| raw_contact_id | INTEGER | 外键，关联 raw_contact |
| content_type | TEXT | 内容类型（phone/email/address 等） |
| detail_info | TEXT | 详细信息 |
| position | INTEGER | 排序位置 |
| is_primary | INTEGER | 是否主要 |
| is_deleted | INTEGER | 是否已删除 |

#### calllog 表

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INTEGER | 主键，自增 |
| phone_number | TEXT | 电话号码 |
| display_name | TEXT | 显示名称 |
| call_type | INTEGER | 通话类型（来电/去电/未接） |
| call_direction | INTEGER | 通话方向 |
| sim_id | INTEGER | SIM 卡 ID |
| call_status | INTEGER | 通话状态 |
| start_time | INTEGER | 开始时间 |
| duration | INTEGER | 通话时长 |
| is_new | INTEGER | 是否新建 |
| is_deleted | INTEGER | 是否已删除 |

## HAP 配置

### module.json 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| moduleName | "Contacts_DataAbility" | 模块名称 |
| moduleType | "entry" | 模块类型 |
| ability | Contacts_DataAbility | 入口 Ability |
| type | "data" | Ability 类型（DataAbility） |

### 安装路径

| 配置项 | 值 | 说明 |
|--------|-----|------|
| module_install_dir | "app/com.ohos.contactsdataability" | HAP 安装目录 |

## 日志级别配置

contacts_data 子系统支持以下日志级别：

| 日志级别 | 值 | 用途 |
|----------|-----|------|
| LOG_DEBUG | 0 | 调试信息 |
| LOG_INFO | 1 | 普通信息 |
| LOG_WARN | 2 | 警告信息 |
| LOG_ERROR | 3 | 错误信息 |
| LOG_FATAL | 4 | 致命错误 |

## 性能配置

### 分页配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| 默认每页数量 | 50 | 查询时的默认分页大小 |
| 最大每页数量 | 500 | 单次查询的最大记录数 |
| 预加载数量 | 10 | 滚动到边缘时预加载的数量 |

### 索引配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| 拼音索引 | 启用 | 按拼音首字母建立索引 |
| 姓名索引 | 启用 | 按姓名建立索引 |
| 电话索引 | 启用 | 按电话号码建立索引 |

## 相关文档

| 文档 | 描述 | 链接 |
|------|------|------|
| 项目概览 | 了解项目定位和能力 | [00_Overview.md](../00_Overview.md) |
| 目录结构 | 了解模块划分 | [01_Directory_Structure.md](../01_Directory_Structure.md) |
| 架构设计 | 理解整体架构 | [02_Architecture.md](../02_Architecture.md) |
| API 参考 | 学习接口使用 | [03_API_Reference.md](../03_API_Reference.md) |
| 构建文档 | 了解编译配置 | [04_Build.md](../04_Build.md) |
| 安全评审 | 了解安全考量 | [05_Security_Review.md](../05_Security_Review.md) |
