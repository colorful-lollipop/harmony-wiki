# Telephony Data Storage Wiki - 文档导航

## 新人阅读路线（推荐顺序）

```
📖 建议阅读顺序:
1. → 01_Overview.md    (项目概览)
2. → 02_Directory_Structure.md (目录结构)
3. → 03_Architecture.md (架构说明)
4. → 04_DataShare_API.md (对外API)
5. → 05_Inner_API.md (内部API)
6. → 06_Build.md (构建系统)
7. → 07_Security.md (安全评审)
```

---

## 核心文档

### 入门指南

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README](README.md) | Wiki 说明、更新方式、生成信息 | ⭐⭐⭐ |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境、关键概念 | ⭐⭐⭐ |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构、模块职责 | ⭐⭐⭐ |

### 架构与 API

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [03_Architecture](03_Architecture.md) | 组件图、数据流、线程模型、时序图 | ⭐⭐⭐ |
| [04_DataShare_API](04_DataShare_API.md) | DataShare URIs、API 清单表、数据结构 | ⭐⭐⭐ |
| [05_Inner_API](05_Inner_API.md) | 模块接口、依赖方向、稳定性标注 | ⭐⭐ |
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链图 | ⭐ |

### 构建与部署

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [06_Build](06_Build.md) | GN targets、依赖关系、编译产物 | ⭐⭐⭐ |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 关键宏与 Feature Flags | ⭐ |

### 安全与运维

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [07_Security](07_Security.md) | 攻击面、风险点、修复建议 | ⭐⭐⭐ |

---

## 快速索引

### 按功能查找

| 功能 | 相关文档 |
|------|----------|
| SIM 卡数据存储 | [02_Directory_Structure](02_Directory_Structure.md), [04_DataShare_API](04_DataShare_API.md) |
| 短信/多媒体消息 | [02_Directory_Structure](02_Directory_Structure.md), [04_DataShare_API](04_DataShare_API.md) |
| PDP/APN 配置 | [02_Directory_Structure](02_Directory_Structure.md), [04_DataShare_API](04_DataShare_API.md) |
| 运营商密钥 | [02_Directory_Structure](02_Directory_Structure.md), [04_DataShare_API](04_DataShare_API.md) |
| 权限管理 | [04_DataShare_API](04_DataShare_API.md), [07_Security](07_Security.md) |
| 构建编译 | [06_Build](06_Build.md) |
| 安全加固 | [07_Security](07_Security.md) |

### 按角色查找

| 角色 | 推荐阅读 |
|------|----------|
| 新人开发者 | 01_Overview → 02_Directory_Structure → 03_Architecture |
| API 使用者 | 04_DataShare_API → 附录调用链 |
| 模块维护者 | 03_Architecture → 05_Inner_API → 06_Build |
| 安全评审人员 | 07_Security → 03_Architecture → 04_DataShare_API |
| 构建工程师 | 06_Build → Config_Flags |

---

## 文档更新记录

| 日期 | 变更 | 负责人 |
|------|------|--------|
| 2024-02-06 | 初始版本生成 | Wiki Generator |

---

## 贡献指南

### 如何贡献

1. 在 `wiki/_work/NOTES.md` 中记录新发现
2. 编辑对应的 Wiki 文档
3. 更新 SUMMARY.md 导航（如有必要）
4. 提交 PR 并关联相关 Issue

### 质量要求

- 所有关键结论需有代码证据（路径 + 符号 + 行号）
- 不引用测试代码作为业务证据
- API 变更需同步更新文档
- 安全相关变更需重新评估风险

---

*最后更新: 2024-02-06*
