# SUMMARY - ncurses OpenHarmony Wiki

## 文档结构

本文档集详细分析了 ncurses 在 OpenHarmony 中的集成与适配情况。

### 核心文档

1. **[README.md](README.md)** - 库概览、文档导航、快速开始
2. **[01_Overview.md](01_Overview.md)** - 原始库简介、在 OH 中的作用
3. **[02_Patches.md](02_Patches.md)** - **重点** - 6 个 Patch 的详细分析
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建系统适配
5. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用场景
6. **[05_API_Differences.md](05_API_Differences.md)** - API 兼容性分析
7. **[06_Security.md](06_Security.md)** - 安全风险分析

### 工作文档

- `_work/ASSESSMENT.md` - 项目评估结果
- `_work/NOTES.md` - 分析过程记录
- `_work/PLAN.md` - 任务进度计划

---

## 阅读路线

### 按角色阅读

#### 系统架构师
```
README.md → 01_Overview.md → 02_Patches.md → 04_Usage_in_OH.md
```

#### 构建工程师
```
README.md → 02_Patches.md → 03_Build_Integration.md → 06_Security.md
```

#### 应用开发者
```
README.md → 01_Overview.md → 05_API_Differences.md
```

#### 安全工程师
```
README.md → 02_Patches.md (CVE部分) → 06_Security.md
```

#### 维护者
```
README.md → 全部文档 → _work/ASSESSMENT.md
```

---

## 关键发现摘要

### 库基本信息

| 项目 | 内容 |
|------|------|
| **名称** | ncurses |
| **版本** | 6.5 |
| **许可证** | MIT |
| **构建系统** | Autotools (无 BUILD.gn) |
| **Patch 数量** | 6 个 |

### Patch 分类

| 类型 | 数量 | 代表 Patch |
|------|------|-----------|
| 终端类型增强 | 2 | kbs.patch, urxvt.patch |
| 构建系统适配 | 2 | libs.patch, config.patch |
| OH 平台适配 | 1 | cross_compile_support_ohos.patch |
| 安全修复 | 1 | CVE-2023-29491.patch |

### 安全状况

- ✅ **CVE-2023-29491**: 已修复
- ✅ **历史 CVE**: 全部已修复
- ✅ **OH Patch**: 未引入新风险

### 依赖关系

- ⚠️ **未在 BUILD.gn 中发现直接依赖**
- 可能用于开发工具、terminfo 数据库或 prebuilt 二进制

### API 兼容性

- ✅ **100% 与上游 6.5 兼容**
- ✅ 无 API 变更
- ✅ 无行为变更 (除 CVE 修复外)

---

## 核心结论

### 1. 适配质量

ncurses 的 OpenHarmony 适配是**高质量且审慎**的：

- 6 个 Patch 都有明确目的
- OH 特有适配与通用修复分离
- 包含关键安全修复

### 2. 维护建议

**升级时保留**:
- cross_compile_support_ohos.patch (平台支持)
- ncurses-config.patch (配置适配)
- ncurses-libs.patch (链接适配)

**可推向上游**:
- ncurses-kbs.patch
- ncurses-urxvt.patch

### 3. 待办事项

- [ ] 确认 ncurses 在 OH 中的实际使用场景
- [ ] 验证依赖者信息
- [ ] 确认 terminfo 数据库的部署位置

---

## 术语表

| 术语 | 说明 |
|------|------|
| **ncurses** | new curses，终端控制库 |
| **terminfo** | 终端能力数据库格式 |
| **TUI** | Text User Interface，文本用户界面 |
| **Patch** | 代码补丁/修改 |
| **CVE** | Common Vulnerabilities and Exposures，通用漏洞披露 |
| **spec 文件** | RPM 包配置文件 |
| **autotools** | GNU 构建系统 (autoconf/automake) |
| **cross-compile** | 交叉编译 |

---

## 更新历史

| 日期 | 版本 | 变更 |
|------|------|------|
| 2025-02-08 | 1.0 | 初始版本，完成全部分析文档 |

---

## 反馈与改进

如有问题或建议，请联系：
- OH 维护者: liyiming13@huawei.com
- 上游维护者: Thomas E. Dickey (dickey@invisible-island.net)

---

*本文档由 OpenHarmony Wiki Agent 自动生成*
