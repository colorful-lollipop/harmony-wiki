# 文档目录

本文档旨在全面说明 MarkupSafe 库在 OpenHarmony 生态系统中的集成、适配和使用情况。

## 核心文档

### 01_Overview.md —— 原始库简介

- 库名称、版本、许可证信息
- 原始功能详细描述
- 上游项目地址和社区资源
- MarkupSafe 在 OH 中的定位和作用

### 02_Patches.md —— Patch 详细分析

- **重点文档**：说明为什么 MarkupSafe 不需要任何 Patch
- Patch 清单表（空表，表明无 Patch）
- 无 Patch 原因的技术分析
- Patch 维护建议

### 03_Build_Integration.md —— OH 构建适配

- 构建系统集成方式
- 纯 Python 库的特殊处理
- 文件裁剪策略
- 版本管理说明

### 04_Usage_in_OH.md —— 依赖关系与使用

- 直接依赖者（Jinja2）详细分析
- 使用方式和场景
- 依赖关系图（Mermaid）
- 导入模式和接口使用

### 05_API_Differences.md —— API/接口差异

- OH 与上游的 API 对比
- 新增接口说明
- 行为变更记录
- 兼容性说明

### 06_Security.md —— 安全风险分析

- 已知 CVE 和修复状态
- OH 特定安全考量
- 安全升级策略建议

## 快速阅读指南

### 我需要了解...

| 场景 | 推荐阅读 |
|------|----------|
| 了解 MarkupSafe 是什么 | 01_Overview.md |
| 查找是否有 Patch 修改 | 02_Patches.md |
| 了解如何在 OH 中使用 | 04_Usage_in_OH.md |
| 了解构建配置 | 03_Build_Integration.md |
| 了解安全风险 | 06_Security.md |
| 查看完整技术评估 | _work/ASSESSMENT.md |

## 技术深度

### 依赖链路

```
应用层
  │
  ▼
OpenHarmony 模板渲染
  │
  ▼
Jinja2 模板引擎
  │
  ▼
MarkupSafe（安全转义层）
  │
  ▼
Python 标准库
```

### 关键文件

| 文件 | 说明 |
|------|------|
| `__init__.py` | 主模块，定义 Markup、escape 等核心接口 |
| `_speedups.c` | 可选 C 扩展，用于性能优化 |
| `_native.py` | 纯 Python 回退实现 |
| `bundle.json` | OH 组件描述文件 |

## 版本信息

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| 1.0 | 2024-01-01 | 初始文档版本 |
| 1.1 | 2024-XX-XX | 更新版本信息 |

## 贡献指南

如需补充或修正本文档，请：

1. 在对应的 _work/NOTES.md 中记录发现
2. 更新相关文档
3. 提交 Pull Request

## 许可证

本文档遵循与 MarkupSafe 相同的 BSD 3-Clause License。
