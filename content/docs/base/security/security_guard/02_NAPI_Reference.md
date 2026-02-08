# N-API 参考手册

## 概述

SecurityGuard 通过 N-API 向 JS/TS 应用提供 8 个核心 API。所有 API 通过 `security.securityGuard` 模块导出。

**模块注册位置**: `frameworks/js/napi/security_guard_napi.cpp:1374-1405`

**模块名**: `security.securityGuard`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_modname = "security.securityGuard",
    .nm_register_func = SecurityGuardNapiRegister,
};
```

---

## API 清单

### 1. getModelResult

获取安全模型分析结果（异步 Promise）。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `getModelResult` |
| C++ 实现 | `NapiGetModelResult` |
| 位置 | `security_guard_napi.cpp:537` |
| 模式 | 异步 (Promise) |
| 权限 | 无特殊权限 |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `modelRule` | [ModelRule](#modelrule-结构) | 是 | 模型规则对象 |

#### ModelRule 结构

```typescript
interface ModelRule {
  modelName: string;     // 模型名称，如 'SecurityGuard_JailbreakCheck'
  param?: string;        // 可选参数
}
```

**证据来源**: `security_guard_napi.cpp:549-553`

#### 模型 ID 列表

| 模型名称 | 模型 ID | 功能 |
|----------|---------|------|
| `SecurityGuard_JailbreakCheck` | `3001000000` | 越狱检测 |
| `SecurityGuard_IntegrityCheck` | `3001000001` | 设备完整性检测 |
| `SecurityGuard_SimulatorCheck` | `3001000002` | 物理机检测 |
| `SecurityGuard_RiskFactorCheck` | `3001000009` | 安全风险因子检测 |
| `SecurityGuard_WifiCheck` | `3001000011` | WLAN 风险检测 |

**证据来源**: `security_guard_napi.h:125-132`

#### 返回值

| 类型 | 说明 |
|------|------|
| Promise\<RiskResult> | 风险分析结果对象 |

#### 错误码

| 错误码 | 条件 |
|--------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 801 | API 不支持 |
| 21200001 | 未知错误 |

**证据来源**: `security_guard_napi.h:136-148`

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

async function checkDevice() {
  try {
    // 使用模型规则对象
    const result = await securityGuard.getModelResult({
      modelName: 'SecurityGuard_JailbreakCheck'
    });
    console.log(`Risk level: ${result.riskLevel}`);
  } catch (error) {
    console.error(`Error: ${error.code} - ${error.message}`);
  }
}
```

---

### 2. querySecurityEvent

查询安全事件（异步 Callback）。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `querySecurityEvent` |
| C++ 实现 | `NapiQuerySecurityEvent` |
| 位置 | `security_guard_napi.cpp:894` |
| 模式 | 异步 (Callback) |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `rules` | SecurityEventRuler[] | 是 | 查询条件数组 |
| `querier` | QuerierCallback | 是 | 查询回调 |

#### SecurityEventRuler 结构

```typescript
interface SecurityEventRuler {
  eventId: number;      // 事件 ID
  beginTime?: string;   // 开始时间 (ISO8601)
  endTime?: string;     // 结束时间 (ISO8601)
  param?: string;      // 筛选参数
}
```

#### QuerierCallback 回调

| 回调 | 参数 | 说明 |
|------|------|------|
| onComplete | Array | 查询完成返回结果数组 |
| onError | Error | 查询失败返回错误 |

**证据来源**: `napi_security_event_querier.cpp/h`

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

const rules = [{ eventId: 4100 }];
securityGuard.querySecurityEvent(rules, {
  onComplete: (events) => {
    events.forEach(e => console.log(`Event ${e.eventId}: ${e.content}`));
  },
  onError: (error) => {
    console.error(`Query failed: ${error.code}`);
  }
});
```

---

### 3. reportSecurityEvent

上报安全事件（同步）。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `reportSecurityEvent` |
| C++ 实现 | `NapiReportSecurityInfo` |
| 位置 | `security_guard_napi.cpp:318` |
| 模式 | 同步 |
| 权限 | `ohos.permission.COLLECT_SECURITY_EVENT` |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `eventId` | number | 是 | 事件 ID |
| `version` | string | 是 | 事件版本 |
| `content` | string | 是 | 事件内容 |

#### 返回值

| 类型 | 说明 |
|------|------|
| boolean | true=成功, false=失败 |

**证据来源**: `security_guard_napi.cpp:318-410`

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

const success = securityGuard.reportSecurityEvent({
  eventId: 4100,
  version: '1.0',
  content: JSON.stringify({ type: 'login', userId: 12345 })
});

console.log(success ? 'Report success' : 'Report failed');
```

---

### 4. startSecurityEventCollector

启动安全事件采集器。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `startSecurityEventCollector` |
| C++ 实现 | `NapiStartSecurityEventCollector` |
| 位置 | `security_guard_napi.cpp:694` |
| 模式 | 异步 |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `eventInfo` | [Event](#event-结构) | 是 | 事件信息对象 |
| `duration` | number | 否 | 采集持续时间（毫秒），不传或传 0 表示无限 |

#### Event 结构

```typescript
interface Event {
  eventId: number;       // 事件 ID
  version?: string;       // 事件版本
  content?: string;       // 事件内容
  param?: string;        // 额外参数
}
```

**证据来源**: `security_guard_napi.cpp:706-730`

#### 返回值

| 类型 | 说明 |
|------|------|
| Promise\<number> | 返回错误码 (0=成功) |

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

// 启动采集器，持续 60 秒
await securityGuard.startSecurityEventCollector({
  eventId: 4100,
  version: '1.0',
  content: JSON.stringify({ type: 'security' })
}, 60000);
console.log('Collector started');
```

---

### 5. stopSecurityEventCollector

停止安全事件采集器。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `stopSecurityEventCollector` |
| C++ 实现 | `NapiStopSecurityEventCollector` |
| 位置 | `security_guard_napi.cpp:740` |
| 模式 | 同步 |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `eventInfo` | [Event](#event-结构) | 是 | 事件信息对象 |

**证据来源**: `security_guard_napi.cpp:752-757`

#### 返回值

| 类型 | 说明 |
|------|------|
| number | 错误码 (0=成功) |

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

// 停止采集器
const result = securityGuard.stopSecurityEventCollector({
  eventId: 4100,
  version: '1.0'
});
console.log(result === 0 ? 'Collector stopped' : 'Stop failed');
```

---

### 6. on

订阅安全事件通知。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `on` |
| C++ 实现 | `Subscribe` |
| 位置 | `security_guard_napi.cpp:1254` |
| 模式 | 事件订阅 |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `eventType` | string | 是 | 事件类型 |
| `callback` | function | 是 | 回调函数 |

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

securityGuard.on('securityEvent', (event) => {
  console.log(`Received event: ${event.eventId}`);
});
```

---

### 7. off

取消订阅安全事件通知。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `off` |
| C++ 实现 | `Unsubscribe` |
| 位置 | `security_guard_napi.cpp:1346` |
| 模式 | 事件取消 |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `eventType` | string | 是 | 事件类型 |
| `callback` | function | 否 | 指定回调，不传则取消所有 |

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

// 取消特定回调
securityGuard.off('securityEvent', myCallback);

// 取消所有该类型订阅
securityGuard.off('securityEvent');
```

---

### 8. updatePolicyFile

更新安全策略配置文件。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `updatePolicyFile` |
| C++ 实现 | `NapiUpdatePolicyFile` |
| 位置 | `security_guard_napi.cpp:633` |
| 模式 | 异步 |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `fileInfo` | [FileInfo](#fileinfo-结构) | 是 | 文件信息对象 |

#### FileInfo 结构

```typescript
interface FileInfo {
  fd: number;     // 文件描述符
  name: string;   // 策略名称
}
```

**证据来源**: `security_guard_napi.cpp:581-599`

#### 返回值

| 类型 | 说明 |
|------|------|
| Promise\<void> | 更新成功时 resolve，失败时 reject |

#### 示例

```javascript
import securityGuard from '@ohos.security.securityGuard';

try {
  // 获取文件描述符（需要使用基础文件系统 API）
  const file = fs.openSync('/data/config/new_policy.cfg');
  await securityGuard.updatePolicyFile({
    fd: file.fd,
    name: 'security_policy'
  });
  console.log('Policy updated');
} catch (error) {
  console.error(`Update failed: ${error.code}`);
}
```

---

## 调用链概览

```
JS API Call
    ↓
security.securityGuard (N-API Module)
    ↓
N-API Function (e.g., NapiGetModelResult)
    ↓
Parse JS arguments (napi_get_named_property)
    ↓
SecurityGuardSdkAdaptor::InnerRequestSecurityModelResult
    ↓
DataCollectManager (IPC) → RiskClassifyService
    ↓
Async Work (napi_create_async_work)
    ↓
JS Promise Resolution (napi_resolve_deferred)
```

---

## 错误码定义

**证据来源**: `security_guard_napi.h:136-148`

| 错误码 | 常量名 | 消息 | 触发条件 |
|--------|--------|------|----------|
| 0 | SUCCESS | "The operation was successful" | 操作成功 |
| 201 | NO_PERMISSION | "check permission fail" | 权限校验失败 |
| 202 | NO_SYSTEMCALL | "non-system application uses the system API" | 非系统应用调用系统 API |
| 401 | BAD_PARAM | "Parameter error, please make sure using the correct value" | 参数错误 |
| 801 | API_SUPPORT_ERROR | "API is not supported" | API 不支持 |
| 21200001 | UNKNOWN_ERROR | "Unknown error, please reboot your device and try again" | 未知错误 |

---

## 相关文档

- [项目概览](./01_Overview.md)
- [架构详解](./03_Architecture.md)
- [构建配置](./04_Build.md)
