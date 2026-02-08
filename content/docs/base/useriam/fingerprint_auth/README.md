# Fingerprint Auth Wiki

## Overview

本 Wiki 提供了 OpenHarmony 指纹认证组件 (`base/useriam/fingerprint_auth`) 的完整技术文档。

**组件说明**：指纹认证服务（fingerprintauth）作为 UserIAM（用户身份认证）框架下的执行器（Executor），通过 HDF（Hardware Driver Foundation）驱动接口与硬件驱动层交互，实现指纹录入、删除、认证和识别功能。

## 文档覆盖范围

| 文档 | 内容 |
|--------|------|
| [01_Overview.md](./01_Overview.md) | 组件定位、核心能力、运行环境、关键概念 |
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构与模块职责 |
| [03_Architecture.md](./03_Architecture.md) | 架构说明：组件图、数据流、线程模型、关键时序 |
| [04_HDI_Interfaces.md](./04_HDI_Interfaces.md) | 硬件驱动接口（HDI）定义与实现 |
| [05_Internal_APIs.md](./05_Internal_APIs.md) | 内部模块接口、依赖方向、稳定性 |
| [06_GN_Targets.md](./06_GN_Targets.md) | GN 目标梳理：targets 列表、类型、依赖、产物、开关 |
| [07_Build_Artifacts.md](./07_Build_Artifacts.md) | 编译产物：.so/.a 文件、安装路径、运行时加载关系 |
| [08_Security_Analysis.md](./08_Security_Analysis.md) | 安全风险评审：攻击面、信任边界、可被利用点、修复建议 |
| [09_Troubleshooting.md](./09_Troubleshooting.md) | 常见构建/运行/调试问题与定位路径 |

## 快速导航

**新人阅读顺序**：
1. [01_Overview.md](./01_Overview.md) - 了解组件全貌
2. [02_Directory_Structure.md](./02_Directory_Structure.md) - 熟悉代码组织
3. [03_Architecture.md](./03_Architecture.md) - 理解架构设计
4. [04_HDI_Interfaces.md](./04_HDI_Interfaces.md) - 掌握 HDI 接口
5. [06_GN_Targets.md](./06_GN_Targets.md) - 了解构建系统
6. [08_Security_Analysis.md](./08_Security_Analysis.md) - 关注安全风险

**开发者重点关注**：
- [03_Architecture.md](./03_Architecture.md) - 架构设计与通信模式
- [04_HDI_Interfaces.md](./04_HDI_Interfaces.md) - HDI 接口契约
- [05_Internal_APIs.md](./05_Internal_APIs.md) - 模块间接口
- [06_GN_Targets.md](./06_GN_Targets.md) - 构建配置与依赖

**安全审核重点关注**：
- [08_Security_Analysis.md](./08_Security_Analysis.md) - 安全分析与风险清单

## 更新方式

本文档基于代码库实际内容生成。随着代码变更，应通过以下方式更新：

1. 重新生成文档：运行文档生成脚本
2. 手动更新：针对特定章节更新代码证据和示例
3. 版本控制：保持文档与代码版本同步

## 生成时间

- **生成时间**：2026-02-06
- **代码版本**：基于当前代码仓库快照

## 未覆盖范围

以下内容未包含在本 Wiki 中：

1. **N-API/JavaScript API**：本组件无 N-API 绑定。JS API 位于 `useriam_user_auth_framework` 组件。
2. **测试相关内容**：所有单元测试、模糊测试（fuzztest）相关内容均未包含。
3. **上层框架细节**：`user_auth_framework` 的详细实现不属于本文档范围。
4. **南向驱动实现**：`drivers_peripheral/fingerprint_auth` 的桩实现仅供参考，未详细分析。

## 相关资源

- **官方文档**：[OpenHarmony 用户 IAM 文档](https://docs.openharmony.cn/)
- **架构图**：`figures/fingerprintauth_architecture_ZH.png`
- **相关组件**：
  - [useriam_user_auth_framework](https://gitee.com/openharmony/useriam_user_auth_framework) - 统一用户认证框架
  - [useriam_pin_auth](https://gitee.com/openharmony/useriam_pin_auth) - PIN 认证执行器
  - [useriam_face_auth](https://gitee.com/openharmony/useriam_face_auth) - 人脸认证执行器
  - [drivers_interface_fingerprint_auth](https://gitee.com/openharmony/drivers_interface) - HDI 接口定义
