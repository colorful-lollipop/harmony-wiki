# Linux 内核 6.6 Wiki - 全站导航

## 阅读路线

### 快速开始（推荐新人）

1. [00_Overview.md](00_Overview.md) - 了解项目定位与核心概念（10 分钟）
2. [01_Directory_Structure.md](01_Directory_Structure.md) - 熟悉代码布局（15 分钟）
3. [02_Architecture.md](02_Architecture.md) - 理解子系统交互（30 分钟）

### 驱动开发者

1. [01_Directory_Structure.md](01_Directory_Structure.md) - 了解目录与模块职责
2. [02_Architecture.md](02_Architecture.md) - 理解驱动模型
3. [04_Internal_APIs.md](04_Internal_APIs.md) - 学习内部 API 接口
4. [05_Build_System.md](05_Build_System.md) - 掌握构建方法

### 内核开发者

1. [03_System_Calls.md](03_System_Calls.md) - 了解系统调用接口
2. [04_Internal_APIs.md](04_Internal_APIs.md) - 掌握子系统 API
3. [02_Architecture.md](02_Architecture.md) - 理解核心架构
4. [05_Build_System.md](05_Build_System.md) - 学习构建系统

### 安全审计人员

1. [07_Security_Review.md](07_Security_Review.md) - 安全风险综述
2. [03_System_Calls.md](03_System_Calls.md) - 分析攻击面
3. [02_Architecture.md](02_Architecture.md) - 理解数据流与信任边界
4. [08_Troubleshooting.md](08_Troubleshooting.md) - 了解安全调试方法

---

## 核心文档

### 概览与基础

| 文档 | 说明 | 预计阅读时间 |
|------|------|-------------|
| [README](README.md) | Wiki 说明与更新指南 | 5 分钟 |
| [00_Overview.md](00_Overview.md) | 项目定位、核心概念、边界 | 10 分钟 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责 | 15 分钟 |

### 架构与接口

| 文档 | 说明 | 预计阅读时间 |
|------|------|-------------|
| [02_Architecture.md](02_Architecture.md) | 组件、数据流、线程模型 | 30 分钟 |
| [03_System_Calls.md](03_System_Calls.md) | 系统调用接口（对外 API） | 20 分钟 |
| [04_Internal_APIs.md](04_Internal_APIs.md) | 子系统接口（内部 API） | 25 分钟 |

### 构建与部署

| 文档 | 说明 | 预计阅读时间 |
|------|------|-------------|
| [05_Build_System.md](05_Build_System.md) | Kbuild/Kconfig 构建系统 | 20 分钟 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物与加载关系 | 15 分钟 |

### 安全与调试

| 文档 | 说明 | 预计阅读时间 |
|------|------|-------------|
| [07_Security_Review.md](07_Security_Review.md) | 安全风险评审与可利用点 | 30 分钟 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 常见问题与定位路径 | 20 分钟 |

---

## 附录

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链图 | 可选 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键配置选项参考 | 可选 |

---

## 按主题浏览

### 子系统导航

- **进程管理**: [01_Directory_Structure.md](01_Directory_Structure.md#kernel) → [02_Architecture.md](02_Architecture.md#进程调度)
- **内存管理**: [01_Directory_Structure.md](01_Directory_Structure.md#mm) → [02_Architecture.md](02_Architecture.md#内存管理)
- **文件系统**: [01_Directory_Structure.md](01_Directory_Structure.md#fs) → [03_System_Calls.md](03_System_Calls.md#文件系统调用)
- **网络协议栈**: [01_Directory_Structure.md](01_Directory_Structure.md#net) → [02_Architecture.md](02_Architecture.md#网络子系统)
- **驱动模型**: [01_Directory_Structure.md](01_Directory_Structure.md#drivers) → [02_Architecture.md](02_Architecture.md#驱动模型)
- **安全框架**: [01_Directory_Structure.md](01_Directory_Structure.md#security) → [07_Security_Review.md](07_Security_Review.md)

### OpenHarmony 特定

- **EPFS**: [01_Directory_Structure.md](01_Directory_Structure.md#openharmony-特定组件)
- **HMDFS**: [01_Directory_Structure.md](01_Directory_Structure.md#openharmony-特定组件)
- **Hyperhold**: [01_Directory_Structure.md](01_Directory_Structure.md#openharmony-特定组件)
- **Vendor Hooks**: [01_Directory_Structure.md](01_Directory_Structure.md#openharmony-特定组件)

---

## 文档版本

- **v1.0** (2026-02-06): 初始版本，覆盖 Linux 6.6 核心内容

---

**最后更新**: 2026-02-06
