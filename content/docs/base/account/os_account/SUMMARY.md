# os_account 子系统 Wiki 导航

> 本导航页面自动生成，最后更新时间：2026-02-06

## 快速入口

- [首页 / 概述](./README.md)
- [代码仓库](https://gitee.com/openharmony/base_account_os_account)

---

## 新人阅读路线

### 路线 1：快速概览（10 分钟）
1. [README.md](./README.md) → 项目定位与架构
2. [01_Overview.md](./01_Overview.md) → 核心能力一览

### 路线 2：开发者视角（30 分钟）
1. [README.md](./README.md)
2. [01_Overview.md](./01_Overview.md)
3. [03_NAPI_Interfaces.md](./03_NAPI_Interfaces.md) → 了解对外 API
4. [06_Build_System.md](./06_Build_System.md) → 编译与构建

### 路线 3：系统集成（60 分钟）
1. 全部文档按顺序阅读
2. 重点关注 [05_Service_IPC.md](./05_Service_IPC.md) 和 [07_Security.md](./07_Security.md)

---

## 完整文档列表

### 1. 入门指南

| 文档 | 说明 | 难度 |
|------|------|------|
| [README.md](./README.md) | 项目概述、架构图、更新指南 | ⭐ |
| [01_Overview.md](./01_Overview.md) | 核心功能、模块划分、Feature Flags | ⭐⭐ |

### 2. 架构与模块

| 文档 | 说明 | 难度 |
|------|------|------|
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构与模块职责 | ⭐⭐ |
| [05_Service_IPC.md](./05_Service_IPC.md) | 服务架构与 IPC 通信机制 | ⭐⭐⭐ |

### 3. 接口文档

| 文档 | 说明 | 难度 |
|------|------|------|
| [03_NAPI_Interfaces.md](./03_NAPI_Interfaces.md) | JS/TS 对外接口完整清单 | ⭐⭐ |
| [04_Inner_API.md](./04_Inner_API.md) | C++ 内部接口与依赖关系 | ⭐⭐⭐ |

### 4. 构建与配置

| 文档 | 说明 | 难度 |
|------|------|------|
| [06_Build_System.md](./06_Build_System.md) | GN 构建配置与编译产物 | ⭐⭐ |
| [08_Config_Flags.md](./08_Config_Flags.md) | Feature Flags 与编译开关 | ⭐⭐ |

### 5. 安全与运维

| 文档 | 说明 | 难度 |
|------|------|------|
| [07_Security.md](./07_Security.md) | 权限模型、攻击面、安全建议 | ⭐⭐⭐ |
| [09_FAQ_Debug.md](./09_FAQ_Debug.md) | 常见问题与调试指南 | ⭐⭐ |

---

## 文档更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-06 | 1.0 | 初始版本，基础文档结构 |

---

## 贡献指南

### 如何贡献

1. **发现错误**：直接在对应文档页面修改或提 Issue
2. **添加内容**：遵循现有文档格式，添加代码证据
3. **提出建议**：通过 PR 或 Issue 反馈

### 质量要求

- 所有结论必须有代码证据（文件路径+符号）
- 禁止引用测试代码
- 术语需与代码保持一致
- 保持文档结构清晰

---

## 关联项目

| 项目 | 说明 |
|------|------|
| [OpenHarmony Docs](https://gitee.com/openharmony/docs) | 官方文档仓库 |
| [Account N-API](https://gitee.com/openharmony/interface_sdk-js/blob/master/api/@ohos.account.osAccount.d.ts) | API 类型定义 |
| [Account IAM](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/User-Authentication.md) | 用户认证文档 |
