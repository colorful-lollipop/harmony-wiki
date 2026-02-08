# contacts_data API 参考

> 本文档详细说明 contacts_data 子系统对外提供的 N-API 接口和 Inner API 接口。

## N-API 接口概述

contacts_data 子系统通过 N-API 对外提供 JavaScript 接口，模块名为 `@ohos.contacts.data`。开发者可以通过 import 语句导入该模块，然后调用提供的 API 进行联系人数据的增删改查操作。

N-API 接口遵循 OpenHarmony N-API 规范，支持异步操作模式（Promise 和 Callback 两种方式）。所有 API 都需要在获得相应权限后才能调用，否则会抛出安全异常。

### 模块导入

```javascript
// 导入 contacts_data 模块
import contacts from '@ohos.contacts.data';

// 或者使用 dataShare 模块访问
import dataShare from '@ohos.data.dataShare';
```

### 权限声明

使用 contacts_data API 需要在应用的 config.json 中声明相应权限：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.READ_CONTACTS",
        "reason": "读取联系人数据",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.WRITE_CONTACTS",
        "reason": "写入联系人数据",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

## N-API 清单

### 联系人操作接口

| JS API | 功能 | 同步/异步 | 对应 C++ 实现 |
|--------|------|-----------|---------------|
| addContact() | 添加联系人 | 异步 | ContactAdd |
| deleteContact() | 删除联系人 | 异步 | ContactDelete |
| UpdateContact() | 更新联系人 | 异步 | ContactUpdate |
| queryContacts() | 查询联系人 | 异步 | ContactQuery |
| mergeContacts() | 合并联系人 | 异步 | ContactMerge |

### 通话记录操作接口

| JS API | 功能 | 同步/异步 | 对应 C++ 实现 |
|--------|------|-----------|---------------|
| addCallLog() | 添加通话记录 | 异步 | CallLogAdd |
| deleteCallLog() | 删除通话记录 | 异步 | CallLogDelete |
| updateCallLog() | 更新通话记录 | 异步 | CallLogUpdate |
| queryCallLogs() | 查询通话记录 | 异步 | CallLogQuery |

### 语音信箱操作接口

| JS API | 功能 | 同步/异步 | 对应 C++ 实现 |
|--------|------|-----------|---------------|
| addVoicemail() | 添加语音信箱 | 异步 | VoicemailAdd |
| deleteVoicemail() | 删除语音信箱 | 异步 | VoicemailDelete |
| queryVoicemails() | 查询语音信箱 | 异步 | VoicemailQuery |

## N-API 详细说明

### addContact - 添加联系人

**功能描述**：向联系人数据库添加新的联系人记录。

**注册位置**：`contacts/src/napi_contact.cpp`

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| contact | Contact | 是 | 要添加的联系人对象 |

**Contact 对象结构**：

```javascript
{
    displayName: string,           // 显示名称
    phoneNumbers: [{               // 电话号码数组
        label: string,             // 标签（如：mobile, work, home）
        number: string             // 电话号码
    }],
    emails: [{                     // 邮箱数组
        label: string,             // 标签
        email: string              // 邮箱地址
    }],
    addresses: [{                  // 地址数组
        label: string,             // 标签
        city: string,              // 城市
        detailAddress: string      // 详细地址
    }],
    groups: number[]               // 所属群组ID数组
}
```

**返回值**：Promise\<number\> - 返回新创建联系人的 ID

**使用示例**：

```javascript
import contacts from '@ohos.contacts.data';

async function addNewContact() {
    const contact = {
        displayName: "张三",
        phoneNumbers: [
            { label: "mobile", number: "13800138000" }
        ],
        emails: [
            { label: "work", email: "zhangsan@example.com" }
        ]
    };
    
    try {
        const contactId = await contacts.addContact(contact);
        console.info(`联系人添加成功，ID: ${contactId}`);
        return contactId;
    } catch (error) {
        console.error(`联系人添加失败: ${error.code} - ${error.message}`);
        throw error;
    }
}
```

### queryContacts - 查询联系人

**功能描述**：根据条件查询联系人数据库，返回匹配的联系人列表。

**注册位置**：`contacts/src/napi_contact.cpp`

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| predicates | ContactQueryFilter | 否 | 查询条件过滤器 |
| callback | AsyncCallback\<ResultSet\> | 否 | 回调函数（可选） |

**ContactQueryFilter 对象结构**：

```javascript
{
    displayName: string,           // 按显示名称模糊匹配
    phoneNumber: string,           // 按电话号码精确匹配
    email: string,                 // 按邮箱匹配
    groupId: number,               // 按群组ID筛选
    page: number,                  // 分页页码（从0开始）
    countPerPage: number           // 每页数量（默认50）
}
```

**返回值**：Promise\<ResultSet\> | void - 返回查询结果集

**使用示例**：

```javascript
import contacts from '@ohos.contacts.data';

async function queryContactsByName(name) {
    const filter = {
        displayName: name,
        page: 0,
        countPerPage: 20
    };
    
    try {
        const resultSet = await contacts.queryContacts(filter);
        const contactsList = [];
        
        if (resultSet.goToFirstRow()) {
            do {
                const contact = {
                    id: resultSet.getLong(resultSet.getColumnIndex("id")),
                    displayName: resultSet.getString(resultSet.getColumnIndex("display_name")),
                    phoneNumber: resultSet.getString(resultSet.getColumnIndex("phone_number"))
                };
                contactsList.push(contact);
            } while (resultSet.goToNextRow());
        }
        
        resultSet.close();
        return contactsList;
    } catch (error) {
        console.error(`查询失败: ${error.code} - ${error.message}`);
        throw error;
    }
}
```

### deleteContact - 删除联系人

**功能描述**：从联系人数据库删除指定的联系人。

**注册位置**：`contacts/src/napi_contact.cpp`

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| contactId | number | 是 | 要删除的联系人ID |
| callback | AsyncCallback\<number\> | 否 | 回调函数（可选） |

**返回值**：Promise\<number\> - 返回操作结果码（0表示成功）

**使用示例**：

```javascript
import contacts from '@ohos.contacts.data';

async function deleteContactById(contactId) {
    try {
        const resultCode = await contacts.deleteContact(contactId);
        if (resultCode === 0) {
            console.info(`联系人 ${contactId} 删除成功`);
        } else {
            console.warn(`联系人 ${contactId} 删除失败，错误码: ${resultCode}`);
        }
        return resultCode;
    } catch (error) {
        console.error(`删除操作异常: ${error.message}`);
        throw error;
    }
}
```

### updateContact - 更新联系人

**功能描述**：更新已有联系人的信息。

**注册位置**：`contacts/src/napi_contact.cpp`

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| contact | Contact | 是 | 更新后的联系人对象（需包含id） |
| callback | AsyncCallback\<number\> | 否 | 回调函数（可选） |

**Contact 对象结构（含ID）**：

```javascript
{
    id: number,                    // 联系人ID（必填）
    displayName: string,           // 显示名称
    phoneNumbers: [{               // 电话号码数组
        id: number,                // 号码ID（更新时可选）
        label: string,
        number: string
    }],
    // ... 其他字段
}
```

**返回值**：Promise\<number\> - 返回操作结果码（0表示成功）

**使用示例**：

```javascript
import contacts from '@ohos.contacts.data';

async function updateContact(contact) {
    try {
        const resultCode = await contacts.updateContact(contact);
        if (resultCode === 0) {
            console.info(`联系人 ${contact.id} 更新成功`);
        } else {
            console.warn(`联系人 ${contact.id} 更新失败，错误码: ${resultCode}`);
        }
        return resultCode;
    } catch (error) {
        console.error(`更新操作异常: ${error.message}`);
        throw error;
    }
}
```

### mergeContacts - 合并联系人

**功能描述**：将两个联系人合并为一个，保留双方的最完整信息。

**注册位置**：`contacts/src/napi_contact.cpp`

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| contactId1 | number | 是 | 第一个联系人ID |
| contactId2 | number | 是 | 第二个联系人ID |
| strategy | MergeStrategy | 否 | 合并策略 |
| callback | AsyncCallback\<MergeResult\> | 否 | 回调函数（可选） |

**MergeStrategy 对象结构**：

```javascript
{
    preferFirst: boolean,          // 优先保留第一个联系人（默认true）
    mergePhones: boolean,          // 合并电话号码（默认true）
    mergeEmails: boolean,          // 合并邮箱地址（默认true）
    mergeAddresses: boolean        // 合并地址信息（默认true）
}
```

**返回值**：Promise\<MergeResult\> - 返回合并结果

**MergeResult 对象结构**：

```javascript
{
    success: boolean,              // 是否成功
    mergedContactId: number,       // 合并后的联系人ID
    deletedContactId: number,      // 被删除的联系人ID
    conflicts: [{                  // 字段冲突列表
        field: string,             // 字段名
        value1: string,            // 第一个联系人的值
        value2: string,            // 第二个联系人的值
        resolvedValue: string      // 最终保留的值
    }]
}
```

## Inner API 概述

Inner API 是 contacts_data 子系统对内部提供的 C++ 接口，主要供系统服务和其他子系统使用。Inner API 以 C++ 类和函数的形式提供，性能更高但使用门槛也更高。

### 头文件位置

| 头文件 | 路径 | 说明 |
|--------|------|------|
| contact_cj.h | contactsCJ/include/contact_cj.h | 主要 Inner API 头文件 |
| contact.h | contactsCJ/include/contact.h | 辅助定义头文件 |

### 命名空间

所有 Inner API 位于 `namespace OHOS::Contacts` 下。

### 主要类

| 类名 | 功能 | 头文件 |
|------|------|--------|
| ContactCj | 联系人管理主类 | contact_cj.h |
| CallLogCj | 通话记录管理类 | contact_cj.h |
| VoicemailCj | 语音信箱管理类 | contact_cj.h |
| DataAbilityHelper | DataAbility 辅助类 | contact_cj.h |

## Inner API 清单

### ContactCj 类

| 方法名 | 功能 | 返回类型 |
|--------|------|----------|
| AddContact() | 添加联系人 | int |
| DeleteContact() | 删除联系人 | int |
| UpdateContact() | 更新联系人 | int |
| QueryContact() | 查询单个联系人 | std::shared_ptr\<Contact\> |
| QueryContacts() | 查询联系人列表 | std::vector\<std::shared_ptr\<Contact\>\> |
| MergeContact() | 合并联系人 | int |
| AddGroup() | 添加群组 | int |
| DeleteGroup() | 删除群组 | int |
| UpdateGroup() | 更新群组 | int |
| QueryGroups() | 查询群组列表 | std::vector\<std::shared_ptr\<Group\>\> |

### CallLogCj 类

| 方法名 | 功能 | 返回类型 |
|--------|------|----------|
| AddCallLog() | 添加通话记录 | int |
| DeleteCallLog() | 删除通话记录 | int |
| UpdateCallLog() | 更新通话记录 | int |
| QueryCallLog() | 查询单个通话记录 | std::shared_ptr\<CallLog\> |
| QueryCallLogs() | 查询通话记录列表 | std::vector\<std::shared_ptr\<CallLog\>\> |

### VoicemailCj 类

| 方法名 | 功能 | 返回类型 |
|--------|------|----------|
| AddVoicemail() | 添加语音信箱 | int |
| DeleteVoicemail() | 删除语音信箱 | int |
| QueryVoicemail() | 查询语音信箱 | std::shared_ptr\<Voicemail\> |
| QueryVoicemails() | 查询语音信箱列表 | std::vector\<std::shared_ptr\<Voicemail\>\> |

## Inner API 详细说明

### ContactCj::AddContact

**功能描述**：向联系人数据库添加新的联系人。

**注册位置**：`contactsCJ/src/contact_cj.cpp`

**函数签名**：

```cpp
namespace OHOS::Contacts {

class ContactCj {
public:
    int AddContact(const std::shared_ptr<Contact>& contact);
};

} // namespace OHOS::Contacts
```

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| contact | std::shared_ptr\<Contact\> | 是 | 联系人对象指针 |

**Contact 结构定义**：

```cpp
struct Contact {
    int64_t id;
    std::string displayName;
    std::vector<std::shared_ptr<PhoneNumber>> phoneNumbers;
    std::vector<std::shared_ptr<Email>> emails;
    std::vector<std::shared_ptr<Address>> addresses;
    std::vector<int64_t> groupIds;
    // ... 其他字段
};
```

**返回值**：int - 返回新创建联系人的 ID（大于0表示成功，小于0表示失败）

**实现逻辑**：

1. 权限校验：检查调用者是否具有 WRITE_CONTACTS 权限
2. 参数校验：检查联系人对象的必填字段
3. 开启事务：开始数据库事务
4. 插入 raw_contact：插入原始联系人记录
5. 插入 contact_data：插入关联的联系人详情记录
6. 提交事务：提交事务或回滚

**使用示例**：

```cpp
#include "contact_cj.h"

using namespace OHOS::Contacts;

int AddContactExample() {
    // 创建联系人对象
    auto contact = std::make_shared<Contact>();
    contact->displayName = "李四";
    
    auto phone = std::make_shared<PhoneNumber>();
    phone->number = "13900139000";
    phone->label = "mobile";
    contact->phoneNumbers.push_back(phone);
    
    // 调用 Inner API
    ContactCj contactCj;
    int64_t contactId = contactCj.AddContact(contact);
    
    if (contactId > 0) {
        // 成功
        return 0;
    } else {
        // 失败
        return -1;
    }
}
```

### ContactCj::QueryContacts

**功能描述**：根据条件查询联系人列表。

**注册位置**：`contactsCJ/src/contact_cj.cpp`

**函数签名**：

```cpp
namespace OHOS::Contacts {

class ContactCj {
public:
    std::vector<std::shared_ptr<Contact>> QueryContacts(
        const std::shared_ptr<ContactQuery>& query);
};

} // namespace OHOS::Contacts
```

**参数说明**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| query | std::shared_ptr\<ContactQuery\> | 是 | 查询条件对象 |

**ContactQuery 结构定义**：

```cpp
struct ContactQuery {
    std::string displayName;       // 显示名称模糊匹配
    std::string phoneNumber;       // 电话号码匹配
    std::string email;             // 邮箱匹配
    int64_t groupId;               // 群组ID
    int32_t page;                  // 页码
    int32_t pageSize;              // 每页数量
    std::string orderBy;           // 排序字段
    bool ascending;                // 是否升序
};
```

**返回值**：std::vector\<std::shared_ptr\<Contact\>\> - 返回匹配的联系人列表

**使用示例**：

```cpp
#include "contact_cj.h"

using namespace OHOS::Contacts;

void QueryContactsExample() {
    auto query = std::make_shared<ContactQuery>();
    query->displayName = "张";
    query->page = 0;
    query->pageSize = 20;
    query->orderBy = "display_name";
    query->ascending = true;
    
    ContactCj contactCj;
    auto contacts = contactCj.QueryContacts(query);
    
    for (const auto& contact : contacts) {
        // 处理每个联系人
    }
}
```

## 错误码说明

contacts_data 子系统使用以下错误码：

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | ERR_SUCCESS | 操作成功 |
| -1 | ERR_GENERIC | 通用错误 |
| -2 | ERR_INVALID_PARAM | 参数无效 |
| -3 | ERR_NO_PERMISSION | 无权限 |
| -4 | ERR_NOT_FOUND | 记录不存在 |
| -5 | ERR_DUPLICATED | 记录已存在 |
| -6 | ERR_DATABASE | 数据库错误 |
| -7 | ERR_TRANSACTION | 事务错误 |

## 相关文档

| 文档 | 描述 | 链接 |
|------|------|------|
| 项目概览 | 了解项目定位和能力 | [00_Overview.md](00_Overview.md) |
| 目录结构 | 了解模块划分 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | 理解整体架构 | [02_Architecture.md](02_Architecture.md) |
| 构建文档 | 了解编译配置 | [04_Build.md](04_Build.md) |
| 安全评审 | 了解安全考量 | [05_Security_Review.md](05_Security_Review.md) |
