# GN Targets 梳理

本文档详细梳理 OpenHarmony build 系统中的 GN Targets，包括模板定义、参数说明和使用示例。

## 模板概览

OpenHarmony build 系统提供 44+ 个 GN 模板，支持多种编程语言和构建目标。

### 模板分类

| 分类 | 模板数量 | 主要文件 |
|------|---------|---------|
| C/C++ | 10 | `templates/cxx/cxx.gni`, `prebuilt.gni` |
| Rust | 12 | `templates/rust/*.gni` |
| 仓颉 | 9 | `templates/cangjie/*.gni` |
| 应用 | 8 | `ohos/app/app.gni` |
| 其他 | 8 | `abc`, `bpf`, `idl`, `kernel`, `sa` 等 |

## C/C++ 模板

### 1. ohos_executable

**文件**: `//build/templates/cxx/cxx.gni:38`

**功能**: 定义可执行文件目标

**必需参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `part_name` | string | 所属部件名称 |

**常用参数**:
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `sources` | list | [] | 源文件列表 |
| `deps` | list | [] | 内部依赖 |
| `external_deps` | list | [] | 外部依赖 (格式: "part:module") |
| `configs` | list | [] | 编译配置 |
| `cflags` | list | [] | C 编译器标志 |
| `cflags_cc` | list | [] | C++ 编译器标志 |
| `ldflags` | list | [] | 链接器标志 |
| `include_dirs` | list | [] | 头文件搜索路径 |
| `static_link` | bool | false | 是否静态链接 |
| `use_exceptions` | bool | false | 启用异常 |
| `use_rtti` | bool | false | 启用 RTTI |
| `install_enable` | bool | false | 是否安装 |
| `module_install_dir` | string | "" | 安装目录 |

**示例**:
```gn
import("//build/ohos.gni")

ohos_executable("my_app") {
  sources = [ "main.cpp", "utils.cpp" ]
  deps = [ ":my_lib" ]
  external_deps = [ "hilog:libhilog" ]
  include_dirs = [ "//include" ]
  install_enable = true
  module_install_dir = "bin"
  part_name = "my_part"
}
```

### 2. ohos_shared_library

**文件**: `//build/templates/cxx/cxx.gni:574`

**功能**: 定义共享库 (.so)

**特有参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `innerapi_tags` | list | 内部 API 标签: ndk, llndk, chipsetsdk, platformsdk, passthrough |
| `shlib_type` | string | 共享库类型: sa, hdi, napi, ani, innerapi |
| `version_script` | string | 版本脚本路径 |

**示例**:
```gn
ohos_shared_library("my_lib") {
  sources = [ "my_lib.cpp" ]
  innerapi_tags = [ "ndk" ]
  shlib_type = "napi"
  part_name = "my_part"
}
```

### 3. ohos_static_library

**文件**: `//build/templates/cxx/cxx.gni:1438`

**功能**: 定义静态库 (.a)

**特点**:
- 默认不安装到系统目录
- 用于内部链接

**示例**:
```gn
ohos_static_library("my_static") {
  sources = [ "static.cpp" ]
  deps = [ "//other:lib" ]
  part_name = "my_part"
}
```

### 4. ohos_source_set

**文件**: `//build/templates/cxx/cxx.gni:1773`

**功能**: 定义源码集合（不生成库文件）

**用途**: 将源码分组，供其他目标依赖

**示例**:
```gn
ohos_source_set("common_sources") {
  sources = [
    "utils.cpp",
    "helper.cpp",
  ]
  configs = [ ":common_config" ]
  part_name = "my_part"
}

ohos_executable("my_app") {
  deps = [ ":common_sources" ]
  part_name = "my_part"
}
```

### 5. ohos_prebuilt_etc

**文件**: `//build/templates/cxx/prebuilt.gni:425`

**功能**: 安装预构建文件到系统目录

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `source` | string | 源文件路径 |
| `module_install_dir` | string | 安装目录 |

**示例**:
```gn
ohos_prebuilt_etc("config_file") {
  source = "my_config.json"
  module_install_dir = "etc/my_app"
  part_name = "my_part"
}
```

### 6. ohos_prebuilt_shared_library

**文件**: `//build/templates/cxx/prebuilt.gni:123`

**功能**: 预构建共享库

**示例**:
```gn
ohos_prebuilt_shared_library("prebuilt_lib") {
  source = "libthird_party.so"
  module_install_dir = "lib"
  part_name = "my_part"
}
```

## Rust 模板

### 1. ohos_rust_executable

**文件**: `//build/templates/rust/rust_template.gni:263`

**功能**: Rust 可执行文件

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `crate_name` | string | crate 名称 |
| `sources` | list | Rust 源文件 |
| `features` | list | 特性标志 |
| `rustflags` | list | Rust 编译器标志 |
| `rustc_lints` | string | 代码检查级别: openharmony, vendor, none, allWarning |

**示例**:
```gn
ohos_rust_executable("my_rust_app") {
  sources = [ "src/main.rs" ]
  crate_name = "my_rust_app"
  rustc_lints = "openharmony"
  part_name = "my_part"
}
```

### 2. ohos_rust_shared_library

**文件**: `//build/templates/rust/rust_template.gni:356`

**功能**: Rust 动态库 (dylib)

**示例**:
```gn
ohos_rust_shared_library("my_rust_lib") {
  sources = [ "src/lib.rs" ]
  crate_name = "my_rust_lib"
  innerapi_tags = [ "platformsdk" ]
  part_name = "my_part"
}
```

### 3. ohos_rust_static_library

**文件**: `//build/templates/rust/rust_template.gni:449`

**功能**: Rust 静态库 (rlib)

**示例**:
```gn
ohos_rust_static_library("my_rust_static") {
  sources = [ "src/lib.rs" ]
  crate_name = "my_rust_static"
  part_name = "my_part"
}
```

### 4. ohos_rust_shared_ffi

**文件**: `//build/templates/rust/rust_template.gni:538`

**功能**: Rust FFI 动态库 (cdylib)，用于 C 兼容

**示例**:
```gn
ohos_rust_shared_ffi("my_ffi_lib") {
  sources = [ "src/lib.rs" ]
  crate_name = "my_ffi_lib"
  part_name = "my_part"
}
```

### 5. rust_bindgen

**文件**: `//build/templates/rust/rust_bindgen.gni:16`

**功能**: 从 C 头文件生成 Rust 绑定

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `header` | string | C 头文件路径 |
| `output` | string | 输出 Rust 文件 |

**示例**:
```gn
rust_bindgen("my_bindings") {
  header = "//include/my_header.h"
  output = "src/bindings.rs"
}
```

### 6. rust_cxx

**文件**: `//build/templates/rust/rust_cxx.gni:14`

**功能**: Rust-C++ 互操作桥接

**示例**:
```gn
rust_cxx("my_bridge") {
  sources = [ "src/lib.rs" ]
  part_name = "my_part"
}
```

## 仓颉语言模板

### 1. ohos_cangjie_shared_library

**文件**: `//build/templates/cangjie/cjc.gni:46`

**功能**: 仓颉动态库

**示例**:
```gn
ohos_cangjie_shared_library("my_cj_lib") {
  sources = [ "lib.cj" ]
  output_type = "dylib"
  part_name = "my_part"
}
```

### 2. ohos_cangjie_static_library

**文件**: `//build/templates/cangjie/cjc.gni:102`

**功能**: 仓颉静态库

**示例**:
```gn
ohos_cangjie_static_library("my_cj_static") {
  sources = [ "lib.cj" ]
  output_type = "staticlib"
  part_name = "my_part"
}
```

## 应用模板

### 1. ohos_hap

**文件**: `//build/ohos/app/app.gni`

**功能**: HAP (HarmonyOS Ability Package) 构建

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `hap_profile` | string | HAP 配置文件 |
| `js_assets` | list | JS 资源 |
| `ets_assets` | list | ETS 资源 |
| `resources` | list | 资源文件 |

**示例**:
```gn
ohos_hap("my_app") {
  hap_profile = "entry/src/main/module.json"
  js_assets = [ "entry/src/main/js" ]
  ets_assets = [ "entry/src/main/ets" ]
  resources = [ "entry/src/main/resources" ]
  part_name = "my_part"
}
```

### 2. ohos_abc

**文件**: `//build/templates/abc/ohos_abc.gni:31`

**功能**: ArkTS/TypeScript 编译为 Ark Bytecode

**示例**:
```gn
ohos_abc("my_abc") {
  sources = [ "src/index.ts" ]
  output = "my_abc.abc"
  part_name = "my_part"
}
```

## 其他模板

### 1. ohos_sa_profile

**文件**: `//build/ohos/sa_profile/sa_profile.gni`

**功能**: System Ability 配置文件处理

**示例**:
```gn
ohos_sa_profile("my_sa") {
  sources = [ "my_sa.xml" ]
  part_name = "my_part"
}
```

### 2. ohos_bpf

**文件**: `//build/templates/bpf/ohos_bpf.gni:38`

**功能**: eBPF 程序编译

**示例**:
```gn
ohos_bpf("my_bpf") {
  sources = [ "program.c" ]
  part_name = "my_part"
}
```

### 3. ohos_idl

**文件**: `//build/templates/idl/ohos_idl.gni:32`

**功能**: IDL 接口定义生成

**示例**:
```gn
ohos_idl("my_interface") {
  sources = [ "IMyInterface.idl" ]
  part_name = "my_part"
}
```

### 4. ohos_copy

**文件**: `//build/templates/common/copy.gni:20`

**功能**: 文件复制

**示例**:
```gn
ohos_copy("copy_assets") {
  sources = [ "data/file.txt" ]
  outputs = [ "$target_out_dir/file.txt" ]
}
```

## 关键参数详解

### part_name

所有 OpenHarmony 模板都需要 `part_name` 参数，用于：
- 标识模块所属部件
- 生成部件安装信息
- 依赖检查

```gn
part_name = "my_part"  # 对应 bundle.json 中的部件名
```

### innerapi_tags

用于标记共享库的 API 级别：

| 标签 | 说明 |
|------|------|
| `ndk` | NDK 接口 |
| `llndk` | Low Level NDK |
| `chipsetsdk` | 芯片 SDK |
| `platformsdk` | 平台 SDK |
| `passthrough` | 直通库 |

```gn
innerapi_tags = [ "ndk", "platformsdk" ]
```

### external_deps

跨部件依赖格式：

```gn
external_deps = [
  "part_name:module_name",  # 部件名:模块名
  "hilog:libhilog",
  "ipc:ipc_core",
]
```

### install_images

指定安装到哪个分区镜像：

```gn
install_images = [ "system", "vendor" ]
```

## Target 类型汇总

| Target 类型 | 输出 | 安装 | 典型用途 |
|------------|------|------|---------|
| `ohos_executable` | 可执行文件 | 可选 | 应用程序、工具 |
| `ohos_shared_library` | .so | 是 | 共享库 |
| `ohos_static_library` | .a | 否 | 内部静态库 |
| `ohos_source_set` | - | 否 | 源码分组 |
| `ohos_rust_executable` | 可执行文件 | 可选 | Rust 应用 |
| `ohos_rust_shared_library` | .so | 是 | Rust 动态库 |
| `ohos_rust_static_library` | .rlib | 否 | Rust 静态库 |
| `ohos_hap` | .hap | 是 | HarmonyOS 应用 |
| `ohos_abc` | .abc | 是 | Ark 字节码 |
| `ohos_prebuilt_etc` | 任意 | 是 | 配置文件 |

## 常用构建目标

### 查询所有目标

```bash
gn ls out/my_build
```

### 构建特定目标

```bash
ninja -C out/my_build //path/to:target_name
```

### 查看目标详情

```bash
gn desc out/my_build //path/to:target_name
```

---

*文档生成时间: 2025-02-06*
