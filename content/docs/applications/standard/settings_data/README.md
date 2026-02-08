# SettingsData Wiki 文档

## 文档概述

本文档为 OpenHarmony 系统应用 **SettingsData** 的工程Wiki，覆盖项目的完整技术细节，包括架构设计、API参考、构建流程和安全评审。

### 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目定位与核心能力 | ✅ 完成 | SettingsData 系统应用定位 |
| 目录结构与模块职责 | ✅ 完成 | 源码、配置、资源文件组织 |
| 架构设计 | ✅ 完成 | DataAbility + DataShare 架构 |
| API 参考 | ✅ 完成 | DataShare 接口与 URI 配置 |
| 构建与编译 | ✅ 完成 | hvigor 构建配置与 HAP 产物 |
| 安全评审 | ✅ 完成 | 威胁模型与风险分析 |
| 附录（调用链、配置项） | ✅ 完成 | 关键调用流程与配置 |

### 未覆盖范围

- 测试用例与测试代码（按照规范要求忽略）
- 其他 OpenHarmony 子系统的内部实现
- 运行时具体行为（需要实际设备验证）

### 文档更新方式

本文档基于代码静态分析生成。如需更新：

1. 修改源代码或配置文件后，重新运行文档生成脚本
2. 或手动更新对应章节并补充代码证据

### 生成信息

- **生成时间**: 2026-02-06
- **项目版本**: OpenHarmony SettingsData
- **SDK 版本**: compileSdkVersion 23
- **代码仓库**: `/Volumes/lexar/code/d/work/oh/applications/standard/settings_data`

### 反馈与贡献

如发现文档错误或遗漏，请提交 Issue 或 PR 到对应仓库。

---

## 快速导航

- [首页](./index.md)
- [项目概览](./01_Project_Overview.md)
- [目录结构](./02_Directory_Structure.md)
- [架构设计](./03_Architecture.md)
- [API 参考](./04_API_Reference.md)
- [构建与编译](./05_Build_and_Compilation.md)
- [安全评审](./06_Security_Review.md)
