# 文档导航

## 项目首页

| 文档 | 描述 |
|------|------|
| [index.md](index.md) | 项目首页，包含快速入门和关键信息 |

## 核心文档

| 文档 | 描述 | 阅读优先级 |
|------|------|-----------|
| [01_Overview.md](01_Overview.md) | 项目概览：定位、目标、核心能力 | ⭐⭐⭐ 必读 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构与模块职责 | ⭐⭐⭐ 必读 |
| [03_Architecture.md](03_Architecture.md) | 架构说明：组件图、数据流、线程模型 | ⭐⭐ 推荐 |
| [04_External_API.md](04_External_API.md) | 对外 API：N-API、JS API、参数校验 | ⭐⭐⭐ 必读 |

## 技术细节

| 文档 | 描述 |
|------|------|
| [05_Internal_API.md](05_Internal_API.md) | 内部 API：模块接口、依赖方向 |
| [06_GN_Build.md](06_GN_Build.md) | GN 构建系统：targets、依赖、配置 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物：.so/.a、输出路径 |
| [08_Security_Review.md](08_Security_Review.md) | 安全风险评审：攻击面、风险点 |

## 运维支持

| 文档 | 描述 |
|------|------|
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题：构建、运行、调试 |

---

## 新人阅读路线

### 路线1：API 使用者

```
index.md → 01_Overview.md → 04_External_API.md
```
快速了解项目能力，直接查阅 API 使用方法。

### 路线2：代码贡献者

```
index.md → 01_Overview.md → 02_Directory_Structure.md → 03_Architecture.md
        → 05_Internal_API.md → 06_GN_Build.md
```
全面理解项目架构，为代码贡献做准备。

### 路线3：安全审计

```
index.md → 08_Security_Review.md → 03_Architecture.md
```
重点关注安全风险和架构设计。

---

## 快速跳转

### 按功能模块

| 模块 | 关键文件 | 相关文档 |
|------|---------|---------|
| HiLog | `ohos/hilog/hilog.cj` | [External_API.md](04_External_API.md#hilog-api) |
| HiAppEvent | `ohos/hiviewdfx/hi_app_event/*.cj` | [External_API.md](04_External_API.md#hiappevent-api) |
| HiTraceMeter | `ohos/hi_trace_meter/hi_trace_meter.cj` | [External_API.md](04_External_API.md#hitracemeter-api) |
| PerformanceAnalysisKit | `kit/PerformanceAnalysisKit/index.cj` | [External_API.md](04_External_API.md#performanceanalysiskit) |

### 按任务类型

| 任务 | 查阅文档 |
|------|---------|
| 查找 API 使用方法 | [04_External_API.md](04_External_API.md) |
| 理解代码结构 | [02_Directory_Structure.md](02_Directory_Structure.md) |
| 构建项目 | [06_GN_Build.md](06_GN_Build.md) |
| 定位问题 | [09_Troubleshooting.md](09_Troubleshooting.md) |
| 安全审计 | [08_Security_Review.md](08_Security_Review.md) |

---

## 文档约定

### 术语说明

| 术语 | 说明 |
|------|------|
| DFX | Design for X，软件设计质量属性 |
| N-API | Native API，本地接口 |
| FFI | Foreign Function Interface，外部函数接口 |
| syscap | System Capability，系统能力标识 |

### 代码证据标注

本文档中的关键结论均标注了代码证据：
- `{文件路径}:{行号}` - 代码位置
- `符号名` - 函数/类/宏定义
- `代码片段` - 相关实现

### 忽略的测试内容

本文档**不包含**测试相关内容：
- `test/` 目录
- `*_test.*` 测试文件
- `unittest/` 单元测试
- `fuzz/` 模糊测试

---

## 版本信息

| 项目 | 版本 |
|------|------|
| 项目版本 | 6.1 |
| API Level | 22 |
| 文档版本 | 1.0.0 |
| 最后更新 | 2026-02-06 |
