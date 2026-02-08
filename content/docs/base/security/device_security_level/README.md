# Device Security Level Management (DSLM) Wiki

> OpenHarmony 设备安全等级管理模块完整工程文档

---

## 文档说明

本文档集为 OpenHarmony **Device Security Level Management (DSLM)** 模块的完整工程文档，旨在帮助新人快速理解项目架构、设计、构建和安全机制。

### 覆盖范围

本文档涵盖以下内容：

- ✅ 项目定位与核心能力
- ✅ 系统架构与数据流
- ✅ 对外 C API 接口
- ✅ 攻击面与安全风险分析
- ✅ GN 构建系统与编译产物
- ✅ 内部实现细节

### 未覆盖范围

- ❌ 测试代码（test/ 目录内容不在本文档范围内）
- ❌ 外部依赖的详细文档（如 dsoftbus、device_auth 等）
- ❌ 具体的业务流程代码实现细节（参考源代码）

---

## 适用对象

| 受众 | 推荐阅读路线 |
|------|-------------|
| **新人学习者** | [SUMMARY.md](./SUMMARY.md) - 新人学习路线 |
| **安全研究员** | [SUMMARY.md](./SUMMARY.md) - 安全研究路线 |
| **集成开发者** | [04_Interface.md](./04_Interface.md) → [07_Build.md](./07_Build.md) |
| **安全审计人员** | [05_AttackSurface.md](./05_AttackSurface.md) → [06_SecurityReview.md](./06_SecurityReview.md) |

---

## 快速开始

### 新人：10 分钟快速上手

1. 阅读 [01_Overview.md](./01_Overview.md) 了解项目定位（5 分钟）
2. 查看 [04_Interface.md](./04_Interface.md) 快速集成 SDK（5 分钟）

### 安全评估：快速定位风险

1. 阅读 [05_AttackSurface.md](./05_AttackSurface.md) 了解攻击面（10 分钟）
2. 查看 [06_SecurityReview.md](./06_SecurityReview.md) 分析风险（20 分钟）

---

## 文档列表

| 编号 | 文档 | 说明 | 核心受众 |
|------|------|------|----------|
| 01 | [01_Overview.md](./01_Overview.md) | 项目概览、安全等级、运行环境 | 新人 |
| 02 | [02_Architecture.md](./02_Architecture.md) | 系统架构、数据流、状态机 | 新人/安全 |
| 03 | [03_CodeMap.md](./03_CodeMap.md) | 目录结构、文件定位 | 新人 |
| 04 | [04_Interface.md](./04_Interface.md) | C API 参考、使用示例 | 开发者 |
| 05 | [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面、信任边界 | 安全 |
| 06 | [06_SecurityReview.md](./06_SecurityReview.md) | 安全风险评估与修复 | 安全 |
| 07 | [07_Build.md](./07_Build.md) | 构建配置、产物清单 | 开发者 |
| 08 | [08_Internals.md](./08_Internals.md) | 内部实现、插件机制 | 高级开发者 |

完整导航请查看 [SUMMARY.md](./SUMMARY.md)。

---

## 关键概念

| 概念 | 说明 | 相关文档 |
|------|------|----------|
| **SL1-SL5** | OpenHarmony 设备安全等级（SL5 最高） | [01_Overview.md](./01_Overview.md) |
| **SA ID 3511** | DSLM System Ability 标识符 | [02_Architecture.md](./02_Architecture.md) |
| **C API** | DSLM Native C 接口（非 N-API） | [04_Interface.md](./04_Interface.md) |
| **JWS Credential** | JSON Web Signature 格式设备凭证 | [08_Internals.md](./08_Internals.md) |

---

## 版本信息

| 属性 | 值 |
|------|-----|
| **模块版本** | v3.0.0 |
| **文档版本** | v3.0.0 |
| **最后更新** | 2026-02-07 |

---

## 相关资源

### 代码仓库

- **代码路径**: `base/security/device_security_level`
- **上游文档**: [README.md](../README.md) | [README_ZH.md](../README_ZH.md)

### 关联组件

| 组件 | 关系 | 用途 |
|------|------|------|
| [HUKS](https://gitee.com/openharmony/security_huks) | 依赖 | 硬件密钥服务 |
| [Device Auth](https://gitee.com/openharmony/security_device_auth) | 依赖 | 设备认证 |
| [Data Transfer Management](https://gitee.com/openharmony/security_dataclassification) | 上游 | 数据风险分级 |

---

## 文档维护

### 更新方式

1. **代码变更后**：同步更新相关文档章节
2. **新增功能**：在相应章节补充说明
3. **发现错误**：在对应文档中修正并标注版本

### 质量标准

- [ ] 所有技术结论有代码证据支撑
- [ ] 文档链接有效可访问
- [ ] 术语统一，内部链接正确
- [ ] 代码片段有语法标注

---

**文档作者**: OpenHarmony 工程 Wiki 生成 Agent
**许可协议**: Apache License 2.0
