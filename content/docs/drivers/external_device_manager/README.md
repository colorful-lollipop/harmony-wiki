# External Device Manager - Wiki 文档

## 文档概述

本文档是 OpenHarmony 扩展外部设备管理（External Device Manager）模块的完整技术文档，旨在帮助开发者快速理解项目架构、核心能力、使用方法以及安全注意事项。

### 覆盖范围

本文档涵盖以下内容：

1. **项目概览**：模块定位、核心能力、运行环境
2. **目录结构**：模块划分与职责
3. **架构设计**：组件图、数据流、线程模型
4. **对外 API**：N-API 接口、DDK 接口、权限说明
5. **内部架构**：核心模块、依赖关系、生命周期
6. **构建系统**：GN Targets、编译产物、依赖关系
7. **安全风险分析**：攻击面、信任边界、风险点与修复建议
8. **常见问题**：构建、运行、调试问题定位

### 未覆盖范围

- 测试代码相关内容（test/ 目录）
- 具体的第三方依赖实现细节
- OpenHarmony 框架通用机制的深入原理

### 更新说明

本文档基于代码版本 `external_device_manager` 分支的最新提交生成。当代码发生重大变更时，建议重新生成文档以保持同步。

**文档生成时间**：2025-02-06

### 参与贡献

如发现文档错误或需要补充，请通过 OpenHarmony 社区渠道反馈。

---

## 相关文档链接

- [OpenHarmony 驱动框架文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/驱动子系统.md)
- [HDF 驱动框架核心](https://gitee.com/openharmony/drivers_hdf_core/blob/master/README_zh.md)
- [驱动接口定义](https://gitee.com/openharmony/drivers_interface/blob/master/README_ZH.md)
- [外设驱动示例](https://gitee.com/openharmony/drivers_peripheral/blob/master/README_zh.md)
