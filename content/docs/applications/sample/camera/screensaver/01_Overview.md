# 项目概览

## 1. 项目定位

### 1.1 角色与边界

本项目是 **OpenHarmony 图形子系统**的屏保功能参考实现，属于用户态应用层。

**项目边界**：
- 提供完整的屏保 Ability 实现
- 包含 UI 渲染与事件处理逻辑
- 不涉及系统屏保服务核心调度
- 不涉及壁纸管理或锁屏功能

**依赖关系**：
```
本项目
    │
    ├── 上游依赖
    │   ├── ability_lite (Ability 生命周期框架)
    │   ├── ui_lite (UI 渲染框架)
    │   ├── surface_lite (图形表面管理)
    │   └── bundle_lite (应用包管理)
    │
    └── 第三方依赖
        ├── libjpeg / libpng (图像解码)
        └── giflib (GIF 动画支持)
```

### 1.2 核心能力

| 能力 | 说明 |
|------|------|
| 图像轮播 | 循环播放 5 张预设图片，每张停留 2 秒 |
| 屏幕适配 | 根据设备屏幕尺寸动态调整显示区域 |
| 点击退出 | 用户点击任意位置即可退出屏保 |
| 资源管理 | 完善的资源释放与清理机制 |

### 1.3 运行环境

**适配系统版本**：
- OpenHarmony Mini 系统
- OpenHarmony Small 系统

**支持设备类型**：
- phone (手机)
- tv (电视)
- tablet (平板)
- car (车载)
- smartWatch (智能手表)
- sportsWatch (运动手表)
- smartVision (智能眼镜)

**C++ 标准**：C++11 或更高版本

---

## 2. 关键概念

### 2.1 Ability 与 AbilitySlice

**Ability**：OpenHarmony 应用的基本运行单元，负责管理应用的生命周期。

**AbilitySlice**：Ability 的单个页面/屏幕，一个 Ability 可包含多个 AbilitySlice。

本项目结构：
```
ScreensaverAbility (容器)
    │
    └── ScreensaverAbilitySlice (唯一页面)
             │
             ├── UIImageAnimatorView (图像动画)
             └── EventListener (事件处理)
```

### 2.2 RootView

UI Lite 框架的根视图容器，所有 UI 组件需添加到 RootView 中才能显示。

**证据**：`screensaver_ability_slice.cpp:66`
```cpp
rootView_ = RootView::GetWindowRootView();
```

### 2.3 UIImageAnimatorView

用于显示序列帧动画的 UI 组件，支持自动循环播放。

**配置参数** (`ui_config.h:22-23`)：
- `IMAGE_ANIMATOR_TIME_S`: 动画间隔（2秒）
- `IMAGE_TOTEL_NUM`: 图片数量（5张）

### 2.4 EventListener

自定义事件监听器，实现了点击和长按事件的统一处理接口。

**证据**：`event_listener.h:30-65`

---

## 3. 应用配置

### 3.1 Bundle 信息

| 配置项 | 值 |
|--------|-----|
| bundleName | com.huawei.screensaver |
| vendor | huawei |
| versionCode | 1 |
| versionName | 1.0 |

**证据**：`config.json:3-8`

### 3.2 Ability 配置

| 配置项 | 值 |
|--------|-----|
| name | ScreensaverAbility |
| label | screensaver |
| launchType | standard |
| type | page |
| visible | true |

**证据**：`config.json:31-37`

---

## 4. 系统依赖

### 4.1 组件依赖

| 组件 | 用途 |
|------|------|
| ability_lite | Ability 生命周期管理 |
| bundle_framework_lite | 应用包框架 |
| surface_lite | 图形表面 |
| ui_lite | UI 组件 |
| graphic_utils_lite | 图形工具库 |
| kv_store | 键值存储 |
| syspara_lite | 系统参数 |
| samgr_lite | 系统能力管理 |
| utils_base | 基础工具库 |

### 4.2 第三方依赖

| 库 | 用途 |
|----|------|
| libjpeg | JPEG 图像解码 |
| libpng | PNG 图像解码 |
| giflib | GIF 动画支持 |
| cjson | JSON 解析 |
| bounds_checking_function | 安全字符串函数 |

---

## 5. 文档导航

- **下一步**：[目录结构](02_Directory_Structure.md) → 了解代码组织
- **相关章节**：[架构设计](03_Architecture.md) → 理解组件交互
- **构建说明**：[构建系统](06_Build.md) → 编译流程
