# 内部 API

## 1. 模块接口概览

本章节描述项目内部的 C++ 接口定义与依赖方向。

**说明**：
- 本项目是原生 C++ 应用，不提供 JS 接口
- 所有 API 均为 C++ 接口，供框架层调用

---

## 2. ScreensaverAbility

### 2.1 类定义

**文件**：`screensaver/src/main/cpp/screensaver_ability.h`

```cpp
namespace OHOS {
class ScreensaverAbility : public Ability {
protected:
    void OnStart(const Want& want) override;
    void OnInactive() override;
    void OnActive(const Want& want) override;
    void OnBackground() override;
    void OnStop() override;
};
} // namespace OHOS
```

### 2.2 接口说明

| 接口 | 参数 | 返回 | 职责 | 稳定性 |
|------|------|------|------|--------|
| OnStart | const Want& | void | 初始化，设置主路由 | 稳定 |
| OnActive | const Want& | void | 获得焦点 | 稳定 |
| OnInactive | - | void | 失去焦点 | 稳定 |
| OnBackground | - | void | 进入后台 | 稳定 |
| OnStop | - | void | 停止 | 稳定 |

**证据**：`screensaver_ability.h:23-30`

### 2.3 关键实现

**OnStart 实现**：
```cpp
void ScreensaverAbility::OnStart(const Want& want)
{
    SetMainRoute("ScreensaverAbilitySlice");  // 设置主路由
    Ability::OnStart(want);
}
```

**证据**：`screensaver_ability.cpp:21-25`

---

## 3. ScreensaverAbilitySlice

### 3.1 类定义

**文件**：`screensaver/src/main/cpp/screensaver_ability_slice.h`

```cpp
class ScreensaverAbilitySlice : public AbilitySlice {
public:
    ScreensaverAbilitySlice() : rootView_(nullptr), 
        imageAnimator_(nullptr), exitListener_(nullptr) {}
    virtual ~ScreensaverAbilitySlice();

protected:
    void OnStart(const Want &want) override;
    void OnInactive() override;
    void OnActive(const Want &want) override;
    void OnBackground() override;
    void OnStop() override;

private:
    void SetCyclePlayView();
    RootView* rootView_;
    UIImageAnimatorView* imageAnimator_;
    EventListener* exitListener_;
};
```

### 3.2 接口说明

| 接口 | 参数 | 返回 | 职责 | 稳定性 |
|------|------|------|------|--------|
| OnStart | const Want& | void | UI 初始化 | 稳定 |
| OnActive | const Want& | void | 获得焦点 | 稳定 |
| OnInactive | - | void | 失去焦点 | 稳定 |
| OnBackground | - | void | 进入后台 | 稳定 |
| OnStop | - | void | 停止 | 稳定 |
| SetCyclePlayView | - | void | 私有，创建动画视图 | 稳定 |

### 3.3 关键实现

**SetCyclePlayView 实现**：
```cpp
void ScreensaverAbilitySlice::SetCyclePlayView()
{
    imageAnimator_ = new UIImageAnimatorView();
    imageAnimator_->Resize(Screen::GetInstance().GetWidth(), 
                           Screen::GetInstance().GetHeight());
    imageAnimator_->SetImageAnimatorSrc(g_imageAnimatorInfo, 
                                        IMAGE_TOTEL_NUM, 
                                        IMAGE_ANIMATOR_TIME_S);
    rootView_->Add(imageAnimator_);

    auto onClick = [this] (UIView& view, const Event& event) -> bool {
        TerminateAbility();  // 点击退出
        return true;
    };
    exitListener_ = new EventListener(onClick, nullptr);
    imageAnimator_->SetOnClickListener(exitListener_);
    imageAnimator_->SetTouchable(true);
    imageAnimator_->Start();
}
```

**证据**：`screensaver_ability_slice.cpp:46-62`

---

## 4. EventListener

### 4.1 类定义

**文件**：`screensaver/src/main/cpp/event_listener.h`

```cpp
namespace OHOS {
using OnEventFunc = std::function<bool(UIView &view, const Event &event)>;

class EventListener : public UIView::OnClickListener, 
                      public UIView::OnLongPressListener {
public:
    EventListener() = delete;
    ~EventListener() override = default;
    EventListener(OnEventFunc onClick, OnEventFunc onLongPress);

    bool OnClick(UIView& view, const ClickEvent &event) override;
    bool OnLongPress(UIView& view, const LongPressEvent &event) override;

private:
    OnEventFunc onClick_ {};
    OnEventFunc onLongPress_ {};
};
} // namespace OHOS
```

### 4.2 接口说明

| 接口 | 参数 | 返回 | 职责 | 稳定性 |
|------|------|------|------|--------|
| 构造函数 | onClick, onLongPress | - | 初始化回调函数 | 稳定 |
| OnClick | UIView&, ClickEvent | bool | 处理点击事件 | 稳定 |
| OnLongPress | UIView&, LongPressEvent | bool | 处理长按事件 | 稳定 |

**证据**：`event_listener.h:30-65`

### 4.3 空指针检查

```cpp
bool OnClick(UIView& view, const ClickEvent &event) override
{
    if (!onClick_) {
        return false;
    }
    UIView *currentView = &view;
    if (currentView == nullptr) {
        return false;
    }
    return onClick_(*currentView, event);
}
```

**证据**：`event_listener.h:38-48`

---

## 5. UI 配置常量

### 5.1 常量定义

**文件**：`screensaver/src/main/cpp/ui_config.h`

```cpp
namespace OHOS {
static constexpr uint16_t IMAGE_ANIMATOR_TIME_S = 2 * 1000;  // 2秒
static constexpr uint8_t IMAGE_TOTEL_NUM = 5;                // 5张图片

static const char* const IMG_DEFAULT_001_PATH = 
    "/storage/app/run/com.huawei.screensaver/screensaver/assets/...";
static const char* const IMG_DEFAULT_002_PATH = ...;
static const char* const IMG_DEFAULT_003_PATH = ...;
static const char* const IMG_DEFAULT_004_PATH = ...;
static const char* const IMG_DEFAULT_005_PATH = ...;
} // namespace OHOS
```

**证据**：`ui_config.h:22-34`

---

## 6. 依赖方向

```
ScreensaverAbility
    │
    └── 继承自 Ability (ability_lite)
    
ScreensaverAbilitySlice
    │
    ├── 继承自 AbilitySlice (ability_lite)
    ├── 依赖 RootView (ui_lite)
    ├── 依赖 UIImageAnimatorView (ui_lite)
    ├── 依赖 EventListener (自实现)
    └── 依赖 Screen (common)

EventListener
    │
    ├── 依赖 UIView (ui_lite)
    ├── 依赖 ClickEvent/LongPressEvent (ui_lite)
    └── 依赖 std::function (C++11)
```

**稳定性标注**：
- **稳定**：框架层定义的接口，版本间兼容
- **内部**：自实现模块，可能变化

---

## 7. 文档导航

- **返回**：[架构设计](03_Architecture.md) → 组件交互
- **下一步**：[构建系统](06_Build.md) → 编译配置
- **相关**：[安全评审](07_Security.md) → 安全考量
