# 阅读路线建议

## 推荐阅读顺序

### 入门路线 (30 分钟)

1. **[README.md](README.md)** - 快速了解 MindSpore 在 OH 中的定位
2. **[01_Overview.md](01_Overview.md)** - 了解库的功能和 OH 适配
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 查看依赖关系和使用场景

### 开发者路线 (60 分钟)

完成入门路线后：

4. **[03_Build_Integration.md](03_Build_Integration.md)** - 深入理解构建系统
5. **[02_Patches.md](02_Patches.md)** - **核心文档** - 了解所有 Patch 的作用

### 安全维护路线 (45 分钟)

6. **[06_Security.md](06_Security.md)** - 安全补丁分析
7. **[05_API_Differences.md](05_API_Differences.md)** - API 变更记录

---

## 完整目录

### 基础信息

| 文件 | 内容摘要 | 必读 |
|------|----------|------|
| [README.md](README.md) | 项目概览、快速开始 | ✅ |
| [01_Overview.md](01_Overview.md) | 原始功能、OH 定位 | ✅ |
| [SUMMARY.md](SUMMARY.md) | 阅读导航 | ⏭️ |

### 核心文档

| 文件 | 内容摘要 | 重要性 |
|------|----------|--------|
| [02_Patches.md](02_Patches.md) | 40 个 Patch 详细分析 | ⭐⭐⭐ |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 构建适配 | ⭐⭐ |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系、使用场景 | ⭐⭐⭐ |

### 进阶文档

| 文件 | 内容摘要 | 目标读者 |
|------|----------|----------|
| [05_API_Differences.md](05_API_Differences.md) | API 差异、OH 新增接口 | API 开发者 |
| [06_Security.md](06_Security.md) | CVE 修复、安全建议 | 安全维护者 |

---

## 按角色推荐

### 应用开发者

> 需要在应用中集成 MindSpore 进行 AI 推理

**必读**:
- README.md
- 01_Overview.md
- 04_Usage_in_OH.md (重点: "NDK API 使用")
- 05_API_Differences.md

**参考**:
- [HarmonyOS AI 开发指南](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/mindspore-guidelines-based-native)

---

### 系统集成工程师

> 需要理解 MindSpore 如何与 OH 系统集成

**必读**:
- 03_Build_Integration.md (完整)
- 02_Patches.md (完整)
- 04_Usage_in_OH.md (重点: "依赖关系图")

**参考**:
- `_work/ASSESSMENT.md` - 完整评估报告

---

### 安全维护者

> 负责跟踪和更新安全补丁

**必读**:
- 06_Security.md (完整)
- 02_Patches.md (重点: "CVE 安全修复")
- `_work/ASSESSMENT.md` (重点: "安全补丁年度统计")

---

### 上游贡献者

> 想要将 OH 适配贡献回上游

**必读**:
- 02_Patches.md (重点: "OH 特定适配")
- 03_Build_Integration.md (重点: "OH 构建系统差异")

---

## 文档更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| v1.0 | 2025-02-07 | 初始版本，完整文档覆盖 |

---

## 反馈与贡献

如发现文档错误或有改进建议，请提交 Issue 到 OpenHarmony third_party_mindspore 仓库。
