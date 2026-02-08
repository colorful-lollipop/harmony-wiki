# weex-loader Wiki 文档

**OpenHarmony 第三方库适配文档**

---

## 库概览

| 属性 | 值 |
|------|-----|
| **库名称** | weex-loader |
| **上游版本** | v0.7.12 |
| **OH 版本** | 3.1 |
| **上游地址** | https://github.com/apache/weex-loader.git |
| **许可证** | Apache-2.0 |
| **OH 组件名** | @ohos/weex-loader |
| **所属子系统** | thirdparty |
| **维护者** | sunbingxin@huawei.com |

## 一句话描述

**weex-loader** 是一个 webpack loader，用于将 Weex/HML 格式的模板文件编译为 JavaScript，在 OpenHarmony 中作为 ace_js2bundle 的核心依赖，用于 JS FA (Feature Ability) 应用的编译构建。

## OH 适配概述

### 适配方式

该库在 OpenHarmony 中**未使用传统 Patch 文件**进行适配，而是采用**直接修改源代码**的方式。主要适配内容包括：

1. **新增 GN 构建系统支持** - 添加 BUILD.gn 和 Python 构建脚本
2. **设备分级支持** - 添加 Rich/Lite/Card 三种设备级别的编译支持
3. **OH 模块系统适配** - 支持 `@ohos.xxx` 和 `@system.xxx` 模块导入
4. **Lite 设备优化** - 添加轻量设备的模板和样式转换逻辑

### 关键特性

- ✅ **分级设备支持**: 根据 `DEVICE_LEVEL` 环境变量输出不同的代码
- ✅ **HML 编译**: 支持 HML 模板编译为 JavaScript 渲染函数
- ✅ **CSS 解析**: 使用 weex-styler 解析 CSS 样式
- ✅ **JS 处理**: 使用 weex-scripter 处理 JavaScript 代码
- ✅ **资源引用**: 支持资源引用路径处理

### 文档导航

#### 📚 核心文档

| 文档 | 内容 | 推荐阅读顺序 |
|------|------|-------------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 定位 | 1 |
| [02_Patches.md](./02_Patches.md) | Patch 分析、源代码修改 | 2 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn、构建流程 | 3 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景 | 4 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异、环境变量 | 5 |
| [06_Security.md](./06_Security.md) | 安全分析、CVE | 6 |

#### 📂 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度 |

#### 📖 阅读路线

- **快速了解**: README.md → 01_Overview.md → 04_Usage_in_OH.md
- **开发参考**: 03_Build_Integration.md → 05_API_Differences.md
- **深度分析**: 02_Patches.md → _work/NOTES.md
- **维护升级**: 02_Patches.md → 06_Security.md

## 重要提示

### ⚠️ 无 Patch 文件

该库**没有使用 .patch 文件**进行适配，所有修改都是直接修改源代码。升级上游版本时需要：
1. 人工对比原始文件和 OH 版本
2. 重新应用 OH 特有修改
3. 验证功能完整性

### 🔗 深度耦合 ace_js2bundle

weex-loader 是 ace_js2bundle 的核心依赖，两者版本需要保持一致。修改 weex-loader 时需要同时验证 ace_js2bundle 的兼容性。

### 🎯 关键环境变量

编译时依赖以下环境变量：
- `DEVICE_LEVEL` - 设备级别 (rich/lite/card)
- `abilityType` - 应用类型 (page/app)
- `projectPath` - 项目路径
- `aceManifestPath` - manifest 路径

## 联系方式

- **维护者**: sunbingxin@huawei.com
- **上游社区**: https://github.com/apache/weex-loader
- **OH 问题反馈**: OpenHarmony 社区 Issue

---

**文档版本**: 1.0  
**最后更新**: 2026-02-07
