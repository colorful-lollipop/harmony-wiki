# 20_Security_Review - 安全风险评审

## 概述

本文档对 Authentication Widget 进行安全风险评审，基于代码证据识别潜在攻击面和安全风险。

**评审范围**: `entry/src/main/ets/` 下所有源代码（不含测试）

**评审方法**: 静态代码分析 + 威胁建模

## 攻击面分析

### 外部输入点

**证据**: `Index.ets`, `UserAuthAbility.ts`, `AuthUtils.ts` 分析

| 输入源 | 输入类型 | 处理位置 | 信任级别 |
|--------|----------|----------|----------|
| `Want.parameters` | 认证参数 (WantParams) | `UserAuthAbility.ts:50` | 高 |
| `userAuthWidgetMgr.on('command')` | 认证指令 (WidgetCommand) | `Index.ets:176` | 高 |
| `PINAuth.onGetData` | 密码数据 | `PasswordAuth.ets:105` | 高 |
| `screen.getAllScreens` | 屏幕信息 | `Index.ets:56` | 中 |
| AppStorage | 持久化状态 | 全局共享 | 低 |

### 信任边界

```
┌──────────────────────────────────────────────────────────────┐
│                    信任边界（高信任区）                         │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ user_auth_framework (IPC 通信)                         │  │
│  │ ├── Want 参数验证                                       │  │
│  │ └── WidgetCommand 验证                                  │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              │                                │
│                              ▼                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ UserAuthExtensionAbility                                │  │
│  │ ├── 参数解析 AppStorage.setOrCreate()                   │  │
│  │ └── UIExtensionContentSession 管理                      │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              │                                │
│                              ▼                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ Index 页面                                               │  │
│  │ ├── 参数验证 getParams()                                │  │
│  │ ├── 类型映射 authType = UserAuthType[]                 │  │
│  │ └── 状态管理 @State                                     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              │                                │
│                              ▼                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ 认证组件 (FaceAuth/FingerprintAuth/PasswordAuth)        │  │
│  │ ├── 用户输入处理                                         │  │
│  │ └── AuthUtils.sendNotice()                              │  │
│  └─────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

## 安全风险清单

### 风险 1: Want 参数未完全验证

**证据**: `UserAuthAbility.ts:50` 和 `Index.ets:236-258`

**描述**: `WantParams` 从 `want.parameters.useriamCmdData` 直接解析，未验证必填字段完整性。

**触发条件**:
```typescript
// UserAuthAbility.ts:50
AppStorage.setOrCreate('wantParams', want?.parameters?.useriamCmdData);
```

```typescript
// Index.ets:236-258
getParams(result: WantParams): void {
  this.pinSubType = resultInfo?.pinSubType;  // 可为 undefined
  this.authType = newType;                    // 可为 undefined
  // ...
}
```

**影响**:
- 可能导致 UI 渲染异常
- 认证类型未定义时的行为未定义
- 可能触发 TypeError

**修复建议**:
```typescript
getParams(result: WantParams): void {
  if (!result || !result.type || result.type.length === 0) {
    LogUtils.error(TAG, 'Invalid wantParams: missing type');
    (AppStorage.get('session') as UIExtensionContentSession)?.terminateSelf();
    return;
  }
  // 继续正常处理
}
```

### 风险 2: 敏感信息日志泄露

**证据**: `LogUtils.ts` 和 `PasswordAuth.ets:47-48`

**描述**: PIN 密码数据通过日志工具输出，可能泄露敏感认证信息。

**触发条件**:
```typescript
// PasswordAuth.ets:47-48
LogUtils.info(TAG, 'aboutToAppear registerInputer onGetData');
const uint8PW = FuncUtils.getUint8PW(pinData);
```

**影响**:
- 调试日志可能包含敏感密码数据
- 生产环境日志泄露认证凭据

**修复建议**:
- 在生产版本禁用敏感日志
- 使用 `DEBUG` 级别而非 `INFO` 级别输出调试信息
- 对敏感数据脱敏后输出

### 风险 3: AppStorage 跨组件数据污染

**证据**: `UserAuthAbility.ts:32` 和 `AuthUtils.ts:26`

**描述**: 使用 AppStorage 全局共享 `widgetContextId` 等敏感数据，可能被其他组件读取。

**触发条件**:
```typescript
// UserAuthAbility.ts:32
AppStorage.setOrCreate('context', this.context);

// AuthUtils.ts:26
private widgetContextId: bigint = BigInt(AppStorage.get<string>('widgetContextId')) ?? BigInt(-1);
```

**影响**:
- 恶意应用可能读取 `widgetContextId`
- 状态数据可能被意外覆盖
- 多实例场景下数据混淆

**修复建议**:
- 使用组件实例状态管理替代全局 AppStorage
- 对敏感数据加密后存储
- 添加数据来源验证

### 风险 4: PIN 输入缓冲区未清零

**证据**: `PasswordAuth.ets:35, 118-123`

**描述**: PIN 密码数据存储在全局变量 `pinData`，`clearPassword()` 仅清空字符串引用。

**触发条件**:
```typescript
// PasswordAuth.ets:35
let pinData = '';

// PasswordAuth.ets:118-123
clearPassword(): void {
  this.textValue = '';
  this.inputValue = '';
  pinData = '';  // 只是赋值新字符串，旧数据可能仍在内存中
}
```

**影响**:
- 密码数据可能残留在内存中
- GC 未及时回收时敏感数据泄露风险

**修复建议**:
```typescript
clearPassword(): void {
  // 使用 ArrayBuffer 替代字符串存储敏感数据
  const buf = new ArrayBuffer(pinData.length);
  const view = new Uint8Array(buf);
  view.fill(0);  // 清零
  this.textValue = '';
  this.inputValue = '';
  pinData = '';
}
```

### 风险 5: command 回调缺乏验证

**证据**: `Index.ets:176-191`

**描述**: `userAuthWidgetMgr.on('command')` 回调直接解析 JSON，缺乏 schema 验证。

**触发条件**:
```typescript
// Index.ets:176-191
userAuthWidgetMgr.on('command', {
  sendCommand: (result) => {
    const cmdDataObj: WidgetCommand = JSON.parse(result || '{}');  // 无验证
    this.skipLockedBiometricAuth = cmdDataObj?.skipLockedBiometricAuth ?? false
    this.handleCmdDataObjItems(cmdDataObj);
    // ...
  }
});
```

**影响**:
- 恶意框架消息可能导致解析错误
- 异常数据可能导致状态不一致
- 拒绝服务风险

**修复建议**:
```typescript
sendCommand: (result) => {
  if (!result) return;
  try {
    const cmdDataObj = JSON.parse(result);
    // Schema 验证
    if (!cmdDataObj.cmd || !Array.isArray(cmdDataObj.cmd)) {
      LogUtils.error(TAG, 'Invalid command format');
      return;
    }
    // 继续处理
  } catch (e) {
    LogUtils.error(TAG, 'Command parse error: ' + e);
  }
}
```

### 风险 6: UI 注入风险（低风险）

**证据**: `Index.ets:265-266`

**描述**: 通过资源管理器获取的字符串可能包含用户可控内容。

**触发条件**:
```typescript
// Index.ets:265-266
Text((AppStorage.get('context') as common.ExtensionContext)?.resourceManager
  .getStringSync($r('app.string.unified_authwidget_tip_verify_in_portrait_mode').id))
```

**评估**: 低风险，因为字符串来自预定义资源文件 (`resources/base/element/string.json`)，非运行时输入。

## 安全建议总结

| 风险 | 严重程度 | 建议优先级 |
|------|----------|------------|
| Want 参数未验证 | 中 | 高 |
| 敏感信息日志泄露 | 中 | 高 |
| AppStorage 数据污染 | 低 | 中 |
| PIN 输入缓冲区未清零 | 低 | 中 |
| command 回调缺乏验证 | 中 | 高 |

## 相关文档

- [概览](00_Overview.md) - 项目定位与运行环境
- [架构说明](01_Architecture.md) - 组件间数据流
- [认证框架 API](02_UserAuth_API.md) - API 安全考虑
- [常见问题](90_FAQ.md) - 安全相关 FAQ
