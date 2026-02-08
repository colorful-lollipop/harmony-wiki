# 文档导航

## 核心文档

| 文件 | 说明 | 新人必读 |
|------|------|----------|
| [README.md](./README.md) | 文档说明、更新方式、生成信息 | ✓ |
| [00_Overview.md](./00_Overview.md) | 项目定位、目录结构、核心能力 | ✓ |
| [01_N-API_Reference.md](./01_N-API_Reference.md) | 对外接口清单、API 详解 | ✓ |
| [02_Architecture.md](./02_Architecture.md) | 内部架构、模块职责、依赖关系 | ✓ |
| [03_Build_and_Targets.md](./03_Build_and_Targets.md) | GN 构建配置、编译产物 | ✓ |
| [04_Security_Review.md](./04_Security_Review.md) | 安全风险评审、威胁模型 | ✓ |

## 附录

| 文件 | 说明 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链图示 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 关键宏与 Feature Flags |

## 新人阅读路线

### 路线一：应用开发者（仅使用 N-API）

1. `README.md` → 了解文档结构
2. `00_Overview.md` → 理解模块边界
3. `01_N-API_Reference.md` → 掌握 API 用法

### 路线二：系统开发者（参与模块开发）

1. `README.md` → 了解文档结构
2. `00_Overview.md` → 理解模块边界
3. `02_Architecture.md` → 深入架构设计
4. `03_Build_and_Targets.md` → 掌握构建配置

### 路线三：安全评审

1. `README.md` → 了解文档范围
2. `04_Security_Review.md` → 查看安全风险清单

## 术语表

| 术语 | 说明 |
|------|------|
| **N-API** | OpenHarmony Native API，C 语言接口规范 |
| **Inner API** | 内部模块间接口，不对外暴露 |
| **EFilter** | Effect Filter，图像效果滤镜基类 |
| **Pipeline** | 处理流水线，串联多个滤镜 |
| **EffectBuffer** | 效果处理缓冲区，封装图像数据 |
| **NativeBuffer** | 原生缓冲区，用于 GPU 内存共享 |

## 快速索引

### 按功能索引

| 功能 | 文档位置 |
|------|----------|
| 创建图像效果 | `01_N-API_Reference.md` |
| 添加滤镜 | `01_N-API_Reference.md` |
| 色彩空间处理 | `02_Architecture.md` |
| 自定义滤镜 | `02_Architecture.md` |
| 编译构建 | `03_Build_and_Targets.md` |
| 安全检查 | `04_Security_Review.md` |

### 按代码位置索引

| 模块 | 代码位置 | 文档位置 |
|------|----------|----------|
| C API 实现 | `frameworks/native/capi/` | `01_N-API_Reference.md` |
| 核心引擎 | `frameworks/native/effect/` | `02_Architecture.md` |
| 滤镜实现 | `frameworks/native/efilter/` | `02_Architecture.md` |
| 内部接口 | `interfaces/inner_api/native/` | `02_Architecture.md` |
| 对外接口 | `interfaces/kits/native/` | `01_N-API_Reference.md` |
