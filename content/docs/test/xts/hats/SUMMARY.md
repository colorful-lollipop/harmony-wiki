# 文档导航

本文档提供 HATS Wiki 的完整导航结构，按新人阅读顺序编排。

---

## 推荐阅读顺序

### 第 1 阶段：快速入门

1. **[README.md](./README.md)** - 文档使用指南
2. **[00_Overview.md](./00_Overview.md)** - 项目定位、核心能力、运行环境

### 第 2 阶段：架构理解

3. **[01_Architecture.md](./01_Architecture.md)** - 整体架构、组件图、数据流
4. **[02_Modules.md](./02_Modules.md)** - 各子系统模块详解

### 第 3 阶段：接口与构建

5. **[03_N-API.md](./03_N-API.md)** - N-API 接口说明（HAL 层测试）
6. **[04_Build.md](./04_Build.md)** - GN 目标与编译产物
7. **[05_Security.md](./05_Security.md)** - 安全风险评审

### 第 4 阶段：参考

8. **[06_Appendix.md](./06_Appendix.md)** - 调用链、配置参数

---

## 完整目录

### 根文档

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README.md](./README.md) | 文档使用指南 | 必需 |
| [SUMMARY.md](./SUMMARY.md) | 本导航文件 | 必需 |

### 概览与架构

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [00_Overview.md](./00_Overview.md) | 项目概览、定位、核心能力 | 高 |
| [01_Architecture.md](./01_Architecture.md) | 系统架构、组件交互、数据流 | 高 |
| [02_Modules.md](./02_Modules.md) | 子系统模块详解 | 高 |

### 接口与实现

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [03_N-API.md](./03_N-API.md) | N-API/HAL 接口说明 | 中 |
| [04_Build.md](./04_Build.md) | GN 构建系统详解 | 高 |
| [05_Security.md](./05_Security.md) | 安全风险评估 | 高 |

### 附录

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [06_Appendix.md](./06_Appendix.md) | 调用链、配置参数 | 低 |

---

## 子系统快速跳转

| 子系统 | 路径 | 相关文档 |
|--------|------|----------|
| AI | `/hats/ai` | [02_Modules.md#ai](./02_Modules.md#ai) |
| 分布式硬件 | `/hats/distributedhardware` | [02_Modules.md#distributedhardware](./02_Modules.md#distributedhardware) |
| HDF | `/hats/hdf` | [02_Modules.md#hdf](./02_Modules.md#hdf) |
| 内核 | `/hats/kernel` | [02_Modules.md#kernel](./02_Modules.md#kernel) |
| 电源管理 | `/hats/powermgr` | [02_Modules.md#powermgr](./02_Modules.md#powermgr) |
| 启动 | `/hats/startup` | [02_Modules.md#startup](./02_Modules.md#startup) |
| 电信 | `/hats/telephony | [02_Modules.md#telephony](./02_Modules.md#telephony) |
| 用户认证 | `/hats/useriam` | [02_Modules.md#useriam](./02_Modules.md#useriam) |

---

## 术语表

| 术语 | 定义 |
|------|------|
| HATS | Hardware Abstract Test Suite，硬件抽象测试套件 |
| HDI | Hardware Driver Interface，硬件驱动接口 |
| HAL | Hardware Abstraction Layer，硬件抽象层 |
| HDF | Hardware Driver Foundation，硬件驱动框架 |
| XTS | X Test Suite，认证测试套件集合 |
| GN | Generate Ninja，构建系统 |
| SA | System Ability，系统能力 |

---

## 版本信息

- **文档版本**：1.0
- **生成日期**：2026-02-06
- **适用版本**：HATS 4.0
