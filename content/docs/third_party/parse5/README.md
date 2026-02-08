# Parse5 - OpenHarmony 集成文档

> parse5 HTML 解析器在 OpenHarmony 中的集成与适配说明

**原始库**: [parse5](https://github.com/inikulin/parse5) v7.2.1
**OH 组件**: `@ohos/parse5` v4.0
**许可证**: MIT
**文档版本**: 1.0
**最后更新**: 2026-02-07

---

## 📋 目录

- [库概览](#库概览)
- [OH 适配概述](#oh-适配概述)
- [文档导航](#文档导航)
- [快速开始](#快速开始)

---

## 库概览

parse5 是一个符合 WHATWG HTML Living Standard (HTML5) 规范的 HTML 解析和序列化工具集，是 Node.js 上最快的规范兼容 HTML 解析器。

### 核心功能

- **HTML 解析器**: 将 HTML 字符串解析为 DOM 树
- **HTML 序列化器**: 将 DOM 树序列化为 HTML 字符串
- **SAX 解析器**: 流式 HTML 解析
- **完整规范支持**: 遵循 WHATWG HTML 标准

### 在 OpenHarmony 中的定位

parse5 在 OH 中主要为 **Ace Engine (ArkUI)** 提供 HTML 解析能力，支持 Web 相关功能，如：

- HTML 内容解析
- DOM 树构建
- Web 组件的 HTML 处理

---

## OH 适配概述

### 适配类型

parse5 在 OH 中的集成采用 **纯构建系统适配** 方案：

| 适配类型       | 状态    | 说明                     |
| -------------- | ------- | ------------------------ |
| **代码 Patch** | ❌ 无   | 无任何源代码修改         |
| **条件编译**   | ❌ 无   | 无 OH 特定宏定义         |
| **构建系统**   | ✅ 完整 | GN 构建配置 + 自定义脚本 |
| **代码压缩**   | ✅ 有   | uglify-js 压缩           |
| **模块重命名** | ✅ 是   | parse5 → parse           |

### 适配特点

✅ **零代码修改**: 直接使用 upstream 代码，保证与上游同步
✅ **构建流程优化**: TypeScript → CommonJS → 压缩
✅ **体积优化**: 去除类型声明和 sourceMap
✅ **模块集成**: 提供 ARK HAP 专用构建目标

---

## 文档导航

### 核心文档

| 文档                                               | 内容           | 重要性 |
| -------------------------------------------------- | -------------- | ------ |
| [SUMMARY.md](SUMMARY.md)                           | 阅读路线建议   | ⭐⭐⭐ |
| [01_Overview.md](01_Overview.md)                   | 原始库简介     | ⭐⭐   |
| [02_Patches.md](02_Patches.md)                     | Patch 详细分析 | ⭐⭐⭐ |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配    | ⭐⭐⭐ |
| [04_Usage_in_OH.md](04_Usage_in_OH.md)             | 依赖关系与使用 | ⭐⭐   |

### 工作文档

| 文档                                        | 内容         |
| ------------------------------------------- | ------------ |
| [\_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果 |
| [\_work/NOTES.md](_work/NOTES.md)           | 分析过程记录 |
| [\_work/PLAN.md](_work/PLAN.md)             | 任务进度     |

---

## 快速开始

### 对于开发者

如果你需要了解 parse5 如何集成到 OH：

1. 📖 先阅读 [SUMMARY.md](SUMMARY.md) 了解阅读路线
2. 📖 阅读 [01_Overview.md](01_Overview.md) 了解原始库功能
3. 🔧 阅读 [02_Patches.md](02_Patches.md) 了解为什么不需要代码 Patch
4. 🏗️ 阅读 [03_Build_Integration.md](03_Build_Integration.md) 了解构建适配详情
5. 🔗 阅读 [04_Usage_in_OH.md](04_Usage_in_OH.md) 了解在 OH 中的使用方式

### 对于维护者

如果你需要维护或升级 parse5：

1. 📊 查看 [\_work/ASSESSMENT.md](_work/ASSESSMENT.md) 了解当前状态
2. 🔧 参考 [03_Build_Integration.md](03_Build_Integration.md) 修改构建配置
3. ⚠️ 注意：无代码 Patch，升级时应保持 upstream 代码纯净

---

## 关键发现

### ✅ 优势

- **易于升级**: 无代码 Patch，可直接同步上游新版本
- **体积优化**: 通过 uglify-js 压缩，减少 ROM 占用
- **稳定可靠**: 使用经过验证的 upstream 代码

### ⚠️ 注意事项

- **模块名称**: OH 中输出名为 `parse` 而非 `parse5`
- **构建依赖**: 需要 TypeScript 和 uglify-js 作为构建工具
- **构建产物**: 仅生成 CommonJS 模块，无类型声明

---

## 参考资料

- [parse5 官方文档](https://parse5.js.org/)
- [parse5 GitHub 仓库](https://github.com/inikulin/parse5)
- [WHATWG HTML 标准](https://html.spec.whatwg.org/)
- [OpenHarmony 构建系统文档](https://docs.openharmony.cn/cn/)

---

**维护者**: OpenHarmony Third Party Wiki Agent
**反馈**: 如有问题或建议，请通过 OpenHarmony 社区反馈
