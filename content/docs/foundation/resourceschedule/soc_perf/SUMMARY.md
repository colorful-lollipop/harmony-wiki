# 文档导航

## 新人学习路线

建议按以下顺序阅读，逐步建立全局认知：

```
1. 快速入门 → 01_Overview.md
2. 架构设计 → 02_Architecture.md
3. 代码地图 → 03_CodeMap.md
4. 接口文档 → 04_Interface.md
5. 构建说明 → 07_Build.md
```

## 安全研究路线

针对安全审计场景，建议按以下优先级查阅：

```
1. 攻击面分析 → 05_AttackSurface.md
2. 安全风险评估 → 06_SecurityReview.md
3. 架构设计 → 02_Architecture.md（信任边界）
```

## 完整文档列表

### 入门与架构

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [README](README.md) | 文档说明与导航 | 所有用户 |
| [01_Overview](01_Overview.md) | 项目定位、能力边界、运行环境 | P0 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型 | P0 |
| [03_CodeMap](03_CodeMap.md) | 目录结构、核心文件定位 | P0 |

### 接口与安全

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [04_Interface](04_Interface.md) | IPC 接口、API 参考 | P1 |
| [05_AttackSurface](05_AttackSurface.md) | 外部输入、敏感操作、信任边界 | P0 |
| [06_SecurityReview](06_SecurityReview.md) | 风险分析、代码证据、修复建议 | P0 |

### 工程与实现

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [07_Build](07_Build.md) | GN Targets、编译产物、Feature 开关 | P2 |
| [08_Internals](08_Internals.md) | 核心类、资源生命周期 | P2 |

---

*导航更新时间：2026-02-07*
