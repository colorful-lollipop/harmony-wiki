# VPE 视频处理引擎 Wiki 导航

本文档为 VPE 视频处理引擎 Wiki 的全局导航页面，提供所有文档的索引和新人阅读路线。

---

## 文档列表

| 文档 | 说明 | 阅读优先级 |
|------|------|----------|
| [README](./README.md) | Wiki 使用指南、覆盖范围、更新方式 | ⭐ 必读 |
| [index](./index.md) | 项目概览、核心能力、定位与边界 | ⭐ 必读 |
| [Architecture](./Architecture.md) | 组件图、数据流、线程模型、时序图 | ⭐ 必读 |
| [NAPI_Reference](./NAPI_Reference.md) | N-API、C API 接口清单与使用示例 | 📖 查阅 |
| [Inner_API](./Inner_API.md) | Inner API 模块接口与依赖关系 | 🔧 开发 |
| [Build_System](./Build_System.md) | GN Targets、构建配置、依赖关系 | 🔧 构建 |
| [Artifacts](./Artifacts.md) | 编译产物、安装路径、运行时加载 | 📦 部署 |
| [Security_Review](./Security_Review.md) | 安全风险、攻击面、修复建议 | 🛡️ 安全 |
| [Troubleshooting](./Troubleshooting.md) | 常见问题、调试方法、定位路径 | 🔧 排错 |

---

## 新人阅读路线

### 路线一：应用开发者

> 目标：快速上手使用 VPE 的 API 进行视频/图像处理开发

```mermaid
graph LR
    A[README] --> B[index]
    B --> C[NAPI_Reference]
    C --> D[开始开发]
```

| 步骤 | 内容 | 预计时间 |
|------|------|---------|
| 1 | 阅读 README 了解 Wiki 结构 | 2 分钟 |
| 2 | 阅读 index 理解项目定位 | 5 分钟 |
| 3 | 阅读 NAPI_Reference 查找所需 API | 10 分钟 |
| 4 | 参考示例代码开始开发 | - |

### 路线二：系统开发者

> 目标：理解 VPE 内部架构，进行深度定制或问题定位

```mermaid
graph LR
    A[README] --> B[index]
    B --> C[Architecture]
    C --> D[Inner_API]
    D --> E[Build_System]
    E --> F[Artifacts]
```

| 步骤 | 内容 | 预计时间 |
|------|------|---------|
| 1 | 阅读 README 了解 Wiki 结构 | 2 分钟 |
| 2 | 阅读 index 理解项目定位 | 5 分钟 |
| 3 | 阅读 Architecture 理解整体架构 | 15 分钟 |
| 4 | 阅读 Inner API 了解模块接口 | 20 分钟 |
| 5 | 阅读 Build System 了解构建配置 | 10 分钟 |
| 6 | 阅读 Artifacts 了解产物加载 | 10 分钟 |

### 路线三：安全审计

> 目标：评估 VPE 的安全风险，识别潜在攻击面

```mermaid
graph LR
    A[README] --> B[index]
    B --> C[Architecture]
    C --> D[Security_Review]
```

| 步骤 | 内容 | 预计时间 |
|------|------|---------|
| 1 | 阅读 README 了解 Wiki 结构 | 2 分钟 |
| 2 | 阅读 index 理解项目定位 | 5 分钟 |
| 3 | 阅读 Architecture 理解数据流 | 15 分钟 |
| 4 | 阅读 Security Review 进行安全评估 | 30 分钟 |

### 路线四：构建工程师

> 目标：理解编译配置，进行定制化构建

```mermaid
graph LR
    A[README] --> B[index]
    B --> C[Build_System]
    C --> D[Artifacts]
```

| 步骤 | 内容 | 预计时间 |
|------|------|---------|
| 1 | 阅读 README 了解 Wiki 结构 | 2 分钟 |
| 2 | 阅读 index 理解项目定位 | 5 分钟 |
| 3 | 阅读 Build System 了解 Targets | 20 分钟 |
| 4 | 阅读 Artifacts 了解产物 | 10 分钟 |

---

## 模块索引

### 按功能分类

| 功能模块 | 相关文档 |
|---------|---------|
| 项目定位与边界 | [index](./index.md) |
| 架构设计 | [Architecture](./Architecture.md) |
| JS/TS API | [NAPI_Reference](./NAPI_Reference.md) |
| C API | [NAPI_Reference](./NAPI_Reference.md) |
| Inner API | [Inner_API](./Inner_API.md) |
| 构建配置 | [Build_System](./Build_System.md) |
| 产物部署 | [Artifacts](./Artifacts.md) |
| 安全评估 | [Security_Review](./Security_Review.md) |
| 问题排查 | [Troubleshooting](./Troubleshooting.md) |

### 按代码路径分类

| 代码路径 | 相关文档 |
|---------|---------|
| `interfaces/kits/` | [NAPI_Reference](./NAPI_Reference.md) |
| `interfaces/inner_api/` | [Inner_API](./Inner_API.md) |
| `framework/algorithm/` | [Architecture](./Architecture.md), [Inner_API](./Inner_API.md) |
| `framework/capi/` | [NAPI_Reference](./NAPI_Reference.md) |
| `services/` | [Architecture](./Architecture.md), [Build_System](./Build_System.md) |
| `BUILD.gn` | [Build_System](./Build_System.md) |

---

## 快速跳转

### API 快速查找

| API 类型 | 跳转链接 |
|---------|---------|
| JS/TS N-API | [NAPI_Reference.md](./NAPI_Reference.md#js-ts-n-api) |
| C API (图像) | [NAPI_Reference.md](./NAPI_Reference.md#图像处理-c-api) |
| C API (视频) | [NAPI_Reference.md](./NAPI_Reference.md#视频处理-c-api) |
| Inner API | [Inner_API.md](./Inner_API.md) |

### 构建与部署

| 需求 | 跳转链接 |
|------|---------|
| GN Targets | [Build_System.md](./Build_System.md) |
| 编译产物 | [Artifacts.md](./Artifacts.md) |
| 安装路径 | [Artifacts.md](./Artifacts.md#安装路径) |

### 安全与调试

| 需求 | 跳转链接 |
|------|---------|
| 安全风险 | [Security_Review.md](./Security_Review.md) |
| 常见问题 | [Troubleshooting.md](./Troubleshooting.md) |
| 调试方法 | [Troubleshooting.md](./Troubleshooting.md#调试方法) |

---

## 术语表

| 术语 | 全称 | 说明 |
|------|------|------|
| VPE | Video Processing Engine | 视频处理引擎 |
| N-API | Native API | OpenHarmony 原生 API |
| CAPI | C API | C 语言接口 |
| SA | System Ability | 系统能力 |
| GN | Generate Ninja | 构建系统 |
| NDK | Native Development Kit | 原生开发套件 |

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本 |
