# SmartPerf Wiki 文档

## 项目概述

SmartPerf 是专为 OpenHarmony 打造的性能功耗调优工具，包含 **Device 端**（设备采集）和 **Host 端**（PC 分析）两大组件。

## 文档覆盖范围

### 已覆盖内容

| 文档 | 状态 | 说明 |
|------|------|------|
| [README.md](README.md) | ✅ | 本文档 |
| [SUMMARY.md](SUMMARY.md) | ✅ | 全站导航 |
| [00_Overview.md](00_Overview.md) | ✅ | 项目概览 |
| [01_Architecture.md](01_Architecture.md) | ✅ | 系统架构 |
| [02_WASM_API.md](02_WASM_API.md) | ✅ | WASM/JS 绑定接口 |
| [03_InnerAPI.md](03_InnerAPI.md) | ✅ | 内部 API（Inner Kit） |
| [04_GNBuild.md](04_GNBuild.md) | ✅ | GN 构建配置 |
| [05_BuildArtifacts.md](05_BuildArtifacts.md) | ✅ | 编译产物 |
| [05_AttackSurface.md](05_AttackSurface.md) | ✅ | 攻击面分析（新增） |
| [06_Security.md](06_Security.md) | ✅ | 安全风险评审 |
| [07_Troubleshooting.md](07_Troubleshooting.md) | ✅ | 问题排查 |

### 技术栈

| 模块 | 语言 | 机制 |
|------|------|------|
| device_command | C++ | Socket IPC + Inner Kit |
| device_ui | ArkTS/ETS | Socket 通信 |
| trace_streamer | C++ (WASM) | Emscripten 导出 |
| ide | TypeScript + Go | WASM 调用 + HDC |

## 文档更新方式

### 何时更新 Wiki

- 新增核心模块
- 修改对外接口（WASM/Inner Kit）
- 变更构建配置（GN）
- 新增安全相关代码
- 发现文档错误或过时

### 更新步骤

1. 修改对应的 `wiki/*.md` 文件
2. 更新 `SUMMARY.md` 导航链接
3. 验证链接有效性
4. 提交更改

### 文档规范

- 关键结论必须有代码证据（文件路径 + 符号名）
- API 文档需包含参数、返回值、错误码
- 安全文档需包含攻击面和风险等级
- 默认使用中文

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于 developtools/smartperf_host 最新代码
- **生成工具**: OpenHarmony Wiki Generator
