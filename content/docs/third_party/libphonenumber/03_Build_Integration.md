# libphonenumber - OH 构建系统适配

## 概述

libphonenumber 使用 OpenHarmony 的 GN (Generate Ninja) 构建系统进行编译和打包。本文档说明 BUILD.gn 的配置、关键编译选项、依赖关系以及与上游构建系统的差异。

---

## BUILD.gn 结构

### 主目标

libphonenumber 在 OH 中构建为两个共享库：

| 目标 | 类型 | 描述 |
|-----|------|------|
| `phonenumber_standard` | ohos_shared_library | 电话号码解析核心库 |
| `geocoding` | ohos_shared_library | 地理编码库（依赖 phonenumber_standard） |

### 文件位置

- **主配置**: `cpp/BUILD.gn`
- **测试配置**: `cpp/test/BUILD.gn`

---

## 关键编译选项

### 1. 定义 (defines)

#### phonenumber_defines

```gn
phonenumber_defines = [
  "I18N_PHONENUMBERS_USE_ALTERNATE_FORMATS",  # 启用备用格式
  "I18N_PHONENUMBERS_USE_ICU_REGEXP",         # 使用 ICU 正则表达式
  "HAVE_PTHREAD",                              # 支持 pthread
]

if (is_ohos) {
  phonenumber_defines += [ "LIBPHONENUMBER_UPGRADE" ]  # OHOS 特有：运行时更新
}
```

**含义**:

| 宏 | 功能 | 必要性 |
|-----|------|--------|
| `I18N_PHONENUMBERS_USE_ALTERNATE_FORMATS` | 启用备用号码格式化 | 可选（推荐） |
| `I18N_PHONENUMBERS_USE_ICU_REGEXP` | 使用 ICU 正则表达式引擎 | 必须 |
| `HAVE_PTHREAD` | 线程支持 | 必须 |
| `LIBPHONENUMBER_UPGRADE` | OHOS 运行时元数据更新 | OH 特有 |

---

#### geocoding_defines

```gn
geocoding_defines = [
  "I18N_PHONENUMBERS_USE_ALTERNATE_FORMATS",
  "I18N_PHONENUMBERS_USE_ICU_REGEXP",
  "HAVE_PTHREAD",
]

if (is_ohos) {
  geocoding_defines += [ "LIBPHONENUMBER_UPGRADE" ]
}
```

**说明**: geocoding 库使用相同的编译选项。

---

### 2. 配置 (configs)

#### phonenumber_config

```gn
config("phonenumber_config") {
  visibility = [ "./*" ]
  include_dirs = [ "./src" ]  # 头文件搜索路径

  cflags = [ "-Wno-implicit-fallthrough" ]  # 抑制 fallthrough 警告

  cflags_cc = [
    "-DI18N_PHONENUMBERS_USE_ALTERNATE_FORMATS",
    "-DI18N_PHONENUMBERS_USE_ICU_REGEXP",
    "-Dphonenumber_shared_EXPORTS",
    "-Wall",                   # 启用所有警告
    "-fPIC",                   # 位置无关代码（共享库）
    "-Wno-sign-compare",         # 抑制符号比较警告
    "-Wno-error=unused-parameter",
    "-Wno-error=unused-const-variable",
    "-Wno-error=unneeded-internal-declaration",
    "-Wno-implicit-fallthrough",
    "-Wno-deprecated-builtins",
  ]
}
```

**关键字段**:

| 字段 | 值 | 作用 |
|------|------|------|
| `include_dirs` | `[ "./src" ]` | 指定头文件搜索基础路径 |
| `visibility` | `[ "./*" ]` | 控制此配置的可见性 |
| `cflags` | `[ "-Wno-implicit-fallthrough" ]` | C 编译器标志 |
| `cflags_cc` | `[ ... ]` | C++ 编译器标志 |

**编译警告处理**:
- 启用 `-Wall` (所有警告)
- 抑制已知无害警告（`-Wno-*`）
- 使用 CFI 和 PAC_RET 保护（通过 `branch_protector_ret`）

---

#### phonenumber_public_config

```gn
config("phonenumber_public_config") {
  include_dirs = [
    "./src",
    "./src/phonenumbers",
  ]
}
```

**作用**: 为依赖此库的模块提供公共头文件路径

---

## OHOS 条件编译

### is_ohos 逻辑

libphonenumber 使用 `is_ohos` 标志区分 OpenHarmony 构建和其他平台。

#### phonenumber_source 扩展

```gn
phonenumber_source = [
  # 原始源文件
  "src/phonenumbers/phonenumber.cc",
  "src/phonenumbers/phonenumberutil.cc",
  # ... 其他文件
]

if (is_ohos) {
  phonenumber_source += [
    # OHOS 特有源文件
    "src/phonenumbers/ohos/geocoding_data.pb.cc",
    "src/phonenumbers/ohos/update_libphonenumber.cc",
    "src/phonenumbers/ohos/update_metadata.cc",
  ]
}
```

**新增文件**:
- `geocoding_data.pb.cc` - Protobuf 生成的地理编码数据
- `update_libphonenumber.cc` - 元数据加载入口
- `update_metadata.cc` - 元数据更新逻辑

---

#### geocoding_source 扩展

```gn
geocoding_source = [
  # 原始地理编码源文件
  "src/phonenumbers/geocoding/phonenumber_offline_geocoder.cc",
  "src/phonenumbers/geocoding/area_code_map.cc",
  # ... 其他文件
]

if (is_ohos) {
  geocoding_source += [
    # OHOS 特有源文件
    "src/phonenumbers/ohos/geocoding_data.pb.cc",
    "src/phonenumbers/ohos/update_geocoding.cc",
    "src/phonenumbers/ohos/update_libgeocoding.cc",
  ]
}
```

---

## 外部依赖

### public_external_deps (公共外部依赖)

```gn
public_external_deps = [
  "abseil-cpp:absl_strings",   # Abseil 字符串库
  "abseil-cpp:absl_time",      # Abseil 时间库
  "protobuf:protobuf_lite",       # Protocol Buffers 轻量级库
]
```

**说明**: 这些依赖对 `phonenumber_standard` 的公共 API 也是可见的。

---

### external_deps (私有外部依赖)

#### phonenumber_standard

```gn
external_deps = [
  "bounds_checking_function:libsec_shared",  # 安全边界检查
  "icu:shared_icui18n",                   # ICU 国际化 (18n)
  "icu:shared_icuuc",                     # ICU Unicode 库
]
```

**依赖说明**:

| 依赖 | 库 | 功能 | 版本要求 |
|------|------|------|----------|
| `libsec_shared` | bounds_checking_function | 边界检查（溢出保护） | OH 专用 |
| `shared_icui18n` | icu | 国际化（日期、格式化） | ICU 4.8+ |
| `shared_icuuc` | icu | Unicode 处理 | ICU 4.8+ |

---

#### geocoding

```gn
external_deps = [
  "bounds_checking_function:libsec_shared",
  "icu:shared_icuuc",  # 仅需要 icuuc（不需要 icui18n）
]
```

**说明**: geocoding 库依赖 `phonenumber_standard`（通过 `deps`），只需要 ICU Unicode 支持。

---

## 构建产物

### phonenumber_standard 目标

```gn
ohos_shared_library("phonenumber_standard") {
  branch_protector_ret = "pac_ret"  # 返回地址保护
  configs = [ ":phonenumber_config" ]
  public_configs = [ ":phonenumber_public_config" ]
  sources = phonenumber_source
  deps = phonenumber_deps
  public_external_deps = [ ... ]
  external_deps = [ ... ]
  defines = phonenumber_defines
  innerapi_tags = [ "platformsdk_indirect" ]  # 平台 SDK 间接 API
  part_name = "libphonenumber"
  subsystem_name = "thirdparty"
  ldflags = [ "-shared" ]
  install_enable = true
}
```

**关键字段**:

| 字段 | 值 | 说明 |
|------|------|------|
| `branch_protector_ret` | `"pac_ret"` | 返回地址保护（Pointer Authentication） |
| `innerapi_tags` | `[ "platformsdk_indirect" ]` | 标记为平台 SDK 间接 API |
| `part_name` | `"libphonenumber"` | OpenHarmony 分区名 |
| `subsystem_name` | `"thirdparty"` | 子系统名 |
| `install_enable` | `true` | 启用安装到系统 |

**安装位置**: 系统库路径（`/system/lib64/` 或 `/system/lib/`）

---

### geocoding 目标

```gn
ohos_shared_library("geocoding") {
  configs = [ ":phonenumber_config" ]
  sources = geocoding_source
  deps = [ ":phonenumber_standard" ]  # 依赖核心库
  external_deps = [ ... ]
  defines = geocoding_defines
  part_name = "libphonenumber"
  subsystem_name = "thirdparty"
  relative_install_dir = "platformsdk"  # 安装到 platformsdk 目录
  ldflags = [ "-shared" ]
  install_enable = true
}
```

**关键字段**:

| 字段 | 值 | 说明 |
|------|------|------|
| `deps` | `[ ":phonenumber_standard" ]` | 依赖电话号码核心库 |
| `relative_install_dir` | `"platformsdk"` | 相对于 SDK 目录的安装路径 |

**安装位置**: `/system/sdk/platformsdk/lib64/` 或 `/system/sdk/platformsdk/lib/`

---

## 与上游构建系统的差异

### 上游构建系统

原始 libphonenumber 使用多种构建系统：

1. **CMake** - 主要构建系统（`cpp/CMakeLists.txt`）
2. **Bazel** - Google 内部使用
3. **Maven** - Java 版本构建

**关键配置** (`CMakeLists.txt`):

```cmake
option(BUILD_GEOCODER "Build geocoding functionality" ON)
option(I18N_PHONENUMBERS_USE_ICU_REGEXP "Use ICU regexp" ON)
```

---

### OHOS 差异总结

| 方面 | 上游 | OHOS |
|------|--------|-------|
| **构建系统** | CMake, Bazel, Maven | GN (Generate Ninja) |
| **库类型** | 静态/共享库可选 | 共享库（ohos_shared_library） |
| **运行时更新** | 不支持 | 支持（LIBPHONENUMBER_UPGRADE） |
| **安全机制** | 无 | libsec_shared + CFI + PAC |
| **安装路径** | 标准库路径 | 平台 SDK 目录 |
| **外部依赖** | ICU, protobuf | ICU, protobuf, libsec_shared, abseil-cpp |

---

## 源文件组织

### 原始源文件

#### phonenumber_standard 源文件

```gn
phonenumber_source = [
  # 核心实现
  "src/phonenumbers/phonenumber.cc",
  "src/phonenumbers/phonenumberutil.cc",
  "src/phonenumbers/phonenumbermatch.cc",
  "src/phonenumbers/phonenumbermatcher.cc",

  # 元数据
  "src/phonenumbers/metadata.cc",
  "src/phonenumbers/lite_metadata.cc",
  "src/phonenumbers/short_metadata.cc",
  "src/phonenumbers/alternate_format.cc",

  # 功能模块
  "src/phonenumbers/asyoutypeformatter.cc",
  "src/phonenumbers/shortnumberinfo.cc",
  "src/phonenumbers/stringutil.cc",
  "src/phonenumbers/regexp_adapter_icu.cc",

  # Protobuf 生成文件
  "src/phonenumbers/phonemetadata.pb.cc",
  "src/phonenumbers/phonenumber.pb.cc",

  # 工具类
  "src/phonenumbers/base/strings/string_piece.cc",
  "src/phonenumbers/utf/unilib.cc",
  "src/phonenumbers/unicodestring.cc",
]
```

**文件数量**: 约 25 个核心源文件

---

#### geocoding 源文件

```gn
geocoding_source = [
  # 核心实现
  "src/phonenumbers/geocoding/phonenumber_offline_geocoder.cc",
  "src/phonenumbers/geocoding/area_code_map.cc",
  "src/phonenumbers/geocoding/default_map_storage.cc",
  "src/phonenumbers/geocoding/mapping_file_provider.cc",

  # 数据文件
  "src/phonenumbers/geocoding/geocoding_data.cc",

  # C 包装器
  "src/phonenumbers/geocoding/geocoding_warpper.cc",
]
```

**文件数量**: 约 6 个地理编码源文件

---

## 编译流程

### 完整构建命令

```bash
# 构建电话号码核心库
ohos_build.py --build-target phonenumber_standard \
              --product-name ohos-sdk-phone \
              --ccache \
              --build-target ohos-arm64

# 构建地理编码库
ohos_build.py --build-target geocoding \
              --product-name ohos-sdk-phone \
              --ccache \
              --build-target ohos-arm64
```

### GN 生成阶段

```bash
# 生成 Ninja 文件
gn gen --args='is_ohos=true' \
        --target-cpu=arm64 \
        --dotfile=build/graph.dot
```

---

## 头文件暴露

### 公共头文件基础路径

根据 `phonenumber_public_config`，公共头文件位于：

```gn
include_dirs = [
  "./src",
  "./src/phonenumbers",
]
```

**关键公共头文件**:

| 头文件 | 功能 | 使用者 |
|--------|------|--------|
| `phonenumberutil.h` | 核心 API（解析、验证、格式化） | 所有模块 |
| `phonenumber.h` | PhoneNumber 数据结构 | 所有模块 |
| `phonenumber_offline_geocoder.h` | 地理编码 API | SMS/MMS |
| `phonenumber.pb.h` | Protobuf 生成的元数据结构 | 内部使用 |
| `phonemetadata.pb.h` | 元数据 Protobuf 定义 | 内部使用 |

---

## 测试构建

### cpp/test/BUILD.gn

```gn
ohos_unittest("libphonenumber_test") {
  sources = [ "phonenumberutil_test.cc", ... ]
  deps = [ ":phonenumber_standard" ]
  external_deps = [ "googletest:googletest" ]
  part_name = "libphonenumber"
  subsystem_name = "thirdparty"
}
```

**测试框架**: Google Test (googletest)

**测试范围**:
- 电话号码解析测试
- 验证测试（所有国家）
- 格式化测试
- 元数据更新测试

---

## 构建优化

### 编译器优化

```gn
cflags_cc = [
  "-Wall",           # 启用警告
  "-fPIC",           # 位置无关代码
  "-O2",             # 优化级别 2（隐含）
]
```

### 链接器优化

```gn
ldflags = [
  "-shared",         # 共享库
  "-Wl,--as-needed", # 仅链接必需符号
]
```

### 安全加固

```gn
branch_protector_ret = "pac_ret"  # PAC 返回保护
# CFI 在 sanitize 中启用：
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

---

## 故障排查

### 常见编译错误

**错误**: `undefined reference to 'UpdateMetadata::LoadUpdatedMetadata'`

**原因**: 忘记添加 `update_metadata.cc` 到源文件列表

**解决**: 确保 `is_ohos` 条件编译正确添加 OHOS 特有文件

---

**错误**: `cannot find -licuuc`

**原因**: ICU 依赖未正确配置

**解决**:
```gn
external_deps = [
  "icu:shared_icuuc",
]
```

---

**错误**: `warning: implicit declaration of function 'xyz'`

**原因**: 头文件未包含或可见性配置错误

**解决**:
```gn
include_dirs = [
  "./src",
  "./src/phonenumbers",
]
```

---

## 构建产物验证

### 库文件检查

```bash
# 检查符号
nm -D libphonenumber_standard.so | grep UpdateMetadata

# 检查依赖
otool -L libphonenumber_standard.so

# 检查架构
lipo -info libphonenumber_standard.so
```

### 动态加载测试

```cpp
#include <dlfcn.h>

void* handle = dlopen("libphonenumber_standard.so", RTLD_NOW);
auto* create_func = (void*(*)())dlsym(handle, "create_i18n_phonenumbers_PhoneNumberUtil");
```

---

## 总结

### 关键要点

1. **OHOS 特性**:
   - 使用 GN 构建系统
   - 运行时元数据更新（`LIBPHONENUMBER_UPGRADE`）
   - 集成安全库（`libsec_shared`）

2. **两个共享库**:
   - `phonenumber_standard.so` - 核心功能
   - `geocoding.so` - 地理编码（依赖核心库）

3. **外部依赖**:
   - ICU（国际化）
   - protobuf（序列化）
   - abseil-cpp（工具库）
   - libsec_shared（安全）

4. **安装位置**:
   - `phonenumber_standard`: 系统库路径
   - `geocoding`: 平台 SDK 目录

---

**最后更新**: 2026-02-08
