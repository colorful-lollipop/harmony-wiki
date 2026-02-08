# Hilog Lite GN 构建 Targets

本文档梳理 hilog_lite 组件的所有 GN 构建 Target、依赖关系和配置选项。

## 构建配置入口

### bundle.json 定义

```json
{
  "build": {
    "sub_component": [
      "//base/hiviewdfx/hilog_lite/frameworks/mini:hilog_lite",
      "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_static",
      "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
      "//base/hiviewdfx/hilog_lite/services/apphilogcat:apphilogcat",
      "//base/hiviewdfx/hilog_lite/frameworks/js:ace_hilog_kits",
      "//base/hiviewdfx/hilog_lite/test:hilog_lite_test"
    ],
    "inner_kits": [
      {
        "name": "//base/hiviewdfx/hilog_lite/frameworks/mini:hilog_lite",
        "header": {
          "header_files": ["hiview_log.h", "log.h"],
          "header_base": "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite"
        }
      },
      {
        "name": "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
        "header": {
          "header_files": ["hilog_cp.h", "hilog_trace.h", "hiview_log.h", "log.h"],
          "header_base": "//base/hiviewdfx/hilog_lite/interfaces/native/innerkits/hilog"
        }
      }
    ]
  }
}
```

> 证据来源: bundle.json:59-91

---

## 轻量系统构建 Targets

### frameworks/mini/BUILD.gn

#### hilog_lite_static

```gn
static_library("hilog_lite_static") {
  sources = [
    "hilog_lite.c",
    "hiview_log.c",
    "hiview_log_limit.c",
    "hiview_output_log.c",
  ]
  defines = [
    "HIVIEW_LOG_FILE_SIZE=$hilog_lite_file_size",
    "LOG_LIMIT_DEFAULT=$hilog_lite_limit_level_default",
    "LOG_STATIC_CACHE_SIZE=$hilog_lite_log_static_cache_size",
    "HIVIEW_HILOG_FILE_BUF_SIZE=$hilog_lite_hiview_hilog_file_buf_size",
  ]
  public_configs = [
    "//base/hiviewdfx/hiview_lite:hiview_lite_config",
    ":hilog_lite_config",
  ]
  deps = [ "//base/hiviewdfx/hiview_lite" ]
}
```

**依赖**:
- `//base/hiviewdfx/hiview_lite`

> 证据来源: frameworks/mini/BUILD.gn:36-68

#### hilog_lite (Group)

```gn
group("hilog_lite") {
  if (ohos_kernel_type == "liteos_m") {
    if (hilog_lite_customize_implementation) {
      public_configs = [
        "//base/hiviewdfx/hiview_lite:hiview_lite_config",
        ":hilog_lite_config",
      ]
    } else {
      public_deps = [ ":hilog_lite_static" ]
    }
  }
}
```

**逻辑**:
- LiteOS-M 内核：使用静态库
- 其他内核：空组

#### hilog_lite_ndk (NDK)

```gn
ndk_lib("hilog_lite_ndk") {
  deps = [ ":hilog_lite" ]
  head_files = [ "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite" ]
}
```

---

## 小型系统构建 Targets

### frameworks/featured/BUILD.gn

#### hilog_static

```gn
lite_library("hilog_static") {
  target_type = "static_library"
  sources = [
    "hilog.cpp",
    "hiview_log.c",
  ]
  public_configs = [ ":hilog_config" ]
  public_deps = [ "//third_party/bounds_checking_function:libsec_static" ]
}
```

**条件**:
- `if (!hilog_lite_disable_hilog_static)`

> 证据来源: frameworks/featured/BUILD.gn:44-52

#### hilog_shared

```gn
lite_library("hilog_shared") {
  target_type = "shared_library"
  sources = [
    "hilog.cpp",
    "hiview_log.c",
  ]
  public_configs = [ ":hilog_config" ]
  public_deps = [ "//third_party/bounds_checking_function:libsec_shared" ]
}
```

**条件**:
- `if (ohos_kernel_type != "liteos_m")`

> 证据来源: frameworks/featured/BUILD.gn:55-64

#### hilog_ndk (NDK)

```gn
ndk_lib("hilog_ndk") {
  lib_extension = ".so"
  deps = [ ":hilog_shared" ]
  head_files = [
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog",
    "//third_party/bounds_checking_function/include",
  ]
}
```

---

## 服务构建 Targets

### services/hilogcat/BUILD.gn

```gn
lite_component("hilogcat") {
  target_type = "executable"
  features = [ ":hilogcat_static" ]
}

static_library("hilogcat_static") {
  sources = [ "hiview_logcat.c" ]
  include_dirs = [ "//third_party/bounds_checking_function/include" ]
  deps = [
    "//base/hiviewdfx/hilog_lite/command:hilog_command_static",
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}
```

**依赖**:
- `//base/hiviewdfx/hilog_lite/command:hilog_command_static`
- `//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared`
- `//third_party/bounds_checking_function:libsec_shared`

> 证据来源: services/hilogcat/BUILD.gn:16-29

### services/apphilogcat/BUILD.gn

```gn
lite_component("apphilogcat") {
  target_type = "executable"
  features = [ ":apphilogcat_static" ]
}

static_library("apphilogcat_static") {
  sources = [ "hiview_applogcat.c" ]
  include_dirs = [ "//third_party/bounds_checking_function/include" ]
  deps = [
    "//base/hiviewdfx/hilog_lite/command:hilog_command_static",
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
  public_configs = [ ":apphilogcat_config" ]

  if (hilog_lite_enable_hilogcat_build) {
    deps += [ "//base/hiviewdfx/hilog_lite/services/hilogcat:hilogcat" ]
  }
}
```

**条件**:
- `if (ohos_kernel_type != "liteos_m")`

> 证据来源: services/apphilogcat/BUILD.gn:52-70

---

## 命令构建 Targets

### command/BUILD.gn

```gn
lite_library("hilog_command_static") {
  target_type = "static_library"
  sources = [ "hilog_command.c" ]
  public_configs = [
    ":hilog_command_config",
    "//base/hiviewdfx/hilog_lite/services/apphilogcat:apphilogcat_config",
  ]
  deps = [
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}

lite_library("hilog_command_shared") {
  target_type = "shared_library"
  sources = [ "hilog_command.c" ]
  public_configs = [
    ":hilog_command_config",
    "//base/hiviewdfx/hilog_lite/services/apphilogcat:apphilogcat_config",
  ]
  deps = [
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}
```

> 证据来源: command/BUILD.gn:20-44

---

## JS 构建 Targets

### frameworks/js/BUILD.gn

```gn
if (ohos_kernel_type != "liteos_m" && !hilog_lite_disable_js_feature) {
  lite_component("ace_hilog_kits") {
    features = [ "builtin:ace_kit_hilog" ]
  }
} else {
  group("ace_hilog_kits") {
  }
}
```

**条件**:
- 非 LiteOS-M 内核
- 且未禁用 JS 功能

> 证据来源: frameworks/js/BUILD.gn:21-28

---

## 配置参数汇总

### 框架层配置 (frameworks/mini/BUILD.gn)

| 参数 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `hilog_lite_file_size` | int | 8192 | 日志文件大小 |
| `hilog_lite_disable_cache` | bool | false | 禁用日志缓存 |
| `hilog_lite_limit_level_default` | int | 30 | 默认限流级别 |
| `hilog_lite_disable_print_limit` | bool | false | 禁用打印限流 |
| `hilog_lite_log_static_cache_size` | int | 1024 | 静态缓存大小 |
| `hilog_lite_hiview_hilog_file_buf_size` | int | 512 | 文件缓冲区大小 |
| `hilog_lite_disable_core_init` | bool | false | 禁用核心初始化 |
| `hilog_lite_customize_implementation` | bool | false | 自定义实现 |

> 证据来源: frameworks/mini/BUILD.gn:16-25

### 框架层配置 (frameworks/featured/BUILD.gn)

| 参数 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `hilog_lite_disable_privacy_feature` | bool | false | 禁用隐私功能 |
| `hilog_lite_disable_hilog_static` | bool | false | 禁用静态库 |

> 证据来源: frameworks/featured/BUILD.gn:16-19

### 服务层配置 (services/apphilogcat/BUILD.gn)

| 参数 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `hilog_lite_hilog_file_size` | int | 1024000 | 日志文件大小 |
| `hilog_lite_enable_apphilogcat_init_release` | bool | false | Release 版初始化 |
| `hilog_lite_enable_apphilogcat_init_debug` | bool | true | Debug 版初始化 |
| `hilog_lite_apphilogcat_log_level_release` | int | 5 | Release 日志级别 |
| `hilog_lite_apphilogcat_log_level_debug` | int | 3 | Debug 日志级别 |
| `hilog_lite_enable_hilogcat_build` | bool | true | 启用 hilogcat 构建 |
| `hilog_lite_apphilogcat_log_dir` | string | "/storage/data/log" | 日志目录 |

> 证据来源: services/apphilogcat/BUILD.gn:15-24

---

## 依赖关系图

```
frameworks/mini:hilog_lite
├── hiview_lite
└── samgr_lite (via config)

frameworks/featured:hilog_static
└── bounds_checking_function:libsec_static

frameworks/featured:hilog_shared
└── bounds_checking_function:libsec_shared

services/hilogcat:hilogcat
├── command:hilog_command_static
├── featured:hilog_shared
└── bounds_checking_function:libsec_shared

services/apphilogcat:apphilogcat
├── command:hilog_command_static
├── featured:hilog_shared
├── bounds_checking_function:libsec_shared
└── hilogcat:hilogcat (optional)

command:hilog_command_static
├── featured:hilog_shared
└── bounds_checking_function:libsec_shared

frameworks/js:ace_hilog_kits
└── builtin:ace_kit_hilog
```

---

## 相关文档

- [概览](01_Overview.md)
- [编译产物](06_Build_Outputs.md)
- [配置项](appendix/Config_Flags.md)
