# 内部模块接口 (Inner API)

## 概述

Inner API 供 OpenHarmony 系统内部模块使用，主要包括：
- Render Service 内部接口
- Native SDK 接口
- 平台适配层接口

---

## Render Service 客户端 API

### 节点类层次

**基类**: `RSNode` (`rosen/modules/render_service_client/core/ui/rs_node.h`)

```cpp
class RSNode : public std::enable_shared_from_this<RSNode> {
    // 节点操作
    void AddChild(SharedPtr child, int index = -1);
    void RemoveChild(SharedPtr child);
    void RemoveFromTree();
    void ClearChildren();
    
    // 属性操作
    NodeId GetId() const;
    bool IsInstanceOf(RSUINodeType type) const;
    
    // 修饰器
    void AddModifier(const std::shared_ptr<RSModifier>& modifier);
    void RemoveModifier(ModifierId id);
};
```

### 节点派生类

| 类名 | 文件 | 职责 |
|------|------|------|
| **RSDisplayNode** | `rs_display_node.h` | 显示节点 |
| **RSSurfaceNode** | `rs_surface_node.h` | Surface 节点 |
| **RSCanvasNode** | `rs_canvas_node.h` | Canvas 绘制节点 |
| **RSEffectNode** | `rs_effect_node.h` | 效果节点 |
| **RSRootNode** | `rs_root_node.h` | 根节点 |
| **RSProxyNode** | `rs_proxy_node.h` | 代理节点 |
| **RSUnionNode** | `rs_union_node.h` | 联合节点 |

**证据来源**: `rosen/modules/render_service_client/core/ui/*.h`

---

## 动画 API

### 客户端动画

**基类**: `RSAnimation` (`rs_animation.h`)

```cpp
class RSAnimation {
    void Start(const std::shared_ptr<RSNode>& target);
    void Pause();
    void Resume();
    void Finish();
    void Reverse();
    void SetFraction(float fraction);
    bool IsRunning() const;
};
```

### 动画派生类

| 类名 | 说明 |
|------|------|
| **RSPropertyAnimation** | 属性动画基类 |
| **RSCurveAnimation** | 曲线动画 |
| **RSSpringAnimation** | 弹簧动画 |
| **RSKeyframeAnimation** | 关键帧动画 |
| **RSPathAnimation** | 路径动画 |
| **RSTransition** | 转场动画 |

### 动画时间曲线

```cpp
class RSAnimationTimingCurve {
    static RSAnimationTimingCurve DEFAULT;
    static RSAnimationTimingCurve LINEAR;
    static RSAnimationTimingCurve EASE;
    static RSAnimationTimingCurve EASE_IN;
    static RSAnimationTimingCurve EASE_OUT;
    static RSAnimationTimingCurve EASE_IN_OUT;
    
    static RSAnimationTimingCurve CreateCustomCurve(
        std::function<float(float)> curve);
    static RSAnimationTimingCurve CreateCubicCurve(
        float, float, float, float);
    static RSAnimationTimingCurve CreateStepsCurve(
        int, StepType);
    static RSAnimationTimingCurve CreateSpringCurve(
        float, float, float, float);
};
```

**证据来源**: `rosen/modules/render_service_client/core/animation/`

---

## Modifier API

### RSModifier (客户端)

```cpp
class RSModifier {
    ModifierId GetId() const;
    RSModifierType GetType() const;
    void OnAttach(RSNode& node);
    void OnDetach();
    void AttachProperty(const std::shared_ptr<RSPropertyBase>& property);
    void SetDirty(bool isDirty);
};
```

### Modifier 类型

| 类型 | 说明 |
|------|------|
| **RSFrameModifier** | 框架修饰 |
| **RSBoundsModifier** | 边界修饰 |
| **RSTransformModifier** | 变换修饰 |
| **RSAlphaModifier** | 透明度修饰 |
| **RSVisibilityModifier** | 可见性修饰 |
| **RSBorderModifier** | 边框修饰 |
| **RSShadowModifier** | 阴影修饰 |
| **RSBackgroundColorModifier** | 背景色修饰 |
| **RSForegroundColorModifier** | 前景色修饰 |

**证据来源**: `rosen/modules/render_service_client/core/modifier_ng/`

---

## Render Service 基础 API

### Render Node (服务端)

**基类**: `RSRenderNode` (`rs_render_node.h`)

```cpp
class RSRenderNode : public std::enable_shared_from_this<RSRenderNode> {
    // 节点操作
    void AddChild(SharedPtr child, int index = -1);
    void RemoveChild(SharedPtr child, bool skipTransition = false);
    void RemoveFromTree(bool skipTransition = false);
    
    // 渲染准备
    void QuickPrepare(const std::shared_ptr<RSNodeVisitor>& visitor);
    void Prepare(const std::shared_ptr<RSNodeVisitor>& visitor);
    void Process(const std::shared_ptr<RSNodeVisitor>& visitor);
    
    // 状态查询
    bool IsDirty() const;
    bool IsSubTreeDirty() const;
    void SetSubTreeDirty(bool val);
};
```

**证据来源**: `rosen/modules/render_service_base/include/pipeline/`

---

## IPC 接口 (Inner API)

### 主服务接口

**RSIRenderService** (`rs_irender_service.h`)

```cpp
class RSIRenderService : public IRemoteBroker {
    virtual std::pair<sptr<RSIClientToServiceConnection>, 
                      sptr<RSIClientToRenderConnection>>
        CreateConnection(const sptr<RSIConnectionToken>& token) = 0;
    virtual bool RemoveConnection(
        const sptr<RSIConnectionToken>& token) = 0;
};
```

### 客户端到服务连接

**RSIClientToServiceConnection** (`rs_iclient_to_service_connection.h`)

包含 150+ 方法，包括：
- 节点创建: `CreateNode()`, `CreateNodeAndSurface()`
- 屏幕管理: `CreateVirtualScreen()`, `SetScreenPowerStatus()`
- 截屏: `TakeSurfaceCapture()`, `GetPixelmap()`
- 刷新率: `SetScreenRefreshRate()`, `SyncFrameRateRange()`

### 客户端到渲染连接

**RSIClientToRenderConnection** (`rs_iclient_to_render_connection.h`)

包含渲染线程直接通信方法：
- `CommitTransaction()`, `ExecuteSynchronousTask()`
- `TakeSurfaceCapture()`, `SetWindowFreezeImmediately()`
- `RegisterSurfaceBufferCallback()`, `DropFrameByPid()`

**证据来源**: `rosen/modules/render_service_base/include/platform/ohos/`

---

## Drawable API

### 基类

```cpp
class RSDrawable {
    // UI 线程
    virtual bool OnUpdate(const RSRenderNode& node);
    
    // 渲染线程
    virtual void OnDraw(Drawing::Canvas* canvas, 
                       const Drawing::Rect* rect) const;
    
    // 同步
    virtual void OnSync();
    
    // 清理
    virtual void OnPurge();
};
```

### Drawable 分类

| 分类 | 示例 | 说明 |
|------|------|------|
| **Property** | RSBackgroundColorDrawable | 属性渲染 |
| **Filter** | RSBackgroundFilterDrawable | 滤镜 |
| **Container** | RSChildrenDrawable | 子节点 |
| **Shadow** | RSShadowDrawable | 阴影 |
| **Mask** | RSMaskDrawable | 遮罩 |

**证据来源**: `rosen/modules/render_service_base/include/drawable/`

---

## 2D 绘图 Inner API

### Canvas 层次

```
CoreCanvas
└── Canvas
    ├── OverDrawCanvas    - Overdraw 可视化
    ├── NoDrawCanvas      - 非绘制
    │   └── RecordingCanvas - 命令录制
    └── (平台适配器)
```

### 核心方法

| 类 | 关键方法 |
|------|----------|
| **Canvas** | `DrawRect`, `DrawCircle`, `DrawPath`, `DrawImage` |
| **Paint** | `SetColor`, `SetStyle`, `SetStrokeWidth` |
| **Brush** | `SetColor`, `SetAlpha` |
| **Pen** | `SetColor`, `SetWidth`, `SetCap` |
| **Path** | `AddRect`, `AddCircle`, `Concat` |
| **Matrix** | `SetValues`, `PreTranslate`, `PostScale` |
| **Bitmap** | `Build`, `ExtractPixels` |
| **Font** | `MeasureText`, `GetBounds` |

**证据来源**: `rosen/modules/2d_graphics/include/draw/`

---

## VSync API

### VSync 接收器

```cpp
class VsyncReceiver {
    void RequestNextVSync(const std::string& fromWhom);
    void SetVSyncRate(int32_t rate, const std::string& name);
    void RemoveVSyncRate(const std::string& name);
};
```

**证据来源**: `rosen/modules/composer/vsync/`

---

## 平台适配层

### Platform 目录结构

```
rosen/modules/platform/
├── adapter/           # 平台适配器
│   ├── android/
│   ├── ios/
│   ├── linux/
│   ├── mac/
│   ├── ohos/         # OpenHarmony
│   └── window/
├── common/           # 公共代码
└── window/           # 窗口抽象
```

**证据来源**: `rosen/modules/platform/`

---

## 相关文档

- [架构说明](03_Architecture.md) - 模块交互图
- [N-API 接口](04_N-API.md) - 对外 JS API
- [构建文档](06_Build.md) - Inner API 构建配置
