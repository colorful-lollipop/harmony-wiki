# SmartPerf Wiki 导航

## 项目定位

SmartPerf 是专为 OpenHarmony 打造的性能功耗调优工具，通过 GUI 泳道图细粒度分析 CPU 调度、频点、线程时间片、内存及帧率等数据。

## 新人阅读路线

### 第一阶段：概览认知（建议 15 分钟）

| 顺序 | 文档 | 阅读重点 |
|------|------|----------|
| 1 | [README](README.md) | 文档使用说明、覆盖范围 |
| 2 | [00_Overview](00_Overview.md) | 项目定位、核心能力、技术栈概览 |

### 第二阶段：架构理解（建议 30 分钟）

| 顺序 | 文档 | 阅读重点 |
|------|------|----------|
| 3 | [01_Architecture](01_Architecture.md) | 模块划分、数据流、线程模型 |
| 4 | [02_WASM_API](02_WASM_API.md) | trace_streamer WASM 接口 |
| 5 | [03_InnerAPI](03_InnerAPI.md) | device_command Inner Kit 接口 |

### 第三阶段：开发配置（建议 20 分钟）

| 顺序 | 文档 | 阅读重点 |
|------|------|----------|
| 6 | [04_GNBuild](04_GNBuild.md) | 构建配置、Targets 依赖 |
| 7 | [05_BuildArtifacts](05_BuildArtifacts.md) | 产物清单、运行时加载 |

### 第四阶段：安全与运维（建议 30 分钟）

| 顺序 | 文档 | 阅读重点 |
|------|------|----------|
| 8 | [05_AttackSurface](05_AttackSurface.md) | 攻击面分析、输入清单、信任边界 |
| 9 | [06_Security](06_Security.md) | 安全风险评估、漏洞分析、修复建议 |
| 10 | [07_Troubleshooting](07_Troubleshooting.md) | 常见问题、调试方法 |

## 文档索引

### 快速入口

- [README](README.md) — 文档说明
- [SUMMARY](SUMMARY.md) — 本导航页

### 核心文档

| 文档 | 说明 | 更新日期 |
|------|------|----------|
| [00_Overview.md](00_Overview.md) | 项目概览、边界、运行环境 | 2026-02-06 |
| [01_Architecture.md](01_Architecture.md) | 系统架构、组件图、数据流 | 2026-02-06 |
| [02_WASM_API.md](02_WASM_API.md) | WASM 接口、JS 调用 | 2026-02-06 |
| [03_InnerAPI.md](03_InnerAPI.md) | Inner Kit API、回调接口 | 2026-02-06 |
| [04_GNBuild.md](04_GNBuild.md) | GN 构建配置、Targets | 2026-02-06 |
| [05_BuildArtifacts.md](05_BuildArtifacts.md) | 编译产物、安装路径 | 2026-02-06 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析、输入清单 | 2026-02-07 |
| [06_Security.md](06_Security.md) | 安全风险评估、漏洞分析 | 2026-02-06 |
| [07_Troubleshooting.md](07_Troubleshooting.md) | 问题排查指南 | 2026-02-06 |

## 模块索引

| 模块 | 路径 | 关键文档 |
|------|------|----------|
| device_command | `smartperf_device/device_command/` | [03_InnerAPI](03_InnerAPI.md), [04_GNBuild](04_GNBuild.md) |
| device_ui | `smartperf_device/device_ui/` | [01_Architecture](01_Architecture.md) |
| trace_streamer | `smartperf_host/trace_streamer/` | [02_WASM_API](02_WASM_API.md), [04_GNBuild](04_GNBuild.md) |
| ide | `smartperf_host/ide/` | [01_Architecture](01_Architecture.md) |

## 相关链接

- [OpenHarmony SmartPerf 仓库](https://gitee.com/openharmony/developtools_smartperf_host)
- [bundle.json 配置](bundle.json)
- [构建配置](smartperf_device/build/config.gni)
