# OpenHarmony Developer Test Framework - Wiki

## 项目概述

本文档是 OpenHarmony **Developer Test Framework (developer_test)** 的工程 Wiki，旨在帮助开发者快速理解项目架构、使用方法、构建系统和安全风险。

### 覆盖范围

| 文档 | 说明 |
|------|------|
| [README.md](README.md) | 本文档，包含 Wiki 使用说明 |
| [SUMMARY.md](SUMMARY.md) | 全站导航，新人阅读路线 |
| [01_Overview.md](01_Overview.md) | 项目定位、核心能力、运行环境 |
| [02_Architecture.md](02_Architecture.md) | 系统架构、组件图、数据流 |
| [03_Directory_Structure.md](03_Directory_Structure.md) | 目录结构与模块职责 |
| [04_Configuration.md](04_Configuration.md) | 配置项详解 |
| [05_Build_System.md](05_Build_System.md) | GN 构建系统与 targets |
| [06_Usage_Guide.md](06_Usage_Guide.md) | 使用指南与命令参考 |
| [07_Examples.md](07_Examples.md) | 测试用例示例说明 |
| [08_Security_Review.md](08_Security_Review.md) | 安全风险评审 |

### 适用范围

- **目标系统**: OpenHarmony (mini/small/standard)
- **适用开发者**: 测试框架使用者、测试用例开发者、框架维护者
- **前置要求**: Python 3.7.5+, OpenHarmony SDK, HDC 工具

### 更新方式

当代码变更时，请同步更新相关 Wiki 章节：
1. 新增模块 → 更新 `03_Directory_Structure.md`
2. 修改配置 → 更新 `04_Configuration.md`
3. 修改构建 → 更新 `05_Build_System.md`
4. 新增命令 → 更新 `06_Usage_Guide.md`

### 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 3.1.0
- **仓库路径**: `test/testfwk/developer_test`

### 相关链接

- [OpenHarmony 测试子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/test.md)
- [XDevice 测试调度框架](https://gitee.com/openharmony/testfwk_xdevice)
- [官方测试开发文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/device-test/developer_test.md)
