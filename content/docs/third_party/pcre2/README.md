# PCRE2 - OpenHarmony Wiki

## 库概览

| 属性 | 值 |
|-----|-----|
| **库名称** | PCRE2 (Perl Compatible Regular Expressions) |
| **上游版本** | 10.46 |
| **上游地址** | https://github.com/PhilipHazel/pcre2.git |
| **许可证** | BSD-3-Clause WITH PCRE2-exception |
| **OH 组件** | @ohos/pcre2 |
| **子系统** | thirdparty |
| **适配类型** | standard |

## 核心功能

PCRE2 是一个实现了与 Perl 5 语法和语义兼容的正则表达式库。在 OpenHarmony 中，它主要用于：

1. **ArkCompiler 运行时** - ArkTS/ETS 语言的正则表达式支持
2. **SELinux 安全框架** - 安全策略的解析和匹配
3. **仓颉语言运行时** - 仓颉语言的正则表达式支持

## OpenHarmony 适配要点

### 关键 Patch

本库包含一个重要的 **运行时 Patch**，用于适配 ArkTS/ETS 语言规范：

- **Patch 文件**: `arkcompiler/runtime_core/static_core/plugins/ets/runtime/patches/pcre2_newline.patch`
- **Patch 目的**: 修改换行符识别行为，仅识别 LF/CR/LS/PS 为换行符
- **影响范围**: ArkCompiler 使用的静态库版本（`libpcre2_static`、`libpcre2_static_16`）

### 构建适配

- **构建系统**: 从 CMake/Autotools 迁移到 GN (Generate Ninja)
- **配置方式**: 使用预生成的 `.generic` 配置文件
- **字符集**: 同时支持 8-bit 和 16-bit 字符集

## 文档导航

| 文档 | 内容 |
|-----|-----|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | **核心文档** - Patch 详细分析 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异说明 |
| [06_Security.md](./06_Security.md) | 安全分析与 CVE |

## 快速链接

- [上游 PCRE2 文档](https://PCRE2Project.github.io/pcre2/doc/html/index.html)
- [PCRE2 GitHub](https://github.com/PCRE2Project/pcre2)
- [OpenHarmony 安全公告](https://gitee.com/openharmony/security)

## 维护者

- **Owner**: maliang34@huawei.com
- **最后更新**: 2025-02-07

---

> **注意**: 本文档专注于 OpenHarmony 对 PCRE2 的定制化内容。关于 PCRE2 原始库的详细功能，请参考[上游文档](https://PCRE2Project.github.io/pcre2/doc/html/index.html)。
