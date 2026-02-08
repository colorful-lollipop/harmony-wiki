# popt - 命令行参数解析库

## 简介

popt 是 OpenHarmony 第三方库中的一个 C 语言命令行参数解析库。它是 RPM 软件包管理器项目的一部分，提供了比标准 getopt 更强大的命令行解析功能。

在 OpenHarmony 中，popt 主要用于 **gptfdisk** 工具的命令行参数解析。

## 文档导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析 (本库无 Patch) |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建适配 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | OH 中的使用方式与依赖关系 |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

## 快速概览

### 基本信息
| 属性 | 值 |
|------|-----|
| 上游版本 | 1.19 |
| OH 组件名 | @ohos/popt |
| OH 版本 | 3.1 |
| 许可证 | MIT License |
| Patch 数量 | **0** (无需修改) |

### OpenHarmony 适配亮点
- ✅ **零 Patch**: 原始代码完全满足需求，无需修改
- ✅ **轻量级**: 仅启用核心功能，构建产物小
- ✅ **易维护**: 升级上游版本无障碍

### 在 OH 中的作用
popt 为命令行工具提供专业的参数解析能力，当前主要服务于磁盘分区管理工具 gptfdisk 的 sgdisk 组件。

## 技术特点

### 核心功能
- 支持长选项 (`--option`) 和短选项 (`-o`)
- 支持参数别名和参数配置文件
- 自动生成 `--help` 和 `--usage` 信息
- 完全可重入 (线程安全)
- 支持任意 argv[] 风格数组解析

### OH 特定配置
- 静态库形式提供 (`popt_static`)
- 启用国际化支持 (i18n)
- 禁用配置文件功能 (gptfdisk 不需要)
- 代码段优化 (`-ffunction-sections`)

## 使用建议

### 适用场景
- 需要专业命令行参数解析的 OH 命令行工具
- 替代手动 getopt 解析的复杂逻辑
- 需要自动生成帮助信息的工具

### 注意事项
- 该库仅提供 C API，C++ 项目需 extern "C" 包裹
- 头文件路径: `//third_party/popt/src/popt.h`
- 依赖方式: `external_deps = ["popt:popt_static"]`

## 相关链接

- 上游仓库: https://github.com/rpm-software-management/popt
- 上游文档: popt.pdf (包含在源码中)
- OpenHarmony 组件: `//third_party/popt:popt_static`
