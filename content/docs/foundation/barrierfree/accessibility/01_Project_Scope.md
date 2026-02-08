# 项目定位与边界

## 目的

本文档说明 Accessibility 子系统的定位、边界、核心能力、运行环境和关键概念，帮助读者理解项目范围。

## 适用范围

- 需要了解项目边界的产品经理
- 架构师和技术负责人
- 新项目成员

## 关键结论

### 项目定位

Accessibility 子系统是 OpenHarmony 无障碍能力的核心实现，位于 **BarrierFree 子系统**下，提供：
1. **标准机制**: 应用程序和辅助应用之间信息交换
2. **开发支持**: 无障碍应用和辅助应用的双向开发工具

### 核心能力

| 能力类别 | 功能 |
|---------|------|
| **Core** | 基础无障碍能力（事件、元素访问、操作执行） |
| **Vision** | 视觉障碍支持（屏幕朗读、放大镜、高对比度等） |
| **Hearing** | 听觉障碍支持（字幕、音频调整） |

### 项目边界

#### 在范围内

✅ 无障碍事件传输和处理
✅ 无障碍元素信息查询和操作
✅ 无障碍配置管理
✅ 手势注入
✅ 辅助能力连接管理
✅ 窗口和显示管理
✅ 多用户支持

#### 不在范围内

❌ 应用 UI 渲染（由 ACE 负责）
❌ 输入事件原始采集（由 Input 负责）
❌ 系统底层权限管理（由 Access Token 负责）
❌ 设备底层驱动（由 HAL 负责）

---

## 详细内容

### 核心能力说明

#### SystemCapability.BarrierFree.Accessibility.Core

**能力描述**: 提供基础无障碍能力

**主要功能**:
- 无障碍事件类型定义（60+ 种）
- 无障碍操作类型定义（18 种）
- 元素信息查询
- 操作执行
- 手势注入

**证据**: `interfaces/innerkits/common/include/accessibility_constants.h:1`

#### SystemCapability.BarrierFree.Accessibility.Vision

**能力描述**: 提供视觉障碍支持

**主要功能**:
- 屏幕朗读
- 屏幕放大镜（全屏/窗口）
- 高对比度文本
- 反色
- 触摸探索
- 屏幕遮蔽

**证据**: `services/aams/include/magnification_manager.h`, `services/aams/include/accessibility_touch_exploration.h`

#### SystemCapability.BarrierFree.Accessibility.Hearing

**能力描述**: 提供听觉障碍支持

**主要功能**:
- 字幕显示
- 音频单声道
- 音频平衡

**证据**: `services/aams/include/accessibility_settings.h` 中的字幕和音频设置

### 运行环境

#### 系统要求

- **OpenHarmony 版本**: 4.0+
- **依赖的系统服务**:
  - Window Manager（窗口管理）
  - Input Manager（输入管理）
  - Event Handler（事件处理）
  - Display Manager（显示管理）
  - Bundle Manager（包管理）
  - Access Token（权限管理）
  - HiSysEvent（系统事件）
  - Power Manager（电源管理）
  - Data Share（数据共享）

#### 依赖组件（证据）

**证据**: `bundle.json:35-76`

关键依赖：
- `graphic_2d` - 图形渲染
- `window_manager` - 窗口管理
- `input` - 输入事件
- `ability_runtime` - 能力运行时
- `access_token` - 权限验证
- `bundle_framework` - 包管理

### 关键概念

#### 无障碍扩展能力 (Accessibility Extension Ability)

**定义**: 使用 AccessibilityExtensionAbility 开发的特殊应用，提供无障碍服务。

**功能**:
- 监听应用无障碍事件
- 查询应用无障碍元素信息
- 执行无障碍操作
- 注入手势

**证据**: `interfaces/kits/napi/accessibility_extension/`

#### 无障碍元素 (Accessibility Element)

**定义**: UI 组件的无障碍表示，包含可访问性属性和操作。

**属性示例**:
- 文本、描述、提示文本
- 组件类型、状态
- 支持的操作
- 位置、大小

**证据**: `interfaces/innerkits/common/include/accessibility_element_info.h`

#### 无障碍事件 (Accessibility Event)

**定义**: UI 中发生的可访问性事件。

**类型示例**:
- 焦点变化
- 点击、长按
- 滚动
- 文本更新
- 窗口变化

**证据**: `interfaces/innerkits/common/include/accessibility_event_info.h`

#### 手势注入 (Gesture Injection)

**定义**: 无障碍扩展应用向系统注入触摸手势的能力。

**用途**:
- 模拟用户触摸
- 执行多指手势
- 执行复杂手势序列

**证据**: `interfaces/kits/napi/accessibility_extension_context/` 中的 injectGesture

### 特性开关

| 特性 | 默认值 | 说明 |
|------|---------|------|
| accessibility_feature_coverage | false | 覆盖率特性 |
| accessibility_watch_feature | false | 手表特性 |
| accessibility_dynamic_support | false | 动态支持 |

**证据**: `bundle.json:21-25`

### 安装路径

| 产物类型 | 安装路径 |
|---------|---------|
| 系统服务 .so | /system/lib64/libaccessibleabilityms.z.so |
| N-API .so | /system/lib64/module/ |
| ANI .abc | /system/lib64/module/ets/ |

---

## 相关链接

- [项目概览](00_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)

---

最后更新: 2026-02-06
