# OpenHarmony drivers_liteos Wiki

## 概述

本文档是 OpenHarmony `drivers_liteos` 子系统的工程 Wiki，旨在帮助开发者快速理解项目定位、架构设计、API 接口、构建配置以及安全考量。

## 覆盖范围

| 模块 | 状态 | 说明 |
|-----|------|------|
| **hievent** | ✅ 完整 | 事件日志管理驱动，完整的代码分析 |
| **tzdriver** | ⚠️ 受限 | REE/TEE 通信驱动，README 声明"未完全开源" |
| **video** | 📍 外部 | 帧缓冲驱动，位于 `third_party/NuttX` |
| **mem/random/quickstart** | 📍 外部 | 位于 `kernel/liteos_a/drivers/char` |

## 文档结构

- [首页](index.md) - 项目概览与快速入门
- [目录结构](02_Directory_Structure.md) - 源码目录说明
- [架构说明](03_Architecture.md) - 组件图、数据流、线程模型
- [hievent API](04_Hievent_API.md) - 驱动接口与事件处理
- [构建配置](05_Build_Configuration.md) - GN targets 与 Kconfig
- [安全分析](06_Security_Analysis.md) - 威胁模型与风险评估
- [常见问题](07_FAQ.md) - 构建、运行、调试问题

## 快速导航

```
新人入门: README → index.md → 03_Architecture.md → 04_Hievent_API.md
开发者:   04_Hievent_API.md → 05_Build_Configuration.md → 06_Security_Analysis.md
安全审计: 06_Security_Analysis.md → 03_Architecture.md (攻击面分析)
```

## 证据来源

所有关键结论均可追溯到代码证据：

- **文件路径**: 包含相对路径和行号
- **符号名称**: 函数、宏、结构体、全局变量
- **代码片段**: 关键实现逻辑

示例：
> 环形缓冲区大小为 1024 字节
> - 证据: `hievent_driver.c:56` → `#define HIEVENT_LOG_BUFFER 1024`

## 更新方式

### 何时更新 Wiki

| 变更类型 | 需要更新 |
|---------|---------|
| 新增/删除源文件 | 02_Directory_Structure.md, 04_Hievent_API.md |
| 修改 API 接口 | 04_Hievent_API.md, SUMMARY.md |
| 修改构建配置 | 05_Build_Configuration.md |
| 发现新的安全风险 | 06_Security_Analysis.md |

### 更新步骤

1. 修改对应源码文件
2. 更新 `wiki/_work/NOTES.md` 添加发现
3. 修改相关 Wiki 文档
4. 更新 `wiki/SUMMARY.md` 链接（如有必要）
5. 运行 `lsp_diagnostics` 确保无语法错误

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Kernel LiteOS_A 仓库](https://gitee.com/openharmony/kernel_liteos_a)
- [drivers_liteos 源码](https://gitee.com/openharmony/drivers_liteos)

---

*最后更新: 2024*
*基于代码版本: 当前 Git HEAD*
