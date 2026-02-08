# 架构说明 (Architecture)

## 系统架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         应用层 (Application)                          │
│                    @Component + build() 声明式 UI                     │
├─────────────────────────────────────────────────────────────────────┤
│                       接口层 (Interface Layer)                        │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────┐ │
│  │   UI Components │ │   UI Context    │ │ State Management Macros │ │
│  │  80+ 组件声明   │ │ 动画/路由/字体   │ │ @State/@Prop/@Link     │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────┤
│                      框架层 (Framework Layer)                         │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │  Cangjie ArkUI 开发框架 FFI 接口定义                            │  │
│  │  (C 语言互操作层，连接 Cangjie 前端与 ArkUI 引擎)                │  │
│  └────────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│                       引擎层 (Engine Layer)                          │
│                    ace_engine:cj_frontend_ohos                        │
│                    (Cangjie 前端 + UI 渲染引擎)                       │
├─────────────────────────────────────────────────────────────────────┤
│                      系统能力层 (System Layer)                        │
│  access_token │ global │ multimedia │ arkweb │ hiviewdfx            │
└─────────────────────────────────────────────────────────────────────┘
```

## 分层说明

### 接口层 (Interface Layer)

**位置**: `kit/ArkUI/index.cj`

为开发者提供 Cangjie 语言的 UI 开发接口：

| 子模块 | 职责 |
|--------|------|
| **UI Components** | 声明式 UI 组件（Text, Button, Column 等） |
| **UI Context** | 动画控制器、页面路由、自定义字体、文本测量 |
| **State Management Macros** | `@State`, `@Prop`, `@Link` 等状态装饰器 |

### 框架层 (Framework Layer)

**位置**: `ohos/arkui/`

Cangjie 语言的 UI 框架实现：

| 子模块 | 职责 |
|--------|------|
| **UI Component Encapsulation** | 组件封装（事件、属性、生命周期） |
| **UI Context Encapsulation** | 上下文能力封装（动画、弹窗、路由） |
| **State Management** | 状态管理框架（观察者模式实现） |
| **FFI Interface Definitions** | C 语言互操作接口定义 |

### 引擎层 (Engine Layer)

**外部依赖**: `ace_engine:cj_frontend_ohos`

提供：
- Cangjie 编译器前端
- UI 渲染引擎
- 事件分发系统

## 数据流

### 单向数据流

```
┌──────────────┐     状态变化      ┌─────────────────┐
│   Component  │ ───────────────> │ State Management │
│   (@State)   │                   │ (Observable)     │
└──────────────┘                   └────────┬────────┘
                                            │
                                            v
                                   ┌─────────────────┐
                                   │   UI Re-render  │
                                   │ (ace_engine)    │
                                   └─────────────────┘
```

### 双向绑定

```
┌──────────────┐    @Link    ┌──────────────┐
│   Parent     │ <─────────> │    Child      │
│   @State     │             │   @Link       │
└──────────────┘             └──────────────┘
```

## 状态管理架构

### 观察者模式

```
┌─────────────────────────────────────────────────────────────┐
│                     Observable Property                      │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    变化通知    ┌─────────────────────┐ │
│  │ ObservedProperty│ ────────────> │ Subscriber Manager  │ │
│  │                 │               │ (订阅者管理)         │ │
│  └─────────────────┘               └──────────┬──────────┘ │
│                                               │             │
│                              ┌────────────────┼────────────┤ │
│                              │                │            │ │
│                              v                v            v │
│                     ┌──────────────┐  ┌──────────────┐  ... │
│                     │   Component  │  │   Component  │      │
│                     │  (订阅重建)   │  │  (订阅重建)   │      │
│                     └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 存储层级

```
┌─────────────────────────────────────────┐
│           AppStorage (应用级)             │
│         全局、单例生命周期                 │
├─────────────────────────────────────────┤
│         LocalStorage (组件实例)           │
│       组件级、可跨页面共享                │
├─────────────────────────────────────────┤
│            @State (组件内)               │
│         组件内部、局部状态                │
└─────────────────────────────────────────┘
```

## 组件渲染流程

```
1. Cangjie 源码解析
   └── @Component + build() 函数解析

2. 组件树构建
   └── 创建 Component Node 树

3. 状态初始化
   └── @State, @Prop, @Link 绑定

4. 渲染指令生成
   └── 生成 UI 描述指令

5. 引擎渲染
   └── ace_engine 接收指令并渲染

6. 事件响应
   └── 用户交互 → 事件分发 → 状态更新

7. 增量更新
   └── 状态变化 → 仅重建受影响组件
```

## 线程模型

### 主线程架构

```
┌─────────────────────────────────────────────────────────────┐
│                      Main Thread                            │
│  ┌─────────────┐ ┌─────────────┐ ┌───────────────────────┐ │
│  │ UI Thread   │ │ JS Thread   │ │ Cangjie Thread        │ │
│  │ (渲染)      │ │ (ArkTS)     │ │ (Cangjie 运行时)      │ │
│  └─────────────┘ └─────────────┘ └─────────────────────┬───┘ │
│                                                       │     │
│                         Shared State                   │     │
│                         & Event Bus                    │     │
└───────────────────────────────────────────────────────┼─────┘
                                                        │
                                                        v
                                            ┌───────────────────────┐
                                            │   Ace Engine Core     │
                                            │   (UI 渲染引擎)        │
                                            └───────────────────────┘
```

### 关键约束

| 线程 | 职责 | 注意事项 |
|------|------|----------|
| **UI Thread** | 渲染、布局、绘制 | 禁止阻塞 |
| **Cangjie Thread** | Cangjie 运行时 | 避免长时间计算 |
| **JS Thread** | ArkTS 运行时 | 与 Cangjie 隔离 |

## 组件交互模式

### 父子组件通信

```cangjie
// Parent.ci
@Component
struct Parent {
    @State count: Int64 = 0
    
    func build() {
        Column {
            ChildComponent(count: this.count)
            Button("Increment")
                .onClick(() => {
                    this.count++
                })
        }
    }
}

// Child.cj
@Component
struct ChildComponent {
    @Prop count: Int64
    
    func build() {
        Text("Count: \(${this.count})")
    }
}
```

### 跨层级通信

```cangjie
// Provider.ci
@Component
struct Provider {
    @Provide message: String = "Hello"
    
    func build() {
        Child {
            GrandChild()  // 可访问 message
        }
    }
}

// Consumer.cj
@Component
struct GrandChild {
    @Consume message: String
    
    func build() {
        Text(this.message)
    }
}
```

## 关键时序图

### 页面跳转时序

```
┌──────┐     ┌──────┐     ┌──────────┐     ┌──────┐     ┌────────┐
│ User │     │ View │     │ Router   │     │ Ace  │     │ Engine │
│      │     │      │     │          │     │ Eng  │     │        │
└──┬───┘     └──┬───┘     └────┬─────┘     └──┬───┘     └────┬───┘
   │            │               │               │               │
   │ onClick    │               │               │               │
   │──────────> │               │               │               │
   │            │ pushUrl       │               │               │
   │            │──────────────>│               │               │
   │            │               │ Update Stack  │               │
   │            │               │───────────────>               │
   │            │               │               │ Render        │
   │            │               │               │──────────────> │
   │            │               │               │               │ Paint
   │            │               │               │               │─────>
```

### 状态更新时序

```
┌──────┐     ┌──────────┐     ┌─────────────────┐     ┌──────┐     ┌──────┐
│ User │     │ Callback │     │ ObservedProperty│     │ Sub  │     │ Eng  │
│      │     │          │     │                 │     │ Mgr  │     │      │
└──┬───┘     └────┬─────┘     └────────┬────────┘     └──┬───┘     └──┬───┘
   │              │                    │                    │             │
   │ onClick      │                    │                    │             │
   │─────────────>│                    │                    │             │
   │              │ Set Value          │                    │             │
   │              │────────────────---->                    │             │
   │              │                    │ Notify Change     │             │
   │              │                    │───────────────────>             │
   │              │                    │                    │ Notify      │
   │              │                    │                    │─────────────>
   │              │                    │                    │             │ Re-render
   │              │                    │                    │             │─────────>
```

## 代码证据

| 架构元素 | 证据位置 |
|----------|----------|
| Kit 入口 | `kit/ArkUI/index.cj:16-42` |
| 组件聚合 | `ohos/arkui/component/component.cj:18-103` |
| 状态管理 | `ohos/arkui/state_management/*.cj` |
| UI 上下文 | `ohos/arkui/ui_context/*.cj` |
| 路由 | `ohos/arkui/ui_context/cj_router.cj` |
| 弹窗 | `ohos/arkui/ui_context/cj_prompt_action.cj` |

## 相关文档

- [00_Overview.md](./00_Overview.md) - 项目概览
- [01_Directory_Structure.md](./01_Directory_Structure.md) - 目录结构
- [03_N-API.md](./03_N-API.md) - API 参考
- [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 关键调用链
