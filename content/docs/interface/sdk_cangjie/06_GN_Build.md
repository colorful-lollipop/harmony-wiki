# GN 构建系统

## 概述

Cangjie SDK 使用 GN (Generate Ninja) 作为构建系统，通过 `BUILD.gn` 和 `.gni` 文件定义构建规则。

**关键文件**:
- `BUILD.gn`: 根构建入口，定义 SDK 构建目标
- `sdk_cangjie.gni`: 构建模板定义，包含自定义 GN 模板

> **证据**: `BUILD.gn` 和 `sdk_cangjie.gni`

## 根构建文件 (BUILD.gn)

### 文件结构

```gn
# 导入构建模板
import("//build/templates/cangjie/cjc.gni")
import("//build/templates/cangjie/cjc_toolchain.gni")
import("sdk_cangjie.gni")

# API 库依赖列表
sdk_packages_api_libs = [
  "cangjie_ark_interop:copy_cangjie_ark_interop_libs",
  "arkui_cangjie_wrapper:copy_sdk_arkui_cangjie_libs",
  # ... 共 26 个组件
]

# Header 复制目标
cj_ohos_sdk_header_copy("sdk_header_ohos_aarch64_api") { ... }
cj_ohos_sdk_header_copy("sdk_header_ohos_aarch64_kit") { ... }

# Lib 复制目标
cj_ohos_sdk_shared_lib("sdk_ohos_aarch64_libs") { ... }
cj_ohos_sdk_shared_lib("sdk_ohos_x86_64_libs") { ... }

# Macro 构建目标
cj_ohos_sdk_macro("sdk_windows_x86_64_macro") { ... }
cj_ohos_sdk_macro("sdk_linux_x86_64_macro") { ... }
```

> **证据**: `BUILD.gn` 第 14-65 行

## SDK 组件依赖列表

### 26 个组件 (sdk_packages_api_libs)

| 序号 | 组件 | Target |
|------|------|--------|
| 1 | cangjie_ark_interop | copy_cangjie_ark_interop_libs |
| 2 | hiviewdfx_cangjie_wrapper | copy_sdk_hiviewdfx_cangjie_libs |
| 3 | arkui_cangjie_wrapper | copy_sdk_arkui_cangjie_libs |
| 4 | global_cangjie_wrapper | copy_sdk_global_cangjie_libs |
| 5 | arkweb_cangjie_wrapper | copy_sdk_arkweb_cangjie_libs |
| 6 | applications_cangjie_wrapper | copy_sdk_applications_cangjie_libs |
| 7 | window_cangjie_wrapper | copy_sdk_window_cangjie_libs |
| 8 | time_cangjie_wrapper | copy_sdk_time_cangjie_libs |
| 9 | powermgr_cangjie_wrapper | copy_sdk_powermgr_cangjie_libs |
| 10 | location_cangjie_wrapper | copy_sdk_location_cangjie_libs |
| 11 | multimedia_cangjie_wrapper | copy_sdk_multimedia_cangjie_libs |
| 12 | ability_cangjie_wrapper | copy_sdk_ability_cangjie_libs |
| 13 | testfwk_cangjie_wrapper | copy_sdk_testfwk_cangjie_libs |
| 14 | telephony_cangjie_wrapper | copy_sdk_telephony_cangjie_libs |
| 15 | distributeddatamgr_cangjie_wrapper | copy_sdk_distributeddatamgr_cangjie_libs |
| 16 | request_cangjie_wrapper | copy_sdk_request_cangjie_libs |
| 17 | netmanager_cangjie_wrapper | copy_sdk_netmanager_cangjie_libs |
| 18 | security_cangjie_wrapper | copy_sdk_security_cangjie_libs |
| 19 | accesscontrol_cangjie_wrapper | copy_sdk_accesscontrol_cangjie_libs |
| 20 | connectivity_cangjie_wrapper | copy_sdk_connectivity_cangjie_libs |
| 21 | communication_cangjie_wrapper | copy_sdk_communication_cangjie_libs |
| 22 | graphic_cangjie_wrapper | copy_sdk_graphic_cangjie_libs |
| 23 | filemanagement_cangjie_wrapper | copy_sdk_filemanagement_cangjie_libs |
| 24 | sensors_cangjie_wrapper | copy_sdk_sensors_cangjie_libs |
| 25 | bundlemanager_cangjie_wrapper | copy_sdk_bundlemanager_cangjie_libs |
| 26 | startup_cangjie_wrapper | copy_sdk_startup_cangjie_libs |
| 27 | notification_cangjie_wrapper | copy_sdk_notification_cangjie_libs |

> **证据**: `BUILD.gn` 第 18-65 行

## 构建模板定义 (sdk_cangjie.gni)

### 模板列表

| 模板 | 文件 | 用途 |
|------|------|------|
| `cj_ohos_sdk_header_copy` | sdk_cangjie.gni:22-52 | 复制 Cangjie 头文件 |
| `cj_ohos_sdk_shared_lib` | sdk_cangjie.gni:55-83 | 构建 SDK 共享库 |
| `cj_ohos_sdk_macro` | sdk_cangjie.gni:86-119 | 构建宏库 |
| `cj_ohos_sdk_copy` | sdk_cangjie.gni:122-143 | 通用复制 |

### cj_ohos_sdk_header_copy

```gn
template("cj_ohos_sdk_header_copy") {
  action_with_pydeps(target_name) {
    script = "//interface/sdk_cangjie/build-tools/script/copy_cangjie_headers.py"
    args = [
      "--input", rebase_path(source_dir, root_build_dir),
      "--output", rebase_path("${target_out_dir}/${out_relative_dir}", root_build_dir),
    ]
    # 可选: 排除目录
    if (defined(exculde_dirs)) {
      args += ["--exclude-dirs", string_join(",", exculde_dirs)]
    }
  }
}
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `source_dir` | String | 源目录 |
| `out_relative_dir` | String | 输出相对路径 |
| `exculde_dirs` | String[] | 排除目录（可选） |

> **证据**: `sdk_cangjie.gni` 第 22-52 行

### cj_ohos_sdk_shared_lib

```gn
template("cj_ohos_sdk_shared_lib") {
  action_with_pydeps(target_name) {
    script = "//interface/sdk_cangjie/build-tools/script/process_libs.py"
    external_deps = invoker.external_deps
    deps = [ "build-tools/lib/mocks/mock:ohos.mock(${toolchain})" ]
    args = [
      "--copy-cjo-dir", rebase_path(_source_dir, root_build_dir),
      "--mock", rebase_path(_mock_so_target, root_build_dir),
      "--output-dir", rebase_path(_output_dir, root_build_dir),
    ]
  }
}
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `platform` | String | 平台标识 |
| `toolchain` | String | 工具链标识 |
| `external_deps` | String[] | 外部依赖 |

> **证据**: `sdk_cangjie.gni` 第 55-83 行

### cj_ohos_sdk_macro

```gn
template("cj_ohos_sdk_macro") {
  ohos_copy(target_name) {
    forward_variables_from(invoker, ["toolchain", "module_name", "dyn_extension"])
    sources = []
    external_deps = []
    foreach(item, cangjie_sdk_macro_libs) {
      external_deps += ["${item}(${toolchain})", "${item}_cjo(${toolchain})"]
    }
    outputs = [ "${target_out_dir}/cangjie_sdk/api/macro/${module_name}/{{source_file_part}}" ]
  }
}
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `toolchain` | String | 工具链 |
| `module_name` | String | 模块名 |
| `dyn_extension` | String | 动态库扩展名 |

**宏库依赖**:
```gn
cangjie_sdk_macro_libs = [
  "arkui_cangjie_wrapper:ohos.arkui.state_macro_manage",
  "cangjie_ark_interop:ohos.ark_interop_macro",
]
```

> **证据**: `sdk_cangjie.gni` 第 86-119 行

## 构建产物

### 主要构建目标

| Target | 类型 | 输出 | 平台 |
|--------|------|------|------|
| `sdk_header_ohos_aarch64` | Group | 头文件 | ohos-aarch64 |
| `sdk_ohos_aarch64_libs` | Action | 动态库 + cjo | ohos-aarch64 |
| `sdk_ohos_x86_64_libs` | Action | 动态库 + cjo | ohos-x86_64 |
| `sdk_windows_x86_64_macro` | OhosCopy | 宏库 (.dll) | Windows |
| `sdk_linux_x86_64_macro` | OhosCopy | 宏库 (.so) | Linux |
| `sdk_darwin_x86_64_macro` | OhosCopy | 宏库 (.dylib) | Mac-x64 |
| `sdk_darwin_aarch64_macro` | OhosCopy | 宏库 (.dylib) | Mac-arm64 |

> **证据**: `BUILD.gn` 第 88-170 行

### 构建条件

```gn
# 条件编译: Previewer
if (sdk_build_cangjie_previewer == true) {
  group("sdk_cangjie_previewer_windows") { ... }
  group("sdk_cangjie_previewer_mac") { ... }
}

# 条件编译: ARM 平台
if (sdk_build_cangjie_ohos_arm == true) {
  cj_ohos_sdk_shared_lib("sdk_ohos_arm_libs") { ... }
}

# 条件编译: Mac 平台
if (host_os == "mac") {
  if (host_cpu == "arm64") {
    cj_ohos_sdk_macro("sdk_darwin_aarch64_macro") { ... }
  } else if (host_cpu == "x64") {
    cj_ohos_sdk_macro("sdk_darwin_x86_64_macro") { ... }
  }
}
```

> **证据**: `BUILD.gn` 第 67-86 行、第 118-170 行

## 构建脚本

### 脚本列表

| 脚本 | 位置 | 用途 |
|------|------|------|
| `copy_cangjie_headers.py` | build-tools/script/ | 复制头文件 |
| `copy_and_prue.py` | build-tools/script/ | 复制编译器工具链 |
| `process_libs.py` | build-tools/script/ | 处理库文件 |

### copy_cangjie_headers.py

```python
# 用法
python copy_cangjie_headers.py \
  --input <source_dir> \
  --output <output_dir> \
  --depfile <depfile> \
  --stamp <stamp_file> \
  [--exclude-dirs <dirs>]
```

### process_libs.py

```python
# 用法
python process_libs.py \
  --copy-cjo-dir <cjo_dir> \
  --mock <mock_so_path> \
  --output-dir <output_dir>
```

> **证据**: `sdk_cangjie.gni` 第 32 行、第 65 行

## 构建命令

### 标准构建

```bash
./build.sh --product-name ohos-sdk --ccache --build-target out/sdk/gen/build/ohos/sdk:cangjie
```

### 指定平台

```bash
# Windows
python build.py --product-name ohos-sdk --target-os windows

# Linux
python build.py --product-name ohos-sdk --target-os linux

# Mac
python build.py --product-name ohos-sdk --target-os mac
```

## 构建产物配置

### Declare Args (sdk_cangjie.gni)

```gn
declare_args() {
  sdk_build_cangjie_previewer = false    # 是否构建 Previewer
  sdk_build_cangjie_ohos_arm = false     # 是否支持 ohos-arm 平台
}
```

> **证据**: `sdk_cangjie.gni` 第 17-20 行

## Mock 库机制

### Mock 库依赖

```gn
deps = [ "build-tools/lib/mocks/mock:ohos.mock(${toolchain})" ]
```

### Mock 库生成

```
build-tools/lib/mocks/
├── BUILD.gn              # Mock 构建配置
├── sdk_mock.gni         # Mock 模板
├── generate_mock.py     # Mock 生成脚本
├── mock_stub.cpp       # 桩实现
├── mock_stub.h         # 桩头文件
└── mock/
    └── ohos.mock.cj.d  # Mock 模块声明
```

> **证据**: `build-tools/lib/mocks/` 目录结构

## GN 目标 ↔ 产物映射

| GN Target | 产物路径 | 说明 |
|-----------|---------|------|
| `sdk_header_ohos_aarch64` | `cangjie_sdk/api/modules/linux_ohos_aarch64_cjnative/ohos` | 头文件 |
| `sdk_header_ohos_aarch64_kit` | `cangjie_sdk/api/modules/linux_ohos_aarch64_cjnative/kit` | Kit 头文件 |
| `sdk_ohos_aarch64_libs` | `cangjie_sdk/api/lib/linux_ohos_aarch64_cjnative/` | 动态库 |
| `sdk_ohos_x86_64_libs` | `cangjie_sdk/api/lib/linux_ohos_x86_64_cjnative/` | 动态库 |
| `sdk_linux_x86_64_macro` | `cangjie_sdk/api/macro/ohos/` | 宏库 (.so) |

> **证据**: `BUILD.gn` 产物路径定义
