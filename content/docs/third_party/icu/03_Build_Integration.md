# 03 - ICU OpenHarmony 构建适配

## 3.1 构建系统概述

ICU 在 OpenHarmony 中使用 GN/Ninja 构建系统，完全替代了 ICU 原生的 Makefile/Autoconf 构建系统。

### 构建文件结构

```
third_party/icu/
├── BUILD.gn                    # 根构建文件 (Java 部分)
├── icu.gni                     # 全局 ICU 变量定义
├── data_filter.json            # 数据裁剪配置
├── icu4c/
│   ├── BUILD.gn               # ICU4C 主构建文件
│   ├── icu4c_config.gni       # ICU4C 配置
│   └── source/
│       └── BUILD.gn           # 主机工具构建
└── ohos_icu4c/
    └── BUILD.gn               # NDK 封装构建
```

### 构建目标层次

```
┌─────────────────────────────────────────────┐
│              根 BUILD.gn                    │
│         (ohos_icu 组目标)                    │
├─────────────────────────────────────────────┤
│  icu4c/BUILD.gn      │  ohos_icu4c/BUILD.gn │
│  ├─ shared_icuuc     │  ├─ icundk           │
│  ├─ shared_icui18n   │  ├─ ohos_icudat      │
│  ├─ static_icuuc     │  └─ ohos_old_icudat  │
│  ├─ static_icui18n   │                      │
│  └─ static_icu       │                      │
├──────────────────────┼──────────────────────┤
│ icu4c/source/BUILD.gn│                      │
│  ├─ bin_host         │                      │
│  ├─ icuuc_host       │                      │
│  └─ icui18n_host     │                      │
└──────────────────────┴──────────────────────┘
```

---

## 3.2 全局配置 (icu.gni)

**文件**: `icu.gni`

### 全局参数定义

```gn
declare_args() {
  # 功能开关
  icu_support_locales = true
  build_feature = "normal"
  icu_support_libbegetutil = true
  
  # TZData 路径配置
  distro_tzdata_dir = "\"/system/etc/tzdata_distro\""
  system_tzdata_dir = "\"/system/etc/icu_tzdata\""
  
  # 数据文件配置
  icu_dat_name = "icudt74l"
  icu_data_filter_dir = "full"
}
```

### 配置项说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `icu_support_locales` | true | 是否支持多语言本地化 |
| `build_feature` | "normal" | 构建特性: normal/lite |
| `icu_support_libbegetutil` | true | 是否支持 libbegetutil |
| `distro_tzdata_dir` | /system/etc/tzdata_distro | 发行版 TZData 路径 |
| `system_tzdata_dir` | /system/etc/icu_tzdata | 系统 TZData 路径 |
| `icu_dat_name` | icudt74l | ICU 数据文件名 |
| `icu_data_filter_dir` | full | 数据过滤配置目录 |

### 子系统配置 (icu4c_config.gni)

```gn
if (is_standard_system) {
  icu_part_name = "icu"
  icu_subsystem_name = "thirdparty"
} else {
  icu_part_name = "i18n"
  icu_subsystem_name = "global"
}

icu_lib_host_subsystem_name = "global"
icu_bin_host_subsystem_name = "global"
```

---

## 3.3 ICU4C 主构建配置 (icu4c/BUILD.gn)

### 公共配置

```gn
config("icu_config") {
  include_dirs = [
    "//third_party/icu/icu4c/source/common",
    "//third_party/icu/icu4c/source/i18n",
    "//third_party/icu/icu4c/source",
  ]
  if ("${product_name}" == "ohcore") {
    defines = [ "U_ICU_USE_OLD_DATA" ]
  }
}
```

### 动态库: shared_icuuc

```gn
ohos_shared_library("shared_icuuc") {
  branch_protector_ret = "pac_ret"
  ldflags = [ "-shared", "-lm" ]
  
  configs = [ ":icu_config", "//build/config/compiler:rtti" ]
  public_configs = [ ":icu_config", ":static_icustubdata_all_deps_config" ]
  
  defines = [
    "U_ATTRIBUTE_DEPRECATED=",
    "U_COMMON_IMPLEMENTATION",
    "UPRV_BLOCK_MACRO_BEGIN=",
    "UPRV_BLOCK_MACRO_END=",
    "UCONFIG_USE_WINDOWS_LCID_MAPPING_API=0",
    "_REENTRANT",
    "DISTRO_TZDATA_DIR=${distro_tzdata_dir}",
    "SYSTEM_TZDATA_DIR=${system_tzdata_dir}",
  ]
  
  sources = icu_common_source  # 约 250+ 个源文件
  deps = [ ":static_icustubdata" ]
  
  cflags_cc = [
    "-O3", "-W", "-Wall", "-pedantic",
    "-Wpointer-arith", "-Wwrite-strings",
    "-std=c++11",
    # 忽略特定警告
    "-Wno-error=unused-parameter",
    "-Wno-error=unused-const-variable",
    "-Wno-ignored-attributes",
    "-Wno-deprecated-declarations",
  ]
  
  output_name = "hmicuuc"
  innerapi_tags = [ "platformsdk" ]
  relative_install_dir = "platformsdk"
}
```

### 动态库: shared_icui18n

```gn
ohos_shared_library("shared_icui18n") {
  branch_protector_ret = "pac_ret"
  ldflags = [ "-shared", "-lm" ]
  
  if (!build_ohos_sdk) {
    version_script = "libhmicui18n.versionscript"
  }
  
  sources = icu_i18n_source  # 约 250+ 个源文件
  configs = [ ":icu_config", "//build/config/compiler:rtti" ]
  deps = [ ":shared_icuuc" ]
  
  defines = [
    "U_ATTRIBUTE_DEPRECATED=",
    "U_I18N_IMPLEMENTATION",
    "UPRV_BLOCK_MACRO_BEGIN=",
    "UPRV_BLOCK_MACRO_END=",
    "_REENTRANT",
    "PIC",
  ]
  
  output_name = "hmicui18n"
  innerapi_tags = [ "platformsdk" ]
}
```

### 静态库版本

| 目标 | 输出名 | 用途 |
|------|--------|------|
| `static_icuuc` | libhmicuuc.a | 静态链接通用库 |
| `static_icui18n` | libhmicui18n.a | 静态链接 i18n 库 |
| `static_icu` | libhmicu.a | 合并静态库 |

### 静态库特殊配置

```gn
ohos_static_library("static_icuuc") {
  defines = [
    "U_STATIC_IMPLEMENTATION",  # 关键: 静态库标记
    "DISTRO_TZDATA_DIR=${distro_tzdata_dir}",
    "SYSTEM_TZDATA_DIR=${system_tzdata_dir}",
  ]
  
  cflags = [
    "-fdata-sections",
    "-ffunction-sections",
  ]
  
  ldflags = [
    "-static",
    "-ldl",
    "-lm",
  ]
}
```

### 与上游构建的差异

| 特性 | ICU 原生构建 | OpenHarmony 构建 |
|------|-------------|------------------|
| 构建系统 | Autoconf/Make | GN/Ninja |
| 动态库命名 | libicuuc.so | libhmicuuc.so |
| 数据文件 | 运行时加载 | 系统预装 |
| 安装路径 | /usr/lib | /system/lib/platformsdk |
| 交叉编译 | 需手动配置 | 内置支持 |

---

## 3.4 主机工具构建 (icu4c/source/BUILD.gn)

### 目的

ICU 数据文件需要使用 ICU 工具生成，因此需要在**主机**上构建 ICU 工具链。

### 主机库

```gn
ohos_shared_library("shared_icuuc_host") {
  sources = [
    # 约 200 个源文件
    "//third_party/icu/icu4c/source/ohos/init_data.cpp",
    "//third_party/icu/icu4c/source/stubdata/stubdata.cpp",
  ]
  
  defines = [
    "DISTRO_TZDATA_DIR=\"/system/etc/tzdata_distro\"",
    "SYSTEM_TZDATA_DIR=\"/system/etc/icu_tzdata\"",
  ]
  
  output_name = "hmicuuchost"
}
```

### 工具列表

| 工具 | 用途 |
|------|------|
| genbrk | 生成断词规则 |
| genccode | 生成 C 代码数据 |
| gencnval | 生成转换器别名 |
| gendict | 生成字典数据 |
| genrb | 生成资源包 |
| icupkg | ICU 包管理 |
| makeconv | 编码转换器生成 |
| pkgdata | 数据打包 |

### 使用方式

```gn
group("bin_host") {
  deps = [
    ":genbrk(${host_toolchain})",
    ":genrb(${host_toolchain})",
    ":icupkg(${host_toolchain})",
    ":pkgdata(${host_toolchain})",
    # ...
  ]
}
```

---

## 3.5 NDK 构建 (ohos_icu4c/BUILD.gn)

### NDK 共享库

```gn
ohos_shared_library("icundk") {
  ldflags = [ "-shared", "-lm" ]
  configs = [
    "//third_party/icu/icu4c:icu_config",
    "//build/config/compiler:rtti",
  ]
  
  sources = [ "src/icu_addon.cpp" ]
  
  deps = [
    ":ohos_icudat",
    ":ohos_old_icudat",
    "//third_party/icu/icu4c:shared_icui18n",
    "//third_party/icu/icu4c:shared_icuuc",
  ]
  
  if (icu_support_libbegetutil && is_ohos) {
    defines += [ "ICU_SUPPORT_LIBBEGETUTIL" ]
    external_deps += [ "init:libbegetutil" ]
  }
  
  version_script = "libicu.map"  # 符号版本控制
  output_name = "icu"
  output_extension = "so"
  relative_install_dir = "ndk"
}
```

### 符号版本控制

**文件**: `ohos_icu4c/libicu.map`

```
{
  global:
    u_charDigitValue;
    u_charDirection;
    # ... 约 300 个导出符号
    ucal_open;
    udat_format;
    unorm2_normalize;
    # ...
  local:
    *;  # 其他符号不导出
};
```

### 数据打包

```gn
action("pkg_icudata") {
  script = "build_data/pkgdata.sh"
  deps = [ "//third_party/icu/icu4c/source:bin_host" ]
  
  args = [
    "-o", "$root_out_dir",
    "-b", "$icu_bin_root_out_dir",
    "-f", "$icu_data_filter_dir",
    "-v", "$icu_dat_name",
  ]
  
  outputs = [ "$root_out_dir/thirdparty/icu/out/$icu_dat_name.dat" ]
}

ohos_prebuilt_etc("ohos_icudat") {
  source = "$root_out_dir/thirdparty/icu/out/$icu_dat_name.dat"
  module_install_dir = "usr/icu/"
}

ohos_prebuilt_etc("ohos_old_icudat") {
  source = "//third_party/icu/ohos_icu4j/data/old/icudt72l.dat"
  module_install_dir = "usr/ohos_icu/"
}
```

---

## 3.6 数据裁剪配置详解

### 裁剪流程

```
完整 ICU 数据 (icudt74l.dat)
        ↓
  data_filter.json (裁剪规则)
        ↓
  pkgdata.sh (打包脚本)
        ↓
  icu 工具链 (genrb, pkgdata)
        ↓
  裁剪后的数据 (icudt74l.dat)
```

### 裁剪配置结构

```json
{
    "strategy": "subtractive",
    "localeFilter": { ... },
    "featureFilters": {
        "conversion_mappings": { ... },
        "brkitr_tree": { ... },
        "lang_tree": { ... },
        "unit_tree": { ... },
        "zone_tree": { ... },
        "curr_tree": { ... },
        "coll_tree": { ... },
        "region_tree": { ... }
    }
}
```

### 语言白名单示例

```json
"localeFilter": {
    "filterType": "locale",
    "includeChildren": false,
    "includelist": [
        "root",
        "zh", "zh_CN", "zh_Hans", "zh_Hant",
        "en", "en_US", "en_GB",
        "ja", "ja_JP",
        "fr", "fr_FR",
        "de", "de_DE"
        // ... 共约 50 个
    ]
}
```

### 功能树白名单

每种功能树都需要单独配置支持的语言：

```json
"lang_tree": {
    "includelist": [ "root", "zh", "en", "ja", ... ]
},
"coll_tree": {
    "includelist": [ "root", "zh", "en", "ja", ... ]
},
"zone_tree": {
    "includelist": [ "root", "zh", "en", "ja", ... ]
}
```

---

## 3.7 关键编译选项

### 预处理器定义

| 宏 | 说明 |
|----|----|
| `U_COMMON_IMPLEMENTATION` | 编译 icuuc 库 |
| `U_I18N_IMPLEMENTATION` | 编译 icui18n 库 |
| `U_STATIC_IMPLEMENTATION` | 编译静态库 |
| `U_ATTRIBUTE_DEPRECATED=` | 禁用废弃属性 |
| `UPRV_BLOCK_MACRO_BEGIN/END=` | 禁用块宏 |
| `UCONFIG_USE_WINDOWS_LCID_MAPPING_API=0` | 禁用 Windows LCID |
| `DISTRO_TZDATA_DIR` | 发行版 TZData 路径 |
| `SYSTEM_TZDATA_DIR` | 系统 TZData 路径 |
| `U_ICU_USE_OLD_DATA` | 使用旧数据 (ohcore) |

### 编译警告控制

```gn
cflags_cc = [
  "-O3",                    # 优化级别
  "-W", "-Wall",            # 开启警告
  "-pedantic",              # 严格标准
  "-Wpointer-arith",        # 指针运算警告
  "-Wwrite-strings",        # 字符串写入警告
  "-std=c++11",             # C++11 标准
  "-Wno-ignored-attributes", # 忽略属性警告
  "-Wno-deprecated-declarations", # 忽略废弃声明
  "-Wno-error=unused-parameter", # 未使用参数不报错
]
```

### 链接选项

```gn
ldflags = [
  "-shared",    # 动态库
  "-lm",        # 数学库
  "-ldl",       # 动态加载库
]
```

---

## 3.8 构建命令参考

### 构建 ICU 目标

```bash
# 构建所有 ICU 目标
ninja -C out/default third_party/icu/...

# 构建特定目标
ninja -C out/default third_party/icu/icu4c:shared_icuuc
ninja -C out/default third_party/icu/icu4c:shared_icui18n
ninja -C out/default third_party/icu/ohos_icu4c:icundk

# 构建主机工具
ninja -C out/default --toolchain=//build/toolchain/host:clang_x64 \
    third_party/icu/icu4c/source:bin_host
```

### 数据生成

```bash
# 生成裁剪后的数据文件
ninja -C out/default third_party/icu/ohos_icu4c:ohos_icudat
```

### 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 数据文件找不到 | 未生成数据 | 运行 `pkg_icudata` 目标 |
| 主机工具编译失败 | 主机 ICU 未构建 | 先构建 `bin_host` |
| 符号未定义 | 版本脚本问题 | 检查 `libicu.map` |
| TZData 错误 | 路径配置错误 | 检查 `icu.gni` 中的路径 |
