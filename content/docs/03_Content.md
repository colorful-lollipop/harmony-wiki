# 文档内容体系

> 本文档描述 OpenHarmony 文档仓库的文档分类体系、各模块内容和写作规范。

## 文档分类总览

| 分类 | 中文目录 | 英文目录 | 说明 |
|------|---------|---------|------|
| 入门指南 | `readme.md` | `readme.md` | 文档导航和快速入门 |
| 应用开发 | `application-dev/` | `application-dev/` | 应用开发指南和教程 |
| 贡献指南 | `contribute/` | `contribute/` | 社区贡献相关文档 |
| 设计规范 | `design/` | `design/` | UX/UI 设计规范和模板 |
| 设备开发 | `device-dev/` | `device-dev/` | 设备驱动和系统开发 |
| 发布说明 | `release-notes/` | `release-notes/` | 各版本发布说明 |
| 第三方案例 | `third-party-cases/` | - | 社区贡献的实践案例 |
| 第三方组件 | `third-party-components/` | `third-party-components/` | 第三方库和组件说明 |
| 术语表 | `glossary.md` | `glossary.md` | 专业术语解释 |

## 文档模块详解

### 1. 入门指南 (readme)

**位置**：`zh-cn/readme.md`、`en/readme.md`

**内容**：
- OpenHarmony 简介
- 文档导航
- 快速开始指引
- 相关资源链接

**证据**：`zh-cn/readme.md` 文件存在

### 2. 应用开发 (application-dev)

**位置**：`zh-cn/application-dev/`、`en/application-dev/`

**内容分类**：

| 子模块 | 描述 |
|--------|------|
| quick-start | 快速入门教程 |
| core-work | 核心能力开发 |
| ui-ability | UI 与Ability开发 |
| connectivity | 网络与连接 |
| security | 安全与权限 |
| performance | 性能优化 |
| testing | 测试指南 |
| packaging | 应用打包 |

**证据**：`zh-cn/application-dev/` 包含约50个条目

### 3. 贡献指南 (contribute)

**位置**：`zh-cn/contribute/`、`en/contribute/`

**中文文档列表**：

| 文件 | 描述 |
|------|------|
| `参与贡献.md` | 贡献总览 |
| `贡献文档.md` | 文档贡献指南 |
| `第三方开源软件及许可证说明.md` | 许可证说明 |
| `OpenHarmony-ArkTS-coding-style-guide.md` | ArkTS 编码规范 |
| `OpenHarmony-Application-Typescript-JavaScript-coding-guide.md` | TS/JS 编码规范 |
| `OpenHarmony-c-cpp-secure-coding-guide.md` | C/C++ 安全编码规范 |
| `OpenHarmony-cpp-coding-style-guide.md` | C++ 编码规范 |
| `OpenHarmony-c-coding-style-guide.md` | C 编码规范 |
| `OpenHarmony-hdf-coding-guide.md` | HDF 编码规范 |
| `OpenHarmony-build-rule.md` | 构建规则 |
| `OpenHarmony-security-design-guide.md` | 安全设计指南 |
| `OpenHarmony-security-test-guide.md` | 安全测试指南 |
| `code-contribution.md` | 代码贡献流程 |
| `how-to-contribute.md` | 贡献方式说明 |
| `introducing-open-source-software.md` | 开源软件介绍 |
| `licenses-and-special-license-review.md` | 许可证审查 |
| `open-source-compliance-issue-management.md` | 合规问题管理 |
| `open-source-design-specification-document-and-usage-guide.md` | 设计规范使用 |
| `prebuilts-readme-template.md` | 预置模板 |
| `writing-instructions.md` | 写作指南 |
| `communication-in-community.md` | 社区沟通 |
| `docs-release-process.md` | 文档发布流程 |
| `contribution-guide.md` | 贡献指南 |
| `best-practices-and-suggestions-for-contributions-to-upstream-open-source-projects.md` | 上游贡献建议 |
| `FAQ.md` | 常见问题 |

**证据**：`zh-cn/contribute/` 包含约43个文档

### 4. 设计规范 (design)

**位置**：`zh-cn/design/`、`en/design/`

**结构**：

```
design/
├── API-Review-Template.md    # API 评审模板
└── ux-design/                # UX 设计规范
    ├── Readme-CN.md
    ├── animation-overview.md
    ├── animation-design-principles.md
    ├── adaptive-layout.md
    ├── design-checklist.md
    ├── multimodal-dialog.md
    ├── multimodal-divider.md
    ├── multimodal-menu.md
    ├── multimodal-search-box.md
    ├── multimodal-text-box.md
    ├── multimodal-text.md
    ├── multimodal-tick-box.md
    ├── responsive-layout.md
    └── visual-app-icons.md
```

**证据**：`zh-cn/design/` 包含约10个文档

### 5. 设备开发 (device-dev)

**位置**：`zh-cn/device-dev/`、`en/device-dev/`

**内容分类**：

| 子模块 | 描述 |
|--------|------|
| quick-start | 设备开发快速入门 |
| porting | 芯片适配指南 |
| driver | 驱动开发 |
| subsystem | 子系统开发 |
| debug | 调试指南 |
| tools | 开发工具 |

**证据**：`zh-cn/device-dev/` 包含约20个条目

### 6. 发布说明 (release-notes)

**位置**：`zh-cn/release-notes/`、`en/release-notes/`

**版本列表**：

| 版本 | API Level | 中文 | 英文 |
|------|-----------|------|------|
| OpenHarmony 6.0 Release | 20 | ✅ | ✅ |
| OpenHarmony 5.1.0 Release | 18 | ✅ | ✅ |
| OpenHarmony 5.0.3 | 15 | ✅ | - |
| OpenHarmony 5.0.2 | 14 | ✅ | - |
| OpenHarmony 5.0.1 | 13 | ✅ | - |
| OpenHarmony 5.0.0 Release | 12 | ✅ | ✅ |
| OpenHarmony 4.1 Release | 11 | ✅ | ✅ |
| OpenHarmony 4.0 Release | 10 | ✅ | ✅ |
| OpenHarmony 3.2 Release | 9 | ✅ | ✅ |
| OpenHarmony 3.2.3 Release | - | ✅ | ✅ |
| OpenHarmony 3.2 Beta5 | - | ✅ | ✅ |
| OpenHarmony 3.2 Beta4 | - | ✅ | ✅ |
| OpenHarmony 3.2 Beta1 | - | ✅ | ✅ |
| OpenHarmony 3.1 Release | - | ✅ | ✅ |
| OpenHarmony 3.1.5 Release | - | ✅ | ✅ |
| OpenHarmony 3.1 Beta | - | - | ✅ |
| OpenHarmony 3.0 LTS | - | ✅ | ✅ |
| OpenHarmony 3.0.3 LTS | - | ✅ | ✅ |
| OpenHarmony 3.0.1 LTS | - | ✅ | ✅ |
| OpenHarmony 1.1.5 LTS | - | ✅ | ✅ |
| OpenHarmony 1.1.4 LTS | - | ✅ | ✅ |

**证据**：`zh-cn/release-notes/` 包含约67个文档，`en/release-notes/` 包含约65个文档

### 7. 第三方案例 (third-party-cases)

**位置**：`zh-cn/third-party-cases/`

**说明**：存放社区贡献的实践案例，包含详细的应用开发示例。

**案例列表（部分）**：

| 文件 | 描述 |
|------|------|
| `Readme-CN.md` | 第三方案例说明 |
| `Multi-level-linkage.md` | 多级联动案例 |
| `distributed-file.md` | 分布式文件案例 |
| `diverse-dialogues.md` | 对话框案例 |
| `griditem-drag-and-drop.md` | 拖拽案例 |
| `how-to-add-delete-listitems.md` | 列表操作案例 |
| `how-to-develop-frame-animation.md` | 帧动画案例 |
| `how-to-implement-fluid-layout.md` | 流式布局案例 |
| `how-to-load-images-from-internet.md` | 图片加载案例 |
| `immersion-mode.md` | 沉浸模式案例 |
| `interact-lists.md` | 列表交互案例 |
| `listitem-slide-to-display-menu.md` | 侧滑菜单案例 |
| `multi-device-app-dev.md` | 多设备开发案例 |
| `navigation-drawer.md` | 导航抽屉案例 |
| `operation-regulations.md` | 操作规范案例 |
| `photo-pixelmap-transfer.md` | 图片处理案例 |
| `pixel-format-transfer.md` | 像素格式转换案例 |
| `realization-of-collapsible-title-effect.md` | 折叠标题效果 |
| `set-volume-brightness-through-gesture.md` | 手势控制案例 |
| `take-picture-and-preview.md` | 拍照预览案例 |
| `transition-animation.md` | 转场动画案例 |

**证据**：`zh-cn/third-party-cases/` 包含约67个文档

### 8. 第三方组件 (third-party-components)

**位置**：`zh-cn/third-party-components/`、`en/third-party-components/`

**说明**：说明文档中引用的第三方开源组件及其许可证信息。

**证据**：`zh-cn/third-party-components/` 包含约4个文档

### 9. 术语表 (glossary)

**位置**：`zh-cn/glossary.md`、`en/glossary.md`

**说明**：OpenHarmony 专业术语解释。

| 文件 | 大小 | 描述 |
|------|------|------|
| `zh-cn/glossary.md` | 5,498 bytes | 中文术语表 |
| `en/glossary.md` | 6,156 bytes | 英文术语表 |

## 文档写作规范

### 编码规范文档

| 语言 | 文档 | 证据 |
|------|------|------|
| ArkTS | `zh-cn/contribute/OpenHarmony-ArkTS-coding-style-guide.md` | 文件存在 |
| TypeScript/JavaScript | `zh-cn/contribute/OpenHarmony-Application-Typescript-JavaScript-coding-guide.md` | 文件存在 |
| C/C++ 安全编码 | `zh-cn/contribute/OpenHarmony-c-cpp-secure-coding-guide.md` | 文件存在 |
| C++ | `zh-cn/contribute/OpenHarmony-cpp-coding-style-guide.md` | 文件存在 |
| C | `zh-cn/contribute/OpenHarmony-c-coding-style-guide.md` | 文件存在 |
| HDF | `zh-cn/contribute/OpenHarmony-hdf-coding-guide.md` | 文件存在 |

### API 评审模板

**位置**：`zh-cn/design/API-Review-Template.md`

**用途**：API 设计评审的标准模板。

**证据**：`zh-cn/design/API-Review-Template.md` 文件存在

---

**相关文档**：
- [项目概览](01_Overview.md) - 了解项目背景
- [目录结构](02_Structure.md) - 熟悉仓库结构
- [贡献指南](05_Contribute.md) - 学习如何贡献

---

*最后更新：2026-02-06*
