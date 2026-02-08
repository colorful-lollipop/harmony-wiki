# libedit Wiki

OpenHarmony `third_party/libedit` 库文档，专注于记录 libedit 在 OH 中的集成、适配和使用情况。

> **重要提示**：经过分析，libedit 目前在 OpenHarmony 中**未被实际使用**，仅进行了跨编译支持适配。

---

## 文档覆盖范围

本文档描述 OpenHarmony 的 libedit 第三方库，涵盖：

- **库概览**：libedit 的功能、版本、许可证
- **OH 适配状态**：跨编译支持、Patch 分析
- **使用情况**：依赖关系、使用场景（当前：无）
- **构建配置**：autotools 构建系统
- **安全评估**：已知漏洞、升级策略

---

## 快速导航

### 核心文档

| 文档 | 内容 | 优先级 |
|------|------|--------|
| [项目评估](./_work/ASSESSMENT.md) | 完整的项目评估结果 | ⭐⭐⭐ |
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 | ⭐⭐⭐ |
| [02_Patches.md](./02_Patches.md) | Patch 详细分析 | ⭐⭐⭐ |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配 | ⭐⭐ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用情况 | ⭐⭐ |
| [06_Security.md](./06_Security.md) | 安全风险分析 | ⭐ |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | Phase 0 项目评估结果 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度跟踪 |

---

## 关键发现

### 当前状态

| 项目 | 状态 |
|------|------|
| **OH 使用情况** | ❌ 未发现任何依赖者 |
| **BUILD.gn 适配** | ❌ 无 GN 构建配置 |
| **Patch 文件** | ✅ 历史性跨编译支持（已整合到上游） |
| **代码级修改** | ❌ 无 OH 特定代码修改 |

### 唯一的 OH 适配

**Patch**: `cross_compile_support_ohos.patch`

- **提交**: a89010b (2024-04-20)
- **内容**: 在 `config.sub` 中添加 OHOS 系统支持
- **状态**: ✅ 已整合到上游版本 (3.1-20250104)
- **回归风险**: ❌ 无（上游已支持）

---

## 阅读路线

### 新人入门

1. **[项目评估](_work/ASSESSMENT.md)** - 了解完整的评估结果和发现
2. **[01_Overview.md](./01_Overview.md)** - 了解 libedit 库的基本信息
3. **[02_Patches.md](./02_Patches.md)** - 了解 OH 对 libedit 的唯一修改
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 了解为什么 libedit 在 OH 中未被使用

### 维护者路线

1. **[项目评估](_work/ASSESSMENT.md)** - 完整评估结果
2. **[02_Patches.md](./02_Patches.md)** - Patch 升级建议
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建配置说明
4. **[06_Security.md](./06_Security.md)** - 安全维护策略

---

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 阅读路线建议
├── 01_Overview.md              # 原始库简介
├── 02_Patches.md               # Patch 详细分析
├── 03_Build_Integration.md     # OH 构建适配
├── 04_Usage_in_OH.md          # 依赖关系与使用
├── 05_API_Differences.md      # API/接口差异（如有）
├── 06_Security.md             # 安全风险分析
└── _work/
    ├── ASSESSMENT.md          # 项目评估结果 ✅
    ├── NOTES.md              # 分析过程记录
    └── PLAN.md              # 任务进度跟踪
```

---

## 关于 libedit

### 基本信息

- **名称**: Editline Library (libedit)
- **版本**: 3.1-20250104
- **许可证**: BSD-3-Clause
- **功能**: 提供与 GNU Readline 相似的行编辑、历史记录功能
- **上游地址**: https://www.thrysoee.dk/editline/

### 在 OpenHarmony 中的定位

⚠️ **当前状态**: libedit 在 OpenHarmony 中**未被实际使用**。

可能的原因：
1. 预引入未使用（为将来需要命令行编辑功能做准备）
2. 已被替代（其他方案实现了类似功能）
3. 仅用于交叉编译环境（构建工具依赖）

**建议**: 与 OH 项目组确认 libedit 的实际用途和维护计划。

---

## 更新日志

| 日期 | 更新内容 |
|------|----------|
| 2025-02-07 | 初始版本，完成 Phase 0 评估 |

---

## 相关链接

- **OpenHarmony 官网**: https://www.openharmony.cn/
- **libedit 上游**: https://www.thrysoee.dk/editline/
- **NetBSD Editline**: https://cvsweb.netbsd.org/bsdweb.cgi/src/lib/libedit
- **GitHub Issues**: https://gitee.com/openharmony/third_party_llvm-project/issues/I9GMT2

---

**文档最后更新**: 2025-02-07
**评估版本**: libedit 3.1-20250104
