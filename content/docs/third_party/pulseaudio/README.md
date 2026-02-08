# PulseAudio Wiki 文档

## 简介

本文档库专注于 **PulseAudio 在 OpenHarmony (OHOS) 中的集成与适配**，详细说明 OHOS 对 PulseAudio 的特殊定制、Patch 分析和构建配置。

## 文档导航

### 入门阅读
1. [01_Overview.md](01_Overview.md) - 库概览与 OHOS 定位
2. [02_Patches.md](02_Patches.md) - **核心文档**：OHOS 适配详情
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - OHOS 中的依赖关系

### 深入阅读
4. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 配置详解
5. [05_API_Differences.md](05_API_Differences.md) - API 差异分析
6. [06_Security.md](06_Security.md) - 安全分析

## 关键信息速览

| 属性 | 值 |
|-----|-----|
| 上游版本 | PulseAudio 17.0 |
| OHOS 版本 | 3.1 |
| 许可证 | LGPL-2.1 |
| Patch 数量 | **0** (采用文件替换式适配) |
| 主要 OHOS 适配 | ohos_pa_main.c, ohos_socket-server.c, audio_log.h |

## 特色说明

本库是 OpenHarmony 中**无传统 Patch 文件**的第三方库典型案例。OHOS 通过以下方式实现适配：
- 新增带 `ohos_` 前缀的替代源文件
- 使用 `HAVE_NO_OHOS` 条件编译宏
- 集成 init 进程 socket 机制
- 适配 HiLog 日志系统

---

*文档生成时间: 2025-02-07*
