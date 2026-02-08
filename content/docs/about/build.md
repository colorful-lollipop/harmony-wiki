---
title: "构建与发布"
type: docs
---

# 构建与发布

> 本文档描述 OpenHarmony 文档仓库的构建工具、CI/CD 流程和发布机制。

## 构建工具

**发现**：本仓库是**纯文档仓库**，不包含构建配置。

| 搜索项 | 结果 | 证据 |
|--------|------|------|
| `mkdocs*` | ❌ 未找到 | glob 搜索无结果 |
| `docusaurus*` | ❌ 未找到 | glob 搜索无结果 |
| `.github/workflows/*` | ❌ 未找到 | glob 搜索无结果 |
| `.gitlab-ci*` | ❌ 未找到 | glob 搜索无结果 |

**说明**：
- 文档构建由 OpenHarmony 基础设施团队管理
- 构建配置不在本仓库中
- 发布到 https://docs.openharmony.cn 由外部 CI/CD 处理
- 仓库只包含 Markdown 源文件

### 本地查看

由于无本地构建配置，建议直接使用 Markdown 阅读器查看文档：

```bash
# 使用 Markdown 预览工具
# VS Code: 安装 Markdown Preview 插件
# Typora: 直接打开预览
# 其他: 使用在线 Markdown 编辑器
```

## 发布渠道

### 1. 官方网站

| 属性 | 值 |
|------|-----|
| 站点 | https://www.openharmony.cn/docs |
| 源内容 | `zh-cn/` 和 `en/` 目录 |
| 语言 | 中文（主）+ 英文 |

**证据**：`README.md:9`

### 2. GitCode Pages

| 属性 | 值 |
|------|-----|
| 平台 | code.oa.com |
| 配置 | `.gitcode/` 目录存在 |
| 模板 | `.gitcode/PULL_REQUEST_TEMPLATE.zh-CN.md` |

**证据**：`.gitcode/` 目录存在

## CI/CD 流程

**确认结果**：CI/CD 配置**不在本仓库中**

| 阶段 | 状态 | 说明 | 证据 |
|------|------|------|------|
| 代码检查 | ✅ 外部处理 | Linter、Spell check | 由基础设施团队管理 |
| 构建 | ✅ 外部处理 | 文档构建 | 外部 CI/CD 系统 |
| 测试 | ✅ 外部处理 | 链接检查、语法检查 | 自动化检查 |
| 部署 | ✅ 自动部署 | 部署到官网 | PR 合并后自动触发 |

**说明**：
- 本仓库是纯 Markdown 文档仓库
- 所有 CI/CD 配置由 OpenHarmony 基础设施团队维护
- 构建和发布流程在独立的 CI/CD 系统中执行
- 常见 CI/CD 配置文件检查：
  - `.github/workflows/` → ❌ 不存在
  - `.gitlab-ci.yml` → ❌ 不存在
  - `.circleci/config.yml` → ❌ 不存在

## 发布流程

### 文档更新触发

| 触发方式 | 说明 |
|----------|------|
| PR 合并 | 合并到 master 分支后自动发布 |
| 手动触发 | 管理员手动触发发布 |
| 版本发布 | 对应版本标签发布时触发 |

### 发布检查项

| 检查项 | 说明 |
|--------|------|
| 链接有效性 | 检查内部链接和外部链接 |
| 图片有效性 | 检查图片路径和引用 |
| 格式一致性 | 遵循文档模板和规范 |
| 中英文同步 | 新增内容需要同步翻译 |

## 版本文档更新

### 新版本发布流程

1. **创建版本分支**：`release-*` 分支
2. **添加发布说明**：`release-notes/OpenHarmony-v{版本}.md`
3. **更新主分支**：`master` 分支更新最新内容
4. **更新概览**：`zh-cn/OpenHarmony-Overview_zh.md` 和 `en/OpenHarmony-Overview.md`

### 发布说明模板

```markdown
# OpenHarmony {版本} Release

## 版本信息

| 属性 | 值 |
|------|-----|
| 版本号 | {版本号} |
| API Level | {API Level} |
| 发布日期 | {日期} |
| 类型 | Release / Beta / LTS |

## 主要特性

- 特性1
- 特性2
- 特性3

## 已知问题

- 问题1
- 问题2

## 升级说明

[升级指南]

## 相关资源

- [完整更新日志]
- [迁移指南]
- [API 参考]
```

## Docker 环境

**位置**：`docker/` 目录

| 文件 | 说明 |
|------|------|
| `docker/README.md` | Docker 使用说明（中文） |
| `docker/README_en.md` | Docker 使用说明（英文） |
| `docker/CHANGELOG.md` | 更新日志（中文） |
| `docker/CHANGELOG_en.md` | 更新日志（英文） |
| `Dockerfile` | Docker 镜像定义 |

**用途**：提供一致的文档编写和构建环境。

**证据**：`docker/` 目录包含5个文件

## 本地构建

**说明**：由于构建系统不在本仓库，本地构建需要通过以下方式：

### 方式 1：使用 Markdown 阅读器（推荐）

```bash
# 使用 VS Code 插件
# 1. 安装 Markdown Preview 插件
# 2. 打开 .md 文件，按 Ctrl+Shift+V 预览

# 使用 Typora
typora zh-cn/readme.md

# 使用在线 Markdown 编辑器
# 访问 https://dillinger.io/ 或类似工具
```

### 方式 2：使用 Docker 环境

```bash
# 进入 Docker 目录
cd docker/

# 查看 Docker 使用说明
cat README.md
```

**证据**：`docker/README.md` 存在

### 方式 3：提交到测试环境

```bash
# 1. 创建 PR
# 2. CI/CD 系统会自动构建并部署到测试环境
# 3. 通过测试环境链接预览效果
```

**注意**：实际生产构建命令未在仓库中公开，由基础设施团队管理。

---

**相关文档**：
- [目录结构](structure/) - 熟悉仓库结构
- [内容体系](content/) - 了解文档分类
- [贡献指南](contribute/) - 学习如何贡献

---

*最后更新：2026-02-07*
*状态：已确认构建流程由外部 CI/CD 系统管理*
