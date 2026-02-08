# 项目概览

## 项目定位

**Cangjie API Public Repository** (`@interface/sdk_cangjie`) 是 OpenHarmony Cangjie SDK 的核心仓库，用于：

1. **存储和管理** 各 Cangjie 子系统的公共 API 声明
2. **提供 SDK 构建工具**，支持 Cangjie API 在 OpenHarmony SDK 中的构建

> **证据**: `README.md` 第 1-5 行

### 核心定位

| 维度 | 说明 |
|------|------|
| **产品定位** | OpenHarmony 原生应用开发 SDK |
| **语言** | Cangjie（仓颉） |
| **目标设备** | OpenHarmony 标准设备（standard） |
| **开发平台** | Windows / Linux / Mac-x64 / Mac-arm64 |
| **能力等级** | Beta（测试阶段） |

## 核心能力

### 1. API 声明管理

- **25+ Kit**: 提供完整的应用开发 API 覆盖
- **221+ API 模块**: 包含类型定义、函数、类、枚举等
- **版本管理**: 通过 `@!APILevel` 注解管理 API 版本

> **证据**: `api/` 目录包含 25 个 Kit 目录，共 221+ 个 `.cj.d` 文件

### 2. SDK 构建能力

| 工具 | 用途 |
|------|------|
| **cjo 生成工具链** | 将接口声明序列化为 cjo 格式 |
| **API mock 库生成** | 生成空实现动态库用于 SDK 构建 |
| **头文件复制** | 复制 Cangjie API 头文件到 SDK |

> **证据**: `build-tools/` 目录结构

### 3. Kit 聚合能力

- **统一导入**: 通过 `kit.*.cj.d` 文件聚合相关模块
- **23 个 Kit**: 按功能组织，提供统一的导入点

> **证据**: `kits/` 目录包含 23 个 `kit.*.cj.d` 文件

## 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony SDK                          │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │     js       │  │     ets      │  │   cangjie    │      │
│  │   (JS API)   │  │  (ArkTS)    │  │  (Cangjie)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                          │                                    │
│                    ┌─────┴─────┐                            │
│                    │  interface │                            │
│                    │  sdk_*     │                            │
│                    └─────┬─────┘                            │
│                          │                                  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              interface/sdk_cangjie                      │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐               │  │
│  │  │   api/  │  │  kits/  │  │build-   │               │  │
│  │  │ Kit API │  │ Kit decl│  │ tools   │               │  │
│  │  └─────────┘  └─────────┘  └─────────┘               │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
         │                    │                    │
    ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
    │wrapper  │          │wrapper  │          │wrapper  │
    │repos    │          │repos    │          │repos    │
    └─────────┘          └─────────┘          └─────────┘
```

> **证据**: `README.md` 图 1 和目录结构说明（第 35-52 行）

## 依赖关系

### 内部依赖

| 组件 | 类型 | 用途 |
|------|------|------|
| `api/` | 声明文件 | Kit API 接口声明 |
| `kits/` | 声明文件 | Kit 统一外部声明 |
| `build-tools/` | 工具 | SDK 构建工具链 |

### 外部依赖

| 组件 | 仓库 | 用途 |
|------|------|------|
| cangjie_ark_interop | arkcompiler_cangjie_ark_interop | Cangjie-ArkTS 互操作接口 |
| arkui_cangjie_wrapper | arkui_arkui_cangjie_wrapper | ArkUI Cangjie 接口 |
| flatbuffers | third_party_flatbuffers | 序列化工具 |

> **证据**: `bundle.json` 第 27-31 行

## 运行环境

### 开发环境要求

| 平台 | 支持状态 | 说明 |
|------|---------|------|
| Windows | ✅ 支持 | MinGW 工具链 |
| Linux | ✅ 支持 | clang 工具链 |
| Mac-x64 | ✅ 支持 | clang 工具链 |
| Mac-arm64 | ✅ 支持 | clang 工具链 |

### 运行时要求

| 组件 | 版本/要求 |
|------|----------|
| 目标设备 | OpenHarmony 标准设备 |
| API Level | 22+ |

## 关键概念

### cjo 格式

cjo 是 Cangjie API 的序列化格式，用于存储 API 声明的二进制表示。

- **生成**: 通过 flatc 工具和 FlatBuffers Schema 序列化 `.cj.d` 文件
- **用途**: SDK 构建、分发、运行时加载

> **证据**: `README.md` 第 20-25 行

### Kit

Kit 是 Cangjie API 的组织单元，将相关功能的模块聚合为一个导入目标。

- **命名**: `kit.{KitName}`（如 `kit.NetworkKit`）
- **声明**: `kits/kit.{KitName}.cj.d`
- **导入**: `import kit.NetworkKit`

### @!APILevel 注解

用于标注 API 的版本信息、能力要求和权限声明。

```cangjie
@!APILevel[
    since: "22",                                   // API 起始版本
    syscap: "SystemCapability.Security.AccessToken", // 系统能力要求
    permission: "ohos.permission.CAMERA"            // 权限要求
]
public func getCamera(): Camera
```

## 约束与限制

| 约束 | 说明 |
|------|------|
| **平台限制** | 目前仅支持标准设备（standard） |
| **构建限制** | 支持交叉编译，不支持在 ohos 平台上构建应用 |
| **Beta 状态** | Cangjie API 处于 Beta 阶段 |

> **证据**: `README.md` 第 8 行、第 136-137 行

## 相关文档

| 文档 | 位置 | 说明 |
|------|------|------|
| 构建指南 | `docs/cangjie_sdk_build_guide.md` | SDK 集成构建说明 |
| cjo 序列化指南 | `docs/cangjie_cjo_serialization_and_deserialization_guide.md` | cjo 格式详解 |
| API 文档 | `04_Kit_API.md` | 详细 API 清单 |
| 构建文档 | `06_GN_Build.md` | GN Targets 说明 |
