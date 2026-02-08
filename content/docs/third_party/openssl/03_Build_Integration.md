# OpenHarmony 构建适配

## 构建系统概述

OpenSSL 在 OpenHarmony 中采用 GN（Generate Ninja）构建系统进行构建管理，与上游 OpenSSL 使用的 Configure + Make 方式存在本质差异。OpenHarmony 的适配策略是保留上游的 Configure 脚本用于生成平台特定代码，通过 BUILD.gn 文件定义完整的 GN 构建规则，最终由 Ninja 执行实际编译。这种混合架构既复用了上游的架构检测和优化生成逻辑，又完全集成到 OpenHarmony 的构建体系中。

构建产物主要包括动态链接库（libcrypto.z.so、libssl.z.so）、静态链接库（libcrypto.a、libssl.a）以及命令行工具（openssl）。所有产出物的安装路径遵循 OpenHarmony 组件系统的规范，库文件安装到系统库目录，配置文件安装到 /system/etc/openssl.cnf，头文件安装到指定的 include 目录。

---

## BUILD.gn 结构详解

### 文件组织

BUILD.gn 文件按功能分为以下主要部分：

| 行号范围 | 功能模块 | 代码行数 |
|----------|----------|----------|
| 1-50 | 声明参数和平台检测 | 约 50 行 |
| 96-270 | 汇编优化源文件配置 | 约 175 行 |
| 296-365 | 公共配置和编译选项 | 约 70 行 |
| 416-450 | 公共 CFLAGS 配置 | 约 35 行 |
| 450-625 | 平台特定 CFLAGS | 约 175 行 |
| 830-1340 | 源文件列表定义 | 约 510 行 |
| 1340-1470 | 构建目标定义 | 约 130 行 |

这种模块化的文件结构便于维护和升级，每个功能区域都有清晰的边界定义。

### 平台检测逻辑

平台检测是 BUILD.gn 的核心逻辑之一，负责根据目标设备确定正确的 OpenSSL 平台配置：

```gn
openssl_selected_platform = ""
if (current_cpu == "arm" && !(current_os == "linux" || host_os == "mac")) {
  openssl_selected_platform = "linux-armv4"
} else if (current_cpu == "arm64" &&
           (!(current_os == "linux" || host_os == "mac") ||
            current_os == "ohos" ||
            (current_os == "linux" || host_os == "linux"))) {
  openssl_selected_platform = "linux-aarch64"
} else if ((current_cpu == "x64" || current_cpu == "x86_64") &&
           current_os != "mingw") {
  openssl_selected_platform = "linux-x86_64"
} else if (is_mingw) {
  openssl_selected_platform = "mingw64"
} else if (current_cpu == "loongarch64" && current_os == "ohos") {
  openssl_selected_platform = "linux64-loongarch64"
}
```

检测流程首先检查 CPU 架构（current_cpu），然后检查操作系统类型（current_os、host_os、is_mingw）。对于 ARM64 架构，额外检查 ohos 值以支持 OHOS 操作系统。对于 LoongArch64 架构，OHOS 变体有专门的平台名称定义。

**支持的平台映射**：

| openssl_selected_platform | 目标设备 |
|---------------------------|----------|
| linux-armv4 | ARM 32位设备 |
| linux-aarch64 | ARM 64位标准设备、OHOS 设备 |
| linux-x86_64 | x86_64 Linux 设备 |
| linux64-loongarch64 | 龙芯 64位 OHOS 设备 |
| darwin64-x86_64-cc | macOS x86_64 开发机 |
| darwin64-arm64-cc | macOS ARM64 开发机 |
| mingw64 | Windows 交叉编译 |

---

## 编译选项配置

### 公共编译选项

以下编译选项应用于所有平台：

```gn
crypto_config_common_cflags = [
  "-Wa,--noexecstack",        # 汇编器：禁用可执行堆栈
  "-DNDEBUG",                  # 禁用调试断言
  "-DOPENSSL_BUILDING_OPENSSL", # 标记为 OpenSSL 自身构建
  "-DOPENSSL_CPUID_OBJ",      # 启用 CPUID 检测代码
  "-DOPENSSL_PIC",            # 生成位置无关代码
  "-DENGINESDIR=\"\"",         # 引擎路径置空（禁用动态加载）
  "-DMODULESDIR=\"\"",        # 模块路径置空
  "-DOPENSSLDIR=\"/system/etc\"", # OpenSSL 配置路径
]
```

关键配置说明：
- ENGINESDIR 和 MODULESDIR 置空禁用了 OpenSSL 的动态引擎和模块加载功能，减小了攻击面
- OPENSSLDIR 设置为 /system/etc 确保配置文件位于系统目录
- OPENSSL_BUILDING_OPENSSL 宏用于区分库内部构建和外部调用

### 平台特定编译选项

各平台的编译选项主要差异在于算法相关的宏定义：

**ARM64 平台（linux-aarch64）**：

```gn
crypto_config_linux_aarch64_cflags = [
  "-DOPENSSL_USE_NODELETE",
  "-fPIC",
  "-pthread",
  "-DECP_NISTZ256_ASM",      # NIST P-256 曲线优化
  "-DKECCAK1600_ASM",        # SHA-3 算法优化
  "-DOPENSSL_BN_ASM_MONT",   # Montgomery 乘法优化
  "-DPOLY1305_ASM",          # Poly1305 MAC 优化
  "-DMD5_ASM",
  "-DSHA1_ASM",
  "-DSHA256_ASM",
  "-DSHA512_ASM",
  "-DVPAES_ASM",             # VAES 指令优化
]
```

**x86_64 平台（linux-x86_64）**：

```gn
crypto_config_linux_x86_64_cflags = [
  "-DL_ENDIAN",              # 小端字节序
  "-DOPENSSL_IA32_SSE2",     # IA-32 SSE2 指令
  "-DAES_ASM", "-DBSAES_ASM",       # AES-NI 指令
  "-DCMLL_ASM",              # Camellia 优化
  "-DGHASH_ASM",             # GHASH (GCM) 优化
  "-DMD5_ASM", "-DWHIRLPOOL_ASM",
  "-DX25519_ASM",           # X25519 椭圆曲线优化
  "-DOPENSSL_BN_ASM_GF2m", "-DOPENSSL_BN_ASM_MONT", "-DOPENSSL_BN_ASM_MONT5",
  "-DSHA1_ASM", "-DSHA256_ASM", "-DSHA512_ASM",
]
```

### 警告抑制选项

为确保编译成功，BUILD.gn 配置了针对 OpenHarmony 工具链的警告抑制：

```gn
openssl_internal_cflags = [
  "-Wno-error=int-conversion",     # 整数指针转换
  "-Wno-error=constant-conversion", # 常量转换溢出
  "-Wno-error=shift-count-overflow", # 移位计数溢出
  "-Wno-error=macro-redefined",      # 宏重定义
  "-Wno-error=implicit-fallthrough", # switch 贯穿
  "-Wno-error=sign-compare",        # 符号比较
]
```

这些抑制选项仅在非兼容旧版构建系统时启用（!compatible_with_legacy_build_system），反映了对现代编译器严格检查的务实处理。

---

## 汇编优化配置

### ARMv4 平台汇编源文件

```gn
libcrypto_build_all_generated_linux_armv4_sources = [
  "${build_all_generated_path}/crypto/aes/aes-armv4.S",
  "${build_all_generated_path}/crypto/aes/aesv8-armx.S",
  "${build_all_generated_path}/crypto/aes/bsaes-armv7.S",
  "${build_all_generated_path}/crypto/armv4cpuid.S",
  "${build_all_generated_path}/crypto/bn/armv4-gf2m.S",
  "${build_all_generated_path}/crypto/bn/armv4-mont.S",
  "${build_all_generated_path}/crypto/chacha/chacha-armv4.S",
  "${build_all_generated_path}/crypto/ec/ecp_nistz256-armv4.S",
  "${build_all_generated_path}/crypto/modes/ghash-armv4.S",
  "${build_all_generated_path}/crypto/modes/ghashv8-armx.S",
  "${build_all_generated_path}/crypto/poly1305/poly1305-armv4.S",
  "${build_all_generated_path}/crypto/sha/keccak1600-armv4.S",
  "${build_all_generated_path}/crypto/sha/sha1-armv4-large.S",
  "${build_all_generated_path}/crypto/sha/sha256-armv4.S",
  "${build_all_generated_path}/crypto/sha/sha512-armv4.S",
]
```

### ARM64 平台汇编源文件

```gn
libcrypto_build_all_generated_linux_aarch64_sources = [
  "${build_all_generated_path}/crypto/aes/aesv8-armx.S",
  "${build_all_generated_path}/crypto/aes/vpaes-armv8.S",
  "${build_all_generated_path}/crypto/arm64cpuid.S",
  "${build_all_generated_path}/crypto/bn/armv8-mont.S",
  "${build_all_generated_path}/crypto/chacha/chacha-armv8.S",
  "${build_all_generated_path}/crypto/ec/ecp_nistz256-armv8.S",
  "${build_all_generated_path}/crypto/md5/md5-aarch64.S",
  "${build_all_generated_path}/crypto/modes/aes-gcm-armv8_64.S",
  "${build_all_generated_path}/crypto/modes/ghashv8-armx.S",
  "${build_all_generated_path}/crypto/poly1305/poly1305-armv8.S",
  "${build_all_generated_path}/crypto/sha/keccak1600-armv8.S",
  "${build_all_generated_path}/crypto/sha/sha1-armv8.S",
  "${build_all_generated_path}/crypto/sha/sha256-armv8.S",
  "${build_all_generated_path}/crypto/sha/sha512-armv8.S",
]
```

### x86_64 平台汇编源文件

x86_64 平台支持最丰富的汇编优化，包括 AES-NI、SSE、AVX、AVX2、AVX512 等指令集扩展：

```gn
libcrypto_build_all_generated_linux_x86_64_sources = [
  "${build_all_generated_path}/crypto/aes/aes-x86_64.s",
  "${build_all_generated_path}/crypto/aes/aesni-mb-x86_64.s",
  "${build_all_generated_path}/crypto/aes/aesni-sha1-x86_64.s",
  "${build_all_generated_path}/crypto/aes/aesni-sha256-x86_64.s",
  "${build_all_generated_path}/crypto/aes/aesni-x86_64.s",
  "${build_all_generated_path}/crypto/aes/bsaes-x86_64.s",
  "${build_all_generated_path}/crypto/aes/vpaes-x86_64.s",
  "${build_all_generated_path}/crypto/bn/rsaz-avx2.s",
  "${build_all_generated_path}/crypto/bn/rsaz-avx512.s",
  "${build_all_generated_path}/crypto/bn/rsaz-x86_64.s",
  "${build_all_generated_path}/crypto/bn/x86_64-mont.s",
  "${build_all_generated_path}/crypto/bn/x86_64-mont5.s",
  "${build_all_generated_path}/crypto/sha/sha1-mb-x86_64.s",
  "${build_all_generated_path}/crypto/sha/sha256-mb-x86_64.s",
  "${build_all_generated_path}/crypto/modes/aesni-gcm-x86_64.s",
  "${build_all_generated_path}/crypto/ec/ecp_nistz256-x86_64.s",
  "${build_all_generated_path}/crypto/ec/x25519-x86_64.s",
  // ... 更多文件
]
```

---

## 构建目标定义

### 动态链接库

```gn
ohos_shared_library("libcrypto_shared") {
  output_name = "libcrypto_openssl"
  sources = libcrypto_sources
  public_configs = [ ":crypto_config_public" ]
  configs = [ ":crypto_config_private" }
  deps = [
    ":openssl_build_all_generated",
    ":libapps",
  ]
}
```

动态链接库命名为 libcrypto_openssl.z.so 和 libssl_openssl.z.so，遵循 OpenHarmony 的库命名规范。

### 静态链接库

```gn
ohos_static_library("libcrypto_static") {
  sources = libcrypto_sources
  public_configs = [ ":crypto_config_public" ]
  configs = [ ":crypto_config_private" }
  deps = [
    ":openssl_build_all_generated",
    ":libapps",
  ]
}
```

### 命令行工具

```gn
ohos_executable("openssl") {
  sources = [
    "apps/openssl.c",
    "apps/s_cb.c",
    // ... 更多源文件
  ]
  configs = [ ":crypto_config_private" ]
  deps = [
    ":libcrypto_shared",
    ":libssl_shared",
  ]
}
```

---

## 升级适配检查点

BUILD.gn 中标记了 29 个「升级修改适配检查点」，这些位置在升级上游版本时需要重点验证：

| 检查点编号 | 配置内容 | 验证要点 |
|------------|----------|----------|
| 1-6 | 各平台汇编源文件列表 | 文件是否存在、名称是否变更 |
| 7-8 | 生成的源文件列表 | libcommon、libdefault 生成文件 |
| 9-10 | 头文件路径和公共 CFLAGS | 配置是否仍然有效 |
| 11-16 | 各平台特定 CFLAGS | 选项是否兼容新版本 |
| 17-19 | Provider 源文件列表 | 是否新增或移除 provider |
| 20-29 | 库源文件列表 | 源文件增删情况 |

---

*文档版本：1.0*  
*创建日期：2026-02-08*  
*参考文件：third_party/openssl/BUILD.gn*