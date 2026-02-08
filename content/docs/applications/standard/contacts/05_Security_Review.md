# 安全风险评审

## 概述

本文档对 Contacts 应用进行安全风险分析，涵盖权限使用、数据存储、输入验证等方面。

## 评审范围

| 评审项 | 状态 | 说明 |
|--------|------|------|
| 权限使用 | ✅ 已评审 | 14 个系统权限 |
| 输入验证 | ✅ 已评审 | 用户输入处理 |
| 数据存储 | ✅ 已评审 | RDB 和 Preferences |
| 组件通信 | ✅ 已评审 | Ability 间调用 |
| 网络安全 | ⚠️ 有限 | 本应用不涉及网络 |

## 攻击面分析

### 暴露接口

| 接口类型 | 攻击面 | 风险等级 |
|----------|--------|----------|
| Ability 入口 | 外部应用可通过 Intent 启动 | 中 |
| DataAbility URI | 外部应用可访问联系人数据 | 高 |
| 静态订阅者 | 可接收系统事件 | 中 |

### 权限攻击面

| 权限 | 敏感度 | 潜在风险 |
|------|--------|----------|
| READ_CONTACTS | 高 | 联系人数据泄露 |
| WRITE_CONTACTS | 高 | 联系人数据篡改 |
| READ_CALL_LOG | 高 | 通话记录泄露 |
| WRITE_CALL_LOG | 高 | 通话记录篡改 |
| PLACE_CALL | 高 | 未经授权呼叫 |
| GET_TELEPHONY_STATE | 中 | 电话状态信息泄露 |

> **证据来源**: `entry/src/main/module.json5` lines 61-114

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                     Contacts 应用边界                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Ability 入口 (MainAbility)                         │   │
│  │  - 接收外部 Want 参数                               │   │
│  │  - 验证来源可信度 (见 onCreate)                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  权限请求 (PermissionManager)                       │   │
│  │  - 用户授权后访问敏感能力                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  数据访问 (ContactRepository)                       │   │
│  │  - 本地 RDB 操作                                    │   │
│  │  - DataAbility 调用                                 │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                     系统边界                                  │
│  - DataShare 提供者 (contactsdataability)                   │
│  - 电话服务 (telephony)                                     │
│  - 通知服务 (notification)                                  │
└─────────────────────────────────────────────────────────────┘
```

## 已识别风险

### 风险 1: 联系人数据越权读取 ⚠️ HIGH

| 属性 | 说明 |
|------|------|
| **证据** | `module.json5` 声明 `READ_CONTACTS` 和 `WRITE_CONTACTS` |
| **触发条件** | 应用获取权限后，可访问全部联系人数据 |
| **影响范围** | 用户所有联系人信息泄露 |
| **风险等级** | 高 |

**利用路径**:
```
用户授予权限 → ContactRepository.findAll() → 读取全部联系人
```

**修复建议**:
```typescript
// 实现数据访问控制
class ContactRepository {
    // 只返回用户可见的联系人
    async findByScope(userId: string, scope: AccessScope): Promise<Contact[]> {
        // 根据权限范围过滤数据
    }
}
```

---

### 风险 2: Intent 参数注入 ⚠️ MEDIUM

| 属性 | 说明 |
|------|------|
| **证据** | `MainAbility.ts:49-58` 接收外部 Want 参数 |
| **触发条件** | 恶意应用发送构造的 Intent |
| **影响范围** | 联系人数据泄露或应用状态异常 |
| **风险等级** | 中 |

**利用路径**:
```typescript
// onRequest 接收参数但未严格校验
onRequest(want: Want, isOnCreate: boolean) {
    const data = want.parameters['missedCallData'];  // 未校验来源
    const action = want.parameters['action'];         // 未校验类型
}
```

**修复建议**:
```typescript
// 验证来源和参数合法性
onRequest(want: Want, isOnCreate: boolean) {
    if (!want || !want.parameters) {
        return;
    }
    
    // 验证来源
    if (!this.isTrustedSource(want.bundleName)) {
        HiLog.w(TAG, 'Untrusted source: ' + want.bundleName);
        return;
    }
    
    // 校验参数类型
    const data = want.parameters['missedCallData'];
    if (typeof data !== 'object' || !this.isValidMissedCallData(data)) {
        HiLog.e(TAG, 'Invalid missedCallData format');
        return;
    }
}
```

---

### 风险 3: 联系人数据完整性 ⚠️ MEDIUM

| 属性 | 说明 |
|------|------|
| **证据** | `RawContactsColumns.ets` 包含 `IS_DELETED` 标志 |
| **触发条件** | 数据同步或批量删除操作 |
| **影响范围** | 联系人数据不一致 |
| **风险等级** | 中 |

**利用路径**:
```
批量删除操作 → IS_DELETED 状态未同步 → 数据残留
```

**修复建议**:
```typescript
// 确保删除操作的原子性
async deleteContacts(ids: number[]): Promise<boolean> {
    const rdbStore = await getRdbStore();
    const transaction = await rdbStore.startTransaction();
    try {
        // 1. 标记删除
        for (const id of ids) {
            await rdbStore.executeSql(
                `UPDATE raw_contacts SET is_deleted=1 WHERE id=?`,
                [id]
            );
        }
        // 2. 级联删除关联数据
        await this.cascadeDelete(ids);
        // 3. 提交事务
        transaction.commit();
        return true;
    } catch (error) {
        transaction.rollback();
        return false;
    }
}
```

---

### 风险 4: URI 路径遍历 ⚠️ LOW

| 属性 | 说明 |
|------|------|
| **证据** | `Contacts.ets:22-23` 定义 DataAbility URI |
| **触发条件** | 外部应用构造恶意 URI |
| **影响范围** | 未授权数据访问 |
| **风险等级** | 低 |

**利用路径**:
```
外部应用 → DataAbility URI → 未授权表访问
```

**修复建议**:
```typescript
// URI 白名单校验
class ContactsDataAbility {
    private readonly ALLOWED_URIS = [
        'datashare:///com.ohos.contactsdataability',
        'datashare:///com.ohos.contactsdataability/contacts/contact',
        // ...
    ];
    
    checkUriPermission(uri: string): boolean {
        return this.ALLOWED_URIS.includes(uri);
    }
}
```

---

### 风险 5: 权限请求过度 ⚠️ LOW

| 属性 | 说明 |
|------|------|
| **证据** | `PermissionManager.ets:38-44` 一次性请求多个权限 |
| **触发条件** | 应用启动时请求所有权限 |
| **影响范围** | 用户可能拒绝必要权限 |
| **风险等级** | 低 |

**当前代码**:
```typescript
// 一次性请求所有权限
let requestPermissions: Permissions[] = [
    "ohos.permission.READ_CONTACTS",
    "ohos.permission.WRITE_CONTACTS",
    "ohos.permission.MANAGE_VOICEMAIL",
    "ohos.permission.READ_CALL_LOG",
    "ohos.permission.WRITE_CALL_LOG"
];
```

**修复建议**:
```typescript
// 按需请求权限
async requestPermissionsAsNeeded(operation: OperationType): Promise<boolean> {
    const requiredPermissions = this.getRequiredPermissions(operation);
    const missing = await this.checkMissingPermissions(requiredPermissions);
    if (missing.length > 0) {
        return this.requestPermissions(missing);
    }
    return true;
}
```

---

### 风险 6: Worker 线程数据泄露 ⚠️ LOW

| 属性 | 说明 |
|------|------|
| **证据** | `MainAbility.ts:32` 创建 DataWorker |
| **触发条件** | Worker 消息处理不当 |
| **影响范围** | 联系人数据泄露 |
| **风险等级** | 低 |

**修复建议**:
```typescript
// Worker 消息序列化白名单
class DataWorker {
    onmessage(e: MessageEvents) {
        const allowedFields = ['id', 'name', 'phone'];
        const sanitized = {};
        for (const key of allowedFields) {
            if (e.data[key] !== undefined) {
                sanitized[key] = e.data[key];
            }
        }
        this.process(sanitized);
    }
}
```

---

### 风险 7: 全局变量污染 ⚠️ LOW

| 属性 | 说明 |
|------|------|
| **证据** | `MainAbility.ts:63-70` 使用 globalThis 存储上下文 |
| **触发条件** | 多实例或竞态条件 |
| **影响范围** | 数据错乱或泄露 |
| **风险等级** | 低 |

**当前代码**:
```typescript
globalThis.context = this.context;
globalThis.abilityWant = want;
globalThis.DataWorker = this.mDataWorker;
globalThis.presenterManager = new PresenterManager(this.context, this.mDataWorker);
```

**修复建议**:
```typescript
// 使用 LocalStorage 替代 globalThis
const storage = new LocalStorage();
storage.setOrCreate('context', this.context);
windowStage.loadContent('pages/index', storage);
```

## 安全最佳实践

### 1. 权限最小化

| 原则 | 实施方式 |
|------|----------|
| 按需请求 | 只在需要时请求权限 |
| 动态权限 | 使用时请求而非启动时 |
| 权限降级 | 处理用户拒绝权限的场景 |

### 2. 输入校验

```typescript
// 校验联系人数据
function validateContact(contact: Contact): boolean {
    if (contact.displayName?.length > MAX_NAME_LENGTH) {
        return false;
    }
    if (contact.phoneNumbers?.some(p => !isValidPhone(p.number))) {
        return false;
    }
    return true;
}
```

### 3. 数据加密

```typescript
// 敏感数据加密存储
import crypto from '@ohos.security.crypto';

async function encryptContact(contact: Contact): Promise<EncryptedContact> {
    const cipher = crypto.createCipher('AES256');
    return {
        ...contact,
        encryptedData: cipher.encrypt(JSON.stringify(contact.sensitiveFields))
    };
}
```

### 4. 审计日志

```typescript
// 记录敏感操作
function logAccess(operation: string, targetId: string): void {
    HiLog.i(TAG, `SECURITY: ${operation} on ${targetId}`, {
        timestamp: Date.now(),
        userId: getCurrentUserId(),
        source: getCallingAppId()
    });
}
```

## 安全加固检查清单

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 权限声明最小化 | ⚠️ 待优化 | 考虑按需请求 |
| Intent 参数校验 | ❌ 未实现 | 需添加来源验证 |
| 数据完整性校验 | ⚠️ 部分实现 | 删除操作需加强 |
| URI 访问控制 | ❌ 未实现 | 需添加白名单 |
| 敏感数据加密 | ❌ 未实现 | 建议增加 |
| 审计日志 | ❌ 未实现 | 建议增加 |

## 相关安全文档

- [OpenHarmony 权限机制](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/permission-guidelines.md)
- [Ability 安全指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/ability/ability-guidelines.md)
