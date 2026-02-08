# 安全风险评审

## 目的

本文档识别 PrintSpooler 项目的安全风险，包括攻击面分析、可被利用点和修复建议。

## 适用范围

本文档适用于：

- 安全工程师
- 安全审计人员
- 开发人员（了解安全风险）

## 关键结论

1. **主要攻击面**：文件操作、网络通信、权限使用
2. **输入校验不足**：文件路径、URI 参数缺乏充分校验
3. **权限过度**：申请了 9 个敏感权限
4. **信息泄露风险**：日志可能泄露敏感信息
5. **竞态条件**：并发任务管理可能存在竞态

---

## 威胁模型

### 数据流分析

```
[外部应用]
    ↓ (打印请求)
[PrintSpooler]
    ↓ (解析参数)
[文件系统]
    ↓ (打开文件)
[图像处理]
    ↓ (预览)
[用户界面]
    ↓ (设置参数)
[IPP 协议]
    ↓ (网络通信)
[打印机]
```

### 信任边界

| 边界 | 信任源 | 不信任源 |
|------|--------|---------|
| 应用自身 | 内部代码、常量 | 外部应用参数 |
| 文件系统 | 应用自身文件 | 用户传入的文件 URI |
| 打印服务 | 系统打印框架 | 网络打印机 |
| 用户操作 | UI 交互 | 外部输入 |

---

## 攻击面清单

### 1. 文件操作

**攻击面**：
- 接收外部传入的文件路径
- 读写文件内容
- 解析文件格式

**风险点**：
- 路径遍历攻击
- 文件权限绕过
- 恶意文件格式解析

**证据**：
- 文件打开：`entry/src/main/ets/Common/Utils/FileUtil.ts`
- 参数接收：`entry/src/main/ets/MainAbility/MainAbility.ets:81-96`

### 2. 网络通信

**攻击面**：
- IPP 协议通信
- Wifi P2P 连接
- mDNS 服务发现

**风险点**：
- 中间人攻击
- 恶意打印机
- 网络数据嗅探

**证据**：
- IPP 通信：`feature/ippPrint/src/main/ets/common/ipp/Backend.ts`
- P2P 连接：`feature/ippPrint/src/main/ets/common/connect/P2pPrinterConnection.ts`

### 3. 权限使用

**攻击面**：
- 文件访问权限
- WiFi 信息权限
- Bundle 信息权限

**风险点**：
- 权限滥用
- 信息泄露
- 权限提升

**证据**：
- 权限声明：`entry/src/main/module.json5:66-161`

### 4. 参数传递

**攻击面**：
- Ability Want 参数
- 打印机 URI
- 打印属性 JSON

**风险点**：
- 参数注入
- JSON 解析异常
- 类型混淆

**证据**：
- 参数解析：`entry/src/main/ets/MainAbility/MainAbility.ets:81-96`

### 5. 日志输出

**攻击面**：
- 文件内容日志
- 用户信息日志
- 打印机信息日志

**风险点**：
- 敏感信息泄露
- 日志注入
- 存储空间耗尽

**证据**：
- 日志使用：所有模块使用 `Log.info/error/debug`

---

## 可被利用点（至少 5 条）

### 利用点 1：路径遍历漏洞

**位置**：`entry/src/main/ets/MainAbility/MainAbility.ets:81-96`

**证据**：
```typescript
// MainAbility.ets:88-89
fileList = want.parameters[Constants.WANT_FILE_LIST_KEY] as string[];
callerPid = want.parameters[Constants.WANT_CALLERPID_KEY] as string;
pkgName = want.parameters[Constants.WANT_PKG_NAME_KEY] as string;
docName = want.parameters[Constants.WANT_DOCUMENT_NAME_KEY] as string;
```

**问题**：
- 直接使用 `want.parameters` 中的文件路径
- 未对文件路径进行校验
- 可能传入 `../../etc/passwd` 等恶意路径

**触发路径**：
1. 外部应用调用打印 API
2. 传入恶意文件路径（如 `file:///data/../../sensitive_file`）
3. PrintSpooler 直接使用该路径打开文件

**影响**：
- 读取系统敏感文件
- 信息泄露
- 可能导致权限提升

**修复建议**：
```typescript
// 添加路径校验
function validateFilePath(filePath: string): boolean {
  // 检查路径遍历
  if (filePath.includes('..')) {
    return false;
  }
  // 检查路径白名单
  if (!filePath.startsWith('/data/')) {
    return false;
  }
  return true;
}

// 在使用前校验
if (fileList && fileList.length > 0) {
  for (let path of fileList) {
    if (!validateFilePath(path)) {
      Log.error(TAG, 'Invalid file path: ' + path);
      return; // 拒绝处理
    }
  }
}
```

---

### 利用点 2：URI 注入漏洞

**位置**：`feature/ippPrint/src/main/ets/common/napi/NativeApi.ts:43`

**证据**：
```typescript
// NativeApi.ts:43
print.queryPrinterCapabilityByUri(uri, id).then((result) => {
  Log.debug(TAG, 'queryPrinterCapabilityByUri result: ' + JSON.stringify(result));
  // ...
});
```

**问题**：
- 直接使用传入的 `uri` 参数
- 未校验 URI 格式和内容
- 可能传入恶意 URI（如 `http://evil.com/`）

**触发路径**：
1. 外部应用传入恶意打印机 URI
2. PrintSpooler 直接使用该 URI 查询能力
3. 恶意服务器返回响应，可能注入恶意代码

**影响**：
- SSRF（服务端请求伪造）
- 信息泄露
- 可能执行恶意代码

**修复建议**：
```typescript
function validatePrinterUri(uri: string): boolean {
  // 校验 URI 格式
  if (!uri.startsWith('ipp://') && !uri.startsWith('ipps://')) {
    return false;
  }
  // 校验 IP 地址范围
  let ipPattern = /ipp:\/\/(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})/;
  if (!ipPattern.test(uri)) {
    return false;
  }
  return true;
}

// 在使用前校验
if (!validatePrinterUri(uri)) {
  Log.error(TAG, 'Invalid printer URI: ' + uri);
  callback(ERROR);
  return;
}
```

---

### 利用点 3：JSON 注入漏洞

**位置**：`entry/src/main/ets/MainAbility/MainAbility.ets:94-99`

**证据**：
```typescript
// MainAbility.ets:94
attributes = want.parameters[Constants.wantPrintAttributeKey] as string;

// MainAbility.ets:98-99
if (!CheckEmptyUtils.checkStrIsEmpty(attributes)) {
  printAttributes = JSON.parse(attributes);
}
```

**问题**：
- 直接使用 `JSON.parse()` 解析外部输入
- 未捕获解析异常（虽然 try-catch 存在）
- 未校验 JSON 结构和内容

**触发路径**：
1. 外部应用传入恶意 JSON 字符串
2. `JSON.parse()` 可能解析失败或注入恶意内容
3. 应用异常或行为异常

**影响**：
- 应用崩溃
- 数据注入
- 逻辑绕过

**修复建议**：
```typescript
function validatePrintAttributes(attributes: string): boolean {
  try {
    let attrs = JSON.parse(attributes);
    // 校验必要字段
    if (!attrs.colorMode || !attrs.duplex || !attrs.pageSize) {
      return false;
    }
    // 校验字段类型和范围
    if (typeof attrs.colorMode !== 'number' ||
        attrs.colorMode < 0 || attrs.colorMode > 1) {
      return false;
    }
    return true;
  } catch (e) {
    return false;
  }
}

// 在使用前校验
if (attributes && !validatePrintAttributes(attributes)) {
  Log.error(TAG, 'Invalid print attributes: ' + attributes);
  return; // 使用默认值
}
```

---

### 利用点 4：竞态条件

**位置**：`entry/src/main/ets/Controller/PrintJobController.ets:177-201`

**证据**：
```typescript
// PrintJobController.ets:177-201
private onJobStateChanged = (state: print.PrintJobState, job: print.PrintJob): void => {
  if (state === null || job === null) {
    Log.error(TAG, 'device state changed null data');
    return;
  }
  this.deleteLocalSource(<number>state, <string>job.jobId);
  switch (state) {
    case PrintJobState.PRINT_JOB_PREPARED:
    case PrintJobState.PRINT_JOB_QUEUED:
    case PrintJobState.PRINT_JOB_RUNNING:
    case PrintJobState.PRINT_JOB_BLOCKED:
    case PrintJobState.PRINT_JOB_COMPLETED:
      this.onPrintJobStateChange(job);
      break;
    default:
      break;
  }
};
```

**问题**：
- 多个任务状态变化同时发生时
- `deleteLocalSource` 和 `onPrintJobStateChange` 可能竞态
- 未使用锁或原子操作

**触发路径**：
1. 多个打印任务同时运行
2. 状态变化事件并发触发
3. 任务队列数据可能不一致

**影响**：
- 任务状态混乱
- 数据损坏
- UI 显示错误

**修复建议**：
```typescript
import { Worker } from '@ohos.worker';

// 使用 Worker 处理状态变化
class PrintJobStateHandler {
  private stateQueue: Queue<JobStateEvent> = new Queue();

  onJobStateChanged(state: print.PrintJobState, job: print.PrintJob): void {
    // 将状态事件加入队列
    this.stateQueue.push({ state, job });

    // 异步处理队列
    this.processQueue();
  }

  private processQueue(): void {
    // 原子性地处理队列
    while (!this.stateQueue.isEmpty()) {
      let event = this.stateQueue.pop();
      // 处理单个事件
      this.onPrintJobStateChange(event.job);
    }
  }
}
```

---

### 利用点 5：敏感信息泄露

**位置**：多处日志输出

**证据**：
```typescript
// NativeApi.ts:44
Log.debug(TAG, 'queryPrinterCapabilityByUri result: ' + JSON.stringify(result));

// MainAbility.ets:79
Log.info(TAG + 'onSessionCreate, want: ' + JSON.stringify(want));

// PrintJobController.ets:178
Log.error(TAG, 'device state changed null data');
```

**问题**：
- 直接输出完整的 `want` 对象
- 输出打印机能力信息（可能包含敏感配置）
- 未对敏感信息进行脱敏处理

**触发路径**：
1. 攻击者获取日志访问权限
2. 读取应用日志
3. 获取敏感信息（用户文件、打印机配置等）

**影响**：
- 用户隐私泄露
- 系统配置泄露
- 符合安全合规性问题

**修复建议**：
```typescript
// 定义脱敏函数
function sanitizeWant(want: Want): any {
  let sanitized = {
    parameters: {
      [Constants.WANT_FILE_LIST_KEY]: '[REDACTED]', // 文件列表脱敏
      [Constants.WANT_DOCUMENT_NAME_KEY]: want.parameters[Constants.WANT_DOCUMENT_NAME_KEY],
      [Constants.WANT_JOB_ID_KEY]: want.parameters[Constants.WANT_JOB_ID_KEY],
      // ... 其他非敏感字段
    }
  };
  return sanitized;
}

// 使用脱敏后的数据
Log.info(TAG + 'onSessionCreate, want: ' + JSON.stringify(sanitizeWant(want)));
```

---

## 其他潜在风险

### 6. 权限过度申请

**证据**：`entry/src/main/module.json5:66-161`

**问题**：
- 申请了 `ohos.permission.GET_RUNNING_INFO` - 可能获取其他应用运行信息
- 申请了 `ohos.permission.securityguard.REPORT_SECURITY_INFO` - 可能上报敏感信息

**建议**：
- 评估每个权限的必要性
- 移除不必要的权限
- 文档说明权限使用场景

### 7. 缺少权限校验

**证据**：未找到显式的权限检查代码

**问题**：
- 未在代码中检查权限是否已授予
- 直接使用需要权限的 API

**建议**：
```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';

// 在使用权限前检查
async function checkPermission(permission: string): Promise<boolean> {
  let atManager = abilityAccessCtrl.createAtManager();
  let tokenID = application.getContext().applicationInfo.accessTokenId;
  let permissionStates = await atManager.checkAccessToken(tokenID, permission);
  return permissionStates.grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
}

// 使用前检查
if (await checkPermission('ohos.permission.GET_WIFI_INFO')) {
  // 使用 WiFi API
} else {
  // 提示用户授予权限
}
```

---

## 安全加固建议

### 1. 输入校验

- ✅ 所有外部输入必须进行格式校验
- ✅ 所有文件路径必须防止路径遍历
- ✅ 所有 URI 必须校验格式和协议

### 2. 输出脱敏

- ✅ 日志输出必须脱敏敏感信息
- ✅ 错误消息不能泄露系统信息
- ✅ 文件内容不能记录到日志

### 3. 权限最小化

- ✅ 仅申请必要的权限
- ✅ 使用前检查权限授予状态
- ✅ 文档说明权限使用场景

### 4. 加密通信

- ✅ 优先使用 IPPS（加密 IPP）而非 IPP
- ✅ 验证打印机证书
- ✅ 防止中间人攻击

### 5. 错误处理

- ✅ 所有异常必须妥善处理
- ✅ 避免在错误中泄露系统信息
- ✅ 提供用户友好的错误提示

---

## 检查范围

### 已检查的范围

- ✅ Ability 参数传递和解析
- ✅ 文件操作和路径处理
- ✅ IPP 协议通信
- ✅ Wifi P2P 连接
- ✅ 权限声明和使用
- ✅ 日志输出
- ✅ 任务状态管理

### 未检查的范围

- ❌ CUPS 和 SANE 后端实现（系统服务，非本项目代码）
- ❌ Wifi 驱动层实现（系统服务）
- ❌ 打印机固件安全性

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目整体介绍
- [对外 API](04_External_API.md) - OpenHarmony 系统 API 使用
- [架构设计](03_Architecture.md) - 组件关系和数据流
