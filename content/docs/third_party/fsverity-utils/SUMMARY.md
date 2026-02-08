# fsverity-utils Wiki 文档

## 文档概述

本文档描述了 **fsverity-utils** 库在 OpenHarmony 系统中的集成、适配和使用方式。

**fsverity-utils** 是 Linux kernel fs-verity 功能的用户空间工具集，提供文件完整性验证的完整解决方案。

---

## 阅读路线建议

### 快速入门（5 分钟）

| 读者角色 | 推荐阅读顺序 | 预计时间 |
|----------|--------------|----------|
| 应用开发者 | README.md → 04_Usage_in_OH.md | 5 分钟 |
| 构建工程师 | README.md → 03_Build_Integration.md | 5 分钟 |
| 安全工程师 | README.md → 06_Security.md | 5 分钟 |

### 完整阅读（30 分钟）

```
1. README.md          (5 min)  - 整体概述
2. 01_Overview.md     (5 min)  - 原始库功能
3. 02_Patches.md      (5 min)  - Patch 分析
4. 03_Build_Integration.md (5 min) - 构建适配
5. 04_Usage_in_OH.md (10 min) - OH 使用场景
6. 06_Security.md     (5 min)  - 安全考虑
```

### 专题深入

| 主题 | 关键文档 | 目标读者 |
|------|----------|----------|
| 升级上游版本 | 02_Patches.md, 03_Build_Integration.md | 维护者 |
| 新增依赖 | 03_Build_Integration.md, 04_Usage_in_OH.md | 架构师 |
| 安全审计 | 06_Security.md | 安全工程师 |
| API 使用 | 04_Usage_in_OH.md, 05_API_Differences.md | 应用开发者 |

---

## 文档索引

### 快速参考

- **[README.md](README.md)** - 项目总览、导航、快速开始
- **[SUMMARY.md](SUMMARY.md)** - 本文档，阅读路线建议

### 核心文档

| 编号 | 文档 | 内容摘要 | 更新频率 |
|------|------|----------|----------|
| 01 | [01_Overview.md](01_Overview.md) | 原始库功能、架构、API 概览 | 低 |
| 02 | [02_Patches.md](02_Patches.md) | OH Patch 分析、升级注意事项 | 中 |
| 03 | [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 配置、编译选项 | 中 |
| 04 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 中的使用场景、依赖关系图 | 中 |
| 05 | [05_API_Differences.md](05_API_Differences.md) | API 差异、兼容性说明 | 低 |
| 06 | [06_Security.md](06_Security.md) | 安全风险、漏洞修复建议 | 高 |

### 工作文档

| 编号 | 文档 | 内容摘要 |
|------|------|----------|
| - | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估原始记录 |
| - | [_work/NOTES.md](_work/NOTES.md) | 分析过程笔记 |
| - | [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |

---

## 与其他模块的关系

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 安全模块                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────────┐                                     │
│   │   hapsigner      │  ← 使用 fsverity 进行代码签名        │
│   └────────┬─────────┘                                     │
│            │                                               │
│            ▼                                               │
│   ┌──────────────────┐                                     │
│   │ code_signature   │  ← 核心依赖模块                      │
│   │  (base/security) │    - 本地代码签名                    │
│   │                  │    - 文件完整性验证                  │
│   └────────┬─────────┘                                     │
│            │                                               │
│            ▼                                               │
│   ┌──────────────────┐                                     │
│   │  fsverity-utils  │  ← 第三方库                          │
│   │ (third_party)    │    - Merkle 树计算                   │
│   │                  │    - 摘要签名                        │
│   └──────────────────┘                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 版本兼容性

| OH 版本 | fsverity-utils 版本 | 兼容性说明 |
|---------|---------------------|------------|
| 4.0+ | v1.6 / OH 3.1 | 当前版本 |
| 3.x | v1.5 | 历史版本 |

---

## 贡献指南

### 文档更新

1. **修改文档前**：查看 `_work/NOTES.md` 了解历史分析
2. **更新 Patch 信息**：确保与实际 Patch 文件一致
3. **更新依赖关系**：验证 BUILD.gn 引用是否正确

### 测试验证

修改后应运行以下测试：

```bash
# 代码签名模块测试
cd base/security/code_signature
# 运行相关单元测试
```

---

## 常见问题

### Q: 为什么没有 Patch 文件？

**A**: fsverity-utils 的上游代码设计为平台无关，仅依赖 OpenSSL。OH 适配仅通过 BUILD.gn 完成，无需修改源代码。

### Q: 如何升级上游版本？

**A**: 参考 02_Patches.md 的"升级建议"章节。由于无 OH Patch，升级流程相对简单。

### Q: 库的安全更新策略是什么？

**A**: 参考 06_Security.md。安全更新主要跟随 OpenSSL 的安全公告。

---

## 反馈与贡献

**文档问题**：请在 OpenHarmony docs 仓库提交 Issue

**代码问题**：请在 OpenHarmony third_party 仓库提交 Issue
