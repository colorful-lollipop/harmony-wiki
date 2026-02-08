# T2Stack Wiki 导航

## 双路线阅读指南

### 路线 A：新人学习路线

> **目标**：快速理解项目，能上手开发

```
1. 概览 → 2. 架构 → 3. 模块 → 4. 接口 → 5. 构建 → 6. 安全
```

| 阶段 | 推荐文档 | 预计时间 | 目的 |
|------|----------|----------|------|
| **入门** | [00_Overview.md](./00_Overview.md) | 10 分钟 | 了解项目定位和核心能力 |
| **进阶** | [01_Architecture.md](./01_Architecture.md) | 15 分钟 | 理解系统架构和数据流 |
| **实操** | [02_Module_Structure.md](./02_Module_Structure.md) | 15 分钟 | 掌握模块划分和职责 |
| **开发** | [03_CAPI_Reference.md](./03_CAPI_Reference.md) | 30 分钟 | 学习 API 接口使用 |
| **构建** | [05_Build_System.md](./05_Build_System.md) | 15 分钟 | 理解构建配置和产物 |
| **安全** | [07_Security_Review.md](./07_Security_Review.md) | 20 分钟 | 了解安全风险和防护 |

### 路线 B：安全研究路线

> **目标**：快速识别攻击面，定位漏洞点

```
1. 概览 → 2. 攻击面 → 3. 代码地图 → 4. 接口入口 → 5. 风险点
```

| 阶段 | 推荐文档 | 预计时间 | 目的 |
|------|----------|----------|------|
| **快速了解** | [00_Overview.md](./00_Overview.md) | 5 分钟 | 了解项目定位和核心能力 |
| **攻击面分析** | [07_Security_Review.md](./07_Security_Review.md) | 20 分钟 | 识别所有外部输入入口 |
| **代码地图** | [02_Module_Structure.md](./02_Module_Structure.md) | 10 分钟 | 快速定位关键代码位置 |
| **接口清单** | [03_CAPI_Reference.md](./03_CAPI_Reference.md) | 15 分钟 | 了解 API 调用链 |
| **内部实现** | [04_Inner_API.md](./04_Inner_API.md) | 20 分钟 | 理解内部处理逻辑 |

## 完整文档列表

### 核心文档

| 文档 | 说明 |
|------|------|
| [README.md](./README.md) | 本文档，覆盖范围、更新方式、术语表 |
| [00_Overview.md](./00_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |
| [01_Architecture.md](./01_Architecture.md) | 组件图、数据流、线程模型、关键时序图 |
| [02_Module_Structure.md](./02_Module_Structure.md) | 目录结构、模块职责、依赖关系 |
| [03_CAPI_Reference.md](./03_CAPI_Reference.md) | C API 接口清单、参数、返回值、错误码 |
| [04_Inner_API.md](./04_Inner_API.md) | 模块间内部接口、稳定性标注 |
| [05_Build_System.md](./05_Build_System.md) | GN Targets、Feature Flags、编译配置 |
| [06_Runtime_Artifacts.md](./06_Runtime_Artifacts.md) | 编译产物清单、安装路径、加载关系 |
| [07_Security_Review.md](./07_Security_Review.md) | 攻击面分析、风险点、修复建议 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 常见问题、定位方法、日志路径 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 关键宏定义、Feature Flags |

## 模块速查

| 模块 | 主要文件 | 核心功能 |
|------|----------|----------|
| **fillp** | `fillp/include/fillpinc.h` | 流传输（音视频） |
| **nstackx_core/dfile** | `nstackx_core/dfile/interface/nstackx_dfile.h` | 文件传输 |
| **nstackx_ctrl** | `nstackx_ctrl/interface/nstackx.h` | 设备发现 |
| **nstackx_util** | `nstackx_util/interface/nstackx_error.h` | 公共基础模块 |
| **nstackx_congestion** | `nstackx_congestion/interface/` | 拥塞控制算法 |

## 快速跳转

### API 接口
- [Fillp 流传输 API](./03_CAPI_Reference.md#1-fillp-流传输接口)
- [DFile 文件传输 API](./03_CAPI_Reference.md#2-dfile-文件传输接口)
- [NStackX 设备发现 API](./03_CAPI_Reference.md#3-nstackx-设备发现接口)

### 构建配置
- [Feature Flags](./05_Build_System.md#2-feature-flags)
- [模块 Targets](./05_Build_System.md#3-关键-targets-列表)
- [依赖关系](./05_Build_System.md#4-依赖关系)

### 安全相关
- [攻击面清单](./07_Security_Review.md#1-攻击面清单)
- [风险点列表](./07_Security_Review.md#2-可利用风险点)
- [修复建议](./07_Security_Review.md#3-修复建议)

---

*最后更新：2026-02-06*
