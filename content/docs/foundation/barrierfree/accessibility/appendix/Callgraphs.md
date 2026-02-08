# 调用链图（附录）

## 目的

本文档提供 Accessibility 子系统关键 API 的调用链示例，帮助理解从 JS API 到核心逻辑的完整流程。

## 适用范围

- 需要理解调用流程的开发者
- 调试问题的开发者
- 架构师

## 关键结论

### 主要调用链类型

1. **配置调用链** - 设置/获取无障碍配置
2. **元素查询调用链** - 查询无障碍元素信息
3. **手势注入调用链** - 注入手势操作
4. **事件分发调用链** - 无障碍事件分发

---

## 详细内容

### 1. 配置调用链

#### 场景：应用设置屏幕放大镜状态

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as accessibility 模块
    participant ASACkit as AccessibilityConfig
    participant Proxy as Config Proxy
    participant AAMS as AccessibilityService
    participant Settings as 配置存储

    App->>NAPI: setMagnificationState(true)
    NAPI->>ASACkit: SetScreenMagnificationState(true)
    ASACkit->>Proxy: SetScreenMagnificationState(state=true)
    Proxy->>AAMS: IPC: SetScreenMagnificationState
    AAMS->>Settings: 更新配置
    Settings->>AAMS: 配置变化通知
    AAMS->>AAMS: 更新放大镜状态
    AAMS->>Proxy: 返回成功
    Proxy->>ASACkit: 返回结果
    ASACkit->>NAPI: 返回结果
    NAPI->>App: Promise resolved
```

**证据**:
- N-API: `interfaces/kits/napi/src/native_module.cpp:2790` (SetMagnificationState)
- ASACkit: `frameworks/acfwk/src/accessibility_config.cpp`
- Proxy: `common/interface/src/accessible_ability_manager_config_observer_proxy.cpp`
- Service: `services/aams/src/accessible_ability_manager_service.cpp:2740`

### 2. 元素查询调用链

#### 场景：无障碍扩展查询焦点元素

```mermaid
sequenceDiagram
    participant Ext as 无障碍扩展
    participant NAPI as ExtensionContext
    participant Proxy as Channel Proxy
    participant AAMS as AccessibilityService
    participant Ace as ACE 框架
    participant Target as 目标应用

    Ext->>NAPI: getFocusElement()
    NAPI->>Proxy: GetFocusElementInfo()
    Proxy->>AAMS: IPC: GetFocusElementInfo
    AAMS->>AAMS: 验证 Token ID
    AAMS->>Ace: GetFocusElementInfo
    Ace->>Target: 查询焦点元素
    Target->>Ace: 返回元素信息
    Ace->>AAMS: 返回元素信息
    AAMS->>Proxy: 返回元素信息
    Proxy->>NAPI: 转换为 JS 对象
    NAPI->>Ext: 返回 AccessibilityElement
```

**证据**:
- N-API: `interfaces/kits/napi/accessibility_extension_context/napi_accessibility_extension_context.cpp`
- Proxy: `common/interface/src/accessibility_element_operator_proxy.cpp`
- Service: `services/aams/src/accessibility_element_operator_stub.cpp`

### 3. 手势注入调用链

#### 场景：无障碍扩展注入手势

```mermaid
sequenceDiagram
    participant Ext as 无障碍扩展
    participant NAPI as ExtensionContext
    participant Proxy as Channel Proxy
    participant AAMS as AccessibilityService
    participant Input as Input Manager
    participant App as 目标应用

    Ext->>NAPI: injectGesture(gesturePath)
    NAPI->>Proxy: SendSimulateGesturePath()
    Proxy->>AAMS: IPC: SendSimulateGesturePath
    AAMS->>AAMS: 验证权限
    AAMS->>Input: 注入手势事件
    Input->>App: 分发触摸事件
    App->>App: 执行手势操作
    Input->>AAMS: 手势完成通知
    AAMS->>Proxy: 返回成功
    Proxy->>NAPI: 返回结果
    NAPI->>Ext: Promise resolved
```

**证据**:
- N-API: `interfaces/kits/napi/accessibility_extension_context/napi_accessibility_extension_context.cpp`
- Proxy: `common/interface/src/accessibility_element_operator_proxy.cpp`
- Service: `services/aams/src/accessibility_touchEvent_injector.cpp`

### 4. 事件分发调用链

#### 场景：目标应用发送无障碍事件

```mermaid
sequenceDiagram
    participant App as 目标应用
    participant Ace as ACE 框架
    participant AAMS as AccessibilityService
    participant Channel as Ability Channel
    participant Ext as 无障碍扩展

    App->>Ace: UI 变化
    Ace->>AAMS: SendEvent(eventInfo)
    AAMS->>AAMS: 过滤和路由
    AAMS->>Channel: OnAccessibilityEvent(eventInfo)
    Channel->>Ext: 事件回调
    Ext->>Ext: 处理事件
```

**证据**:
- ACE: 集成层（假设）
- Service: `services/aams/src/accessibility_event_transmission.cpp`
- Channel: `common/interface/src/accessible_ability_channel_stub.cpp`

---

## 相关链接

- [项目概览](00_Overview.md)
- [架构说明](03_Architecture.md)
- [N-API 接口](04_N-API.md)
- [内部 API](05_Inner_API.md)

---

最后更新: 2026-02-06
