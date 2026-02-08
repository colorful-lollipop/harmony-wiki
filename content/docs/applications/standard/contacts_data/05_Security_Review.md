# contacts_data 安全评审

> 本文档对 contacts_data 子系统进行全面的安全风险分析，涵盖攻击面识别、信任边界分析和潜在安全风险评估。所有风险点均基于代码证据进行分析。

## 安全模型概述

contacts_data 子系统处理用户的敏感个人信息，包括联系人、通话记录和语音信箱等数据。根据 OpenHarmony 的安全架构设计，contacts_data 位于应用层，需要与系统的权限管理、数据隔离等安全机制协同工作。

### 资产分类

contacts_data 子系统管理的敏感资产包括：

| 资产类型 | 敏感级别 | 说明 |
|----------|----------|------|
| 联系人姓名 | 高 | 用户社会关系信息 |
| 电话号码 | 高 | 用户联系方式 |
| 邮箱地址 | 高 | 用户联系方式 |
| 通话记录 | 高 | 用户通信历史 |
| 语音信箱 | 高 | 用户语音留言 |
| 头像图片 | 中 | 用户个人形象 |
| 群组信息 | 中 | 用户社交关系 |

## 攻击面分析

contacts_data 子系统的攻击面主要包括以下几个方面：

### N-API 攻击面

N-API 是 contacts_data 对外暴露的主要接口，所有 JavaScript 应用通过 N-API 访问联系人数据。

**攻击向量**：

1. **参数注入**：通过构造恶意参数值尝试注入 SQL 语句或系统命令
2. **类型混淆**：通过传递非预期类型的参数触发类型处理漏洞
3. **资源耗尽**：通过大量请求耗尽系统资源
4. **权限绕过**：尝试绕过权限检查直接访问数据

**相关代码**：`contacts/src/contacts_napi_utils.cpp`（N-API 参数处理）

### DataShare 攻击面

contacts_data 使用 DataShare 机制提供数据共享服务，这也是一个重要的攻击面。

**攻击向量**：

1. **URI 篡改**：修改 DataShare URI 尝试访问未授权数据
2. **Predicates 注入**：在 DataSharePredicates 中注入恶意条件
3. **跨应用数据访问**：尝试访问其他应用的联系人数据

**相关代码**：`dataBusiness/contacts/src/contacts_datashare_stub_impl.cpp`（DataShare 实现）

### IPC 攻击面

contactsdataability 以 System Ability 形式运行，通过 IPC 与客户端通信。

**攻击向量**：

1. **消息伪造**：构造伪造的 IPC 消息
2. **调用权限验证不足**：IPC 调用可能绕过权限检查
3. **序列化/反序列化漏洞**：IPC 数据序列化过程中的漏洞

**相关代码**：`ability/common/utils/src/uri_utils.cpp`（URI 处理）

### 数据库攻击面

联系人数据存储在 SQLite 数据库中，数据库操作也是潜在的攻击面。

**攻击向量**：

1. **SQL 注入**：通过用户输入构造恶意 SQL 语句
2. **数据库文件篡改**：直接修改数据库文件
3. **竞态条件**：并发操作导致的数据不一致

**相关代码**：`dataBusiness/contacts/src/contacts_database.cpp`（数据库操作）

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界图                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    不可信区域                              │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │  │
│  │  │第三方应用│  │未知来源 │  │网络攻击 │  │恶意输入数据 │  │  │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └──────┬──────┘  │  │
│  └────────┼───────────┼───────────┼───────────────┼─────────┘  │
│           │           │           │               │            │
│           ▼           ▼           ▼               ▼            │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    边界检查点                              │  │
│  │  • 参数校验（类型/长度/范围/格式）                          │  │
│  │  • 权限验证（ohos.permission.READ_CONTACTS）               │  │
│  │  • URI 验证（数据表路径校验）                               │  │
│  │  • Predicates 验证（SQL 注入防护）                         │  │
│  └───────────────────────────┬───────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    可信区域                                │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              contactsdataability 服务               │  │  │
│  │  │  • Inner API 层                                     │  │  │
│  │  │  • dataBusiness 层                                  │  │  │
│  │  │  • ability 层                                       │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                              │                             │  │
│  │                              ▼                             │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              relational_store (SQLite)              │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 边界说明

**不可信区域**：第三方应用、未知来源的输入数据、网络攻击等均被视为不可信来源。所有来自不可信区域的数据都必须经过严格的边界检查。

**边界检查点**：
- **参数校验**：检查所有输入参数的类型、长度、范围和格式
- **权限验证**：检查调用者是否具有相应权限
- **URI 验证**：验证 DataShare URI 的合法性
- **Predicates 验证**：验证查询条件，防止 SQL 注入

**可信区域**：contactsdataability 服务及其内部模块运行在可信环境中，可以安全地执行数据库操作和业务逻辑。

## 安全风险清单

### 风险 1：N-API 参数校验不完整

**风险级别**：中

**证据位置**：`contacts/src/contacts_napi_utils.cpp`

**问题描述**：N-API 接口层对某些参数的校验不够严格，可能允许非法数据进入后续处理流程。

**触发条件**：

```javascript
// 构造超长字符串参数
import contacts from '@ohos.contacts.data';

const maliciousContact = {
    displayName: "A".repeat(100000),  // 超长字符串
    phoneNumbers: [{
        number: "12345678901".repeat(1000)  // 超长电话号码
    }]
};

contacts.addContact(maliciousContact);
```

**影响**：

1. 可能导致内存分配异常（拒绝服务）
2. 可能触发缓冲区溢出漏洞
3. 可能导致数据库存储异常

**修复建议**：

```cpp
// 在 contacts_napi_utils.cpp 中添加参数长度校验
constexpr size_t MAX_DISPLAY_NAME_LENGTH = 256;
constexpr size_t MAX_PHONE_NUMBER_LENGTH = 32;

bool ValidateContactParam(const Contact& contact) {
    if (contact.displayName.length() > MAX_DISPLAY_NAME_LENGTH) {
        CONTACTS_LOGE("Display name too long: %{public}zu", 
            contact.displayName.length());
        return false;
    }
    
    for (const auto& phone : contact.phoneNumbers) {
        if (phone.number.length() > MAX_PHONE_NUMBER_LENGTH) {
            CONTACTS_LOGE("Phone number too long: %{public}zu",
                phone.number.length());
            return false;
        }
    }
    
    return true;
}
```

### 风险 2：DataShare Predicates 可能存在 SQL 注入

**风险级别**：高

**证据位置**：`ability/common/utils/src/predicates_convert.cpp`

**问题描述**：DataSharePredicates 在转换为 SQL 语句时，如果没有正确过滤用户输入，可能导致 SQL 注入漏洞。

**触发条件**：

```javascript
import dataShare from '@ohos.data.dataShare';

const uri = "datashare:///com.ohos.contactsdataability/contacts/raw_contact";
const predicates = new dataShare.DataSharePredicates();

// 恶意构造的查询条件
predicates.equalTo("id", "1; DROP TABLE raw_contact; --");

dataShare.createDataShareHelper(uri).query(uri, predicates, ["*"]);
```

**影响**：

1. 可能导致敏感数据泄露
2. 可能导致数据库表被删除
3. 可能导致数据被篡改

**修复建议**：

```cpp
// 在 predicates_convert.cpp 中添加输入过滤
std::string SanitizeSqlString(const std::string& input) {
    std::string output;
    output.reserve(input.size());
    
    for (char c : input) {
        switch (c) {
            case '\'':
                output += "''";  // 转义单引号
                break;
            case ';':
                output += ' ';   // 移除分号
                break;
            case '-':
                if (output.size() >= 2 && output[output.size()-1] == '-') {
                    continue;    // 移除注释起始符
                }
                output += c;
                break;
            default:
                output += c;
        }
    }
    
    return output;
}
```

### 风险 3：通话记录号码未校验格式

**风险级别**：中

**证据位置**：`dataBusiness/calllog/src/calllog_database.cpp`

**问题描述**：通话记录模块在插入电话号码时没有验证号码格式，可能存储非法格式的电话号码。

**触发条件**：

```javascript
import dataShare from '@ohos.data.dataShare';

const calllogUri = "datashare:///com.ohos.calllogability/calls/calllog";
const helper = dataShare.createDataShareHelper(calllogUri);

// 插入非法格式的通话记录
const value = {
    "phone_number": "'; DROP TABLE calllog; --",
    "display_name": "Attacker"
};

helper.insert(calllogUri, value);
```

**影响**：

1. 可能导致 SQL 注入
2. 可能导致数据显示异常
3. 可能被用于钓鱼攻击

**修复建议**：

```cpp
// 在 calllog_database.cpp 中添加号码格式校验
bool ValidatePhoneNumber(const std::string& number) {
    // 只允许数字、加号、空格和连字符
    for (char c : number) {
        if (!isdigit(c) && c != '+' && c != ' ' && c != '-') {
            CONTACTS_LOGE("Invalid phone number character: %{public}c", c);
            return false;
        }
    }
    
    // 限制号码长度
    if (number.length() > MAX_PHONE_NUMBER_LENGTH) {
        CONTACTS_LOGE("Phone number too long: %{public}zu", 
            number.length());
        return false;
    }
    
    return true;
}
```

### 风险 4：多账户数据隔离可能存在边界问题

**风险级别**：中

**证据位置**：`ability/account/src/account_manager.cpp`

**问题描述**：多账户场景下，不同用户的数据隔离依赖于账户 ID 的正确传递。如果账户 ID 可被伪造，可能导致跨用户数据访问。

**触发条件**：

```cpp
// 在 contacts_data_ability.cpp 中可能存在的问题代码
int64_t ContactsDataAbility::QueryContacts(
    const std::string& uri,
    const std::vector<std::string>& columns,
    const std::shared_ptr<DataSharePredicates>& predicates) {
    
    // 获取调用者账户 ID
    int32_t callerAccountId = GetCallerAccountId();
    
    // 如果 GetCallerAccountId 返回值可被操控
    // 则可能访问其他账户的数据
    std::string accountWhere = "account_id = " + std::to_string(callerAccountId);
    predicates->And(accountWhere, "");
    
    return queryRawContacts(uri, columns, predicates);
}
```

**影响**：

1. 用户 A 可能读取用户 B 的联系人
2. 用户 A 可能修改或删除用户 B 的联系人
3. 用户隐私泄露

**修复建议**：

```cpp
// 使用系统提供的安全账户获取接口
int32_t GetSecureAccountId() {
    // 从 IPC 调用上下文中安全地获取账户 ID
    // 不要依赖调用者传递的账户 ID
    auto tokenId = IPCSkeleton::GetCallingTokenId();
    
    int32_t accountId = OsAccountManager::GetOsAccountLocalIdFromToken(tokenId);
    if (accountId < 0) {
        CONTACTS_LOGE("Failed to get account ID from token");
        return -1;
    }
    
    return accountId;
}
```

### 风险 5：语音信箱文件路径遍历

**风险级别**：高

**证据位置**：`dataBusiness/voicemail/src/voicemail_database.cpp`

**问题描述**：语音信箱模块在处理音频文件路径时如果没有正确校验路径格式，可能导致路径遍历攻击。

**触发条件**：

```javascript
import dataShare from '@ohos.data.dataShare';

const voicemailUri = "datashare:///com.ohos.voicemailability/calls/voicemail";
const helper = dataShare.createDataShareHelper(voicemailUri);

// 构造包含路径遍历的音频文件路径
const value = {
    "phone_number": "12345678901",
    "display_name": "Test",
    "voicemail_path": "../../../data/security/token.txt"
};

helper.insert(voicemailUri, value);
```

**影响**：

1. 可能读取系统敏感文件
2. 可能覆盖系统文件
3. 可能执行恶意代码

**修复建议**：

```cpp
// 在 voicemail_database.cpp 中添加路径校验
bool ValidateVoicemailPath(const std::string& path) {
    // 检查是否为绝对路径
    if (path.empty() || path[0] != '/') {
        CONTACTS_LOGE("Invalid path: not absolute");
        return false;
    }
    
    // 检查路径遍历
    if (path.find("..") != std::string::npos) {
        CONTACTS_LOGE("Invalid path: path traversal detected");
        return false;
    }
    
    // 检查允许的目录
    constexpr const char* ALLOWED_DIR = "/data/contactsdata/voicemail/";
    if (path.substr(0, strlen(ALLOWED_DIR)) != ALLOWED_DIR) {
        CONTACTS_LOGE("Invalid path: outside allowed directory");
        return false;
    }
    
    return true;
}
```

## 数据流安全分析

```
┌─────────────────────────────────────────────────────────────────┐
│                       数据流安全分析图                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  用户输入                                                   用户数据  │
│     │                                                         │   │
│     ▼                                                         ▼   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    输入处理层                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │ │
│  │  │ 字符串过滤  │  │ 类型校验    │  │ 长度校验    │         │ │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │ │
│  └─────────┼────────────────┼────────────────┼─────────────────┘ │
│            │                │                │                   │
│            ▼                ▼                ▼                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    权限验证层                               │ │
│  │  ┌─────────────────────────────────────────────────────┐   │ │
│  │  │ • 检查调用者是否具有相应权限                         │   │ │
│  │  │ • 验证调用者的 bundle name                          │   │ │
│  │  │ • 验证调用者的 signature                            │   │ │
│  │  │ • 验证调用者的 account ID                           │   │ │
│  │  └─────────────────────────────────────────────────────┘   │ │
│  └───────────────────────────┬───────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    业务处理层                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │ │
│  │  │ 业务逻辑    │  │ 数据组装    │  │ 事务管理    │         │ │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │ │
│  └─────────┼────────────────┼────────────────┼─────────────────┘ │
│            │                │                │                   │
│            ▼                ▼                ▼                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    数据持久化层                             │ │
│  │  ┌─────────────────────────────────────────────────────┐   │ │
│  │  │ • 使用参数化查询防止 SQL 注入                        │   │
│  │  │ • 使用预处理语句                                    │   │
│  │  │ • 验证所有输入数据                                  │   │
│  │  │ • 加密敏感数据（如需要）                            │   │
│  │  └─────────────────────────────────────────────────────┘   │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│                       SQLite 数据库                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 安全建议总结

### 高优先级建议

| 建议 | 优先级 | 相关风险 |
|------|--------|----------|
| 实现完整的输入参数校验（长度、类型、格式） | 高 | 风险 1、3 |
| 修复 Predicates SQL 注入漏洞 | 高 | 风险 2 |
| 修复语音信箱路径遍历漏洞 | 高 | 风险 5 |
| 强化多账户数据隔离 | 高 | 风险 4 |

### 中优先级建议

| 建议 | 优先级 | 说明 |
|------|--------|------|
| 增加安全日志记录 | 中 | 记录安全相关操作 |
| 添加异常行为检测 | 中 | 检测异常访问模式 |
| 实现数据加密 | 中 | 敏感数据加密存储 |
| 增加模糊测试 | 中 | 发现潜在漏洞 |

### 低优先级建议

| 建议 | 优先级 | 说明 |
|------|--------|------|
| 安全代码审计 | 低 | 定期安全审计 |
| 安全培训 | 低 | 开发者安全意识培训 |
| 依赖库更新 | 低 | 更新依赖库到安全版本 |

## 安全检查清单

| 检查项 | 状态 | 说明 |
|--------|------|------|
| N-API 参数校验 | ⚠️ 需加强 | 部分参数校验不完整 |
| Predicates SQL 注入防护 | ❌ 需修复 | 存在 SQL 注入风险 |
| 权限验证完整性 | ✅ 正常 | 权限验证机制完善 |
| 账户数据隔离 | ⚠️ 需审查 | 多账户边界需验证 |
| 文件路径校验 | ❌ 需修复 | 存在路径遍历风险 |
| 日志记录 | ⚠️ 需加强 | 安全日志不完整 |
| 异常行为检测 | ❌ 需实现 | 未实现异常检测 |

## 相关文档

| 文档 | 描述 | 链接 |
|------|------|------|
| 项目概览 | 了解项目定位和能力 | [00_Overview.md](00_Overview.md) |
| 目录结构 | 了解模块划分 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | 理解整体架构 | [02_Architecture.md](02_Architecture.md) |
| API 参考 | 学习接口使用 | [03_API_Reference.md](03_API_Reference.md) |
| 构建文档 | 了解编译配置 | [04_Build.md](04_Build.md) |
