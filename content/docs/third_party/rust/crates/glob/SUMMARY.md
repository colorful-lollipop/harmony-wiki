# 阅读路线建议

本文档提供针对不同读者的 glob 库 Wiki 文档阅读指南。

---

## 📋 文档结构总览

```
wiki/
├── README.md              # 库概览、OH 适配概述、文档导航
├── SUMMARY.md             # 本文档 - 阅读路线建议
├── 01_Overview.md        # 原始库简介
├── 03_Build_Integration.md # OH 构建适配详解
├── 04_Usage_in_OH.md     # 依赖关系与使用场景
├── 05_API_Differences.md # API/接口差异（无差异）
└── 06_Security.md        # 安全风险分析
```

---

## 🎯 按角色阅读路线

### 👨‍💻 开发者

**目标**: 了解如何使用 glob 库及其在 OH 中的配置

**阅读顺序**:
1. [README.md](README.md) - 快速了解库概览
2. [01_Overview.md](01_Overview.md) - 了解库的核心功能
3. [03_Build_Integration.md](03_Build_Integration.md) - 学习如何在 BUILD.gn 中依赖 glob

**预计阅读时间**: 15 分钟

**关键要点**:
- glob 是零 Patch 适配，直接使用上游版本
- 使用 `//third_party/rust/crates/glob:lib` 作为依赖
- 通过静态链接方式集成

---

### 🔧 维护者

**目标**: 掌握库的集成细节和升级策略

**阅读顺序**:
1. [README.md](README.md) - 总体情况
2. [01_Overview.md](01_Overview.md) - 库的基本信息
3. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置详解
4. [06_Security.md](06_Security.md) - 安全性考虑
5. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 完整评估报告

**预计阅读时间**: 30 分钟

**关键要点**:
- 无 Patch，升级风险低
- 建议定期同步上游更新
- 只需更新 `BUILD.gn` 中的 `cargo_pkg_version`

---

### 🔍 审计者

**目标**: 验证库的集成合规性和安全性

**阅读顺序**:
1. [README.md](README.md) - 快速扫描
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系验证
3. [05_API_Differences.md](05_API_Differences.md) - API 差异检查（无差异）
4. [06_Security.md](06_Security.md) - 安全风险评估
5. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 详细分析

**预计阅读时间**: 20 分钟

**关键要点**:
- 源代码零修改
- 无 OH 特定代码
- 仅依赖 clang-sys（用于 libclang 查找）

---

### 📊 系统架构师

**目标**: 理解 glob 在 OH 系统中的定位和价值

**阅读顺序**:
1. [README.md](README.md) - 总览
2. [01_Overview.md](01_Overview.md) - 功能定位
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖链分析
4. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 完整评估

**预计阅读时间**: 25 分钟

**关键要点**:
- glob 作为基础工具库，为 Rust 生态提供文件匹配能力
- 在 OH 中主要服务于 clang-sys 的库文件查找
- 属于第三方子系统，适配复杂度极低

---

### 🆕 新手入门

**目标**: 快速理解 glob 是什么及其用途

**阅读顺序**:
1. [README.md](README.md) - 快速浏览
2. [01_Overview.md](01_Overview.md) - 基础介绍

**预计阅读时间**: 10 分钟

**跳过的部分**:
- 技术细节文档（BUILD.gn 配置、依赖关系图等）
- _work/ 目录（内部工作文档）

---

## 📖 按主题阅读路线

### 主题 1: 了解库的基本信息

**文档**: [01_Overview.md](01_Overview.md)

**内容**:
- 原始库名称、版本、许可证
- 功能描述
- 上游地址
- 在 OH 中的作用和定位

---

### 主题 2: 理解 OH 构建集成

**文档**: [03_Build_Integration.md](03_Build_Integration.md)

**内容**:
- BUILD.gn 结构说明
- Cargo 到 GN 的映射
- 与上游构建系统的差异
- 特殊处理说明

---

### 主题 3: 分析依赖关系

**文档**: [04_Usage_in_OH.md](04_Usage_in_OH.md)

**内容**:
- 直接依赖者列表
- 使用方式
- 关键使用场景
- 依赖关系图

---

### 主题 4: 检查 API 变更

**文档**: [05_API_Differences.md](05_API_Differences.md)

**内容**:
- OH 新增的 API（无）
- 行为变更的 API（无）
- 废弃或禁用的功能（无）

---

### 主题 5: 评估安全性

**文档**: [06_Security.md](06_Security.md)

**内容**:
- 已知 CVE 分析
- OH Patch 引入的新攻击面（无 Patch，无新攻击面）
- 安全升级策略

---

## 🚀 深度阅读路线

### 路线 1: 完整理解（推荐）

**目标**: 全面掌握 glob 在 OH 中的所有方面

**顺序**:
1. README.md
2. SUMMARY.md（本文档）
3. 01_Overview.md
4. 03_Build_Integration.md
5. 04_Usage_in_OH.md
6. 05_API_Differences.md
7. 06_Security.md
8. _work/ASSESSMENT.md（可选，查看详细分析）

**预计阅读时间**: 60 分钟

---

### 路线 2: 快速评估

**目标**: 快速判断库的集成质量

**顺序**:
1. README.md（重点看适配状态表）
2. 04_Usage_in_OH.md（看依赖关系）
3. 06_Security.md（看安全风险）
4. _work/ASSESSMENT.md（看总结）

**预计阅读时间**: 20 分钟

---

### 路线 3: 技术深入

**目标**: 深入了解技术细节

**顺序**:
1. 01_Overview.md
2. 03_Build_Integration.md
3. 04_Usage_in_OH.md
4. _work/ASSESSMENT.md
5. _work/PLAN.md
6. _work/NOTES.md（分析过程记录）

**预计阅读时间**: 45 分钟

---

## ⚠️ 重要提示

### 必读文档

所有读者都应阅读:
- ✅ [README.md](README.md) - 了解库的基本情况和适配状态

### 可选文档

根据需求选择:
- [01_Overview.md](01_Overview.md) - 了解库功能
- [03_Build_Integration.md](03_Build_Integration.md) - 学习构建配置
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 分析依赖关系
- [05_API_Differences.md](05_API_Differences.md) - 检查 API 差异
- [06_Security.md](06_Security.md) - 评估安全性

### 内部文档（仅维护者/审计者）

- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - Phase 0 评估结果
- [_work/PLAN.md](_work/PLAN.md) - 文档生成工作计划
- [_work/NOTES.md](_work/NOTES.md) - 分析过程记录

---

## 📚 延伸阅读

### 相关库文档

- [clang-sys](../../clang-sys/wiki/) - glob 的直接依赖者
- [bindgen](../../bindgen/wiki/) - glob 的间接依赖者

### OpenHarmony 文档

- [第三方库管理指南](https://gitee.com/openharmony/community/blob/master/contributing/binary-guideline/binary_library_management_guide.md)
- [GN 构建系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/subsystems/sub-system-build.md)
- [Rust 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/ide/rust-develop.md)

---

## 🎓 学习建议

### 新手

1. 先读 [README.md](README.md) 了解基本情况
2. 再读 [01_Overview.md](01_Overview.md) 了解功能
3. 如需使用，阅读 [03_Build_Integration.md](03_Build_Integration.md)

### 中级开发者

1. 快速浏览 [README.md](README.md)
2. 重点阅读 [03_Build_Integration.md](03_Build_Integration.md) 和 [04_Usage_in_OH.md](04_Usage_in_OH.md)
3. 参考 [上游文档](https://docs.rs/glob/0.3.1) 学习 API

### 高级开发者/架构师

1. 通读所有文档
2. 查看 [_work/ASSESSMENT.md](_work/ASSESSMENT.md) 了解完整分析
3. 根据需求深入阅读 _work/ 目录中的其他文档

---

**最后更新**: 2026-02-08
