# ui_lite 架构说明

## 文档信息

- **文档用途**: 描述 ui_lite 的系统架构、组件关系、数据流和线程模型
- **适用范围**: 架构师、开发者、安全评审人员
- **相关文档**: [项目概览](01_Overview.md), [目录结构](03_Directory_Structure.md)

## 架构概览

ui_lite 采用分层架构设计，从下到上依次为：硬件抽象层、引擎层、核心层、组件层和接口层。

```mermaid
graph TB
    subgraph "应用层"
        APP[应用程序]
    end
    
    subgraph "接口层 (interfaces/)"
        KITS[kits/ 对外API]
        INNER[innerkits/ 内部API]
    end
    
    subgraph "框架层 (frameworks/)"
        COMP[components/ 组件系统]
        LAYOUT[layout/ 布局系统]
        ANIM[animator/ 动画系统]
        EVENT[events/ 事件系统]
        FONT[font/ 字体系统]
        DRAW[draw/ 绘制系统]
        RENDER[render/ 渲染系统]
        CORE[core/ 核心管理器]
        DFX[dfx/ 调试诊断]
    end
    
    subgraph "适配层 (dock/)"
        INPUT[input_device/ 输入设备]
        SCREEN[screen_device/ 显示设备]
        OHOS[ohos/ OHOS平台适配]
    end
    
    subgraph "引擎层 (engines/)"
        GFX[gfx/ 图形引擎]
    end
    
    subgraph "依赖层"
        WMS[Window Manager Service]
        HAL[Hardware HAL]
        UTILS[graphic_utils_lite]
    end
    
    APP --> KITS
    KITS --> COMP
    KITS --> ANIM
    KITS --> EVENT
    INNER --> CORE
    INNER --> GFX
    
    COMP --> LAYOUT
    COMP --> DRAW
    DRAW --> RENDER
    RENDER --> GFX
    
    ANIM --> CORE
    EVENT --> INPUT
    FONT --> DRAW
    
    CORE --> TASK[task_manager]
    CORE --> RENDER_M[render_manager]
    
    INPUT --> OHOS
    GFX --> HAL
    
    COMP --> UTILS
    DRAW --> UTILS
```

## 核心组件架构

### 1. 视图层次结构

```mermaid
classDiagram
    class HeapBase {
        <<基类>>
    }
    
    class UIView {
        +Rect rect_
        +Style style_
        +OnClickListener* onClickListener_
        +OnDragListener* onDragListener_
        +TransformMap transMap_
        +SetPosition()
        +SetSize()
        +OnDraw()
        +OnEvent()
    }
    
    class UIViewGroup {
        +List~UIView*~ children_
        +Add()
        +Remove()
        +GetChildById()
    }
    
    class RootView {
        +GetInstance() RootView*
        +Render()
        +OnKeyEvent()
        +Invalidate()
    }
    
    class Window {
        +CreateWindow() Window*
        +BindRootView()
        +Show()
        +Hide()
    }
    
    HeapBase <|-- UIView
    UIView <|-- UIViewGroup
    UIViewGroup <|-- RootView
    Window --> RootView
```

**证据**:
- `UIView` 基类: [interfaces/kits/components/ui_view.h](interfaces/kits/components/ui_view.h)
- `UIViewGroup` 容器: [interfaces/kits/components/ui_view_group.h](interfaces/kits/components/ui_view_group.h)
- `RootView` 根视图: [interfaces/kits/components/root_view.h](interfaces/kits/components/root_view.h)
- `Window` 窗口: [interfaces/kits/window/window.h](interfaces/kits/window/window.h)

### 2. 组件继承体系

```mermaid
graph TD
    A[UIView] --> B[UIViewGroup]
    A --> C[UILabel]
    A --> D[UIButton]
    A --> E[UIImageView]
    A --> F[UIProgressBar]
    A --> G[UICanvas]
    
    B --> H[UIScrollView]
    B --> I[UIList]
    B --> J[UISwipeView]
    B --> K[UIDialog]
    B --> L[UILayout]
    
    L --> M[FlexLayout]
    L --> N[GridLayout]
    
    F --> O[UIBoxProgress]
    F --> P[UICircleProgress]
    
    D --> Q[UILabelButton]
    D --> R[UICheckBox]
    D --> S[UIRadioButton]
```

## 数据流

### 渲染数据流

```mermaid
sequenceDiagram
    participant App as 应用
    participant RV as RootView
    participant RM as RenderManager
    participant GE as GfxEngine
    participant HAL as Hardware HAL
    
    App->>RV: Invalidate()
    RV->>RV: AddInvalidateRect()
    
    loop TaskHandler
        RV->>RM: Render()
        RM->>RM: ProcessInvalidRects()
        
        loop ForEachView
            RM->>GE: DrawRect()
            RM->>GE: DrawImage()
            RM->>GE: DrawLetter()
        end
        
        GE->>HAL: Flush()
    end
```

**关键代码路径**:
1. `RootView::Invalidate()` → `AddInvalidateRect()` ([frameworks/components/root_view.cpp](frameworks/components/root_view.cpp))
2. `RenderManager::Render()` → 处理脏矩形 ([frameworks/core/render_manager.cpp](frameworks/core/render_manager.cpp))
3. `BaseGfxEngine::DrawRect/DrawImage/DrawLetter` ([interfaces/innerkits/engines/gfx/gfx_engine_manager.h](interfaces/innerkits/engines/gfx/gfx_engine_manager.h))

### 事件数据流

```mermaid
sequenceDiagram
    participant Device as 输入设备
    participant IDM as InputDeviceManager
    participant RV as RootView
    participant View as UIView
    participant Listener as EventListener
    
    Device->>IDM: 读取输入事件
    IDM->>IDM: 分发事件
    IDM->>RV: OnKeyEvent/OnPointEvent
    
    RV->>RV: GetTargetView()
    RV->>View: OnClickEvent/OnPressEvent
    
    alt 有监听器
        View->>Listener: OnClick/OnPress
        Listener-->>View: 返回消费状态
    end
    
    alt 未消费
        View->>RV: 冒泡到父视图
    end
```

**关键代码路径**:
1. `InputDeviceManager` 事件分发 ([interfaces/innerkits/common/input_device_manager.h](interfaces/innerkits/common/input_device_manager.h))
2. `RootView::GetTargetView()` 命中测试 ([frameworks/components/root_view.cpp](frameworks/components/root_view.cpp))
3. `UIView::OnClickEvent/OnPressEvent` 事件处理 ([interfaces/kits/components/ui_view.h](interfaces/kits/components/ui_view.h))

## 线程模型

### 主线程架构

```
┌─────────────────────────────────────────────────────────────┐
│                        主线程 (UI Thread)                    │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   事件循环    │  │   动画更新    │  │   渲染输出    │       │
│  │              │  │              │  │              │       │
│  │ InputDevice  │  │ Animator     │  │ RenderManager│       │
│  │   → Event    │  │   → Task     │  │   → GfxEngine│       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              TaskManager::TaskHandler()               │  │
│  │  统一任务调度：输入事件 + 动画任务 + 渲染任务          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 任务调度机制

```cpp
// 证据: interfaces/innerkits/common/task_manager.h
class TaskManager : public HeapBase {
    void Add(Task* task);      // 添加任务
    void Remove(Task* task);   // 移除任务
    void TaskHandler();        // 主循环调用
    void SetTaskRun(bool enable);
};
```

**任务类型**:
1. **输入任务** - 读取输入设备状态
2. **动画任务** - 更新动画进度
3. **渲染任务** - 执行绘制操作

### 线程安全

```cpp
// 证据: frameworks/components/root_view.h:342
#if defined __linux__ || defined __LITEOS__ || defined __APPLE__
    pthread_mutex_t lock_;   // 脏矩形操作锁
#endif
```

**线程安全策略**:
- RootView 使用互斥锁保护脏矩形列表
- 组件树操作应在同一线程完成
- 图像解码可能使用后台线程（取决于平台）

## 关键时序

### 启动时序

```mermaid
sequenceDiagram
    participant Main as main()
    participant GS as GraphicStartUp
    participant TM as TaskManager
    participant RV as RootView
    participant WMS as WindowManager
    
    Main->>GS: Init()
    GS->>GS: 初始化字体引擎
    GS->>GS: 初始化 ICU
    
    Main->>RV: GetInstance()
    RV->>RV: 创建单例
    
    Main->>TM: GetInstance()
    Main->>TM: SetTaskRun(true)
    
    Main->>WMS: CreateWindow()
    WMS->>RV: BindRootView()
    
    loop TaskHandler
        TM->>TM: 处理任务队列
    end
```

### 窗口创建时序

```mermaid
sequenceDiagram
    participant App as Application
    participant W as Window
    participant RV as RootView
    participant WM as WindowImpl
    
    App->>W: CreateWindow(config)
    W->>WM: new WindowImpl()
    WM->>WMS: 申请 Surface
    WMS-->>WM: 返回 buffer
    
    App->>RV: GetWindowRootView()
    RV->>RV: new RootView()
    
    App->>W: BindRootView(rv)
    W->>RV: 建立关联
    
    App->>W: Show()
    W->>WM: 显示到屏幕
```

## 模块依赖关系

### 依赖图

```mermaid
graph LR
    subgraph "核心模块"
        CORE[core]
        COMP[components]
        EVENT[events]
    end
    
    subgraph "渲染模块"
        DRAW[draw]
        RENDER[render]
        GFX[gfx_engine]
    end
    
    subgraph "功能模块"
        ANIM[animator]
        FONT[font]
        IMG[imgdecode]
        LAYOUT[layout]
        THEME[themes]
    end
    
    subgraph "适配模块"
        DOCK[dock]
        WINDOW[window]
    end
    
    COMP --> CORE
    COMP --> DRAW
    COMP --> EVENT
    
    DRAW --> RENDER
    RENDER --> GFX
    
    ANIM --> CORE
    FONT --> DRAW
    IMG --> DRAW
    LAYOUT --> COMP
    THEME --> COMP
    
    WINDOW --> DOCK
    DOCK --> EVENT
```

### 依赖规则

1. **上层依赖下层** - 组件层依赖核心层
2. **同层可互调** - 组件之间可互相引用
3. **禁止循环依赖** - 模块间无循环依赖
4. **接口隔离** - 通过 interfaces 暴露，frameworks 实现

## 内存管理

### 对象生命周期

```mermaid
graph TD
    A[创建 UIView] --> B{添加到父视图?}
    B -->|是| C[父视图管理生命周期]
    B -->|否| D[开发者手动管理]
    
    C --> E[Remove 或父视图销毁]
    E --> F[自动 delete]
    
    D --> G[开发者调用 delete]
```

### 内存分配策略

```cpp
// 证据: 继承自 HeapBase
class UIView : public HeapBase {
    // 使用统一的内存分配器
};
```

**策略**:
- 所有 UI 对象继承 `HeapBase`，支持自定义分配器
- 视图树由父视图统一管理子视图生命周期
- 字体缓存、图像缓存使用 LRU 策略

## 扩展点

### 1. 自定义组件

```cpp
// 继承 UIView 实现自定义绘制
class MyCustomView : public UIView {
protected:
    void OnDraw(BufferInfo& gfxDstBuffer, const Rect& invalidatedArea) override;
};
```

### 2. 自定义布局

```cpp
// 继承 UIViewGroup 实现自定义布局
class MyLayout : public UIViewGroup {
public:
    void LayoutChildren(bool needInvalidate = false) override;
};
```

### 3. 自定义引擎

```cpp
// 继承 BaseGfxEngine 实现硬件加速
class MyGfxEngine : public BaseGfxEngine {
public:
    void DrawRect(...) override;
    void DrawImage(...) override;
};
```

## 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [目录结构](03_Directory_Structure.md) - 代码组织
- [对外 API](04_Public_API.md) - API 详细说明
- [内部 API](05_Internal_API.md) - 模块间接口
