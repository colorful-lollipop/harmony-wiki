# 项目概述

> 本文档描述 Form Fwk 的项目定位、边界与核心能力

## 项目定位

**Form Fwk**（卡片管理框架）是 OpenHarmony 元能力子系统的核心组成部分，负责管理系统中卡片（Form）的全生命周期。

- **名称**: `@ohos/form_fwk`
- **版本**: 3.1
- **系统能力**: `SystemCapability.Ability.Form`
- **所属子系统**: ability

## 核心能力

### 1. 卡片生命周期管理
- 卡片的创建、删除、更新、释放
- 临时卡片转正常卡片
- 卡片可见性通知

### 2. 卡片刷新机制
- 周期性刷新
- 定时刷新
- 事件触发刷新
- 按需刷新

### 3. 卡片渲染
- 卡片视图渲染
- 渲染状态管理
- 渲染异常恢复

### 4. 跨设备卡片（待补充）

### 5. 卡片分享
- 卡片数据分享
- 跨设备分享

## 子模块职责

| 子模块 | 职责 | 主要文件位置 |
|--------|------|-------------|
| 卡片 JS N-API 模块 | 提供外部接口，与卡片管理服务交互，负责事件通知调度，通过 ArkUI 更新卡片视图 | `frameworks/js/napi/` |
| 卡片管理服务模块 | 管理系统中卡片的常驻代理服务，管理卡片生命周期，维护卡片信息及事件调度 | `services/` |

## 运行环境

### 依赖组件

Form Fwk 依赖以下核心组件：

| 组件 | 用途 |
|------|------|
| ability_runtime | Ability 运行时环境 |
| bundle_framework | Bundle 管理框架 |
| ipc | 进程间通信 |
| napi | Node-API |
| safwk | System Ability 框架 |
| samgr | 系统服务管理框架 |
| hilog | 日志系统 |
| ffrt | 高性能任务调度 |

### 系统要求

- **适配系统类型**: standard（标准系统）
- **最小 API 版本**: API 8（FA 模型）、API 9+（Stage 模型）

## 关键概念

### 卡片 ID (FormId)
- 每张卡片拥有唯一标识符
- 用于卡片的所有操作

### 卡片配置 (FormInfo)
- 卡片元数据
- 包含卡片名称、尺寸、组件列表等

### 卡片数据 (FormBindingData)
- 卡片绑定的动态数据
- 用于更新卡片显示内容

### 卡片状态 (FormState)
- 卡片当前状态
- 包括：默认、就绪、未知等

## 相关仓库

| 仓库 | 用途 |
|------|------|
| interface_sdk-js | ArkTS API 接口 |
| ability_ability_base | 元能力基础库 |
| ability_ability_runtime | 元能力运行时 |
| ability_dmsfwk | 分布式调度框架 |
| arkui_ace_engine | ArkUI 框架 |
