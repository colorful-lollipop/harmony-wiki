# OpenSSL 在 OpenHarmony 中的适配

## 库概述

OpenSSL 是 OpenHarmony 第三方库中最重要的安全组件之一，为整个操作系统提供传输层安全（TLS）协议实现和通用密码学服务。本文档记录了 OpenSSL 3.0.9 版本在 OpenHarmony 系统中的集成方式、适配修改和使用情况。

**原始库信息**：OpenSSL 3.0.9，Apache License 2.0许可证，上游地址为 https://www.openssl.org/source/openssl-3.0.9.tar.gz。该库由 wanghaixiang@huawei.com 负责在 OpenHarmony 中的维护工作。OpenSSL 在 OpenHarmony 组件系统中的版本标识为 4.0，这一版本号与上游版本号独立，反映了 OpenHarmony 组件版本管理体系的特殊性。

**适配策略说明**：与 OpenHarmony 大多数第三方库采用 Patch 文件修改源代码的方式不同，OpenSSL 的适配主要通过重写构建系统（BUILD.gn）和添加运行时配置文件来实现。这种策略对上游代码零侵入，版本升级时更容易合并上游变更，但需要在每次升级时验证 BUILD.gn 配置与新版本的兼容性。开发者应当注意，适配文档中标记的 29 个「升级修改适配检查点」是每次版本升级时必须逐一验证的关键位置。

---

## OpenHarmony 适配特性

### 平台支持

OpenSSL 在 OpenHarmony 中支持多种硬件架构和操作系统环境。标准系统支持 ARM 32位（linux-armv4）、ARM 64位（linux-aarch64）、x86_64（linux-x86_64）以及龙芯 64位（linux64-loongarch64）。开发环境方面支持 macOS 的 x86_64 和 ARM64 架构，以及 Windows 的 MinGW 64位交叉编译环境。每种平台都有对应的汇编优化配置文件，充分利用各 CPU 架构的特定指令集以获得最佳密码学运算性能。

轻量级系统方面，LiteOS-A 内核的支持需要特殊适配处理。由于 LiteOS-A 缺少标准 Linux 的部分头文件（如 linux/version.h），OpenSSL 在该平台上禁用了 liblegacy provider 和 OPENSSL_cpuid_setup 功能。这一限制意味着基于 LiteOS-A 的设备无法使用 OpenSSL 的 legacy 加密算法套件和 CPU 特性自动检测功能，开发者在选择加密方案时需要特别注意。

### 构建系统适配

OpenHarmony 使用 GN（Generate Ninja）构建系统，而上游 OpenSSL 使用传统的 Configure + Make 方式。为解决这一差异，BUILD.gn 文件完全重写了构建配置，通过 make_openssl_build_all_generated.sh 脚本调用上游 Configure 生成平台特定的汇编代码和配置头文件，然后由 GN 工具完成实际编译。这种混合架构既复用了上游的架构检测和优化生成逻辑，又完全集成到 OpenHarmony 的构建体系中。

库形态方面，OpenHarmony 主要提供动态链接库形态（libcrypto.z.so 和 libssl.z.so），同时保留静态链接库以满足特定场景需求。与上游不同的是，OpenSSL 的引擎（Engines）和模块（Modules）功能被禁用，不支持动态加载加密引擎，所有功能均内置于库中。这一设计决策有利于减小攻击面并简化部署，但牺牲了运行时加载扩展能力的灵活性。

### 安全配置

**重要安全提示**：OpenSSL 的运行时配置文件（openssl.cnf）包含了几项可能影响系统安全水位的配置。安全级别（SECLEVEL）被设置为 0，这是最低级别，允许使用已知存在安全问题的弱加密算法。TLS 最低协议版本限制（MinProtocol）被设置为 None，不强制要求最低 TLS 版本。UnsafeLegacyRenegotiation 选项被启用，允许不安全的 TLS 重协商操作。

这些配置的启用主要是为了兼容旧系统和使用 legacy 加密算法的场景。部署团队应当评估实际安全需求，必要时通过定制配置文件提升安全水位。默认配置位于 /system/etc/openssl.cnf，系统集成商可以根据设备安全策略进行修改。值得注意的是，legacy provider 的启用意味着 DES、3DES、RC4 等已知不安全的算法仍然可用，这些算法的使用应当受到严格限制或仅在绝对必要时才被允许。

---

## 依赖关系

### 直接依赖模块

OpenSSL 在 OpenHarmony 第三方库生态中扮演着基础设施角色，被以下十个模块直接依赖：

**网络通信模块**：curl 库使用 OpenSSL 实现 HTTPS 功能，是系统中最主要的 HTTP/HTTPS 客户端。grpc 框架依赖 OpenSSL 提供 TLS 加密的 RPC 通信能力。libwebsockets 库通过 OpenSSL 支持安全的 WebSocket 连接。wpa_supplicant 是 Wi-Fi 认证的核心组件，使用 OpenSSL 处理 EAP-TLS 证书认证。这些模块构成了 OpenHarmony 网络安全通信的基础设施层，任何对 OpenSSL 的变更都可能影响这些模块的功能和安全性。

**数据存储模块**：sqlite 数据库通过 OpenSSL 实现透明数据加密（TE），使用 libcrypto 提供的加密算法保护数据库内容。fsverity-utils 工具使用 OpenSSL 计算文件哈希，用于验证系统文件的完整性和真实性。这两个模块确保了数据在存储状态下的机密性和完整性保护。

**系统服务模块**：cups 打印系统使用 OpenSSL 保护 IPP 打印协议的通信安全。toybox 命令行工具在启用 SELinux 支持时依赖 OpenSSL。cangjie_runtime 输入法运行时仅引用 OpenSSL 头文件，用于可能的加密相关功能。rust-openssl 项目提供了 Rust 语言对 OpenSSL 的 FFI 绑定，是 Rust 生态接入 OpenSSL 的桥梁。

### 依赖关系图

```
                    ┌─────────────────────────────────────┐
                    │         OpenSSL (third_party)        │
                    │   libcrypto + libssl (3.0.9)        │
                    └─────────────────────────────────────┘
                               │
        ┌──────────┬───────────┼───────────┬──────────┐
        ▼          ▼           ▼           ▼          ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌────────┐ ┌──────────┐
   │  curl   │ │  grpc   │ │  wpa_   │ │ sqlite │ │ rust-    │
   │ HTTP库  │ │ RPC框架 │ │supplicant│ │ 加密DB  │ │openssl  │
   └─────────┘ └─────────┘ └─────────┘ └────────┘ └──────────┘
        │          │           │           │           │
        ▼          ▼           ▼           ▼           ▼
   HTTPS/TLS   TLS/RPC    Wi-Fi认证   数据库加密   Rust绑定
```

---

## 文档导航

本文档集包含以下内容，按阅读优先级排列：

**快速入门**（必读）：README.md（本文档）提供整体概览和导航；SUMMARY.md 包含详细的阅读路线建议和文档摘要。

**核心内容**（推荐）：01_Overview.md 介绍原始 OpenSSL 库的功能特性；02_Patches.md 详细记录 OpenHarmony 的适配修改（由于采用构建集成策略，实际为适配分析）；03_Build_Integration.md 说明构建系统配置和编译选项；04_Usage_in_OH.md 描述各模块的具体使用方式；06_Security.md 分析安全配置的影响和建议。

**参考信息**（按需）：05_API_Differences.md 记录 OpenHarmony 新增或修改的 API 接口；_work/ASSESSMENT.md 包含项目评估的原始记录；_work/NOTES.md 记录分析过程中的发现和待确认事项。

---

## 快速参考

### 关键文件路径

| 路径 | 用途 |
|------|------|
| third_party/openssl/BUILD.gn | OpenHarmony 构建配置 |
| third_party/openssl/bundle.json | 组件元数据定义 |
| third_party/openssl/open_harmony_openssl_config/openssl.cnf | 运行时配置 |
| third_party/openssl/README.OpenSource | 开源归属信息 |

### 编译产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libcrypto.z.so | //third_party/openssl:libcrypto_shared | 动态链接库 |
| libssl.z.so | //third_party/openssl:libssl_shared | 动态链接库 |
| libcrypto.a | //third_party/openssl:libcrypto_static | 静态链接库 |
| libssl.a | //third_party/openssl:libssl_static | 静态链接库 |
| openssl.cnf | //third_party/openssl:openssl.cnf | 运行时配置 |

### 安全相关宏定义

| 宏定义 | 值 | 影响 |
|--------|-----|------|
| OPENSSLDIR | /system/etc | 配置文件的系统路径 |
| ENGINESDIR | "" | 禁用动态引擎加载 |
| MODULESDIR | "" | 禁用动态模块加载 |
| STATIC_LEGACY | 条件启用 | Legacy provider 内置 |

---

## 维护指南

### 版本升级注意事项

升级 OpenSSL 上游版本时，必须检查 BUILD.gn 中的 29 个「升级修改适配检查点」。主要验证内容包括：平台汇编优化文件的完整性、源文件列表的变更、编译选项的兼容性以及新增或移除的依赖项。建议在所有支持的平台上进行完整编译测试，特别关注 liteos_a 平台是否成功编译。

### 常见问题

**编译失败排查**：首先检查目标平台是否在 BUILD.gn 的平台检测逻辑中得到正确识别；其次验证汇编优化文件是否完整生成；最后确认编译警告抑制选项是否仍然有效。如果遇到特定平台的编译错误，应当检查对应平台的配置部分是否缺少必要的源文件或包含不存在的文件。

**安全配置调整**：如果设备需要更高的安全水位，可以修改 openssl.cnf 中的 SECLEVEL 设置（建议至少设为 2），移除 UnsafeLegacyRenegotiation 选项，并考虑禁用 legacy provider。但这些修改可能导致与旧系统的兼容性问题，需要在安全性与兼容性之间进行权衡。

---

## 相关资源

- **上游文档**：https://www.openssl.org/docs/man3.0/
- **OpenSSL 3.0 迁移指南**：https://www.openssl.org/docs/man3.0/man7/migration_guide.html
- **OpenHarmony 构建系统**：参见 GN 构建文档
- **安全问题报告**：通过 OpenHarmony 安全渠道报告

---

*文档版本：1.0*  
*创建日期：2026-02-08*  
*维护责任人：wanghaixiang@huawei.com*