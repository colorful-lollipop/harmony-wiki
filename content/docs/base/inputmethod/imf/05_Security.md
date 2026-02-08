# 安全风险评审

## 概述

本章节对输入法框架进行安全风险分析，包括攻击面识别、信任边界分析和安全风险点梳理。

## 攻击面清单

### 1. N-API 接口 (高风险)

| 模块 | 文件路径 | 函数数量 | 风险等级 |
|------|----------|----------|----------|
| `inputMethod` | `frameworks/js/napi/inputmethodclient/js_input_method.cpp` | 10+ | **高** |
| `InputMethodController` | `frameworks/js/napi/inputmethodclient/js_get_input_method_controller.cpp` | 20+ | **高** |
| `InputMethodSetting` | `frameworks/js/napi/inputmethodclient/js_get_input_method_setting.cpp` | 10+ | **高** |
| `inputMethodEngine` | `frameworks/js/napi/inputmethodability/` | 15+ | **高** |
| `keyboardPanelManager` | `frameworks/js/napi/keyboardpanelmanager/` | 6+ | 中 |
| `inputMethodPanel` | `frameworks/js/napi/inputmethodpanel/` | 2 | 中 |

**N-API 注册点**:
```cpp
// frameworks/js/napi/inputmethodclient/input_method_module.cpp:47
static napi_module _module = {
    .nm_version = 1,
    .nm_register_func = Init,
    .nm_modname = "inputMethod",
};
napi_module_register(&_module);
```

### 2. IPC 接口 (高风险)

**SystemAbility**: `InputMethodSystemAbility` (SA ID: 3008)

| 接口 | 文件 | 风险等级 |
|------|------|----------|
| `IInputMethodSystemAbility` | `services/src/input_method_system_ability.cpp` | **高** |
| `IInputClient` | `frameworks/native/inputmethod_controller/` | **高** |
| `IInputMethodCore` | `frameworks/native/inputmethod_ability/` | **高** |
| `IInputDataChannel` | 数据通道 | 中 |
| `ISystemCmdChannel` | 系统命令通道 | **高** |

**核心 IPC 方法** (40+ 个):
- `StartInput()` / `StopInputSession()` / `ReleaseInput()`
- `ShowInput()` / `HideInput()` / `ShowCurrentInput()` / `HideCurrentInput()`
- `SwitchInputMethod()` / `ListInputMethod()`
- `SetCoreAndAgent()` - IME 注册
- `PanelStatusChange()` - 面板状态变更
- `SendPrivateData()` - 私有数据传输

### 3. 配置文件 (中风险)

| 文件 | 路径 | 用途 |
|------|------|------|
| 系统配置 | `etc/inputmethod/inputmethod_framework_config.json` | 框架配置 |
| IME 配置 | `/data/service/el1/public/imf/ime_cfg.json` | IME 状态持久化 |
| 服务配置 | `etc/init/inputmethodservice.cfg` | 启动配置 |
| 参数配置 | `etc/para/inputmethod.para` | 系统参数 |

### 4. 文件系统操作 (中风险)

**文件操作类**: `services/file/src/file_operator.cpp`

| 操作 | 函数 | 行号 |
|------|------|------|
| 读文件 | `FileOperator::Read()` | Line 33, 116 |
| 写文件 | `FileOperator::Write()` | Line 77 |

---

## 信任边界

### 信任边界图

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界分析                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐                                              │
│  │  应用进程     │  ←── Untrusted (第三方应用)                    │
│  │  (JS/Native)  │                                              │
│  └──────┬───────┘                                              │
│         │ N-API                                                │
│         ▼                                                      │
│  ┌──────────────┐                                              │
│  │  JS/N-API    │  ←── 参数校验层                               │
│  │  Framework   │                                              │
│  └──────┬───────┘                                              │
│         │ Inner API                                            │
│         ▼                                                      │
│  ┌──────────────┐                                              │
│  │   Native     │  ←── 客户端框架                               │
│  │  Framework   │                                              │
│  └──────┬───────┘                                              │
│         │ IPC                                                  │
│         ▼                                                      │
│  ╔══════════════╗  ←── Trusted (SystemAbility)                 │
│  ║    IMSA      ║     SA 3008                                  │
│  ║  ( services ) ║     权限校验、身份验证                        │
│  ╚═══════╤══════╝                                              │
│          │ IPC                                                  │
│          ▼                                                     │
│  ┌──────────────┐                                              │
│  │   IME 进程    │  ←── Trusted (已签名输入法)                   │
│  │  (Extension)  │                                              │
│  └──────────────┘                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 边界 1: 应用进程 → IMSA

**跨越方式**: N-API → Inner API → IPC

**信任依据**:
- Token 类型检查 (`AccessTokenKit::GetTokenTypeFlag`)
- 权限校验 (`ohos.permission.CONNECT_IME_ABILITY`)
- 焦点检查 (`IsFocused()`)

**校验点**:
```cpp
// services/src/input_method_system_ability.cpp:526
AccessTokenID tokenId = IPCSkeleton::GetCallingTokenID();
// services/identity_checker/src/identity_checker_impl.cpp:163-171
AccessTokenKit::VerifyAccessToken(tokenId, permission);
```

### 边界 2: IMSA → IME 进程

**跨越方式**: IPC → `IInputMethodCore` / `IInputMethodAgent`

**信任依据**:
- Bundle Name 校验
- 签名校验
- IME 白名单

**校验点**:
```cpp
// services/identity_checker/src/identity_checker_impl.cpp:148-161
IsBundleNameValid(bundleName);
// services/identity_checker/src/identity_checker_impl.cpp:143-146
IsSystemApp(tokenId);
```

### 边界 3: 配置文件读取

**跨越方式**: 文件系统操作

**信任依据**:
- 路径白名单
- 权限校验 (DAC/MAC)

**校验点**:
```cpp
// services/file/src/file_operator.cpp:92
// Path validation before file operations
```

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                 │
│                                                                 │
│  ┌─────────────┐   IPC    ┌─────────────┐   IPC    ┌─────────┐ │
│  │   应用进程   │ ───────▶ │   IMSA      │ ───────▶ │ IME进程 │ │
│  │  (Untrusted)│          │ (Trusted)   │          │(Trusted)│ │
│  └─────────────┘          └─────────────┘          └─────────┘ │
│         │                      │                      │        │
│         ▼                      ▼                      ▼        │
│    N-API 调用            IPC 调用               IPC 调用       │
│    参数校验              权限校验               身份校验       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 边界 1: 应用 → IMSA

- **调用方式**: N-API → Inner API → IPC
- **信任依据**: 需持有 `ohos.permission.CONNECT_IME_ABILITY` 权限
- **校验点**: `AccessTokenKit::GetCallingTokenID()`

### 边界 2: IMSA → IME

- **调用方式**: IPC → InputMethodAgent
- **信任依据**: Bundle Name 校验 + 签名校验
- **校验点**: `IdentityChecker`

### 边界 3: 文件读取

- **调用方式**: 配置解析
- **信任依据**: 路径白名单 + 权限校验
- **校验点**: `SysCfgParser`

## 安全风险评估

### R1: N-API 参数校验不完整 (中危)

**位置**: `frameworks/js/napi/inputmethodclient/js_input_method.cpp:255-258`

**证据**:
```cpp
// 类型检查但部分参数未严格校验范围
napi_valuetype valueType = napi_undefined;
napi_typeof(env, argv, &valueType);
if (valueType != napi_object) {
    IMSA_HILOGE("type is not object!");
    return status;  // 仅日志，未抛出异常
}
```

**触发路径**:
```
JS 调用 → N-API → 类型检查不完整 → 非法参数进入 Native 层
```

**影响**: 可能导致 Native 层崩溃或未定义行为

**修复建议**:
1. 所有 N-API 参数使用 `PARAM_CHECK_RETURN` 宏进行严格校验
2. 字符串参数检查长度和内容
3. 数值参数检查范围
4. 对象参数检查必需字段

---

### R2: IPC 权限校验绕过风险 (高危)

**位置**: `services/src/input_method_system_ability.cpp:875`

**证据**:
```cpp
// SetCoreAndAgent 中的权限检查
if (!HasPermission(tokenId, PERMISSION_CONNECT_IME_ABILITY)) {
    IMSA_HILOGE("%{public}s: failed", __func__);
    return ErrorCode::ERROR_STATUS_PERMISSION_DENIED;
}
```

**问题**: 部分接口权限检查不完整

**证据** (Line 2475-2494):
```cpp
// CheckEnableAndSwitchPermission 实现
bool InputMethodSystemAbility::CheckEnableAndSwitchPermission() {
    // 仅检查是否系统应用，未检查具体权限
    return IsSystemApp();
}
```

**触发路径**:
```
恶意应用 → IPC 调用 → 权限检查不完整 → 非法操作
```

**影响**: 权限绕过，非法切换输入法或获取敏感信息

**修复建议**:
1. 所有 IPC 接口统一使用 `HasPermission()` 进行权限校验
2. 敏感操作（如 `SwitchInputMethod`, `EnableIme`）增加额外权限校验
3. 添加审计日志记录所有权限检查失败

---

### R3: 路径遍历风险 (中危)

**位置**: `services/file/src/file_operator.cpp:77-116`

**证据**:
```cpp
// Write 操作，直接使用传入路径
int32_t FileOperator::Write(const std::string &path, const std::string &content) {
    int fd = open(path.c_str(), O_CREAT | O_WRONLY | O_SYNC | O_TRUNC, 0660);
    // ...
}
```

**触发路径**:
```
恶意配置 → 路径包含 '../' → 写入敏感文件 → 系统受损
```

**影响**: 可能导致配置文件被写入到非预期位置

**修复建议**:
1. 使用 `realpath()` 规范化路径
2. 路径白名单机制
3. 禁止路径中包含 `..` 或绝对路径

---

### R4: 竞态条件风险 (中危)

**位置**: `frameworks/native/inputmethod_controller/src/input_method_controller.cpp`

**证据**:
```cpp
// abilityLock_ 保护 abilityManager_
std::mutex abilityLock_;
sptr<IInputMethodSystemAbility> abilityManager_ = nullptr;
```

**问题**: 多线程环境下可能存在 Use-after-free

**触发路径**:
```
线程 A: 检查 abilityManager_ != nullptr
线程 B: 销毁 abilityManager_
线程 A: 使用 abilityManager_ → Use-after-free
```

**影响**: 内存安全问题，可能导致崩溃

**修复建议**:
1. 使用 `std::shared_ptr` 替代原始指针
2. 使用原子操作或更细粒度锁
3. 添加对象生命周期管理

---

### R5: 敏感信息泄露 (低危)

**位置**: `common/imf_hisysevent/src/imc_hisysevent_reporter.cpp:116`

**证据**:
```cpp
auto tokenType = AccessTokenKit::GetTokenTypeFlag(tokenId);
if (AccessTokenKit::GetHapTokenInfo(tokenId, hapInfo) == 0) {
    // 记录用户信息到日志
    IMSA_HILOGI("BundleName: %{public}s, Uid: %{public}d", 
                hapInfo.bundleName.c_str(), hapInfo.uid);
}
```

**触发路径**:
```
正常操作 → 日志记录 → 日志文件被读取 → 信息泄露
```

**影响**: 用户隐私信息可能被泄露

**修复建议**:
1. 日志中敏感信息脱敏 (如只显示 bundleName 前 3 字符)
2. 区分日志级别，敏感信息仅在 debug 级别输出
3. 日志文件访问权限控制

---

### R6: 私有数据传输风险 (中危)

**位置**: `services/src/input_method_system_ability.cpp:68`

**证据**:
```cpp
// SendPrivateData 接口
ErrCode SendPrivateData(const Value &value);
```

**问题**: 私有数据传输可能缺乏来源校验

**触发路径**:
```
应用 A → SendPrivateData → IMSA → 转发给 IME
应用 B (恶意) → 拦截或篡改数据
```

**影响**: 应用间数据可能被窃取或篡改

**修复建议**:
1. 私有数据添加来源校验
2. 数据加密传输
3. 接收方确认机制

---

## 权限模型

### 核心权限清单

| 权限 | 描述 | 保护的操作 | 位置 |
|------|------|------------|------|
| `ohos.permission.CONNECT_IME_ABILITY` | 连接输入法能力 | IPC 调用、输入法切换 | `input_method_system_ability.cpp:74` |
| `ohos.permission.INJECT_INPUT_EVENT` | 注入输入事件 | 键盘事件注入 | MMI 调用 |
| `ohos.permission.PRIVACY_WINDOW` | 隐私窗口 | 隐私键盘创建 | 面板创建 |
| `ohos.permission.REPORT_RESOURCE_SCHEDULE_EVENT` | 上报资源调度 | 遥测上报 | 事件上报 |

### 身份校验机制

**IdentityChecker**: `services/identity_checker/src/identity_checker_impl.cpp`

| 方法 | 用途 | 行号 |
|------|------|------|
| `HasPermission()` | 权限校验 | Line 163-171 |
| `IsFocused()` | 焦点检查 | Line 34-59 |
| `IsSystemApp()` | 系统应用检查 | Line 143-146 |
| `IsBundleNameValid()` | Bundle 名校验 | Line 148-161 |
| `GetBundleNameByToken()` | Token 解析 | Line 208-224 |

---

## 安全机制

### Seccomp 策略

**文件**: `seccomp_policy/imf_ext_secure_filter`

限制输入法扩展的系统调用，防止 exploit。

### 代码签名

所有 IPC 接口调用方需通过 Bundle 签名校验。

### 内存安全

- AddressSanitizer 支持
- CFI (Control Flow Integrity)
- UBSan (Undefined Behavior Sanitizer)

## 安全最佳实践

### 对开发者

1. **最小权限原则**: 仅申请必要的输入法权限
2. **输入验证**: 所有 N-API 调用参数需本地校验
3. **错误处理**: 正确处理权限拒绝错误
4. **日志脱敏**: 不在日志中记录敏感输入内容

### 对安全研究员

1. **重点审计区域**:
   - `services/src/input_method_system_ability.cpp` - IPC 入口
   - `frameworks/js/napi/*` - N-API 参数校验
   - `services/identity_checker/` - 权限校验逻辑
   - `services/file/` - 文件操作

2. **测试建议**:
   - 模糊测试 N-API 参数边界
   - 测试 IPC 并发场景
   - 验证权限校验完备性

---

*本安全评审基于代码版本 2026-02-07，仅供参考。实际安全测试需结合具体场景进行。*

## 相关文档

- [架构说明](./01_Architecture.md)
- [N-API 接口](./02_N-API.md)
- [Inner API 接口](./03_Inner_API.md)
