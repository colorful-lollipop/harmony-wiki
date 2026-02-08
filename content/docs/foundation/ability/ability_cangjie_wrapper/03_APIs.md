# N-API 完整清单

## 概述

本文档列出 ability_cangjie_wrapper 导出的所有仓颉 API，包括 JS/Cangjie API 名称、参数、返回值、错误码以及对应的原生实现位置。

## API 命名空间索引

| 命名空间 | 说明 | 主要类 |
|---------|------|-------|
| `ohos.app.ability` | 基础能力类型 | `Ability`, `BaseContext` |
| `ohos.app.ability.ui_ability` | UIAbility 核心 | `UIAbility`, `UIAbilityContext`, `ApplicationContext` |
| `ohos.app.ability.ability_stage` | 组件管理器 | `AbilityStage`, `AbilityStageContext` |
| `ohos.app.ability.error_manager` | 错误观测 | `ErrorManager`, `ErrorObserver` |
| `ohos.app.ability.ability_delegator_registry` | 测试框架 | `AbilityDelegatorRegistry`, `AbilityDelegator` |
| `ohos.app.ability.want` | 意图传递 | `Want`, `WantValueType` |
| `ohos.app.ability.app_recovery` | 应用恢复 | `restartApp()` |
| `ohos.app.ability.common` | 公共类型 | `AbilityResult`, `ConnectOptions` |
| `kit.AbilityKit` | Kit 聚合 | 导出所有上述模块 |

---

## 1. UIAbility 模块（ohos.app.ability.ui_ability）

### 1.1 UIAbility 类

**命名空间**：`ohos.app.ability.ui_ability`

**说明**：UIAbility 是包含 UI 界面的应用组件，提供完整的生命周期管理能力。

#### 1.1.1 生命周期回调

| API | 参数 | 返回值 | 同步/异步 | 异常抛出 | C++ 实现位置 |
|-----|------|-------|---------|---------|--------------|
| `onCreate(want: Want, launchParam: LaunchParam)` | `want`: 启动参数；`launchParam`: 启动参数 | `Unit` | 同步 | 无 | `ui_ability.cj:581` |
| `onWindowStageCreate(windowStage: WindowStage)` | `windowStage`: 窗口舞台 | `Unit` | 同步 | 无 | `ui_ability.cj:592` |
| `onWindowStageDestroy()` | 无 | `Unit` | 同步 | 无 | `ui_ability.cj:603` |
| `onForeground()` | 无 | `Unit` | 同步 | 无 | `ui_ability.cj:639` |
| `onBackground()` | 无 | `Unit` | 同步 | 无 | `ui_ability.cj:660` |
| `onDestroy()` | 无 | `Unit` | 同步 | 无 | `ui_ability.cj:624` |
| `onNewWant(want: Want, launchParam: LaunchParam)` | `want`: 新请求意图；`launchParam`: 启动参数 | `Unit` | 同步 | 无 | `ui_ability.cj:686` |

#### 1.1.2 属性访问

| API | 类型 | 说明 | C++ 实现位置 |
|-----|------|-----|--------------|
| `context` | `UIAbilityContext` | UIAbility 的上下文 | `ui_ability.cj:513-523` |
| `launchWant` | `Want` | 启动时的 Want 参数 | `ui_ability.cj:534-541` |
| `lastRequestWant` | `Want` | 最近一次请求的 Want | `ui_ability.cj:553-560` |

#### 1.1.3 生命周期常量

| 枚举 | 说明 | C++ 实现位置 |
|-----|------|--------------|
| `LaunchReason` | 启动原因枚举 | `ability_constant.cj` |
| `LastExitReason` | 上次退出原因枚举 | `ability_constant.cj` |
| `OnContinueResult` | 继续迁移结果 | `ability_constant.cj` |
| `OnSaveResult` | 保存状态结果 | `ability_constant.cj` |
| `StateType` | 状态类型枚举 | `ability_constant.cj` |

---

### 1.2 UIAbilityContext 类

**命名空间**：`ohos.app.ability.ui_ability`

**说明**：UIAbility 的上下文，提供组件级别的操作能力。

#### 1.2.1 组件操作

| API | 参数 | 返回值 | 异常抛出 | C++ 实现位置 |
|-----|------|-------|---------|--------------|
| `startAbility(want: Want)` | `want`: 目标 Ability 参数 | `Unit` | BusinessException | `ui_ability_context.cj:72-81` |
| `startAbilityWithOptions(want: Want, options: StartOptions)` | `want`: 目标 Ability；`options`: 启动选项 | `Unit` | BusinessException | `ui_ability_context.cj:90-99` |
| `terminateSelf()` | 无 | `Unit` | BusinessException | `ui_ability_context.cj:108-117` |
| `startServiceExtensionAbility(want: Want)` | `want`: 目标 ExtensionAbility | `Unit` | BusinessException | `ui_ability_context.cj:127-136` |
| `stopServiceExtensionAbility(want: Want)` | `want`: 目标 ExtensionAbility | `Unit` | BusinessException | `ui_ability_context.cj:145-154` |

#### 1.2.2 权限请求

| API | 参数 | 返回值 | 异常抛出 | C++ 实现位置 |
|-----|------|-------|---------|--------------|
| `requestPermissionsFromUser(permissions: Array<String>)` | `permissions`: 权限名称数组 | `Unit` | BusinessException | `ui_ability_context.cj:173-182` |

#### 1.2.3 资源访问

| API | 参数 | 返回值 | C++ 实现位置 |
|-----|------|-------|--------------|
| `filesDir` | 无 | `String` | `context.cj:140-149` |
| `cacheDir` | 无 | `String` | `context.cj:134` |
| `tempDir` | 无 | `String` | `context.cj:136` |
| `resourceDir` | 无 | `String` | `context.cj:138` |
| `databaseDir` | 无 | `String` | `context.cj:140` |
| `preferencesDir` | 无 | `String` | `context.cj:142` |
| `bundleCodeDir` | 无 | `String` | `context.cj:144` |
| `distributedFilesDir` | 无 | `String` | `context.cj:146` |
| `cloudFileDir` | 无 | `String` | `context.cj:148` |

#### 1.2.4 应用信息

| API | 参数 | 返回值 | C++ 实现位置 |
|-----|------|-------|--------------|
| `resourceManager` | 无 | `ResourceManager` | `context.cj:90-100` |
| `applicationInfo` | 无 | `ApplicationInfo` | `context.cj:111-123` |
| `area` | 无/设置 | `AreaMode` | `context.cj:184-197` |

#### 1.2.5 错误码

| 错误码 | 说明 | 来源 |
|-------|------|------|
| 16000001 | 指定 ability 不存在 | `ability_errorcode.cj:577` |
| 16000002 | ability 类型错误 | `ability_errorcode.cj:581` |
| 16000004 | 无法启动不可见组件 | `ability_errorcode.cj:585` |
| 16000005 | 指定进程无权限 | `ability_errorcode.cj:589` |
| 16000050 | 内部错误 | `ability_errorcode.cj:613` |
| 201 | 权限校验失败 | `ability_errorcode.cj:567` |

---

### 1.3 ApplicationContext 类

**命名空间**：`ohos.app.ability.ui_ability`

**说明**：应用级别上下文，共享于所有 Ability 实例。

#### 1.3.1 应用级操作

| API | 参数 | 返回值 | 异常抛出 | C++ 实现位置 |
|-----|------|-------|---------|--------------|
| `getProcessName()` | 无 | `String` | 无 | `application_context.cj` |
| `getApplicationContext()` | 无 | `Context` | 无 | `application_context.cj` |

---

## 2. AbilityStage 模块（ohos.app.ability.ability_stage）

### 2.1 AbilityStage 类

**命名空间**：`ohos.app.ability.ability_stage`

**说明**：模块级组件管理器，用于协调模块内的资源和生命周期。

#### 2.1.1 生命周期回调

| API | 参数 | 返回值 | C++ 实现位置 |
|-----|------|-------|--------------|
| `onCreate()` | 无 | `Unit` | `ability_stage.cj:228` |
| `context` | 无 | `AbilityStageContext` | `ability_stage.cj:190-200` |

#### 2.1.2 创建者注册

| API | 参数 | 返回值 | 说明 | C++ 实现位置 |
|-----|------|-------|------|--------------|
| `registerCreator(moduleName: String, creator: () -> AbilityStage)` | `moduleName`: 模块名；`creator`: 创建函数 | `Unit` | 注册 AbilityStage 创建者 | `ability_stage.cj:158-167` |

---

## 3. 错误观测模块（ohos.app.ability.error_manager）

### 3.1 ErrorManager 类

**命名空间**：`ohos.app.ability.error_manager`

**说明**：提供错误观测器的注册与注销能力。

| API | 参数 | 返回值 | 异常抛出 | C++ 实现位置 |
|-----|------|-------|---------|--------------|
| `on(eventType: ErrorManagerEvent, observer: ErrorObserver)` | `eventType`: 事件类型；`observer`: 错误观察器 | `Int32`（观察器 ID） | BusinessException | `error_manager.cj:86-104` |
| `off(eventType: ErrorManagerEvent, observerId: Int32)` | `eventType`: 事件类型；`observerId`: 观察器 ID | `Unit` | BusinessException | `error_manager.cj:120-134` |

#### 3.1.1 错误码

| 错误码 | 说明 | 来源 |
|-------|------|------|
| 0 | 成功 | `error_manager.cj:31` |
| 401 | 参数校验失败 | `error_manager.cj:32` |
| 16200001 | 调用者错误（非主线程） | `error_manager.cj:33` |
| 16000050 | 内部错误 | `error_manager.cj:34` |

---

## 4. 自动化测试框架模块（ohos.app.ability.ability_delegator_registry）

### 4.1 AbilityDelegatorRegistry 类

**命名空间**：`ohos.app.ability.ability_delegator_registry`

**说明**：测试框架注册表，用于获取测试工具和参数。

| API | 参数 | 返回值 | C++ 实现位置 |
|-----|------|-------|--------------|
| `getAbilityDelegator()` | 无 | `AbilityDelegator` | `ability_delegator_registry.cj:281-290` |
| `getArguments()` | 无 | `AbilityDelegatorArgs` | `ability_delegator_registry.cj:301-314` |

### 4.2 AbilityDelegator 类

**命名空间**：`ohos.app.ability.ability_delegator_registry`

**说明**：测试工具，提供 Ability 生命周期控制和测试执行能力。

| API | 参数 | 返回值 | 异常抛出 | C++ 实现位置 |
|-----|------|-------|---------|--------------|
| `startAbility(want: Want)` | `want`: 目标 Ability | `Unit` | BusinessException | `ability_delegator_registry.cj:421-427` |
| `executeShellCommand(cmd: String, timeoutSecs: Int64)` | `cmd`: shell 命令；`timeoutSecs`: 超时时间（秒） | `ShellCmdResult` | BusinessException | `ability_delegator_registry.cj:441-449` |
| `getAppContext()` | 无 | `Context` | 无 | `ability_delegator_registry.cj:460-466` |
| `finishTest(msg: String, code: Int64)` | `msg`: 日志信息；`code`: 结果码 | `Unit` | 无 | `ability_delegator_registry.cj:480-487` |
| `addAbilityMonitor(monitor: AbilityMonitor)` | `monitor`: 能力监控器 | `Unit` | BusinessException | `ability_delegator_registry.cj:501-512` |
| `removeAbilityMonitor(monitor: AbilityMonitor)` | `monitor`: 能力监控器 | `Unit` | BusinessException | `ability_delegator_registry.cj:526-537` |
| `waitAbilityMonitor(monitor: AbilityMonitor, timeout: Int64)` | `monitor`: 能力监控器；`timeout`: 超时（毫秒） | `UIAbility` | BusinessException | `ability_delegator_registry.cj:554-570` |
| `addAbilityStageMonitor(monitor: AbilityStageMonitor)` | `monitor`: 组件管理器监控器 | `Unit` | BusinessException | `ability_delegator_registry.cj:585-597` |
| `removeAbilityStageMonitor(monitor: AbilityStageMonitor)` | `monitor`: 组件管理器监控器 | `Unit` | BusinessException | `ability_delegator_registry.cj:611-623` |
| `waitAbilityStageMonitor(monitor: AbilityStageMonitor, timeout: Int64)` | `monitor`: 组件管理器监控器；`timeout`: 超时 | `AbilityStage` | BusinessException | `ability_delegator_registry.cj:640-656` |
| `print(msg: String)` | `msg`: 日志信息 | `Unit` | BusinessException | `ability_delegator_registry.cj:669-677` |
| `getAbilityState(ability: UIAbility)` | `ability`: UIAbility 实例 | `AbilityLifecycleState` | BusinessException | `ability_delegator_registry.cj:689-698` |
| `getCurrentTopAbility()` | 无 | `UIAbility` | BusinessException | `ability_delegator_registry.cj:713-723` |
| `doAbilityForeground(ability: UIAbility)` | `ability`: UIAbility 实例 | `Unit` | BusinessException | `ability_delegator_registry.cj:737-747` |
| `doAbilityBackground(ability: UIAbility)` | `ability`: UIAbility 实例 | `Unit` | BusinessException | `ability_delegator_registry.cj:761-771` |

#### 4.2.1 错误码

| 错误码 | 说明 | 来源 |
|-------|------|------|
| 16000100 | 通用失败 | `ability_delegator_registry.cj:30` |
| 401 | 参数错误 | `ability_delegator_registry.cj:31` |
| 16000001 | 指定 ability 不存在 | `ability_delegator_registry.cj:32` |
| 16000002 | ability 类型错误 | `ability_delegator_registry.cj:33` |
| 16000004 | 无法启动不可见组件 | `ability_delegator_registry.cj:34` |
| 16000005 | 指定进程无权限 | `ability_delegator_registry.cj:35` |
| 16000006 | 跨用户操作不允许 | `ability_delegator_registry.cj:36` |
| 16000050 | 内部错误 | `ability_delegator_registry.cj:43` |
| 16000053 | ability 不在栈顶 | `ability_delegator_registry.cj:44` |
| 16200001 | 调用者已释放 | `ability_delegator_registry.cj:46` |

### 4.3 AbilityMonitor 类

**命名空间**：`ohos.app.ability.ability_delegator_registry`

**说明**：用于监控指定 Ability 生命周期状态变化。

| 属性 | 类型 | 说明 |
|-----|------|------|
| `abilityName` | `String` | Ability 名称 |
| `moduleName` | `String` | 模块名称 |

### 4.4 AbilityLifecycleState 枚举

| 枚举值 | 说明 |
|-------|------|
| `Uninitialized` | 未初始化 |
| `Create` | 已创建 |
| `Foreground` | 前台 |
| `Background` | 后台 |
| `Destroy` | 已销毁 |

---

## 5. Want 模块（ohos.app.ability.want）

### 5.1 Want 类

**命名空间**：`ohos.app.ability.want`

**说明**：组件间信息传递的载体，用于指定启动目标和携带数据。

#### 5.1.1 属性

| 属性 | 类型 | 说明 | 约束 |
|-----|------|-----|------|
| `bundleName` | `String` | 目标包名 | 必填 |
| `abilityName` | `String` | 目标 Ability 名 | 必填 |
| `deviceId` | `String` | 设备 ID | 可选，默认本地设备 |
| `moduleName` | `String` | 模块名 | 可选 |
| `uri` | `String` | 数据 URI | 可选 |
| `action` | `String` | 操作 | 可选 |
| `entities` | `Array<String>` | 实体列表 | 可选 |
| `dataType` | `String` | MIME 类型 | 可选 |
| `flags` | `UInt32` | 标志位 | 可选 |
| `parameters` | `HashMap<String, WantValueType>` | 参数 | 可选，最长 200KB |

#### 5.1.2 构造方法

| API | 参数 | 说明 | C++ 实现位置 |
|-----|------|-----|--------------|
| `init(deviceId, bundleName, abilityName, moduleName, flags, uri, action, entities, dataType, parameters, fds)` | 详见属性 | 构造函数 | `want.cj:264-293` |

---

## 6. 应用恢复模块（ohos.app.ability.app_recovery）

### 6.1 restartApp 函数

**命名空间**：`ohos.app.ability.app_recovery`

**说明**：重启当前进程并启动首个 Ability（通常是入口 Ability）。

| API | 参数 | 返回值 | 异常抛出 | C++ 实现位置 |
|-----|------|-------|---------|--------------|
| `restartApp()` | 无 | `Unit` | 无 | `app_recovery.cj:42-46` |

---

## 7. TestRunner 模块（ohos.application.test_runner）

### 7.1 TestRunner 类

**命名空间**：`ohos.application.test_runner`

**说明**：测试框架基类，用于实现单元测试。

| API | 参数 | 返回值 | C++ 实现位置 |
|-----|------|-------|--------------|
| `registerCreator(name: String, creator: () -> TestRunner)` | `name`: 模块名；`creator`: 创建函数 | `Unit` | `test_runner.cj:120-129` |
| `onPrepare()` | 无 | `Unit` | `test_runner.cj:149` |
| `onRun()` | 无 | `Unit` | `test_runner.cj:158` |

---

## 8. Kit 导出（kit.AbilityKit）

**导出文件**：`kit/AbilityKit/index.cj`

**说明**：Kit 聚合目标，一站式导入所有 Ability 相关 API。

```cangjie
public import ohos.app.ability.ability_constant.*
public import ohos.app.ability.ability_stage.*
public import ohos.app.ability.app_recovery.*
public import ohos.app.ability.common.*
public import ohos.app.ability.completion_handler.*
public import ohos.app.ability.configuration.*
public import ohos.app.ability.context_constant.*
public import ohos.app.ability.dialog_request.*
public import ohos.app.ability.error_manager.*
public import ohos.app.ability.open_link_options.*
public import ohos.app.ability.start_options.*
public import ohos.app.ability.ui_ability.*
public import ohos.app.ability.want.*
public import ohos.app.ability.want_constant.*
public import ohos.ability_access_ctrl.*
public import ohos.bundle.bundle_manager.*
```

---

## 关键 API 调用链示例

### UIAbility 启动流程

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      UIAbility 启动调用链                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 用户调用 startAbility(want)                                          │
│     ↓                                                                   │
│  2. UIAbilityContext.startAbility()                                      │
│     ↓                                                                   │
│  3. FFI 调用: FfiContextStartAbilityWithOptions(id, wantHandle, options)│
│     ↓                                                                   │
│  4. ability_runtime: cj_ability_context_native                            │
│     ↓                                                                   │
│  5. Ability 管理服务调度                                                  │
│     ↓                                                                   │
│  6. 通过 RegisterCJAbilityFuncs 注册的回调通知 UIAbility                  │
│     ↓                                                                   │
│  7. abilityOnStart(id, wantHandle, launchParam)                         │
│     ↓                                                                   │
│  8. FFIDataManager.getData<UIAbility>(id)                              │
│     ↓                                                                   │
│  9. 调用开发者实现的 onCreate(want, launchParam)                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**证据来源**：`ui_ability.cj:117-138` - abilityOnStart 回调实现

### Want 创建与传递

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Want 创建与传递流程                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 开发者创建 Want(bundleName: "com.example.app", abilityName: "Entry")│
│     ↓                                                                   │
│  2. 调用 Want.createWantHandle()                                          │
│     ↓                                                                   │
│  3. FFI 调用: FFICJWantCreateWithWantInfo(params)                       │
│     ↓                                                                   │
│  4. Native Want 对象创建                                                 │
│     ↓                                                                   │
│  5. WantHandle 传递给 startAbility                                       │
│     ↓                                                                   │
│  6. Native 层解析 WantHandle，提取 bundleName, abilityName 等字段        │
│     ↓                                                                   │
│  7. 定位目标 Ability 并启动                                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**证据来源**：`want.cj:339-364` - createWantHandle 实现

---

## 权限与前置条件

### 需声明的权限

| 权限名称 | 用途 | API |
|---------|------|-----|
| `ohos.permission.ABILITY_BACKGROUND_COMMUNICATION` | 后台通信 | `callee.cj:24` |

### 系统能力要求

所有 API 需要声明以下系统能力：

```
SystemCapability.Ability.AbilityRuntime.Core
SystemCapability.Ability.AbilityRuntime.AbilityCore
SystemCapability.Ability.AbilityBase
```

**证据来源**：`ui_ability.cj:493-496` - `@APILevel` 注解中的 syscap 声明
