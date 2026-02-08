# aho-corasick - OpenHarmony Wiki

## 概述

本 Wiki 文档记录了 **aho-corasick** 第三方库在 OpenHarmony (OH) 中的集成与适配情况。

aho-corasick 是一个高性能的多模式字符串匹配库，实现了经典的 [Aho-Corasick 算法](https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm)，能够在文本中同时搜索多个模式字符串，具有线性时间复杂度 O(n+m) 的优秀性能。

---

## 文档导航

### 快速开始

- [库概述](01_Overview.md) - 了解 aho-corasick 的基本信息和在 OH 中的定位
- [Patch 分析](02_Patches.md) - 查看 OH 对该库的修改和适配
- [构建集成](03_Build_Integration.md) - BUILD.gn 配置详解
- [OH 中的使用](04_Usage_in_OH.md) - 依赖关系和使用场景

### 详细信息

| 文档 | 内容 |
|-----|------|
| [ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告（含详细技术调查） |
| [NOTES.md](_work/NOTES.md) | 分析过程记录 |

---

## 关键信息速览

### 库基本信息

| 属性 | 值 |
|-----|---|
| **名称** | aho-corasick |
| **上游版本** | 0.7.20 |
| **上游作者** | Andrew Gallant (BurntSushi) |
| **许可证** | MIT / Unlicense (双许可) |
| **上游地址** | https://github.com/BurntSushi/aho-corasick |
| **OH 组件名** | @ohos/rust_aho_corasick |
| **OH 版本** | 6.1 |
| **所属子系统** | thirdparty |

### 适配状态

| 项目 | 状态 | 说明 |
|-----|------|------|
| **Patch 数量** | 0 | 无任何 Patch，使用原始代码 |
| **构建适配** | 完成 | BUILD.gn 配置完成 |
| **OH 特有代码** | 无 | 无 OH 特定修改 |
| **维护成本** | 低 | 纯算法库，无需特殊维护 |

---

## 核心特性

aho-corasick 库提供以下功能：

- **多模式匹配**: 同时搜索多个模式字符串
- **线性时间复杂度**: O(n+m) 的高效搜索
- **SIMD 加速**: 利用 SIMD 指令优化性能
- **多种匹配语义**: 标准、leftmost-first、leftmost-longest
- **流式处理**: 支持流式搜索和替换
- **大小写不敏感**: 支持 ASCII 大小写不敏感匹配
- **内存安全**: Rust 语言保证，无内存安全风险

---

## 在 OpenHarmony 中的作用

aho-corasick 在 OpenHarmony 中主要作为**底层算法库**使用：

1. **正则表达式引擎的基础**: 为 regex crate 提供多模式匹配支持
2. **文本处理工具**: 支持系统组件中的文本搜索和过滤
3. **日志分析**: 多模式日志匹配和分析
4. **安全检测**: 敏感词过滤和模式匹配

---

## 文档使用建议

### 如果你是...

**开发者（使用此库）**:
1. 阅读 [01_Overview.md](01_Overview.md) 了解基本 API
2. 查看上游文档: https://docs.rs/aho-corasick
3. 注意 OH 版本为 0.7.20，使用对应版本文档

**维护者（升级此库）**:
1. 阅读 [02_Patches.md](02_Patches.md) - 当前无 Patch，升级相对简单
2. 检查 [03_Build_Integration.md](03_Build_Integration.md) 确认构建配置
3. 验证依赖此库的组件兼容性

**安全审计人员**:
1. 查看 [02_Patches.md](02_Patches.md) - 无 Patch 意味着无代码修改风险
2. 关注上游 CVE 公告
3. 该库为纯算法实现，内存安全由 Rust 保证

---

## 相关链接

- **上游仓库**: https://github.com/BurntSushi/aho-corasick
- **Crates.io**: https://crates.io/crates/aho-corasick
- **文档**: https://docs.rs/aho-corasick/0.7.20/aho_corasick/
- **算法介绍**: https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm

---

## 维护记录

| 日期 | 操作 | 负责人 |
|-----|------|-------|
| 2025-02-07 | 创建 Wiki 文档 | Wiki Agent |

---

**注意**: 本文档专注于 OpenHarmony 对该库的集成和适配。关于库的原始功能，请参考上游文档。
