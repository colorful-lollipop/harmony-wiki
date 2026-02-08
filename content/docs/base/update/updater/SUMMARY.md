# OpenHarmony Updater Wiki 导航

## 快速导航

### 基础阅读（必读）

| 文档 | 说明 | 预估阅读时间 |
|------|------|-------------|
| [README](./README.md) | 本文档说明与项目信息 | 5 分钟 |
| [00_Overview](./00_Overview.md) | 项目定位、核心能力、运行环境 | 15 分钟 |
| [01_Architecture](./01_Architecture.md) | 架构组件、数据流、线程模型 | 20 分钟 |
| [02_Directory_Structure](./02_Directory_Structure.md) | 目录结构与模块职责 | 10 分钟 |

### 接口文档

| 文档 | 说明 | 适用读者 |
|------|------|---------|
| [03_Public_API](./03_Public_API.md) | 对外 C++ API 完整清单 | 应用开发者 |
| [04_Inner_API](./04_Inner_API.md) | 内部模块接口 | 子系统开发者 |

### 工程文档

| 文档 | 说明 | 适用读者 |
|------|------|---------|
| [05_GN_Build](./05_GN_Build.md) | 构建系统与编译产物 | 构建工程师 |
| [06_Security_Analysis](./06_Security_Analysis.md) | 安全风险评审 | 安全工程师 |
| [07_Troubleshooting](./07_Troubleshooting.md) | 问题定位与调试 | 现场工程师 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs](./appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags](./appendix/Config_Flags.md) | 编译配置标志 |

## 阅读路线建议

### 路线 1: 新人入门（30 分钟）

```
README → 00_Overview → 01_Architecture → 02_Directory_Structure
```

目标：建立对 Updater 子系统的整体认知。

### 路线 2: 应用开发（45 分钟）

```
00_Overview → 03_Public_API → 05_GN_Build → 07_Troubleshooting
```

目标：掌握如何调用 Updater API 进行 OTA 升级。

### 路线 3: 子系统开发（60 分钟）

```
01_Architecture → 02_Directory_Structure → 04_Inner_API → 05_GN_Build
```

目标：深入理解内部实现，进行二次开发。

### 路线 4: 安全审计（40 分钟）

```
00_Overview → 01_Architecture → 06_Security_Analysis → appendix/Callgraphs
```

目标：评估安全风险，识别攻击面。

## 文档索引

### 按主题索引

**升级流程**
- [01_Architecture.md#升级流程](./01_Architecture.md#升级流程)
- [03_Public_API.md#updaterkits](./03_Public_API.md#updaterkits)
- [appendix/Callgraphs.md#ota-升级调用链](./appendix/Callgraphs.md#ota-升级调用链)

**分区管理**
- [01_Architecture.md#分区模型](./01_Architecture.md#分区模型)
- [03_Public_API.md#misc_info](./03_Public_API.md#misc_info)
- [03_Public_API.md#slot_info](./03_Public_API.md#slot_info)

**包管理**
- [03_Public_API.md#packages](./03_Public_API.md#packages)
- [04_Inner_API.md#pkg_manager](./04_Inner_API.md#pkg_manager)

**安全验证**
- [01_Architecture.md#安全模型](./01_Architecture.md#安全模型)
- [06_Security_Analysis.md#攻击面清单](./06_Security_Analysis.md#攻击面清单)

**构建配置**
- [05_GN_Build.md#feature-标志](./05_GN_Build.md#feature-标志)
- [appendix/Config_Flags.md](./appendix/Config_Flags.md)

## 外部参考

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [升级子系统 README](../README.md)
- [升级子系统中文 README](../README_zh.md)

---

**提示**: 使用浏览器的查找功能（Ctrl+F）在本文档中搜索关键词可快速定位相关页面。
