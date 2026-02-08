# HDF Core 工程 Wiki

## 项目概述

本文档是 OpenHarmony HDF Core（Hardware Driver Foundation Core）的工程 Wiki，旨在帮助开发者快速理解项目架构、接口定义、构建系统和安全风险。

**生成时间**: 2025-02-06  
**覆盖版本**: 4.0  
**仓库路径**: `//drivers/hdf_core`

## 覆盖范围

### 已覆盖内容
- 项目定位与核心能力
- 目录结构与模块职责
- 架构说明（组件图、数据流、线程模型）
- 对外 API（HDI 接口、导出符号）
- 内部 API（模块接口、依赖方向）
- GN 构建目标与编译产物
- 安全风险评审（攻击面、信任边界、可利用点）

### 未覆盖内容
- **N-API**: 本项目为纯 C/C++ 驱动框架，无 JavaScript 绑定层
- 测试代码（`test/`, `unittest/`, `fuzztest/` 等）
- 具体驱动实现细节（请参考各驱动模块文档）

## 阅读指南

### 新人阅读顺序
1. [项目概览](./01_Overview.md) - 了解 HDF 是什么、能做什么
2. [目录结构](./02_Directory_Structure.md) - 熟悉代码组织方式
3. [架构说明](./03_Architecture.md) - 理解整体架构设计
4. [HDI 接口](./04_HDI_API.md) - 了解对外接口
5. [内部 API](./05_Inner_API.md) - 了解内部模块接口
6. [GN 构建](./06_GN_Build.md) - 了解构建系统
7. [安全风险](./07_Security.md) - 了解安全注意事项

### 快速导航
- [SUMMARY.md](./SUMMARY.md) - 完整文档导航
- [附录：调用链](./appendix/Callgraphs.md) - 关键调用链
- [附录：配置项](./appendix/Config_Flags.md) - 关键宏和配置

## 更新方式

本文档基于代码分析自动生成，建议随代码版本更新而更新：

1. 修改代码后，检查相关 Wiki 页面是否需要更新
2. 新增模块时，在对应章节添加说明
3. 安全相关修改必须同步更新 [安全风险](./07_Security.md)

## 术语表

| 术语 | 说明 |
|------|------|
| HDF | Hardware Driver Foundation，硬件驱动框架 |
| KHDF | Kernel HDF，内核态驱动框架 |
| UHDF | User HDF，用户态驱动框架 |
| HDI | Hardware Driver Interface，硬件驱动接口 |
| HCS | HDF Configuration Source，HDF 配置源 |
| SA | System Ability，系统能力 |
| DevHost | 驱动主机进程 |
| DevMgr | 设备管理器 |

## 参考链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Driver 子系统说明](https://gitee.com/openharmony/docs/blob/master/en/readme/driver.md)
- [HDF 开发指南](https://gitee.com/openharmony/docs/blob/master/en/device-dev/driver/driver-hdf-manage.md)
