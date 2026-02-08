# 关键配置项

本文档汇总 OpenHarmony build 系统中的关键配置项和宏定义。

## 构建变量 (ohos_var.gni)

### 目录配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `system_base_dir` | "system" | 系统包目录 |
| `ramdisk_base_dir` | "ramdisk" | ramdisk 目录 |
| `vendor_base_dir` | "vendor" | vendor 目录 |
| `chipset_base_dir` | "vendor" | chipset 目录 |
| `updater_base_dir` | "updater" | 更新器目录 |
| `sys_prod_base_dir` | "sys_prod" | sys_prod 目录 |
| `eng_system_base_dir` | "eng_system" | eng_system 目录 |
| `chip_prod_base_dir` | "chip_prod" | chip_prod 目录 |
| `cloud_rom_base_dir` | "cloud_rom" | cloud_rom 目录 |
| `ndk_dir` | "ndk" | NDK 目录 |
| `platformsdk_dir` | "platformsdk" | Platform SDK 目录 |
| `chipset_sdk_dir` | "chipset-sdk" | Chipset SDK 目录 |

### SDK/NDK 构建开关

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `build_ohos_sdk` | false | 是否构建 OHOS SDK |
| `build_ohos_ndk` | false | 是否构建 OHOS NDK |
| `sdk_platform` | "default" | SDK 目标平台 |
| `ndk_platform` | "default" | NDK 目标平台 |
| `build_default_sdk_target` | false | 构建默认 SDK |
| `build_mac_sdk_target` | false | 构建 Mac SDK |
| `build_linux_sdk_target` | false | 构建 Linux SDK |
| `build_windows_sdk_target` | false | 构建 Windows SDK |

### 编译选项

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `build_variant` | "root" | 构建变体 |
| `build_framework_compiler_optimize_level` | "-O2" | 编译优化级别 |
| `armv8_1_lse_tuning_optimization` | false | ARMv8.1+ LSE 优化 |
| `enable_java` | is_large_system | 是否启用 Java |
| `build_ark` | true | 是否构建 Ark 字节码 |
| `scalable_build` | false | 是否启用可扩展构建 |
| `pycache_enable` | true | 是否启用 pycache |

### 安全与检查

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `enforce_selinux` | false | 是否强制 SELinux |
| `check_sdk_interface` | true | 检查 SDK 接口 |
| `check_innersdk_interface` | true | 检查内部 SDK 接口 |
| `enable_notice_collection` | true | 启用声明收集 |

### 签名配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `default_key_alias` | "OpenHarmony Application Release" | 默认密钥别名 |
| `default_signature_algorithm` | "SHA256withECDSA" | 签名算法 |
| `default_keystore_path` | "//developtools/hapsigner/dist/OpenHarmony.p12" | 密钥库路径 |
| `default_hap_certificate_file` | "//developtools/hapsigner/dist/OpenHarmonyApplication.pem" | 证书文件 |

## 编译器配置

### Clang 配置

| 变量 | 值 | 说明 |
|------|-----|------|
| `clang_version` | "15.0.4" | Clang 版本 |
| `toolchains_dir` | "//prebuilts/clang/ohos" | 工具链目录 |
| `clang_base_path` | "${toolchains_dir}/${host_platform_dir}/llvm" | Clang 基础路径 |

### 编译标志

```gn
# 安全标志
cflags += [ "-fstack-protector-strong" ]
ldflags += [ "-Wl,-z,noexecstack" ]
ldflags += [ "-Wl,-z,now" ]
ldflags += [ "-Wl,-z,relro" ]

# 优化标志
cflags += [ "-fPIC" ]
cflags_cc += [ "-std=c++17" ]
ldflags += [ "-fuse-ld=lld" ]
```

## Sanitizer 配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `is_asan` | false | Address Sanitizer |
| `use_hwasan` | false | Hardware ASAN |
| `is_lsan` | false | Leak Sanitizer |
| `is_msan` | false | Memory Sanitizer |
| `is_tsan` | false | Thread Sanitizer |
| `is_ubsan` | false | Undefined Behavior Sanitizer |
| `is_cfi` | - | Control Flow Integrity |

## 架构配置

### ARM 配置

```gn
# ARM v7
arm_arch = "armv7-a"
arm_tune = "generic-armv7-a"
arm_float_abi = "softfp"
arm_fpu = "neon"
arm_use_thumb = true

# ARM64
arm_arch = "armv8-a"
arm_cpu = "cortex-a55"
arm_fpu = "neon-fp-armv8"
arm_float_abi = "hard"
```

### 目标平台三元组

| CPU | 三元组 |
|-----|--------|
| arm | arm-linux-ohos |
| arm64 | aarch64-linux-ohos |
| x86_64 | x86_64-linux-ohos |

## 镜像配置

### 镜像大小

| 镜像 | 大小 | 文件系统 |
|------|------|---------|
| system | 1610612224 (1.5GB) | ext4 |
| vendor | 268434944 (256MB) | ext4 |
| userdata | 1468006400 (1.4GB) | f2fs |

## 环境变量

### 构建相关

| 变量 | 说明 |
|------|------|
| `OHOS_ROOT_PATH` | OpenHarmony 根目录 |
| `OHOS_PRODUCT` | 产品名称 |
| `OHOS_TARGET_CPU` | 目标 CPU |
| `OHOS_TARGET_OS` | 目标 OS |

### 工具链相关

| 变量 | 说明 |
|------|------|
| `CC` | C 编译器 |
| `CXX` | C++ 编译器 |
| `AR` | 归档工具 |
| `LD` | 链接器 |

## 配置文件

### 子系统配置

**文件**: `//build/subsystem_config.json`

```json
{
  "subsystem_name": {
    "path": "foundation/subsystem",
    "name": "subsystem_name"
  }
}
```

### 部件配置

**文件**: `bundle.json`

```json
{
  "component": {
    "name": "part_name",
    "subsystem": "subsystem_name",
    "build": {
      "sub_component": ["//path/to:target"],
      "inner_kits": [],
      "test": []
    }
  }
}
```

## 常用 GN 参数

### 构建类型

```bash
# Debug 构建
gn gen out/debug --args='is_debug=true'

# Release 构建
gn gen out/release --args='is_debug=false'

# 组件构建
gn gen out/component --args='is_component_build=true'
```

### 目标平台

```bash
# ARM64
gn gen out/arm64 --args='target_cpu="arm64" target_os="ohos"'

# ARM
gn gen out/arm --args='target_cpu="arm" target_os="ohos"'

# x86_64
gn gen out/x86_64 --args='target_cpu="x86_64" target_os="ohos"'
```

### Sanitizer

```bash
# Address Sanitizer
gn gen out/asan --args='is_asan=true'

# Thread Sanitizer
gn gen out/tsan --args='is_tsan=true'

# Undefined Behavior Sanitizer
gn gen out/ubsan --args='is_ubsan=true'
```

---

*文档生成时间: 2025-02-06*
