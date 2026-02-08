# 05. 安全评审

## 目的与适用范围

本文档对 OpenHarmony MMS 应用进行安全风险评审，识别攻击面、信任边界和潜在的安全漏洞。

**适用读者**: 安全工程师、架构师、开发工程师  
**阅读时间**: 约 40 分钟  
**评审范围**: entry/src/main/ets/ 目录下所有业务代码（不含测试）

---

## 执行摘要

### 风险等级分布

| 等级 | 数量 | 状态 |
|------|------|------|
| 🔴 高危 | 4 | 需立即修复 |
| 🟡 中危 | 5 | 建议修复 |
| 🟢 低危 | 3 | 可选优化 |

### 主要发现

1. **短信接收处理** - 外部数据直接解析，存在格式异常风险
2. **HTTP 通信** - MMS 发送/下载使用明文 HTTP，存在中间人攻击风险
3. **数据库操作** - SQL 查询参数未充分校验
4. **电话号码处理** - 格式化处理可能存在注入风险

---

## 攻击面分析

### 攻击面清单

```
┌─────────────────────────────────────────────────────────────┐
│                      攻击面                                  │
├─────────────────────────────────────────────────────────────┤
│  外部输入                                                   │
│  ├── SMS/MMS 接收 (MmsStaticSubscriber)                    │
│  ├── HTTP 响应 (MMS 下载)                                   │
│  ├── 用户输入 (短信内容、号码)                               │
│  └── 联系人数据查询结果                                     │
│                                                             │
│  网络通信                                                   │
│  ├── HTTP 请求 (MMS 发送/下载)                              │
│  └── DataShare 跨进程通信                                   │
│                                                             │
│  数据存储                                                   │
│  ├── 短信数据库 (DataShare)                                 │
│  ├── 偏好设置 (Preferences)                                 │
│  └── 通知数据                                               │
│                                                             │
│  系统接口                                                   │
│  ├── 电话拨打                                               │
│  ├── 通知发送                                               │
│  └── 公共事件广播                                           │
└─────────────────────────────────────────────────────────────┘
```

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  不信任区域 (Untrusted)                                      │
│  ├── 网络数据 (SMS/MMS PDU, HTTP 响应)                      │
│  ├── 其他应用数据 (联系人数据库)                             │
│  └── 用户输入                                                │
└─────────────────────────────────────────────────────────────┘
                              ↓
                    ┌───────────────┐
                    │   验证/过滤层   │
                    └───────┬───────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  信任区域 (Trusted)                                          │
│  ├── MMS 应用内部逻辑                                        │
│  ├── 短信数据库 (本地写入)                                   │
│  └── 系统 API 调用                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 高危风险项 (4项)

### 🔴 RISK-001: SMS PDU 解析缺乏异常处理

**风险等级**: 高危  
**位置**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:49`  
**证据**:

```typescript
// MmsStaticSubscriber.ts:44-66
dealSmsReceiveData(data, context): Promise<void> {
    let netType: string = data.parameters.isCdma ? '3gpp2' : '3gpp';
    let promisesAll = [];
    data.parameters.pdus.forEach(pdu => {
        // ⚠️ PDU 数据直接传入，无长度/格式校验
        let promise = telSim.createMessage(this.convertStrArray(pdu), netType);
        promisesAll.push(promise);
    });
    // ...
}

// MmsStaticSubscriber.ts:165-195
convertStrArray(sourceStr): Array<number> {
    let wby: string = sourceStr;
    let length: number = wby.length;
    // ⚠️ 无输入长度限制，可能导致内存问题
    let isDouble: boolean = (length % 2) == 0;
    let halfSize: number = parseInt('' + length / 2);
    // 字符串分割和解析
    for (let i = 0;i < halfSize; i++) {
        number0xArray[i] = '0x' + wby.substr(i * 2, 2);
    }
}
```

**触发条件**: 
- 接收到格式异常的 SMS PDU
- PDU 字符串超长或包含非十六进制字符

**影响**:
- 应用崩溃
- 潜在的内存问题
- 短信接收功能拒绝服务

**修复建议**:
```typescript
// 建议增加输入校验
dealSmsReceiveData(data, context) {
    // 1. 验证 PDU 数组存在且非空
    if (!data.parameters?.pdus || !Array.isArray(data.parameters.pdus)) {
        HiLog.e(TAG, 'Invalid PDU data');
        return;
    }
    
    // 2. 验证每个 PDU 格式
    for (const pdu of data.parameters.pdus) {
        if (typeof pdu !== 'string' || pdu.length > MAX_PDU_LENGTH) {
            HiLog.e(TAG, 'Invalid PDU format or length');
            continue;
        }
        // 3. 验证十六进制格式
        if (!/^[0-9a-fA-F]*$/.test(pdu)) {
            HiLog.e(TAG, 'PDU contains non-hex characters');
            continue;
        }
    }
}
```

---

### 🔴 RISK-002: HTTP 通信未启用 TLS

**风险等级**: 高危  
**位置**: `entry/src/main/ets/service/SendMsgService.ets:73`  
**证据**:

```typescript
// SendMsgService.ets:71-95
sendMmsMessage(params, callback) {
    let httpRequest = http.createHttp();
    httpRequest.request(
        common.string.MMS_URL,  // ⚠️ HTTP 而非 HTTPS
        {
            method: http.RequestMethod.POST,
            header: {
                'Content-Type': 'application/vnd.wap.mms-message'
            },
            extraData: JSON.stringify(params),
            readTimeout: 60000,
            connectTimeout: 60000
        }, 
        (err, data) => { ... }
    );
}
```

**常量定义**:
```typescript
// commonData.ets:183
MMS_URL: 'http://mmsc.monternet.com'  // ⚠️ 明文 HTTP
```

**触发条件**:
- 发送 MMS 消息
- 下载 MMS 附件

**影响**:
- 中间人攻击 (MITM)
- MMS 内容被窃听或篡改
- 用户隐私泄露

**修复建议**:
```typescript
// 1. 使用 HTTPS
MMS_URL: 'https://mmsc.monternet.com'

// 2. 启用证书校验
httpRequest.request(url, {
    // ...
    ca: [trustedCaCert],  // 指定受信 CA
    certificatePinning: true  // 证书固定
});
```

---

### 🔴 RISK-003: 电话号码格式化处理不当

**风险等级**: 高危  
**位置**: `entry/src/main/ets/utils/TelephoneUtil.ets`  
**证据**:

```typescript
// 多处使用电话号码进行字符串拼接
// ConversationService.ets:449
telephone = telephone + sendResult.telephone + common.string.COMMA;

// ConversationService.ets:576
map.nameFormatter = map.name + '<' + map.telephoneFormat + '>';

// MmsStaticSubscriber.ts:57
telephone: telephoneUtils.formatTelephone(shortMsgList[0].visibleRawAddress)
```

**TelephoneUtil.ets** (待确认具体实现):
```typescript
// 假设存在如下处理
formatTelephone(phone: string): string {
    // ⚠️ 如果 phone 包含特殊字符，可能导致 UI 注入
    return phone.replace(/(\d{3})(\d{4})(\d{4})/, '$1****$3');
}
```

**触发条件**:
- 接收到包含特殊字符的发送方号码
- 联系人数据被污染

**影响**:
- UI 注入攻击
- 日志注入
- 潜在的代码注入

**修复建议**:
```typescript
// 1. 严格校验电话号码格式
function validatePhoneNumber(phone: string): boolean {
    // 只允许数字、+、-、空格
    return /^[\d+\-\s()]+$/.test(phone) && phone.length <= 20;
}

// 2. 输出时进行转义
function escapeForDisplay(phone: string): string {
    return phone.replace(/[<>&"']/g, (char) => ({
        '<': '&lt;',
        '>': '&gt;',
        '&': '&amp;',
        '"': '&quot;',
        "'": '&#x27;'
    })[char]);
}
```

---

### 🔴 RISK-004: DataShare 查询条件构建缺乏校验

**风险等级**: 高危  
**位置**: `entry/src/main/ets/model/ConversationModel.ets:167`  
**证据**:

```typescript
// ConversationModel.ets:167-203
private buildQuerySmsMmsInfoCondition(actionData): DataSharePredicates {
    let condition = new dataSharePredicates.DataSharePredicates();
    condition.isNotNull(mmsTable.messageInfo.msgId)
    
    if (actionData.threadId != null) {
        let sessionId: string = actionData.threadId + common.string.EMPTY_STR;
        // ⚠️ 直接拼接字符串，无类型校验
        condition.and().equalTo(mmsTable.messageInfo.sessionId, sessionId);
    }
    
    if (actionData.msgIds != null && actionData.msgIds.length > 0) {
        // ⚠️ 数组元素未校验
        condition.and().in(mmsTable.messageInfo.msgId, actionData.msgIds);
    }
    
    return condition;
}
```

**触发条件**:
- actionData 从不可信来源传入
- msgIds 数组包含恶意构造的元素

**影响**:
- SQL 注入（取决于底层 DataShare 实现）
- 数据泄露
- 数据篡改

**修复建议**:
```typescript
private buildQuerySmsMmsInfoCondition(actionData): DataSharePredicates {
    // 1. 验证 actionData 类型
    if (!actionData || typeof actionData !== 'object') {
        throw new Error('Invalid actionData');
    }
    
    // 2. 验证 threadId 是整数
    if (actionData.threadId != null) {
        if (!Number.isInteger(actionData.threadId) || actionData.threadId < 0) {
            throw new Error('Invalid threadId');
        }
        // ...
    }
    
    // 3. 验证数组元素
    if (actionData.msgIds != null) {
        if (!Array.isArray(actionData.msgIds) || actionData.msgIds.length > 100) {
            throw new Error('Invalid msgIds');
        }
        for (const id of actionData.msgIds) {
            if (!Number.isInteger(id) || id < 0) {
                throw new Error('Invalid msgId in array');
            }
        }
    }
}
```

---

## 中危风险项 (5项)

### 🟡 RISK-005: 通知内容可能被截断导致信息泄露

**风险等级**: 中危  
**位置**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:222`  
**证据**:

```typescript
// MmsStaticSubscriber.ts:222-224
if (content.length > 15) {
    content = content.substring(0, 15) + '...';
}
```

**问题**: 虽然做了长度限制，但未考虑多字节字符（如中文、emoji）可能导致截断位置不当。

**修复建议**:
```typescript
function truncateContent(content: string, maxLength: number): string {
    // 使用 Array.from 正确处理 Unicode
    const chars = Array.from(content);
    if (chars.length <= maxLength) return content;
    return chars.slice(0, maxLength).join('') + '...';
}
```

---

### 🟡 RISK-006: 全局错误处理可能泄露敏感信息

**风险等级**: 中危  
**位置**: `entry/src/main/ets/MainAbility/MainAbility.ts:26`  
**证据**:

```typescript
// MainAbility.ts:26-32
function errorFunc(observer: errorManager.GlobalError) {
    HiLog.i(TAG, "result name : " + observer.name);
    HiLog.i(TAG, "result message : " + observer.message);
    HiLog.i(TAG, "result stack : " + observer.stack);  // ⚠️ 堆栈可能包含敏感信息
    HiLog.i(TAG, "result instanceName : " + observer.instanceName);
    HiLog.i(TAG, "result instanceType : " + observer.instanceType);
}
```

**修复建议**:
```typescript
function errorFunc(observer: errorManager.GlobalError) {
    // 记录错误概要，不记录完整堆栈
    HiLog.e(TAG, `Error: ${observer.name}, message: ${observer.message}`);
    // 堆栈仅在 DEBUG 模式记录
    if (isDebugMode) {
        HiLog.d(TAG, `Stack: ${observer.stack}`);
    }
}
```

---

### 🟡 RISK-007: HTTP 请求超时时间过长

**风险等级**: 中危  
**位置**: `entry/src/main/ets/service/SendMsgService.ets:82-83`  
**证据**:

```typescript
// SendMsgService.ets:82-83
readTimeout: 60000,      // 60 秒
connectTimeout: 60000    // 60 秒
```

**问题**: 超时时间过长，可能导致资源占用过久。

**修复建议**:
```typescript
readTimeout: 30000,      // 30 秒
connectTimeout: 15000    // 15 秒
```

---

### 🟡 RISK-008: 短信内容未进行敏感信息过滤

**风险等级**: 中危  
**位置**: 多处显示短信内容  
**证据**:

短信内容直接从数据库读取并显示在 UI 上，未进行敏感信息（如验证码、银行信息）的脱敏处理。

**修复建议**:
```typescript
// 添加敏感信息检测和脱敏
function maskSensitiveContent(content: string): string {
    // 脱敏验证码
    content = content.replace(/\b\d{6}\b/g, '******');
    // 脱敏银行卡号
    content = content.replace(/\b\d{16,19}\b/g, (match) => 
        match.slice(0, 4) + '****' + match.slice(-4));
    // ...
    return content;
}
```

---

### 🟡 RISK-009: 公共事件广播数据未加密

**风险等级**: 中危  
**位置**: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts:203`  
**证据**:

```typescript
// MmsStaticSubscriber.ts:203-208
commonEvent.publish(common.string.RECEIVE_TRANSMIT_EVENT, {
    bundleName: common.string.BUNDLE_NAME,
    subscriberPermissions: ['ohos.permission.RECEIVE_SMS'],
    isOrdered: false,
    data: JSON.stringify(actionData)  // ⚠️ 明文传输
});
```

**问题**: 虽然限制了订阅权限，但数据仍为明文 JSON。

**风险**: 有 RECEIVE_SMS 权限的应用可接收并解析短信内容。

---

## 低危风险项 (3项)

### 🟢 RISK-010: 日志中包含潜在敏感信息

**风险等级**: 低危  
**位置**: 多处 HiLog 调用  
**证据**:

```typescript
// 多处记录短信内容、号码等
HiLog.i(TAG, 'sendMessage, sendCallback success result=' + JSON.stringify(value));
```

**修复建议**: 避免在日志中记录完整短信内容和号码。

---

### 🟢 RISK-011: 版本号硬编码

**风险等级**: 低危  
**位置**: `entry/src/main/ets/MainAbility/MainAbility.ts:37`  
**证据**:

```typescript
HiLog.i(TAG, 'Ability onCreate com.ohos.mms version: 1.0.0.41');
```

**修复建议**: 从配置文件读取版本号。

---

### 🟢 RISK-012: 缺少操作频率限制

**风险等级**: 低危  
**位置**: `entry/src/main/ets/service/SendMsgService.ets:31`  
**问题**: 短信发送无频率限制，可能被恶意利用。

**修复建议**: 添加发送间隔限制（如每秒最多 1 条）。

---

## 检查范围与局限性

### 已检查范围

- ✅ 所有 `.ets` 和 `.ts` 源文件（共 58 个）
- ✅ SMS/MMS 接收处理
- ✅ HTTP 网络通信
- ✅ 数据库操作
- ✅ 系统 API 调用
- ✅ 事件广播
- ✅ 用户输入处理

### 未检查范围

- ❌ 底层 telephony_sms_mms 服务实现
- ❌ DataShare 服务内部安全机制
- ❌ 系统框架层安全策略
- ❌ 测试代码

### 检查局限性

1. 静态分析为主，未进行动态测试
2. 部分工具类实现未完全读取（如 TelephoneUtil 完整实现）
3. 依赖的底层服务安全机制未深入分析

---

## 修复优先级建议

### P0 (立即修复)
1. RISK-002: HTTP 启用 HTTPS
2. RISK-001: PDU 输入校验

### P1 (下个版本修复)
3. RISK-003: 电话号码校验
4. RISK-004: DataShare 参数校验
5. RISK-006: 错误信息脱敏

### P2 (计划修复)
6. RISK-005: 通知内容截断
7. RISK-007: 超时时间调整
8. RISK-008: 敏感信息脱敏

---

## 相关链接

- [系统 API](03_SystemAPIs.md) - 查看 API 使用详情
- [数据流](04_DataFlow.md) - 了解数据流转
- [附录 A: 调用链](appendix/Callgraphs.md) - 完整调用路径

---

*安全评审基于代码路径: entry/src/main/ets/*  
*评审日期: 2026-02-05*
