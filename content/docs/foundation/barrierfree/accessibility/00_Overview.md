# 无障碍子系统概览

## 目的

本文档提供 OpenHarmony Accessibility 子系统的总体介绍，包括项目背景、主要功能、架构概览和快速开始指南。

## 适用范围

- 所有关注 Accessibility 子系统的开发者
- 需要了解系统整体架构的开发者
- 新项目成员

## 关键结论

### 项目定位

Accessibility 子系统提供在应用程序和辅助应用之间交换信息的标准机制，支持开发辅助应用增强无障碍功能体验。

### 核心功能

1. **无障碍访问**: 为残障人士提供使用应用的能力（如屏幕朗读）
2. **UI 自动化**: 为开发者提供与应用交互的能力（如 UI 自动化测试）

### 架构概览

Accessibility 子系统采用分层架构：

```
应用层 (Apps)
    ↓
应用框架层 (AAkit/ASACkit/ACkit)
    ↓
系统服务层 (AccessibilityService)
    ↓
接口层 (Innerkits/Kits)
```

### System Ability

- **SA ID**: 801
- **进程名**: accessibility
- **库名**: libaccessibleabilityms.z.so

---

## 详细内容

### 项目背景

Accessibility 子系统是 OpenHarmony BarrierFree 子系统的核心组件，遵循无障碍国际标准（如 WCAG），提供完整的无障碍支持体系。

### 主要功能

#### 1. 无障碍访问

支持残障人士使用应用：
- 视觉障碍：屏幕朗读、屏幕放大镜、高对比度、反色
- 听觉障碍：字幕显示、音频单声道、音频平衡
- 运动障碍：触摸探索、鼠标键、快捷方式、自动点击
- 认知障碍：动画关闭、内容超时调整

#### 2. UI 自动化

为开发者提供自动化能力：
- 元素信息查询
- 操作执行
- 手势注入
- 事件监听

### 架构分层

#### 应用层

- 使用 **AccessibilityExtensionAbility** 开发辅助能力应用
- 一般应用集成为无障碍目标应用
- 系统设置应用配置无障碍开关

#### 应用框架层

| Kit | 全名 | 说明 |
|-----|-------|------|
| **AAkit** | AccessibleAbility Kit | 无障碍辅助能力开发套件 |
| **ASACkit** | AccessibilitySystemAbilityClient Kit | 无障碍能力客户端开发套件 |
| **ACkit** | AccessibilityConfiguration Kit | 无障碍功能设定开发套件 |

#### 系统服务层

- **AccessibilityService**: 无障碍系统服务 (SA 801)
  - 管理无障碍辅助能力连接
  - 提供无障碍能力给客户端
  - 连接其他系统服务提供无障碍输入能力

#### 接口层

- **Innerkits**: 内部 C/C++ 接口（AAkit/ASACkit/ACkit）
- **Kits**: 对外 TS/JS/N-API/ANI/CJ 接口

### 快速开始

#### 作为应用开发者使用无障碍能力

```typescript
import accessibility from '@ohos.accessibility';

// 检查无障碍是否开启
const isOpen = await accessibility.isOpenAccessibility();

// 获取可用无障碍能力
const abilities = await accessibility.getAbilityList();

// 监听状态变化
accessibility.on('accessibilityStateChange', (state) => {
    console.log('State changed:', state);
});
```

#### 作为无障碍扩展开发者

```typescript
import AccessibilityExtensionAbility from '@ohos.application.AccessibilityExtensionAbility';

export default class MyAccessibilityExtension extends AccessibilityExtensionAbility {
    onConnect() {
        console.log('Extension connected');
    }

    onDisconnect() {
        console.log('Extension disconnected');
    }
}
```

---

## 相关链接

- [项目定位与边界](01_Project_Scope.md)
- [目录结构](02_Directory_Structure.md)
- [架构详细说明](03_Architecture.md)
- [N-API 接口文档](04_N-API.md)

---

最后更新: 2026-02-06
