# 安全风险评审

## 概述

本文档对 ability_cangjie_wrapper 子系统进行安全风险评审，识别攻击面、信任边界，并提供可被利用点分析与修复建议。

**评审范围**：本仓库的仓颉封装层代码（`ohos/`、`kit/` 目录），不包括原生实现（`ability_runtime` 等子系统）。

---

## 攻击面清单

### 1. N-API 边界

**位置**：`context_interop.cj:26-140`

**描述**：Context 对象在仓颉与 ArkTS 之间双向转换

| 接口 | 功能 | 风险等级 |
|-----|------|---------|
| `FfiConvertBaseContext2Napi()` | Cangjie → ArkTS | 高 |
| `FfiCreateBaseContextFromNapi()` | ArkTS → Cangjie | 高 |
| `toJSValue()` | 上下文值转换 | 高 |
| `create*FromJSValue()` | 从 JS 值创建 Cangjie 对象 | 高 |

**证据来源**：`context_interop.cj:26-40`

### 2. FFI 函数调用

**位置**：所有 `foreign func` 声明

**描述**：50+ 个外部函数调用，连接仓颉代码与原生 runtime

| 模块 | FFI 函数数量 | 主要用途 |
|-----|-------------|---------|
| ui_ability | 20+ | 生命周期管理、上下文操作 |
| want | 5 | Want 对象创建与解析 |
| ability_delegator_registry | 15+ | 测试框架操作 |
| error_manager | 3 | 错误观察注册 |
| app_recovery | 4 | 应用恢复功能 |

**证据来源**：`ui_ability.cj:38-55`、`want.cj:46-56`、`ability_delegator_registry.cj:48-72`

### 3. Shell 命令执行

**位置**：`ability_delegator_registry.cj:441-447`

**描述**：`executeShellCommand()` 直接执行用户输入的命令

| 接口 | 参数 | 风险 |
|-----|------|-----|
| `executeShellCommand(cmd: String, timeoutSecs: Int64)` | `cmd`: shell 命令字符串 | **严重** |

**证据来源**：`ability_delegator_registry.cj:441-447`

### 4. URI 解析

**位置**：`want.cj:54,116,247,283`

**描述**：Want 对象中 URI 字段的解析与传递

| 字段 | 说明 | 风险 |
|-----|------|-----|
| `uri: String` | 外部传入的 URI 字符串 | 中 |
| `FFICJWantParseUri()` | 原生 URI 解析 | 中 |

**证据来源**：`want.cj:54` - `FFICJWantParseUri` 声明

### 5. 文件系统访问

**位置**：`context.cj:32-58`、`context.cj:140-148`

**描述**：通过 FFI 获取各类目录路径

| 接口 | 返回值 | 风险 |
|-----|-------|-----|
| `FfiContextGetFilesDir()` | `String` | 低 |
| `FfiContextGetCacheDir()` | `String` | 低 |
| `FfiContextGetDatabaseDir()` | `String` | 低 |
| `FfiContextGetDistributedFilesDir()` | `String` | 低 |

**证据来源**：`context.cj:32-48`

### 6. 权限校验

**位置**：`callee.cj:26`、`ui_ability_context.cj:121`

**描述**：权限检查与运行时权限请求

| 接口 | 功能 | 风险 |
|-----|------|-----|
| `FfiOHOSAbilityAccessCtrlCheckAccessTokenSync()` | 访问令牌校验 | 低 |
| `requestPermissionsFromUser()` | 运行时权限请求 | 低 |

**证据来源**：`callee.cj:26` - `FfiOHOSAbilityAccessCtrlCheckAccessTokenSync` 声明

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界图                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────┐         ┌─────────────────────┐                    │
│  │   用户应用代码        │         │   ArkTS/JavaScript  │                    │
│  │   (可信)            │         │   (边界外)           │                    │
│  └─────────────────────┘         └─────────────────────┘                    │
│           │                               │                                  │
│           │ FFI 调用                       │ N-API 调用                      │
│           │ (边界 1)                      │ (边界 2)                        │
│           ▼                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │                    ability_cangjie_wrapper                        │        │
│  │                                                                 │        │
│  │  ┌───────────────────────────────────────────────────────────┐   │        │
│  │  │  仓颉代码层                                               │   │        │
│  │  │  - 参数校验                                               │   │        │
│  │  │  - 类型转换                                               │   │        │
│  │  │  - 错误处理                                               │   │        │
│  │  └───────────────────────────────────────────────────────────┘   │        │
│  │                               │                                   │        │
│  │                    FFI 边界 (最关键)                              │        │
│  │                               │                                   │        │
│  │  ┌───────────────────────────────────────────────────────────┐   │        │
│  │  │  原生 FFI 接口层                                          │   │        │
│  │  │  (ability_runtime FFI targets)                            │   │        │
│  │  └───────────────────────────────────────────────────────────┘   │        │
│  │                               │                                   │        │
│  └───────────────────────────────┼───────────────────────────────────┘        │
│                                  │                                            │
│                                  ▼                                            │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │                    ability_runtime (Native)                       │        │
│  │                                                                 │        │
│  │  - 权限校验 (access_token)                                      │        │
│  │  - IPC 通信 (binder)                                            │        │
│  │  - 文件系统操作                                                 │        │
│  │  - 网络操作 (如需要)                                             │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                                                                              │
│  边界说明：                                                                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  │ 边界 1: FFI 边界 (Cangjie → Native C++)                             │        │
│  │ 风险等级: 高                                                        │        │
│  │ 控制措施: foreign func 类型约束、Option 类型空值检查                  │        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  │ 边界 2: N-API 边界 (Cangjie ↔ ArkTS/JS)                            │        │
│  │ 风险等级: 高                                                        │        │
│  │ 控制措施: context_interop 中的类型转换验证                           │        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  │ 边界 3: IPC/RPC 边界 (跨进程通信)                                   │        │
│  │ 风险等级: 中                                                        │        │
│  │ 控制措施: RemoteObject/IRemoteObject 处理                           │        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 数据流分析

### 输入数据流

| 数据源 | 数据类型 | 处理方式 | 校验位置 |
|-------|---------|---------|---------|
| 用户代码参数 | Want、String、Array | 直接传递给 FFI | `want.cj` |
| JS 值转换 | napi_value | context_interop 转换 | `context_interop.cj` |
| Shell 命令 | String | 直接传递给 FFI | `ability_delegator_registry.cj:443` |
| URI | String | 原生层解析 | `want.cj:54` |
| 配置数据 | CJConfiguration | 直接使用 | `ui_ability.cj:270` |

---

## 可被利用点分析

### 高危风险

#### R1：Shell 命令注入

| 属性 | 值 |
|-----|-----|
| **编号** | R1 |
| **风险等级** | 严重 (High) |
| **位置** | `ability_delegator_registry.cj:441-447` |
| **触发方式** | 调用 `executeShellCommand()` 时传入未校验的用户输入 |
| **影响** | 任意命令执行，可能导致权限提升、敏感数据泄露、系统破坏 |
| **证据** | `ability_delegator_registry.cj:441-447` - `executeShellCommand` 直接传递 `cmd` 参数给 FFI |

**代码证据**：
```cangjie
public func executeShellCommand(cmd: String, timeoutSecs!: Int64 = 0): ShellCmdResult {
    unsafe {
        var cCmd = LibC.mallocCString(cmd)  // 直接转换，无校验
        let shellCmdResultID = FFIAbilityDelegatorExecuteShellCommand(...)
        LibC.free(cCmd)
        return ShellCmdResult(Int64(shellCmdResultID))
    }
}
```

**修复建议**：
1. 添加命令白名单校验
2. 使用参数化 API 替代字符串拼接
3. 限制可执行的命令范围
4. 在 API 文档中明确要求传入预定义命令

---

#### R2：N-API 类型混淆

| 属性 | 值 |
|-----|-----|
| **编号** | R2 |
| **风险等级** | 高 (Medium) |
| **位置** | `context_interop.cj:58-140` |
| **触发方式** | 传入错误类型的 JS 值进行上下文转换 |
| **影响** | 类型混淆可能导致内存损坏或安全检查绕过 |
| **证据** | `context_interop.cj:81-140` - `create*FromJSValue()` 使用模式匹配进行类型识别 |

**代码证据**：
```cangjie
public func createUIAbilityContextFromJSValue(env: napi_env, value: napi_value): UIAbilityContext {
    let contextId = unsafe { FfiCreateUIAbilityContextFromNapi(env, value) }
    return UIAbilityContext(contextId)
}
```

**修复建议**：
1. 在 FFI 层面添加类型标识验证
2. 在转换前后添加一致性校验
3. 使用更严格的类型检查机制

---

### 中等风险

#### R3：URI 处理风险

| 属性 | 值 |
|-----|-----|
| **编号** | R3 |
| **风险等级** | 中 (Medium) |
| **位置** | `want.cj:54,116,247,283` |
| **触发方式** | 传入恶意构造的 URI 字符串 |
| **影响** | 可能导致路径遍历、信息泄露 |
| **证据** | `want.cj:54` - `FFICJWantParseUri(uri: CString)` URI 解析在原生层进行 |

**代码证据**：
```cangjie
public var uri: String

foreign func FFICJWantParseUri(uri: CString): WantHandle

// 使用时
var uri = params.uri.toString()  // 直接使用，无校验
```

**修复建议**：
1. 在仓颉层添加 URI 格式验证
2. 检查 URI 包含的路径遍历模式
3. 记录 URI 相关的安全错误码（`want.cj:339-342` 已定义相关错误码）

---

#### R4：Want 参数解析

| 属性 | 值 |
|-----|-----|
| **编号** | R4 |
| **风险等级** | 中 (Medium) |
| **位置** | `want.cj:324-327` |
| **触发方式** | 传入超长或畸形 JSON 参数 |
| **影响** | 拒绝服务（DoS），潜在内存问题 |
| **证据** | `want.cj:203-204` - 文档说明参数最长 200KB |

**代码证据**：
```cangjie
public var parameters: HashMap<String, WantValueType>
// 注意：最大 200KB 数据，超出可能导致问题
```

**修复建议**：
1. 添加参数大小预检查
2. 在文档中明确限制
3. 对异常大小的参数抛出明确错误

---

### 低风险

#### R5：FFI Handle 空值处理

| 属性 | 值 |
|-----|-----|
| **编号** | R5 |
| **风险等级** | 低 (Low) |
| **位置** | `want.cj:301-302`、`ui_ability.cj:107` |
| **触发方式** | 原生层返回空 Handle |
| **影响** | 拒绝服务（抛异常），非安全漏洞 |
| **证据** | 多处使用 `Option<T>` 类型和 `match` 进行空值检查 |

**代码证据**：
```cangjie
// want.cj:301-302
if (wantHandle == NULL_PTR) {
    throw BusinessException(ERROR_CODE_INNER, "Internal error.")
}

// ui_ability.cj:107
case None => throw BusinessException(ERROR_CODE_INNER, "No such Ability: ...")
```

**评估**：已有良好的空值检查机制，风险可控。

---

#### R6：内存管理

| 属性 | 值 |
|-----|-----|
| **编号** | R6 |
| **风险等级** | 低 (Low) |
| **位置** | `want.cj:342-361`、`ability_delegator_registry.cj:503-505` |
| **触发方式** | CString 分配后未释放 |
| **影响** | 内存泄漏 |
| **证据** | 使用 `try...asResource()` 模式确保释放 |

**代码证据**：
```cangjie
// want.cj:342-345 - 使用 asResource 确保释放
try (unsafeUri = LibC.mallocCString(uri).asResource(), ...) {
    // ...
}  // 自动释放
```

**评估**：代码遵循良好的内存管理实践，风险可控。

---

## 安全检查清单

| 检查项 | 状态 | 证据 |
|-------|------|-----|
| Shell 命令白名单 | 未实现 | `ability_delegator_registry.cj:441-447` |
| N-API 类型校验 | 部分实现 | `context_interop.cj` |
| URI 路径遍历检查 | 未实现 | `want.cj:54` |
| 参数大小限制 | 部分实现 | `want.cj:203-204` |
| Handle 空值检查 | 已实现 | 多处 `Option<T>` + `match` |
| 内存释放检查 | 已实现 | `try...asResource()` 模式 |
| 权限校验 | 已实现 | `callee.cj:26` |
| 错误信息泄露 | 部分实现 | 错误码映射完整 |

---

## 修复建议优先级

| 优先级 | 风险 | 修复建议 |
|-------|-----|---------|
| P0 (立即) | R1 | 实现 `executeShellCommand()` 命令白名单 |
| P1 (短期) | R2 | 增强 N-API 边界类型校验 |
| P2 (中期) | R3 | 添加 URI 格式验证 |
| P3 (长期) | R4 | 完善参数大小检查与文档 |

---

## 依赖子系统安全

以下安全控制依赖于下游子系统：

| 依赖项 | 安全控制 | 风险转移 |
|-------|---------|---------|
| `ability_runtime` | 原生层权限校验、输入验证 | 高 |
| `access_token` | 访问令牌校验 | 高 |
| `cangjie_ark_interop` | FFI 类型安全 | 中 |

**说明**：本仓库作为仓颉封装层，大量安全控制依赖原生实现。建议对依赖子系统进行独立的安全评审。

---

## 安全最佳实践

### 1. API 使用安全

```cangjie
// 不推荐：直接传入用户输入
let cmd = userInput  // 危险！
executeShellCommand(cmd)

// 推荐：使用预定义命令
executeShellCommand("ls -la")
```

### 2. Want 参数安全

```cangjie
// 推荐：限制参数大小
if (params.size > 200 * 1024) {
    throw BusinessException(..., "Parameters too large")
}
```

### 3. URI 安全

```cangjie
// 推荐：验证 URI 格式
if (uri.contains("../")) {
    throw BusinessException(..., "Invalid URI")
}
```

---

## 参考文献

| 资源 | 说明 |
|-----|------|
| [OpenHarmony 安全指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/Readme.md) | 官方安全指导 |
| [仓颉安全编码规范](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) | 仓颉语言安全实践 |
| [Access Token 安全模型](https://gitee.com/openharmony/security_access_token) | 权限校验机制 |

---

## 评审结论

| 类别 | 评估 |
|-----|------|
| **整体风险** | 中等偏高（存在严重风险点 R1） |
| **主要风险** | Shell 命令注入、R2 |
| **已有控制** | Handle 空值检查、内存管理较为完善 |
| **主要漏洞** | Shell 命令执行缺少输入校验 |
| **建议** | 优先修复 R1，添加安全审计机制 |
