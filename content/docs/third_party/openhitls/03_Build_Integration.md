# OpenHarmony 构建适配详解

## 概述

openHiTLS 通过 `BUILD.gn` 文件实现与 OpenHarmony 构建系统的集成。**无代码修改，纯配置驱动**。

---

## 1. BUILD.gn 结构总览

### 1.1 文件信息

| 属性 | 值 |
|-----|-----|
| **文件大小** | 约 36 KB |
| **代码行数** | 1170 行 |
| **主要部分** | 5 个输出目标 + 配置 |

### 1.2 文件结构

```
BUILD.gn
├── 导入和参数声明 (1-25 行)
├── 平台检测逻辑 (26-46 行)
├── 宏定义配置 (47-185 行)
│   ├── ARMv8 优化宏
│   ├── x86_64 优化宏
│   └── 通用功能宏 (100+)
├── BSL 组件 (193-292 行)
├── Crypto 组件 (294-792 行)
├── PKI 组件 (794-879 行)
├── TLS 组件 (881-1110 行)
└── Auth 组件 (1112-1170 行)
```

---

## 2. 关键配置详解

### 2.1 平台检测

```gn (lines 26-36)
openhitls_selected_platform = ""

if (current_cpu == "arm64" && current_os == "ohos" && host_os == "linux") {
    print("openhitls selected linux-armv8")
    openhitls_selected_platform = "linux-armv8"
} else if (current_cpu == "x86_64" && current_os == "ohos" && host_os == "linux") {
    print("openhitls selected linux-x86_64")
    openhitls_selected_platform = "linux-x86_64"
}
```

**说明**：
- 自动检测目标平台架构
- 支持 ARM64 (linux-armv8) 和 x86_64 (linux-x86_64)
- 其他平台使用纯 C 实现（无汇编优化）

### 2.2 链接选项

```gn (lines 38-46)
public_ldflags = [
    "-fPIC",
    "-Wl,-Bsymbolic"
]

# ARM64 特定选项
if (current_cpu == "arm64" && current_os == "ohos") {
    public_ldflags += [ "-Wl,--lto-O0" ]
}
```

**说明**：
- `-fPIC`: 生成位置无关代码（共享库必需）
- `-Wl,-Bsymbolic`: 优先绑定本地符号
- `-Wl,--lto-O0`: ARM64 禁用 LTO 优化（解决兼容性问题）

### 2.3 功能宏定义

BUILD.gn 定义了约 **100+ 个宏** 控制功能开关：

#### ARMv8 优化宏 (lines 50-64)

```gn
public_armv8_defines = [
    "HITLS_CRYPTO_AES_ARMV8",           # AES ARMv8 汇编优化
    "HITLS_CRYPTO_BN_ARMV8",            # 大数运算 ARMv8 优化
    "HITLS_CRYPTO_CHACHA20_ARMV8",      # ChaCha20 ARMv8 优化
    "HITLS_CRYPTO_CHACHA20POLY1305_ARMV8",
    "HITLS_CRYPTO_ECC_ARMV8",           # ECC ARMv8 优化
    "HITLS_CRYPTO_GCM_ARMV8",           # GCM 模式 ARMv8 优化
    "HITLS_CRYPTO_SHA1_ARMV8",          # SHA1 ARMv8 优化
    "HITLS_CRYPTO_SHA2_ARMV8",          # SHA2 ARMv8 优化
    "HITLS_CRYPTO_SHA3",                # SHA3 支持
    "HITLS_CRYPTO_SHA3_ARMV8",          # SHA3 ARMv8 优化
    "HITLS_CRYPTO_SM3_ARMV8",           # SM3 ARMv8 优化 (国密)
    "HITLS_CRYPTO_SM4_ARMV8",           # SM4 ARMv8 优化 (国密)
    "HITLS_CRYPTO_X25519_ARMV8",        # X25519 ARMv8 优化
]
```

#### x86_64 优化宏 (lines 66-78)

类似 ARMv8，启用 x86_64 汇编优化版本。

#### 通用功能宏 (lines 80-175)

**BSL 功能宏**：
```gn
"HITLS_BSL_UIO_BUFFER",     # UIO 缓冲区支持
"HITLS_BSL_UIO_MEM",        # UIO 内存支持
"HITLS_BSL_UIO_TCP",        # UIO TCP 支持
"HITLS_BSL_SAL_LINUX",      # Linux SAL 适配
"HITLS_BSL_SAL_FILE",       # 文件操作
"HITLS_BSL_SAL_MEM",        # 内存操作
"HITLS_BSL_SAL_NET",        # 网络操作
# ... 更多
```

**Crypto 功能宏**：
```gn
"HITLS_CRYPTO_AES",         # AES 算法
"HITLS_CRYPTO_SM4",         # SM4 国密算法
"HITLS_CRYPTO_SHA1",        # SHA1
"HITLS_CRYPTO_SHA2",        # SHA2
"HITLS_CRYPTO_SHA3",        # SHA3
"HITLS_CRYPTO_SM3",         # SM3 国密算法
"HITLS_CRYPTO_RSA",         # RSA
"HITLS_CRYPTO_ECC",         # 椭圆曲线
"HITLS_CRYPTO_ECDSA",       # ECDSA
"HITLS_CRYPTO_ECDH",        # ECDH
"HITLS_CRYPTO_SM2",         # SM2 国密算法
"HITLS_CRYPTO_MLKEM",       # ML-KEM 后量子
"HITLS_CRYPTO_MLDSA",       # ML-DSA 后量子
"HITLS_CRYPTO_SLH_DSA",     # SLH-DSA 后量子
# ... 更多
```

**TLS 功能宏**：
```gn
"HITLS_TLS",                      # TLS 支持
"HITLS_TLS_PROTO_TLCP11",         # TLCP 国密协议 (关键！)
"HITLS_TLS_SUITE_ECDHE_SM4_CBC_SM3",   # 国密套件
"HITLS_TLS_SUITE_ECC_SM4_CBC_SM3",
"HITLS_TLS_SUITE_ECDHE_SM4_GCM_SM3",
"HITLS_TLS_SUITE_ECC_SM4_GCM_SM3",
# ... 更多
```

**关键宏**：`HITLS_TLS_PROTO_TLCP11` 启用国密 TLCP 协议支持，这是 curl 国密 HTTPS 的基础。

---

## 3. 组件构建配置

### 3.1 BSL (Base Support Layer)

```gn (lines 277-292)
ohos_shared_library("openhitls_bsl") {
    subsystem_name = "thirdparty"
    part_name = "openhitls"
    sources = []
    deps = [":bsl_source"]
    
    # 关键：NDK 暴露
    innerapi_tags = ["llndk", "ndk"]
    
    public_configs = [":bsl_public_config"]
    
    # 安装到 system 和 updater 镜像
    install_images = ["system", "updater"]
}
```

**说明**：
- `innerapi_tags = ["llndk", "ndk"]`：**关键配置**，允许应用通过 NDK 调用
- `install_images`: 同时安装到 system（运行）和 updater（升级）镜像

### 3.2 Crypto (密码算法)

```gn (lines 775-792)
ohos_shared_library("openhitls_crypto") {
    subsystem_name = "thirdparty"
    part_name = "openhitls"
    sources = []
    deps = [
        ":crypto_source",
        # 依赖 BSL
    ]
    innerapi_tags = ["llndk", "ndk"]
    public_configs = [":crypto_public_config"]
    install_images = ["system", "updater"]
}
```

**源文件选择逻辑** (lines 727-738)：
```gn
if (openhitls_selected_platform == "linux-armv8") {
    # ARMv8: 使用汇编优化源文件
    sources += armv8_asm_sources
} else if (openhitls_selected_platform == "linux-x86_64") {
    # x86_64: 使用汇编优化源文件
    sources += x86_64_asm_sources
} else {
    # 其他: 使用纯 C 实现
    sources += c_only_sources
}
```

**源文件分类**：
- `openhitls_libcrypto_build_all_generated_linux_armv8_sources` (lines 357-396)
- `openhitls_libcrypto_build_all_generated_linux_x8664_sources` (lines 398-441)
- `openhitls_libcrypto_build_all_generated_linux_c_sources` (lines 443-488)

### 3.3 PKI (证书处理)

```gn (lines 862-879)
ohos_shared_library("openhitls_pki") {
    # ...
    deps = [":openhitls_bsl", ":openhitls_crypto"]
    # ...
}
```

**源文件** (lines 824-836)：
```gn
sources = [
    "pki/cms/src/hitls_cms_common.c",
    "pki/pkcs12/src/hitls_pkcs12_common.c",
    "pki/x509_cert/src/hitls_x509_cert.c",
    "pki/x509_verify/src/hitls_x509_verify.c",
    # ... 共 13 个源文件
]
```

### 3.4 TLS (协议实现)

```gn (lines 1093-1110)
ohos_shared_library("openhitls_tls") {
    # ...
    deps = [":openhitls_bsl", ":openhitls_crypto", ":openhitls_pki"]
    # ...
}
```

**源文件** (lines 929-1066)：约 130+ 个源文件，涵盖：
- 连接管理 (`tls/cm/`)
- 加密适配 (`tls/crypt/`)
- 告警处理 (`tls/alert/`)
- 握手协议 (`tls/handshake/`)
- 记录层 (`tls/record/`)
- 特性扩展 (`tls/feature/`)

### 3.5 Auth (认证)

```gn (lines 1154-1169)
ohos_shared_library("openhitls_auth") {
    # ...
    deps = [":openhitls_bsl", ":openhitls_crypto"]
    # ...
}
```

**源文件** (lines 1128-1132)：
```gn
sources = [
    "auth/privpass_token/src/privpass_token.c",
    "auth/privpass_token/src/privpass_token_util.c",
    "auth/privpass_token/src/privpass_token_wrapper.c",
]
```

---

## 4. 公共依赖

### 4.1 外部依赖

所有组件共享的外部依赖：

```gn
external_deps = [
    "bounds_checking_function:libsec_shared"
]
```

**说明**：依赖 `bounds_checking_function` 组件的安全函数库，提供字符串和内存安全操作。

### 4.2 公共编译选项

```gn
cflags = [
    "-Wno-int-conversion",  # 禁用 int 转换警告
    "-fPIC",                # 位置无关代码
]

defines = all_defines     # 所有功能宏

ldflags = public_ldflags  # 链接选项
```

---

## 5. 与上游构建系统对比

### 5.1 上游构建方式

openHiTLS 上游支持：
- **CMake**: `CMakeLists.txt` + `configure.py` 配置
- **Makefile**: 底层编译

### 5.2 OpenHarmony 适配

| 特性 | 上游 CMake | OH BUILD.gn |
|-----|-----------|-------------|
| 平台检测 | `configure.py` 参数 | GN 条件判断 |
| 特性选择 | `--enable` 参数 | 宏定义列表 |
| 源文件选择 | CMake 条件 | GN 条件 + 源文件列表 |
| 汇编优化 | CMake 检测 | GN 平台检测 |
| 输出格式 | 静态/动态库可选 | 共享库 |
| 安装位置 | `make install` | system/updater 镜像 |

### 5.3 关键差异

**OH 特有配置**：
1. `innerapi_tags = ["llndk", "ndk"]` - NDK 暴露
2. `install_images = ["system", "updater"]` - 双镜像安装
3. `subsystem_name = "thirdparty"` - OH 子系统声明
4. `part_name = "openhitls"` - OH 组件声明
5. ARM64 LTO 禁用 `-Wl,--lto-O0` - 兼容性处理

---

## 6. 使用示例

### 6.1 在 BUILD.gn 中引用

```gn
# 使用 Crypto 组件
ohos_shared_library("my_lib") {
    sources = ["my_source.c"]
    external_deps = [
        "openhitls:openhitls_crypto",
    ]
}

# 使用 TLS 组件（会自动依赖 crypto, bsl, pki）
ohos_shared_library("my_tls_lib") {
    sources = ["my_tls_source.c"]
    external_deps = [
        "openhitls:openhitls_tls",
    ]
}
```

### 6.2 完整示例（参考 curl）

```gn
# 来自 third_party/curl/BUILD.gn
if (defined(global_parts_info.thirdparty_openhitls) &&
    global_parts_info.thirdparty_openhitls) {
    
    support_gmssl = true
    
    deps += [
        "openhitls:openhitls_bsl",
        "openhitls:openhitls_crypto",
        "openhitls:openhitls_pki",
        "openhitls:openhitls_tls",
        "openhitls:openhitls_auth",
    ]
    
    sources += ["lib/vtls/openhitls.c"]  # curl 适配层
}
```

---

## 7. 构建命令

### 7.1 编译 openHiTLS

```bash
# 全量编译
./build.sh --product-name rk3568 --ccache --build-target openhitls

# 编译单个组件
./build.sh --product-name rk3568 --build-target //third_party/openhitls:openhitls_crypto
```

### 7.2 输出位置

```
out/rk3568/thirdparty/openhitls/
├── libopenhitls_bsl.so
├── libopenhitls_crypto.so
├── libopenhitls_pki.so
├── libopenhitls_tls.so
└── libopenhitls_auth.so
```

---

## 8. 维护指南

### 8.1 升级检查清单

当升级 openHiTLS 版本时，检查以下 BUILD.gn 相关项：

- [ ] **源文件变更**: 上游是否新增/删除/重命名 `.c` 文件
- [ ] **头文件路径**: `include_dirs` 是否需要调整
- [ ] **宏定义变更**: 是否有新增功能宏
- [ ] **依赖变更**: `external_deps` 是否需要更新
- [ ] **API 变更**: `public_configs` 的 include_dirs 是否仍正确

### 8.2 添加新算法支持

如需启用新算法：

1. 在 `all_defines` 列表中添加对应宏（如 `HITLS_CRYPTO_NEWALG`）
2. 添加对应的源文件到源文件列表
3. 更新头文件路径（如有新头文件）

### 8.3 调试构建问题

```gn
# BUILD.gn 中有诊断打印
print("current_cpu = ${current_cpu}")
print("current_os = ${current_os}")
print("host_os = ${host_os}")
print("openhitls selected linux-armv8")
print("openhitls detecting os done, openhitls_selected_platform = ${openhitls_selected_platform}")
```

查看构建日志可确认平台检测是否正确。
