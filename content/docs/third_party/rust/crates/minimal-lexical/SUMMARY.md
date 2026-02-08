# 阅读路线建议

本文档提供不同角色的阅读路线建议，帮助您快速获取所需信息。

## 阅读路线总览

```
README.md (入口)
    │
    ├── 快速了解路线（5分钟）
    │       └── 01_Overview.md
    │
    ├── 集成开发者路线（15分钟）
    │       ├── 01_Overview.md
    │       ├── 03_Build_Integration.md
    │       └── 04_Usage_in_OH.md
    │
    ├── 维护升级路线（30分钟）
    │       ├── 01_Overview.md
    │       ├── 02_Patches.md
    │       ├── 03_Build_Integration.md
    │       └── 06_Security.md
    │
    └── 深度研究路线（60分钟+）
            └── 所有文档 + 源代码
```

---

## 路线一：快速了解（5 分钟）

**目标读者**：
- 第一次接触此库的开发者
- 需要快速了解基本信息的决策者
- 评估技术选型的架构师

**阅读顺序**：

1. **[README.md](README.md)** - 库概览
   - 了解基本属性（名称、版本、许可证）
   - 了解 OH 适配类型（无 Patch 直接集成）
   - 查看关键结论

2. **[01_Overview.md](01_Overview.md)** - 库简介
   - 了解原始库功能
   - 了解在 OH 中的定位
   - 查看算法实现概述

**获取的知识**：
- minimal-lexical 是什么
- 它在 OH 中的作用
- 是否需要特别关注

---

## 路线二：集成开发者（15 分钟）

**目标读者**：
- 需要在项目中使用此库的开发者
- 配置构建系统的工程师
- 调试依赖问题的开发者

**阅读顺序**：

1. **[01_Overview.md](01_Overview.md)** - 库简介
   - 了解 API 接口
   - 了解功能特性

2. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建集成
   - 了解 BUILD.gn 配置
   - 了解如何在自己的 BUILD.gn 中引用
   - 了解 features 配置

3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - OH 中的使用
   - 了解依赖关系
   - 查看使用示例
   - 了解典型应用场景

**获取的知识**：
- 如何在代码中使用 minimal-lexical
- 如何在 BUILD.gn 中添加依赖
- 该库的依赖者有哪些

---

## 路线三：维护升级（30 分钟）

**目标读者**：
- 负责第三方库维护的工程师
- 需要升级库版本的开发者
- 进行安全审计的人员

**阅读顺序**：

1. **[01_Overview.md](01_Overview.md)** - 库简介
   - 了解版本信息
   - 了解上游地址

2. **[02_Patches.md](02_Patches.md)** - Patch 分析
   - 确认无 Patch（重要！）
   - 了解无 Patch 的原因
   - 了解升级时的注意事项

3. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建集成
   - 了解构建配置细节
   - 了解与上游 Cargo.toml 的映射关系

4. **[06_Security.md](06_Security.md)** - 安全分析
   - 了解已知 CVE
   - 了解安全风险
   - 了解安全升级策略

**获取的知识**：
- 如何升级到新版本
- 升级时需要注意什么
- 是否存在安全风险

---

## 路线四：深度研究（60 分钟+）

**目标读者**：
- 需要深入理解实现细节的开发者
- 进行源码级调试的工程师
- 贡献代码的开发者

**阅读顺序**：

1. **所有 Wiki 文档**
   - 按编号顺序阅读 01-06
   - 查看 _work/ 目录下的工作文档

2. **源代码阅读**
   ```
   src/
   ├── lib.rs          # 入口和模块导出
   ├── parse.rs        # 核心解析逻辑
   ├── lemire.rs       # Eisel-Lemire 算法
   ├── bellerophon.rs  # Bellerophon 算法
   └── slow.rs         # 慢速路径（大整数）
   ```

3. **测试代码阅读**
   ```
   tests/
   ├── parse_tests.rs      # 解析测试
   ├── lemire_tests.rs     # Lemire 算法测试
   ├── bellerophon_tests.rs # Bellerophon 测试
   └── slow_tests.rs       # 慢速路径测试
   ```

4. **相关论文和资料**
   - Eisel-Lemire 算法论文
   - Bellerophon 算法论文
   - Rust 浮点数解析最佳实践

**获取的知识**：
- 浮点数解析的算法原理
- 代码实现细节
- 测试覆盖情况

---

## 按角色推荐路线

| 角色 | 推荐路线 | 预计时间 |
|------|---------|---------|
| 应用开发者 | 快速了解 | 5 分钟 |
| 系统开发者 | 集成开发者 | 15 分钟 |
| 构建工程师 | 集成开发者 + 维护升级 | 30 分钟 |
| 维护工程师 | 维护升级 | 30 分钟 |
| 安全工程师 | 维护升级 | 30 分钟 |
| 技术专家 | 深度研究 | 60 分钟+ |

---

## 常见问题速查

### Q: 这个库有没有 Patch？
**A**: 没有。查看 [02_Patches.md](02_Patches.md)

### Q: 哪些组件依赖这个库？
**A**: 只有 nom。查看 [04_Usage_in_OH.md](04_Usage_in_OH.md)

### Q: 如何在自己的项目中使用？
**A**: 在 BUILD.gn 中添加 `//third_party/rust/crates/minimal-lexical:lib`。查看 [03_Build_Integration.md](03_Build_Integration.md)

### Q: 如何升级到新版本？
**A**: 直接替换源码，更新 BUILD.gn 版本号。查看 [02_Patches.md](02_Patches.md) 的升级建议章节

### Q: 这个库有安全风险吗？
**A**: 风险较低。查看 [06_Security.md](06_Security.md)

---

*本文档最后更新：2026-02-07*
