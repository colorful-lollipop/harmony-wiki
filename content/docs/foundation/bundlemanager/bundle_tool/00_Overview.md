# 项目概览 (Overview)

> bundle_tool 项目定位、边界、核心能力与运行环境

## 项目定位

**bundle_tool** 是 OpenHarmony 系统中用于管理应用 Bundle (应用包) 的**命令行调试工具**，通过 hdc (HarmonyOS Device Connector) 在设备 shell 环境中执行。

### 核心定位

```
┌─────────────────────────────────────────────────────────────┐
│                     OpenHarmony 系统                        │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                   hdc shell                           │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │                   bm                             │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │         bundle_tool                       │  │  │  │
│  │  │  │  (Native CLI 工具)                        │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────┘  │
│                      ↑                                     │
│              用户在主机端执行 hdc shell                     │
└─────────────────────────────────────────────────────────────┘
```

**证据来源**: `README_zh.md:5` - "bm工具被hdc工具封装，进入hdc shell命令后，就可以使用bm工具"

---

## 项目边界

### 包含范围

| 模块 | 路径 | 职责 |
|------|------|------|
| frameworks/include | `frameworks/include/` | 头文件声明 |
| frameworks/src | `frameworks/src/` | 源文件实现 |
| BUILD.gn | 根目录 + frameworks/ | GN 构建配置 |

### 不包含范围

| 模块 | 说明 |
|------|------|
| test/ | 测试代码（不纳入 Wiki 范围） |
| bundle_framework | 独立仓库，bundle_tool 仅调用其接口 |
| N-API | 本项目为 CLI 工具，不提供 JS 接口 |

**证据来源**: `bundle.json:7` - `destPath: "foundation/bundlemanager/bundle_tool"`

---

## 核心能力

### 命令清单

| 类别 | 命令 | 功能 |
|------|------|------|
| **包管理** | install | 安装 HAP/HSP |
| | uninstall | 卸载应用 |
| **调试** | dump | 查询应用信息 |
| | get | 获取设备 UDID |
| **生命周期** | enable | 使能应用 |
| | disable | 禁用应用 |
| **清理** | clean | 清理缓存/数据 |
| **快速修复** | quickfix | 补丁安装/查询 |
| **编译** | compile | AOT 编译 |
| | copy-ap | 拷贝 AP 文件 |
| **Overlay** | dump-overlay | 查询 Overlay |
| | dump-target-overlay | 查询目标 Overlay |
| **共享库** | dump-shared | 查询 HSP |
| | dump-dependencies | 查询依赖 |
| **插件** | install-plugin | 安装插件 |
| | uninstall-plugin | 卸载插件 |

**证据来源**: `bundle_command.h:28-43` - HELP_MSG 定义

---

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| 操作系统 | OpenHarmony 4.0+ |
| 运行环境 | hdc shell |
| 依赖子系统 | bundle_framework, ability_runtime, ipc, samgr |

### 硬件要求

| 指标 | 要求 |
|------|------|
| ROM | ~300KB |
| RAM | ~100KB |

**证据来源**: `bundle.json:17-18`

### 调用链路

```
用户 (主机) → hdc shell → bm → bundle_framework (IPC) → BundleManagerService
```

**证据来源**: `frameworks/BUILD.gn:71` - 依赖 `ipc:ipc_core`

---

## 关键概念

### Bundle

指 OpenHarmony 的应用包格式，包括：
- **HAP** (Harmony Ability Package): 应用安装包
- **HSP** (Harmony Shared Package): 共享库包

### ShellCommand 模式

bundle_tool 采用 **ShellCommand** 模式处理命令行：

1. `main()` 接收命令行参数
2. 创建 `BundleManagerShellCommand` 实例
3. 通过 `CreateCommandMap()` 注册命令
4. 执行对应的 `RunAs*Command()` 函数

**证据来源**:
- `main.cpp:19-26`: 入口逻辑
- `bundle_command.h:265`: BundleManagerShellCommand 类

### IPC 通信

bundle_tool 作为客户端，通过 IPC 调用 bundle_framework 的服务：

| 接口 | 功能 |
|------|------|
| IBundleMgr | Bundle 管理（查询、状态变更） |
| IBundleInstaller | Bundle 安装/卸载 |
| IBundleToolCallback | 工具回调 |

**证据来源**: `bundle_command.h:343-344` - 持有 sptr<IBundleMgr>, sptr<IBundleInstaller>

---

## Feature Flags

bundle_tool 支持以下条件编译选项：

| 开关 | 默认值 | 功能 |
|------|--------|------|
| `account_enable_bm` | true | 用户账号相关功能 |
| `overlay_install_bm` | true | Overlay 安装功能 |
| `quick_fix_bm` | true | 快速修复功能 |
| `distributed_bundle_framework_bm` | true | 分布式 Bundle 功能 |

**证据来源**: `bundletool.gni:24-28`

---

## 相关文档

- [02_Command_Reference.md](./02_Command_Reference.md) - 命令详解
- [01_Architecture.md](./01_Architecture.md) - 架构设计
- [03_Inner_API.md](./03_Inner_API.md) - 内部接口
