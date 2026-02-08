# 项目概览

> 本文档描述 OpenHarmony 文档仓库的基本信息、项目定位和关键概念。

## 项目定位

### 仓库类型

**OpenHarmony 文档仓库** 是一个专门存放 OpenHarmony 操作系统开发者文档的仓库，而非代码仓库。

| 属性 | 说明 |
|------|------|
| 仓库性质 | 文档仓库（Documentation Repository） |
| 主要内容 | OpenHarmony 开发者文档（中英文） |
| 目标用户 | 应用开发者、设备开发者、贡献者 |
| 维护方 | OpenHarmony Documentation SIG |

### 与代码仓库的关系

本仓库与 OpenHarmony 源代码仓库的关系：

| 仓库 | 说明 | 链接 |
|------|------|------|
| 文档仓库 | 本仓库，存放文档 | docs.openharmony.cn |
| 代码仓库 | OpenHarmony 源代码 | gitee.com/openharmony |
| 官网 | OpenHarmony 官方网站 | openharmony.cn |

**说明**：本仓库的文档内容基于代码仓库的版本发布进行更新，包含应用开发、设备开发、设计规范等各类文档。

## 许可证

| 项目 | 许可证 |
|------|--------|
| 文档内容 | CC BY 4.0 (Creative Commons Attribution 4.0 International) |
| 代码片段 | 遵循 OpenHarmony 源码许可证 |
| 第三方组件 | 详见各文档头部声明 |

**证据**：`LICENSE:1-57`

## 关键概念

### API Level

API Level 是 OpenHarmony API 的版本标识，用于区分不同版本的系统能力。

| API Level | 对应版本 | 状态 |
|-----------|----------|------|
| 20 | OpenHarmony 6.0 Release | ✅ 最新 |
| 18 | OpenHarmony 5.1.0 Release | ✅ 维护中 |
| 15 | OpenHarmony 5.0.3 | ✅ 维护中 |
| 14 | OpenHarmony 5.0.2 | ✅ 维护中 |
| 13 | OpenHarmony 5.0.1 | ✅ 维护中 |
| 12 | OpenHarmony 5.0.0 Release | ✅ 维护中 |
| 11 | OpenHarmony 4.1 Release | ❌ 已停止 |
| 10 | OpenHarmony 4.0 Release | ❌ 已停止 |
| 9 | OpenHarmony 3.2 Release | ❌ 已停止 |

### LTS 版本

LTS（Long Term Support）版本是 OpenHarmony 的长期支持版本，提供更长的维护周期。

| LTS 版本 | 发布日期 | 维护状态 |
|----------|----------|----------|
| OpenHarmony 3.0 LTS | - | ❌ 已停止 |

### XTS 测试

XTS（X Test Suite）是 OpenHarmony 的兼容性测试套件，用于验证设备是否符合 OpenHarmony 技术规范。

**证据**：`xts.diff` 文件存在

## 版本发布策略

### 分支策略

| 分支 | 说明 |
|------|------|
| master | 最新开发版本文档 |
| release-* | 各发布版本文档分支 |

### 维护周期

各版本的维护策略详见：[release-management](https://gitee.com/openharmony/release-management)

**证据**：`README.md:51`

## 语言支持

### 官方语言

| 语言 | 目录 | 状态 |
|------|------|------|
| 中文 (简体) | `zh-cn/` | ✅ 完整 |
| 英文 | `en/` | ✅ 完整 |

### 同步机制

中英文文档原则上保持同步更新，具体同步策略详见 [05_Contribute.md](05_Contribute.md)。

## 相关资源

### 官方链接

| 资源 | 链接 |
|------|------|
| OpenHarmony 官网 | https://www.openharmony.cn/ |
| 邮件列表 | docs@openharmony.io |
| Zulip 群组 | documentation_sig |
| Gitee | https://gitee.com/openharmony |

### 文档链接

| 资源 | 链接 |
|------|------|
| 中文文档入口 | [zh-cn/readme.md](../zh-cn/readme.md) |
| 英文文档入口 | [en/readme.md](../en/readme.md) |
| 贡献指南 | [zh-cn/contribute/参与贡献.md](../zh-cn/contribute/参与贡献.md) |
| 第三方许可证 | [zh-cn/contribute/第三方开源软件及许可证说明.md](../zh-cn/contribute/第三方开源软件及许可证说明.md) |

---

**相关文档**：
- [目录结构](02_Structure.md) - 了解仓库目录结构
- [文档内容体系](03_Content.md) - 了解文档分类
- [贡献指南](05_Contribute.md) - 学习如何贡献

---

*最后更新：2026-02-06*
