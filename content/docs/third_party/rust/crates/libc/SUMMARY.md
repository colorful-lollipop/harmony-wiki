# SUMMARY - 阅读路线建议

> **适用对象**: 所有需要了解 libc 在 OpenHarmony 中适配情况的人员
> **更新时间**: 2026-02-08

---

## 📚 文档结构总览

本 Wiki 按照以下结构组织：

```
wiki/
├── README.md                  # 文档首页和导航（从这里开始）
├── SUMMARY.md                 # 本文件 - 阅读路线建议
│
├── 01_Overview.md             # 原始库简介
├── 02_Patches.md              # Patch 详细分析
├── 03_Build_Integration.md    # OH 构建适配
├── 04_Usage_in_OH.md         # 依赖关系与使用
├── 05_API_Differences.md      # API/接口差异（如有）
└── 06_Security.md            # 安全风险分析
│
└── _work/
    ├── ASSESSMENT.md          # 项目评估结果
    ├── NOTES.md               # 分析过程记录
    └── PLAN.md                # 任务进度
```

---

## 🎯 按角色推荐阅读路线

### 👨‍💻 开发者

**目标**: 了解如何在 OHOS 中使用 libc，以及如何升级/维护

**推荐路线** (约 45 分钟):

```
1. 01_Overview.md (10 min)
   └─ 了解 libc 的基本功能和在 OH 中的作用

2. 03_Build_Integration.md (15 min)
   └─ 了解 BUILD.gn 配置和特性

3. 04_Usage_in_OH.md (10 min)
   └─ 了解依赖关系和使用场景

4. 02_Patches.md (10 min)
   └─ 了解现有 Patch，为升级做准备
```

**关键章节**:
- 01_Overview.md: "1.4 OpenHarmony 特性总结"
- 03_Build_Integration.md: "3.2 BUILD.gn 详细说明" 和 "3.3 build.rs 功能说明"
- 04_Usage_in_OH.md: "典型使用场景"

---

### 🔧 维护者

**目标**: 理解 OHOS 的所有适配细节，能够安全地升级版本

**推荐路线** (约 90 分钟):

```
1. _work/ASSESSMENT.md (15 min)
   └─ 了解整体评估结果和风险

2. 01_Overview.md (15 min)
   └─ 完整了解库的定位和 OHOS 特性

3. 02_Patches.md (20 min)
   └─ 详细的 Patch 分析和升级策略

4. 03_Build_Integration.md (30 min)
   └─ 完整的构建系统说明和特殊处理

5. 04_Usage_in_OH.md (10 min)
   └─ 依赖关系和影响范围
```

**关键章节**:
- 02_Patches.md: "2.3 Patch 与上游的同步策略"
- 03_Build_Integration.md: "3.5 特殊处理" 和 "3.9 构建调试"
- 04_Usage_in_OH.md: "依赖关系图"

---

### 🛡️ 安全审计人员

**目标**: 评估 libc 在 OHOS 中的安全风险

**推荐路线** (约 30 分钟):

```
1. 02_Patches.md (5 min)
   └─ 了解 Patch 的安全影响

2. 06_Security.md (20 min)
   └─ 详细的安全风险分析

3. 03_Build_Integration.md (5 min)
   └─ 了解构建配置的安全考虑
```

**关键章节**:
- 02_Patches.md: "2.2 详细 Patch 分析" - "回归风险"
- 06_Security.md: "已知 CVE 和修复状态" 和 "OHOS Patch 的安全风险"

---

### 🎓 新手学习者

**目标**: 了解 libc 是什么，以及它在 OHOS 中如何工作

**推荐路线** (约 20 分钟):

```
1. 01_Overview.md (15 min)
   └─ 重点阅读：
      - 1.2 功能概述
      - 1.3 在 OpenHarmony 中的作用和定位
      - 1.4 OpenHarmony 特性总结
      - 1.6 常见问题（FAQ）

2. README.md (5 min)
   └─ 快速导航和关键发现摘要
```

---

### 🏗️ 架构师/技术决策者

**目标**: 理解 libc 在 OHOS 技术栈中的位置和重要性

**推荐路线** (约 30 分钟):

```
1. 01_Overview.md (15 min)
   └─ 重点阅读：
      - 1.1 库基本信息
      - 1.3 在 OpenHarmony 中的作用和定位
      - "在 OHOS 系统层次中的位置"图

2. 04_Usage_in_OH.md (10 min)
   └─ 了解依赖关系和主要使用者

3. README.md (5 min)
   └─ 关键发现摘要
```

**关键章节**:
- 01_Overview.md: "1.3 在 OpenHarmony 中的作用和定位"
- 01_Overview.md: Mermaid 依赖图（在 OH 系统层次中的位置）
- 04_Usage_in_OH.md: "依赖关系图"

---

## 📋 按任务类型推荐阅读路线

### 任务: 升级 libc 到新版本

**推荐路线** (约 60 分钟):

```
1. _work/ASSESSMENT.md (10 min)
   └─ 了解当前状态和风险

2. 02_Patches.md (20 min)
   └─ 重点：
      - 2.2 详细 Patch 分析
      - 2.3 Patch 与上游的同步策略
      - 2.4 未来 Patch 预测

3. 03_Build_Integration.md (20 min)
   └─ 重点：
      - 3.5 特殊处理（禁用功能、类型对齐）
      - 3.9 构建调试（常见问题）

4. 04_Usage_in_OH.md (10 min)
   └─ 了解依赖范围，评估影响
```

**升级检查清单**:
- [ ] 验证 OHOS 特定的适配是否仍然有效（utmpx 布局、locale 常量等）
- [ ] 检查上游新增的代码块是否需要排除 OHOS
- [ ] 运行完整的测试套件
- [ ] 确认与 OHOS C 库的兼容性
- [ ] 保留或更新 CI Patch

---

### 任务: 修复编译错误

**推荐路线** (约 30 分钟):

```
1. 03_Build_Integration.md (25 min)
   └─ 重点：
      - 3.9 构建调试（常见构建问题）
      - 3.6 编译配置详解

2. 01_Overview.md (5 min)
   └─ 确认 OHOS 特性总结
```

**常见错误类型**:
- 类型冲突 → 查看特殊处理中的类型对齐
- 未定义符号 → 查看禁用的功能列表
- 宏未找到 → 检查 build.rs 配置

---

### 任务: 添加新的 OHOS 适配

**推荐路线** (约 45 分钟):

```
1. 01_Overview.md (10 min)
   └─ 了解 OHOS 特性总结

2. 03_Build_Integration.md (20 min)
   └─ 重点：
      - 3.5 特殊处理（现有适配模式）
      - 3.6 编译配置详解（条件编译）

3. 02_Patches.md (10 min)
   └─ 了解 Patch 管理最佳实践

4. _work/ASSESSMENT.md (5 min)
   └─ 参考 OHOS 特定支持部分
```

**适配模式参考**:
- utmpx 布局差异 → `src/unix/linux_like/linux/musl/mod.rs`
- 缺失函数排除 → `src/unix/linux_like/linux/mod.rs`
- locale 常量扩展 → `src/unix/linux_like/mod.rs`
- 类型对齐规则 → `src/unix/linux_like/linux/align.rs`

---

### 任务: 安全审计

**推荐路线** (约 30 分钟):

```
1. 02_Patches.md (5 min)
   └─ Patch 的回归风险

2. 06_Security.md (20 min)
   └─ 完整阅读

3. README.md (5 min)
   └─ 风险评估摘要
```

**审计检查点**:
- [ ] 已知 CVE 是否在 OH 版本中修复
- [ ] OHOS Patch 是否引入新的安全风险
- [ ] 禁用的功能是否有安全影响
- [ ] 类型对齐是否正确（防止内存安全问题）

---

### 任务: 理解依赖关系

**推荐路线** (约 15 分钟):

```
1. 04_Usage_in_OH.md (10 min)
   └─ 依赖关系分析

2. 01_Overview.md (5 min)
   └─ 在 OH 系统层次中的位置图
```

**关键输出**:
- 直接依赖者列表
- 典型使用场景
- 依赖关系图
- 在 OH 技术栈中的位置

---

## 📖 文档阅读技巧

### 快速浏览
- 使用文档中的 **表格** 和 **Mermaid 图** 快速理解结构
- 关注 **⚠️ 风险评估** 和 **🔧 Patch 分析** 部分
- 查看 **常见问题（FAQ）** 解决基础疑问

### 深入理解
- 阅读 **源码示例** 理解具体实现
- 查看 **外部参考** 链接获取更多信息
- 参考 **内部资源** 链接直接查看代码

### 实际应用
- 根据具体任务选择相应的 **推荐路线**
- 使用 **检查清单** 确保不遗漏关键步骤
- 参考 **构建调试** 部分解决实际问题

---

## 🔗 相关文档索引

### 核心概念
- **FFI（Foreign Function Interface）**: Rust 与 C 语言互调的机制
- **musl libc**: OpenHarmony 使用的轻量级 C 标准库
- **GN（Generate Ninja）**: OHOS 使用的构建系统
- **条件编译**: 使用 `#[cfg(...)]` 或 `#ifdef` 实现平台特定代码

### 关键文件
- [BUILD.gn](../BUILD.gn) - OH 构建配置
- [Cargo.toml](../Cargo.toml) - Cargo 包配置
- [build.rs](../build.rs) - 构建脚本
- [src/lib.rs](../src/lib.rs) - Rust crate 入口

### 外部资源
- [Rust FFI 文档](https://doc.rust-lang.org/nomicon/ffi.html)
- [The Rustonomicon](https://doc.rust-lang.org/nomicon/)
- [musl libc 官网](https://musl.libc.org/)

---

## 💡 学习建议

1. **从 Overview 开始**: 先阅读 01_Overview.md 建立整体认识
2. **结合源码**: 阅读文档时对照实际源码，加深理解
3. **实践验证**: 尝试修改配置或添加代码，验证理解
4. **关注更新**: lib c 和 OHOS 都在不断更新，保持关注

---

## ❓ 获取帮助

如果您在阅读或使用过程中遇到问题：

1. 📖 查阅 **常见问题（FAQ）**（各文档的最后一节）
2. 🔍 查看 **构建调试** 部分（03_Build_Integration.md 3.9）
3. 📝 提交 Issue 或 PR
4. 💬 联系维护者

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
**维护者**: Sisyphus (OpenHarmony Third-Party Wiki Agent)
