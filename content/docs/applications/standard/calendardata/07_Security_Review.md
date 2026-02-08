# 安全风险评审

## 目的

本文档对 CalendarData 组件进行安全风险分析，包括威胁模型、攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- 目标读者：安全审计员、架构师、高级开发者
- 知识级别：高级
- 前置知识：了解 OpenHarmony 安全机制、权限模型

## 威胁模型

### 攻击者假设

| 攻击者类型 | 描述 | 影响等级 |
|------------|------|---------|
| 恶意应用 | 已安装的第三方应用 | 高 |
| 越级应用 | 拥有高权限的预置应用 | 高 |
| 越权应用 | 利用漏洞提权的应用 | 中 |
| 物理攻击者 | 拥有设备物理访问 | 中 |

### 攻击面

```
┌─────────────────────────────────────────────┐
│              外部世界                      │
└───────────┬─────────────────────────────┘
            │
            ↓ 1. N-API 调用
┌─────────────────────────────────────────────┐
│  calendarmanager (N-API)               │
└───────────┬─────────────────────────────┘
            │
            ↓ 2. DataShare IPC
┌─────────────────────────────────────────────┐
│  DataShareExtAbility                 │
│  - 权限验证                          │
│  - URI 解析                          │
└───────────┬─────────────────────────────┘
            │
            ↓ 3. 数据库操作
┌─────────────────────────────────────────────┐
│  RDB 数据库                          │
└───────────┬─────────────────────────────┘
            │
            ↓ 4. 文件系统
┌─────────────────────────────────────────────┐
│  日历数据文件 (.db)                  │
└─────────────────────────────────────────────┘
```

## 攻击面分析

### 1. N-API 调用面

**入口**: JavaScript/ArkTS 应用调用 N-API

**潜在风险**:
- 参数注入（恶意构造的 Event 对象）
- 权限绕过（通过 URI 注入）
- 资源泄漏（通过错误消息）

**证据**: calendarmanager/napi/src/*_napi.cpp

### 2. DataShare IPC 面

**入口**: DataShareExtAbility 处理跨应用请求

**潜在风险**:
- URI 注入（恶意构造的 URI 路径）
- Token 欺骗（伪造或重用 tokenId）
- 权限提升（利用权限检查漏洞）

**证据**:
- entry/src/main/ets/DataAbility/DataShareExtAbility.ets:48-177
- dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets:112-165

### 3. URI 解析面

**入口**: 从 URI 提取 bundleName 和 tokenId

**URI 格式**: `datashare:///com.ohos.calendarData/<table>/<bundleName>/<tokenId>`

**潜在风险**:
- 路径遍历（`../`）绕过表名检查
- Token 暴露（明文传输 tokenId）
- 表名注入（恶意表名）

**证据**: dataprovider/src/main/ets/AuthenticationUriHelper.ets:23-32

### 4. 权限检查面

**入口**: 验证调用者权限

**潜在风险**:
- TOCTOU（Time-of-Check-Time-of-Use）
- 权限检查绕过
- 条件竞争（检查与使用之间状态变更）

**证据**:
- dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets:112-165
- calendarmanager/native/src/data_share_helper_manager.cpp:73-86

### 5. 数据库操作面

**入口**: RDB 增删改查

**潜在风险**:
- SQL 注入（如果使用拼接 SQL）
- 越权数据访问
- 数据篡改

**证据**: datamanager/src/main/ets/processor/*.ets

## 可被利用点

### 风险 1: URI 注入导致权限绕过

**严重性**: 🔴 高危

**证据**:
- dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets:114
- dataprovider/src/main/ets/AuthenticationUriHelper.ets:23-32

**触发**:
1. 恶意应用构造恶意 URI：
   ```
   datashare:///com.ohos.calendarData/../../other_table/malicious_bundle/fake_token
   ```
2. URI 解析未充分验证路径遍历

**调用链**:
```javascript
// 恶意应用
const maliciousUri = 'datashare:///com.ohos.calendarData/../../other_table/bundle/token';
calendar.getEvents({ uri: maliciousUri });
    ↓
entry/DataShareExtAbility.query()
    ↓
AuthenticationUriHelper.getBundleNameAndTokenIDByUri()
    ↓
// 路径未充分验证
return { bundleName, tokenId }
```

**影响**:
- 绕过表名检查
- 访问其他表数据
- 潜在数据泄露

**修复建议**:
```typescript
// dataprovider/src/main/ets/AuthenticationUriHelper.ets
function getBundleNameAndTokenIDByUri(uri: string): BundleNameAndTokenId {
  // 1. 验证 URI 格式
  if (!uri.startsWith('datashare:///com.ohos.calendarData/')) {
    throw new Error('Invalid URI format');
  }
  
  // 2. 提取路径部分
  const path = uri.substring('datashare:///com.ohos.calendarData/'.length);
  
  // 3. 分割路径
  const parts = path.split('/');
  
  // 4. 验证部分数量
  if (parts.length !== 3) {
    throw new Error('Invalid URI structure');
  }
  
  // 5. 验证表名（白名单）
  const validTables = [
    CalendarsColumns.TABLE_NAME,
    EventColumns.TABLE_NAME,
    InstancesColumns.TABLE_NAME,
    RemindersColumns.TABLE_NAME,
    CalendarAlertsColumns.TABLE_NAME
  ];
  const tableName = parts[0];
  if (!validTables.includes(tableName)) {
    throw new Error('Invalid table name');
  }
  
  // 6. 验证 bundleName 和 tokenId 格式
  const bundleName = parts[1];
  const tokenId = parts[2];
  if (!/^[a-zA-Z0-9.]+$/.test(bundleName)) {
    throw new Error('Invalid bundle name');
  }
  if (!/^[0-9]+$/.test(tokenId)) {
    throw new Error('Invalid token id');
  }
  
  return { bundleName, tokenId };
}
```

---

### 风险 2: Token 欺骗导致权限提升

**严重性**: 🔴 高危

**证据**:
- dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets:114
- calendarmanager/native/src/data_share_helper_manager.cpp:73-86

**触发**:
1. 恶意应用伪造或重用其他应用的 tokenId
2. 权限检查时仅验证 tokenId 而未验证 bundleName 匹配

**调用链**:
```javascript
// 恶意应用
const stolenTokenId = '123456';  // 伪造的 token
const maliciousUri = `datashare:///com.ohos.calendarData/Events/malicious_app/${stolenTokenId}`;

calendar.addEvent(event, maliciousUri);
    ↓
DataShareAbilityAuthenticateProxy.verifyByUri()
    ↓
getBundleNameAndTokenIDByUri(maliciousUri)  // { bundleName: 'malicious_app', tokenId: '123456' }
    ↓
verifyAccessByTokenId(stolenTokenId, permission)  // 仅验证 tokenId，未验证 bundleName
```

**影响**:
- 权限提升
- 访问其他应用数据
- 数据篡改

**修复建议**:
```typescript
// dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets
async function verifyByUri(...): Promise<number> {
  let bundleNameAndTokenId = getBundleNameAndTokenIDByUri(uri);
  const { bundleName, tokenId } = bundleNameAndTokenId;
  
  // 1. 获取调用者的真实 bundleName 和 tokenId
  const callerTokenId = IPCSkeleton.GetCallingTokenID();
  const callerBundleName = GetBundleNameByTokenId(callerTokenId);
  
  // 2. 验证 URI 中的 bundleName 与调用者匹配
  if (callerBundleName !== bundleName) {
    return PERMISSIONS_FLAG_UNAUTHORIZED;
  }
  
  // 3. 验证 URI 中的 tokenId 与调用者匹配
  if (callerTokenId.toString() !== tokenId) {
    return PERMISSIONS_FLAG_UNAUTHORIZED;
  }
  
  // 4. 继续原有的权限验证
  // ...
}
```

---

### 风险 3: 权限检查 TOCTOU 竞争条件

**严重性**: 🟡 中危

**证据**:
- dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets:112-165
- calendarmanager/native/src/data_share_helper_manager.cpp:73-86

**触发**:
1. 权限检查后、操作前，应用权限被撤销
2. 多线程并发访问导致竞争条件

**调用链**:
```typescript
// 应用 A
async function maliciousOperation() {
  // 1. 并发发起请求
  const requests = [
    calendar.addEvent(event1),
    calendar.addEvent(event2),
    // ...
  ];
  
  // 2. 在权限检查和操作之间
  await new Promise(r => setTimeout(r, 100));
  
  // 3. 撤销应用权限
  
  await Promise.all(requests);
}
```

**影响**:
- 权限检查后、操作前数据访问
- 潜在数据泄露或篡改

**修复建议**:
```typescript
// dataprovider/src/main/ets/DataShareAbilityDelegate.ets
insertByHighAuthority(uri, value, callback) {
  // 使用数据库级别的权限检查
  // 而非在应用层检查
  
  rdbStore.insertWithPermissionCheck(uri, value, WRITE_WHOLE_CALENDAR)
    .then(result => callback(null, result))
    .catch(error => callback(error, -1));
}
```

---

### 风险 4: 错误信息泄露敏感数据

**严重性**: 🟢 低危

**证据**:
- calendarmanager/native/src/native_util.cpp (错误处理）
- datamanager/src/main/ets/utils/Log.ets (日志记录）

**触发**:
1. 数据库查询失败
2. 权限验证失败
3. URI 解析失败

**调用链**:
```javascript
// 恶意应用
try {
  const events = await calendar.getEvents({ uri: maliciousUri });
} catch (error) {
  // 错误消息可能包含：
  // - 数据库表名
  // - 内部路径
  // - 调试信息
  console.error(error.message);  // 泄露给攻击者
}
```

**影响**:
- 信息泄露
- 辅助进一步攻击

**修复建议**:
```cpp
// calendarmanager/native/src/native_util.cpp
void LogError(const std::string &message, int errorCode) {
  // 不要记录敏感信息
  std::string sanitizedMessage = message;
  
  // 移除路径信息
  size_t pos = 0;
  while ((pos = sanitizedMessage.find("/data/calendardata/", pos)) != std::string::npos) {
    sanitizedMessage.replace(pos, 20, "[REDACTED]");
    pos += 10;
  }
  
  // 记录到日志
  LOG_ERROR("Error %d: %s", errorCode, sanitizedMessage.c_str());
  
  // 返回给调用者：仅返回错误码
  return errorCode;
}
```

---

### 风险 5: 批量操作资源耗尽

**严重性**: 🟢 低危

**证据**:
- dataprovider/src/main/ets/DataShareAbilityDelegate.ets:66-83
- dataprovider/src/main/ets/DataShareAbilityDelegate.ets:108-126

**触发**:
1. 恶意应用发起大批量操作
2. 无大小限制

**调用链**:
```javascript
// 恶意应用
const maliciousEvents = new Array(100000).fill(createEvent());
await calendar.addEvents(maliciousEvents);  // 10万个事件
```

**影响**:
- 数据库资源耗尽
- 系统性能下降
- 拒绝服务

**修复建议**:
```typescript
// dataprovider/src/main/ets/DataShareAbilityDelegate.ets
batchInsertByHighAuthority(uri, value, callback) {
  // 限制批量操作大小
  const MAX_BATCH_SIZE = 1000;
  if (value.length > MAX_BATCH_SIZE) {
    const error = {
      code: ErrorCode.ILLEGAL_ARGUMENT_ERROR,
      name: 'BatchSizeExceeded',
      message: `Batch size exceeds limit of ${MAX_BATCH_SIZE}`
    };
    callback(error, -1);
    return;
  }
  
  // 分批处理
  const chunks = [];
  for (let i = 0; i < value.length; i += MAX_BATCH_SIZE) {
    chunks.push(value.slice(i, i + MAX_BATCH_SIZE));
  }
  
  // 处理每个批次
  // ...
}
```

---

## 信任边界

### 进程边界

| 进程 | 信任级别 | 说明 |
|------|---------|------|
| 日历应用 | 高 | 系统预置应用 |
| 第三方应用 | 低 | 需权限验证 |
| 系统服务 | 高 | 信任的系统组件 |

### 数据边界

| 数据类型 | 信任级别 | 说明 |
|---------|---------|------|
| 用户输入数据 | 低 | 需验证和清理 |
| 数据库存储数据 | 中 | 已经过应用层验证 |
| 内部配置数据 | 高 | 系统控制的 |

### 权限边界

| 权限 | 信任级别 | 说明 |
|------|---------|------|
| *_WHOLE_CALENDAR | 高 | 系统级权限 |
| *_CALENDAR | 中 | 应用级权限 |
| 无权限 | 低 | 公开接口 |

## 安全检查清单

### 输入验证

- [ ] 所有 N-API 参数都经过类型检查
- [ ] URI 路径使用白名单验证
- [ ] tokenId 和 bundleName 都验证
- [ ] 大小和范围限制已设置

### 权限控制

- [ ] 权限检查在操作前进行
- [ ] TOCTOU 竞争条件已防护
- [ ] Token 欺骗已防护
- [ ] 权限提升已检测

### 数据保护

- [ ] 敏感数据不在日志中
- [ ] 错误消息已清理
- [ ] 数据库查询使用参数化
- [ ] 越权访问已阻止

### 资源限制

- [ ] 批量操作大小有限制
- [ ] 查询结果集大小有限制
- [ ] 内存使用有限制
- [ ] CPU 使用有限制

## 检查范围与局限性

### 已检查范围

1. ✅ N-API 参数验证
2. ✅ DataShare 权限检查
3. ✅ URI 解析安全性
4. ✅ 数据库操作安全
5. ✅ 错误处理和日志
6. ✅ 批量操作限制

### 未检查范围

⚠️ 以下内容未在本评审中覆盖：

1. **网络通信**: CalendarData 不直接进行网络通信，由 sync 模块负责
2. **文件系统访问**: RDB 内部实现未检查
3. **内存安全**: C++ 代码未进行内存安全审计
4. **加密**: 数据存储未加密
5. **防篡改**: 数据库完整性未验证
6. **审计日志**: 详细的审计日志未检查

### 建议进一步检查

1. 进行完整的代码安全审计（SAST）
2. 进行动态测试（DAST）
3. 进行模糊测试
4. 审查 RDB 配置和访问控制
5. 审查系统日志和审计机制

## 修复优先级

| 优先级 | 风险 | 预估工作量 |
|--------|------|-----------|
| P0 | URI 注入权限绕过 | 1-2 天 |
| P0 | Token 欺骗 | 2-3 天 |
| P1 | TOCTOU 竞争条件 | 3-5 天 |
| P2 | 错误信息泄露 | 1 天 |
| P2 | 批量操作资源耗尽 | 1-2 天 |

## 安全最佳实践

### 开发阶段

1. 使用最小权限原则
2. 所有输入都验证
3. 使用安全 API（参数化查询）
4. 避免记录敏感信息
5. 实施资源限制

### 测试阶段

1. 单元测试覆盖安全场景
2. 集成测试模拟攻击
3. 渗透测试定期执行
4. 代码审查包括安全专家

### 部署阶段

1. 生产环境关闭调试日志
2. 定期审计权限配置
3. 监控异常行为
4. 及时应用安全补丁

## 相关文档

- [对外 API](03_External_API.md) - 权限要求
- [内部 API](04_Internal_API.md) - 接口安全考虑
- [架构说明](02_Architecture.md) - 信任边界

---

返回 [目录](SUMMARY.md) | [首页](README.md)
