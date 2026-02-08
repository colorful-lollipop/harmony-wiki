# 分布式数据对象部件 Wiki

> 本 Wiki 基于 OpenHarmony \`distributeddatamgr/data_object\` 仓库代码自动生成。

## 文档覆盖范围

### 已覆盖内容

| 文档 | 描述 | 状态 |
|------|------|------|
| [概览](./00_Overview.md) | 项目定位、核心能力、运行环境、关键概念 | ✅ 已更新 |
| [架构](./01_Architecture.md) | 组件图、数据流、线程模型、关键时序 | ✅ 已更新 |
| [代码地图](./02_CodeMap.md) | 目录结构、核心文件定位、代码导航图 | ✅ 已更新 |
| [接口文档](./03_Interface.md) | N-API、IPC 接口、错误码 | ✅ 已更新 |
| [攻击面分析](./04_AttackSurface.md) | 外部输入清单、敏感操作、信任边界 | ✅ 已更新 |
| [安全评审](./05_SecurityReview.md) | 5 类安全风险、可利用性评估、修复建议 | ✅ 已更新 |
| [构建与产物](./06_Build.md) | GN Targets、编译产物、运行时加载关系 | ✅ 已更新 |
| [内部实现](./07_Internals.md) | 核心类、内部 API 契约、资源生命周期 | ✅ 已更新 |
| [全站导航](./SUMMARY.md) | 双路线导航（新人/安全）、文档索引 | ✅ 已创建 |

### 未覆盖内容

| 内容 | 原因 |
|------|------|
| 协作编辑模块详细 API | 需要额外深入分析 |
| ETS/ANI 绑定详细规格 | 需要专门章节（待补充） |

---

## 文档更新方式

### 方式一：手动更新

直接编辑对应 \`.md\` 文件，遵循以下规范：

1. **关键结论必须可追溯**：每个重要声明需标注代码证据
   \`\`\`markdown
   - 结论描述
   - 证据：\`interfaces/innerkits/distributed_object.h:27\` (类定义)
   \`\`\`

2. **API 文档格式**：使用下表结构
   \`\`\`markdown
   | JS API | C++ 实现 | 参数 | 返回值 |
   \`\`\`

3. **代码证据格式**：\`路径:行号\` (仅主文件，详细的可在正文中展开)

### 方式二：重新生成

使用 Wiki 生成脚本重新扫描代码库：

\`\`\`bash
# 假设存在 generate_wiki.sh 脚本
./generate_wiki.sh --input /path/to/codebase --output ./wiki
\`\`\`

### 方式三：增量更新

当代码变更时，手动更新相关章节并更新本文档的"最后验证时间"。

---

## 生成信息

| 属性 | 值 |
|------|-----|
| 生成时间 | 2026-02-07 |
| 代码版本 | OpenHarmony master 分支 |
| 代码仓库 | \`foundation/distributeddatamgr/data_object\` |
| 生成工具 | Sisyphus Wiki Generator |
| 代码证据截止 | Phase 1 全局扫描完成 |

---

## 快速导航

\`\`\`mermaid
graph LR
    A[开始] --> B{我是?}
    B -->|新人学习者| C[全站导航 SUMMARY.md]
    B -->|安全研究员| D[项目概览 00_Overview.md]
    C --> E{学习目标?}
    E -->|快速入门| F[架构详解 01_Architecture.md]
    E -->|深度审计| G[攻击面分析 04_AttackSurface.md]
    F --> H[接口文档 03_Interface.md]
    G --> I[安全评审 05_SecurityReview.md]
\`\`\`

---

## 贡献指南

欢迎贡献和改进本 Wiki：

1. **发现文档错误**：直接在对应文件提交 Issue 或 PR
2. **补充新内容**：遵循现有格式，添加代码证据
3. **更新代码变更**：代码重构后同步更新相关章节

## 联系方式

- **仓库**：[distributeddatamgr_data_object](https://gitee.com/openharmony/distributeddatamgr_data_object)
- **组件名**：\`@ohos/data_object\`
- **子系统**：\`distributeddatamgr\`
