# PyYAML OpenHarmony 集成文档

**库版本**: 6.0.2
**最后更新**: 2026-02-07
**文档版本**: 1.0

---

## 概述

本文档记录了 PyYAML 在 OpenHarmony 中的集成与适配情况，重点说明 OpenHarmony 对该库的 Patch、特殊适配、以及被系统使用的方式。

### PyYAML 简介

PyYAML 是一个功能完整的 YAML 处理框架，专为 Python 设计。它提供了完整的 YAML 1.1 解析器、Unicode 支持、pickle 支持、强大的扩展 API 和友好的错误消息。

### 在 OpenHarmony 中的定位

PyYAML 在 OpenHarmony 中作为 **第三方库组件** 提供，归属于 `thirdparty` 子系统。它是一个**纯 Python 库**，不涉及 C/C++ 代码的直接集成。

### 集成方式

PyYAML 在 OpenHarmony 中采用 **轻量级适配** 方式：

- **无源代码修改**: 保持与上游版本完全一致
- **配置文件适配**: 通过 `bundle.json`、`OAT.xml` 等声明组件信息
- **构建工具增强**: 添加 PEP 517 构建后端支持配置化构建

---

## 文档导航

### 核心文档

| 文档 | 描述 | 读者 |
|------|------|------|
| [01_Overview.md](./01_Overview.md) | PyYAML 原始库简介、核心功能、项目结构 | 所有人 |
| [02_Patches.md](./02_Patches.md) | OpenHarmony Patch 详细分析、修改清单、升级建议 | 开发者 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OpenHarmony 构建适配、bundle.json 配置、构建流程 | 构建工程师 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 在 OpenHarmony 中的依赖关系、使用场景、API 使用 | 开发者 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异、兼容性分析 | 开发者 |
| [06_Security.md](./06_Security.md) | 安全风险分析、CVE 修复、最佳实践 | 安全工程师 |

### 辅助文档

| 文档 | 描述 | 读者 |
|------|------|------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议、快速导航 | 所有人 |
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估结果、基础信息收集 | 分析师 |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录、技术细节 | 维护者 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度、待办事项 | 维护者 |

---

## 快速导航

### 我想了解...

#### PyYAML 是什么？

→ 阅读 [01_Overview.md](./01_Overview.md)

#### PyYAML 在 OpenHarmony 中有哪些定制化修改？

→ 阅读 [02_Patches.md](./02_Patches.md)

#### PyYAML 如何集成到 OpenHarmony 构建系统？

→ 阅读 [03_Build_Integration.md](./03_Build_Integration.md)

#### 哪些 OH 模块在使用 PyYAML？

→ 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md)

#### OpenHarmony 版本的 PyYAML 有哪些 API 差异？

→ 阅读 [05_API_Differences.md](./05_API_Differences.md)

#### 使用 PyYAML 有哪些安全注意事项？

→ 阅读 [06_Security.md](./06_Security.md)

---

## 关键发现

### 1. 无源代码修改

PyYAML 在 OpenHarmony 中保持了 100% 的原始代码，**没有修改任何源文件**。所有适配都通过配置文件和构建工具实现。

### 2. 轻量级配置适配

OpenHarmony 对 PyYAML 的适配仅限于：
- `bundle.json` - 组件声明
- `OAT.xml` - 开源合规检查
- `README.OpenSource` - 开源信息声明
- `packaging/_pyyaml_pep517.py` - PEP 517 构建后端

### 3. Python 3.8+ 支持

OpenHarmony 使用的 PyYAML 6.0.2 支持 Python 3.8 - 3.13，与 OpenHarmony 的 Python 环境完全兼容。

### 4. 广泛的使用场景

PyYAML 在 OpenHarmony 中被 **88 处引用**，主要用于：
- 配置文件解析（最常见的使用方式）
- 测试元数据处理
- 构建系统代码生成
- 工具链配置管理

### 5. 安全使用实践

OpenHarmony 代码库中 88% 的 PyYAML 使用采用了 `safe_load()` 安全加载方式，符合最佳实践。

---

## 版本信息

| 项目 | 值 |
|------|-----|
| **库版本** | 6.0.2 |
| **上游地址** | https://github.com/yaml/pyyaml |
| **许可证** | MIT |
| **子系统** | thirdparty |
| **适配系统类型** | mini, standard |
| **Python 版本** | 3.8 - 3.13 |
| **Cython 版本** | < 3.0 (Python 3.8-3.12), >= 3.0 (Python 3.13) |

---

## 贡献与反馈

### 文档维护

本文档由 Sisyphus Agent 生成和维护。

### 问题反馈

如发现文档错误或需要补充的内容，请通过以下方式反馈：
- 提交 Issue 到 OpenHarmony 仓库
- 联系 PyYAML 维护者（xuyong59@huawei.com）

### 贡献指南

欢迎贡献文档改进：
1. Fork 本仓库
2. 创建分支进行修改
3. 提交 Pull Request

---

## 参考资源

### PyYAML 官方资源

- [PyYAML 官方文档](https://pyyaml.org/wiki/PyYAMLDocumentation)
- [PyYAML GitHub 仓库](https://github.com/yaml/pyyaml)
- [PyYAML PyPI 页面](https://pypi.org/project/PyYAML/)
- [YAML 1.1 规范](https://yaml.org/spec/1.1/)

### OpenHarmony 资源

- [OpenHarmony 官方网站](https://www.openharmony.cn/)
- [OpenHarmony 文档中心](https://docs.openharmony.cn/)
- [OpenHarmony 源码仓库](https://gitee.com/openharmony)

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|---------|
| 1.0 | 2026-02-07 | 初始版本，包含完整的 PyYAML OpenHarmony 集成文档 |

---

**版权声明**: 本文档基于 MIT 许可证发布。

**致谢**: 感谢 OpenHarmony 社区和 PyYAML 社区的贡献。
