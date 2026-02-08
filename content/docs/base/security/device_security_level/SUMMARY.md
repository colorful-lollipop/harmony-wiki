# DSLM Wiki 文档导航

> 最后更新：2026-02-07
> 版本：v3.0.0

---

## 文档总览

本文档集为 OpenHarmony **Device Security Level Management (DSLM)** 模块的完整工程文档，涵盖项目概览、架构设计、接口文档、安全分析和内部实现。

---

## 新人学习路线

> 建议按以下顺序阅读，逐步深入理解 DSLM 模块

### 第一阶段：快速入门

| 顺序 | 文档 | 内容 | 预计时间 |
|------|------|------|----------|
| 1️⃣ | [01_Overview.md](./01_Overview.md) | 项目定位、安全等级体系、核心能力 | 15 分钟 |
| 2️⃣ | [03_CodeMap.md](./03_CodeMap.md) | 目录结构、核心文件定位 | 15 分钟 |

### 第二阶段：深入理解

| 顺序 | 文档 | 内容 | 预计时间 |
|------|------|------|----------|
| 3️⃣ | [02_Architecture.md](./02_Architecture.md) | 系统架构、数据流、状态机设计 | 30 分钟 |
| 4️⃣ | [04_Interface.md](./04_Interface.md) | C API 参考、使用示例、错误处理 | 20 分钟 |

### 第三阶段：工程实践

| 顺序 | 文档 | 内容 | 预计时间 |
|------|------|------|----------|
| 5️⃣ | [07_Build.md](./07_Build.md) | GN Targets、编译产物、依赖关系 | 15 分钟 |
| 6️⃣ | [08_Internals.md](./08_Internals.md) | 内部实现、数据结构、插件机制 | 30 分钟 |

---

## 安全研究路线

> 建议按以下顺序阅读，快速定位攻击面和安全风险

### 第一阶段：攻击面识别

| 顺序 | 文档 | 内容 | 预计时间 |
|------|------|------|----------|
| 1️⃣ | [05_AttackSurface.md](./05_AttackSurface.md) | 外部输入清单、敏感操作、信任边界 | 20 分钟 |
| 2️⃣ | [02_Architecture.md](./02_Architecture.md) | 模块划分、数据流、IPC 接口 | 15 分钟 |

### 第二阶段：风险评估

| 顺序 | 文档 | 内容 | 预计时间 |
|------|------|------|----------|
| 3️⃣ | [06_SecurityReview.md](./06_SecurityReview.md) | 7 类安全风险、证据链、修复建议 | 40 分钟 |
| 4️⃣ | [04_Interface.md](./04_Interface.md) | API 参数验证、错误码说明 | 15 分钟 |

### 第三阶段：深度分析

| 顺序 | 文档 | 内容 | 预计时间 |
|------|------|------|----------|
| 5️⃣ | [08_Internals.md](./08_Internals.md) | 凭证格式、签名验证、内存安全 | 30 分钟 |
| 6️⃣ | [03_CodeMap.md](./03_CodeMap.md) | 代码文件定位、风险点追踪 | 15 分钟 |

---

## 文档索引

| 编号 | 文档 | 说明 | 受众 | 预计时间 |
|------|------|------|------|----------|
| **01** | [01_Overview.md](./01_Overview.md) | 项目概览、安全等级、运行环境 | 新人 | 15 分钟 |
| **02** | [02_Architecture.md](./02_Architecture.md) | 系统架构、数据流、状态机 | 新人/安全 | 30 分钟 |
| **03** | [03_CodeMap.md](./03_CodeMap.md) | 目录结构、文件定位 | 新人 | 15 分钟 |
| **04** | [04_Interface.md](./04_Interface.md) | C API 参考、错误码 | 新人 | 20 分钟 |
| **05** | [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面、信任边界 | 安全 | 20 分钟 |
| **06** | [06_SecurityReview.md](./06_SecurityReview.md) | 安全风险评估与修复 | 安全 | 40 分钟 |
| **07** | [07_Build.md](./07_Build.md) | 构建配置、产物清单 | 开发者 | 15 分钟 |
| **08** | [08_Internals.md](./08_Internals.md) | 内部实现、插件机制 | 高级开发者 | 30 分钟 |

---

## 关键概念速查

| 概念 | 说明 | 相关文档 |
|------|------|----------|
| **SL1-SL5** | OpenHarmony 设备安全等级（1-5 级，SL5 最高） | [01_Overview.md](./01_Overview.md) |
| **SA ID 3511** | Device Security Level Manager 的 System Ability ID | [02_Architecture.md](./02_Architecture.md) |
| **C API** | DSLM 提供的 Native C 接口（非 N-API） | [04_Interface.md](./04_Interface.md) |
| **DslmDeviceInfo** | 设备信息结构体，包含状态机和凭证信息 | [08_Internals.md](./08_Internals.md) |
| **JWS Credential** | JSON Web Signature 格式的设备凭证 | [08_Internals.md](./08_Internals.md) |
| **DSL** | Device Security Level 模块缩写 | 全局 |

---

## 代码证据索引

### 核心 API

| 功能 | 头文件 | 关键函数 |
|------|--------|----------|
| 同步查询 | `device_security_info.h:42` | `RequestDeviceSecurityInfo()` |
| 异步查询 | `device_security_info.h:53` | `RequestDeviceSecurityInfoAsync()` |
| 释放资源 | `device_security_info.h:60` | `FreeDeviceSecurityInfo()` |
| 提取等级 | `device_security_info.h:68` | `GetDeviceSecurityLevelValue()` |

### 服务注册

| 功能 | 文件 | 行号 |
|------|------|------|
| SA 注册 | `dslm_service.cpp` | 38 |
| 服务启动 | `dslm_service.cpp` | 83 |
| IPC 处理 | `dslm_ipc_process.cpp` | 62 |

### 凭证验证

| 功能 | 文件 | 行号 |
|------|------|------|
| JWS 解析 | `dslm_credential_utils.c` | 89 |
| ECDSA 验证 | `dslm_credential_utils.c` | 522 |
| Nonce 验证 | `dslm_ohos_verify.c` | 68 |

---

## 快速参考

### SDK 使用模板

```cpp
#include "device_security_defines.h"
#include "device_security_info.h"

// 同步查询
int32_t ret = RequestDeviceSecurityInfo(device, NULL, &info);
if (ret == SUCCESS) {
    GetDeviceSecurityLevelValue(info, &level);
}
FreeDeviceSecurityInfo(info);

// 异步查询
RequestDeviceSecurityInfoAsync(device, NULL, callback);
```

### 错误码速查

| 错误码 | 含义 |
|--------|------|
| 0 (`SUCCESS`) | 成功 |
| 1 (`ERR_INVALID_PARA`) | 无效参数 |
| 8 (`ERR_TIMEOUT`) | 超时 |
| 14 (`ERR_NOT_ONLINE`) | 设备不在线 |
| 30 (`ERR_PERMISSION_DENIAL`) | 权限拒绝 |

---

## 文档规范

### 编写标准

- **语言**：中文（专业术语保留英文）
- **证据**：关键结论必须包含代码路径和行号
- **范围**：不涉及 test/ 测试代码
- **更新**：代码变更后需同步更新文档

### 证据格式

```markdown
**证据**：`interfaces/inner_api/include/device_security_info.h:42-43`
```

---

## 相关资源

### 代码仓库

- **主仓库**: `base/security/device_security_level`
- **上游文档**: [README.md](../README.md) | [README_ZH.md](../README_ZH.md)

### 关联组件

| 组件 | 关系 | 说明 |
|------|------|------|
| [HUKS](https://gitee.com/openharmony/security_huks) | 依赖 | 硬件密钥服务 |
| [Device Auth](https://gitee.com/openharmony/security_device_auth) | 依赖 | 设备认证服务 |
| [Data Transfer Management](https://gitee.com/openharmony/security_dataclassification) | 上游 | 数据风险等级映射 |

---

**文档版本**: v3.0.0
**最后更新**: 2026-02-07
