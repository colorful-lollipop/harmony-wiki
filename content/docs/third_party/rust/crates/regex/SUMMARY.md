# 阅读路线指南

本文档帮助不同角色的读者快速找到所需信息。

## 读者类型与推荐阅读顺序

### 集成开发者

**目标**：了解如何依赖和使用 regex 库

推荐阅读顺序：

1. **README.md** - 了解库的基本信息和 OH 适配特点
2. **04_Usage_in_OH.md** - 查看依赖声明方式和使用场景
3. **03_Build_Integration.md** - 了解构建配置和可选特性

### 构建系统维护者

**目标**：理解 regex 库的构建配置和依赖关系

推荐阅读顺序：

1. **03_Build_Integration.md** - 详细分析 BUILD.gn 配置
2. **04_Usage_in_OH.md** - 查看依赖图和依赖者列表
3. **02_Patches.md** - 确认无 Patch 影响的构建

### 安全审计人员

**目标**：评估 regex 库的安全风险

推荐阅读顺序：

1. **06_Security.md** - 安全风险分析
2. **01_Overview.md** - 了解库的基本功能和定位
3. **05_API_Differences.md** - 确认是否有 OH 定制的 API

### 版本升级负责人

**目标**：了解升级 regex 库版本时的注意事项

推荐阅读顺序：

1. **02_Patches.md** - 确认 Patch 回归风险（本库无 Patch）
2. **03_Build_Integration.md** - 对比配置差异
3. **05_API_Differences.md** - API 变更检查

## 文档结构

```
regex/
├── README.md              # 快速概览和导航
├── SUMMARY.md             # 本文档，阅读路线指南
├── _work/
│   ├── ASSESSMENT.md      # 项目评估报告（内部工作文档）
│   ├── NOTES.md           # 分析过程记录（待创建）
│   └── PLAN.md            # 任务进度（待创建）
├── 01_Overview.md         # 库功能介绍和 OH 定位
├── 02_Patches.md          # Patch 详细分析
├── 03_Build_Integration.md # BUILD.gn 构建配置
├── 04_Usage_in_OH.md      # 依赖关系和使用场景
├── 05_API_Differences.md  # API 差异分析
└── 06_Security.md         # 安全风险分析
```

## 关键信息速查

### 无 Patch 声明

> **重要**：regex 库在 OpenHarmony 中没有使用任何 Patch 文件。
>
> 这意味着：
> - 版本升级时无需担心 Patch 回归
> - 功能与上游完全一致
> - 构建配置是唯一需要关注的差异点

### 核心依赖链

```
regex
├── aho-corasick（多模式匹配）
├── memchr（字节搜索）
└── regex-syntax（语法解析）
```

### 主要依赖者

| 组件 | 用途 |
|------|------|
| bindgen | 解析 C/C++ 头文件 |
| env_logger | 日志模式过滤 |

## 版本信息速查

| 属性 | 值 |
|------|-----|
| 上游版本 | 1.7.1 |
| OH 版本 | 6.1 |
| Rust Edition | 2018 |
| 许可证 | Apache 2.0 / MIT |

## 更新日志

- **v6.1** (OH 版本): 初始适配版本，基于 regex 1.7.1
