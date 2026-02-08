# OpenSSL 原始库简介

## 项目背景

OpenSSL 是一个历史悠久且应用广泛的开源密码学工具库，其发展历程可追溯至 1995 年 SSLeay 库的开发。该项目由 Eric A. Young 和 Tim J. Hudson 最初创建，后由全球开源社区共同维护和发展。OpenSSL 名称中的「SSL」虽然源自早期的安全套接字层协议，但该项目早已扩展支持更新的传输层安全（TLS）协议标准，成为互联网安全通信的基础设施之一。

OpenSSL 的重要性在现代计算环境中无可替代。根据公开统计数据，全球约有超过三分之二的 Web 服务器使用 OpenSSL 或其衍生版本提供 HTTPS 服务。操作系统、网络设备、嵌入式系统以及各类应用程序都依赖 OpenSSL 实现加密通信、身份认证和数据保护功能。这种广泛的应用基础使得 OpenSSL 的任何安全漏洞都可能影响大量系统，历史上 Heartbleed、POODLE、BEAST 等漏洞都曾引发全球性的安全响应。

## 主要组件

### libcrypto — 通用密码学库

libcrypto 是 OpenSSL 的核心密码学组件，提供了丰富的加密算法实现。该库支持多种对称加密算法，包括 AES（支持 ECB、CBC、CFB、OFB、GCM、CCM 等模式）、DES 和 3DES（由于安全原因已不推荐使用）、Camellia、ChaCha20 以及 SM4（中国国家密码算法标准）。在哈希算法方面，libcrypto 实现了 MD5、SHA-1、SHA-2 系列（SHA-224/256/384/512）以及 SHA-3 标准算法，还包括 SM3 国产哈希算法。

非对称密码学方面，libcrypto 提供了 RSA、DSA、ECDSA、EdDSA、DH、ECDH 等密钥交换和签名算法的完整实现。特别值得注意的是对椭圆曲线密码学（ECC）的广泛支持，包括 NIST 曲线（P-192/P-224/P-256/P-384/P-521）、Curve25519、Curve448 以及中国国家标准的 SM2 曲线。这些算法为现代安全通信提供了高效的公钥密码学基础。

此外，libcrypto 还包含密钥派生函数（KDF）实现（如 PBKDF2、HKDF、Scrypt）、消息认证码（HMAC、CMAC、Poly1305）、随机数生成、X.509 证书处理、PKCS#7/12/8 封装格式支持等众多功能。这些功能共同构成了安全应用开发的密码学原语基础。

### libssl — TLS/SSL 协议实现

libssl 库实现了完整的 TLS/SSL 协议栈，支持从 SSLv2 到 TLSv1.3 的所有主流协议版本。其中 TLSv1.3 是最新版本，引入了 0-RTT 恢复、前向保密增强、简化握手流程等重要改进。libssl 提供了客户端（SSL_CTX_new、SSL_new 等 API）和服务器端（SSL_accept 等 API）的完整实现，支持证书认证和预共享密钥（PSK）等多种认证方式。

协议实现方面，libssl 管理 TLS 记录层封包、握手协议协商、密钥材料派生、加密套件选择和警报处理等复杂流程。该库支持超过 30 种加密套件，涵盖各种安全级别和性能特性的组合。开发者可以根据应用需求选择合适的套件，平衡安全性与兼容性。

### openssl — 命令行工具

除了库文件，OpenSSL 还提供了一个同名的命令行工具，是系统管理和安全运维的常用工具。该工具支持生成 RSA/ECDH 密钥对、创建 X.509 证书签名请求和自签名证书、计算文件摘要、进行对称加密解密操作、测试 SSL/TLS 连接、查看证书信息等众多任务。运维人员经常使用 openssl s_client、openssl x509、openssl req 等子命令进行故障诊断和安全配置验证。

---

## OpenSSL 3.0 新特性

### Provider 架构

OpenSSL 3.0 引入的最重要架构变化是 Provider 机制，将加密功能模块化为可替换的组件。默认情况下，系统加载 default provider 提供标准加密算法，legacy provider 提供遗留算法。当应用需要使用特定算法或实现自定义加密逻辑时，可以实现自定义 provider 并动态加载。

这种架构设计提高了 OpenSSL 的灵活性，使得算法实现可以独立于核心库进行更新和替换。Provider 还支持运行时加载和卸载，为资源受限的嵌入式环境提供了按需加载的可能性。然而，在 OpenHarmony 的适配中，动态 provider 加载功能被禁用，所有 provider 都静态编译到库中。

### FIPS 模块

OpenSSL 3.0 集成了符合 FIPS 140-2/140-3 标准的加密模块（FIPS Provider），为需要合规认证的应用提供经过验证的密码学实现。该模块对算法实现进行了额外的安全审查和测试，使用受到更严格的质量控制。启用 FIPS 模块后，系统只能使用经过认证的算法，安全性得到保障但灵活性有所降低。

OpenHarmony 适配未显式启用 FIPS provider，这可能出于以下考虑：FIPS 认证增加了维护复杂度、认证状态与具体软硬件配置相关、部分 FIPS 限制可能影响与旧系统的兼容性。需要在合规环境中部署的设备应当评估是否需要启用 FIPS 模块。

### 错误处理改进

OpenSSL 3.0 重构了错误处理机制，引入了 ERR_RETAIL_BLOB 等宏用于在生产环境中输出更详细的诊断信息。与旧版本相比，新版本提供了更丰富的错误上下文信息和更好的可追溯性，有助于开发者定位和解决集成问题。库的内部错误代码体系也进行了整理，错误分类更加清晰。

---

## 在 OpenHarmony 中的定位

### 安全基础设施角色

OpenSSL 在 OpenHarmony 系统中承担着安全基础设施的核心职责。从网络通信角度看，它是 HTTPS 连接、DTLS 通信、MQTT over TLS 等安全通道的底层依赖。foundation/communication 子系统中的网络模块、wpa_supplicant 中的 Wi-Fi 安全认证、grpc 的 TLS 传输都直接依赖 libssl 的功能。

从数据保护角度看，sqlite 的透明加密、文件系统完整性校验（fsverity）、安全存储服务等都需要 libcrypto 提供密码学原语。这些功能保护了用户数据和系统资源的机密性与完整性，是设备安全能力的组成部分。

### 组件接口定义

OpenHarmony 通过 inner_kits 机制定义了 OpenSSL 的对外接口。libcrypto 套件提供加密库（libcrypto_shared 和 libcrypto_static 两种形态），libssl 套件提供 SSL/TLS 库（同样支持静态和动态两种链接方式）。所有头文件从 //third_party/openssl/include 目录导出，运行时配置文件 openssl.cnf 安装到 /system/etc 目录。

这种接口设计使得依赖模块无需关心 OpenSSL 的内部实现细节，只需通过标准头文件调用相关 API。组件版本的独立管理也意味着 OpenSSL 的升级不会直接影响依赖模块的构建（只要 API 兼容）。

---

## 版本信息

| 项目 | 值 |
|------|-----|
| 上游版本 | 3.0.9 |
| OH 组件版本 | 4.0 |
| 许可证 | Apache License 2.0 |
| 上游地址 | https://www.openssl.org/source/openssl-3.0.9.tar.gz |
| 维护责任人 | wanghaixiang@huawei.com |

---

## 相关文档

本文档简要介绍了 OpenSSL 库的基本功能特性。如需了解 OpenHarmony 的适配详情，请参阅：02_Patches.md（适配修改分析）、03_Build_Integration.md（构建配置说明）、06_Security.md（安全配置分析）。

---

*文档版本：1.0*  
*创建日期：2026-02-08*  
*参考资料：上游 OpenSSL 官方文档、CHANGES.md*