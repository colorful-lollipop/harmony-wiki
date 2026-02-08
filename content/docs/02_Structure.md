# 目录结构

> 本文档描述 OpenHarmony 文档仓库的目录结构、各模块职责和文件组织规范。

## 顶层目录

```
docs/                          # 仓库根目录
├── wiki/                      # 本 Wiki（工程文档）
├── zh-cn/                     # 中文文档主目录
├── en/                        # 英文文档主目录
├── docker/                    # Docker 环境相关
├── .gitcode/                  # GitCode 平台配置
├── .config/                   # IDE/编辑器配置
├── README.md                  # 项目说明（英文）
├── README_zh.md               # 项目说明（中文）
├── LICENSE                    # CC BY 4.0 许可证
├── DCO.txt                    # 开发者原创声明
├── CODEOWNERS                # 代码所有者配置
├── OAT.xml                   # OpenAPI 规范？
├── xts.diff                  # XTS 测试相关
├── image.png                 # 项目图片
├── .gitattributes            # Git 属性配置
└── .gitignore                # Git 忽略规则
```

## 文档内容目录详解

### 中文文档 (zh-cn/)

```
zh-cn/
├── readme.md                     # 中文文档首页导航
├── OpenHarmony-Overview_zh.md   # OpenHarmony 概览（中文）
├── glossary.md                   # 术语表（中文）
├── website.md                    # 网站相关说明
├── Legal-Notices.md              # 法律声明
├── application-dev/              # 应用开发指南
│   ├── readme.md                 # 模块说明
│   └── [50个左右的子目录和文件]
├── contribute/                   # 贡献指南
│   ├── 参与贡献.md
│   ├── 贡献文档.md
│   ├── 第三方开源软件及许可证说明.md
│   └── [40个左右的文档]
├── design/                      # 设计规范
│   ├── API-Review-Template.md   # API 评审模板
│   ├── ux-design/               # UX 设计规范
│   └── [其他设计文档]
├── device-dev/                  # 设备开发指南
│   └── [20个左右的子目录和文件]
├── release-notes/               # 发布说明
│   ├── OpenHarmony-v6.0-release.md
│   ├── OpenHarmony-v5.0.0-release.md
│   ├── OpenHarmony-v3.0-LTS.md
│   └── [67个左右的版本文档]
├── third-party-cases/           # 第三方案例
│   ├── Readme-CN.md
│   └── [64个左右的案例文档]
├── third-party-components/      # 第三方组件
│   └── [4个左右的组件文档]
└── figures/                     # 图片资源
    └── [图片文件]
```

### 英文文档 (en/)

```
en/
├── readme.md                     # 英文文档首页导航
├── OpenHarmony-Overview.md       # OpenHarmony 概览（英文）
├── glossary.md                   # 术语表（英文）
├── website.md                    # 网站相关说明
├── Legal-Notices.md              # 法律声明
├── application-dev/              # 应用开发指南
│   ├── readme.md
│   └── [51个左右的子目录和文件]
├── contribute/                   # 贡献指南
│   ├── how-to-contribute.md
│   ├── documentation-contribution.md
│   ├── open-source-software-and-license-notice.md
│   └── [40个左右的文档]
├── design/                       # 设计规范
│   ├── API-Review-Template.md
│   ├── ux-design/
│   └── [其他设计文档]
├── device-dev/                   # 设备开发指南
│   └── [19个左右的子目录和文件]
├── release-notes/                # 发布说明
│   ├── OpenHarmony-v6.0-release.md
│   ├── OpenHarmony-v5.1.0-release.md
│   └── [65个左右的版本文档]
├── third-party-components/       # 第三方组件
│   └── [4个左右的组件文档]
└── figures/                      # 图片资源
    └── [图片文件]
```

### Docker 环境 (docker/)

```
docker/
├── README.md              # Docker 使用说明（中文）
├── README_en.md           # Docker 使用说明（英文）
├── CHANGELOG.md           # 更新日志（中文）
├── CHANGELOG_en.md        # 更新日志（英文）
└── [Dockerfile等配置文件]
```

### GitCode 配置 (.gitcode/)

```
.gitcode/
├── ISSUE_TEMPLATE/        # Issue 模板
└── PULL_REQUEST_TEMPLATE.zh-CN.md  # PR 模板（中文）
```

## Wiki 目录 (wiki/)

```
wiki/                          # 本 Wiki
├── README.md                  # Wiki 说明
├── SUMMARY.md                 # 全站导航
├── 01_Overview.md             # 项目概览
├── 02_Structure.md            # 目录结构（本文档）
├── 03_Content.md              # 文档内容体系
├── 04_Build.md               # 构建与发布
├── 05_Contribute.md          # 贡献指南
├── _work/                    # 工作区
│   ├── NOTES.md              # 事实记录
│   └── PLAN.md               # 任务计划
└── appendix/                 # 附录
    ├── History.md            # 版本历史
    └── Templates.md          # 文档模板
```

## 模块职责说明

| 目录/模块 | 职责 | 证据 |
|----------|------|------|
| `zh-cn/` | 存放所有中文文档，保持与英文同步更新 | 目录存在 |
| `en/` | 存放所有英文文档，与中文版本一一对应 | 目录存在 |
| `docker/` | Docker 开发环境配置和使用说明 | `docker/README.md` |
| `.gitcode/` | GitCode 平台的 Issue 和 PR 模板 | `.gitcode/` 目录 |
| `wiki/` | 本工程 Wiki，用于描述仓库本身 | `wiki/README.md` |

## 命名规范

### 目录命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 文档模块 | 英文小写，使用连字符 | `application-dev`, `device-dev` |
| 版本文档 | `OpenHarmony-{版本号}-{类型}.md` | `OpenHarmony-v6.0-release.md` |
| 案例文档 | 英文描述，使用连字符 | `how-to-connect-to-bluetooth.md` |

### 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 规范文档 | 英文描述，使用连字符 | `coding-style-guide.md` |
| 翻译文档 | 与原文同名 | `OpenHarmony-Overview_zh.md` |
| 模板文件 | 包含 `Template` 或 `Example` | `API-Review-Template.md` |

## 文件统计

| 目录 | Markdown 文件数(约) |
|------|-------------------|
| `zh-cn/` | 200+ |
| `en/` | 200+ |
| `docker/` | 5 |
| `wiki/` | 10 |
| **总计** | **415+** |

---

**相关文档**：
- [项目概览](01_Overview.md) - 了解项目背景
- [文档内容体系](03_Content.md) - 了解文档分类
- [贡献指南](05_Contribute.md) - 学习如何贡献

---

*最后更新：2026-02-06*
