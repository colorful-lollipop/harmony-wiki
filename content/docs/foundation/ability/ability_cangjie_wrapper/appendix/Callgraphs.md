# 关键调用链

## 概述

本文档展示 ability_cangjie_wrapper 中关键操作的调用链图示。

---

## UIAbility 启动流程

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant Context as UIAbilityContext
    participant FFI as FFI 层
    participant Native as ability_runtime
    participant UIAbility as UIAbility 实例

    User->>Context: startAbility(want)
    Context->>FFI: FfiContextStartAbilityWithOptions(id, wantHandle, options)
    FFI->>Native: 启动 Ability 请求
    Native->>Native: 调度生命周期
    Native->>FFI: 触发 onStart 回调
    FFI->>UIAbility: abilityOnStart(id, wantHandle, launchParam)
    Note over UIAbility: FFIDataManager.getData(id)
    UIAbility->>User: onCreate(want, launchParam)
```

**关键代码路径**：
1. `ui_ability_context.cj:72-81` - startAbility 实现
2. `ui_ability.cj:117-138` - abilityOnStart 回调
3. `ui_ability.cj:581` - onCreate 虚方法

---

## UIAbility 上下文获取

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant Context as Context 类
    participant FFI as FFI 层
    participant Native as ability_runtime

    User->>Context: resourceManager
    Context->>FFI: FfiContextGetContext(id, contextType)
    FFI->>Native: 获取 Context native handle
    Native-->>FFI: 返回 native handle
    FFI-->>Context: ResourceManager 实例
```

**关键代码路径**：
1. `context.cj:90-100` - resourceManager 属性实现
2. `ui_ability.cj:725` - abilityInit 调用

---

## Want 对象创建

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant Want as Want 类
    participant FFI as FFI 层
    participant Native as ability_runtime

    User->>Want: Want(bundleName, abilityName, ...)
    Note over Want: 初始化 Cangjie 端字段
    User->>Want: createWantHandle()
    Want->>FFI: FFICJWantCreateWithWantInfo(params)
    FFI->>Native: 创建 native Want
    Native-->>FFI: 返回 WantHandle
    Want-->>User: WantHandle
```

**关键代码路径**：
1. `want.cj:264-293` - 构造函数
2. `want.cj:339-364` - createWantHandle 实现

---

## AbilityStage 生命周期

```mermaid
sequenceDiagram
    participant Native as ability_runtime
    participant FFI as FFI 层
    participant Stage as AbilityStage

    Native->>FFI: loadAbilityStage(moduleName)
    FFI->>Stage: loadAbilityStage(name)
    Note over Stage: AbilityStage.create(name)
    Stage-->>Native: 返回 AbilityStageHandle
    Native->>FFI: abilityStageOnCreate(id)
    FFI->>Stage: onCreate()
    Stage-->>Native: 完成
```

**关键代码路径**：
1. `ability_stage.cj:63-73` - loadAbilityStage 实现
2. `ability_stage.cj:81-86` - onCreate 回调

---

## 错误观测注册

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant Manager as ErrorManager
    participant FFI as FFI 层
    participant Native as ability_runtime

    User->>Manager: on(eventType, observer)
    Manager->>FFI: FfiOHOSErrorManagerOn(eventType, observer)
    FFI->>Native: 注册错误观察者
    Native-->>FFI: 返回观察者 ID
    FFI-->>Manager: observerId
```

**关键代码路径**：
1. `error_manager.cj:86-104` - on 方法实现

---

## 测试框架操作

```mermaid
sequenceDiagram
    participant User as 测试代码
    participant Delegator as AbilityDelegator
    participant FFI as FFI 层
    participant Native as ability_runtime

    User->>Delegator: startAbility(want)
    Delegator->>FFI: FFIAbilityDelegatorStartAbility(id, wantHandle)
    FFI->>Native: 启动 Ability
    Native-->>FFI: 返回结果
    Delegator-->>User: 完成

    User->>Delegator: executeShellCommand(cmd)
    Note over Delegator: 高风险：直接执行命令
    Delegator->>FFI: FFIAbilityDelegatorExecuteShellCommand(id, cmd)
    FFI->>Native: 执行 shell 命令
    Native-->>FFI: 返回结果
```

**关键代码路径**：
1. `ability_delegator_registry.cj:421-427` - startAbility
2. `ability_delegator_registry.cj:441-449` - executeShellCommand（高风险）

---

## Context 类型转换（N-API 互操作）

```mermaid
sequenceDiagram
    participant ArkTS as ArkTS/JS
    participant Interop as context_interop
    participant Cangjie as Cangjie Context
    participant Native as ability_runtime

    ArkTS->>Interop: toJSValue(env)
    Interop->>Native: FfiConvert*2Napi(env, id)
    Native-->>Interop: napi_value
    Interop-->>ArkTS: JS 对象

    ArkTS->>Interop: create*FromJSValue(env, value)
    Interop->>Native: FfiCreate*FromNapi(env, value)
    Native-->>Interop: id
    Interop-->>Cangjie: Cangjie 实例
```

**关键代码路径**：
1. `context_interop.cj:58-66` - toJSValue 实现
2. `context_interop.cj:81-140` - create*FromJSValue 实现

---

## 模块依赖调用链

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         模块依赖调用链                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  kit.AbilityKit                                                         │
│       │                                                                 │
│       ├──▶ ohos.app.ability.ui_ability                                  │
│       │       │                                                         │
│       │       ├──▶ ohos.app.ability (BaseContext)                       │
│       │       │         │                                               │
│       │       │         └──▶ ohos.ffi (cangjie_ark_interop)             │
│       │       │                                                           │
│       │       ├──▶ ohos.app.ability.want                                │
│       │       │         │                                                 │
│       │       │         ├──▶ ohos.bundle.bundle_manager                  │
│       │       │         └──▶ ohos.encoding.json                          │
│       │       │                                                           │
│       │       ├──▶ ohos.app.ability.ability_stage                       │
│       │       │         │                                                 │
│       │       │         └──▶ ability_runtime:appkit_native               │
│       │       │                                                           │
│       │       └──▶ ability_runtime:cj_*_ffi (外部 FFI)                  │
│       │                                                                 │
│       └──▶ ohos.app.ability.error_manager                               │
│                 │                                                       │
│                 └──▶ ability_runtime:cj_errormanager_ffi                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```
