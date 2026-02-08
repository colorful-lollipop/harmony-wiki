# 架构说明

## 整体架构

本项目采用分层架构设计，通过 FFI（Foreign Function Interface）桥接仓颉代码与原生 Ability Runtime。整体架构可分为四个层次：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         用户应用层（User Application）                     │
│                     开发者编写的仓颉 UIAbility 代码                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     仓颉封装层（Cangjie Wrapper）                          │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      能力 API 导出                                    ││
│  │   UIAbility、Context、AbilityStage、ErrorManager、Want 等类           ││
│  └─────────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      FFI 回调注册                                     ││
│  │   CJAbilityFuncs、CJAbilityStageFuncs 等回调函数表                    ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     运行时桥接层（Runtime Bridge）                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                     Cangjie-ArkTS 互操作                              ││
│  │   context_interop.cj：Context 对象与 N-API 值的双向转换                ││
│  └─────────────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                     FFIData 数据管理                                   ││
│  │   FFIDataManager：跨运行时对象生命周期管理                               ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     原生实现层（Native Runtime）                          │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────────┐│
│  │  ability_runtime  │  │   access_token   │  │   cangjie_ark_interop  ││
│  │                   │  │                  │  │                        ││
│  │  cj_ability_ffi   │  │  cj_ability_     │  │    ohos.ffi           ││
│  │  cj_context_ffi    │  │  access_ctrl_ffi │  │    ohos.labels        ││
│  │  appkit_native     │  │                  │  │    ohos.business_     ││
│  │                   │  │                  │  │    exception          ││
│  └──────────────────┘  └──────────────────┘  └────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

**证据来源**：`README_zh.md:13-44` - 架构图与层次说明

## 核心组件

### UIAbility 组件

UIAbility 是包含 UI 界面的应用组件，提供完整的生命周期管理能力。

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           UIAbility                                       │
├─────────────────────────────────────────────────────────────────────────┤
│  属性：                                                                   │
│  ├─ context: UIAbilityContext        （组件上下文）                       │
│  ├─ launchWant: Want                 （启动参数）                         │
│  └─ lastRequestWant: Want            （最近请求意图）                      │
│                                                                          │
│  生命周期回调：                                                            │
│  ├─ onCreate(want, launchParam)      （创建时调用）                       │
│  ├─ onWindowStageCreate(windowStage) （窗口创建时调用）                   │
│  ├─ onWindowStageDestroy()           （窗口销毁时调用）                   │
│  ├─ onForeground()                   （进入前台）                         │
│  ├─ onBackground()                   （进入后台）                         │
│  ├─ onDestroy()                      （销毁时调用）                        │
│  └─ ... 其他回调方法                                           │
└─────────────────────────────────────────────────────────────────────────┘
         │
         │ 继承
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           Ability                                         │
├─────────────────────────────────────────────────────────────────────────┤
│  基础能力抽象类                                                           │
└─────────────────────────────────────────────────────────────────────────┘
```

**证据来源**：`ui_ability.cj:497-783` - UIAbility 类定义

### 上下文层级关系

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      ApplicationContext                                   │
│              （应用级别上下文，共享于所有 Ability 实例）                     │
├─────────────────────────────────────────────────────────────────────────┤
│  功能：                                                                   │
│  ├─ getApplicationContext()          （获取应用上下文）                    │
│  ├─ getProcessName()                 （获取进程名）                        │
│  └─ getHapModuleInfo()              （获取模块信息）                      │
└─────────────────────────────────────────────────────────────────────────┘
         │
         │ 包含多个
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      AbilityStageContext                                   │
│              （模块级别上下文，关联 AbilityStage）                          │
├─────────────────────────────────────────────────────────────────────────┤
│  功能：                                                                   │
│  ├─ getModuleName()                  （获取模块名）                       │
│  └─ getConfiguration()               （获取配置）                         │
└─────────────────────────────────────────────────────────────────────────┘
         │
         │ 包含多个
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      UIAbilityContext                                     │
│              （组件级别上下文，关联单个 UIAbility 实例）                    │
├─────────────────────────────────────────────────────────────────────────┤
│  功能：                                                                   │
│  ├─ startAbility(want)                （启动 Ability）                    │
│  ├─ startAbilityWithOptions(want, options)                              │
│  ├─ terminateSelf()                   （终止自身）                        │
│  ├─ getBundleName()                   （获取包名）                       │
│  └─ getResourceManager()              （获取资源管理器）                  │
└─────────────────────────────────────────────────────────────────────────┘
```

**证据来源**：`context.cj:71-201` - Context 类定义

## 数据流分析

### 生命周期回调数据流

```
┌──────────────┐     FFI 调用      ┌──────────────────┐     回调分发      ┌──────────────────┐
│   原生 Runtime  │ ───────────────▶ │  FFI 回调函数    │ ───────────────▶ │   UIAbility     │
│  (ability_rt)   │                  │  (ability_*.cj)  │                  │   实例方法        │
└──────────────┘                  └──────────────────┘                  └──────────────────┘
                                              │
                                              │ 通过 FFIDataManager
                                              │ 获取对象实例
                                              ▼
                                      ┌──────────────────┐
                                      │   FFIDataManager  │
                                      │   (对象注册表)     │
                                      └──────────────────┘
```

**关键数据流路径**：

| 阶段 | 数据 | 来源 | 处理方式 |
|-----|------|-----|---------|
| **输入** | native handle | ability_runtime | 通过 FFIDataManager 转换为 Cangjie 对象 |
| **输入** | Want 对象 | 启动参数 | 解析为 WantHandle 传递给 native |
| **配置** | WindowStage | onWindowStageCreate | 通过 WindowStageHandle 桥接 |
| **事件** | 配置变更 | onConfigurationUpdated | 通过 CJConfiguration 传递 |

**证据来源**：`ui_ability.cj:38-773` - FFI 回调函数实现

### Want 数据传递

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Want 数据流                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   调用者                    Wrapper                    Native Runtime     │
│     │                         │                            │               │
│     │ create Want            │                            │               │
│     │ (bundleName,           │                            │               │
│     │  abilityName,          │                            │               │
│     │  parameters...)        │                            │               │
│     ├──────────────────────▶│                            │               │
│     │                       │ createWantHandle()         │               │
│     │                       ├────────────────────────────▶│               │
│     │                       │                            │ 创建 native     │
│     │                       │                            │ Want 对象       │
│     │                       │ ◀────────────────────────────┤               │
│     │                       │ 返回 WantHandle             │               │
│     │                       │                            │               │
│     │                       │ 操作完成后                  │               │
│     │                       │ releaseWantHandle()        │               │
│     │                       ├────────────────────────────▶│               │
│     │                       │                            │ 释放 native    │
│     │                       │                            │ Want 对象       │
│     ▼                       ▼                            ▼               │
```

**证据来源**：`want.cj:300-365` - Want 句柄创建与释放

## 线程模型

### 线程约束

本模块遵循 OpenHarmony 的线程安全模型，关键约束如下：

| API | 线程要求 | 原因 |
|-----|---------|-----|
| `startAbility()` | 主线程调用 | 确保生命周期回调在正确线程执行 |
| `terminateSelf()` | 主线程调用 | 防止并发终止操作 |
| `requestPermissionsFromUser()` | 主线程调用 | UI 交互必须在主线程 |
| `executeShellCommand()` | 工作线程（`workerthread: true`）| 耗时操作 |
| `ErrorManager.on()` | 未指定 | 需要确认 |
| `ErrorManager.off()` | 工作线程 | 异步操作 |

**证据来源**：`ability_delegator_registry.cj:415-420` - `@APILevel` 注解中的 `workerthread: true` 标记

### 回调执行模型

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         回调执行模型                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Native Runtime                    Cangjie Wrapper                      │
│        │                                 │                               │
│        │ 1. 触发生命周期事件              │                               │
│        ├────────────────────────────────▶│                               │
│        │                                 │ 2. 从 FFIDataManager          │
│        │                                 │    获取 UIAbility 实例        │
│        │                                 │                               │
│        │                                 │ 3. 调用回调方法               │
│        │                                 │    onCreate()                │
│        │                                 │    onForeground()            │
│        │                                 │    ...                       │
│        │                                 │                               │
│        │                                 │ 4. 返回结果                  │
│        │◀────────────────────────────────┤                               │
│        │                                 │                               │
│        ▼                                 ▼                               │
│                                                                          │
│   关键点：                                                                │
│   ├─ 回调在 Native Runtime 管理的线程池中执行                              │
│   ├─ UIAbility 实例通过 FFIDataManager 跨线程访问                          │
│   └─ 状态变更通过 EventHub 通知监听者                                     │
└─────────────────────────────────────────────────────────────────────────┘
```

## 模块依赖关系

### 核心模块依赖树

```
kit.AbilityKit
│
├── ohos.app.ability.ability_constant
│   └── 配置常量、启动原因枚举
│
├── ohos.app.ability.ability_stage
│   ├── ohos.application.event_hub
│   ├── ohos.app.ability.configuration
│   ├── ohos.app.ability.context_constant
│   ├── ohos.app.ability.ui_ability
│   │   ├── ohos.app.ability (基础)
│   │   ├── ohos.app.ability.want (意图传递)
│   │   ├── ohos.app.ability.ability_constant
│   │   ├── ohos.app.ability.completion_handler
│   │   ├── ohos.app.ability.context_constant
│   │   ├── ohos.app.ability.dialog_request
│   │   ├── ohos.app.ability.open_link_options
│   │   └── ohos.app.ability.start_options
│   └── ohos.app.ability.want
│
├── ohos.app.ability.error_manager
│   └── ohos.application.error_observer
│
├── ohos.app.ability.common
│   ├── ohos.ability.ability_result
│   ├── ohos.ability.connect_options
│   ├── ohos.application.error_observer
│   └── ohos.app.ability
│
└── ohos.app.ability.ui_ability (最复杂模块)
    ├── ohos.app.ability
    ├── ohos.ability.ability_result
    ├── ohos.ability.connect_options
    ├── ohos.application.event_hub
    ├── ohos.app.ability.ability_constant
    ├── ohos.app.ability.completion_handler
    ├── ohos.app.ability.configuration
    ├── ohos.app.ability.context_constant
    ├── ohos.app.ability.dialog_request
    ├── ohos.app.ability.open_link_options
    ├── ohos.app.ability.start_options
    └── ohos.app.ability.want
```

**证据来源**：`kit/AbilityKit/BUILD.gn:22-37` - kit.AbilityKit 的 cj_deps 依赖列表

### 外部子系统依赖

| 子系统 | 依赖模块 | 用途 |
|-------|---------|------|
| `ability_runtime` | cj_ability_ffi, cj_context_ffi, appkit_native 等 | 原生能力 FFI 接口 |
| `access_token` | cj_ability_access_ctrl_ffi | 权限校验 |
| `cangjie_ark_interop` | ohos.ffi, ohos.labels, ohos.business_exception | 互操作基础 |
| `arkui_cangjie_wrapper` | ohos.base | 基础类型 |
| `bundlemanager_cangjie_wrapper` | ohos.bundle.bundle_manager | 包信息 |
| `communication_cangjie_wrapper` | ohos.rpc | IPC 通信 |
| `global_cangjie_wrapper` | ohos.resource_manager | 资源管理 |
| `hiviewdfx_cangjie_wrapper` | ohos.hilog | 日志 |
| `window_cangjie_wrapper` | ohos.window | 窗口管理 |

**证据来源**：`ohos/app/ability/ui_ability/BUILD.gn:84-99` - ui_ability 模块的 external_deps

## 关键设计模式

### 1. FFI 回调注册模式

```
@C                                    // C 调用约定注解
struct CJAbilityFuncs {
    CJAbilityFuncs(
        let createAbility: CFunc<(CString) -> Int64>,
        let abilityOnStart: CFunc<(Int64, WantHandle, CJLaunchParam) -> Unit>,
        // ... 更多回调
    ) {}
}

@C                                    // 实现回调函数
func abilityOnStart(id: Int64, wantHandle: WantHandle, launchParam: CJLaunchParam): Unit {
    let optAbility = FFIDataManager.getInstance().getData<UIAbility>(id)
    match (optAbility) {
        case Some(ability) => ability.onCreate(want, param)
        case None => throw BusinessException(...)
    }
}

@C
func abilityCjFuncsRegister(result: CPointer<CJAbilityFuncs>): Unit {
    let funcs = CJAbilityFuncs(createAbility, abilityOnStart, ...)
    unsafe { result.write(funcs) }
}

foreign func RegisterCJAbilityFuncs(funcs: CFunc<(CPointer<CJAbilityFuncs>) -> Unit>): Unit

let REGISTER_ABILITY = unsafe { RegisterCJAbilityFuncs(abilityCjFuncsRegister) }
```

**证据来源**：`ui_ability.cj:41-773` - 完整的 FFI 回调注册实现

### 2. FFIData 数据管理模式

```
public open class UIAbility <: FFIData {
    // 在创建时注册到 FFI 数据管理器
    init(...) {
        FFIDataManager.getInstance().register(this)
    }
    
    // 在 FFI 回调中通过 ID 检索
    static func create(name: String): Option<UIAbility> {
        let ability = UIAbility(...)
        FFIDataManager.getInstance().register(ability)
        return Some(ability)
    }
    
    // 在释放时注销
    func release() {
        FFIDataManager.getInstance().releaseData(getID())
    }
}
```

**证据来源**：`ui_ability.cj:88-97` - UIAbility 创建与注册

### 3. Option 类型安全模式

```
// 使用 Option 类型处理可能的空值
let optAbility = FFIDataManager.getInstance().getData<UIAbility>(id)
match (optAbility) {
    case Some(ability) =>
        // 安全使用 ability
        ability.onCreate(want, param)
    case None =>
        // 处理空值情况
        throw BusinessException(ERROR_CODE_INNER, "Internal error.")
}
```

**证据来源**：`ui_ability.cj:118-137` - Option 类型模式匹配

## 稳定性标注

### API 稳定性分级

| 标注 | 含义 | 使用场景 |
|-----|------|---------|
| `@!APILevel[since: "22"]` | 正式 API | 默认使用 |
| `@!Hide[isChecked: true]` | 内部 API | 系统使用，不对外部暴露 |
| Beta 特性 | 标注在包级别 | README 声明 |

### 内部 API 识别

以下标记的 API 为内部使用，不应在应用代码中直接调用：

| 位置 | 符号 | 说明 |
|-----|------|-----|
| `callee.cj` | `Callee` | 内部 IPC 调用端 |
| `hilog.cj` | `ABILITY_LOG` | 内部日志 |
| `context_interop.cj` | `FfiConvert*` | N-API 互操作函数 |

**证据来源**：`ui_ability.cj:641-666` - `@!Hide` 注解的内部方法
