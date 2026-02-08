# 文档阅读路线建议

本文档为 OpenHarmony 第三方库 mksh 的完整技术文档。以下提供多种阅读路线，适应不同角色的需求。

## 阅读路线

### 路线 1：快速概览（5 分钟）

适合：初次接触该库，想了解基本情况的开发者

1. [README.md](README.md) - 库概览和适配概述（约 2 分钟）
2. [01_Overview.md](01_Overview.md) - 原始库简介（约 3 分钟）

### 路线 2：技术深入（15-20 分钟）

适合：需要理解 OH 适配细节的开发者

1. [README.md](README.md) - 库概览（约 2 分钟）
2. [01_Overview.md](01_Overview.md) - 原始库简介（约 3 分钟）
3. [02_Patches.md](02_Patches.md) - OH 适配代码详解（约 5 分钟）
4. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置（约 3 分钟）
5. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用方式（约 2 分钟）

### 路线 3：完整技术文档（30 分钟）

适合：需要全面理解该库在 OH 中的实现的开发者

1. [README.md](README.md) - 库概览（约 2 分钟）
2. [01_Overview.md](01_Overview.md) - 原始库简介（约 3 分钟）
3. [02_Patches.md](02_Patches.md) - OH 适配代码详解（约 8 分钟）
4. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置（约 5 分钟）
5. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系和使用（约 5 分钟）
6. [06_Security.md](06_Security.md) - 安全分析（约 5 分钟）
7. [05_API_Differences.md](05_API_Differences.md) - API 差异（如有）（约 2 分钟）

### 路线 4：构建系统专项（10 分钟）

适合：构建系统开发者或维护者

1. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置（约 5 分钟）
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系（约 3 分钟）
3. 参考 [mksh.gni](../mksh.gni) 和 [BUILD.gn](../BUILD.gn) 源文件（约 2 分钟）

### 路线 5：安全审计专项（15 分钟）

适合：安全工程师或进行安全审计的开发者

1. [README.md](README.md) - 库概览（约 1 分钟）
2. [01_Overview.md](01_Overview.md) - 原始库简介（约 2 分钟）
3. [06_Security.md](06_Security.md) - 安全风险分析（约 8 分钟）
4. [02_Patches.md](02_Patches.md) - 适配代码安全审查（约 4 分钟）

## 角色专项路线

### 系统集成开发者

推荐顺序：
1. [README.md](README.md)
2. [04_Usage_in_OH.md](04_Usage_in_OH.md)
3. [01_Overview.md](01_Overview.md)
4. 参考实际使用示例

### 构建系统维护者

推荐顺序：
1. [03_Build_Integration.md](03_Build_Integration.md)
2. [02_Patches.md](02_Patches.md)（构建相关部分）
3. 直接分析 [BUILD.gn](../BUILD.gn) 和 [mksh.gni](../mksh.gni)

### 上游升级负责人

推荐顺序：
1. [02_Patches.md](02_Patches.md) - 理解所有 OH 适配
2. [06_Security.md](06_Security.md) - 确认安全修复
3. [03_Build_Integration.md](03_Build_Integration.md) - 检查构建差异
4. 创建升级计划并验证

## 文档更新日志

| 日期 | 更新内容 | 更新人 |
|------|----------|--------|
| 2026-02-08 | 初始版本创建 | OpenHarmony Doc Agent |
