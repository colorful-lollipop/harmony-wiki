# 关键调用链图

> Settings 应用的关键调用链（入口 → 核心逻辑）

---

## 目的

本文档提供 Settings 应用的关键调用链图，帮助理解从入口到核心逻辑的执行路径。

## 适用范围

- 目标读者：系统开发者
- 项目：@ohos/settings (Settings 3.1)

---

## getValue 调用链

### N-API 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant NAPI as N-API 层
    participant Native as Native C++
    participant Helper as DataShareHelper
    participant Ability as DataAbility
    participant Storage as 数据库（三张表）

    App->>NAPI: settings.getValue("brightness", "system")
    Note over NAPI: unwrap 参数
    NAPI->>Native: 调用 getValue 实现
    Native->>Helper: 创建 DataShareHelper
    Note over Helper: 全局缓存
    Helper->>Ability: Query(Select KEYWORD, VALUE<br/>WHERE KEYWORD = "brightness")
    Ability->>Storage: 查询 system 表
    Storage-->>Ability: 返回结果集
    Ability-->>Helper: 返回值
    Helper-->>Native: 返回值
    Native-->>NAPI: 返回值（string）
    NAPI-->>App: 返回 Promise
```

### ANI 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant ANI as ANI 层
    participant Native as Native C++
    participant Helper as DataShareHelper
    participant Ability as DataAbility

    App->>ANI: settings.getValue("brightness", "system")
    Note over ANI: unwrap 参数
    ANI->>Native: 调用 getValue 实现
    Native->>Helper: 创建 DataShareHelper
    Note over Helper: 全局缓存
    Helper->>Ability: Query(Select KEYWORD, VALUE<br/>WHERE KEYWORD = "brightness")
    Ability->>Storage: 查询 system 表
    Storage-->>Ability: 返回结果集
    Ability-->>Helper: 返回值
    Helper-->>Native: 返回值
    Native-->>ANI: 返回值（ani_string）
    ANI-->>App: 返回值
```

---

## setValue 调用链

### N-API 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant NAPI as N-API 层
    participant Native as Native C++
    participant Helper as DataShareHelper
    participant Ability as DataAbility
    participant Storage as 数据库
    participant Observers as 所有观察者

    App->>NAPI: settings.setValue("brightness", "100", "system")
    Note over NAPI: unwrap 参数
    NAPI->>Native: 调用 setValue 实现
    Native->>Helper: 创建 DataShareHelper
    Note over Helper: 全局缓存
    Helper->>Ability: Update(SET VALUE = "100"<br/>WHERE KEYWORD = "brightness")
    Ability->>Storage: 更新 system 表
    Storage-->>Ability: 返回成功/失败
    Note over Ability, Storage: 触发观察者
    Ability->>Observers: onChange("brightness", "100")
    Ability-->>Helper: 返回成功/失败
    Helper-->>Native: 返回状态
    Native-->>NAPI: 返回 Promise
    NAPI-->>App: 返回结果
```

### ANI 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant ANI as ANI 层
    participant Native as Native C++
    participant Helper as DataShareHelper
    participant Ability as DataAbility
    participant Observers as 所有观察者

    App->>ANI: settings.setValue("brightness", "100", "system")
    ANI->>Native: 调用 setValue 实现
    Native->>Helper: 创建 DataShareHelper
    Helper->>Ability: Update(SET VALUE = "100"<br/>WHERE KEYWORD = "brightness")
    Ability->>Storage: 更新 system 表
    Storage-->>Ability: 返回成功/失败
    Note over Ability, Storage: 触发观察者
    Ability->>Observers: onChange("brightness", "100")
    Ability-->>Helper: 返回成功/失败
    Helper-->>Native: 返回状态
    Native-->>ANI: 返回状态（ani_boolean）
    ANI-->>App: 返回结果
```

---

## registerKeyObserver 调用链

### N-API 版本

```mermaid
sequenceDiagram
    participant App1 as 应用 A（注册者）
    participant App2 as 应用 B（修改者）
    participant NAPI as N-API 层
    participant Native as Native C++
    participant Observer as SettingsObserver
    participant Ability as DataAbility
    participant Storage as 数据库

    App1->>NAPI: settings.registerKeyObserver("brightness", "system", observer)
    Note over NAPI: unwrap observer 对象
    NAPI->>Native: 调用 registerKeyObserver
    Native->>Observer: 创建 SettingsObserver 实例
    Note over Observer: 继承 DataAbilityObserverStub
    Native->>Ability: 注册观察者（RegisterObserver）
    Ability->>Storage: 创建观察者记录
    Note over Ability, Storage: 准备监听
    Ability-->>Native: 注册成功
    Native-->>NAPI: 返回 Promise

    Note over Storage: 数据变更发生
    Storage->>Ability: 触发 onChange

    Note over Ability, Storage: 通过所有已注册的观察者
    Ability->>Observer: onChange("brightness", "新值")
    Observer->>NAPI: 回调 JS observer.onDataChange()
    NAPI->>App2: 调用回调
```

### ANI 版本

```mermaid
sequenceDiagram
    participant App1 as 应用 A（注册者）
    participant App2 as 应用 B（修改者）
    participant ANI as ANI 层
    participant Native as Native C++
    participant Observer as SettingsObserver
    participant Ability as DataAbility

    App1->>ANI: settings.registerKeyObserver("brightness", "system", observer)
    ANI->>Native: 调用 registerKeyObserver
    Native->>Observer: 创建 SettingsObserver 实例
    Note over Observer: 继承 DataAbilityObserverStub
    Native->>Ability: 注册观察者
    Ability-->>Native: 注册成功
    Native-->>ANI: 返回状态（ani_boolean）
```

---

## enableAirplaneMode 调用链

### N-API 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant NAPI as N-API 层
    participant Native as Native C++
    participant Ability as AbilityManager
    participant System as 系统服务

    App->>NAPI: settings.enableAirplaneMode(true)
    Note over NAPI: unwrap 参数
    NAPI->>Native: 调用 enableAirplaneMode 实现
    Note over Native: 实现位置待确认
    Native->>Ability: 调用 AbilityManager 接口
    Note over Ability: 可能使用 SetNetworkAirplaneMode
    Ability->>System: 请求启用飞行模式
    System-->>Ability: 返回成功/失败
    Ability-->>Native: 返回状态
    Native-->>NAPI: 返回 Promise
    NAPI-->>App: 返回结果
```

---

## canShowFloating 调用链

### N-API 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant NAPI as N-API 层
    participant Native as Native C++
    participant WindowManager as 窗口管理器

    App->>NAPI: settings.canShowFloating()
    NAPI->>Native: 调用 canShowFloating 实现
    Note over Native: 实现位置待确认
    Native->>WindowManager: 查询悬浮窗权限
    Note over WindowManager: 可能使用 CheckFloatingWindowPermission
    WindowManager-->>Native: 返回权限状态（boolean）
    Native-->>NAPI: 返回值
    NAPI-->>App: 返回 Promise
```

---

## openNetworkManagerSettings 调用链

### N-API 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant NAPI as N-API 层
    participant Native as Native C++
    participant Ability as AbilityManager
    participant Router as 路由器

    App->>NAPI: settings.openNetworkManagerSettings()
    NAPI->>Native: 调用 openNetworkManagerSettings 实现
    Note over Native: 实现位置：napi/settings/open_network_settings/napi_open_network_settings.cpp
    Native->>Ability: 获取 Want 对象
    Note over Ability: 使用 ability_base:want
    Native->>Router: router.pushUrl()
    Router->>Router: 导航到网络管理页面
    Router-->>Router: 页面跳转完成
    Native-->>NAPI: 返回 Promise
    NAPI-->>App: 返回结果
```

---

## isDoNotDisturbEnabled 调用链

### N-API 版本

```mermaid
sequenceDiagram
    participant App as ArkTS 应用
    participant NAPI as N-API 层
    participant Native as Native C++
    participant Notification as 通知服务

    App->>NAPI: intelligentscene.isDoNotDisturbEnabled()
    Note over NAPI: 智能场景模块
    NAPI->>Native: 调用 is_do_not_disturb_enabled 实现
    Native->>Notification: 查询免打扰状态
    Note over Notification: 使用 distributed_notification_service:ans_innerkits
    Notification-->>Native: 返回状态（boolean）
    Native-->>NAPI: 返回值
    NAPI-->>App: 返回结果
```

---

## 相关跳转

- **[00_Overview.md](00_Overview.md)** - 项目概览
- **[03_Architecture.md](03_Architecture.md)** - 架构说明（更详细的时序图）
- **[04_NAPI_API.md](04_NAPI_API.md)** - 对外 API 文档

---

**最后更新**：2026-02-06 00:11:23
