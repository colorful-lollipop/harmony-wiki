# 内部 API 与模块接口 (Inner API)

## 概述

本文档描述 arkui_cangjie_wrapper 的内部模块接口，包括模块间依赖方向、稳定性标注和可替换点。

> **注意**: 内部 API 可能在版本间发生变化，不建议外部直接使用。

---

## 模块依赖关系图

```
kit.ArkUI (对外 Stable)
├── ohos.arkui.component (Stable)
│   ├── ohos.arkui.component.common (Stable)
│   ├── 80+ individual components (Stable)
│   └── external: ace_engine:cj_frontend_ohos
│
├── ohos.arkui.component_utils (Stable)
├── ohos.arkui.shape (Stable)
├── ohos.arkui.state_management (Stable)
├── ohos.arkui.ui_context (Stable)
│   ├── cj_animator
│   ├── cj_font
│   ├── cj_measure
│   ├── cj_prompt_action
│   ├── cj_router
│   └── cj_ui_context
│
├── ohos.base (Stable)
│   └── external: cangjie_ark_interop, hiviewdfx_cangjie_wrapper
│
├── ohos.curves (Stable)
│
└── external: window_cangjie_wrapper
```

---

## 状态管理模块 (State Management)

### 稳定性: Stable

**路径**: `ohos/arkui/state_management/`

### 内部类型

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| **ObservedPropertyAbstract** | Unstable | 可观察属性抽象基类 |
| **SubscriberManager** | Unstable | 订阅者管理 |
| **ViewStackProcessor** | Unstable | 视图栈处理 |
| **LocalStorageInterOp** | Unstable | JS 互操作 |

### 依赖方向

```
LocalStorage
    ├── HashMap<String, ObservedPropertyAbstract>
    ├── SubscriberManager
    └── Observable

AppStorage (继承 LocalStorage)
    └── Environment
```

### 可替换点

| 组件 | 可替换点 | 说明 |
|------|----------|------|
| SubscriberManager | 订阅算法 | 可替换为更高效的发布-订阅实现 |
| Observable | 通知机制 | 可扩展为分布式观察者 |

---

## UI 上下文模块 (UI Context)

### 稳定性: Stable

**路径**: `ohos/arkui/ui_context/`

### Router 模块

**路径**: `ohos/arkui/ui_context/cj_router.cj`

#### 内部类型

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| **RouterOptions** | Stable | 路由参数 |
| **RouterState** | Stable | 路由状态 |
| **Router** | Stable | 路由控制器 |

#### 依赖

```
Router
    └── ace_engine:cj_frontend_ohos
```

### PromptAction 模块

**路径**: `ohos/arkui/ui_context/cj_prompt_action.cj` (46KB)

#### 内部类型

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| **ToastOptions** | Stable | Toast 参数 |
| **DialogOptions** | Stable | 对话框参数 |
| **ActionMenuOptions** | Stable | 操作菜单参数 |
| **PromptAction** | Stable | 弹窗控制器 |

### Animator 模块

**路径**: `ohos/arkui/ui_context/cj_animator.cj` (20KB)

#### 内部类型

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| **AnimatorOptions** | Stable | 动画参数 |
| **Animator** | Stable | 动画控制器 |
| **AnimateOptions** | Stable | 补间动画参数 |

---

## 基础类型模块 (Base)

### 稳定性: Stable

**路径**: `ohos/base/`

### 内部类型

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| **LengthImpl** | Unstable | 长度实现 |
| **ColorType** | Unstable | 颜色实现 |
| **ResourceImpl** | Unstable | 资源实现 |

### 依赖

```
ohos.base
    ├── cangjie_ark_interop:ohos.ffi
    ├── cangjie_ark_interop:ohos.labels
    ├── cangjie_ark_interop:ohos.business_exception
    └── hiviewdfx_cangjie_wrapper:ohos.hilog
```

---

## 组件通用模块 (Component Common)

### 稳定性: Stable

**路径**: `ohos/arkui/component/common/`

### 通用属性类型

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| **CommonAttribute** | Stable | 通用属性接口 |
| **TouchEvent** | Stable | 触摸事件 |
| **GestureEvent** | Stable | 手势事件 |
| **FocusEvent** | Stable | 焦点事件 |

### 通用事件回调

```cangjie
// 通用事件接口示例
public interface ClickEventResult {
    func onClick(): Void
}

public interface TouchEventResult {
    func onTouch(event: TouchEvent): Void
}
```

---

## 状态宏管理 (State Macro Management)

### 稳定性: Unstable

**路径**: `ohos/arkui/state_macro_manage/`

### 宏定义

| 宏 | 稳定性 | 说明 |
|-----|--------|------|
| **@State** | Stable | 组件状态 |
| **@Prop** | Stable | 父子单向 |
| **@Link** | Stable | 父子双向 |
| **@Provide** | Stable | 跨层级提供 |
| **@Consume** | Stable | 跨层级消费 |
| **@ObjectLink** | Unstable | 对象链接 |

### 实现机制

```
@State 宏
    ├── 编译时: 属性装饰
    └── 运行时: ObservableProperty 包装
```

---

## 模块间数据流

### 状态更新流

```
┌──────────────┐     @State      ┌────────────────────┐
│   Component  │ ──────────────> │ ObservedProperty  │
│              │                 │ (Observable)       │
└──────────────┘                 └─────────┬──────────┘
                                           │
                                           v
                                 ┌────────────────────┐
                                 │   SubscriberMgr   │
                                 │   (观察者注册)      │
                                 └─────────┬──────────┘
                                           │
                    ┌──────────────────────┼──────────────────────┐
                    │                      │                      │
                    v                      v                      v
           ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
           │   Component  │       │   Component  │       │   Component  │
           │   (重建)     │       │   (重建)     │       │   (重建)     │
           └──────────────┘       └──────────────┘       └──────────────┘
```

### 路由流

```
┌──────────────┐     pushUrl      ┌────────────────────┐     ace_engine
│   Component  │ ──────────────> │   Router           │ ──────────────>│
└──────────────┘                 │   (页面栈管理)      │                 │
                                 └────────────────────┘                 │
                                                                            v
                                                           ┌────────────────────┐
                                                           │   页面渲染          │
                                                           └────────────────────┘
```

---

## 稳定性标注规则

| 标注 | 含义 | 使用建议 |
|------|------|----------|
| **Stable** | 公开 API | 正常使用 |
| **Unstable** | 内部实现 | 避免直接使用 |
| **@Hide** | 隐藏 API | 禁止使用 |

---

## 错误码参考

### 基础错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 100001 | `Internal error` | 内部错误 |

### 错误码使用示例

```cangjie
// Length 接口中的错误码使用
prop value: Float64 {
    get() {
        throw BusinessException(100001, "Internal error.")
    }
}
```

---

## 代码证据

| 模块 | 证据位置 |
|------|----------|
| 状态管理 | `ohos/arkui/state_management/local_storage.cj:34-100` |
| 路由 | `ohos/arkui/ui_context/cj_router.cj` |
| 弹窗 | `ohos/arkui/ui_context/cj_prompt_action.cj` |
| 基础类型 | `ohos/base/length.cj:31-60` |

---

## 相关文档

- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [03_N-API.md](./03_N-API.md) - 对外 API
- [05_GN_Build.md](./05_GN_Build.md) - 构建系统
