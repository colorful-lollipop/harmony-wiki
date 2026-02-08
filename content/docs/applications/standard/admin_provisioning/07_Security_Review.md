# 07_安全风险评审

> 本文档基于代码证据编写，证据来源见各章节引用。

## 评审范围

| 范围 | 说明 |
|-----|-----|
| **评审对象** | admin_provisioning HAP 应用 |
| **代码范围** | `entry/src/main/ets/` 下所有 ArkTS 源码 |
| **排除范围** | 测试代码、构建脚本、第三方签名文件 |

---

## 1. 攻击面分析

### 1.1 外部输入点

| 输入源 | 接收方式 | 数据类型 | 用途 | 证据文件 |
|-------|---------|---------|-----|---------|
| Want 参数 | `onCreate(want)` | Want | 启动参数传递 | `MainAbility.ts:24` |
| Want.parameters | `storage.get()` | any | 企业信息、管理员信息 | `applicationInfo.ets:337` |
| 配置文件 | `configPolicy.getOneCfgFile()` | JSON | 企业配置 | `UIExtensionAbility.ets:43` |
| 用户交互 | `onClick()` | Event | 页面点击事件 | `managerStart.ets:135` |

### 1.2 敏感操作

| 操作 | API | 权限要求 | 风险等级 |
|-----|-----|---------|---------|
| 启用管理员 | `adminManager.enableAdmin()` | MANAGE_ENTERPRISE_DEVICE_ADMIN | 高 |
| 禁用管理员 | `adminManager.disableAdmin()` | MANAGE_ENTERPRISE_DEVICE_ADMIN | 高 |
| 恢复出厂设置 | `update.factoryReset()` | FACTORY_RESET | 高 |
| 获取账户信息 | `accountManager.queryCurrentOsAccount()` | MANAGE_LOCAL_ACCOUNTS | 中 |
| 读取配置文件 | `fs.readTextSync()` | 无 | 低 |

---

## 2. 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  应用内部 (信任)                      │    │
│  │  ├── Logger (日志工具)                              │    │
│  │  ├── Utils (工具函数)                               │    │
│  │  ├── BaseData (常量定义)                            │    │
│  │  └── LocalStorage (页面间数据)                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                            ↓                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   系统 API (边界)                     │    │
│  │  ├── @ohos.enterprise.adminManager                  │    │
│  │  ├── @ohos.bundle.bundleManager                     │    │
│  │  ├── @ohos.account.osAccount                        │    │
│  │  └── @ohos.update                                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                            ↓                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                 外部输入 (不可信)                      │    │
│  │  ├── Want 参数 (来自系统或其他应用)                   │    │
│  │  ├── 配置文件 (edm_provision_config.json)          │    │
│  │  └── 用户界面输入                                    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 已识别安全风险

### 3.1 Want 参数校验缺失

**风险编号**: SEC-001

**风险等级**: 中

**描述**: `MainAbility` 和 `AutoManagerAbility` 从 `want.parameters` 提取敏感信息，但存在参数缺失时直接调用 `terminateSelf()` 而非返回错误。

**证据** (`applicationInfo.ets:339-347`):

```typescript
if (!utils.checkObjPropertyValid(data, 'parameters.elementName.abilityName') ||
    !utils.checkObjPropertyValid(data, 'parameters.enterprise.name') ||
    !utils.isValid((data.parameters?.elementName as Record<string, string>).bundleName) ||
    !utils.isValid(data.parameters?.activeType') ||
    !utils.isValid((data.parameters?.enterprise as Record<string, string>).description)) {
  logger.warn(TAG, 'data.parameters = ' + JSON.stringify(data.parameters));
  (getContext(this) as common.UIAbilityContext).terminateSelf();
  return;
}
```

**潜在问题**:
- 参数校验失败后直接终止应用，未返回有意义的错误码
- 恶意调用可能导致应用意外终止

**修复建议**:
- 返回明确的错误码给调用方
- 记录详细的审计日志

---

### 3.2 字符串长度限制不一致

**风险编号**: SEC-002

**风险等级**: 低

**描述**: 企业名称和描述的长度限制为 30 字符 (`baseData.MAX_LEN = 30`)，但此限制在 UI 层而非 API 层实施。

**证据** (`baseData.ets:17`):

```typescript
export class BaseData {
  public MAX_LEN = 30;
}
```

**证据** (`applicationInfo.ets:355-357`):

```typescript
enterInfo.name = (data.parameters?.enterprise as Record<string, string>).name
  .substring(0, baseData.MAX_LEN);
enterInfo.description =
  (data.parameters?.enterprise as Record<string, string>).description
  .substring(0, baseData.MAX_LEN);
```

**潜在问题**:
- 超出长度的数据被静默截断，可能丢失重要信息
- 截断行为可能在 API 层未实施

**修复建议**:
- 在 API 入口处实施长度校验
- 返回截断警告

---

### 3.3 配置文件路径硬编码

**风险编号**: SEC-003

**风险等级**: 低

**描述**: 配置文件路径硬编码为 `'etc/edm/edm_provision_config.json'`，缺乏路径遍历防护。

**证据** (`UIExtensionAbility.ets:42`):

```typescript
let realpath = 'etc/edm/edm_provision_config.json';
await configPolicy.getOneCfgFile(realpath).then((value: string) => {
```

**潜在问题**:
- `configPolicy.getOneCfgFile()` 通常有路径白名单机制
- 但如机制失效，可能导致任意文件读取

**修复建议**:
- 验证路径是否符合预期前缀
- 使用标准化的配置文件读取 API

---

### 3.4 敏感信息日志泄露

**风险编号**: SEC-004

**风险等级**: 中

**描述**: 应用可能将敏感信息（如企业名称、描述）输出到日志。

**证据** (`UIExtensionAbility.ets:44-46`):

```typescript
await configPolicy.getOneCfgFile(realpath).then((value: string) => {
  logger.info(TAG, 'getOneCfgFile value is : ' + value);
  let configStr = fs.readTextSync(value);
```

**潜在问题**:
- 日志可能被导出或被非授权人员查看
- 企业敏感信息泄露

**修复建议**:
- 避免记录完整的企业信息
- 对敏感字段进行脱敏处理

---

### 3.5 管理员类型校验缺失

**风险编号**: SEC-005

**风险等级**: 高

**描述**: `MainAbility` 调用 `enableAdmin` 时仅支持 `ADMIN_TYPE_NORMAL`，不支持 `ADMIN_TYPE_SUPER`，但此限制在代码中以警告形式输出，可能被绕过。

**证据** (`applicationInfo.ets:302-304`):

```typescript
} else {
  logger.warn(TAG, 'not support AdminType.ADMIN_TYPE_SUPER enable Admin')
}
```

**潜在问题**:
- 如果存在其他调用路径，可能激活超级管理员
- 权限升级风险

**修复建议**:
- 明确限制超级管理员的激活入口
- 实施 API 级别的权限检查

---

## 4. 安全机制分析

### 4.1 权限模型

| 权限 | 用途 | 声明位置 | 实施方式 |
|-----|-----|---------|---------|
| `MANAGE_LOCAL_ACCOUNTS` | 账户管理 | module.json5 | 系统强制 |
| `GET_BUNDLE_INFO` | Bundle 查询 | module.json5 | 系统强制 |
| `MANAGE_ENTERPRISE_DEVICE_ADMIN` | 管理员管理 | module.json5 | 系统强制 |
| `FACTORY_RESET` | 恢复出厂 | module.json5 | 系统强制 |
| `INTERNET` | 网络访问 | module.json5 | 系统强制 |
| `PROVISIONING_MESSAGE` | 发放消息 | module.json5:48 | AutoManagerAbility 专属 |

**证据** (`module.json5:63-79`):

```json5
"requestPermissions": [
  { "name": "ohos.permission.MANAGE_LOCAL_ACCOUNTS" },
  { "name": "ohos.permission.GET_BUNDLE_INFO" },
  { "name": "ohos.permission.MANAGE_ENTERPRISE_DEVICE_ADMIN" },
  { "name": "ohos.permission.FACTORY_RESET" },
  { "name": "ohos.permission.INTERNET" }
]
```

### 4.2 输入验证

**验证函数** (`utils.ts:21-40`):

```typescript
class Utils {
  // 对象有效性检查
  isValid(item: unknown): boolean {
    return item !== null && item !== undefined;
  }

  // 嵌套属性检查
  checkObjPropertyValid<T>(obj: T, tree: string): boolean {
    let arr = tree.split('.');
    let tempObj = obj;
    for (let i = 0; i < arr.length; i++) {
      if (!this.isValid(tempObj[arr[i]])) {
        return false;
      }
      tempObj = tempObj[arr[i]];
    }
    return true;
  }
}
```

**验证覆盖**:
- ✅ Want 参数有效性检查
- ✅ 对象属性存在性检查
- ✅ 字符串非空检查
- ⚠️ 字符串长度限制（仅 UI 层）
- ⚠️ JSON 格式验证（依赖 JSON.parse 自动验证）

### 4.3 日志安全

**日志域**: `0x6977`

**日志级别**: debug, info, warn, error

**证据** (`logger.ts:18-26`):

```typescript
class Logger {
  private domain: number;
  private prefix: string;
  private format: string = '%{public}s, %{public}s';

  constructor(prefix: string) {
    this.prefix = prefix;
    this.domain = 0x6977;
  }
}
```

**注意**: `hilog` 使用 `%{public}s` 格式，可能泄露敏感信息。

---

## 5. 安全建议

### 5.1 高优先级

| 建议 | 原因 | 实施位置 |
|-----|-----|---------|
| 完善错误码机制 | 当前错误处理过于简单 | `applicationInfo.ets` |
| API 级权限检查 | 防止权限绕过 | `@ohos.enterprise.adminManager` 调用处 |

### 5.2 中优先级

| 建议 | 原因 | 实施位置 |
|-----|-----|---------|
| 日志脱敏 | 防止敏感信息泄露 | `logger` 调用处 |
| 配置文件完整性校验 | 防止配置篡改 | `UIExtensionAbility.ets` |

### 5.3 低优先级

| 建议 | 原因 | 实施位置 |
|-----|-----|---------|
| 配置文件路径白名单 | 防止路径遍历 | `UIExtensionAbility.ets` |
| 统一长度校验 | 防止数据截断 | `baseData.ets` |

---

## 6. 总结

### 风险统计

| 风险等级 | 数量 | 说明 |
|---------|-----|-----|
| 高 | 1 | 管理员类型校验缺失 |
| 中 | 2 | Want 参数校验、日志泄露 |
| 低 | 2 | 长度限制、路径硬编码 |

### 总体评估

| 评估项 | 等级 | 说明 |
|-------|-----|-----|
| 权限控制 | 良好 | 依赖系统权限机制 |
| 输入验证 | 一般 | 基础验证完整，深度不足 |
| 错误处理 | 待改进 | 错误码机制缺失 |
| 日志安全 | 待改进 | 敏感信息可能泄露 |
| 配置安全 | 一般 | 依赖系统配置机制 |

### 改进建议

1. **实施完整的错误码机制**: 为每个失败场景定义明确的错误码
2. **加强日志脱敏**: 对敏感字段进行脱敏处理
3. **API 层面实施长度校验**: 将字符串长度限制从 UI 层移到 API 层
4. **增加审计日志**: 记录管理员的启用/禁用操作

---

*文档版本: 1.0*
