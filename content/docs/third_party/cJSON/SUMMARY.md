# 文档阅读指南

## 文档结构

```
wiki/
├── README.md              # 库概览和快速导航
├── SUMMARY.md             # 本文档，阅读指南
├── 01_Overview.md         # 原始库简介
├── 02_Patches.md          # Patch 分析
├── 03_Build_Integration.md # 构建适配详解
├── 04_Usage_in_OH.md      # 使用场景和依赖
├── 05_API_Differences.md  # API 差异
├── 06_Security.md         # 安全风险分析
└── _work/
    ├── ASSESSMENT.md      # 项目评估报告
    ├── NOTES.md           # 分析记录
    └── PLAN.md            # 任务进度
```

## 阅读路线建议

### 快速了解（5 分钟）

1. 阅读 [README.md](README.md) 了解库的基本信息
2. 阅读本指南，确定需要深入的部分

### 深入理解（15-30 分钟）

#### 开发者（需要适配或修改）

1. 阅读 [01_Overview.md](01_Overview.md) 了解原始功能
2. 阅读 [03_Build_Integration.md](03_Build_Integration.md) 理解构建配置
3. 阅读 [04_Usage_in_OH.md](04_Usage_in_OH.md) 了解使用场景
4. 参考 [02_Patches.md](02_Patches.md) 和 [05_API_Differences.md](05_API_Differences.md) 确认兼容性

#### 维护者（需要升级版本）

1. 阅读 [03_Build_Integration.md](03_Build_Integration.md) 了解 OH 适配
2. 阅读 [06_Security.md](06_Security.md) 评估安全风险
3. 参考 [02_Patches.md](02_Patches.md) 确认无 Patch 需要合并

#### 安全审计

1. 直接阅读 [06_Security.md](06_Security.md)
2. 参考 [ASSESSMENT.md](_work/ASSESSMENT.md) 获取完整评估信息

## 关键信息速查

### 无 Patch

本库**未应用任何 Patch**，这是因为：
- cJSON 本身具有良好的跨平台兼容性
- OH 仅使用了基础功能
- 所有适配通过 BUILD.gn 配置完成

### 主要适配点

| 适配项 | 说明 |
|-------|------|
| 嵌套深度限制 | 128（上游默认 1000） |
| 栈保护 | PAC 保护 |
| 安装目标 | system/updater 分区 |

### 主要使用方

| 模块 | 用途 |
|-----|------|
| bundle_lite | Bundle 配置解析 |
| global_resource_tool | 资源配置工具 |
| previewer/util | IDE 预览器 |

## 术语表

| 术语 | 说明 |
|-----|------|
| **上游 (Upstream)** | 原始开源项目 |
| **Patch** | 代码修改补丁 |
| **GN** | OpenHarmony 构建系统 |
| **PAC** | Pointer Authentication Code，栈保护技术 |
