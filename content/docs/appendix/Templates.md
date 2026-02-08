# 文档模板

> 本文档提供 OpenHarmony 文档仓库的标准模板和使用指南。

## 模板列表

| 模板 | 用途 | 位置 |
|------|------|------|
| API 评审模板 | API 设计评审 | `zh-cn/design/API-Review-Template.md` |
| PR 模板 | Pull Request 说明 | `.gitcode/PULL_REQUEST_TEMPLATE.zh-CN.md` |
| Issue 模板 | GitCode Issue | `.gitcode/ISSUE_TEMPLATE/` |

---

## API 评审模板

**位置**：`zh-cn/design/API-Review-Template.md`

**用途**：API 设计文档的标准化评审模板。

### 模板结构

```markdown
# API 评审模板

## 基本信息

| 属性 | 值 |
|------|-----|
| API 名称 | {API名称} |
| 模块 | {所属模块} |
| 版本 | {API版本} |
| 评审人 | {评审人} |
| 评审日期 | {日期} |

## 设计说明

### 背景和目标

[描述 API 设计的背景和要解决的问题]

### API 签名

```typescript
// 或 C/C++
{API 签名}
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| param1 | Type | 是 | 说明 |

### 返回值

[返回值说明]

### 使用示例

```typescript
{示例代码}
```

## 评审项

### 功能性

- [ ] API 功能完整
- [ ] 命名规范符合项目要求
- [ ] 参数设计合理

### 安全性

- [ ] 参数校验完整
- [ ] 权限检查到位
- [ ] 无注入风险

### 性能

- [ ] 无性能瓶颈
- [ ] 异步操作合理

### 兼容性

- [ ] 向下兼容
- [ ] 无隐藏 breaking changes

## 评审结论

- [ ] 通过
- [ ] 有条件通过
- [ ] 不通过

## 备注

[其他说明]
```

---

## Pull Request 模板

**位置**：`.gitcode/PULL_REQUEST_TEMPLATE.zh-CN.md`

**用途**：标准化的 Pull Request 说明文档。

### 模板结构

```markdown
## 描述

[清晰描述本次 PR 的修改内容和目的]

## 修改类型

- [ ] Bug 修复
- [ ] 新功能
- [ ] 文档更新
- [ ] 重构
- [ ] 其他

## 影响范围

[描述本次修改影响的模块或文件]

## 测试情况

- [ ] 已本地测试
- [ ] 已添加测试用例
- [ ] 需要协助测试

## 相关信息

- Issue 编号：#{issue-number}
- 讨论链接：{url}
```

---

## Issue 模板

**位置**：`.gitcode/ISSUE_TEMPLATE/`

**用途**：标准化的 Issue 报告模板。

### 可用模板

| 模板名称 | 用途 |
|----------|------|
| Bug 报告 | 报告文档错误 |
| 功能请求 | 提出新文档需求 |
| 问题咨询 | 咨询技术问题 |

### Bug 报告模板结构

```markdown
## 问题描述

[清晰描述遇到的问题]

## 问题类型

- [ ] 内容错误（技术信息错误）
- [ ] 格式问题（排版、样式）
- [ ] 链接失效
- [ ] 示例代码错误
- [ ] 其他

## 复现步骤

1. [步骤1]
2. [步骤2]
3. [步骤3]

## 预期行为

[描述期望的正确行为]

## 实际行为

[描述实际发生的行为]

## 环境信息

- 浏览器/工具：[版本]
- 操作系统：[版本]
- 文档版本：OpenHarmony {版本}

## 相关截图

[如有截图，添加在此]

## 相关链接

[相关的文档链接或代码链接]
```

---

## 文档通用模板

### 发布说明模板

**路径**：`release-notes/OpenHarmony-v{版本}-{类型}.md`

```markdown
# OpenHarmony {版本} {类型}

## 版本信息

| 属性 | 值 |
|------|-----|
| 版本号 | {版本号} |
| API Level | {API Level} |
| 发布日期 | {日期} |
| 类型 | Release / Beta / LTS |

## 概述

[版本概述，1-2段文字]

## 主要特性

- 特性1
- 特性2
- 特性3

## 兼容性

[与其他版本的兼容性说明]

## 已知问题

[已知问题的列表]

## 升级说明

[升级指南和注意事项]

## 相关资源

- [完整更新日志链接]
- [API 参考链接]
- [迁移指南链接]
```

---

### 贡献者案例模板

**路径**：`third-party-cases/{案例名称}.md`

```markdown
# {案例名称}

## 案例简介

[简要描述案例的功能和用途]

## 适用场景

[说明适用该案例的场景]

## 实现原理

[简要说明实现原理]

## 代码示例

```typescript
// 代码示例
{代码内容}
```

## 运行效果

[描述运行效果或截图]

## 注意事项

[使用该案例需要注意的事项]

## 相关文档

[相关的官方文档链接]
```

---

## 编码规范模板

各语言的编码规范文档请参考：

| 语言 | 模板 |
|------|------|
| ArkTS | `zh-cn/contribute/OpenHarmony-ArkTS-coding-style-guide.md` |
| TypeScript/JavaScript | `zh-cn/contribute/OpenHarmony-Application-Typescript-JavaScript-coding-guide.md` |
| C/C++ | `zh-cn/contribute/OpenHarmony-cpp-coding-style-guide.md` |
| C | `zh-cn/contribute/OpenHarmony-c-coding-style.md` |
| HDF | `zh-cn/contribute/OpenHarmony-hdf-coding-guide.md` |

---

## 文档写作规范

### 基本要求

| 要求 | 说明 |
|------|------|
| 语言 | 中文文档使用中文标点 |
| 标题 | 使用 # ## ### #### 层级 |
| 代码块 | 使用 ``` 包裹，标注语言 |
| 链接 | 使用 Markdown 链接格式 |
| 图片 | 使用相对路径 |

### 推荐工具

| 工具 | 用途 |
|------|------|
| MarkdownLint | Markdown 格式检查 |
| 拼写检查 | 拼写错误检查 |
| 链接检查 | 死链检测 |

---

## 术语使用规范

### 官方术语表

| 术语 | 说明 |
|------|------|
| OpenHarmony | OpenHarmony 操作系统 |
| API | Application Programming Interface |
| SDK | Software Development Kit |
| Ability | 能力组件 |
| FA | Feature Ability |
| PA | Particle Ability |

**参考**：`zh-cn/glossary.md`

### 禁用词汇

| 禁用词 | 替代词 | 原因 |
|--------|--------|------|
| {禁用词} | {替代词} | {原因} |

---

## 审核检查清单

### 提交前检查

- [ ] Markdown 格式正确
- [ ] 链接有效
- [ ] 图片路径正确
- [ ] 术语使用规范
- [ ] 代码示例语法正确
- [ ] 无错别字

### PR 检查

- [ ] 遵循提交规范
- [ ] 填写 PR 模板
- [ ] 关联相关 Issue
- [ ] 通过 CI 检查

---

## 相关资源

| 资源 | 链接 |
|------|------|
| 写作指南 | [zh-cn/contribute/writing-instructions.md](../zh-cn/contribute/writing-instructions.md) |
| 贡献指南 | [05_Contribute.md](../05_Contribute.md) |
| 术语表 | [zh-cn/glossary.md](../zh-cn/glossary.md) |

---

*最后更新：2026-02-06*
