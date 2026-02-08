# API 接口差异

## 概述

**本章节说明**：由于 OpenHarmony 对 OpenSSL 采用**构建系统集成策略**，未对上游源代码进行任何修改，因此**不存在 API 接口差异**。OpenSSL 的所有公开 API 在 OpenHarmony 版本中保持与上游完全一致，依赖模块可以按照标准 OpenSSL API 文档进行开发。

OpenHarmony 的适配工作完全在配置层面完成，包括构建配置（构建目标、编译选项、源文件组织）和运行时配置（openssl.cnf）。这种设计确保了：
- API 兼容性：所有标准 OpenSSL API 均可正常使用
- 二进制兼容性：上游编译的 OpenSSL 应用可以在 OpenHarmony 上运行
- 升级便利性：新版本集成时无需修改 API 兼容层

---

## 公共头文件

OpenSSL 的公共头文件目录为 `//third_party/openssl/include`，包含以下核心头文件：

| 头文件 | 用途 |
|--------|------|
| openssl/ssl.h | TLS/SSL 协议 API |
| openssl/crypto.h | 通用密码学 API |
| openssl/evp.h | 高层密码学 API |
| openssl/x509.h | X.509 证书 API |
| openssl/err.h | 错误处理 API |
| openssl/bio.h | I/O 抽象 API |
| openssl/pem.h | PEM 格式处理 |
| openssl/pkcs12.h | PKCS#12 格式 |

所有头文件均来自上游源码，未进行任何修改。

---

## 配置宏定义

虽然 API 本身无差异，但部分预处理器宏会影响 API 行为：

| 宏定义 | 值 | 影响 |
|--------|-----|------|
| OPENSSL_SUPPRESS_DEPRECATED | 条件定义 | 废弃 API 警告抑制 |
| OPENSSL_BUILDING_OPENSSL | 定义 | 标记库内部构建 |

OPENSSL_SUPPRESS_DEPRECATED 宏由部分依赖模块（如 grpc、sqlite）在构建时定义，用于抑制 OpenSSL 内部废弃 API 的警告。这不影响 API 功能，仅影响编译警告输出。

---

## 组件系统接口

OpenHarmony 通过 inner_kits 机制定义了 OpenSSL 组件的对外接口：

| 组件名称 | 类型 | 说明 |
|----------|------|------|
| libcrypto_shared | shared_library | 动态链接加密库 |
| libcrypto_static | static_library | 静态链接加密库 |
| libssl_shared | shared_library | 动态链接 SSL 库 |
| libssl_static | static_library | 静态链接 SSL 库 |
| libcrypto_restool | shared_library | 资源工具变体 |

这些组件接口定义遵循 OpenHarmony 组件规范，是依赖模块引用 OpenSSL 时使用的标识符。

---

## 运行时配置影响

openssl.cnf 配置文件的修改会影响 OpenSSL 的运行时行为，但**不影响 API 接口**：

| 配置项 | 值 | API 影响 |
|--------|-----|----------|
| provider.default | 启用 | 无 |
| provider.legacy | 启用 | 可访问 legacy 算法 |
| security_level | 0 | 允许弱加密算法 |
| min_protocol | None | 允许旧协议版本 |

这些配置通过 OpenSSL 内部机制影响 API 的行为选择，而非修改 API 本身。

---

## 结论

OpenSSL 在 OpenHarmony 中**不存在 API 接口层面的差异**。适配工作完全通过配置完成，保持了与上游的完全兼容性。依赖模块可以按照标准 OpenSSL 开发文档进行开发，使用任何公开 API。

如需了解配置层面的适配差异，请参阅：
- 02_Patches.md — 适配修改分析
- 03_Build_Integration.md — 构建配置说明
- 06_Security.md — 安全配置影响

---

*文档版本：1.0*  
*创建日期：2026-02-08*