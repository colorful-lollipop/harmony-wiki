# contacts_data 项目概览

> 本文档介绍 contacts_data 子系统的定位、核心能力、运行环境和关键概念。

## 项目定位

contacts_data 是 OpenHarmony 系统中的应用层联系人数据库子系统，属于 `applications` 子系统的一部分。该子系统负责管理用户联系人、通话记录和语音信箱等核心数据，为上层应用提供统一的数据访问接口。

**系统能力标识**：`SystemCapability.Applications.ContactsData`

**项目路径**：`//applications/standard/contacts_data`

**主要职责**：

- 提供联系人数据的增删改查（CRUD）能力
- 管理通话记录和语音信箱数据
- 支持联系人搜索和快速检索
- 实现联系人合并和去重功能
- 提供账户隔离的数据访问机制

## 核心能力

### 数据管理能力

contacts_data 子系统提供三大核心数据管理能力，每种数据类型对应不同的 URI 路径和操作接口：

**联系人数据**（Contacts）：存储用户的联系人信息，包括姓名、电话号码、邮箱、头像等。支持多账户隔离存储，提供联系人分组、收藏、黑白名单等功能。联系人数据存储在 SQLite 关系型数据库中，支持模糊搜索和拼音快速检索。

**通话记录**（CallLog）：记录用户的通话历史，包括来电、去电和未接来电。支持按时间、电话号码、姓名等条件筛选，提供通话时长统计和通话频率分析功能。通话记录与联系人数据关联显示，来电号码可自动匹配联系人信息。

**语音信箱**（Voicemail）：存储用户的语音留言，支持录音上传、播放和删除。语音信箱与通话记录关联，当用户无法接听时可以将来电转为语音留言。

### N-API 接口能力

contacts_data 提供两类 API 接口供开发者使用：

**N-API 接口**（contacts 模块）：面向 JavaScript 开发者的原生 API，通过 `@ohos.contacts.data` 模块提供。开发者可以使用熟悉的 JavaScript 语法访问联系人数据，无需了解底层实现细节。

**Inner API 接口**（contactsCJ 模块）：面向系统内部开发的 C++ 接口，提供更精细的控制和更高的性能。Inner API 主要供系统服务和其他子系统使用，不直接暴露给应用开发者。

### 高级功能能力

除了基础的数据管理，contacts_data 还提供以下高级功能：

**联系人合并**：自动检测重复联系人并提供合并建议，支持手动合并和自动合并两种模式。合并时会智能处理字段冲突，保留信息最完整的字段值。

**数据损坏恢复**：内置数据完整性检查和损坏恢复机制，当检测到数据库异常时自动尝试修复，最大限度减少数据丢失风险。

**汉字转拼音**：内置汉字到拼音的转换功能，支持多音字处理和姓氏简拼，可用于联系人快速检索和排序。

**账户管理**：支持多账户系统，每个用户账户拥有独立的联系人数据。系统自动管理账户切换时的数据隔离，确保用户隐私安全。

## 运行环境

### 系统要求

contacts_data 子系统运行在 OpenHarmony 标准系统上，需要以下系统能力支持：

| 依赖组件 | 用途说明 | 最低版本 |
|----------|----------|----------|
| ability_base | 基础能力框架 | 3.1.0 |
| ability_runtime | 运行时支持 | 3.1.0 |
| data_share | 数据共享机制 | 3.1.0 |
| relational_store | SQLite 数据库 | 3.1.0 |
| ipc | 进程间通信 | 3.1.0 |
| napi | Node.js API | 3.1.0 |

### 资源占用

根据 bundle.json 中的配置，contacts_data 子系统的资源占用如下：

| 资源类型 | 占用大小 | 说明 |
|----------|----------|------|
| ROM | 400KB | 编译后二进制文件大小 |
| RAM | 7MB | 运行时内存占用 |
| 存储空间 | 视数据量 | 联系人数据库文件 |

### 部署位置

contacts_data 子系统编译后部署在系统分区的以下位置：

| 产物类型 | 目标路径 | 说明 |
|----------|----------|------|
| N-API 库 | /system/lib/libcontact.so | 联系人 N-API 实现 |
| Inner API 库 | /system/lib/libcontact_cj.so | C++ 接口实现 |
| SA 服务 | /system/bin/contactsdataability | 数据能力服务 |
| 数据库文件 | /data/contacts/ | 联系人数据存储目录 |

## 关键概念

### DataShare URI

contacts_data 使用 DataShare 机制提供数据访问，所有数据操作通过 URI 进行。URI 采用 `datashare://` 协议头，后跟应用包名和资源路径：

```javascript
// 联系人数据 URI
const contactsUri = "datashare:///com.ohos.contactsdataability/contacts/raw_contact";

// 通话记录 URI
const calllogUri = "datashare:///com.ohos.calllogability/calls/calllog";

// 语音信箱 URI
const voicemailUri = "datashare:///com.ohos.voicemailability/calls/voicemail";
```

URI 格式说明：`datashare://` + 服务名 + 数据表路径。开发者需要根据操作的数据类型选择正确的 URI。

### ValuesBucket

ValuesBucket 是 OpenHarmony 数据管理中使用的键值对结构，用于传递要插入或更新的数据：

```javascript
const contactData = {
    "display_name": "张三",
    "phone_number": "13800138000",
    "email": "zhangsan@example.com"
};
```

ValuesBucket 支持的数据类型包括：字符串、数字、布尔值、数组等，不同数据类型对应数据库中的不同列类型。

### DataSharePredicates

DataSharePredicates 用于构建查询条件，支持丰富的条件表达式：

```javascript
const predicates = new dataShare.DataSharePredicates();
predicates.equalTo("id", "12345");
predicates.greaterThan("create_time", "2024-01-01");
predicates.orderByAsc("display_name");
```

支持的条件操作包括：equalTo、notEqualTo、greaterThan、lessThan、like、in、between 等。

### ResultSet

ResultSet 是查询操作返回的结果集类型，支持游标式数据遍历：

```javascript
const resultSet = dataShareHelper.query(uri, predicates, columns);
if (resultSet.goToFirstRow()) {
    do {
        const name = resultSet.getString(resultSet.getColumnIndex("display_name"));
        const phone = resultSet.getString(resultSet.getColumnIndex("phone_number"));
    } while (resultSet.goToNextRow());
}
resultSet.close();
```

ResultSet 提供了丰富的数据访问方法，包括 getString、getLong、getDouble、getBlob 等。

## 相关文档

| 文档 | 描述 | 链接 |
|------|------|------|
| 目录结构 | 了解模块划分 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | 理解整体架构 | [02_Architecture.md](02_Architecture.md) |
| API 参考 | 学习接口使用 | [03_API_Reference.md](03_API_Reference.md) |
| 构建文档 | 了解编译流程 | [04_Build.md](04_Build.md) |
| 安全评审 | 了解安全考量 | [05_Security_Review.md](05_Security_Review.md) |
