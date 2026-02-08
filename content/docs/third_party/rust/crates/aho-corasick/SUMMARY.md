# 阅读指南

本文档提供了 aho-corasick Wiki 的阅读路线建议，帮助不同类型的读者快速找到所需信息。

---

## 推荐阅读路线

### 路线一：快速了解（5分钟）

适合：初次接触该库，需要快速了解其在 OH 中的情况

1. **[README.md](README.md)** - 关键信息速览
2. **[01_Overview.md](01_Overview.md)** - 库概述（重点看"在 OH 中的作用"部分）
3. **[02_Patches.md](02_Patches.md)** - 查看 Patch 总结（本库无 Patch）

### 路线二：开发者指南（15分钟）

适合：需要在 OH 中使用该库的开发者

1. **[01_Overview.md](01_Overview.md)** - 完整阅读，了解 API 和使用方式
2. **[03_Build_Integration.md](03_Build_Integration.md)** - 了解如何在自己的 BUILD.gn 中添加依赖
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 参考实际使用示例
4. 上游文档: https://docs.rs/aho-corasick/0.7.20/

### 路线三：维护者指南（20分钟）

适合：负责维护和升级该库的工程师

1. **[README.md](README.md)** - 了解整体情况
2. **[02_Patches.md](02_Patches.md)** - 详细阅读 Patch 分析（本库无 Patch）
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 深入理解构建配置
4. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解依赖关系，评估升级影响
5. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 查看完整评估报告

### 路线四：安全审计（10分钟）

适合：进行安全审计的人员

1. **[README.md](README.md)** - 关键信息速览
2. **[02_Patches.md](02_Patches.md)** - 重点阅读 Patch 安全分析
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解使用场景和攻击面
4. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 查看安全评估章节

---

## 文档关系图

```
                    ┌─────────────┐
                    │  README.md  │
                    │   (入口)    │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌────────────┐  ┌────────────┐  ┌────────────┐
    │01_Overview │  │02_Patches  │  │03_Build_   │
    │            │  │            │  │Integration │
    └──────┬─────┘  └──────┬─────┘  └──────┬─────┘
           │               │               │
           └───────────────┼───────────────┘
                           │
                           ▼
                    ┌────────────┐
                    │04_Usage_   │
                    │in_OH.md    │
                    └────────────┘
```

---

## 各文档内容简介

### 核心文档

| 文档 | 内容摘要 | 阅读建议 |
|-----|---------|---------|
| [README.md](README.md) | 项目入口，关键信息速览 | **必读** - 所有读者 |
| [01_Overview.md](01_Overview.md) | 库简介、功能特性、OH 定位 | 开发者和新接触者必读 |
| [02_Patches.md](02_Patches.md) | Patch 详细分析（本库无 Patch） | 维护者和审计人员必读 |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 配置详解 | 需要集成该库的开发者 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用场景 | 维护者和架构师必读 |

### 工作文档

| 文档 | 内容摘要 | 阅读建议 |
|-----|---------|---------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 详细技术评估报告 | 需要深度了解者 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 | 维护者参考 |

---

## 按主题查找

### 我想了解...

**...该库是什么**
→ [01_Overview.md](01_Overview.md) 第一、二节

**...OH 对它做了什么修改**
→ [02_Patches.md](02_Patches.md) - 结论：无修改

**...如何在项目中使用它**
→ [01_Overview.md](01_Overview.md) 第三节（API 示例）  
→ [03_Build_Integration.md](03_Build_Integration.md) 第二节（添加依赖）

**...升级该库需要注意什么**
→ [02_Patches.md](02_Patches.md) 第三节（升级建议）  
→ [04_Usage_in_OH.md](04_Usage_in_OH.md)（依赖关系）

**...它是否安全**
→ [02_Patches.md](02_Patches.md) 第四节（安全分析）  
→ [_work/ASSESSMENT.md](_work/ASSESSMENT.md) 第 7 章

**...谁在使用它**
→ [04_Usage_in_OH.md](04_Usage_in_OH.md) 第一、二节

**...构建配置详解**
→ [03_Build_Integration.md](03_Build_Integration.md)

---

## 外部参考资料

### 上游文档

- **GitHub 仓库**: https://github.com/BurntSushi/aho-corasick
- **API 文档**: https://docs.rs/aho-corasick/0.7.20/aho_corasick/
- **Crates.io**: https://crates.io/crates/aho-corasick/0.7.20

### 算法背景

- **Wikipedia**: https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm
- **原始论文**: Aho, Alfred V.; Corasick, Margaret J. (1975). "Efficient string matching: An aid to bibliographic search"

### OpenHarmony 相关

- **OH 构建系统**: 参考 BUILD.gn 和 GN 文档
- **Rust 在 OH 中的使用**: 参考 third_party/rust 目录其他 crate

---

## 常见问题 FAQ

**Q: 这个库在 OH 中有 Patch 吗？**  
A: 没有。aho-corasick 在 OH 中完全使用上游原始代码，无任何 Patch。详见 [02_Patches.md](02_Patches.md)。

**Q: 如何在自己的组件中使用这个库？**  
A: 在 BUILD.gn 中添加依赖：`deps = ["//third_party/rust/crates/aho-corasick:lib"]`。详见 [03_Build_Integration.md](03_Build_Integration.md)。

**Q: 升级这个库复杂吗？**  
A: 相对简单。由于无 Patch，只需更新版本号并验证依赖兼容性。详见 [02_Patches.md](02_Patches.md) 第三节。

**Q: 这个库安全吗？**  
A: 该库是纯算法实现，使用 Rust 语言（内存安全），无已知 CVE。详见 [02_Patches.md](02_Patches.md) 第四节。

---

## 文档维护

- **创建日期**: 2025-02-07
- **最后更新**: 2025-02-07
- **维护者**: OpenHarmony Wiki Agent

如发现文档错误或有改进建议，请更新相关文档或添加注释。
