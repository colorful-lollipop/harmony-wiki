# 项目概述

## 目的

本文档介绍 OpenHarmony 资源管理组件 (resource_management) 的基本概念、定位、边界、核心能力和运行环境。

## 适用范围

- OpenHarmony 4.0+ 系统
- 标准 (standard) 系统类型
- JavaScript/ArkTS 应用开发
- C/C++ 原生开发

## 关键结论

| 关键词 | 说明 |
|--------|------|
| **组件名称** | @ohos/resource_management (global_resmgr_standard) |
| **子系统** | global (全球化子系统) |
| **系统能力** | SystemCapability.Global.ResourceManager |
| **核心功能** | 根据 configuration (语言、区域、横竖屏、mccmnc) 和 device capability (设备类型、分辨率) 获取应用资源信息 |
| **多语言支持** | JavaScript, C/C++, ArkTS, Cangjie |
| **无 IPC** | 纯本地组件，不使用 IPC/ServiceAbility |

## 项目定位

资源管理组件是 OpenHarmony 全球化子系统中的核心组件，提供应用资源（字符串、图片、主题、原始文件等）的加载、查询和管理能力。

### 核心价值

1. **多语言支持**: 根据用户的语言和区域设置自动选择合适的资源
2. **设备适配**: 根据设备类型、屏幕密度、横竖屏等自动匹配资源
3. **系统资源管理**: 提供系统级资源管理能力，支持系统资源加载
4. **多语言 API**: 支持 JavaScript, C/C++, ArkTS, Cangjie 等多种语言接口

### 应用场景

- 国际化应用开发（多语言、多地区支持）
- 主题切换（深色/浅色模式）
- 设备适配（不同分辨率、屏幕密度）
- 原始文件访问（rawfile）
- 系统资源访问

## 边界

### 组件边界

| 包含 | 不包含 |
|------|--------|
| 资源解析和加载 | 应用运行时环境 |
| 资源查询和匹配 | 应用界面渲染 |
| 多语言资源管理 | 业务逻辑处理 |
| 系统资源管理 | 其他系统服务 |

### 不涉及的功能

- 应用生命周期管理
- 界面渲染和布局
- 跨进程通信（IPC/ServiceAbility）
- 权限验证（依赖系统沙箱机制）
- 网络通信
- 数据持久化（资源文件加载除外）

## 核心能力

### 1. 资源加载

从 HAP (Harmony Ability Package) 包中加载资源，支持：
- 字符串资源 (`getString`)
- 媒体资源 (`getMedia`)
- 颜色资源 (`getColor`)
- 原始文件 (`getRawFile`)
- 原始文件描述符 (`getRawFileDescriptor`)
- 主题包资源

### 2. 资源匹配

根据以下条件自动匹配最佳资源：
- 语言 (locale)
- 区域 (region)
- 方向 (direction: VERTICAL/HORIZONTAL)
- 设备类型 (device type)
- 屏幕密度 (screen density)
- 颜色模式 (color mode: DARK/LIGHT)
- MCC/MNC (移动国家/网络代码)

### 3. 资源管理

- 资源缓存管理
- 资源覆盖 (`addResource`, `removeResource`)
- 配置更新 (`updateOverrideConfiguration`)
- 系统资源管理器与普通应用资源管理器区分

### 4. 多语言 API

- **JavaScript/N-API**: 65 个导出方法
- **Native API**: C 语言接口
- **Inner API**: C++ 内部子系统接口
- **ArkTS/ANI**: ArkTS 接口
- **Cangjie/FFI**: Cangjie 语言接口

## 运行环境

### 硬件要求

| 资源 | 需求 |
|------|------|
| ROM | ~600KB |
| RAM | ~2500KB |
| CPU | 标准 ARM 架构 |
| 存储 | 需要存储 HAP 资源包 |

### 软件依赖

| 组件 | 版本 | 用途 |
|------|------|------|
| napi | - | N-API 框架 |
| hilog | - | 日志系统 |
| hisysevent | - | 系统事件 |
| hitrace | - | 性能追踪 |
| icu | 可选 | 国际化支持 |
| zlib | - | 压缩解压 |
| cJSON | - | JSON 解析 |
| bundle_framework | - | Bundle 框架 |
| ability_base | - | 能力基础库 |
| ace_engine | - | ArkUI 引擎 |

### 平台支持

| 平台 | Target | 支持状态 |
|------|--------|----------|
| OpenHarmony | `global_resmgr` | ✅ |
| Windows | `global_resmgr_win` / `win_resmgr` | ✅ (IDE 预览) |
| macOS | `global_resmgr_mac` / `mac_resmgr` | ✅ (IDE 预览) |
| Linux | `global_resmgr_linux` / `linux_resmgr` | ✅ (IDE 预览) |

## 关键概念

### ResourceManager

资源管理器是资源管理组件的核心类，提供资源的加载、查询和管理功能。

**类型**:
- **应用资源管理器**: 加载和管理应用自身资源
- **系统资源管理器**: 加载和管理系统级资源

### Configuration

资源配置，描述当前的语言、设备类型、屏幕密度等信息。

**包含**:
- 语言 (locale)
- 方向 (direction)
- 设备类型 (device type)
- 屏幕密度 (screen density)
- 颜色模式 (color mode)
- MCC/MNC

### HAP

Harmony Ability Package，OpenHarmony 应用的打包格式，包含代码和资源。

**结构**:
```
MyApp.hap
├── resources
│   ├── base
│   │   ├── element (字符串、颜色等)
│   │   ├── media (图片、音频等)
│   │   └── profile (配置文件)
│   └── zh_CN
│       ├── element
│       └── media
└── ...
```

### RawFile

原始文件，未编译的资源文件，可直接按路径访问。

### 主题包 (Theme Pack)

主题资源包，包含可更换的主题资源，支持动态加载和卸载。

### 沙箱路径

系统资源的沙箱隔离路径，用于 SystemAbility 进程。

- 沙箱路径: `/data/storage/el1/bundle/ohos.global.systemres...`
- 非沙箱路径: `/system/app/ohos.global.systemres/SystemResources.hap`

## 架构特点

### 本地组件

资源管理组件是纯本地组件，不使用 IPC/ServiceAbility，所有操作都在调用进程内完成。

### 多语言绑定

提供 5 种语言的 API 绑定，满足不同开发语言的需求：

```
应用层
    ↓
N-API / ANI / FFI (多语言绑定)
    ↓
核心框架层 (global_resmgr)
```

### 安全模型

- 依赖系统沙箱机制进行隔离
- 通过路径隔离实现系统资源和应用资源分离
- 无显式权限检查，依赖底层隔离

## 相关文档

- [目录结构](02_DirectoryStructure.md) - 代码组织和模块职责
- [架构设计](03_Architecture.md) - 组件架构和数据流
- [N-API 接口](04_NAPI.md) - JavaScript API 详细文档

---

**生成时间**: 2026-02-06
**证据来源**: bundle.json, README_zh.md, frameworks/resmgr/
