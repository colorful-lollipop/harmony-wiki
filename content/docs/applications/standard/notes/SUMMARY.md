# 文档导航

## 新人阅读路线

建议阅读顺序（按依赖关系）：

```
1. [README.md](README.md)          ← 文档概览
   ↓
2. [00_Overview.md](00_Overview.md)  ← 项目定位与核心能力
   ↓
3. [01_Directory_Structure.md](01_Directory_Structure.md)  ← 目录结构
   ↓
4. [02_Architecture.md](02_Architecture.md)  ← 架构设计
   ↓
5. [03_Inner_API.md](03_Inner_API.md)  ← 内部 API
   ↓
6. [04_GN_Targets.md](04_GN_Targets.md)  ← 构建配置
   ↓
7. [05_Build_Artifacts.md](05_Build_Artifacts.md)  ← 编译产物
   ↓
8. [06_Security_Review.md](06_Security_Review.md)  ← 安全评估
   ↓
9. [07_Troubleshooting.md](07_Troubleshooting.md)  ← 问题排查
```

## 完整文档列表

### 基础信息

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README.md](README.md) | 文档说明与覆盖范围 | ⭐⭐⭐ |
| [SUMMARY.md](SUMMARY.md) | 全站导航 | ⭐⭐⭐ |
| [00_Overview.md](00_Overview.md) | 项目定位、核心能力、运行环境 | ⭐⭐⭐ |

### 结构与架构

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责 | ⭐⭐⭐ |
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型 | ⭐⭐⭐ |
| [03_Inner_API.md](03_Inner_API.md) | 模块接口、依赖方向 | ⭐⭐ |

### 构建与部署

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [04_GN_Targets.md](04_GN_Targets.md) | GN targets 列表与依赖 | ⭐⭐ |
| [05_Build_Artifacts.md](05_Build_Artifacts.md) | 编译产物与安装路径 | ⭐⭐ |

### 评估与分析

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [06_Security_Review.md](06_Security_Review.md) | 安全风险评估 | ⭐⭐ |
| [07_Troubleshooting.md](07_Troubleshooting.md) | 常见问题与调试 | ⭐⭐ |

## 快速跳转

### 按功能查找

- **数据持久化**: [01_Directory_Structure.md](01_Directory_Structure.md) → `common/utils` 模块
- **UI 组件**: [01_Directory_Structure.md](01_Directory_Structure.md) → `features` 模块
- **页面路由**: [02_Architecture.md](02_Architecture.md) → 页面导航
- **数据库操作**: [03_Inner_API.md](03_Inner_API.md) → RdbStoreUtil
- **构建配置**: [04_GN_Targets.md](04_GN_Targets.md)
- **安全风险**: [06_Security_Review.md](06_Security_Review.md)

### 按代码位置查找

| 模块 | 主要文档 |
|------|----------|
| `product/default` | [00_Overview.md](00_Overview.md), [02_Architecture.md](02_Architecture.md) |
| `common/utils` | [01_Directory_Structure.md](01_Directory_Structure.md), [03_Inner_API.md](03_Inner_API.md) |
| `features` | [01_Directory_Structure.md](01_Directory_Structure.md) |

## 贡献指南

如需修改本文档：

1. 确保修改基于代码证据（文件路径 + 行号）
2. 更新相关章节后检查 SUMMARY.md 链接
3. 保持术语统一
4. 提交前运行自检（待实现）
