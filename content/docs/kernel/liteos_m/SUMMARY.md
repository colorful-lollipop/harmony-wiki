# 全文导航 - SUMMARY

本文档为 OpenHarmony LiteOS-M 内核 Wiki 的导航页面。

## 新人阅读顺序（推荐）

```
1. README.md           → 了解 Wiki 整体结构
2. 01_Overview.md      → 项目定位、核心能力
3. 02_Directory_Structure.md → 目录结构与模块职责
4. 03_Architecture.md  → 架构设计
5. 04_Kernel_API.md    → 内核 API（LOS_*）
6. 05_KAL.md           → 内核抽象层
7. 06_Components.md     → 可选组件
8. 07_Build.md         → 构建与编译产物
9. 08_Security.md      → 安全评审
```

## 完整文档列表

### 基础文档

| 文档 | 说明 |
|------|------|
| [README](README.md) | Wiki 说明、覆盖范围、更新方式 |
| [SUMMARY](SUMMARY.md) | 本页面，全文导航 |

### 核心文档

| 文档 | 说明 |
|------|------|
| [01_Overview](01_Overview.md) | 项目定位、边界、核心能力、运行环境 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构、模块职责 |
| [03_Architecture](03_Architecture.md) | 架构图、数据流、线程模型 |
| [04_Kernel_API](04_Kernel_API.md) | LOS_* 内核 API 清单 |
| [05_KAL](05_KAL.md) | CMSIS/POSIX API 层 |
| [06_Components](06_Components.md) | 可选组件说明 |
| [07_Build](07_Build.md) | GN Targets、编译产物 |
| [08_Security](08_Security.md) | 安全风险评审 |
| [09_FAQ](09_FAQ.md) | 常见问题与调试指南 |

## 快速跳转

- **构建入口**: [07_Build.md](07_Build.md) → `BUILD.gn`
- **API 清单**: [04_Kernel_API.md](04_Kernel_API.md)
- **安全相关**: [08_Security.md](08_Security.md)
- **代码证据**: 各文档均标注了代码路径与行号
