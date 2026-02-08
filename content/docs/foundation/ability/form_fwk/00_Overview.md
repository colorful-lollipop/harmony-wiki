# Form Fmk Wiki - 首页

> OpenHarmony 卡片管理框架工程文档

## 什么是 Form？

**卡片（Form）** 是 OpenHarmony 中的一种界面展示形式，可以将应用的重要信息或操作前置到卡片，以达到服务直达的目的。

卡片常用于嵌入到其他应用（当前只支持系统应用）中作为其界面的一部分显示，并支持拉起页面、发送消息等基础的交互功能。

## 核心概念

### 卡片提供方 (Form Provider)
- 提供卡片显示内容原子化服务
- 定义卡片的显示内容、控件布局以及控件点击事件
- 开发卡片生命周期回调函数（FormExtension/FormAbility）

### 卡片使用方 (Form Host)
- 显示卡片内容的应用
- 可自由配置应用中卡片展示的位置

### 卡片管理服务 (Form Manager Service)
- 管理系统中所添加卡片的常驻代理服务
- 管理卡片的生命周期
- 维护卡片信息以及卡片事件的调度

## 支持的模型

| 模型 | API 版本 | 扩展组件 |
|------|----------|----------|
| FA 模型 | API 8 及更早 | FormAbility |
| Stage 模型 | API 9+ | FormExtensionAbility |

## 快速开始

### 开发者角色

开发者仅需作为**卡片提供方**进行卡片内容的开发，卡片使用方和卡片管理服务由系统自动处理。

> **说明**：卡片使用方和提供方不要求常驻运行，在需要添加/删除/请求更新卡片时，卡片管理服务会拉起卡片提供方获取卡片信息。

### 开发步骤

1. **FA 模型**：开发 `LifecycleForm` 生命周期回调 → 创建 `FormBindingData` → 通过 `FormProvider` 更新卡片
2. **Stage 模型**：开发 `FormExtension` 生命周期回调 → 创建 `FormBindingData` → 通过 `FormProvider` 更新卡片

## 文档导航

- [项目概述](01_Project_Overview.md)
- [架构设计](02_Architecture.md)
- [N-API 参考](03_NAPI_Reference.md)
- [Inner API](04_Inner_API.md)
- [GN 构建](05_GN_Targets.md)
- [编译产物](06_Build_Artifacts.md)
- [安全评审](07_Security_Review.md)
