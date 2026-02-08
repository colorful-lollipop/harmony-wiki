# OpenHarmony 适配修改分析

## 适配策略概述

**重要说明**：OpenHarmony 对 OpenSSL 3.0.9 的适配**不采用传统的 Patch 文件修改方式**，而是使用**构建系统集成策略**。这一策略的核心特点是：对上游源代码零侵入性修改，所有适配通过 BUILD.gn 配置文件和运行时配置文件完成。这种设计使得上游版本升级更加平滑，只需验证配置兼容性而无需处理 Patch 冲突和合并问题。

本章节将详细分析 OpenHarmony 对 OpenSSL 的所有适配修改，包括构建系统配置、平台检测逻辑、运行时行为调整等各个方面。与传统 Patch 文档不同，这里使用「适配修改」而非「补丁」来描述这些变更，因为它们是配置层面的集成而非源代码的差异修改。

---

## 适配修改清单

### 适配分类总览

| 分类 | 文件/配置 | 修改类型 | 影响范围 |
|------|----------|----------|----------|
| 构建系统 | BUILD.gn | 新增（完整重写） | 全局 |
| 运行时配置 | open_harmony_openssl_config/openssl.cnf | 新增 | 运行时行为 |
| 平台头文件 | os-dep/haiku.h | 新增 | 编译兼容性 |
| 组件定义 | bundle.json | 新增 | 组件系统集成 |
| 许可证合规 | OAT.xml | 新增 | 审计追踪 |
| 构建脚本 | make_openssl_build_all_generated.sh | 新增 | 代码生成 |
| 构建脚本 | run_command.py | 新增 | 构建辅助 |

---

## 详细适配分析

### 适配一：构建系统完全重写（BUILD.gn）

**修改文件**：third_party/openssl/BUILD.gn（71,637 字节）

**原始问题**：上游 OpenSSL 使用 Configure + Make/CMake 构建系统，而 OpenHarmony 使用 GN 构建工具。两种构建系统的语法、依赖表达、编译流程完全不同，需要重新编写构建配置。

**修改内容**：BUILD.gn 文件完全重写了 OpenSSL 的构建逻辑，主要包含以下组成部分：

1. **平台检测逻辑**（约 50 行）：根据当前 CPU 架构和操作系统选择对应的 OpenSSL 平台配置。检测条件包括 current_cpu、current_os、host_os、is_mingw、ohos_kernel_type 等变量。检测结果存储在 openssl_selected_platform 变量中，供后续配置引用。

```gn
if (current_cpu == "arm" && !(current_os == "linux" || host_os == "mac")) {
  openssl_selected_platform = "linux-armv4"
} else if (current_cpu == "arm64" &&
           (!(current_os == "linux" || host_os == "mac") ||
            current_os == "ohos" ||
            (current_os == "linux" || host_os == "linux"))) {
  openssl_selected_platform = "linux-aarch64"
}
// ... 更多平台检测
```

2. **汇编优化配置**（约 400 行）：为每种目标平台定义对应的汇编源文件列表。这些汇编文件提供了密码学运算的 CPU 指令级优化，对性能至关重要。配置格式为 `${build_all_generated_path}/<模块>/<算法>-<平台>.<扩展名>`，指向构建时生成的文件。

```gn
libcrypto_build_all_generated_linux_aarch64_sources = [
  "${build_all_generated_path}/crypto/aes/aesv8-armx.S",
  "${build_all_generated_path}/crypto/arm64cpuid.S",
  "${build_all_generated_path}/crypto/bn/armv8-mont.S",
  // ... 更多文件
]
```

3. **编译选项配置**（约 300 行）：定义各平台的 C 编译器和汇编器选项。包括警告抑制（-Wno-error=*）、优化标志（-O3）、平台宏定义（-DL_ENDIAN）、算法启用宏（-DAES_ASM）等。这些选项确保代码在 OpenHarmony 工具链下成功编译。

```gn
crypto_config_linux_aarch64_cflags = [
  "-DOPENSSL_USE_NODELETE",
  "-fPIC",
  "-pthread",
  "-DECP_NISTZ256_ASM",
  "-DKECCAK1600_ASM",
  // ... 更多选项
]
```

4. **构建目标定义**（约 1000 行）：定义所有产出目标，包括 libcrypto_static、libcrypto_shared、libssl_static、libssl_shared、libapps、openssl 可执行文件等。每个目标指定源文件列表、依赖关系、配置选项等属性。

5. **平台特殊处理**：针对 liteos_a 内核的特殊适配逻辑，在该平台上禁用 liblegacy provider 和 OPENSSL_cpuid_setup 功能。

**OH 价值**：通过这一适配，OpenSSL 完全集成到 OpenHarmony 构建体系，能够与其他模块统一构建管理，共享构建缓存，并正确处理跨平台编译需求。

**升级影响**：这是最关键的适配部分。升级上游版本时，需要同步更新 BUILD.gn 中的源文件列表、汇编文件配置和编译选项。文件中的 29 个「升级修改适配检查点」标记了需要验证的位置。

---

### 适配二：运行时配置定制（openssl.cnf）

**修改文件**：open_harmony_openssl_config/openssl.cnf（1,061 字节）

**原始问题**：上游 OpenSSL 的默认配置文件面向通用 Linux/Unix 系统设计，路径、加密套件、安全策略等配置不适用于 OpenHarmony 嵌入式环境。

**修改内容**：创建 OH 特权的运行时配置文件，主要修改点：

```ini
openssl_conf = openssl_init

[openssl_init]
providers = provider_sect
ssl_conf = ssl_conf_sect

[provider_sect]
default = default_sect
legacy = legacy_sect        # 启用 legacy provider

[default_sect]
activate = 1                # 默认 provider 启用

[legacy_sect]
activate = 1                 # Legacy provider 也启用

[ssl_conf_system_default_sect]
Options = UnsafeLegacyRenegotiation  # 允许不安全重协商
CipherString = DEFAULT:@SECLEVEL=0   # 最低安全级别
MinProtocol = None        # 无最低 TLS 版本限制
```

**关键配置变更详解**：

1. **默认和 Legacy Provider 同时启用**：上游默认只启用 default provider，legacy provider 需要显式配置。OH 配置文件同时启用两者，是为了兼容需要使用 DES、3DES、RC4 等遗留算法的旧系统。这一配置增加了安全风险，应当在可能的情况下禁用 legacy provider。

2. **UnsafeLegacyRenegotiation 选项**：TLS 协议的 Renegotiation 功能曾存在安全漏洞（ClientHello 注入攻击），后续通过 RFC 5746 修复。某些旧系统可能不支持修复后的安全重协商，此选项允许与这些系统兼容，但可能受到降级攻击。

3. **SECLEVEL=0**：OpenSSL 的安全级别控制允许使用的加密算法强度。级别 0 是最低级别，允许使用 40 位和 56 位加密、以及导出级算法。级别 1 及以上要求至少 80 位加密强度。设置 SECLEVEL=0 主要为了兼容旧系统，生产环境应当提高此设置。

4. **MinProtocol=None**：此配置不强制最低 TLS 版本，允许使用 SSLv3、TLSv1.0、TLSv1.1 等已知的弱协议版本。生产环境应当设置为 TLSv1.2 或更高。

**OH 价值**：这些配置确保 OpenSSL 能够与 OpenHarmony 生态中的各种旧系统和设备通信，同时提供灵活的安全策略调整空间。

**安全建议**：设备制造商应当评估实际安全需求，定制 openssl.cnf 配置。对于面向消费者的设备，建议设置 SECLEVEL=1 或更高、禁用 legacy provider、设置 MinProtocol=TLSv1.2、移除 UnsafeLegacyRenegotiation 选项。

---

### 适配三：平台头文件（haiku.h）

**修改文件**：os-dep/haiku.h（378 字节）

**原始问题**：Haiku 操作系统（一个开源的 BeOS 兼容系统）的头文件包含在 OpenSSL 源码树中，可能与 OpenHarmony 的某些配置产生冲突或被错误引用。

**修改内容**：创建空的或最小化的 haiku.h 头文件，避免编译错误。

```c
// os-dep/haiku.h - OpenHarmony 适配
#ifndef OS_DEPS_HAIKU_H
#define OS_DEPS_HAIKU_H
// Haiku OS specific declarations for OpenHarmony compatibility
// This file is intentionally minimal to avoid conflicts
#include <sys/select.h>
#include <sys/time.h>
#endif /* OS_DEPS_HAIKU_H */
```

**OH 价值**：防止编译时因头文件冲突导致的错误，确保 OpenSSL 在 OpenHarmony 构建环境中的编译正确性。

---

### 适配四：组件元数据（bundle.json）

**修改文件**：bundle.json（2,297 字节）

**原始问题**：OpenHarmony 组件系统需要特定的元数据格式来描述第三方库，包括组件名称、版本、依赖关系、产出接口等信息。

**修改内容**：创建符合 OH 组件规范的 bundle.json 定义：

```json
{
  "name": "@ohos/openssl",
  "description": "OpenSSL is a robust, commercial-grade, full-featured Open Source Toolkit for the Transport Layer Security (TLS) protocol formerly known as the Secure Sockets Layer (SSL) protocol.",
  "version": "4.0",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/openssl"
  },
  "component": {
    "name": "openssl",
    "subsystem": "thirdparty",
    "features": [
      "openssl_enabled",
      "openssl_feature_support_usr_symlink"
    ],
    "adapted_system_type": [
      "standard"
    ],
    "build": {
      "sub_component": [
        "//third_party/openssl:openssl"
      ],
      "inner_kits": [
        {
          "name": "//third_party/openssl:libcrypto_shared",
          "header": {
            "header_base": "//third_party/openssl/include",
            "header_files": []
          }
        },
        // ... 更多 inner_kits 定义
      ]
    }
  }
}
```

**OH 价值**：将 OpenSSL 正确注册到 OpenHarmony 组件系统，使得其他模块能够通过标准依赖机制引用它，并正确管理产出物的安装和分发。

---

### 适配五：LiteOS-A 特殊处理

**修改位置**：BUILD.gn 第 21-38 行

**原始问题**：LiteOS-A 是 OpenHarmony 的轻量级内核，缺少标准 Linux 系统的部分头文件和功能。尝试在该平台上编译 OpenSSL 的 liblegacy 模块时会失败，具体错误为 engines/e_afalg.c 找不到 linux/version.h 头文件。

**修改内容**：在 BUILD.gn 中添加条件判断，当 ohos_kernel_type 为 liteos_a 时禁用 liblegacy：

```gn
if (defined(ohos_kernel_type) && ohos_kernel_type == "liteos_a") {
  print("liteos_a doesn't support OPENSSL_cpuid_setup and liblegacy, disable them.")
  build_with_liblegacy = false
} else {
  build_with_liblegacy = true
}
```

禁用功能的影响：
- OPENSSL_cpuid_setup：自动检测 CPU 特性并选择最优算法实现
- liblegacy provider：DES、3DES、RC4、Blowfish、IDEA、RC2 等遗留算法
- e_afalg 引擎：Linux AF_ALG 加密接口引擎

**OH 价值**：确保 OpenSSL 能够在 LiteOS-A 内核上成功编译，同时明确标识了功能限制，便于开发者理解。

---

## 适配修改汇总表

| 适配项 | 修改类型 | 重要性 | 升级风险 | 安全影响 |
|--------|----------|--------|----------|----------|
| BUILD.gn 重写 | 新增配置 | 高 | 高（29 检查点） | 无直接影响 |
| openssl.cnf | 新增配置 | 高 | 低 | 高（SECLEVEL=0） |
| bundle.json | 新增配置 | 中 | 低 | 无 |
| haiku.h | 新增文件 | 低 | 极低 | 无 |
| LiteOS-A 适配 | 条件禁用 | 中 | 中 | 无 |

---

## 上游合并建议

以下适配修改具有向上游贡献的潜力：

1. **haiku.h 清理**：当前内容可以合并到上游，或与上游确认是否需要此文件
2. **平台检测逻辑**：BUILD.gn 中的平台检测模式可以参考，但 GN 语法与上游构建系统不同
3. **安全配置**：openssl.cnf 的配置是 OH 特权需求，不太适合上游接受

建议仅在安全修复或关键 bugfix 场景考虑向上游提交变更。

---

## 升级检查清单

升级 OpenSSL 上游版本时，按以下顺序检查适配修改：

- [ ] 验证源文件列表完整性
- [ ] 验证汇编文件路径正确性
- [ ] 验证编译选项兼容性
- [ ] 验证 LiteOS-A 编译是否成功
- [ ] 验证 openssl.cnf 配置语法正确性
- [ ] 验证所有 inner_kits 路径正确性
- [ ] 在所有支持平台上执行完整编译测试

---

*文档版本：1.0*  
*创建日期：2026-02-08*  
*注意：由于采用构建系统集成策略，本文档分析的是 OH 适配配置而非传统 Patch 文件*