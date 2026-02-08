# 导航索引

本文档是 `security_cangjie_wrapper` 项目的完整导航索引，提供双路线阅读指南。

---

## 🎓 新人学习路线（快速上手）

**目标**：30 分钟内理解项目定位、API 使用和基本架构

| 顺序 | 文档 | 核心内容 | 预计时间 |
|------|------|----------|----------|
| 1 | [概览](00_Overview.md) | 项目定位、核心能力、约束与限制 | 5 min |
| 2 | [目录结构](01_Directory_Structure.md) | 模块划分、文件组织、代码地图 | 5 min |
| 3 | [N-API 参考 - 快速开始](03_N-API_Reference.md#crypto-framework) | Kit API 清单与使用示例 | 10 min |
| 4 | [架构说明](02_Architecture.md) | FFI 机制、数据流、调用链路 | 10 min |

**进阶路径**：
- **加密算法开发**：`概览` → `N-API 参考 - Crypto Framework` → `目录结构 - crypto_framework/`
- **密钥管理开发**：`概览` → `N-API 参考 - HUKS` → `目录结构 - huks/`
- **问题排查**：`架构说明` → `内部 API` → `GN Targets` → `安全评审`

---

## 🔒 安全研究路线（深度分析）

**目标**：快速识别攻击面、信任边界和潜在漏洞

| 顺序 | 文档 | 核心内容 | 预计时间 |
|------|------|----------|----------|
| 1 | [安全评审](07_Security_Review.md) | 攻击面、信任边界、风险清单 | 15 min |
| 2 | [架构说明](02_Architecture.md) | FFI 桥接机制、内存管理、数据流 | 10 min |
| 3 | [目录结构](01_Directory_Structure.md) | 模块职责、关键文件定位 | 5 min |
| 4 | [N-API 参考](03_N-API_Reference.md) | API 入口点、参数类型、错误处理 | 15 min |
| 5 | [内部 API](04_Internal_API.md) | 模块间接口、稳定性标注 | 10 min |

**深度分析路径**：
- **内存安全**：`安全评审` → `架构说明 - FFI 桥接机制` → 源码分析（`cj_crypto_native.cj`、`huks_struct_ffi.cj`）
- **输入验证**：`安全评审 - 攻击面` → `N-API 参考 - HUKS` → `huks_key_item.cj`
- **并发安全**：`安全评审 - 并发风险` → `架构说明 - 线程模型` → 源码分析

---

## 📦 构建与部署路线

| 顺序 | 文档 | 核心内容 | 预计时间 |
|------|------|----------|----------|
| 1 | [目录结构](01_Directory_Structure.md#构建配置) | BUILD.gn 组织、模块依赖 | 5 min |
| 2 | [GN Targets](05_GN_Targets.md) | 构建目标清单、依赖关系图 | 10 min |
| 3 | [编译产物](06_Build_Artifacts.md) | 产物类型、安装路径、平台适配 | 5 min |

---

## 快速跳转

### 按功能分类导航

**🔐 加密算法相关**：
- [概览 - Crypto Architecture Kit](00_Overview.md#crypto-architecture-kit)
- [N-API - Crypto Framework](03_N-API_Reference.md#crypto-framework)
- [架构说明 - FFI 桥接机制](02_Architecture.md#ffi-桥接机制)
- [内部 API - Crypto 模块](04_Internal_API.md#crypto-framework-模块)

**🔑 密钥管理相关**：
- [概览 - Universal Keystore Kit](00_Overview.md#universal-keystore-kit)
- [N-API - HUKS](03_N-API_Reference.md#huks-密钥管理)
- [架构说明 - HUKS 数据流](02_Architecture.md#huks-调用链)
- [内部 API - HUKS 模块](04_Internal_API.md#huks-模块)

**🏗️ 构建相关**：
- [GN Targets](05_GN_Targets.md)
- [编译产物](06_Build_Artifacts.md)
- [目录结构 - 构建配置](01_Directory_Structure.md#构建配置)

**🛡️ 安全相关**：
- [安全评审](07_Security_Review.md)
- [架构说明 - 错误处理机制](02_Architecture.md#错误处理机制)
- [N-API - 错误码](03_N-API_Reference.md#错误码对照表)

### 按场景分类导航

**🚀 新项目集成**：
1. [概览](00_Overview.md) → 确认能力满足需求（5 min）
2. [N-API 参考](03_N-API_Reference.md) → 选择合适 API（10 min）
3. [目录结构](01_Directory_Structure.md) → 定位源码位置（5 min）
4. [编译产物](06_Build_Artifacts.md) → 确定依赖方式（5 min）
**总耗时**：约 25 分钟

**🔧 问题排查**：
1. [架构说明](02_Architecture.md) → 理解调用链路（10 min）
2. [内部 API](04_Internal_API.md) → 定位模块边界（10 min）
3. [安全评审](07_Security_Review.md) → 检查潜在风险（15 min）
4. [GN Targets](05_GN_Targets.md) → 确认依赖配置（10 min）
**总耗时**：约 45 分钟

**👨‍💻 代码贡献**：
1. [目录结构](01_Directory_Structure.md) → 找到修改位置（5 min）
2. [GN Targets](05_GN_Targets.md) → 更新构建配置（10 min）
3. [安全评审](07_Security_Review.md) → 评估安全影响（15 min）
4. [架构说明](02_Architecture.md) → 理解 FFI 约束（10 min）
**总耗时**：约 40 分钟

---

## 术语表

| 术语 | 含义 |
|------|------|
| **FFI** | Foreign Function Interface，外部函数接口，Cangjie 与 C 语言互调机制 |
| **HUKS** | Huawei Universal KeyStore，通用密钥管理，提供密钥生命周期管理 |
| **Crypto Framework** | 加密算法框架，提供基础加密、解密、摘要、随机数等能力 |
| **SysCap** | System Capability，系统能力标识 |
| **APILevel** | API 版本标注注解，标识 API 的稳定性和向后兼容性 |
| **RemoteDataLite** | 远程数据生命期管理基类，持有 Native 层对象句柄 |
| **Worker Thread** | 工作线程，支持后台执行的 API 标注 |

---

## 外部参考

| 资源 | 链接 | 说明 |
|------|--------|------|
| **项目 README** | [README.md](../README.md) | 项目官方说明 |
| **Cangjie API 文档** | [arkcompiler_cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) | Cangjie 安全 API 官方文档 |
| **Crypto Framework** | [security_crypto_framework](https://gitcode.com/openharmony/security_crypto_framework) | 加密框架底层实现 |
| **HUKS** | [security_huks](https://gitcode.com/openharmony/security_huks) | 密钥管理底层实现 |
| **OpenHarmony 安全指南** | [安全开发指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/security/security-guidelines.md) | 安全开发最佳实践 |

---

## 文档状态

| 文档 | 状态 | 最后更新 |
|------|------|----------|
| 项目评估（ASSESSMENT.md） | ✅ 完成 | 2025-02-07 |
| 概览（00_Overview.md） | ✅ 完成 | - |
| 目录结构（01_Directory_Structure.md） | ✅ 完成 | - |
| 架构说明（02_Architecture.md） | ✅ 完成 | - |
| N-API 参考（03_N-API_Reference.md） | ✅ 完成 | - |
| 内部 API（04_Internal_API.md） | ✅ 完成 | - |
| GN Targets（05_GN_Targets.md） | ✅ 完成 | - |
| 编译产物（06_Build_Artifacts.md） | ✅ 完成 | - |
| 安全评审（07_Security_Review.md） | ✅ 完成 | - |

---

## 快速查找

### 按错误码查找
- **Crypto Framework 错误码**：[N-API 参考 - Result 枚举](03_N-API_Reference.md#11-result错误结果枚举)
- **HUKS 错误码**：[N-API 参考 - HuksExceptionErrCode](03_N-API_Reference.md#huksexceptionerrcode错误码)

### 按文件查找
- **Kit 导出层**：[目录结构 - Kit 导出层](01_Directory_Structure.md#kit-导出层kit)
- **Crypto Framework 实现**：[目录结构 - crypto_framework/](01_Directory_Structure.md#crypto_framework)
- **HUKS 实现**：[目录结构 - huks/](01_Directory_Structure.md#huks)
- **FFI 文件**：[架构说明 - FFI 桥接机制](02_Architecture.md#ffi-桥接机制)

### 按风险类型查找
- **内存安全**：[安全评审 - 内存安全风险](07_Security_Review.md#🟡-中风险)
- **输入验证**：[安全评审 - 输入验证风险](07_Security_Review.md#🔴-高风险)
- **并发安全**：[安全评审 - 并发风险](07_Security_Review.md#🟢-低风险)
