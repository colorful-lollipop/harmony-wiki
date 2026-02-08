# N-API 参考

> 目的：提供设备互信认证模块 N-API（JS/TS 接口）的完整参考，包括 API 清单、参数说明、返回值和调用示例。
>
> 适用范围：使用 JavaScript/TypeScript 开发 OpenHarmony 应用的开发者。

---

## 1. 概述

### 1.1 模块信息

| 项目 | 值 |
|------|-----|
| **模块名** | `security.deviceauth` |
| **导入方式** | `import deviceauth from '@ohos.deviceauth'` |
| **JS 类名** | `CredManager` |
| **版本要求** | OpenHarmony 5.0+ |

### 1.2 接口清单

| 类型 | API 名称 | 说明 |
|------|----------|------|
| **静态方法** | `getCredMgrInstance()` | 获取 CredManager 实例 |
| **实例方法** | `batchUpdateCredentials()` | 批量更新凭证 |

**证据**：`interfaces/kits/napi/src/credmgr_napi.cpp:378-392`

---

## 2. 静态方法

### 2.1 getCredMgrInstance()

获取 `CredManager` 实例。

**签名**：
```typescript
function getCredMgrInstance(): CredManager;
```

**参数**：无

**返回值**：
| 类型 | 说明 |
|------|------|
| `CredManager` | 凭证管理器实例 |
| `null` | 初始化失败时抛出异常 |

**异步模式**：同步

**错误处理**：
- 初始化失败时抛出 `Error` 对象，包含 `code` 和 `message` 属性

**示例**：
```typescript
import deviceauth from '@ohos.deviceauth';

try {
  const credManager = deviceauth.getCredMgrInstance();
  console.log('获取 CredManager 成功');
} catch (error) {
  console.error('获取 CredManager 失败:', error.code, error.message);
}
```

**C++ 实现**：`NapiCredManager::NapiGetCredMgrInstance`

**证据**：`interfaces/kits/napi/src/credmgr_napi.cpp:121-161`

---

## 3. 实例方法

### 3.1 batchUpdateCredentials()

批量更新凭证信息。

**签名**：
```typescript
// Promise 模式
function batchUpdateCredentials(osAccountId: number, requestParams: string): Promise<string>;

// Callback 模式
function batchUpdateCredentials(
  osAccountId: number,
  requestParams: string,
  callback: (error: Error, result: string) => void
): void;
```

**参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `osAccountId` | `number` | 是 | OS 账户 ID |
| `requestParams` | `string` | 是 | JSON 格式的请求参数 |
| `callback` | `Function` | 否 | 回调函数（可选） |

**返回值**：

| 模式 | 类型 | 说明 |
|------|------|------|
| Promise | `Promise<string>` | 成功时返回更新结果 JSON 字符串 |
| Callback | `void` | 通过 callback 返回结果 |

**回调签名**：
```typescript
callback(error: Error | null, result: string): void;
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `error` | `Error | null` | 成功时为 `null`，失败时包含错误信息 |
| `result` | `string` | 成功时的返回数据 |

**异步模式**：异步（使用 `napi_create_async_work` 在工作线程执行）

**示例**：

```typescript
// Promise 模式
const osAccountId = 100;
const requestParams = JSON.stringify({
  credList: [
    { credId: 'cred001', credential: 'base64encoded...' }
  ]
});

try {
  const result = await credManager.batchUpdateCredentials(osAccountId, requestParams);
  console.log('批量更新成功:', result);
} catch (error) {
  console.error('批量更新失败:', error.code, error.message);
}

// Callback 模式
credManager.batchUpdateCredentials(
  osAccountId,
  requestParams,
  (error, result) => {
    if (error) {
      console.error('批量更新失败:', error.code, error.message);
    } else {
      console.log('批量更新成功:', result);
    }
  }
);
```

**C++ 实现**：`NapiCredManager::NapiBatchUpdateCreds`

**调用链**：
```
JS: batchUpdateCredentials()
  ↓
N-API: NapiBatchUpdateCreds()
  ↓
AsyncWork: AsyncBatchUpdateCredsProcess()
  ↓
Native: CredManager->batchUpdateCredentials()
```

**证据**：`interfaces/kits/napi/src/credmgr_napi.cpp:353-376`

---

## 4. CredManager 类

### 4.1 类定义

```typescript
class CredManager {
  // 获取实例
  static getCredMgrInstance(): CredManager;

  // 批量更新凭证
  batchUpdateCredentials(
    osAccountId: number,
    requestParams: string,
    callback?: (error: Error, result: string) => void
  ): Promise<string> | void;
}
```

### 4.2 错误码

错误码通过 `Error.code` 属性返回，完整错误码列表见 [Inner API 错误码](./04_Inner_API.md#4-错误码)。

**常见错误码**：

| 错误码 | 说明 |
|--------|------|
| `0` | 成功 |
| `262146` (`IS_ERR_INVALID_PARAMS`) | 参数错误 |
| `65537` (`IS_ERR_HUKS_GENERATE_KEY_FAILED`) | HUKS 密钥生成失败 |
| `65545` (`IS_ERR_LOCAL_CRED_NOT_EXIST`) | 本地凭据不存在 |
| `65551` (`IS_ERR_AUTH_ERR_PIN_NOT_MATCH`) | PIN 码不匹配 |

---

## 5. 参数格式

### 5.1 requestParams 格式

`requestParams` 为 JSON 字符串，支持以下结构：

```json
{
  "credList": [
    {
      "credId": "string",        // 凭证 ID
      "credential": "string",    // Base64 编码的凭证数据
      "authType": number,        // 认证类型
      "credentialType": number   // 凭证类型
    }
  ],
  "deleteList": [
    "credId1", "credId2"        // 待删除的凭证 ID 列表
  ]
}
```

### 5.2 返回值格式

返回值为 JSON 字符串：

```json
{
  "result": 0,                   // 操作结果码
  "credList": [
    {
      "credId": "string",        // 凭证 ID
      "updateTime": number      // 更新时间戳
    }
  ],
  "failedList": [
    {
      "credId": "string",
      "errorCode": number,
      "errorMsg": "string"
    }
  ]
}
```

---

## 6. 线程模型

### 6.1 异步执行

`batchUpdateCredentials()` 方法使用 `napi_create_async_work` 实现异步：

```
JS 线程 ──► N-API ──► AsyncWorkQueue ──► 工作线程 ──► Native 处理
                    │                              │
                    └──────────────────────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    ▼                                   ▼
              Promise.resolve()                    Callback 调用
```

### 6.2 内存管理

- **Ctx 结构**：`BatchUpdateCredsCtx` 管理异步上下文
- **引用计数**：使用 `napi_ref` 管理 C++ 对象引用
- **资源释放**：`FreeBatchUpdateCredsCtx()` 在完成时释放资源

**证据**：`interfaces/kits/napi/src/credmgr_napi.cpp:32-78`

---

## 7. 常见问题

### 7.1 获取实例失败

**问题**：调用 `getCredMgrInstance()` 抛出异常

**可能原因**：
1. 设备认证服务未初始化
2. 系统能力加载失败

**解决方案**：
```typescript
import deviceauth from '@ohos.deviceauth';

// 延迟获取实例
setTimeout(() => {
  const credManager = deviceauth.getCredMgrInstance();
}, 1000);
```

### 7.2 回调未执行

**问题**：Callback 模式回调函数未调用

**可能原因**：
1. `osAccountId` 无效
2. `requestParams` 格式错误

**调试方法**：
```typescript
try {
  JSON.parse(requestParams); // 验证 JSON 格式
} catch (e) {
  console.error('requestParams 格式错误');
}
```

---

## 8. 相关跳转

| 内容 | 文档 |
|------|------|
| 项目概览 | [01_Overview.md](./01_Overview.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| Inner API | [04_Inner_API.md](./04_Inner_API.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*本文档最后更新：2026-02-06*
