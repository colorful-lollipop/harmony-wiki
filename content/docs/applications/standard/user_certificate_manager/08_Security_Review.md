# 安全风险评审

> 本文档基于代码证据，对用户证书管理部件进行全面的安全风险评审

---

## 目的

本文档基于代码证据，识别用户证书管理部件的安全风险、可被利用点和修复建议。

## 适用范围

- OpenHarmony 安全工程师
- 代码审查人员
- 需要了解安全风险的技术人员

## 关键结论

1. **攻击面明确**: 主要攻击面为文件上传、密码输入、用户认证
2. **防御措施存在**: 有防截屏、用户认证等防御措施
3. **潜在风险**: 存在若干中等风险点，需要关注
4. **无证书导出**: 好消息：代码中无证书导出功能
5. **依赖系统服务**: 证书安全主要依赖系统服务

## 相关跳转

- [00_Overview.md](wiki/00_Overview.md) - 项目概览
- [03_Architecture.md](wiki/03_Architecture.md) - 架构设计
- [04_External_API.md](wiki/04_External_API.md) - 外部接口

---

## 1. 威胁模型

### 1.1 外部输入 → 敏感操作

```
┌─────────────────────────────────────────────────────────────┐
│                   外部输入源                           │
├─────────────────────────────────────────────────────────────┤
│  1. 文件上传 (用户选择的证书文件)                   │
│  2. 密码输入 (凭据密码)                            │
│  3. 用户认证 (生物识别/密码)                        │
│  4. Want 参数 (启动参数)                             │
└──────────────┬──────────────────────────────────────────┘
               ↓
┌─────────────────────────────────────────────────────────────┐
│                用户证书管理应用                         │
│  ┌────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │ 文件读取    │  │ 密码处理      │  │ 认证    │ │
│  └────────────┘  └──────────────┘  └─────────┘ │
└──────────────┬──────────────────────────────────────────┘
               ↓
┌─────────────────────────────────────────────────────────────┐
│              证书管理服务 (系统)                         │
│  - 证书安装                                         │
│  - 证书删除                                         │
│  - 授权管理                                         │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 信任边界

| 边界 | 描述 | 防御措施 |
|------|------|---------|
| 文件上传 → 应用 | 用户选择证书文件 | 文件后缀校验 |
| 应用 → 证书服务 | 证书操作通过 API | 权限检查、参数校验 |
| 用户 → 应用 | 用户交互 | 用户认证、防截屏 |
| 应用 → 其他应用 | 跨应用访问 | 权限检查 |

---

## 2. 攻击面清单

### 2.1 文件上传攻击面

| 入口点 | 证据 | 风险等级 |
|---------|------|---------|
| 文件选择器 | `CmFaPresenter.ets:16` | 中 |
| 文件读取 | `FileIoModel.ets:21-44` | 中 |
| 文件后缀解析 | `FileIoModel.ets:47-59` | 低 |

**威胁**:
- 恶意文件上传
- 路径遍历攻击
- 文件格式欺骗

**证据位置**: `certmanager/src/main/ets/model/FileIoModel.ets`, `certmanager/src/main/ets/presenter/CmFaPresenter.ets:64-93`

---

### 2.2 密码输入攻击面

| 入口点 | 证据 | 风险等级 |
|---------|------|---------|
| 凭据密码输入 | `certPwdInput.ets` | 高 |
| 密码存储 | `GlobalContext.ts:20-34` | 高 |

**威胁**:
- 密码泄露 (日志、内存)
- 密码被截屏
- 密码猜测攻击

**防御措施**:
- 防截屏机制 ✓
- 全局存储 (非持久化) ✓

**证据位置**: `certmanager/src/main/ets/model/PreventScreenshotsModel.ets`, `certmanager/src/main/ets/common/GlobalContext.ts:20-34`

---

### 2.3 用户认证攻击面

| 入口点 | 证据 | 风险等级 |
|---------|------|---------|
| 生物识别认证 | `CheckUserAuthModel.ets:61-117` | 中 |
| 密码认证 | `CheckUserAuthModel.ets:61-117` | 中 |

**威胁**:
- 绕过认证
- 重放攻击
- Challenge 可预测性

**防御措施**:
- 随机 Challenge ✓
- 认证结果验证 ✓

**证据位置**: `certmanager/src/main/ets/model/CheckUserAuthModel.ets:78-116`

---

### 2.4 IPC/权限攻击面

| 入口点 | 证据 | 风险等级 |
|---------|------|---------|
| 权限检查 | `module.json:84-115` | 中 |
| Extension Ability | `CertPickerUiExtAbility.ets:27-111` | 中 |

**威胁**:
- 权限提升
- 恶意调用
- 参数伪造

**证据位置**: `certmanager/src/main/module.json:84-115`, `certmanager/src/main/ets/MainAbility/CertPickerUiExtAbility.ets:36-73`

---

## 3. 可被利用点

### 3.1 密码明文存储风险 ⚠️

**证据位置**: `certmanager/src/main/ets/common/GlobalContext.ts:20-34`

**问题描述**:
```typescript
export class PwdStore {
  private certPwd: string = '';  // ⚠️ 明文存储

  setCertPwd(pwd: string): void {
    this.certPwd = pwd;
  }

  getCertPwd(): string {
    return this.certPwd;  // ⚠️ 返回明文
  }
}
```

**风险**: 密码以明文形式存储在内存中，可能被：
1. 日志泄露
2. 内存转储泄露
3. 调试器查看

**触发路径**:
1. 用户在凭据安装页面输入密码
2. 密码通过 `GlobalContext.getPwdStore().getCertPwd()` 获取
3. 传递给证书管理服务

**影响**: 密码可能被恶意应用或攻击者获取

**修复建议**:
1. 避免记录密码到日志
2. 尽快清除内存中的密码
3. 考虑使用 `SecureString` 或类似机制
4. 在使用后立即调用 `clearCertPwd()`

**当前状态**: ⚠️ 部分修复（有 `clearCertPwd()`）

---

### 3.2 文件读取无长度限制风险 ⚠️

**证据位置**: `certmanager/src/main/ets/model/FileIoModel.ets:21-44`

**问题描述**:
```typescript
getMediaFileData(mediaUri: string, callback: Function): void {
  // ...
  let stat = fs.statSync(file.fd);
  let buf = new ArrayBuffer(Number(stat.size));  // ⚠️ 无大小限制
  let num = fs.readSync(file.fd, buf);
  // ...
}
```

**风险**:
1. 攻击者可以上传超大文件，导致 OOM
2. 文件格式错误可能导致解析失败
3. 恶意文件可能导致解析器崩溃

**触发路径**:
1. 用户通过文件选择器选择超大文件
2. `FileIoModel.getMediaFileData()` 读取整个文件
3. 导致内存耗尽或应用崩溃

**影响**: DoS 攻击、应用崩溃

**修复建议**:
1. 添加文件大小限制 (如 10MB)
2. 添加超时机制
3. 使用流式读取而非一次性加载

```typescript
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
if (stat.size > MAX_FILE_SIZE) {
  callback(undefined);
  return;
}
```

---

### 3.3 证书安装无并发控制风险 ⚠️

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:697-730`

**问题描述**:
```typescript
private async installUserCertificate(data: Uint8Array, alias: string, optType: CMModelOptType, callback: Function): Promise<void> {
  // ...
  let result = await CertManager.installUserTrustedCertificate({
    inData: data,
    alias: alias,
    certFormat: certFormat,
    certScope: CertManager.CertScope.CURRENT_USER
  });
  // ⚠️ 无并发控制
}
```

**风险**:
1. 用户可以快速连续安装多个证书
2. 可能导致系统证书库溢出
3. 可能绕过数量限制检查

**触发路径**:
1. 用户连续触发多次证书安装
2. 每次安装独立执行，无互斥
3. 可能导致状态不一致

**影响**:
- 绕过 `CM_ERROR_MAX_CERT_COUNT_REACHED` 检查
- 系统证书库损坏
- 资源耗尽

**修复建议**:
1. 添加安装锁机制
2. 在安装前检查当前数量
3. 添加操作确认对话框

```typescript
private installing = false;

private async installUserCertificate(...) {
  if (this.installing) {
    callback(CMModelErrorCode.CM_MODEL_ERROR_FAILED);
    return;
  }
  this.installing = true;
  try {
    // 安装逻辑
  } finally {
    this.installing = false;
  }
}
```

---

### 3.4 错误信息泄露风险 ⚠️

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:719-728`

**问题描述**:
```typescript
catch (err) {
  let e: BusinessError = err as BusinessError;
  hilogError('installUserCertificate failed with err, message: ' + e.message + ', code: ' + e.code);
  // ⚠️ 直接输出错误信息到日志
  callback(CMModelErrorCode.CM_MODEL_ERROR_EXCEPTION, '');
}
```

**风险**:
1. 敏感错误信息可能被日志系统捕获
2. 可能泄露内部实现细节
3. 帮助攻击者了解系统结构

**触发路径**:
1. 任何证书操作失败
2. 错误信息被记录到 `hilog`
3. 攻击者可通过日志查看错误详情

**影响**: 信息泄露、系统结构暴露

**修复建议**:
1. 过滤敏感信息后再记录日志
2. 使用错误码而非详细信息
3. 在生产环境关闭详细日志

```typescript
catch (err) {
  let e: BusinessError = err as BusinessError;
  hilogError('installUserCertificate failed with code: ' + e.code);  // 仅记录错误码
  // 不记录详细消息
}
```

---

### 3.5 用户认证 Challenge 可预测风险 ⚠️

**证据位置**: `certmanager/src/main/ets/model/CheckUserAuthModel.ets:48-59`

**问题描述**:
```typescript
private getRandomData(): Uint8Array {
  let randData: Uint8Array;
  try {
    const rand: cryptoFramework.Random =  cryptoFramework.createRandom();
    const dataBlob: cryptoFramework.DataBlob = rand.generateRandomSync(RANDOM_DATA_LENGTH);
    randData = dataBlob.data;
  } catch (err) {
    hilogInfo('generate random failed');
    throw new Error('getRandomData failed.');
  }
  return randData;
}
```

**风险**:
1. 虽然使用了 `cryptoFramework.createRandom()`，但仅 16 字节
2. 如果随机数生成器实现不当，可能被预测
3. Challenge 长度可能不足

**触发路径**:
1. 用户触发证书安装
2. 调用 `auth()` 进行认证
3. 生成随机 Challenge
4. 攻击者可能通过分析预测 Challenge

**影响**:
- 重放攻击
- 认证绕过

**修复建议**:
1. 增加 Challenge 长度 (如 32 或 64 字节)
2. 添加 Challenge 的时间戳
3. 验证 Challenge 的唯一性

```typescript
const RANDOM_DATA_LENGTH = 32;  // 增加到 32 字节
```

---

## 4. 输入校验审查

### 4.1 文件后缀校验 ✅

**证据位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:68-73`

**状态**: ✓ 有校验

```typescript
if ((suffix === 'cer') || (suffix === 'pem') || (suffix === 'crt') || (suffix === 'der') ||
  (suffix === 'p7b') || (suffix === 'spc')) {
  // 允许的格式
} else {
  this.unrecognizedFileTips();
}
```

**评价**: 白名单校验，安全性较好

---

### 4.2 证书数据格式校验 ✅

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:699-703`

**状态**: ✓ 有校验

```typescript
if ((data === undefined) || (data.length === 0)) {
  callback(CMModelErrorCode.CM_MODEL_ERROR_INCORRECT_FORMAT);
  return;
}
```

**评价**: 有空值检查，但无格式深度验证

---

### 4.3 别名长度校验 ⚠️

**证据位置**: 无

**状态**: ✗ 无本地校验

**问题描述**: 代码中无别名长度校验，依赖证书管理服务返回错误

**修复建议**:
1. 在前端添加别名长度限制
2. 提示用户别名长度要求

---

## 5. 日志安全审查

### 5.1 敏感信息记录

| 文件 | 问题 | 证据 |
|------|------|------|
| `CertMangerModel.ets` | 记录错误详细信息 | `CertMangerModel.ets:719` |
| `FileIoModel.ets` | 记录文件路径 | `FileIoModel.ets:22` |

**建议**:
1. 避免记录文件完整路径
2. 避免记录错误详细信息
3. 使用脱敏机制

---

## 6. 内存安全审查

### 6.1 内存泄露风险

| 模块 | 问题 | 证据 |
|------|------|------|
| `GlobalContext` | 全局单例，生命周期长 | `GlobalContext.ts:36-105` |
| `PwdStore` | 密码未及时清除 | `GlobalContext.ts:20-34` |

**建议**:
1. 在使用密码后立即清除
2. 避免长期存储敏感信息

---

## 7. 修复优先级

| 优先级 | 风险点 | 工作量 | 影响 |
|--------|---------|--------|------|
| **高** | 文件读取无长度限制 | 低 | DoS 攻击 |
| **高** | 密码明文存储 | 中 | 密码泄露 |
| **中** | 证书安装无并发控制 | 中 | 绕过数量限制 |
| **中** | 错误信息泄露 | 低 | 信息泄露 |
| **低** | Challenge 长度不足 | 低 | 重放攻击 |

---

## 8. 未发现的风险

### 8.1 证书导出功能 ✅

**状态**: ✓ 良好

**证据**: 代码中无证书导出功能

**说明**: 所有证书操作都通过系统服务完成，无直接证书导出功能，降低了证书泄露风险。

---

### 8.2 路径遍历攻击 ✅

**状态**: ✓ 低风险

**证据**: `FileIoModel.ets` 使用 `fs.openSync()` 和文件 URI

**说明**: 使用系统文件 API，而非直接路径拼接，降低了路径遍历风险。

---

## 9. 安全建议总结

### 9.1 短期修复 (高优先级)

1. **添加文件大小限制** - 防止 DoS 攻击
2. **改进密码存储** - 使用安全存储机制
3. **添加并发控制** - 防止安装竞态

### 9.2 中期改进 (中优先级)

1. **过滤日志信息** - 避免敏感信息泄露
2. **增加前端校验** - 别名长度、格式校验
3. **增加 Challenge 长度** - 提高认证安全性

### 9.3 长期优化 (低优先级)

1. **安全审计** - 定期安全代码审查
2. **渗透测试** - 进行安全测试
3. **依赖更新** - 及时更新依赖

---

## 10. 检查范围与局限性

### 10.1 检查范围

✓ 证书管理逻辑 (`CertMangerModel.ets`)
✓ 用户认证逻辑 (`CheckUserAuthModel.ets`)
✓ 文件 I/O 逻辑 (`FileIoModel.ets`)
✓ 密码存储 (`GlobalContext.ts`)
✓ 权限声明 (`module.json`)
✓ 错误处理 (所有 Model 层)

### 10.2 未检查范围

✗ 证书管理服务实现 (`@ohos.security.certManager`)
✗ 用户认证服务实现 (`@ohos.userIAM.userAuth`)
✗ 测试代码 (已按规则排除)
✗ 第三方依赖 (hvigor/)

**说明**: 本评审仅覆盖应用层代码，系统服务实现不在范围内。

---

**END OF 08_Security_Review.md**
