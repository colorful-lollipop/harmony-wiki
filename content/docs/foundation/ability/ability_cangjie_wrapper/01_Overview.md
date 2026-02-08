# 项目概览

## 项目定位

`ability_cangjie_wrapper` 是 OpenHarmony 系统中 **元能力（Ability）子系统** 的仓颉（Cangjie）语言封装层。

### 核心职责

| 职责 | 描述 | 相关模块 |
|-----|------|---------|
| **生命周期管理** | 提供 UIAbility 的创建、销毁、前后台切换等生命周期回调 | `ui_ability` |
| **上下文访问** | 提供应用上下文（Context）访问应用资源、获取组件信息 | `context.cj` |
| **组件协调** | 提供模块级组件管理器（AbilityStage）协调模块内资源 | `ability_stage` |
| **错误观测** | 提供错误观测器（ErrorObserver）注册与注销能力 | `error_manager` |
| **测试支持** | 提供自动化测试框架（AbilityDelegator）管理能力 | `ability_delegator_registry` |

### 在系统中的位置

```
用户应用 (Cangjie)
       │
       ▼
┌─────────────────────────────────────────┐
│    ability_cangjie_wrapper (本仓库)      │
│  - N-API 绑定层                          │
│  - 仓颉 FFI 桥接                         │
│  - API 导出与类型转换                     │
└─────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│     ability_runtime (Native 实现)        │
│  - 生命周期调度                         │
│  - IPC 通信                             │
│  - 权限管理                             │
└─────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│       OpenHarmony 内核 (Linux)           │
└─────────────────────────────────────────┘
```

**证据来源**：`README_zh.md:5-6` - "元能力仓颉封装实现对Ability的运行及生命周期进行统一的调度和管理"

## 核心能力

### 已支持功能

| 功能 | 说明 | API 入口 |
|-----|------|---------|
| **UIAbility 组件** | 含 UI 界面的应用组件，提供生命周期回调 | `UIAbility` |
| **应用上下文** | 访问组件信息、获取资源、拉起/销毁 Ability | `Context`, `UIAbilityContext` |
| **组件管理器** | Module 级资源预加载、线程创建初始化 | `AbilityStage` |
| **错误观测管理** | 注册/注销错误观察器 | `ErrorManager` |
| **自动化测试框架** | 监视 Ability 生命周期、获取测试参数 | `AbilityDelegatorRegistry` |
| **应用恢复** | 应用故障后自动重启并恢复状态 | `AppRecovery` |
| **意图传递** | 组件间信息传递与启动参数 | `Want` |

### 暂不支持功能

| 功能 | 说明 | 原因 |
|-----|------|-----|
| **ExtensionAbility** | 特定场景拓展能力 | Beta 阶段未实现 |
| **意图执行基类** | 对接端侧意图框架 | Beta 阶段未实现 |
| **AppStartup** | 应用启动框架 | Beta 阶段未实现 |

**证据来源**：`README_zh.md:90-95` - "与ArkTS提供的API能力相比，暂不支持以下功能"

## 运行环境

### 系统要求

| 要求 | 说明 |
|-----|------|
| **操作系统** | OpenHarmony（标准设备 standard） |
| **系统版本** | API Level 22+ |
| **运行时** | ArkTS Runtime + Cangjie Runtime |

### 依赖的子系统

| 子系统 | 用途 | 依赖方式 |
|-------|------|---------|
| `ability_runtime` | Ability 核心功能原生实现 | FFI 绑定 |
| `access_token` | 权限校验与访问控制 | FFI 绑定 |
| `cangjie_ark_interop` | 仓颉-ArkTS 互操作基础 | 编译依赖 |
| `arkui_cangjie_wrapper` | UI 组件与基础类型 | 编译依赖 |
| `hiviewdfx_cangjie_wrapper` | 日志打印 | 编译依赖 |
| `bundlemanager_cangjie_wrapper` | 包信息获取 | 编译依赖 |
| `global_cangjie_wrapper` | 资源管理 | 编译依赖 |
| `window_cangjie_wrapper` | 窗口实例管理 | 编译依赖 |
| `communication_cangjie_wrapper` | IPC 通信 | 编译依赖 |
| `multimedia_cangjie_wrapper` | 媒体资源 | 编译依赖 |
| `accesscontrol_cangjie_wrapper` | 权限管理 | 编译依赖 |
| `testfwk_cangjie_wrapper` | 测试框架 | 编译依赖 |

**证据来源**：`bundle.json:25-38` - `deps.components` 列表

## 关键概念

### Stage 模型

本仓库采用 **Stage 模型**，是 OpenHarmony 应用的核心架构模式：

```
┌─────────────────────────────────────────────────────┐
│                    Process (进程)                    │
│  ┌──────────────────────────────────────────────┐  │
│  │           AbilityStage (Module级)              │  │
│  │  ┌────────────────────────────────────────┐   │  │
│  │  │  UIAbility 实例 1                       │   │  │
│  │  │  - UIAbilityContext                    │   │  │
│  │  │  - ApplicationContext (共享)            │   │  │
│  │  └────────────────────────────────────────┘   │  │
│  │  ┌────────────────────────────────────────┐   │  │
│  │  │  UIAbility 实例 2                       │   │  │
│  │  │  - UIAbilityContext                    │   │  │
│  │  │  - ApplicationContext (共享)            │   │  │
│  │  └────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### 上下文层级

| 上下文类型 | 作用域 | 说明 |
|-----------|-------|------|
| **ApplicationContext** | 应用级 | 共享于所有 Ability 实例 |
| **AbilityStageContext** | 模块级 | 所属 AbilityStage 的上下文 |
| **UIAbilityContext** | 组件级 | 单个 UIAbility 实例的上下文 |

### 生命周期状态

```
┌─────────┐    onCreate()    ┌──────────┐    onWindowStageCreate()    ┌─────────────┐
│ Initial │ ──────────────▶ │ Created  │ ───────────────────────────▶ │ WindowCreated│
└─────────┘                  └──────────┘                             └─────────────┘
                                                                 │   onForeground()
                                                                 ▼
                                                          ┌─────────────┐
                                                          │ Foreground  │
                                                          └─────────────┘
                                                          │  onBackground()
                                                          ▼
                                                          ┌─────────────┐
                                                          │ Background  │
                                                          └─────────────┘
                                                          │  onDestroy()
                                                          ▼
                                                          ┌─────────┐
                                                          │ Destroy │
                                                          └─────────┘
```

## 模块目录结构

```
foundation/ability/ability_cangjie_wrapper/
├── figures/                              # 架构图
├── kit/
│   └── AbilityKit/                       # Kit 聚合目标
│       ├── BUILD.gn
│       └── index.cj                      # 导出声明
├── ohos/
│   ├── ability/                          # 基础类型
│   │   ├── ability_result/
│   │   └── connect_options/
│   ├── app/
│   │   ├── ability/                      # 核心模块
│   │   │   ├── ability_constant/        # 常量定义
│   │   │   ├── ability_delegator_registry/ # 测试框架
│   │   │   ├── ability_stage/            # 组件管理器
│   │   │   ├── app_recovery/            # 应用恢复
│   │   │   ├── common/                  # 公共类型
│   │   │   ├── completion_handler/      # 完成处理
│   │   │   ├── configuration/          # 配置
│   │   │   ├── context_constant/       # 上下文常量
│   │   │   ├── dialog_request/         # 对话请求
│   │   │   ├── error_manager/          # 错误管理
│   │   │   ├── open_link_options/      # 链接选项
│   │   │   ├── start_options/          # 启动选项
│   │   │   ├── ui_ability/             # UIAbility
│   │   │   ├── want/                   # 意图
│   │   │   └── want_constant/          # 意图常量
│   │   └── application/                # 应用框架
│   │       ├── error_observer/         # 错误观察者
│   │       ├── event_hub/             # 事件中心
│   │       └── test_runner/            # 测试运行器
│   └── BUILD.gn                         # 聚合目标
├── BUILD.gn                              # 根构建入口
├── bundle.json                           # 组件配置
├── README.md / README_zh.md             # 项目说明
└── LICENSE                              # Apache 2.0
```

**证据来源**：`README_zh.md:48-71` - 目录结构说明
