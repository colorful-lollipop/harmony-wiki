# 贡献指南

> 本文档描述 OpenHarmony 文档仓库的贡献方式、审批流程、质量标准和最佳实践。

## 贡献方式

OpenHarmony 文档仓库欢迎以下几种贡献方式：

| 贡献方式 | 描述 | 适合人群 |
|----------|------|----------|
| 📝 **文档评价** | 对现有文档进行评价，提出改进建议 | 所有用户 |
| ✏️ **简单更改** | 修正错别字、语法错误、格式问题 | 初学者 |
| 🐛 **反馈问题** | 反馈文档中的错误、过时信息 | 所有用户 |
| 📚 **原创内容** | 贡献新的教程、指南、案例 | 资深贡献者 |

**证据**：`README.md:62`

## 贡献渠道

### 1. 邮件列表

| 属性 | 值 |
|------|-----|
| 地址 | docs@openharmony.io |
| 用途 | 讨论文档相关问题、反馈建议 |
| 响应时间 | 工作日 1-3 天 |

**证据**：`README.md:66`

### 2. Zulip 群组

| 属性 | 值 |
|------|-----|
| 平台 | Zulip |
| 组名 | documentation_sig |
| 用途 | 实时讨论、问题解答 |

**证据**：`README.md:68`

### 3. GitCode Issues

**位置**：https://gitee.com/openharmony/docs.openharmony.cn/issues

**用途**：
- 报告文档错误
- 提出新文档需求
- 讨论文档改进

### 4. Pull Request

**位置**：https://gitee.com/openharmony/docs.openharmony.cn/pulls

**用途**：直接贡献代码（文档修改）

## 贡献流程

### 1. Fork 仓库

```bash
# 1. 在 Gitee 上 Fork 仓库
# 访问：https://gitee.com/openharmony/docs.openharmony.cn

# 2. 克隆到本地
git clone https://gitee.com/{your-username}/docs.openharmony.cn.git

# 3. 添加上游仓库
git remote add upstream https://gitee.com/openharmony/docs.openharmony.cn.git
```

### 2. 创建分支

```bash
# 创建并切换到新分支
git checkout -b feature/your-feature-name

# 或修复问题
git checkout -b fix/issue-description
```

### 3. 进行更改

**遵循的规范**：

| 规范类型 | 说明 | 证据 |
|----------|------|------|
| 写作指南 | `zh-cn/contribute/writing-instructions.md` | 文件存在 |
| 编码规范 | 参考对应语言的编码规范文档 | `zh-cn/contribute/` |
| API 评审 | 新 API 文档需使用评审模板 | `zh-cn/design/API-Review-Template.md` |

### 4. 提交更改

```bash
# 查看更改
git status

# 添加更改的文件
git add .

# 提交（遵循提交规范）
git commit -m "docs: 简要描述修改内容

- 修改点1
- 修改点2

Closes #issue-number"
```

### 5. 推送并创建 PR

```bash
# 推送到你的 Fork
git push origin feature/your-feature-name

# 在 Gitee 上创建 Pull Request
```

## 提交规范

### 提交信息格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type 类型

| 类型 | 描述 | 示例 |
|------|------|------|
| `docs` | 文档修改 | `docs(readme): update quick start guide` |
| `fix` | 修复错误 | `fix(glossary): correct typo in term definition` |
| `feat` | 新增内容 | `feat(contribute): add new coding guide` |
| `refactor` | 重构整理 | `refactor(release-notes): reorganize version list` |
| `sync` | 同步翻译 | `sync(en): translate new articles` |

### Scope 范围

| 范围 | 描述 |
|------|------|
| `readme` | 入门指南模块 |
| `application-dev` | 应用开发模块 |
| `device-dev` | 设备开发模块 |
| `design` | 设计规范模块 |
| `contribute` | 贡献指南模块 |
| `release-notes` | 发布说明模块 |

## 中英文同步

### 同步原则

| 原则 | 说明 |
|------|------|
| 一一对应 | 英文文档有中文版本，中文有英文版本 |
| 同步更新 | 新增或修改内容需要同步翻译 |
| 版本对应 | 同一版本的中英文发布说明同步 |

### 同步检查清单

- [ ] 新增文档创建中英文两个版本
- [ ] 修改内容同步更新对应语言版本
- [ ] 术语表 `glossary.md` 保持同步
- [ ] 发布说明同步创建对应版本

## 质量标准

### 文档检查项

| 检查项 | 说明 | 优先级 |
|--------|------|--------|
| 链接有效 | 检查内部链接和外部链接 | 🔴 高 |
| 图片有效 | 图片路径正确、Alt 文本完整 | 🔴 高 |
| 格式一致 | 遵循文档模板和格式规范 | 🟡 中 |
| 术语统一 | 使用官方术语表 | 🟡 中 |
| 示例完整 | 代码示例可运行、有注释 | 🟡 中 |
| 过期信息 | 标注版本号、更新时间 | 🟢 低 |

### 编码规范文档

| 语言 | 文档 | 证据 |
|------|------|------|
| ArkTS | `zh-cn/contribute/OpenHarmony-ArkTS-coding-style-guide.md` | ✅ |
| TypeScript/JavaScript | `zh-cn/contribute/OpenHarmony-Application-Typescript-JavaScript-coding-guide.md` | ✅ |
| C/C++ | `zh-cn/contribute/OpenHarmony-cpp-coding-style-guide.md` | ✅ |
| C | `zh-cn/contribute/OpenHarmony-c-coding-style.md.md` | ✅ |
| HDF | `zh-cn/contribute/OpenHarmony-hdf-coding-guide.md` | ✅ |
| 安全编码 | `zh-cn/contribute/OpenHarmony-c-cpp-secure-coding-guide.md` | ✅ |

## 审批流程

### PR 审批流程

```
1. 提交者
   └── 创建 PR，填写 PR 模板
   
2. 自动化检查
   ├── CI 检查（链接、格式、拼写）
   └── 标记需要审查
   
3. 人工审查
   ├── 模块负责人审查
   └── 必要时征询专家意见
   
4. 合并
   ├── 通过审查后合并
   └── 自动触发发布
```

### 审批标准

| 标准 | 说明 |
|------|------|
| 内容准确 | 技术信息准确、示例正确 |
| 格式规范 | 符合文档模板和格式要求 |
| 术语一致 | 使用官方术语 |
| 链接有效 | 无死链 |
| 翻译完整 | 中英文同步 |

## 第三方组件

### 许可证说明

**位置**：`zh-cn/contribute/第三方开源软件及许可证说明.md`

**说明**：文档中引用的第三方开源组件需要说明其许可证。

**证据**：`README.md:56`

### 许可证检查

| 检查项 | 说明 |
|--------|------|
| 许可证声明 | 组件头部包含许可证声明 |
| 许可证兼容 | 许可证与项目兼容（CC BY 4.0） |
| 第三方文件 | 完整列出所有第三方组件 |

## 常见问题

### Q1: 如何报告文档错误？

**A**: 通过以下方式：
1. GitCode Issues：报告错误并标记 `bug`
2. 邮件列表：发送邮件描述问题
3. Zulip 群组：实时反馈

### Q2: 新增文档需要什么审批？

**A**: 
- 小改动（错别字、格式）：直接提交 PR
- 新增章节/模块：创建 Issue 讨论后提交 PR
- 重大变更：需要 SIG 审批

### Q3: 中英文同步需要多长时间？

**A**: 
- 简单修改：PR 合并后自动同步
- 新增内容：提交翻译 PR，通常 1-2 周内完成

### Q4: 如何贡献第三方案例？

**A**: 
1. 创建 Issue 讨论案例内容
2. 提交 PR，包含：
   - 案例说明文档
   - 代码示例
   - 运行说明

---

## 相关资源

### 官方资源

| 资源 | 链接 |
|------|------|
| OpenHarmony 官网 | https://www.openharmony.cn/ |
| 代码仓库 | https://gitee.com/openharmony |
| 邮件列表 | docs@openharmony.io |
| Zulip | documentation_sig |

### 文档资源

| 资源 | 链接 |
|------|------|
| 贡献指南 | [zh-cn/contribute/参与贡献.md](../zh-cn/contribute/参与贡献.md) |
| 文档贡献说明 | [zh-cn/contribute/贡献文档.md](../zh-cn/contribute/贡献文档.md) |
| 写作指南 | [zh-cn/contribute/writing-instructions.md](../zh-cn/contribute/writing-instructions.md) |
| 第三方许可证 | [zh-cn/contribute/第三方开源软件及许可证说明.md](../zh-cn/contribute/第三方开源软件及许可证说明.md) |

---

**相关文档**：
- [项目概览](01_Overview.md) - 了解项目背景
- [目录结构](02_Structure.md) - 熟悉仓库结构
- [文档内容体系](03_Content.md) - 了解文档分类
- [构建与发布](04_Build.md) - 了解发布流程

---

*最后更新：2026-02-06*
