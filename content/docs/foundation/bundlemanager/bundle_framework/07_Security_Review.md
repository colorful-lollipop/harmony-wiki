# 安全概览

## 概述

本文档提供 Bundle Framework 安全相关的文档索引和安全最佳实践总结。

**注意**：详细的安全分析已拆分到以下专门文档，请根据需要查阅：

- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析（外部输入、敏感操作、信任边界）
- [06_SecurityReview](06_SecurityReview.md) - 安全风险评估（漏洞详情、修复建议）

---

## 安全文档索引

| 主题 | 文档 | 内容 |
|------|------|------|
| 攻击面分析 | [05_AttackSurface](05_AttackSurface.md) | 外部输入清单、敏感操作、信任边界图 |
| 风险评估 | [06_SecurityReview](06_SecurityReview.md) | 5 个风险点详情、修复建议 |
| 架构安全 | [02_Architecture](02_Architecture.md) | 多进程架构、IPC 安全机制 |
| FAQ 安全 | [08_FAQ](08_FAQ.md) | 安全相关常见问题 |

---

## 快速安全检查清单

### 开发阶段

- [ ] 使用 `IsValidBundleName()` 验证包名
- [ ] 使用 `NormalizePath()` 规范化路径
- [ ] 使用 `O_NOFOLLOW` 打开文件
- [ ] 遵循最小权限原则

### 审计阶段

- [ ] 检查 N-API 参数校验
- [ ] 检查 IPC 权限验证
- [ ] 检查文件操作路径验证
- [ ] 检查签名验证流程

---

## 相关安全组件

| 组件 | 用途 | 文档位置 |
|------|------|----------|
| `access_token` | 权限管理 | [06_SecurityReview](06_SecurityReview.md) |
| `appverify` | 应用签名验证 | [06_SecurityReview](06_SecurityReview.md) |
| `security_code_signature` | 代码签名 | [06_SecurityReview](06_SecurityReview.md) |

---

## 延伸阅读

- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析
- [06_SecurityReview](06_SecurityReview.md) - 安全风险评估
