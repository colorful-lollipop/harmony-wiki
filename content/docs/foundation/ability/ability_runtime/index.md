# Ability Runtime - 元能力运行时

## 项目简介

**ability_runtime** 是 OpenHarmony 元能力子系统的核心运行时组件，负责管理 Ability 组件的运行及生命周期调度。

### 核心职责

- **Ability 生命周期管理**：UIAbility、ExtensionAbility 等组件的创建、启动、暂停、恢复、停止
- **组件调度**：协调各 Ability 运行关系，支持跨应用进程间和同一进程内调用
- **系统服务集成**：与 AppManagerService、BundleManagerService 等系统服务协同工作
- **多语言运行时支持**：提供 JS、NAPI、ETS (ArkTS)、CJ 等语言的绑定支持

### 架构位置

```
OpenHarmony 系统
├── 应用层
│   └── 开发者开发的 Ability 应用
├── 框架层
│   └── ability_runtime（本组件）
│       ├── frameworks/        # 框架实现
│       ├── interfaces/        # 接口定义
│       └── services/          # 系统服务
└── 系统服务层
    ├── BundleManagerService   # 包管理服务
    ├── AppManagerService      # 应用管理服务
    └── ...
```

## 两种应用模型

### FA 模型（Feature Ability）

- **适用版本**：API 8 及更早版本
- **组件类型**：
  - PageAbility：页面展示
  - ServiceAbility：后台服务
  - DataAbility：数据共享
  - FormAbility：卡片能力

### Stage 模型（推荐）

- **适用版本**：API 9 及更高版本
- **组件类型**：
  - UIAbility：页面展示
  - ExtensionAbility：扩展服务（多种子类型）
    - ServiceExtensionAbility
    - DataExtensionAbility
    - FormExtensionAbility
    - UIExtensionAbility
    - ...

**主要差异**：
| 特性 | FA 模型 | Stage 模型 |
|------|--------|----------|
| 开发方式 | 类 Web API | 面向对象 |
| JS VM 实例 | 每个 Ability 独立 | 进程内共享 |
| 对象共享 | 不支持 | 支持 |
| 包描述文件 | config.json | module.json5 |

## 核心模块

### 系统服务

| 模块 | 职责 | 代码位置 |
|------|------|---------|
| AbilityManagerService | Ability 生命周期管理、组件调度 | `services/abilitymgr/` |
| AppManagerService | 应用进程管理、状态跟踪 | `services/appmgr/` |
| UriPermissionManager | URI 权限管理 | `services/uripermmgr/` |
| QuickFixManager | 热修复管理 | `services/quickfixmgr/` |

### 框架层

| 模块 | 职责 | 代码位置 |
|------|------|---------|
| Native 框架 | C++ 原生实现 | `frameworks/native/` |
| N-API | JS 调用接口 | `frameworks/js/napi/` |
| ETS ANI | ArkTS 绑定 | `frameworks/ets/` |
| CJ FFI | CJ 语言绑定 | `frameworks/cj/` |

## 快速开始

### 启动一个 Ability

```typescript
// 通过 want 描述启动目标 Ability
import { wantAgent } from '@kit.AbilityKit';

let want = {
  bundleName: "com.example.myapp",
  abilityName: "EntryAbility"
};

// 使用 abilityManager API 启动
import { abilityManager } from '@kit.AbilityKit';
abilityManager.startAbility(want).then(() => {
  console.log("Ability 启动成功");
});
```

### 获取应用上下文

```typescript
import { context } from '@kit.AbilityKit';

let applicationContext = context.getApplicationContext();
```

## 关键概念

### 生命周期状态

```
onStart → onForeground → [onBackground] → onStop
              ↑
              └── 可见状态
```

### 组件连接（ConnectAbility）

用于 Service 类型 Ability 的跨进程通信：
1. 客户端调用 `connectAbility()` 建立连接
2. 服务端通过 `IRemoteObject` 回调返回结果
3. 使用完成后调用 `disconnectAbility()` 断开连接

## 相关文档

- [项目概述](01_Project_Overview.md) - 详细了解核心能力和关键概念
- [目录结构](02_Directory_Structure.md) - 代码组织结构
- [架构说明](03_Architecture.md) - 组件图和数据流
- [N-API 参考](04_NAPI_Reference.md) - JS API 接口
- [Inner API](05_Inner_API.md) - 内部组件间接口
- [GN Targets](06_GN_Targets.md) - 构建目标
- [安全评审](08_Security_Review.md) - 安全机制和风险

## 代码证据

本首页内容基于以下源码证据：

- 项目描述：`README_zh.md`
- 模块定义：`bundle.json`
- 架构说明：`README_zh.md` 第 1-25 行
- 两种模型对比：`README_zh.md` 第 27-43 行
- 目录结构：`README_zh.md` 第 48-80 行
