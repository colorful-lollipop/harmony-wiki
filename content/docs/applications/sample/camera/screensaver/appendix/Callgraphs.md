# 关键调用链

## 1. 启动调用链

### 1.1 入口 → UI 显示

```
系统调度屏保
    │
    ▼
ability_lite::Ability::OnStart()
    │
    ├── REGISTRY: REGISTER_AA(ScreensaverAbility)
    │        证据: screensaver_ability.cpp:19
    │
    └── SetMainRoute("ScreensaverAbilitySlice")
              │
              ▼
    ScreensaverAbilitySlice::OnStart()
              │
              ├── RootView::GetWindowRootView()
              │         证据: screensaver_ability_slice.cpp:66
              │
              ├── SetCyclePlayView()
              │         │
              │         ├── new UIImageAnimatorView()
              │         │         证据: screensaver_ability_slice.cpp:48
              │         │
              │         ├── Screen::GetInstance().GetWidth/Height()
              │         │         证据: screensaver_ability_slice.cpp:49
              │         │
              │         ├── SetImageAnimatorSrc()
              │         │         证据: screensaver_ability_slice.cpp:50
              │         │
              │         ├── new EventListener(onClick, nullptr)
              │         │         证据: screensaver_ability_slice.cpp:57
              │         │
              │         └── SetOnClickListener(exitListener_)
              │                   证据: screensaver_ability_slice.cpp:58
              │
              └── SetUIContent(rootView_)
                        证据: screensaver_ability_slice.cpp:71
```

### 1.2 依赖模块

| 调用路径 | 依赖模块 | 说明 |
|----------|----------|------|
| Ability 生命周期 | ability_lite | 框架提供 |
| RootView | ui_lite | UI 框架 |
| Screen | common | 屏幕信息 |
| UIImageAnimatorView | ui_lite | 动画组件 |
| EventListener | 自实现 | 事件处理 |

---

## 2. 用户交互调用链

### 2.1 点击退出

```
用户触摸屏幕
    │
    ▼
系统分发点击事件
    │
    ▼
UIImageAnimatorView::OnClickListener
    │
    ▼
EventListener::OnClick(UIView&, ClickEvent&)
    │
    ├── 证据: event_listener.h:38-48
    │
    └── onClick_(view, event)  // Lambda 回调
              │
              └── TerminateAbility()
                        证据: screensaver_ability_slice.cpp:54
```

---

## 3. 资源管理调用链

### 3.1 析构与释放

```
ScreensaverAbilitySlice 析构
    │
    ▼
~ScreensaverAbilitySlice()
    │
    ├── if (imageAnimator_ != nullptr)
    │         │
    │         ├── delete imageAnimator_
    │         │         证据: screensaver_ability_slice.cpp:36
    │         │
    │         └── imageAnimator_ = nullptr
    │
    └── if (exitListener_ != nullptr)
              │
              ├── delete exitListener_
              │         证据: screensaver_ability_slice.cpp:41
              │
              └── exitListener_ = nullptr
```

---

## 4. 配置加载调用链

### 4.1 图片路径配置

```
ui_config.h (编译时常量)
    │
    ├── IMG_DEFAULT_001_PATH
    │   = "/storage/app/run/com.huawei.screensaver/..."
    │       证据: ui_config.h:25-26
    │
    ├── IMG_DEFAULT_002_PATH
    │   = "/storage/app/run/com.huawei.screensaver/..."
    │       证据: ui_config.h:27-28
    │
    └── ... (共 5 张图片)
```

---

## 5. 构建调用链

```
BUILD.gn
    │
    ├── shared_library("screensaver")
    │         │
    │         ├── sources: [screensaver_ability.cpp, screensaver_ability_slice.cpp]
    │         │         证据: BUILD.gn:17-20
    │         │
    │         ├── deps: [aafwk_abilitykit_lite, ui_lite, surface_lite, ...]
    │         │         证据: BUILD.gn:22-31
    │         │
    │         └── defines: [ENABLE_WINDOW, ABILITY_WINDOW_SUPPORT, ...]
    │                   证据: BUILD.gn:41-45
    │
    └── hap_pack("screensaver_hap")
              │
              ├── deps: [:screensaver]
              │         证据: BUILD.gn:49
              │
              ├── ability_so_path: $root_out_dir/libscreensaver.so
              │         证据: BUILD.gn:52
              │
              └── cert_profile: cert/xxx.p7b
                        证据: BUILD.gn:54
```

---

## 6. 文档导航

- **返回**：[概览](01_Overview.md) → 项目定位
- **相关**：[架构设计](03_Architecture.md) → 组件图
- **相关**：[内部 API](05_Inner_API.md) → 模块接口
