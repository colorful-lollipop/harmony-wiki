# 关键宏与 Feature Flags

## 概述

本文档列出 ability_cangjie_wrapper 中使用的关键注解、宏定义和配置标志。

---

## API 级别注解

### @APILevel

用于标注 API 的引入版本和系统能力要求。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Ability.AbilityRuntime.Core"
]
```

| 属性 | 含义 | 常用值 |
|-----|------|-------|
| `since` | 引入的 API 级别 | `"22"` |
| `syscap` | 系统能力要求 | 见下表 |

### 系统能力常量

| 系统能力 | 用途 | 使用的 API |
|---------|------|----------|
| `SystemCapability.Ability.AbilityRuntime.Core` | 核心能力 | Context、ErrorManager、AbilityDelegator |
| `SystemCapability.Ability.AbilityRuntime.AbilityCore` | UIAbility 能力 | UIAbility 生命周期 |
| `SystemCapability.Ability.AbilityBase` | 基础能力 | Want、ElementName |

**证据来源**：各模块源码文件的 `@APILevel` 注解

---

## 隐藏注解

### @Hide

用于标注不应暴露给开发者的内部 API。

```cangjie
@!Hide[isChecked: true]
public open func onWillForeground(): Unit {}
```

| 注解 | 用途 | 示例 |
|-----|------|------|
| `@!Hide[isChecked: true]` | 内部方法 | `onWillForeground()`, `onDidForeground()` |
| `@!Hide[isChecked: true]` | 内部属性 | `callee` 属性 |

**使用位置**：
- `ui_ability.cj:641-666` - 生命周期相关内部方法
- `ui_ability.cj:562-566` - callee 属性

---

## 异常抛出注解

### throwexception

标注 API 是否会抛出异常。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Ability.AbilityRuntime.Core",
    throwexception: true
]
public static func on(eventType: ErrorManagerEvent, observer: ErrorObserver): Int32
```

| 值 | 含义 |
---|------|
| `throwexception: true` | API 可能抛出 BusinessException |
| （默认） | API 不抛出异常 |

**证据来源**：`error_manager.cj:82-85`

---

## 线程约束注解

### workerthread

标注 API 是否可在工作线程调用。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Ability.AbilityRuntime.Core",
    workerthread: true
]
public func executeShellCommand(cmd: String, timeoutSecs!: Int64 = 0): ShellCmdResult
```

| 注解 | 含义 |
|-----|------|
| `workerthread: true` | 允许在工作线程调用 |
| `workerthread: false` 或省略 | 必须在主线程调用 |

**常见线程约束**：

| API | 线程要求 | 证据 |
|-----|---------|------|
| `startAbility()` | 主线程 | `ui_ability_context.cj:72` |
| `terminateSelf()` | 主线程 | `ui_ability_context.cj:108` |
| `executeShellCommand()` | 工作线程 | `ability_delegator_registry.cj:439` |
| `requestPermissionsFromUser()` | 主线程 | `ui_ability_context.cj:173` |
| `addAbilityMonitor()` | 工作线程 | `ability_delegator_registry.cj:499` |

---

## C 调用约定

### @C

用于标注可从 C 代码调用的函数（FFI 回调）。

```cangjie
@C
struct CJAbilityFuncs {
    CJAbilityFuncs(
        let createAbility: CFunc<(CString) -> Int64>,
        // ...
    ) {}
}

@C
func abilityOnStart(id: Int64, wantHandle: WantHandle, launchParam: CJLaunchParam): Unit {
    // ...
}
```

| 用途 | 示例 |
|-----|------|
| FFI 回调函数 | `abilityOnStart`, `abilityOnStop` |
| 回调结构体 | `CJAbilityFuncs`, `CJLaunchParam` |
| 注册函数 | `abilityCjFuncsRegister` |

**证据来源**：`ui_ability.cj:40-77`

---

## 错误码常量

### 模块级错误码

| 模块 | 常量名 | 值 | 含义 |
|-----|-------|---|------|
| 通用 | `ERROR_CODE_INNER` | `16000050` | 内部错误 |
| ErrorManager | `SUCCESS_CODE` | `0` | 成功 |
| ErrorManager | `CALLER_ERROR` | `16200001` | 调用者错误 |
| AbilityDelegator | `COMMON_FAILED` | `16000100` | 通用失败 |
| AbilityDelegator | `INVALID_PARA` | `401` | 参数错误 |
| Want | `ERROR_CODE_INNER` | `16000050` | 内部错误 |

**证据来源**：
- `error_manager.cj:31-34`
- `ability_delegator_registry.cj:30-46`
- `want.cj`（无显式定义，使用默认）

### 权限错误码

| 错误码 | 含义 | 来源 |
|-------|------|------|
| `201` | 权限校验失败 | `ability_errorcode.cj:567` |
| `202` | 非系统应用使用系统 API | `ability_errorcode.cj:568` |

**证据来源**：`ability_errorcode.cj:567-568`

---

## 权限常量

### CALLEE 权限

| 常量 | 值 | 用途 |
|-----|---|------|
| `PERMISSION_ABILITY_BACKGROUND_COMMUNICATION` | `'ohos.permission.ABILITY_BACKGROUND_COMMUNICATION'` | 后台通信权限 |

**证据来源**：`callee.cj:24`

---

## 构建配置标志

### 平台条件编译

```gn
if (is_mingw || is_mac) {
    sources = [ "mock/..." ]
} else {
    sources = [ "real/..." ]
}
```

| 标志 | 含义 |
|-----|------|
| `is_mingw` | MinGW（Windows）环境 |
| `is_mac` | macOS 环境 |

**证据来源**：`BUILD.gn:14` 及各模块 BUILD.gn

---

## FFIData 标志

### FFIData 类型继承

| 类 | 继承类型 | 用途 |
|---|---------|------|
| `UIAbility` | `FFIData` | 可通过 FFI 管理的 UIAbility 实例 |
| `AbilityStage` | `FFIData` | 可通过 FFI 管理的 AbilityStage 实例 |
| `TestRunner` | `FFIData` | 可通过 FFI 管理的 TestRunner 实例 |
| `BaseContext` | `RemoteData` | 远程数据（Context 基类） |
| `ShellCmdResult` | `RemoteDataLite` | 轻量级远程数据 |
| `AbilityDelegator` | `RemoteDataLite` | 测试委托器 |

**证据来源**：
- `ui_ability.cj:497` - UIAbility 继承
- `ability_stage.cj:141` - AbilityStage 继承
- `test_runner.cj:106` - TestRunner 继承

---

## 包级别注解

### Beta 声明

```cangjie
// The Cangjie API is in Beta. For details on its capabilities and limitations, please refer to the README file.
```

每个 `.cj` 文件头部包含此声明，表明仓颉 API 处于 Beta 阶段。

**证据来源**：所有 `.cj` 文件头部（第 16 行）
