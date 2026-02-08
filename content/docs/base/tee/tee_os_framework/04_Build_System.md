# 构建系统

## 1. 构建系统概述

### 1.1 系统类型

| 类型 | 使用范围 | 证据 |
|------|----------|------|
| **GNU Make** | 所有框架组件 (framework, services, drivers, libraries) | `build/Makefile` |
| **GN** | 仅测试 CA (XTS 测试套件) | `test/xts/ca/tee/BUILD.gn` |

### 1.2 构建入口

```bash
# 从 OpenHarmony 根目录构建
./build.sh --product-name rk3568 --build-target tee --ccache

# 手动构建 (在 build/ 目录)
make TARGET_BOARD_PLATFORM=oh_64 FRAMEWORK_ELF_PATH=./elf_out all
```

**证据**：`build/Makefile:148-156`
```makefile
PHONY += libs $(drivers) $(platform_drivers) $(frameworks) $(service) package
all: install_headers setup_links libs $(drivers) $(platform_drivers) $(frameworks) $(service) package
tees: setup_links libs $(drivers) $(platform_drivers) $(frameworks) $(service)
$(drivers): setup_links libs link_libs
$(platform_drivers): check_platform setup_links libs link_libs
$(frameworks): setup_links libs link_libs
$(service): setup_links libs link_libs
```

---

## 2. 构建配置

### 2.1 顶层配置

| 文件 | 描述 |
|------|------|
| `bundle.json` | OpenHarmony 部件清单 |
| `build/Makefile` | 顶层 Makefile |
| `build/build_framework.sh` | 构建脚本 |
| `build/clean_framework.sh` | 清理脚本 |

### 2.2 构建系统配置

| 文件 | 描述 |
|------|------|
| `build/mk/common/config/var.mk` | 变量定义 |
| `build/mk/common/config/toolchain.mk` | 工具链配置 |
| `build/mk/common/config/arch_config.mk` | 架构配置 |
| `build/mk/common/config/teeos-flags.mk` | 编译器标志 |
| `build/mk/common/config/feature-macro.mk` | 功能宏 |
| `build/mk/common/config/cfg.mk` | 包含路径 |

### 2.3 构建规则

| 文件 | 描述 |
|------|------|
| `build/mk/common/operation/project.mk` | 项目目标 |
| `build/mk/common/operation/common.mk` | 通用操作 |
| `build/mk/common/operation/rule.mk` | 构建规则 |
| `build/mk/lib_common/lib-common.mk` | 库构建规则 |
| `build/mk/service_common/svc-common.mk` | 服务构建规则 |
| `build/mk/service_common/svc-flags.mk` | 服务编译标志 |
| `build/mk/driver_common/drv-common.mk` | 驱动构建规则 |
| `build/mk/driver_common/drv-flags.mk` | 驱动编译标志 |
| `build/mk/framework_common/ta-common.mk` | TA 构建规则 |

### 2.4 平台配置

| 文件 | 描述 |
|------|------|
| `config/release_config/oh_64_release_config` | 64 位发布配置 |
| `config/release_config/oh_32_release_config` | 32 位发布配置 |
| `config/debug_config/oh_64_debug_config` | 64 位调试配置 |
| `config/debug_config/oh_32_debug_config` | 32 位调试配置 |

---

## 3. 产物清单

### 3.1 Frameworks (ELF 可执行文件)

| 组件 | 64位产物 | 32位产物 | 架构 |
|------|----------|----------|------|
| gtask | `gtask.elf` | `gtask_a32.elf` | 64/32-bit |
| teesmcmgr | `teesmcmgr.elf` | - | 64-bit |
| tarunner | `tarunner.elf` | `tarunner_a32.elf` | 64/32-bit |
| drvmgr | `drvmgr.elf` | - | 64-bit |

### 3.2 Services (ELF 可执行文件)

| 组件 | 64位产物 | 32位产物 | 架构 |
|------|----------|----------|------|
| ssa | `ssa.elf` | `ssa_a32.elf` | 64/32-bit |
| permission_service | `permission_service.elf` | - | 64-bit |
| huk_service | `huk_service.elf` | `huk_service_a32.elf` | 64/32-bit |

### 3.3 Drivers (ELF/SO)

| 组件 | 产物 | 类型 |
|------|------|------|
| crypto_mgr | `crypto_mgr.elf` | ELF |
| tee_misc_driver | `tee_misc_driver.elf` | ELF |
| libhardware_crypto_drv | `libhardware_crypto_drv.so` | Shared Lib |

### 3.4 Libraries (静态库 .a)

| 库 | 产物 | 描述 |
|------|------|------|
| libtee_shared | `libtee_shared.so` | 共享库 (64-bit) |
| libdrv_shared | `libdrv_shared.so` | 驱动共享库 (64-bit) |
| libcrypto_hal | `libcrypto_hal.a` | Crypto HAL |
| libtimer | `libtimer.a` | 定时器 |
| libteeos | `libteeos.a` | TEE OS |
| libssa | `libssa.a` | 安全存储 |
| libhuk | `libhuk.a` | HUK |
| libpermission_service | `libpermission_service.a` | 权限服务 |
| libswcrypto_engine | `libswcrypto_engine.a` | 软件加密引擎 |
| libopenssl | `libopenssl.a` | OpenSSL |
| libteedynsrv | `libteedynsrv.a` | 动态服务 |
| libagent | `libagent.a` | Agent |
| libagent_base | `libagent_base.a` | Base Agent |
| libdrv | `libdrv.a` | 驱动 |
| libtaentry | `libtaentry.a` | TA 入口 |
| libteeagentcommon_client | `libteeagentcommon_client.a` | Agent 客户端 |
| libcrypto | `libcrypto.a` | 加密 |
| libteemem | `libteemem.a` | 内存管理 |
| libipc_hal | `libipc_hal.a` | IPC HAL |
| libtee_stub | `libtee_stub.a` | 桩实现 |
| libse | `libse.a` | SE |
| libteeconfig | `libteeconfig.a` | 配置 |

### 3.5 Sys Libraries (静态库)

| 库 | 产物 | 描述 |
|------|------|------|
| libspawn_common | `libspawn_common.a` | 进程创建 |
| libelf_verify | `libelf_verify.a` | ELF 验签 |
| libelf_verify_key | `libelf_verify_key.a` | ELF 验签密钥 |
| libdynconfmgr | `libdynconfmgr.a` | 动态配置 |
| libdynconfbuilder | `libdynconfbuilder.a` | 动态配置构建 |

---

## 4. 输出目录结构

```
output/
├── aarch64/                          # 64位构建
│   ├── apps/                         # 服务可执行文件
│   │   ├── teesmcmgr.elf
│   │   ├── permission_service.elf
│   │   └── huk_service.elf
│   ├── drivers/                      # 驱动
│   │   ├── gtask.elf
│   │   ├── drvmgr.elf
│   │   ├── tarunner.elf
│   │   ├── ssa.elf
│   │   └── crypto_mgr.elf
│   ├── libs/                         # 静态库
│   │   ├── libcrypto_hal.a
│   │   ├── libtimer.a
│   │   └── ...
│   └── obj/                          # 目标文件和共享库
│       └── aarch64/
│           └── libtee_shared/
│               └── libtee_shared.so
├── arm/                              # 32位构建 (当启用时)
│   ├── apps/
│   ├── drivers/
│   ├── libs/
│   └── obj/
│       └── arm/
│           └── libtee_shared/
│               └── libtee_shared_a32.so
├── headers/                          # 安装的头文件
└── kernel/                           # 内核相关产物
```

---

## 5. 编译开关

### 5.1 架构配置

| 开关 | 描述 | 证据 |
|------|------|------|
| `CONFIG_SUPPORT_64BIT=true` | 启用 64 位构建 | `oh_64_release_config` |
| `CONFIG_TA_64BIT=true` | TA 构建为 64 位 | `oh_64_release_config` |
| `CONFIG_SSA_64BIT=true` | SSA 服务为 64 位 | `oh_64_release_config` |
| `CONFIG_GTASK_64BIT=true` | gtask 为 64 位 | `oh_64_release_config` |
| `CONFIG_HUK_SERVICE_64BIT=true` | HUK 服务为 64 位 | `oh_64_release_config` |
| `CONFIG_PERMSRV_64BIT=true` | 权限服务为 64 位 | `oh_64_release_config` |
| `CONFIG_DRVMGR_64BIT=true` | drvmgr 为 64 位 | `oh_64_release_config` |
| `CONFIG_ARCH_AARCH64=y` | 目标 ARM64 架构 | `oh_64_release_config` |
| `CONFIG_ARM_CORTEX_A53=y` | 目标 Cortex-A53 CPU | `oh_64_release_config` |

### 5.2 服务启用

| 开关 | 描述 |
|------|------|
| `CONFIG_APP_TEE_GTASK=y` | 启用 gtask |
| `CONFIG_APP_TEE_SSA=y` | 启用 SSA |
| `CONFIG_APP_TEE_PERM=y` | 启用权限服务 |
| `CONFIG_APP_TEE_HUK=y` | 启用 HUK 服务 |
| `CRYPTO_MGR_SERVER_ENABLE=y` | 启用 Crypto Manager |

### 5.3 加密配置

| 开关 | 描述 |
|------|------|
| `CONFIG_CRYPTO_SOFT_ENGINE=openssl3` | 使用 OpenSSL 3.x |
| `CONFIG_CRYPTO_SUPPORT_EC25519=true` | 启用 EC25519 |
| `CONFIG_CRYPTO_ECC_WRAPPER=true` | 启用 ECC 包装 |
| `CONFIG_CRYPTO_AES_WRAPPER=true` | 启用 AES 包装 |
| `CONFIG_CRYPTO_SUPPORT_SOFT_ECC=true` | 启用软件 ECC |
| `CONFIG_HADRWARE_CRYPTO_ENABLE=y` | 启用硬件加密 |

### 5.4 构建选项

| 开关 | 描述 |
|------|------|
| `CONFIG_LLVM_LTO=y` | 启用 LLVM LTO |
| `CONFIG_LLVM_CFI=y` | 启用 CFI |
| `CONFIG_USER_OPTIMIZATION_Os=y` | 优化大小 (-Os) |
| `CONFIG_CC_STACKPROTECTOR_STRONG=y` | 堆栈保护 |
| `CONFIG_USER_LINKER_GC_SECTIONS=y` | 链接器 GC |
| `CONFIG_USER_FULL_RELRO=y` | 完整 RELRO |
| `CONFIG_UNALIGNED_ACCESS=y` | 非对齐访问 |

### 5.5 TA/动态加载

| 开关 | 描述 |
|------|------|
| `CONFIG_DYN_CONF=true` | 启用动态配置 |
| `CONFIG_DYN_TA_FORMAT=3` | 动态 TA 格式版本 3 |
| `CONFIG_DYNLIB_LOAD_SUPPORT=y` | 启用动态库加载 |
| `CONFIG_TA_LOCAL_SIGN=true` | 启用本地 TA 签名 |

---

## 6. 工具链配置

### 6.1 默认工具链

| 配置 | 默认值 | 环境变量 |
|------|--------|----------|
| 编译器 | LLVM Clang 15.0.4 | `LLVM_BASEVER` |
| 编译目录 | `$(TOPDIR)/../../../../tools/llvm` | `TEE_COMPILER_DIR` |
| 目标架构 | aarch64 | `TARGET_BOARD_PLATFORM` |
| 安全库目录 | `$(THIRDPARTY)/bounds_checking_function` | `TEE_SECUREC_DIR` |

### 6.2 交叉编译目标

| 架构 | 目标 | 前缀 |
|------|------|------|
| 64位 | `aarch64-linux-gnu` | `aarch64-linux-gnu-` |
| 32位 | `arm-linux-gnueabi` | `arm-linux-gnueabi-` |

---

## 7. 关键依赖

### 7.1 libtee_shared.so 依赖

```
libtee_shared.so
├── libcrypto_hal
├── libtimer
├── libagent
├── libagent_base
├── libdrv
├── libteeos
├── libpermission_service
├── libswcrypto_engine
├── libtaentry
├── libteeagentcommon_client
├── libcrypto
├── libteeconfig
├── libteemem
├── libssa
├── libhuk
├── libteedynsrv
├── libipc_hal
├── libtee_stub
└── libopenssl
```

### 7.2 gtask.elf 依赖

```
gtask.elf
├── libc_shared
├── libtee_shared
├── libpermission_service
├── libteemem
├── libdrv
├── libteeconfig
├── libdynconfmgr
├── libdynconfbuilder
└── libspawn_common
```

---

## 8. 构建命令参数

### 8.1 build_framework.sh 参数

```bash
./build_framework.sh <platform> <elf_path> <compiler_dir> <llvm_ver> <topdir> <thirdparty> <platform_name>
```

| 参数 | 描述 | 示例 |
|------|------|------|
| `platform` | 目标平台 | `oh_64`, `oh_32` |
| `elf_path` | ELF 输出路径 | `./elf_out` |
| `compiler_dir` | 编译器目录 | `/path/to/llvm` |
| `llvm_ver` | LLVM 版本 | `15.0.4` |
| `topdir` | 顶层目录 | `/path/to/tee_os_framework` |
| `thirdparty` | 第三方库目录 | `/path/to/third_party` |
| `platform_name` | 平台名称 | `rk3568` |

### 8.2 Make 变量

| 变量 | 描述 | 默认值 |
|------|------|--------|
| `TARGET_BOARD_PLATFORM` | 目标平台 | `oh_64` |
| `FRAMEWORK_ELF_PATH` | ELF 输出路径 | `$(TOPDIR)/elf_out` |
| `TEE_COMPILER_DIR` | 编译器目录 | `$(TOPDIR)/../../../../tools/llvm` |
| `LLVM_BASEVER` | LLVM 版本 | `15.0.4` |
| `THIRDPARTY` | 第三方库 | 自动检测 |

---

## 9. 构建流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    build_framework.sh                            │
│                                                                  │
│  1. GEN_CONF_FILE                                                │
│     └── 生成平台 autoconf 头 (platautoconf.h)                    │
│                                                                  │
│  2. PREBUILD_OPENSSL                                             │
│     └── 构建 OpenSSL（如需要）                                   │
│                                                                  │
│  3. install_headers                                              │
│     └── 安装头文件到 HDR_INSTALL_DIR                              │
│                                                                  │
│  4. libs                                                         │
│     └── 构建所有静态库 (.a)                                       │
│                                                                  │
│  5. tees                                                         │
│     ├── drivers (crypto_mgr, tee_misc_driver)                    │
│     ├── platform_drivers (硬件特定驱动)                          │
│     ├── frameworks (gtask, teesmcmgr, drvmgr, tarunner)         │
│     └── services (ssa, permission_service, huk_service)         │
│                                                                  │
│  6. package                                                      │
│     └── 复制 apps 到 FRAMEWORK_ELF_PATH/apps                     │
│                                                                  │
│  7. check_sym                                                    │
│     └── 检查符号表                                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. GN 构建 (仅测试)

### 10.1 测试 GN 文件

| 文件 | 描述 |
|------|------|
| `test/xts/ca/tee/BUILD.gn` | 测试套件组 |
| `test/xts/ca/tee/tee_test_client_api_vendor/BUILD.gn` | Vendor 测试 |
| `test/xts/ca/tee/tee_test_client_api_system/BUILD.gn` | System 测试 |
| `test/xts/ca/tee/tee_test_crypto_api/BUILD.gn` | Crypto 测试 |
| `test/xts/ca/tee/tee_test_tcf_api/BUILD.gn` | TCF 测试 |
| `test/xts/ca/tee/tee_test_time_api/BUILD.gn` | Time 测试 |
| `test/xts/ca/tee/tee_test_arithmetic_api/BUILD.gn` | Arithmetic 测试 |
| `test/xts/ca/tee/tee_test_trusted_storage_api/BUILD.gn` | Storage 测试 |
| `test/xts/ca/tee/tee_test_device_api/BUILD.gn` | Device 测试 |

---

## 相关文档

- 模块详情 → `02_Module_Detail.md`
- Native API → `03_Native_API.md`
- 安全评审 → `05_Security_Review.md`
