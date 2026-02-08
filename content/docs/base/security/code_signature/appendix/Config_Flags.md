# 配置开关

本文档列出 `code_signature` 组件的所有编译时配置开关（Feature Flags），包括定义位置、默认值、使用条件和相关代码路径。

## 目录

- [根配置](#根配置)
- [接口层配置](#接口层配置)
- [服务层配置](#服务层配置)
- [安全相关配置](#安全相关配置)

---

## 根配置

### 代码签名配置

**文件**：`code_signature.gni`

| 配置项 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `code_signature_support_openharmony_ca` | bool | true | 支持 OpenHarmony CA |
| `code_signature_support_oh_code_sign` | bool | false | OH SDK 代码签名支持 |
| `code_signature_enable_xpm_mode` | int | 0 | XPM 模式 (0-5) |
| `code_signature_support_oh_release_app` | bool | false | Release 应用支持 |
| `code_signature_support_app_allow_list` | bool | false | 应用白名单支持 |
| `code_signature_screenlock_mgr_enable` | bool | false | 屏幕锁管理器支持 |
| `code_signature_support_binary_enable` | bool | false | Binary enable 支持 |
| `jit_code_sign_enable` | bool | false | JIT 代码签名支持 |

### JIT 代码签名自动启用条件

```gn
if (defined(target_cpu) && target_cpu == "arm64" &&
    code_signature_support_oh_code_sign && !is_emulator) {
  jit_code_sign_enable = true
}
```

### 路径变量

| 变量 | 值 |
|------|-----|
| `code_signature_root_dir` | `//base/security/code_signature` |
| `fsverity_utils_dir` | `//third_party/fsverity-utils` |
| `openssl_dir` | `//third_party/openssl` |
| `rust_openssl_dir` | `//third_party/rust/crates/rust-openssl` |
| `selinux_dir` | `//third_party/selinux` |

---

## 接口层配置

### code_sign_utils

**文件**：`interfaces/inner_api/code_sign_utils/BUILD.gn`

| 配置项 | 类型 | 条件 | 定义 |
|--------|------|------|------|
| `SUPPORT_BINARY_ENABLE` | define | `code_signature_support_binary_enable` | ELF 签名块支持 |
| `SUPPORT_OH_CODE_SIGN` | define | `code_signature_support_oh_code_sign` | OH SDK 签名支持 |
| `SUPPORT_PERMISSIVE_MODE` | define | `build_variant == "root" \|\| code_signature_enable_xpm_mode == 0` | 宽容模式 |

### code_sign_attr_utils

**文件**：`interfaces/inner_api/code_sign_attr_utils/BUILD.gn`

| 配置项 | 类型 | 条件 | 定义 |
|--------|------|------|------|
| `SUPPORT_APP_ALLOW_LIST` | define | `code_signature_support_app_allow_list` | 应用白名单 |

### local_code_sign

**文件**：`interfaces/inner_api/local_code_sign/BUILD.gn`

| 配置项 | 类型 | 条件 | 定义 |
|--------|------|------|------|
| `CODE_SIGNATURE_DEBUGGABLE` | define | `build_variant == "root"` | 调试模式 |
| `VERIFY_KEY_ATTEST_CERTCHAIN` | define | `code_signature_support_oh_code_sign` | 证书链验证 |

### jit_code_sign

**文件**：`interfaces/inner_api/jit_code_sign/BUILD.gn`

| 配置项 | 类型 | 值 | 描述 |
|--------|------|-----|------|
| `ARCH_PAC_SUPPORT` | define | - | ARM PAC 支持 |
| 编译标志 | cflags | `-march=armv8.4-a` | ARM 架构 |

---

## 服务层配置

### local_code_sign (SA)

**文件**：`services/local_code_sign/BUILD.gn`

| 配置项 | 类型 | 描述 |
|--------|------|------|
| shlib_type | string | "sa" (SA 类型共享库) |
| install_images | list | ["system"] |

### key_enable (Rust)

**文件**：`services/key_enable/BUILD.gn`

| 环境变量 | 类型 | 条件 | 描述 |
|----------|------|------|------|
| `code_signature_debuggable` | string | `build_variant == "root"` → "on" | 调试模式 |
| | | else | "off" |
| `support_openharmony_ca` | string | `code_signature_support_oh_release_app` → "on" | CA 支持 |
| | | else | "off" |

---

## 安全相关配置

### CFI (Control Flow Integrity)

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

**应用位置**：
- `libcode_sign_utils`
- `liblocal_code_sign_sdk`
- `libjit_code_sign`
- `liblocal_code_sign`
- `fsverity_sign_src_set`

### Branch Protection

```gn
branch_protector_ret = "pac_ret"
```

**应用位置**：所有共享库 targets

### 优化标志

```gn
cflags_cc = [
  "-Os",
  "-fno-asynchronous-unwind-tables",
  "-fno-unwind-tables",
]
```

**应用位置**：
- `libcode_sign_utils`
- `liblocal_code_sign_sdk`
- `libjit_code_sign`
- `liblocal_code_sign`

---

## 配置组合示例

### 标准模式 (Default)

```gn
code_signature_support_oh_code_sign = false
code_signature_enable_xpm_mode = 0
code_signature_support_oh_release_app = false
code_signature_support_app_allow_list = false
code_signature_support_binary_enable = false
jit_code_sign_enable = false
```

### OH SDK 签名模式

```gn
code_signature_support_oh_code_sign = true
code_signature_enable_xpm_mode = 1
code_signature_support_oh_release_app = true
code_signature_support_app_allow_list = false
code_signature_support_binary_enable = false
jit_code_sign_enable = true  # arm64 only, !is_emulator
```

### Root 调试模式

```gn
build_variant = "root"
code_signature_support_binary_enable = true
# 附加 defines:
#   CODE_SIGNATURE_DEBUGGABLE
#   SUPPORT_PERMISSIVE_MODE
#   code_signature_debuggable=on
```

### Release 模式

```gn
build_variant = "release"
code_signature_support_oh_code_sign = true
code_signature_enable_xpm_mode = 3
code_signature_support_oh_release_app = true
# 附加 defines:
#   SUPPORT_PERMISSIVE_MODE (if xpm_mode == 0)
```

---

## 配置与产物的映射

| 配置组合 | 主要产物 | 附加产物 |
|----------|----------|----------|
| Default | libcode_sign_utils.z.so | - |
| + binary_enable | libcode_sign_utils.z.so | elf_code_sign_block.cpp |
| + oh_code_sign | libcode_sign_utils.z.so, libjit_code_sign.z.so | - |
| Root variant | 所有产物 | debug symbols, permissive mode |

---

## SA 配置

### local_code_sign.cfg

```json
{
    "name": "local_code_sign",
    "ondemand": true,
    "apl": "system_basic",
    "uid": "code_sign",
    "gid": "code_sign",
    "permission": [
        "ohos.permission.ATTEST_KEY",
        "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED"
    ]
}
```

### key_enable.cfg

根据 `code_signature_enable_xpm_mode` 选择不同配置：

| XPM 模式 | 配置文件 |
|----------|----------|
| 0 | `cfg/disable_xpm/key_enable.cfg` |
| 1 | `cfg/enable_xpm/level1/key_enable.cfg` |
| 2 | `cfg/enable_xpm/level2/key_enable.cfg` |
| 3 | `cfg/enable_xpm/level3/key_enable.cfg` |
| 4 | `cfg/enable_xpm/level4/key_enable.cfg` |
| 5 | `cfg/enable_xpm/level5/key_enable.cfg` |
