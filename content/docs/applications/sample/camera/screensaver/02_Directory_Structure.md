# 目录结构

## 1. 整体目录树

```
applications/sample/camera/screensaver/
├── cert/                          # 签名证书
│   └── com.huawei.screensaver_AppProvision_release.p7b
├── figures/                       # 文档图片
│   └── screensaver_en.png
├── screensaver/src/main/
│   ├── config.json                # 应用配置（HAP）
│   ├── cpp/                       # C++ 源代码
│   │   ├── screensaver_ability.h     # Ability 声明
│   │   ├── screensaver_ability.cpp   # Ability 实现
│   │   ├── screensaver_ability_slice.h  # AbilitySlice 声明
│   │   ├── screensaver_ability_slice.cpp  # AbilitySlice 实现
│   │   ├── event_listener.h           # 事件监听器
│   │   └── ui_config.h                # UI 配置（图片路径等）
│   └── resources/                 # 应用资源
│       ├── base/
│       │   ├── element/
│       │   │   └── string.json        # 字符串资源
│       │   └── media/
│       │       └── img_default_*.png  # 屏保图片（5张）
│       └── rawfile/
│           └── images/              # 原始图片资源
├── BUILD.gn                       # GN 构建入口
├── bundle.json                    # 组件配置
├── README.md                      # 英文说明
└── README_zh.md                   # 中文说明
```

## 2. 模块职责

### 2.1 根目录文件

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | GN 构建配置，定义 `screensaver` 和 `screensaver_hap` targets |
| `bundle.json` | 组件配置，声明子系统、依赖、构建入口 |
| `cert/` | 签名证书目录，包含 release 版本签名文件 |
| `figures/` | 文档图片资源 |

### 2.2 cpp/ 源代码目录

| 文件 | 职责 | 代码行数 |
|------|------|----------|
| `screensaver_ability.h` | ScreensaverAbility 类声明 | ~33 行 |
| `screensaver_ability.cpp` | ScreensaverAbility 类实现，生命周期管理 | ~46 行 |
| `screensaver_ability_slice.h` | ScreensaverAbilitySlice 类声明 | ~50 行 |
| `screensaver_ability_slice.cpp` | ScreensaverAbilitySlice 类实现，UI 渲染 | ~100 行 |
| `event_listener.h` | EventListener 事件监听器实现 | ~68 行 |
| `ui_config.h` | UI 配置常量（图片路径、动画参数） | ~37 行 |

**证据**：`BUILD.gn:17-20`

### 2.3 resources/ 资源目录

| 目录/文件 | 职责 |
|-----------|------|
| `base/element/string.json` | 字符串资源定义 |
| `base/media/img_default_*.png` | 5 张屏保预设图片 |
| `rawfile/images/` | 原始图片资源备份 |

---

## 3. 代码统计（不含测试）

| 统计项 | 数量 |
|--------|------|
| C++ 头文件 | 3 个 |
| C++ 源文件 | 3 个 |
| 总代码行数 | ~334 行 |
| 配置文件 | 3 个 |
| 图片资源 | 5 张 |

---

## 4. 文件依赖关系

```
screensaver_ability.cpp
    └── screensaver_ability.h
        └── ability_loader.h, want.h

screensaver_ability_slice.cpp
    └── screensaver_ability_slice.h
        ├── ability_loader.h, ability_info.h, bundle_info.h
        ├── components/root_view.h, ui_image_animator.h
        ├── event_listener.h
        ├── ui_config.h
        └── want.h
        └── common/screen.h

event_listener.h
    ├── components/ui_view.h
    ├── events/click_event.h, events/event.h
    └── events/long_press_event.h
```

---

## 5. 文档导航

- **下一步**：[架构设计](03_Architecture.md) → 组件交互
- **返回**：[概览](01_Overview.md) → 项目定位
- **相关**：[构建系统](06_Build.md) → 构建配置
