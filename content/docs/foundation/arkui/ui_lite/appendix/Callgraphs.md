# 调用链分析

## 文档信息

- **文档用途**: 分析关键功能的调用链，帮助理解代码流程
- **适用范围**: 开发者、调试人员
- **相关文档**: [架构说明](../02_Architecture.md), [内部 API](../05_Internal_API.md)

## 启动调用链

### 系统初始化

```
main()
└── GraphicStartUp::Init()
    ├── 初始化内存分配器
    ├── 初始化字体引擎
    │   └── UIFont::GetInstance()
    │       ├── 注册矢量字体 (ENABLE_VECTOR_FONT)
    │       └── 注册位图字体
    ├── 初始化 ICU (ENABLE_ICU)
    │   └── ui_line_break.cpp
    └── 初始化图形引擎
        └── BaseGfxEngine::InitGfxEngine()
            └── SoftEngine 实例化
```

**证据**: `interfaces/innerkits/common/graphic_startup.h:24`

---

## 渲染调用链

### 视图渲染流程

```
RootView::Invalidate()
└── RootView::AddInvalidateRect()
    └── 添加脏矩形到列表

TaskManager::TaskHandler() [周期调用]
└── RenderManager::Render()
    ├── 合并脏矩形
    ├── 遍历视图树
    │   └── UIView::OnDraw()
    │       ├── DrawRect()      - 绘制背景
    │       ├── DrawImage()     - 绘制图像
    │       └── DrawLabel()     - 绘制文字
    └── BaseGfxEngine::Flush()
        └── 提交到显示设备
```

**证据**: 
- `frameworks/components/root_view.h:77`
- `interfaces/innerkits/common/task_manager.h:31`

---

## 事件处理调用链

### 触摸事件流程

```
InputDeviceManager::ProcessInputEvents()
└── PointerInputDevice::Read()
    └── 读取硬件输入
    
RootView::OnPointEvent()
└── RootView::GetTargetView()
    └── 命中测试 (Hit Test)
        └── UIView::GetRect()
        
UIView::OnClickEvent()
└── OnClickListener::OnClick()
    └── 应用回调处理
```

**证据**: `interfaces/innerkits/common/input_device_manager.h`

### 按键事件流程

```
InputDeviceManager::ProcessInputEvents()
└── KeyInputDevice::Read()

RootView::OnKeyEvent()
└── OnKeyActListener::OnKeyAct()
    └── 应用处理按键
    
或
    
FocusManager::MoveFocusUp/Down/Left/Right()
└── 查找下一个可聚焦视图
    └── UIView::RequestFocus()
```

**证据**: `interfaces/kits/components/root_view.h:150`

---

## 图像加载调用链

### 图像显示流程

```
UIImageView::SetSrc(const char* path)
└── Image::ParseHeader(path)
    └── FileImgDecoder::Open()
        ├── 打开文件
        └── 解析图像头

UIImageView::OnDraw()
└── DrawImage::Draw()
    ├── FileImgDecoder::ReadLine() [逐行读取]
    ├── 图像解码 (JPEG/PNG)
    └── BaseGfxEngine::Blit() [绘制到缓冲区]
```

**证据**: `frameworks/imgdecode/file_img_decoder.h:34`

---

## 字体渲染调用链

### 文字显示流程

```
UILabel::SetText(const char* text)
└── 保存文本内容

UILabel::OnDraw()
└── DrawLabel::Draw()
    ├── UIFont::GetInstance()
    │   └── 获取当前字体
    ├── UIFontVector::GetGlyph() [矢量字体]
    │   ├── FreeType 加载字形
    │   └── 栅格化
    ├── GlyphsCache::Get() [缓存查找]
    └── BaseGfxEngine::DrawLetter()
        └── 绘制到缓冲区
```

**证据**: `interfaces/kits/font/ui_font.h`

---

## 动画调用链

### 动画执行流程

```
Animator::Start()
└── TaskManager::GetInstance()->Add(this)
    └── 添加到任务队列

TaskManager::TaskHandler() [每帧调用]
└── Animator::Callback()
    ├── 计算插值值
    │   └── EasingEquation::Calculate()
    ├── 更新视图属性
    │   └── UIView::SetX/Y/Width/Height()
    └── UIView::Invalidate() [触发重绘]

Animator::Stop()
└── TaskManager::GetInstance()->Remove(this)
```

**证据**: `interfaces/kits/animator/animator.h`

---

## 窗口管理调用链

### 窗口创建流程

```
Window::CreateWindow(config)
└── new WindowImpl()
    ├── IWindowsManager::GetInstance()
    │   └── 获取 WMS 接口
    ├── IWindowsManager::CreateWindow()
    │   └── IPC 调用 WMS 服务
    └── Surface::CreateSurface()
        └── 申请图形缓冲区

Window::BindRootView(rootView)
└── 建立窗口与视图关联

Window::Show()
└── IWindow::Show()
    └── 通知 WMS 显示窗口
```

**证据**: `interfaces/kits/window/window.h:132`

---

## 布局调用链

### 布局计算流程

```
RootView::Measure()
└── UIViewGroup::LayoutChildren()
    └── 根据布局类型计算
        ├── FlexLayout::Measure()
        │   ├── 计算主轴位置
        │   └── 计算交叉轴位置
        ├── GridLayout::Measure()
        │   ├── 计算行高
        │   └── 计算列宽
        └── ListLayout::Measure()
            └── 计算列表项位置

UIView::SetPosition() / Resize()
└── 更新视图几何属性
    └── Invalidate() [标记需要重绘]
```

**证据**: `interfaces/kits/layout/flex_layout.h`

---

## 主题应用调用链

### 主题切换流程

```
ThemeManager::SetCurrentTheme(theme)
└── 遍历所有视图
    └── UIView::SetStyle()
        ├── 更新背景色
        ├── 更新边框
        └── Invalidate() [触发重绘]

UIView::OnDraw()
└── GetStyle()
    └── 获取当前主题样式
        └── 应用到绘制
```

**证据**: `interfaces/kits/themes/theme_manager.h`

---

## 调试功能调用链

### 事件注入流程

```
EventInjector::SetClickEvent(point)
└── PointEventInjector::InjectEvent()
    └── 创建模拟事件
        └── RootView::OnPointEvent()
            └── 正常事件处理流程

EventInjector::SetKeyEvent(keyId, state)
└── KeyEventInjector::InjectEvent()
    └── 创建模拟按键
        └── RootView::OnKeyEvent()
```

**证据**: `interfaces/kits/dfx/event_injector.h:64`

**注意**: 仅在 `ENABLE_DEBUG` 时可用

---

## 关键调用链速查

| 功能 | 入口 | 核心调用 | 出口 |
|------|------|----------|------|
| 系统启动 | `GraphicStartUp::Init()` | 字体/引擎初始化 | 完成标志 |
| 视图渲染 | `Invalidate()` | `OnDraw()` → `GfxEngine` | 屏幕显示 |
| 事件处理 | `InputDevice` | `GetTargetView()` → 回调 | 应用处理 |
| 图像加载 | `SetSrc(path)` | `FileImgDecoder` → `Blit()` | 显示图像 |
| 字体渲染 | `SetText()` | `GetGlyph()` → `DrawLetter()` | 显示文字 |
| 动画执行 | `Start()` | `TaskHandler()` → 插值 | `Stop()` |
| 窗口管理 | `CreateWindow()` | `WMS IPC` → `Surface` | `Show()` |
| 布局计算 | `Measure()` | `LayoutChildren()` | 位置更新 |

## 相关文档

- [架构说明](../02_Architecture.md) - 系统架构
- [内部 API](../05_Internal_API.md) - 模块接口
- [对外 API](../04_Public_API.md) - 应用接口
