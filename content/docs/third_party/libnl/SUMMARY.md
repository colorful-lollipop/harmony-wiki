# 文档阅读路线

## 推荐阅读顺序

### 1. 快速了解（5分钟）
1. [README.md](README.md) - 了解库的基本定位和文档结构
2. [01_Overview.md](01_Overview.md) - 了解原始库功能和 OH 中的定位

### 2. 深度理解（30分钟）
3. [02_Patches.md](02_Patches.md) - **重点阅读**：理解 OH 对 libnl 的定制化修改
   - Patch 清单表
   - 每个 Patch 的修改目的和 OH 价值
   - 升级注意事项

### 3. 系统集成（20分钟）
4. [03_Build_Integration.md](03_Build_Integration.md) - 理解 OH 构建适配
   - BUILD.gn 结构
   - 特殊编译选项
   - install.sh 工作流程

5. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 理解依赖关系
   - 谁在依赖 libnl
   - 典型使用场景
   - 依赖关系图

### 4. 安全与维护（15分钟）
6. [06_Security.md](06_Security.md) - 了解安全风险
7. [05_API_Differences.md](05_API_Differences.md) - 了解 API 差异（如适用）

## 按角色阅读

### 如果你是...

**系统集成工程师**
- 必读: [02_Patches.md](02_Patches.md)（Patch 分析）
- 必读: [03_Build_Integration.md](03_Build_Integration.md)（构建集成）
- 选读: [04_Usage_in_OH.md](04_Usage_in_OH.md)（依赖关系）

**安全工程师**
- 必读: [06_Security.md](06_Security.md)（安全分析）
- 必读: [02_Patches.md](02_Patches.md) 中的安全修复部分

**升级维护工程师**
- 必读: [02_Patches.md](02_Patches.md) 中的升级建议
- 必读: [03_Build_Integration.md](03_Build_Integration.md)
- 参考: `_work/ASSESSMENT.md`（评估报告）

**驱动开发者**
- 必读: [04_Usage_in_OH.md](04_Usage_in_OH.md)（使用示例）
- 参考: [02_Patches.md](02_Patches.md) 中内核版本适配部分

## 关键信息速查

| 问题 | 答案位置 |
|------|---------|
| 有哪些 Patch？ | [02_Patches.md](02_Patches.md) Patch 清单表 |
| Patch 修改了什么？ | [02_Patches.md](02_Patches.md) 各 Patch 详细分析 |
| 如何构建？ | [03_Build_Integration.md](03_Build_Integration.md) |
| 谁在使用？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) 依赖者列表 |
| 升级要注意什么？ | [02_Patches.md](02_Patches.md) 升级建议 |
| 有什么安全风险？ | [06_Security.md](06_Security.md) |
