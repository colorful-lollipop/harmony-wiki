# AdminProvisioning Wiki 使用说明

## 文档概述

本文档为 OpenHarmony `admin_provisioning` 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、能力边界、API 接口、构建系统及安全风险。

> **重要说明**：本 Wiki 由代码自动生成，文档内容基于代码证据（文件路径、符号定义、调用链）编写。如发现文档与代码不符，请以代码为准并反馈修正。

## 覆盖范围

### 已覆盖内容
- 项目定位与核心能力
- 目录结构与模块职责
- 架构设计（Ability 组件、数据流）
- ArkTS/JS API 接口清单
- GN 构建系统配置
- 编译产物与安装路径
- 安全风险评审

### 未覆盖内容
- 测试代码相关文档（遵循规范不引用测试）
- 第三方依赖的详细 API（请参阅 OpenHarmony 官方文档）

## 文档更新方式

### 何时需要更新 Wiki
1. 新增/删除/修改 ArkTS API 接口
2. 新增/删除 Ability 组件
3. 修改构建配置（BUILD.gn）
4. 新增安全相关逻辑
5. 目录结构调整

### 更新步骤
1. 修改对应的 `wiki/*.md` 文件
2. 确保引用路径和符号名与代码一致
3. 更新 `SUMMARY.md` 导航（如有新增页面）
4. 运行文档校验（Phase 7）

## 快速导航

| 主题 | 入口文件 |
|-----|---------|
| 项目概览 | [README_zh.md](../README_zh.md) / [index.md](./index.md) |
| API 参考 | [04_API_Reference.md](./04_API_Reference.md) |
| 构建系统 | [05_Build_System.md](./05_Build_System.md) |
| 安全评审 | [07_Security_Review.md](./07_Security_Review.md) |

## 相关链接

- **OpenHarmony 官方文档**: https://gitee.com/openharmony/docs
- **AdminProvisioning 仓库**: https://gitee.com/openharmony/applications_admin_provisioning
- **MDM 企业设备管理**: https://gitee.com/openharmony/customization_enterprise_device_management

## 生成信息

- **生成时间**: 2026-02-05
- **生成工具**: OpenHarmony 工程 Wiki 生成 Agent
- **代码版本**: 基于当前仓库 HEAD

---

*最后更新: 2026-02-05*
