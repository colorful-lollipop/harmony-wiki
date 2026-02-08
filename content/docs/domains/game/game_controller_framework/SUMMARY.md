# SUMMARY - 文档导航

## 新人阅读顺序（推荐）

1. **[00_Overview.md](./00_Overview.md)** - 项目概览与快速入门
2. **[01_Project_Position.md](./01_Project_Position.md)** - 项目定位、边界与核心能力
3. **[02_Directory_Structure.md](./02_Directory_Structure.md)** - 目录结构与模块职责
4. **[03_Architecture.md](./03_Architecture.md)** - 系统架构（组件、数据流、线程模型）
5. **[04_External_CAPI.md](./04_External_CAPI.md)** - 对外 CAPI 接口详解（重点）
6. **[05_Inner_API.md](./05_Inner_API.md)** - 内部 API 与模块接口
7. **[06_GN_Targets.md](./06_GN_Targets.md)** - GN 构建目标与依赖关系
8. **[07_Build_Artifacts.md](./07_Build_Artifacts.md)** - 编译产物与安装路径
9. **[08_Security_Review.md](./08_Security_Review.md)** - 安全风险评审（必读）
10. **[09_Troubleshooting.md](./09_Troubleshooting.md)** - 常见问题与定位路径

## 附录文档（按需阅读）

- **[appendix/Callgraphs.md](./appendix/Callgraphs.md)** - 关键调用链
- **[appendix/Config_Flags.md](./appendix/Config_Flags.md)** - 关键配置与宏定义

## 文档结构说明

### 核心文档

| 文档 | 内容 | 目标读者 |
|------|------|---------|
| 00_Overview.md | 项目概述、快速入门 | 所有人 |
| 01_Project_Position.md | 项目定位、边界、关键概念 | 新人、架构师 |
| 02_Directory_Structure.md | 目录树、模块职责 | 开发者 |
| 03_Architecture.md | 架构图、数据流、时序 | 架构师、高级开发者 |
| 04_External_CAPI.md | **CAPI 清单表、调用链** | **开发者（重点）** |
| 05_Inner_API.md | 内部接口、依赖方向 | 架构师、维护者 |
| 06_GN_Targets.md | GN target、依赖关系 | 构建工程师 |
| 07_Build_Artifacts.md | 编译产物、安装路径 | 部署人员 |
| 08_Security_Review.md | **安全风险、利用点** | **所有人（必读）** |
| 09_Troubleshooting.md | 问题定位、解决方案 | 运维、开发者 |

### 附录文档

| 文档 | 内容 | 适用场景 |
|------|------|---------|
| Callgraphs.md | 关键 API 调用链图 | 调试、性能优化 |
| Config_Flags.md | 配置宏、编译开关 | 定制化构建 |

## 关键术语索引

| 术语 | 说明 | 参考文档 |
|------|------|---------|
| **CAPI** | C Native API，当前版本唯一的对外接口类型 | 04_External_CAPI.md |
| **GameControllerSA** | 系统服务（System Ability）进程 | 03_Architecture.md |
| **InnerAPI** | 内部 API，供终端厂商使用 | 05_Inner_API.md |
| **Input-to-Touch** | 输入转触控特性 | 01_Project_Position.md |
| **KeyMapping** | 按键映射配置 | 02_Directory_Structure.md |
| **GameDevice** | 设备管理模块 | 04_External_CAPI.md |
| **GamePad** | 手柄输入模块 | 04_External_CAPI.md |

## 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本 |

---

**提示**: 本文档严格基于代码证据，所有链接均指向有效文档。如发现链接失效，请更新本文档。
