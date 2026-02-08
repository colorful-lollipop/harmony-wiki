# GN Targets 与编译配置

## 目的

本文档详细描述 c_utils 的 GN 构建配置、targets 定义和编译选项。

## 适用范围

- 构建工程师配置编译
- 开发者添加依赖
- 排查构建问题

---

## 构建文件位置

| 文件 | 路径 | 用途 |
|------|------|------|
| 主构建文件 | `base/BUILD.gn` | 定义所有 targets |
| 部件配置 | `bundle.json` | 组件元数据、feature flags |
| Rust配置 | `base/src/rust/Cargo.toml` | Rust包配置 |

---

## Targets 清单

### 1. utilsbase（静态库）

**定义位置**: `base/BUILD.gn:125-179`

```gn
ohos_static_library("utilsbase") {
  visibility = [ "//applications/...", "//base/...", ... ]  # 受限可见性
  
  sources = sources_utils  # 24个cpp文件
  
  configs = [ ":utils_coverage_config" ]
  public_configs = [ ":utils_config" ]
  
  public_external_deps = [ "bounds_checking_function:libsec_static" ]
  
  subsystem_name = "commonlibrary"
  part_name = "c_utils"
}
```

**属性**:

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | ohos_static_library | 静态库 |
| 源文件 | 24个cpp | 完整功能 |
| 平台适配 | iOS限制5个文件 | 目录、Parcel、RefBase、RWLock、String |
| 依赖 | libsec_static | 安全C库（静态）|
| 可见性 | 受限 | 特定模块白名单 |

**可见性白名单** (`base/BUILD.gn:125-153`):
```gn
visibility = [
  "//applications/standard/contacts_data:contactsdataability",
  "//base/hiviewdfx/faultloggerd/*",
  "//base/startup/init/interfaces/innerkits:libbegetutil_static",
  "//foundation/ability/ability_runtime/interfaces/inner_api/mission_manager/*",
  "//foundation/multimedia/image_framework/*",
  "//foundation/multimodalinput/input/*",
  "//third_party/sqlite/*",
  // ... 更多
]
```

---

### 2. utils（动态库）

**定义位置**: `base/BUILD.gn:196-284`

```gn
ohos_shared_library("utils") {
  innerapi_tags = [
    "chipsetsdk_sp",
    "platformsdk", 
    "sasdk",
  ]
  
  sources = sources_utils
  
  # 安全特性
  if (c_utils_feature_intsan) {
    sanitize = { integer_overflow = true }
    branch_protector_ret = "pac_ret"
  }
  
  # 调试特性
  if (c_utils_debug_refbase) { configs += [ ":debug_refbase" ] }
  if (c_utils_parcel_object_check) { configs += [ ":parcel_object_check" ] }
  
  public_external_deps = [ "bounds_checking_function:libsec_shared" ]
  
  install_images = [ "system", "updater" ]
}
```

**属性**:

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | ohos_shared_library | 动态库 |
| API等级 | platformsdk / sasdk / chipsetsdk_sp | 平台SDK级别 |
| 源文件 | 24个cpp | 完整功能 |
| 平台适配 | Win/Mac/iOS受限 | 仅基础功能 |
| 依赖 | libsec_shared | 安全C库（动态）|
| 安装镜像 | system, updater | 系统分区和升级分区 |

**输出文件**:
- `out/{product}/commonlibrary/c_utils/base/libutils.so`

---

### 3. utils_rust（Rust动态库）

**定义位置**: `base/BUILD.gn:330-367`

```gn
ohos_rust_shared_library("utils_rust") {
  sources = [
    "src/rust/ashmem.rs",
    "src/rust/directory_ex.rs", 
    "src/rust/file_ex.rs",
    "src/rust/lib.rs",
  ]
  
  crate_root = "src/rust/lib.rs"
  crate_name = "utils_rust"
  crate_type = "dylib"
  output_extension = "dylib.so"
  
  deps = [ ":utils_static_cxx_rust" ]
  external_deps = [ "rust_cxx:lib" ]
  
  install_images = [ "system", "updater" ]
}
```

**属性**:

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | ohos_rust_shared_library | Rust动态库 |
| 条件 | Linux && !arm64 && !mac | 有限平台支持 |
| 源文件 | 4个rs文件 | Rust实现 |
| 依赖 | rust_cxx | Rust/C++互操作 |
| 输出 | dylib.so | Rust动态库格式 |

**构建条件** (`base/BUILD.gn:288-294`):
```gn
declare_args() {
  utils_rust_enable = true
  if (defined(global_parts_info) &&
      !defined(global_parts_info.thirdparty_rust_cxx)) {
    utils_rust_enable = false  # 依赖 rust_cxx 部件
  }
}
```

---

### 4. utilsbase_rtti（带RTTI静态库）

**定义位置**: `base/BUILD.gn:181-194`

```gn
ohos_static_library("utilsbase_rtti") {
  visibility = [ "//foundation/multimedia/media_foundation/..." ]
  
  sources = sources_utils
  
  remove_configs = [ "//build/config/compiler:no_rtti" ]
  cflags = [ "-frtti" ]
  
  public_external_deps = [ "bounds_checking_function:libsec_static" ]
}
```

**特殊用途**: 多媒体引擎插件需要 RTTI 支持

---

### 5. utils_static_cxx_rust（Rust/C++混合静态库）

**定义位置**: `base/BUILD.gn:307-328`

```gn
ohos_static_library("utils_static_cxx_rust") {
  sources = [
    "src/ashmem.cpp",
    "src/directory_ex.cpp",
    "src/file_ex.cpp",
    "src/refbase.cpp",
  ]
  sources += get_target_outputs(":cxx_rust_gen")
  
  defines = [ "UTILS_CXX_RUST" ]
  include_dirs = [ "include", "${target_gen_dir}" ]
  
  deps = [ ":cxx_rust_gen" ]
  external_deps = [ "rust_cxx:cxx_cppdeps" ]
}
```

**用途**: Rust FFI 绑定的 C++ 桥接层

---

## Config 定义

### utils_config（基础配置）

**定义**: `base/BUILD.gn:28-46`

```gn
config("utils_config") {
  include_dirs = [ "include" ]
  defines = []
  
  if (current_os == "ios") { defines += [ "IOS_PLATFORM" ] }
  if (current_os == "win" || current_os == "mingw") { defines += [ "WINDOWS_PLATFORM" ] }
  if (current_os == "mac") { defines += [ "MAC_PLATFORM" ] }
  if (is_emulator == true) { defines += [ "EMULATOR_PLATFORM" ] }
  if (current_os == "ohos") { defines += [ "OHOS_PLATFORM" ] }
}
```

**效果**: 所有依赖 c_utils 的模块自动获得：
- include 路径: `//commonlibrary/c_utils/base/include`
- 平台宏定义

### 调试配置

| Config | 宏定义 | 条件 |
|--------|--------|------|
| debug_log_enabled | DEBUG_UTILS | c_utils_debug_log_enabled |
| debug_refbase | DEBUG_REFBASE | c_utils_debug_refbase |
| print_track_at_once | PRINT_TRACK_AT_ONCE | c_utils_print_track_at_once |
| track_all | TRACK_ALL | c_utils_track_all |
| parcel_object_check | PARCEL_OBJECT_CHECK | c_utils_parcel_object_check |

---

## Feature Flags

### 声明位置

`base/BUILD.gn:16-26`

```gn
declare_args() {
  c_utils_feature_coverage = false      # 代码覆盖率
  c_utils_debug_refbase = false         # RefBase调试
  c_utils_track_all = false             # 跟踪所有RefBase
  c_utils_print_track_at_once = false   # 立即打印跟踪
  c_utils_debug_log_enabled = false     # 调试日志
  c_utils_feature_intsan = true         # 整数溢出检测
  c_utils_parcel_object_check = true    # Parcel对象检查
  c_utils_feature_enable_pgo = false    # PGO优化
  c_utils_feature_pgo_path = ""         # PGO数据路径
}
```

### 功能说明

| Flag | 默认值 | 功能 | 影响 |
|------|--------|------|------|
| c_utils_feature_coverage | false | 启用gcov覆盖率 | 添加 `--coverage` 编译选项 |
| c_utils_debug_refbase | false | RefBase调试 | 启用引用跟踪和日志 |
| c_utils_track_all | false | 跟踪所有对象 | 记录所有RefBase生命周期 |
| c_utils_print_track_at_once | false | 立即打印 | 实时输出跟踪信息 |
| c_utils_debug_log_enabled | false | 调试日志 | 启用DEBUG_UTILS宏 |
| c_utils_feature_intsan | true | 整数溢出检测 | 启用IntegerSanitizer |
| c_utils_parcel_object_check | true | Parcel对象检查 | 启用PARCEL_OBJECT_CHECK |
| c_utils_feature_enable_pgo | false | PGO优化 | 使用profile引导优化 |

---

## 编译命令

### 编译整个部件

```bash
./build.sh --product-name rk3568 --build-target c_utils
```

### 编译动态库

```bash
./build.sh --product-name rk3568 --build-target commonlibrary/c_utils/base:utils
```

### 编译静态库

```bash
./build.sh --product-name rk3568 --build-target commonlibrary/c_utils/base:utilsbase
```

### 编译Rust动态库

```bash
./build.sh --product-name rk3568 --build-target commonlibrary/c_utils/base:utils_rust
```

---

## 依赖方式

### 在 BUILD.gn 中添加依赖

```gn
ohos_shared_library("my_module") {
  # 方式1: 依赖动态库（推荐）
  external_deps = [
    "c_utils:utils",
  ]
  
  # 方式2: 依赖静态库
  external_deps = [
    "c_utils:utilsbase",
  ]
  
  # 方式3: 依赖Rust库
  external_deps = [
    "c_utils:utils_rust",
  ]
}
```

### 头文件包含

```cpp
// 自动获得 include 路径，无需指定完整路径
#include "refbase.h"
#include "parcel.h"
#include "safe_map.h"
```

---

## 平台差异处理

### 源文件选择

```gn
# 完整功能（OHOS/Linux/Android）
sources_utils = [
  "src/string_ex.cpp",
  "src/unicode_ex.cpp",
  "src/directory_ex.cpp",
  "src/datetime_ex.cpp",
  "src/refbase.cpp",
  "src/parcel.cpp",
  "src/semaphore_ex.cpp",
  "src/thread_pool.cpp",
  "src/file_ex.cpp",
  "src/mapped_file.cpp",
  "src/observer.cpp",
  "src/thread_ex.cpp",
  "src/io_event_handler.cpp",
  "src/io_event_reactor.cpp",
  "src/io_event_epoll.cpp",
  "src/event_handler.cpp",
  "src/event_reactor.cpp",
  "src/event_demultiplexer.cpp",
  "src/timer.cpp",
  "src/timer_event_handler.cpp",
  "src/ashmem.cpp",
  "src/rwlock.cpp",
]

# Windows/Mac 受限功能
sources_utils_win_and_mac = [
  "src/parcel.cpp",
  "src/refbase.cpp",
  "src/string_ex.cpp",
  "src/unicode_ex.cpp",
]

# iOS 受限功能
sources_utils_ios = [
  "src/directory_ex.cpp",
  "src/parcel.cpp",
  "src/refbase.cpp",
  "src/rwlock.cpp",
  "src/string_ex.cpp",
]
```

---

## 安全编译选项

### 整数溢出检测

```gn
if (c_utils_feature_intsan) {
  sanitize = {
    integer_overflow = true
  }
  branch_protector_ret = "pac_ret"  # ARM64 返回地址保护
}
```

### PGO优化

```gn
if (c_utils_feature_enable_pgo) {
  cflags = [
    "-fprofile-use=${c_utils_feature_pgo_path}/libutils.profdata",
    "-Wno-error=backend-plugin",
    "-Wno-profile-instr-out-of-date",
    "-Wno-profile-instr-unprofiled",
  ]
}

# ARM64 特定优化
ldflags = [ "-Wl,-Bsymbolic" ]
if (c_utils_feature_enable_pgo && target_cpu == "arm64") {
  ldflags += [ "-Wl,--aarch64-inline-plt" ]
}
```

---

## 相关跳转

- [编译产物](07_Build_Artifacts.md) - 输出文件清单
- [配置与宏](appendix/Config_Flags.md) - 详细配置说明
- [内部 API](05_Inner_API.md) - 模块实现细节
