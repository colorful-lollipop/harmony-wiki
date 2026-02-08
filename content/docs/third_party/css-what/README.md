# css-what - OpenHarmony Wiki

> CSS 选择器解析库在 OpenHarmony 中的集成与适配文档

## 库概览

| 项目           | 内容                             |
| -------------- | -------------------------------- |
| **库名称**     | css-what                         |
| **上游版本**   | v7.0.0                           |
| **OH 版本**    | 3.1                              |
| **许可证**     | BSD-2-Clause                     |
| **上游地址**   | https://github.com/fb55/css-what |
| **OH 维护者**  | lixingchi1@huawei.com            |
| **所属子系统** | thirdparty                       |

**功能描述**：CSS 选择器解析器，用于将 CSS 选择器字符串解析为结构化的抽象语法树（AST）。

## OpenHarmony 适配状态

| 适配项            | 状态      | 说明                        |
| ----------------- | --------- | --------------------------- |
| **Patch 数量**    | 0         | 无 Patch，纯上游源码集成    |
| **BUILD.gn 适配** | ✅ 已完成 | 使用 ohos_prebuilt_etc 模板 |
| **OH 特有源码**   | ❌ 无     | 无平台特定代码              |
| **条件编译**      | ❌ 无     | 无 OH 宏定义                |

## 文档导航

### 核心文档

| 文档                                      | 描述                     | 必读 |
| ----------------------------------------- | ------------------------ | ---- |
| [01\_概述](./01_Overview.md)              | 原始库功能介绍及 OH 定位 | ✅   |
| [02_Patch 分析](./02_Patches.md)          | Patch 清单及详细分析     | ✅   |
| [03\_构建适配](./03_Build_Integration.md) | BUILD.gn 配置说明        | ✅   |
| [04\_在 OH 中的使用](./04_Usage_in_OH.md) | 依赖关系及使用场景       | ✅   |
| [05_API 差异](./05_API_Differences.md)    | OH 新增或变更的 API      | ○    |
| [06\_安全风险](./06_Security.md)          | CVE 及安全建议           | ○    |

### 工作文档

| 文档                                   | 描述         |
| -------------------------------------- | ------------ |
| [ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估报告 |
| [NOTES.md](./_work/NOTES.md)           | 分析过程记录 |
| [PLAN.md](./_work/PLAN.md)             | 任务进度追踪 |

## 快速开始

### 获取源码

```bash
# 在 OpenHarmony 源码根目录下
cd third_party/css-what
```

### 查看依赖关系

```bash
# 搜索依赖该库的模块
grep -r "third_party/css-what" ../.. --include="BUILD.gn"
```

### 构建

```bash
# OH 构建系统会自动处理
# 通过 jsframework 或 ace_engine 间接构建
```

## 版本信息

### 上游与 OH 版本对照

| OH 版本 | 上游版本 | 差异说明                 |
| ------- | -------- | ------------------------ |
| 3.1     | v7.0.0   | 跟随上游，版本号策略不同 |

### 版本更新历史

| OH 版本 | 更新内容 | 日期 |
| ------- | -------- | ---- |
| 3.1     | 初始集成 | -    |

## 维护指南

### 升级上游版本

由于该库**无 Patch**，升级流程相对简单：

1. 更新 `package.json` 中的版本号
2. 更新 `README.OpenSource` 中的版本信息
3. 更新 `bundle.json` 中的版本号
4. 验证构建是否正常

### 常见问题

**Q: 为什么不需要 Patch？**

A: css-what 是纯 TypeScript 库，CSS 选择器解析是通用功能，不涉及平台特定 API，因此无需 Patch。

**Q: 该库在 OH 中的具体用途是什么？**

A: 主要用于 jsframework 的 CSS 选择器解析，支持 ArkUI 框架的样式选择功能。

## 相关资源

- [上游仓库](https://github.com/fb55/css-what)
- [OH jsframework](../jsframework/README.md)
- [OH ace_engine](../../foundation/arkui/ace_engine/README.md)

---

_文档版本：1.0_
_最后更新：2026-02-08_
