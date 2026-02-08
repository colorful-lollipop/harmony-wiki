# GN 构建系统

## 1. 构建配置概述

### 1.1 根构建文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `BUILD.gn` | 项目根目录 | 根构建入口，定义组件组 |
| `cf.gni` | 项目根目录 | 编译配置参数 |
| `bundle.json` | 项目根目录 | 部件配置 |

### 1.2 构建参数 (cf.gni)

```gni
enable_coverage = false
```

### 1.3 根 BUILD.gn

**文件路径**: `BUILD.gn`

```gn
declare_args() {
  certificate_framework_enabled = true
}

group("certificate_framework_component") {
  if (os_level == "standard") {
    deps = [ "frameworks:certificate_framework_lib" ]
  }
}

group("certificate_framework_test") {
  testonly = true
  if (os_level == "standard") {
    deps = [ "test/unittest:cf_test" ]
  }
}

group("certificate_framework_fuzztest") {
  testonly = true
  if (os_level == "standard") {
    deps += [
      "test/fuzztest/cfcreate_fuzzer:fuzztest",
      "test/fuzztest/cfgetandcheck_fuzzer:fuzztest",
      "test/fuzztest/cfparam_fuzzer:fuzztest",
      "test/fuzztest/v1.0/x509certchain_fuzzer:fuzztest",
      "test/fuzztest/v1.0/x509certificate_fuzzer:fuzztest",
      "test/fuzztest/v1.0/x509crl_fuzzer:fuzztest",
      "test/fuzztest/v1.0/x509distinguishedname_fuzzer:fuzztest",
    ]
  }
}
```

## 2. 框架构建 (frameworks/BUILD.gn)

```gn
ohos_shared_library("certificate_framework_lib") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    boundary_sanitize = true
    debug = false
    integer_overflow = true
    ubsan = true
  }
  subsystem_name = "security"
  innerapi_tags = [ "platformsdk" ]
  part_name = "certificate_framework"
  public_configs = [ ":cert_framework_config" ]
  configs = [ "../../config/build:coverage_flag" ]
  
  sources = [ "life/cf_api.c" ]
  
  deps = [
    "ability:libcertificate_framework_ability",
    "adapter:libcertificate_framework_adapter",
    "common:libcertificate_framework_common_static",
    "cert:libcertificate_framework_cert_object",
    "extension:libcertificate_framework_extension_object",
    "v1.0:libcertificate_framework_vesion1",
    "attestation:libcertificate_attestation"
  ]
  
  external_deps = [
    "c_utils:utils",
    "crypto_framework:crypto_framework_lib",
    "hilog:libhilog",
  ]
  
  ldflags = [ "-Wl,--whole-archive" ]
  
  cflags = [
    "-DHILOG_ENABLE",
    "-Wall",
    "-Werror",
  ]
}
```

### 2.1 配置 (cert_framework_config)

```gn
config("cert_framework_config") {
  include_dirs = [
    "../../interfaces/inner_api/certificate",
    "../../interfaces/inner_api/common",
    "../../interfaces/inner_api/include",
    "../../interfaces/inner_api/attestation",
  ]
}
```

## 3. 子模块 Targets

### 3.1 ability 模块

```gn
# frameworks/ability/BUILD.gn
ohos_static_library("libcertificate_framework_ability") {
  sources = [
    "src/cf_ability.c",
  ]
  include_dirs = [ "inc" ]
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
  ]
}
```

### 3.2 adapter 模块

```gn
# frameworks/adapter/BUILD.gn
ohos_shared_library("libcertificate_framework_adapter") {
  deps = [
    "v1.0:certificate_openssl_plugin_lib",
    "v2.0:libcertificate_framework_adapter_openssl",
    "attestation:attestation_lib",
  ]
  external_deps = [
    "openssl:libcrypto_shared",
  ]
}
```

**v2.0 OpenSSL 适配**:

```gn
# frameworks/adapter/v2.0/BUILD.gn
ohos_shared_library("libcertificate_framework_adapter_openssl") {
  sources = [
    "src/cf_adapter_ability.c",
    "src/cf_adapter_cert_openssl.c",
    "src/cf_adapter_extension_openssl.c",
  ]
  include_dirs = [ "inc" ]
  external_deps = [
    "openssl:libcrypto_shared",
  ]
}
```

### 3.3 common 模块

```gn
# frameworks/common/BUILD.gn
ohos_static_library("libcertificate_framework_common_static") {
  sources = [
    "common.c",
  ]
  include_dirs = [ "v1.0/inc" ]
}
```

### 3.4 cert 模块

```gn
# frameworks/core/cert/BUILD.gn
ohos_static_library("libcertificate_framework_cert_object") {
  sources = [
    "src/cf_object_cert.c",
  ]
  include_dirs = [ "inc" ]
  deps = [
    "../:cert_framework_config",
    "../../ability:libcertificate_framework_ability",
  ]
}
```

### 3.5 extension 模块

```gn
# frameworks/core/extension/BUILD.gn
ohos_static_library("libcertificate_framework_extension_object") {
  sources = [
    "src/cf_object_extension.c",
  ]
  include_dirs = [ "inc" ]
  deps = [
    "../:cert_framework_config",
    "../../ability:libcertificate_framework_ability",
  ]
}
```

### 3.6 v1.0 模块

```gn
# frameworks/core/v1.0/BUILD.gn
ohos_static_library("libcertificate_framework_vesion1") {
  sources = [
    "certificate/x509_certificate.c",
    "certificate/x509_crl.c",
    "certificate/x509_cert_chain.c",
    "certificate/cert_chain_validator.c",
    "certificate/cert_crl_collection.c",
    "certificate/cert_cms_generator.c",
  ]
  include_dirs = [ "spi" ]
  deps = [
    "../:cert_framework_config",
    "../../ability:libcertificate_framework_ability",
  ]
}
```

### 3.7 attestation 模块

```gn
# frameworks/core/attestation/BUILD.gn
ohos_static_library("libcertificate_attestation") {
  sources = [
    "src/*.c",
  ]
  deps = [
    "../../ability:libcertificate_framework_ability",
  ]
}
```

## 4. JS/N-API 构建

```gn
# frameworks/js/napi/certificate/BUILD.gn
ohos_shared_library("certificate_napi") {
  sources = [ "src/*.cpp" ]
  include_dirs = [ "inc" ]
  deps = [
    "../../:cert_framework_config",
    "../../../ability:libcertificate_framework_ability",
    "../../adapter:libcertificate_framework_adapter",
    "../../core/cert:libcertificate_framework_cert_object",
    "../../core/v1.0:libcertificate_framework_vesion1",
  ]
  external_deps = [
    "napi:napi",
  ]
}
```

## 5. 构建产物

### 5.1 编译产物清单

| 产物类型 | 产物名 | 输出路径 | 说明 |
|---------|--------|---------|------|
| .so | libcertificate_framework_lib.so | system/lib64/ | 主框架库 |
| .so | libcertificate_framework_adapter_openssl.so | system/lib64/ | OpenSSL 适配器 |
| .so | certificate_napi.so | system/lib64/ | N-API 接口 |
| .a | libcertificate_framework_ability.a | system/lib64/ | 静态库 |
| .a | libcertificate_framework_cert_object.a | system/lib64/ | 静态库 |
| .a | libcertificate_framework_extension_object.a | system/lib64/ | 静态库 |
| .a | libcertificate_framework_vesion1.a | system/lib64/ | 静态库 |

### 5.2 依赖关系

```
certificate_napi.so
    │
    ├──► libcertificate_framework_lib.so
    │         │
    │         ├──► libcertificate_framework_ability.a
    │         ├──► libcertificate_framework_adapter_openssl.so
    │         │         │
    │         │         └──► libcrypto.so (OpenSSL)
    │         ├──► libcertificate_framework_cert_object.a
    │         ├──► libcertificate_framework_extension_object.a
    │         ├──► libcertificate_framework_vesion1.a
    │         └──► libcertificate_attestation.a
    │
    └──► libnapi.so
```

## 6. 构建命令

### 6.1 单独编译证书框架

```bash
./build.sh --product-name rk3568 --ccache --build-target certificate_framework
```

### 6.2 编译参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `--product-name` | 产品名称 | rk3568, hi3516d |
| `--ccache` | 启用编译缓存 | - |
| `--build-target` | 编译目标 | certificate_framework |

## 7. 内部 Kits 配置 (bundle.json)

```json
"inner_kits": [
  {
    "name": "//base/security/certificate_framework/frameworks/core:certificate_framework_core",
    "header": {
      "header_files": [
        "certificate/cert_chain_validator.h",
        "certificate/certificate.h",
        "certificate/crl.h",
        "certificate/cert_crl_collection.h",
        "certificate/x509_cert_match_parameters.h",
        "certificate/x509_crl_match_parameters.h",
        "certificate/x509_certificate.h",
        "certificate/x509_cert_chain.h",
        "certificate/x509_distinguished_name.h",
        "certificate/x509_trust_anchor.h",
        "certificate/x509_cert_chain_validate_params.h",
        "certificate/x509_cert_chain_validate_result.h",
        "certificate/x509_crl_entry.h",
        "certificate/x509_crl.h",
        "certificate/cert_crl_common.h",
        "common/cf_blob.h",
        "common/cf_object_base.h",
        "common/cf_result.h",
        "include/cf_api.h",
        "include/cf_param.h",
        "include/cf_type.h"
      ],
      "header_base": "//base/security/certificate_framework/interfaces/inner_api"
    }
  },
  {
    "name": "//base/security/certificate_framework/frameworks/cj:cj_cert_ffi"
  }
]
```

## 8. 安全编译选项

### 8.1 Sanitizer 配置

```gn
sanitize = {
  cfi = true              // 控制流完整性
  cfi_cross_dso = true    // 跨 DSO CFI
  boundary_sanitize = true // 边界检查
  debug = false
  integer_overflow = true // 整数溢出检查
  ubsan = true            // 未定义行为检查
}
```

### 8.2 编译警告

```gn
cflags = [
  "-DHILOG_ENABLE",
  "-Wall",                // 启用所有警告
  "-Werror",              // 警告视为错误
]
```

## 9. 相关文件

| 文件 | 路径 | 说明 |
|------|------|------|
| BUILD.gn | 项目根目录 | 根构建入口 |
| cf.gni | 项目根目录 | 构建参数 |
| bundle.json | 项目根目录 | 部件配置 |
| frameworks/BUILD.gn | frameworks/ | 主框架构建 |
| frameworks/adapter/BUILD.gn | frameworks/adapter/ | 适配器构建 |
| frameworks/js/napi/certificate/BUILD.gn | frameworks/js/napi/certificate/ | N-API 构建 |
