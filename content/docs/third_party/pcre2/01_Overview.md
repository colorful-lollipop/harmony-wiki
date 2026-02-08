# 01 - PCRE2 原始库简介

## 基本信息

| 属性 | 值 |
|-----|-----|
| **全称** | Perl Compatible Regular Expressions 2 |
| **版本** | 10.46 |
| **发布日期** | 2025-08-27 |
| **作者** | Philip Hazel |
| **维护者** | Nicholas Wilson |
| **许可证** | BSD-3-Clause WITH PCRE2-exception |
| **上游地址** | https://github.com/PCRE2Project/pcre2 |

## 原始功能简介

PCRE2 是一个实现了与 **Perl 5 语法和语义兼容**的正则表达式库。它提供了一套 C 语言函数，支持：

- **正则表达式编译**: `pcre2_compile()` - 将正则表达式模式编译为内部格式
- **正则表达式匹配**: `pcre2_match()` - 使用编译后的模式进行匹配
- **DFA 匹配**: `pcre2_dfa_match()` - 使用确定性有限自动机进行匹配
- **字符串替换**: `pcre2_substitute()` - 正则替换功能
- **JIT 编译**: 可选的即时编译器，优化匹配性能
- **Unicode 支持**: 完整的 Unicode 属性支持

### 核心特性

1. **多编码支持**: 8-bit、16-bit、32-bit 字符单元
2. **UTF 支持**: UTF-8、UTF-16、UTF-32 编码
3. **JIT 优化**: 可选的即时编译器，大幅提升匹配性能
4. **POSIX API**: 提供与 POSIX regex 兼容的包装函数
5. **安全特性**: 支持匹配深度限制、堆栈限制等

## PCRE2 与 PCRE1 的区别

PCRE2 于 2015 年首次发布，是原始 PCRE 库的重写版本，主要改进包括：

| 特性 | PCRE1 | PCRE2 |
|-----|-------|-------|
| API 设计 | 单上下文 | 分离的编译和匹配上下文 |
| 内存管理 | 应用管理 | 库内部管理 |
| Ovector | 固定大小 | 动态分配 |
| 错误处理 | 返回码 | 改进的错误信息 |
| 维护状态 | **已废弃** | **活跃维护** |

## OpenHarmony 中的定位

### 为什么需要 PCRE2？

在 OpenHarmony 中，PCRE2 是多个关键组件的基础依赖：

```
├── ArkCompiler (ArkTS/ETS 运行时)
│   └── 正则表达式字面量和 RegExp 对象
├── SELinux 安全框架
│   └── 安全策略规则解析
└── 仓颉语言运行时
    └── 标准库 regex 模块
```

### 与其他正则库对比

| 库 | 用途 | OH 使用情况 |
|---|-----|------------|
| PCRE2 | 通用正则表达式 | ArkCompiler、SELinux、仓颉 |
| Oniguruma | 多语言正则 | Node.js 等 |
| RE2 | 安全正则（无回溯） | 部分安全敏感场景 |

PCRE2 被选中的原因：
1. **功能完整**: 支持 Perl 5 的几乎所有特性
2. **性能优秀**: JIT 编译器提供接近原生的性能
3. **广泛兼容**: 被多种语言和工具采用
4. **活跃维护**: 及时的安全更新

### 版本选择理由

**当前版本**: 10.46 (2025-08-27)

选择该版本的原因：
1. **包含最新安全修复**: 修复了 CVE-2025-58050
2. **功能稳定**: 经过多年生产环境验证
3. **Unicode 15.1**: 支持最新的 Unicode 标准
4. **性能优化**: 持续的 JIT 编译器改进

## 上游社区

- **GitHub**: https://github.com/PCRE2Project/pcre2
- **邮件列表**: pcre2-dev@googlegroups.com
- **发布周期**: 约每 3-6 个月一个小版本
- **安全响应**: 通过 GitHub Security Advisory 报告

## 相关文档

- [PCRE2 官方文档](https://PCRE2Project.github.io/pcre2/doc/html/index.html)
- [PCRE2 API 参考](https://PCRE2Project.github.io/pcre2/doc/html/pcre2api.html)
- [Perl 正则表达式语法](https://perldoc.perl.org/perlre)

---

> **下一步**: 了解 OpenHarmony 对 PCRE2 的关键 Patch 修改，请参阅 [02_Patches.md](./02_Patches.md)。
