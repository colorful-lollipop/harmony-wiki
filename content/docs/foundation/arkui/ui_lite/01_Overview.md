# ui_lite 项目概览

## 文档信息

- **文档用途**: 提供 ui_lite 模块的整体定位、边界和核心能力说明
- **适用范围**: OpenHarmony 3.1+，mini/small/standard 系统类型
- **相关文档**: [目录结构](03_Directory_Structure.md), [架构说明](02_Architecture.md)

## 项目定位

`ui_lite` 是 OpenHarmony 轻量级图形 UI 框架，实现系统级图形引擎，为应用开发提供 UIKit APIs。

### 核心职责

1. **UI 组件渲染** - 提供丰富的 UI 组件（按钮、文本、图像、列表等）
2. **动画支持** - 支持属性动画和插值动画
3. **布局管理** - 支持 Flex、Grid、List 等多种布局
4. **事件处理** - 处理触摸、按键、拖拽等输入事件
5. **字体渲染** - 支持矢量字体和位图字体
6. **图像处理** - 支持多种格式图像解码和显示
7. **窗口管理** - 与 WMS 交互管理窗口生命周期

### 系统定位

```
┌─────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                  │
├─────────────────────────────────────────────────────────────┤
│                     JS/ArkTS 应用框架 (可选)                   │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────┐  │
│  │              ui_lite (本模块) - C++ API                │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │  │
│  │  │ 组件系统  │ │ 动画系统  │ │ 布局系统  │ │ 事件系统  │  │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│              Window Manager Service (WMS)                    │
├─────────────────────────────────────────────────────────────┤
│              Surface / Graphic Utils / HAL                   │
└─────────────────────────────────────────────────────────────┘
```

## 项目边界

### 包含的功能

- ✅ UI 组件库（40+ 组件）
- ✅ 动画框架（属性动画、插值器）
- ✅ 布局系统（Flex、Grid、List、相对布局）
- ✅ 事件系统（触摸、按键、拖拽、旋转）
- ✅ 字体系统（矢量字体、多字体、ICU 断行）
- ✅ 图像解码（JPEG、PNG、二维码）
- ✅ 绘制引擎（软件渲染、硬件加速接口）
- ✅ 主题系统（样式管理）
- ✅ 窗口适配（与 WMS 集成）

### 不包含的功能

- ❌ JavaScript/ArkTS 绑定（纯 C++ 框架）
- ❌ IPC/SA 通信（依赖 WMS 进行跨进程通信）
- ❌ 权限管理（由上层框架处理）
- ❌ 3D 渲染（仅 2D 图形）
- ❌ Web 渲染（非 Web 引擎）

## 核心能力

### 1. 组件系统

**基础组件**（单一功能）:
- `UILabel` - 文本显示
- `UIButton` - 按钮
- `UIImageView` - 图像显示
- `UICheckBox` - 复选框
- `UIRadioButton` - 单选按钮

**容器组件**（组合子组件）:
- `UIScrollView` - 滚动容器
- `UIList` - 列表容器
- `UISwipeView` - 滑动视图
- `UIDialog` - 对话框

**高级组件**:
- `UICanvas` - 画布（自定义绘制）
- `UIChart` - 图表
- `UIVideo` - 视频播放（可选）

### 2. 动画系统

```cpp
// 证据: interfaces/kits/animator/animator.h
class Animator : public HeapBase {
    void Start();
    void Stop();
    void SetDuration(uint16_t duration);
    void SetInterpolator(InterpolatorType type);
};
```

- 支持 30+ 种插值器（线性、加速、减速、弹跳等）
- 支持属性动画（位置、大小、透明度、旋转）
- 支持 2D/3D 变换

### 3. 布局系统

- **FlexLayout** - 弹性布局 ([interfaces/kits/layout/flex_layout.h](interfaces/kits/layout/flex_layout.h))
- **GridLayout** - 网格布局 ([interfaces/kits/layout/grid_layout.h](interfaces/kits/layout/grid_layout.h))
- **ListLayout** - 列表布局 ([interfaces/kits/layout/list_layout.h](interfaces/kits/layout/list_layout.h))
- **相对布局** - 支持相对父视图或兄弟视图定位

### 4. 事件系统

**事件类型**:
- `ClickEvent` - 点击
- `PressEvent` / `ReleaseEvent` - 按下/释放
- `DragEvent` - 拖拽
- `LongPressEvent` - 长按
- `KeyEvent` - 按键
- `RotateEvent` - 旋转（可选）

**事件传递**:
```
输入设备 → InputDeviceManager → RootView → View 树分发
```

### 5. 字体系统

- 矢量字体（FreeType）
- 位图字体（可选）
- ICU 断行支持
- 多字体混排
- 字体缓存管理

### 6. 绘制引擎

**软件渲染** ([interfaces/innerkits/engines/gfx/soft_engine.h](interfaces/innerkits/engines/gfx/soft_engine.h)):
- 基础图形（点、线、矩形、圆、弧）
- 图像绘制（缩放、旋转、混合）
- 文字渲染（抗锯齿）
- 混合模式（SRC_OVER、MULTIPLY、ADDITIVE 等）

**硬件加速接口**:
- 支持 hi3516 硬件引擎
- 可扩展其他硬件平台

## 运行环境

### 支持的系统类型

- **mini** - 轻量级设备（MCU 级别）
- **small** - 小型设备（轻量 Linux）
- **standard** - 标准设备（标准 Linux）

### 资源占用

- **ROM**: ~900KB
- **RAM**: ~90KB（运行时）

### 内核适配

- **LiteOS-M** - 静态库（libui.a），裁剪部分功能
- **LiteOS-A** - 动态库（libui.so），完整功能
- **Linux** - 动态库（libui.so），完整功能

### 依赖组件

```
ui_lite
├── graphic_utils_lite    # 图形工具库
├── surface_lite          # 图形 Surface
├── window_manager_lite   # 窗口管理
├── media_lite (optional) # 媒体播放（视频组件）
├── libjpeg-turbo         # JPEG 解码
├── libpng                # PNG 解码
├── freetype              # 字体渲染
├── icu                   # 国际化
├── cJSON                 # JSON 解析
├── qrcodegen             # 二维码生成
├── harfbuzz              # 字体整形
└── bounds_checking_function # 安全函数
```

## 关键概念

### 1. View（视图）

UI 的基本单元，所有组件继承自 `UIView` ([interfaces/kits/components/ui_view.h](interfaces/kits/components/ui_view.h))。

```cpp
// 证据: interfaces/kits/components/ui_view.h:157
class UIView : public HeapBase {
    // 位置、大小、样式、事件监听等属性
};
```

### 2. RootView（根视图）

视图树的根节点，管理所有子视图的渲染和事件分发 ([interfaces/kits/components/root_view.h](interfaces/kits/components/root_view.h))。

```cpp
// 证据: interfaces/kits/components/root_view.h:77
static RootView* GetInstance();  // 单例模式
```

### 3. Window（窗口）

与 WMS 交互的窗口抽象，每个窗口绑定一个 RootView ([interfaces/kits/window/window.h](interfaces/kits/window/window.h))。

```cpp
// 证据: interfaces/kits/window/window.h:132
static Window* CreateWindow(const WindowConfig& config);
```

### 4. Task（任务）

异步任务抽象，用于动画和延迟操作 ([interfaces/kits/common/task.h](interfaces/kits/common/task.h))。

### 5. Style（样式）

视图外观属性集合（背景色、边框、圆角等）。

### 6. Transform（变换）

2D/3D 变换矩阵，支持旋转、缩放、平移、错切。

## 使用模式

### 基础使用流程

```cpp
// 1. 初始化图形系统
GraphicStartUp::Init();
GraphicStartUp::InitFontEngine(...);

// 2. 获取 RootView
RootView* rootView = RootView::GetInstance();

// 3. 创建组件
UILabel* label = new UILabel();
label->SetText("Hello");
label->SetPosition(10, 10);
label->Resize(100, 50);

// 4. 添加到视图树
rootView->Add(label);

// 5. 设置事件监听
label->SetOnClickListener(new MyClickListener());

// 6. 刷新显示
rootView->Invalidate();
```

### 窗口模式使用

```cpp
// 1. 创建窗口
WindowConfig config;
config.rect = {0, 0, 480, 800};
Window* window = Window::CreateWindow(config);

// 2. 创建 RootView
RootView* rootView = RootView::GetWindowRootView();
window->BindRootView(rootView);

// 3. 添加组件...

// 4. 显示窗口
window->Show();
```

## 相关链接

- [目录结构](03_Directory_Structure.md) - 代码组织
- [架构说明](02_Architecture.md) - 系统架构
- [对外 API](04_Public_API.md) - API 参考
- [GN 构建](06_GN_Targets.md) - 构建配置
