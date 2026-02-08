# device_qemu Wiki 说明

## 文档概述

本文档为 OpenHarmony device_qemu 仓库的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、构建系统和安全特性。

## 覆盖范围

### 已覆盖内容

| 类别 | 说明 |
|------|------|
| **项目定位** | device_qemu 的功能定位、适用范围、核心能力 |
| **目录结构** | 顶层目录和模块职责划分 |
| **架构设计** | 驱动模块架构、HDF 框架集成、VirtIO 虚拟化设备 |
| **构建系统** | GN 构建配置、ohos.build 文件、lite.mk 兼容层 |
| **支持平台** | ARM、RISC-V、Xtensa (ESP32)、C-SKY 等架构支持 |
| **安全评审** | 攻击面分析、信任边界、潜在风险识别 |
| **常见问题** | 构建、运行、调试问题的排查指南 |

### 未覆盖内容

| 类别 | 原因 |
|------|------|
| **N-API 接口** | device_qemu 是底层设备模拟层，不提供 JS/TS 接口 |
| **OpenHarmony IPC/Skeleton** | 设备驱动通过内核 API 交互，不直接参与用户态 IPC 框架 |
| **测试用例** | 根据规范，不引用测试代码作为业务证据 |
| **详细代码注释** | 代码级注释请参考源代码 |

## 文档清单

本 Wiki 共包含 **15个文档文件**：

| 类别 | 文档数 | 说明 |
|------|--------|------|
| **核心文档** | 11 | README、SUMMARY、首页、概览、结构、架构、构建、产物、安全、平台、问题 |
| **附录** | 2 | 调用链图、配置标志 |
| **工作区** | 2 | 事实记录 (NOTES.md)、任务计划 (PLAN.md) |

## 文档更新方式

### 更新时机

当发生以下变更时，应更新本文档：

1. 新增或移除支持的硬件平台
2. 修改 GN 构建配置（新增 target、变更依赖）
3. 新增或修改驱动模块功能
4. 安全相关的代码变更

### 更新流程

1. 在 `wiki/_work/NOTES.md` 中记录变更事实
2. 更新对应的 Wiki 页面
3. 更新 `SUMMARY.md` 确保导航链接正确
4. 执行一致性校验

### 版本与时间

| 项目 | 值 |
|------|-----|
| **文档版本** | 1.0.0 |
| **生成时间** | 2026-02-06 |
| **仓库版本** | OpenHarmony device_qemu (HEAD) |

## 相关链接

- **源码仓库**: [device_qemu (Gitee)](https://gitee.com/openharmony/device_qemu)
- **官方文档**: [OpenHarmony 文档](https://gitee.com/openharmony/docs)
- **构建系统**: [OpenHarmony 构建指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/编译构建子系统.md)
- **驱动框架**: [HDF (Hardware Driver Foundation)](https://gitee.com/openharmony/drivers_hdf_core)

## 贡献指南

欢迎为本 Wiki 贡献内容：

1. 确保关键结论有代码证据支撑（文件路径 + 符号）
2. 忽略测试相关内容
3. 文档语言：简体中文
4. 提交前执行一致性校验
