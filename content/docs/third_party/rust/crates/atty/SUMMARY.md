# 阅读路线建议

## 推荐阅读顺序

### 1. 快速了解（5 分钟）
- [README.md](./README.md) - 获取库的基本信息和 OH 适配概况

### 2. 深度理解（15 分钟）
按照以下顺序阅读：

1. **[01_Overview.md](./01_Overview.md)**
   - 了解原始库的功能定位
   - 理解该库在 OH 中的作用

2. **[02_Patches.md](./02_Patches.md)** ⭐ 核心文档
   - **重点阅读**: 理解为什么没有 Patch
   - 学习如何判断一个库是否需要 OH 适配
   - 了解原生兼容库的特征

3. **[03_Build_Integration.md](./03_Build_Integration.md)**
   - 了解 BUILD.gn 的标准配置
   - 学习 Rust crate 在 OH 中的集成方式

### 3. 按需阅读（可选）

- **[04_Usage_in_OH.md](./04_Usage_in_OH.md)**
  - 了解该库在 OH 中的使用场景
  - 查看依赖关系图

- **[06_Security.md](./06_Security.md)**
  - 了解安全状况
  - 获取升级建议

## 不同角色的阅读重点

### 开发者（使用该库）
> 关注：如何使用、API 说明

推荐阅读：
1. README.md
2. 01_Overview.md（功能介绍部分）
3. 04_Usage_in_OH.md

### 维护者（升级库版本）
> 关注：Patch 状态、升级风险

推荐阅读：
1. 02_Patches.md ⭐（重点：无 Patch 说明）
2. 06_Security.md
3. 03_Build_Integration.md（版本号配置）

### 架构师（技术选型）
> 关注：设计思路、平台兼容性

推荐阅读：
1. 01_Overview.md
2. 02_Patches.md
3. 04_Usage_in_OH.md
4. 06_Security.md

## 关键结论速览

| 问题 | 答案 |
|------|------|
| 是否需要 Patch？ | 否，原生兼容 |
| 升级难度？ | 低，无障碍 |
| 安全状况？ | 良好，历史无严重 CVE |
| 典型用途？ | CLI 工具终端检测 |
