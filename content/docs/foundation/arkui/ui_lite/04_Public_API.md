# ui_lite 对外 C++ API

## 文档信息

- **文档用途**: 提供 ui_lite 对外 C++ API 的完整清单和说明
- **适用范围**: 应用开发者、SDK 使用者
- **相关文档**: [项目概览](01_Overview.md), [内部 API](05_Internal_API.md)

## API 概览

ui_lite 是纯 C++ UI 框架，通过 `interfaces/kits/` 目录下的头文件暴露 API。应用开发者包含这些头文件即可使用 UI 功能。

**重要说明**: 本模块**无 N-API 绑定**，应用需使用 C++ 开发或依赖上层封装。

## API 分类

### 1. 组件类 API (components/)

#### 1.1 视图基类

**UIView** - 所有组件的基类
- **文件**: [interfaces/kits/components/ui_view.h](interfaces/kits/components/ui_view.h)
- **关键方法**:

```cpp
// 位置与大小
void SetPosition(int16_t x, int16_t y);           // 设置位置
void SetX(int16_t x);                              // 设置 X 坐标
void SetY(int16_t y);                              // 设置 Y 坐标
void Resize(int16_t width, int16_t height);        // 设置大小
void SetWidth(int16_t width);                      // 设置宽度
void SetHeight(int16_t height);                    // 设置高度
int16_t GetX() const;                              // 获取 X 坐标
int16_t GetY() const;                              // 获取 Y 坐标
virtual int16_t GetWidth();                        // 获取宽度
virtual int16_t GetHeight();                       // 获取高度

// 百分比布局 (v5.0+)
void SetXPercent(float xPercent);                  // 设置 X 百分比
void SetYPercent(float yPercent);                  // 设置 Y 百分比
void SetWidthPercent(float widthPercent);          // 设置宽度百分比
void SetHeightPercent(float heightPercent);        // 设置高度百分比
void SetPositionPercent(float xPercent, float yPercent);
void ResizePercent(float widthPercent, float heightPercent);

// 可见性与状态
void SetVisible(bool visible);                     // 设置可见性
bool IsVisible() const;                            // 是否可见
void SetTouchable(bool touch);                     // 设置可触摸
bool IsTouchable() const;                          // 是否可触摸
void SetDraggable(bool draggable);                 // 设置可拖拽
bool IsDraggable() const;                          // 是否可拖拽

// 样式
void SetStyle(Style& style);                       // 设置样式
void SetStyle(uint8_t key, int64_t value);         // 设置样式属性
int64_t GetStyle(uint8_t key) const;               // 获取样式属性
void SetOpaScale(uint8_t opaScale);                // 设置透明度

// 事件监听
void SetOnClickListener(OnClickListener* listener);
void SetOnDragListener(OnDragListener* listener);
void SetOnLongPressListener(OnLongPressListener* listener);
void SetOnTouchListener(OnTouchListener* listener);

// 变换（2D/3D）
void SetTransformMap(const TransformMap& transMap);
void Rotate(int16_t angle, const Vector2<float>& pivot);
void Rotate(int16_t angle, const Vector3<float>& pivotStart, const Vector3<float>& pivotEnd);
void Scale(const Vector2<float>& scale, const Vector2<float>& pivot);
void Scale(const Vector3<float>& scale, const Vector3<float>& pivot);
void Translate(const Vector2<int16_t>& trans);

// 重绘
void Invalidate();                                 // 刷新整个视图
void InvalidateRect(const Rect& invalidatedArea);  // 刷新指定区域

// ID 管理
void SetViewId(const char* id);                    // 设置视图 ID
const char* GetViewId() const;                     // 获取视图 ID
UIView* GetChildById(const char* id) const;        // 通过 ID 获取子视图

// 相对布局
void LayoutCenterOfParent(int16_t xOffset = 0, int16_t yOffset = 0);
void LayoutLeftOfParent(int16_t offset = 0);
void LayoutRightOfParent(int16_t offset = 0);
void LayoutTopOfParent(int16_t offset = 0);
void LayoutBottomOfParent(int16_t offset = 0);
void AlignLeftToSibling(const char* id, int16_t offset = 0);
void AlignRightToSibling(const char* id, int16_t offset = 0);
void AlignTopToSibling(const char* id, int16_t offset = 0);
void AlignBottomToSibling(const char* id, int16_t offset = 0);
```

**事件监听接口**:
```cpp
class OnClickListener : public HeapBase {
    virtual bool OnClick(UIView& view, const ClickEvent& event) { return false; }
};

class OnDragListener : public HeapBase {
    virtual bool OnDragStart(UIView& view, const DragEvent& event) { return false; }
    virtual bool OnDrag(UIView& view, const DragEvent& event) { return false; }
    virtual bool OnDragEnd(UIView& view, const DragEvent& event) { return false; }
};

class OnLongPressListener : public HeapBase {
    virtual bool OnLongPress(UIView& view, const LongPressEvent& event) { return false; }
};

class OnTouchListener : public HeapBase {
    virtual bool OnPress(UIView& view, const PressEvent& event) { return false; }
    virtual bool OnRelease(UIView& view, const ReleaseEvent& event) { return false; }
    virtual bool OnCancel(UIView& view, const CancelEvent& event) { return false; }
};
```

#### 1.2 容器基类

**UIViewGroup** - 可容纳子视图的容器基类
- **文件**: [interfaces/kits/components/ui_view_group.h](interfaces/kits/components/ui_view_group.h)

```cpp
void Add(UIView* view);                            // 添加子视图
void Remove(UIView* view);                         // 移除子视图
void RemoveAll();                                  // 移除所有子视图
void Insert(UIView* prevView, UIView* view);       // 插入子视图
UIView* GetChildrenHead() const;                   // 获取第一个子视图
bool IsEmpty() const;                              // 是否为空
```

#### 1.3 根视图

**RootView** - 视图树根节点，单例模式
- **文件**: [interfaces/kits/components/root_view.h](interfaces/kits/components/root_view.h)

```cpp
static RootView* GetInstance();                    // 获取单例

#if ENABLE_WINDOW
static RootView* GetWindowRootView();              // 创建窗口根视图
static bool DestroyWindowRootView(RootView* rootView);
#endif

void OnKeyEvent(const KeyEvent& event);            // 按键事件处理
void SetOnKeyActListener(OnKeyActListener* listener);
void OnVirtualDeviceEvent(const VirtualDeviceEvent& event);
void SetOnVirtualDeviceEventListener(OnVirtualDeviceEventListener* listener);

void Measure();                                    // 测量所有子视图
void UpdateBufferInfo(BufferInfo* fbBufferInfo);   // 更新缓冲区信息
```

#### 1.4 文本标签

**UILabel** - 文本显示组件
- **文件**: [interfaces/kits/components/ui_label.h](interfaces/kits/components/ui_label.h)

```cpp
void SetText(const char* text);                    // 设置文本
const char* GetText() const;                       // 获取文本
void SetLineBreakMode(uint8_t mode);               // 设置换行模式
void SetAlign(uint8_t align);                      // 设置对齐方式
void SetFontId(uint8_t fontId);                    // 设置字体 ID
void SetTextColor(ColorType color);                // 设置文本颜色
void SetDynamicText(const char* text, uint16_t maxLen);  // 设置动态文本
```

**换行模式**:
- `LINE_BREAK_ADAPT` - 自适应
- `LINE_BREAK_STRETCH` - 拉伸
- `LINE_BREAK_WRAP` - 自动换行
- `LINE_BREAK_ELLIPSIS` - 省略号
- `LINE_BREAK_MARQUEE` - 跑马灯
- `LINE_BREAK_CLIP` - 裁剪

#### 1.5 按钮

**UIButton** - 按钮组件
- **文件**: [interfaces/kits/components/ui_button.h](interfaces/kits/components/ui_button.h)

```cpp
void SetImageSrc(const char* src);                 // 设置图片
void SetImageSrc(const ImageInfo* src);
void SetStateImageSrc(const char* src, uint8_t state);
void SetStateImages(const char* defaultImg, const char* triggeredImg, 
                    const char* inactiveImg);
```

**按钮状态**:
- `STATE_NORMAL` - 正常
- `STATE_PRESSED` - 按下
- `STATE_INACTIVE` - 禁用

#### 1.6 图像视图

**UIImageView** - 图像显示组件
- **文件**: [interfaces/kits/components/ui_image_view.h](interfaces/kits/components/ui_image_view.h)

```cpp
void SetSrc(const char* src);                      // 设置图片源
void SetSrc(const ImageInfo* src);
void SetAutoEnable(bool enable);                   // 自动调整大小
void SetBlurLevel(BlurLevel level);                // 设置模糊级别
void SetTransformAlgorithm(TransformAlgorithm algo); // 设置变换算法
void SetResizeMode(ResizeMode mode);               // 设置缩放模式
```

**缩放模式**:
- `RESIZE_MODE_COVER` - 覆盖
- `RESIZE_MODE_CONTAIN` - 包含
- `RESIZE_MODE_FILL` - 填充
- `RESIZE_MODE_CENTER` - 居中
- `RESIZE_MODE_SCALE_DOWN` - 缩小

#### 1.7 滚动视图

**UIScrollView** - 可滚动容器
- **文件**: [interfaces/kits/components/ui_scroll_view.h](interfaces/kits/components/ui_scroll_view.h)

```cpp
void SetScrollDirection(uint8_t direction);        // 设置滚动方向
void SetScrollMode(uint8_t mode);                  // 设置滚动模式
void SetScrollPos(int16_t x, int16_t y);           // 设置滚动位置
void ScrollBy(int16_t deltaX, int16_t deltaY);     // 相对滚动
void SetScrollbarWidth(uint8_t width);             // 设置滚动条宽度
void SetScrollbarColor(ColorType color);           // 设置滚动条颜色
```

**滚动方向**:
- `HORIZONTAL` - 水平
- `VERTICAL` - 垂直
- `HORIZONTAL_AND_VERTICAL` - 双向

#### 1.8 列表

**UIList** - 列表组件
- **文件**: [interfaces/kits/components/ui_list.h](interfaces/kits/components/ui_list.h)

```cpp
void SetAdapter(AbstractAdapter* adapter);         // 设置适配器
void RefreshList();                                // 刷新列表
void SetSelectIndex(uint16_t index);               // 设置选中项
uint16_t GetSelectIndex() const;                   // 获取选中项
void SetLoopState(bool state);                     // 设置循环状态
```

**适配器接口**:
```cpp
class AbstractAdapter : public HeapBase {
public:
    virtual int16_t GetCount() = 0;                // 获取项目数
    virtual UIView* GetView(int16_t index, UIView* convertView) = 0;  // 获取视图
};
```

#### 1.9 画布

**UICanvas** - 自定义绘制画布
- **文件**: [interfaces/kits/components/ui_canvas.h](interfaces/kits/components/ui_canvas.h)

```cpp
void Clear();                                      // 清空画布
void DrawLine(const Point& start, const Point& end, 
              const Paint& paint);                 // 绘制线条
void DrawRect(const Rect& rect, const Paint& paint); // 绘制矩形
void DrawCircle(const Point& center, int16_t radius,
                const Paint& paint);               // 绘制圆形
void DrawArc(const Rect& oval, int16_t startAngle,
             int16_t sweepAngle, const Paint& paint); // 绘制圆弧
void DrawPath(const Path& path, const Paint& paint); // 绘制路径
void DrawText(const char* text, const Point& pos,
              const Paint& paint);                 // 绘制文字
```

### 2. 动画 API (animator/)

**Animator** - 动画控制器
- **文件**: [interfaces/kits/animator/animator.h](interfaces/kits/animator/animator.h)

```cpp
void Start();                                      // 开始动画
void Stop();                                       // 停止动画
void Pause();                                      // 暂停动画
void Resume();                                     // 恢复动画
bool IsRunning() const;                            // 是否运行中

void SetDuration(uint16_t duration);               // 设置持续时间(ms)
void SetRepeatCount(uint16_t count);               // 设置重复次数
void SetRepeat(bool repeat);                       // 设置是否循环
void SetInterpolator(InterpolatorType type);       // 设置插值器
void SetAnimatorCallback(AnimatorCallback* callback); // 设置回调
```

**插值器类型**:
- `INTERPOLATOR_LINEAR` - 线性
- `INTERPOLATOR_EASE_IN_QUAD` - 加速
- `INTERPOLATOR_EASE_OUT_QUAD` - 减速
- `INTERPOLATOR_EASE_IN_OUT_QUAD` - 加减速
- `INTERPOLATOR_BOUNCE` - 弹跳
- ... (30+ 种)

**动画回调**:
```cpp
class AnimatorCallback : public HeapBase {
public:
    virtual void Callback(UIView* view) = 0;
};
```

### 3. 事件 API (events/)

#### 3.1 事件基类

**Event** - 所有事件的基类
- **文件**: [interfaces/kits/events/event.h](interfaces/kits/events/event.h)

```cpp
const Point& GetCurrentPos() const;                // 获取当前位置
const Point& GetLastPos() const;                   // 获取上一个位置
TimeType GetTimeStamp() const;                     // 获取时间戳
```

#### 3.2 点击事件

**ClickEvent** - 点击事件
- **文件**: [interfaces/kits/events/click_event.h](interfaces/kits/events/click_event.h)

```cpp
const Point& GetViewPos() const;                   // 获取视图内坐标
```

#### 3.3 拖拽事件

**DragEvent** - 拖拽事件
- **文件**: [interfaces/kits/events/drag_event.h](interfaces/kits/events/drag_event.h)

```cpp
int16_t GetDeltaX() const;                         // 获取 X 方向增量
int16_t GetDeltaY() const;                         // 获取 Y 方向增量
int16_t GetStartX() const;                         // 获取起始 X
int16_t GetStartY() const;                         // 获取起始 Y
```

#### 3.4 按键事件

**KeyEvent** - 按键事件
- **文件**: [interfaces/kits/events/key_event.h](interfaces/kits/events/key_event.h)

```cpp
uint16_t GetKeyId() const;                         // 获取按键 ID
uint16_t GetState() const;                         // 获取按键状态
```

### 4. 布局 API (layout/)

#### 4.1 Flex 布局

**FlexLayout** - 弹性布局
- **文件**: [interfaces/kits/layout/flex_layout.h](interfaces/kits/layout/flex_layout.h)

```cpp
void SetFlexDirection(uint8_t direction);          // 设置方向
void SetJustifyContent(uint8_t justify);           // 设置主轴对齐
void SetAlignItems(uint8_t align);                 // 设置交叉轴对齐
void SetWrap(uint8_t wrap);                        // 设置换行
```

**方向**:
- `FLEX_DIRECTION_ROW` - 水平
- `FLEX_DIRECTION_COLUMN` - 垂直

**对齐**:
- `FLEX_ALIGN_START` - 起始
- `FLEX_ALIGN_END` - 结束
- `FLEX_ALIGN_CENTER` - 居中
- `FLEX_ALIGN_SPACE_BETWEEN` - 两端对齐
- `FLEX_ALIGN_SPACE_AROUND` - 均匀分布

#### 4.2 Grid 布局

**GridLayout** - 网格布局
- **文件**: [interfaces/kits/layout/grid_layout.h](interfaces/kits/layout/grid_layout.h)

```cpp
void SetRows(uint16_t rows);                       // 设置行数
void SetCols(uint16_t cols);                       // 设置列数
void SetGap(uint16_t rowGap, uint16_t colGap);     // 设置间距
```

### 5. 字体 API (font/)

**UIFont** - 字体管理
- **文件**: [interfaces/kits/font/ui_font.h](interfaces/kits/font/ui_font.h)

```cpp
static UIFont* GetInstance();                      // 获取单例

int8_t SetCurrentFontId(uint8_t fontId);           // 设置当前字体
uint8_t GetCurrentFontId() const;                  // 获取当前字体
int8_t RegisterFontInfo(const char* fontInfo);     // 注册字体
int8_t SetFontPath(const char* path);              // 设置字体路径
uint16_t GetLineMaxHeight(uint8_t fontId, uint8_t fontSize); // 获取行高
uint16_t GetWidth(uint8_t fontId, uint8_t fontSize,
                  const char* text, uint16_t textLen); // 获取文本宽度
```

### 6. 主题 API (themes/)

**Theme** - 主题
- **文件**: [interfaces/kits/themes/theme.h](interfaces/kits/themes/theme.h)

```cpp
void SetThemeId(uint8_t id);                       // 设置主题 ID
uint8_t GetThemeId() const;                        // 获取主题 ID

// 各组件样式
Style* GetButtonStyle();
Style* GetButtonPressedStyle();
Style* GetLabelStyle();
Style* GetEditTextStyle();
Style* GetPickerStyle();
Style* GetProgressBackgroundStyle();
Style* GetProgressForegroundStyle();
Style* GetSliderKnobStyle();
Style* GetSliderBackgroundStyle();
```

**ThemeManager** - 主题管理器
- **文件**: [interfaces/kits/themes/theme_manager.h](interfaces/kits/themes/theme_manager.h)

```cpp
static ThemeManager* GetInstance();                // 获取单例
void SetCurrentTheme(Theme* theme);                // 设置当前主题
Theme* GetCurrentTheme() const;                    // 获取当前主题
```

### 7. 窗口 API (window/)

**Window** - 窗口管理
- **文件**: [interfaces/kits/window/window.h](interfaces/kits/window/window.h)

```cpp
// 静态方法
static Window* CreateWindow(const WindowConfig& config);  // 创建窗口
static void DestroyWindow(Window* window);           // 销毁窗口

// 实例方法
virtual void BindRootView(RootView* rootView) = 0;   // 绑定根视图
virtual void UnbindRootView() = 0;                   // 解绑根视图
virtual RootView* GetRootView() = 0;                 // 获取根视图

virtual void Show() = 0;                             // 显示窗口
virtual void Hide() = 0;                             // 隐藏窗口
virtual void MoveTo(int16_t x, int16_t y) = 0;       // 移动窗口
virtual void Resize(int16_t width, int16_t height) = 0; // 调整大小
virtual void RaiseToTop() = 0;                       // 置顶
virtual void LowerToBottom() = 0;                    // 置底
virtual int32_t GetWindowId() = 0;                   // 获取窗口 ID
```

**WindowConfig** - 窗口配置:
```cpp
struct WindowConfig {
    Rect rect;                    // 位置和大小
    uint8_t opacity;              // 透明度 [0, 255]
    WindowPixelFormat pixelFormat; // 像素格式
    CompositeMode compositeMode;   // 合成模式
    bool isModal;                 // 是否模态
};
```

### 8. 通用 API (common/)

#### 8.1 任务

**Task** - 定时任务基类
- **文件**: [interfaces/kits/common/task.h](interfaces/kits/common/task.h)

```cpp
void SetPeriod(uint32_t period);                   // 设置周期(ms)
uint32_t GetPeriod() const;                        // 获取周期
void SetRepeatCount(uint16_t count);               // 设置重复次数
void SetCallback(TaskCallback* callback);          // 设置回调
```

#### 8.2 图像

**Image** - 图像处理
- **文件**: [interfaces/kits/common/image.h](interfaces/kits/common/image.h)

```cpp
bool ParseHeader(const char* src, ImageHeader& header); // 解析头
bool Open(const char* src);                          // 打开图像
void Close();                                        // 关闭图像
const ImageInfo* GetImageInfo();                     // 获取图像信息
```

### 9. 调试 API (dfx/)

**EventInjector** - 事件注入（仅 DEBUG 模式）
- **文件**: [interfaces/kits/dfx/event_injector.h](interfaces/kits/dfx/event_injector.h)

```cpp
static EventInjector* GetInstance();                 // 获取单例
bool RegisterEventInjector(EventDataType type);      // 注册注入器
bool SetClickEvent(const Point& clickPoint);         // 模拟点击
bool SetLongPressEvent(const Point& longPressPoint); // 模拟长按
bool SetDragEvent(const Point& startPoint, const Point& endPoint,
                  uint32_t dragTime);                // 模拟拖拽
bool SetKeyEvent(uint16_t keyId, uint16_t state);    // 模拟按键
```

## API 使用示例

### 基础使用

```cpp
#include "components/root_view.h"
#include "components/ui_label.h"
#include "components/ui_button.h"
#include "common/graphic_startup.h"

using namespace OHOS;

// 初始化
GraphicStartUp::Init();

// 获取根视图
RootView* rootView = RootView::GetInstance();

// 创建标签
UILabel* label = new UILabel();
label->SetText("Hello UI Lite");
label->SetPosition(50, 50);
label->Resize(200, 50);
label->SetStyle(STYLE_TEXT_COLOR, Color::White().full);
rootView->Add(label);

// 创建按钮
UIButton* button = new UIButton();
button->SetPosition(50, 120);
button->Resize(100, 50);
button->SetImageSrc("button_bg.png");
button->SetOnClickListener(new MyClickListener());
rootView->Add(button);

// 刷新显示
rootView->Invalidate();
```

### 动画使用

```cpp
#include "animator/animator.h"

// 创建动画
Animator* animator = new Animator();
animator->SetTarget(view);
animator->SetDuration(1000);  // 1秒
animator->SetInterpolator(INTERPOLATOR_EASE_IN_OUT_QUAD);
animator->SetAnimatorCallback(new AnimatorCallback() {
    void Callback(UIView* view) override {
        // 动画更新
        view->SetX(newX);
    }
});
animator->Start();
```

### 窗口使用

```cpp
#include "window/window.h"

// 创建窗口
WindowConfig config;
config.rect = {0, 0, 480, 800};
config.opacity = OPA_OPAQUE;
config.pixelFormat = WINDOW_PIXEL_FORMAT_ARGB8888;

Window* window = Window::CreateWindow(config);
RootView* rootView = RootView::GetWindowRootView();
window->BindRootView(rootView);

// 添加组件...

window->Show();
```

## 错误处理

ui_lite 使用返回值进行错误处理：

- `0` 或 `true` - 成功
- `-1` 或 `false` - 失败
- `nullptr` - 对象创建失败

无异常机制，所有 API 均为同步调用。

## 线程安全

- **非线程安全**: 大部分 API 应在 UI 线程调用
- **线程安全**: `RootView::GetInstance()`, `TaskManager::GetInstance()`
- **锁保护**: `RootView` 使用互斥锁保护脏矩形列表

## 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [内部 API](05_Internal_API.md) - 模块间接口
- [架构说明](02_Architecture.md) - 系统架构
