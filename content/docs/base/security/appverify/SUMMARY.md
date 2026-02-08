# Appverify 文档导航

> OpenHarmony 应用完整性校验模块完整文档

## 概览

| 文档 | 描述 | 阅读时间 | 适用人群 |
|------|------|----------|----------|
| [README.md](README.md) | 文档使用指南与更新说明 | 5 min | 所有人 |
| [00_Overview.md](00_Overview.md) | 项目概览、核心功能、架构总览 | 5 min | 所有人 |
| [01_Project_Position.md](01_Project_Position.md) | 项目定位、边界、核心能力、运行环境 | 10 min | 新人、PM |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构、模块职责划分 | 10 min | 开发者 |
| [03_Architecture.md](03_Architecture.md) | 组件图、数据流、验证流程、时序图 | 20 min | 开发者、架构师 |
| [04_Public_API.md](04_Public_API.md) | 对外 C++ API 清单、使用示例 | 15 min | 调用方开发者 |
| [05_Internal_API.md](05_Internal_API.md) | 内部模块接口、依赖关系 | 30 min | 模块开发者 |
| [06_GN_Targets.md](06_GN_Targets.md) | GN 构建目标、依赖关系、配置 | 20 min | 构建系统开发者 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物、安装路径、运行时加载 | 15 min | 部署人员 |
| [08_Security_Review.md](08_Security_Review.md) | 安全风险评审、威胁模型、利用点分析 | 30 min | 安全审计人员 |
| [09_FAQ.md](09_FAQ.md) | 常见问题、定位路径、排查技巧 | 15 min | 开发者、运维 |

## 附录

- [Callgraphs.md](appendix/Callgraphs.md) - 关键调用链详细图解
- [Config_Flags.md](appendix/Config_Flags.md) - 关键编译宏与 Feature Flags

## 新人阅读顺序

```
START
  ↓
README.md (了解文档结构)
  ↓
00_Overview.md (快速了解项目)
  ↓
01_Project_Position.md (理解项目定位)
  ↓
02_Directory_Structure.md (熟悉代码组织)
  ↓
03_Architecture.md (理解核心流程) → 04_Public_API.md (学习如何使用)
  ↓
08_Security_Review.md (了解安全机制)
  ↓
09_FAQ.md (常见问题)
END
```

## 深入阅读路径

### 开发者路径
1. 00_Overview.md → 03_Architecture.md → 04_Public_API.md
2. 05_Internal_API.md（需要修改代码时）
3. 06_GN_Targets.md（修改构建时）
4. 09_FAQ.md（排查问题）

### 安全审计路径
1. 00_Overview.md → 03_Architecture.md
2. 08_Security_Review.md（重点）
3. 05_Internal_API.md（深入实现细节）
4. 附录 Callgraphs.md（调用链分析）

### 构建系统开发者路径
1. 00_Overview.md → 02_Directory_Structure.md
2. 06_GN_Targets.md（重点）
3. 07_Build_Artifacts.md
4. 附录 Config_Flags.md

## 按主题索引

### API 相关
- **对外接口**：[04_Public_API.md](04_Public_API.md)
- **内部接口**：[05_Internal_API.md](05_Internal_API.md)
- **API 使用示例**：[04_Public_API.md](04_Public_API.md#api-使用示例)

### 架构与设计
- **整体架构**：[03_Architecture.md](03_Architecture.md)
- **模块职责**：[02_Directory_Structure.md](02_Directory_Structure.md#模块职责)
- **数据流**：[03_Architecture.md](03_Architecture.md#数据流)
- **时序图**：[03_Architecture.md](03_Architecture.md#关键时序)

### 构建与部署
- **GN 目标**：[06_GN_Targets.md](06_GN_Targets.md)
- **编译产物**：[07_Build_Artifacts.md](07_Build_Artifacts.md)
- **配置文件**：[06_GN_Targets.md](06_GN_Targets.md#配置文件-预构建目标)
- **Feature Flags**：附录 [Config_Flags.md](appendix/Config_Flags.md)

### 安全相关
- **安全机制**：[08_Security_Review.md](08_Security_Review.md)
- **威胁模型**：[08_Security_Review.md](08_Security_Review.md#威胁模型)
- **风险清单**：[08_Security_Review.md](08_Security_Review.md#安全风险清单)
- **证书管理**：[05_Internal_API.md](05_Internal_API.md#可信源管理-trustedsource-manager)

### 调试与排查
- **FAQ**：[09_FAQ.md](09_FAQ.md)
- **常见错误**：[09_FAQ.md](09_FAQ.md#常见错误码)
- **定位路径**：[09_FAQ.md](09_FAQ.md#问题定位)
- **调试模式**：[04_Public_API.md](04_Public_API.md#调试模式)

## 术语表

| 术语 | 全称 | 说明 | 参考文档 |
|------|------|------|----------|
| HAP | Harmony Ability Package | OpenHarmony 应用安装包格式 | [01_Project_Position.md](01_Project_Position.md) |
| PKCS7 | Cryptographic Message Syntax | 签名数据封装格式 | [03_Architecture.md](03_Architecture.md#pkcs7-签名块) |
| Provision | Provisioning Profile | 应用配置文件，包含权限、分发类型等 | [03_Architecture.md](03_Architecture.md#profile-验证) |
| CRL | Certificate Revocation List | 证书吊销列表 | [05_Internal_API.md](05_Internal_API.md#证书验证) |
| InnerKit | Inner APIs | 子系统内部 C++ API | [01_Project_Position.md](01_Project_Position.md) |
| RSA-PSS | Probabilistic Signature Scheme | RSA 签名填充方案 | [03_Architecture.md](03_Architecture.md#支持的签名算法) |

## 文档版本历史

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 1.0 | 2026-02-06 | 初始版本，完整文档体系 |
