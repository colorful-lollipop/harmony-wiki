# GN 构建系统

本文档描述 `code_signature` 组件的 GN 构建配置，包括所有 targets、依赖关系、编译产物和 Feature Flags。

## 1. 根构建文件

### 1.1 根 BUILD.gn

**文件**：`BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/ohos/sa_profile/sa_profile.gni")
import("code_signature.gni")

config("common_public_config") {
  include_dirs = [ "interfaces/inner_api/common/include" ]
}

config("common_utils_config") {
  include_dirs = [ "utils/include" ]
}

group("subcomponents") {
  deps = [
    "${code_signature_root_dir}/interfaces/inner_api/code_sign_attr_utils:libcode_sign_attr_utils",
    "${code_signature_root_dir}/interfaces/inner_api/code_sign_utils:libcode_sign_utils",
    "${code_signature_root_dir}/interfaces/inner_api/jit_code_sign:libjit_code_sign",
    "${code_signature_root_dir}/interfaces/inner_api/local_code_sign:liblocal_code_sign_sdk",
    "${code_signature_root_dir}/services/local_code_sign:liblocal_code_sign",
    "${code_signature_root_dir}/services/local_code_sign:local_code_sign_configs",
  ]

  if (!ohos_indep_compiler_enable) {
    deps += [ "${code_signature_root_dir}/services/key_enable:key_enable_targets" ]
  }
}
```

## 2. Feature Flags

### 2.1 代码签名 Feature Flags

**文件**：`code_signature.gni`

| 开关 | 默认值 | 描述 |
|------|--------|------|
| `code_signature_support_openharmony_ca` | true | 支持 OpenHarmony CA |
| `code_signature_support_oh_code_sign` | false | OH SDK 代码签名支持 |
| `code_signature_enable_xpm_mode` | 0 | XPM 模式 (0-5) |
| `code_signature_support_oh_release_app` | false | Release 应用支持 |
| `code_signature_support_app_allow_list` | false | 应用白名单支持 |
| `code_signature_screenlock_mgr_enable` | false | 屏幕锁管理器支持 |
| `code_signature_support_binary_enable` | false | Binary enable 支持 |
| `jit_code_sign_enable` | false | JIT 代码签名支持 |

### 2.2 JIT 签名自动启用条件

```gn
if (defined(target_cpu) && target_cpu == "arm64" &&
    code_signature_support_oh_code_sign && !is_emulator) {
  jit_code_sign_enable = true
}
```

## 3. 接口层 Targets

### 3.1 libcode_sign_attr_utils

**类型**：`ohos_static_library`

| 属性 | 值 |
|------|-----|
| Sources | `src/code_sign_attr_utils.c`, `src/ownerid_utils.cpp` |
| Defines | `SUPPORT_APP_ALLOW_LIST` (conditional) |
| External Deps | c_utils, hilog, init |
| Install Images | system |

**路径**：`interfaces/inner_api/code_sign_attr_utils/BUILD.gn`

### 3.2 libcode_sign_utils

**类型**：`ohos_shared_library`

| 属性 | 值 |
|------|-----|
| Sources | `code_sign_block.cpp`, `elf_code_sign_block_v1.cpp`, `data_size_report_adapter.cpp`, `file_helper.cpp`, `code_sign_enable_multi_task.cpp`, `code_sign_helper.cpp`, `code_sign_utils.cpp`, `code_sign_utils_in_c.cpp`, `stat_utils.cpp` |
| Defines | `SUPPORT_OH_CODE_SIGN`, `SUPPORT_PERMISSIVE_MODE` (conditional) |
| External Deps | ability_base (extractortool), appverify (libhapverify), c_utils, hilog, hisysevent, hitrace, openssl |
| Conditional Sources | `elf_code_sign_block.cpp` (if `code_signature_support_binary_enable`) |
| Install Images | system |

**路径**：`interfaces/inner_api/code_sign_utils/BUILD.gn`

### 3.3 liblocal_code_sign_sdk

**类型**：`ohos_shared_library`

| 属性 | 值 |
|------|-----|
| Sources | `cert_utils.cpp`, `huks_attest_verifier.cpp`, `openssl_utils.cpp`, `local_code_sign_client.cpp`, `local_code_sign_kit.cpp`, `local_code_sign_load_callback.cpp`, `local_code_sign_proxy.cpp` |
| Defines | `CODE_SIGNATURE_DEBUGGABLE` (root variant), `VERIFY_KEY_ATTEST_CERTCHAIN` (conditional) |
| External Deps | c_utils, hilog, hisysevent, huks, ipc, openssl, safwk, samgr |
| Install Images | system |

**路径**：`interfaces/inner_api/local_code_sign/BUILD.gn`

### 3.4 libjit_code_sign

**类型**：`ohos_shared_library`

| 属性 | 值 |
|------|-----|
| Sources | `src/jit_code_signer.cpp` |
| Configs | `private_jit_code_sign_configs` (`-march=armv8.4-a`, `ARCH_PAC_SUPPORT`) |
| Deps | `pac_sign_feature` |
| External Deps | bounds_checking_function (libsec_shared), hilog |
| Install Images | system |

**子 Target - pac_sign_feature**：
- Sources: `src/pac_sign_ctx.cpp`
- Configs: `private_jit_code_sign_configs` (conditional on `jit_code_sign_enable`)

**路径**：`interfaces/inner_api/jit_code_sign/BUILD.gn`

## 4. 服务层 Targets

### 4.1 liblocal_code_sign (SA)

**类型**：`ohos_shared_library` (sa 类型)

| 属性 | 值 |
|------|-----|
| Sources | `cert_utils.cpp`, `local_code_sign_service.cpp`, `local_code_sign_stub.cpp`, `local_sign_key.cpp`, `permission_utils.cpp` |
| Shlib Type | sa |
| External Deps | access_token, c_utils, eventhandler, fsverity-utils, hilog, hisysevent, hitrace, huks, init, ipc, openssl, safwk, samgr, bundle_framework |
| Install Images | system |

**路径**：`services/local_code_sign/BUILD.gn`

### 4.2 local_code_sign_configs

**类型**：`group`

| 属性 | 值 |
|------|-----|
| Deps | `:local_code_sign.cfg`, `:local_code_sign_sa_profile`, `:trusted_attest_root_ca` |

### 4.3 key_enable (Rust)

**类型**：`ohos_rust_executable`

| 属性 | 值 |
|------|-----|
| Sources | `src/main.rs` |
| Crate Type | bin |
| Crate Name | key_enable |
| External Deps | hilog_rust, hisysevent_rust, ylong_json |
| Conditional Deps | lazy-static, c_utils_rust, rust-openssl, rust_cxx |
| Rustenv | `code_signature_debuggable` (root/non-root), `support_openharmony_ca` (conditional) |

**路径**：`services/key_enable/BUILD.gn`

### 4.4 key_enable_lib (Rust FFI)

**类型**：`ohos_rust_shared_ffi`

| 属性 | 值 |
|------|-----|
| Sources | `src/lib.rs` |
| Crate Type | cdylib |
| Crate Name | key_enable |
| Dependencies | 与 `key_enable` 相同 |

## 5. 工具层 Targets

### 5.1 fsverity_sign_src_set

**类型**：`ohos_source_set`

| 属性 | 值 |
|------|-----|
| Sources | `fsverity_utils_helper.cpp`, `openssl_utils.cpp`, `pkcs7_data.cpp`, `pkcs7_generator.cpp`, `signer_info.cpp` |
| External Deps | bounds_checking_function, fsverity-utils, hilog, openssl, elfio |

**路径**：`utils/BUILD.gn`

## 6. 配置文件

### 6.1 local_code_sign.cfg

```json
{
    "services": [{
        "name": "local_code_sign",
        "path": ["/system/bin/sa_main", "/system/profile/local_code_sign.json"],
        "secon": "u:r:local_code_sign:s0",
        "permission": [
            "ohos.permission.ATTEST_KEY",
            "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED"
        ],
        "ondemand": true,
        "once": 1,
        "apl": "system_basic",
        "uid": "code_sign",
        "gid": "code_sign"
    }]
}
```

### 6.2 SA Profile

**文件**：`services/local_code_sign/sa_profile/3507.json`

**SA ID**: 3507

### 6.3 证书配置文件

| 文件 | 条件 |
|------|------|
| `config/OpenHarmony/trusted_attest_root_ca.cer` | `!code_signature_support_oh_code_sign` |
| `config/trusted_attest_root_ca.cer` | `code_signature_support_oh_code_sign` |
| `config/openharmony/release/trusted_cert_path.json` | `code_signature_support_oh_release_app` |
| `config/trusted_cert_path.json` | 默认 |
| `config/trusted_cert_path_mirror.json` | 默认 |

## 7. 编译产物清单

| 产物 | 类型 | 描述 |
|------|------|------|
| `libcode_sign_attr_utils.z.a` | 静态库 | 代码签名属性工具 |
| `libcode_sign_utils.z.so` | 共享库 | 代码签名工具 |
| `liblocal_code_sign_sdk.z.so` | 共享库 | 本地签名 SDK |
| `libjit_code_sign.z.so` | 共享库 | JIT 签名工具 |
| `liblocal_code_sign.z.so` | SA 共享库 | 本地签名服务 |
| `key_enable` | 可执行文件 | 密钥启用服务 (Rust) |
| `libkey_enable.z.so` | Rust FFI 库 | Rust/C++ 桥接 |
| `local_code_sign.cfg` | 配置文件 | SA 配置 |
| `3507.json` | SA Profile | SA ID 配置 |
| `trusted_attest_root_ca.cer` | 证书 | 根证书 |
| `trusted_cert_path.json` | 配置文件 | 证书路径 |

## 8. 安装路径

| 产物 | 安装路径 |
|------|----------|
| 共享库 | `/system/lib/` |
| SA 共享库 | `/system/lib/` |
| 可执行文件 | `/system/bin/` |
| 配置文件 | `/system/etc/init/` |
| 证书 | `/system/security/` |
| SA Profile | `/system/profile/` |

## 9. 安全编译选项

### 9.1 CFI (Control Flow Integrity)

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

### 9.2 Branch Protection

```gn
branch_protector_ret = "pac_ret"
```

### 9.3 Optimization Flags

```gn
cflags_cc = [
  "-Os",
  "-fno-asynchronous-unwind-tables",
  "-fno-unwind-tables",
]
```

## 10. 条件编译

### 10.1 代码签名支持

```gn
if (code_signature_support_oh_code_sign) {
  defines += [ "SUPPORT_OH_CODE_SIGN" ]
}
```

### 10.2 宽容模式

```gn
if (build_variant == "root" || code_signature_enable_xpm_mode == 0) {
  defines += [ "SUPPORT_PERMISSIVE_MODE" ]
}
```

### 10.3 Binary Enable

```gn
if (code_signature_support_binary_enable) {
  sources += [ "${code_signature_root_dir}/utils/src/elf_code_sign_block.cpp" ]
  defines += [ "SUPPORT_BINARY_ENABLE" ]
}
```

### 10.4 JIT 代码签名

```gn
if (jit_code_sign_enable) {
  configs += [ ":private_jit_code_sign_configs" ]
}
```
