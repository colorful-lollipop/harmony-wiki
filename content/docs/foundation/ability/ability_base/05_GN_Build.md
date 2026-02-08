# GN 构建系统

## 目的

本文档详细说明 ability_base 组件的 GN 构建系统，包括 targets、依赖关系、编译产物、配置选项和构建命令，帮助开发者理解组件的构建流程和产物。

---

## 1. GN 构建文件概览

### 1.1 主要构建文件

| 文件 | 路径 | 作用 | 行数 |
|------|------|------|------|
| 主构建配置 | `BUILD.gn` | 定义所有生产 targets（10 个共享库） | 474 |
| 构建变量 | `ability_base.gni` | 定义路径变量供 BUILD.gn 使用 | 22 |
| 组件元数据 | `bundle.json` | 组件信息、依赖、导出接口 | 164 |

**证据位置**：
- BUILD.gn：`BUILD.gn:1-474`
- ability_base.gni：`ability_base.gni:1-22`
- bundle.json：`bundle.json:1-164`

### 1.2 测试构建文件（不包含在文档中）

| 文件 | 路径 | targets 数量 |
|------|------|-----------|
| 单元测试 | `test/unittest/BUILD.gn` | 30 个 |
| 模糊测试 | `test/fuzztest/BUILD.gn` | 24 个 |

---

## 2. 生产 Targets（10 个共享库）

### 2.1 Target 依赖关系图

```
base_innerkits_target (group)
├── base (ohos_shared_library)
├── configuration (ohos_shared_library)
├── zuri (ohos_shared_library)
├── want (ohos_shared_library) ─┬─> base
│                          └──> zuri
├── view_data (ohos_shared_library)
├── session_info (ohos_shared_library) ─┬─> want
├── string_utils (ohos_shared_library)
├── extractortool (ohos_shared_library) ─┬─> string_utils
├── extractresourcemanager (ohos_shared_library)
└── ability_base_want (ohos_shared_library, NDK) ─┬─> base
                                                    └──> want
```

**证据位置**：
- 依赖关系：`BUILD.gn` - 各 target 的 `deps` 字段

### 2.2 Target 详细列表

#### 2.2.1 base - 基础类型库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:base` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libbase.so` |
| **分支保护** | `pac_ret` |

**源文件（13 个）**：
```
interfaces/inner_api/base/src/
├── base.cpp
├── base_object.cpp
├── bool_wrapper.cpp
├── byte_wrapper.cpp
├── double_wrapper.cpp
├── float_wrapper.cpp
├── int_wrapper.cpp
├── long_wrapper.cpp
├── remote_object_wrapper.cpp
├── short_wrapper.cpp
├── string_wrapper.cpp
├── user_object_wrapper.cpp
└── zchar_wrapper.cpp
```

**Configs**：
- `:base_config` - 私有可见性
- `:base_exceptions_config` - `cflags_cc = ["-fexceptions"]`
- `:base_public_config` - include_dirs

**外部依赖**：
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`

**InnerAPI 标签**：`platformsdk`, `sasdk`

**证据位置**：`BUILD.gn:34-70`

#### 2.2.2 configuration - 系统配置库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:configuration` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libconfiguration.so` |
| **分支保护** | `pac_ret` |

**源文件（2 个）**：
```
interfaces/kits/native/configuration/src/
├── configuration.cpp
└── configuration_convertor.cpp
```

**Feature Flags**：
- `ABILITYBASE_LOG_TAG = "Configuration"`
- `BINDER_IPC_32BIT` (target_cpu == "arm")

**外部依赖**：
- `c_utils:utils`
- `hilog:libhilog`
- `json:nlohmann_json_static`
- `resource_management:global_resmgr` (public)

**InnerAPI 标签**：`platformsdk`

**证据位置**：`BUILD.gn:84-112`

#### 2.2.3 zuri - URI 处理库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:zuri` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libzuri.so` |
| **分支保护** | `pac_ret` |

**源文件（1 个）**：
```
interfaces/kits/native/uri/src/
└── uri.cpp
```

**Feature Flags**：
- `BINDER_IPC_32BIT` (target_cpu == "arm")

**Configs**：
- `:zuri_config` - include_dirs, cflags
- `:zuri_exceptions` - `cflags_cc = ["-fexceptions"]`

**外部依赖**：
- `c_utils:utils`
- `hilog:libhilog`

**InnerAPI 标签**：`platformsdk`, `sasdk`

**证据位置**：`BUILD.gn:128-150`

#### 2.2.4 want - Want 参数库（最大模块）

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:want` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libwant.so` |
| **分支保护** | `pac_ret` |
| **异常支持** | `use_exceptions = true` |

**源文件（11 个）**：
```
interfaces/kits/native/want/src/
├── array_wrapper.cpp
├── element_name.cpp
├── extra_params.cpp
├── operation.cpp
├── operation_builder.cpp
├── pac_map.cpp
├── patterns_matcher.cpp
├── skills.cpp
├── want.cpp
├── want_params.cpp
└── want_params_wrapper.cpp
```

**内部依赖**：
- `:base`
- `:zuri`

**Feature Flags**：
- `ABILITYBASE_LOG_TAG = "Want"`
- `BINDER_IPC_32BIT` (target_cpu == "arm")
- `-Werror,-Wfloat-equal` (cflags)

**Configs**：
- `:want_config`
- `:want_exceptions_config` - `cflags_cc = ["-fexceptions"]`
- `:want_public_config` - include_dirs
- `:want_all_dependent_config` - include_dirs

**外部依赖**：
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core` (public & deps)
- `ipc:ipc_single` (deps)
- `jsoncpp:jsoncpp` (public & deps)
- `json:nlohmann_json_static` (public)

**InnerAPI 标签**：`platformsdk`, `sasdk`

**证据位置**：`BUILD.gn:184-234`

#### 2.2.5 view_data - 视图数据结构库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:view_data` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libview_data.so` |
| **分支保护** | `pac_ret` |

**源文件（3 个）**：
```
interfaces/kits/native/view_data/src/
├── page_node_info.cpp
├── rect.cpp
└── view_data.cpp
```

**Configs**：
- `:view_data_config` - include_dirs

**外部依赖**：
- `hilog:libhilog`
- `json:nlohmann_json_static`

**InnerAPI 标签**：`platformsdk_indirect`

**证据位置**：`BUILD.gn:244-264`

#### 2.2.6 session_info - 会话信息库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:session_info` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libsession_info.so` |
| **分支保护** | `pac_ret` |

**源文件（1 个）**：
```
interfaces/kits/native/session_info/src/
└── session_info.cpp
```

**内部依赖**：
- `:want`

**Configs**：
- `:session_info_all_dependent_config` - include_dirs

**外部依赖**：
- `ability_runtime:ability_start_setting`
- `ability_runtime:process_options`
- `ability_runtime:start_window_option`
- `bundle_framework:appexecfwk_base`
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `window_manager:window_animation_utils`

**InnerAPI 标签**：`platformsdk_indirect`

**证据位置**：`BUILD.gn:274-297`

#### 2.2.7 string_utils - 字符串工具库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:string_utils` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libstring_utils.so` |
| **分支保护** | `pac_ret` |

**源文件（1 个）**：
```
interfaces/kits/native/extractortool/src/
└── file_path_utils.cpp
```

**Feature Flags**：
- `WINDOWS_PLATFORM` (is_mingw)
- `MAC_PLATFORM` (!is_mingw)
- `BINDER_IPC_32BIT` (target_cpu == "arm")

**Configs**：
- `:string_utils_config` - include_dirs

**InnerAPI 标签**：`chipsetsdk_indirect`, `platformsdk_indirect`

**证据位置**：`BUILD.gn:304-328`

#### 2.2.8 extractortool - ZIP 提取工具库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:extractortool` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libextractortool.so` |
| **分支保护** | `pac_ret` |

**源文件（5 个）**：
```
interfaces/kits/native/extractortool/src/
├── extractor.cpp
├── file_mapper.cpp
├── zip_file.cpp
├── zip_file_reader.cpp
└── zip_file_reader_io.cpp
```

**Feature Flags**：
- `BINDER_IPC_32BIT` (target_cpu == "arm")

**Configs**：
- `:exceptions` - `cflags_cc = ["-fexceptions"]`
- `:ability_extractor_config` - include_dirs

**内部依赖**：
- `:string_utils`

**外部依赖**：
- `c_utils:utils`
- `hilog:libhilog`
- `hitrace:hitrace_meter`
- `json:nlohmann_json_static`
- `zlib:libz` (public)
- `zlib:shared_libz` (public)

**InnerAPI 标签**：`chipsetsdk_indirect`, `platformsdk_indirect`

**证据位置**：`BUILD.gn:342-382`

#### 2.2.9 extractresourcemanager - 资源管理库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:extractresourcemanager` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libextractresourcemanager.so` |
| **分支保护** | `pac_ret` |

**源文件（1 个）**：
```
interfaces/kits/native/extractortool/src/
└── extract_resource_manager.cpp
```

**Feature Flags**：
- `BINDER_IPC_32BIT` (target_cpu == "arm")

**Configs**：
- `:ability_extract_resource_manager_config` - include_dirs

**外部依赖**：
- `resource_management:global_resmgr`

**InnerAPI 标签**：`platformsdk_indirect`

**证据位置**：`BUILD.gn:389-408`

#### 2.2.10 ability_base_want - C NDK 库

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:ability_base_want` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libability_base_want.so` |
| **分支保护** | `pac_ret` |
| **Sanitize 选项** | 严格安全检查 |

**源文件（2 个）**：
```
interfaces/kits/c/cwant/src/
├── want.cpp
└── want_manager.cpp
```

**Sanitize 选项**：
```
sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    cfi_vcall_icall_only = true
    debug = false
}
```

**内部依赖**：
- `:base`
- `:want`

**Configs**：
- `:ability_base_ndk_config` - include_dirs

**外部依赖**：
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_single`

**InnerAPI 标签**：`ndk`
**安装镜像**：`system`

**证据位置**：`BUILD.gn:424-459`

### 2.3 Group Targets

#### 2.3.1 base_innerkits_target

| 属性 | 值 |
|------|-----|
| **Target 名称** | `//foundation/ability/ability_base:base_innerkits_target` |
| **类型** | `group` |

**聚合所有生产库**：
- `:ability_base_want`
- `:base`
- `:configuration`
- `:extractortool`
- `:extractresourcemanager`
- `:session_info`
- `:string_utils`
- `:view_data`
- `:want`

这是 `bundle.json` 中引用的主 sub_component。

**证据位置**：`BUILD.gn:461-473`

---

## 3. 构建变量与路径

### 3.1 ability_base.gni 路径变量

| 变量 | 值 |
|------|-----|
| `ability_base_path` | `//foundation/ability/ability_base` |
| `ability_base_innerapi_path` | `${ability_base_path}/interfaces/inner_api` |
| `ability_base_kits_native_path` | `${ability_base_path}/interfaces/kits/native` |
| `ability_base_ndk_path` | `${ability_base_path}/interfaces/kits/c` |
| `base_global_innerapi_path` | `//base/global/resource_management/interfaces/inner_api` |
| `base_fuzz_output_path` | `ability_base/ability_base` |
| `ipc_native_path` | `//foundation/communication/ipc/ipc/native` |

**证据位置**：`ability_base.gni:14-22`

### 3.2 Config include_dirs 路径

#### base
```
include_dirs = [
    "interfaces/inner_api/base/include",
    "${ability_base_innerapi_path}/log/include",
]
```

#### want
```
include_dirs = [
    "interfaces/inner_api/base/include",
    "interfaces/kits/native/uri/include",
    "interfaces/kits/native/want/include",
    "${ability_base_innerapi_path}/log/include",
]
```

**证据位置**：`BUILD.gn` - 各 target 的 `public_configs`

---

## 4. Feature Flags 与配置

### 4.1 日志标签（ABILITYBASE_LOG_TAG）

| Target | 标签值 |
|--------|---------|
| configuration | `"Configuration"` |
| want | `"Want"` |

**证据位置**：
- Configuration：`BUILD.gn:96`
- Want：`BUILD.gn:162`

### 4.2 架构相关 Flags

| Flag | 适用条件 | 说明 |
|------|----------|------|
| `BINDER_IPC_32BIT` | target_cpu == "arm" | 32 位 ARM IPC 支持 |
| `WINDOWS_PLATFORM` | is_mingw | Windows 平台 |
| `MAC_PLATFORM` | !is_mingw | macOS 平台 |

**证据位置**：
- BINDER_IPC_32BIT：`BUILD.gn:99, 120, 159, 197, 317, 358, 398`
- WINDOWS/MAC：`BUILD.gn:307-311`

### 4.3 编译选项

| 选项 | 适用范围 | 值 |
|------|----------|-----|
| `-fexceptions` | 多数 targets | 启用 C++ 异常 |
| `-Werror,-Wfloat-equal` | want | 严格的浮点比较警告 |
| `pac_ret` | 所有共享库 | 分支保护（返回地址保护） |

**证据位置**：
- -fexceptions：`BUILD.gn:31, 81, 125, 166, 210`
- -Werror：`BUILD.gn:161`
- pac_ret：所有 ohos_shared_library

---

## 5. 编译产物

### 5.1 共享库输出

| Target | 输出文件 | 大小估计 | 安装路径 |
|--------|-----------|-----------|----------|
| base | `libbase.so` | ~50KB | /usr/lib/ |
| configuration | `libconfiguration.so` | ~80KB | /usr/lib/ |
| zuri | `libzuri.so` | ~40KB | /usr/lib/ |
| want | `libwant.so` | ~200KB | /usr/lib/ |
| view_data | `libview_data.so` | ~60KB | /usr/lib/ |
| session_info | `libsession_info.so` | ~100KB | /usr/lib/ |
| string_utils | `libstring_utils.so` | ~30KB | /usr/lib/ |
| extractortool | `libextractortool.so` | ~120KB | /usr/lib/ |
| extractresourcemanager | `libextractresourcemanager.so` | ~20KB | /usr/lib/ |
| ability_base_want | `libability_base_want.so` | ~150KB | /system/lib/ |

**证据位置**：
- 输出名称：`BUILD.gn` - 各 target 的输出推断
- 安装路径：`BUILD.gn:456` - `ability_base_want` 的 `install_images = ["system"]`

### 5.2 头文件导出

**bundle.json 中的 inner_kits**：

| Target | 头文件路径 | 导出头文件 |
|--------|-----------|-----------|
| base | `interfaces/inner_api/base/include/` | 17 个头文件 |
| want | `interfaces/kits/native/want/include/` | 12 个头文件 |
| configuration | `interfaces/kits/native/configuration/include/` | 3 个头文件 |
| zuri | `interfaces/kits/native/uri/include/` | 1 个头文件 |
| view_data | `interfaces/kits/native/view_data/include/` | 4 个头文件 |
| session_info | `interfaces/kits/native/session_info/include/` | 2 个头文件 |
| extractortool | `interfaces/kits/native/extractortool/include/` | 8 个头文件 |
| string_utils | `interfaces/kits/native/extractortool/include/` | 1 个头文件 |
| extractresourcemanager | `interfaces/kits/native/extractortool/include/` | 1 个头文件 |
| ability_base_want | `interfaces/kits/c/cwant/include/` | 2 个头文件 |

**证据位置**：`bundle.json:44-156`

---

## 6. 构建命令

### 6.1 生成构建文件

```bash
# 进入 OpenHarmony 源码根目录
cd /path/to/openharmony

# 生成构建文件
gn gen out/ohos-arm64 --target_cpu=arm64

# 生成调试版本
gn gen out/ohos-arm64-debug --target_cpu=arm64 --is_debug=true
```

### 6.2 编译 ability_base

```bash
# 编译所有 ability_base 生产库
ninja -C out/ohos-arm64 //foundation/ability/ability_base:base_innerkits_target

# 编译特定 target
ninja -C out/ohos-arm64 //foundation/ability/ability_base:want

# 编译 C NDK 库
ninja -C out/ohos-arm64 //foundation/ability/ability_base:ability_base_want
```

### 6.3 清理构建产物

```bash
# 清理特定 target
ninja -C out/ohos-arm64 //foundation/ability/ability_base:want --clean

# 清理所有 ability_base
ninja -C out/ohos-arm64 //foundation/ability/ability_base --clean
```

### 6.4 查看依赖关系

```bash
# 查看 target 的依赖树
gn analyze out/ohos-arm64 //foundation/ability/ability_base:want --tree

# 查看依赖图（生成 dot 文件）
gn analyze out/ohos-arm64 //foundation/ability/ability_base:want --dot > want_deps.dot
```

---

## 7. 运行时加载关系

### 7.1 静态链接依赖

```
libwant.so
  ├── libbase.so (动态依赖）
  ├── libzuri.so (动态依赖）
  ├── libipc_core.so (动态外部依赖）
  ├── libipc_single.so (动态外部依赖）
  ├── libjsoncpp.so (动态外部依赖）
  ├── libnlohmann_json_static.a (静态外部依赖）
  ├── libhilog.so (动态外部依赖）
  └── libc++_shared.so (系统库）
```

### 7.2 运行时加载顺序

```
1. 应用启动时，系统加载器首先加载 libability_base_want.so (NDK)
2. libability_base_want.so 依赖并触发加载 libwant.so
3. libwant.so 触发加载 libbase.so 和 libzuri.so
4. 所有外部依赖按需加载（libipc_core.so, libhilog.so 等）
```

---

## 8. 常见构建问题

### 8.1 依赖未找到

**问题**：
```
ERROR at //foundation/ability/ability_base/BUILD.gn:XX:11: Unable to load "//foundation/communication/ipc/ipc/native:ipc_core"
```

**解决方案**：
1. 确保 IPC 子系统已拉取
2. 检查 `build/lite/ohos.gni` 中的路径配置
3. 运行 `hb set --ccache` 重新同步依赖

### 8.2 头文件路径错误

**问题**：
```
fatal error: 'parcel.h' file not found
```

**解决方案**：
1. 检查 `include_dirs` 配置
2. 确保正确引用 `ipc_native_path`
3. 运行 `hb clean && hb build -f` 强制重新构建

### 8.3 C++ 标准库不匹配

**问题**：
```
error: 'std::recursive_mutex' is unavailable
```

**解决方案**：
1. 确保 C++14 或更高标准
2. 检查 `BUILD.gn` 中的 `cflags_cc` 配置
3. 使用正确的 NDK/编译器版本

---

## 9. bundle.json 配置

### 9.1 组件元数据

```json
{
    "name": "@ohos/ability_base",
    "description": "ability子系统中的基础库,want,base等",
    "version": "3.1",
    "license": "Apache License 2.0",
    "component": {
        "name": "ability_base",
        "subsystem": "ability",
        "syscap": ["SystemCapability.Ability.AbilityBase"],
        "adapted_system_type": ["standard"]
    }
}
```

**证据位置**：`bundle.json:2-22`

### 9.2 组件依赖

```json
"deps": {
    "components": [
        "ability_runtime",
        "bundle_framework",
        "c_utils",
        "hilog",
        "hitrace",
        "ipc",
        "resource_management",
        "json",
        "jsoncpp",
        "zlib",
        "window_manager"
    ]
}
```

**证据位置**：`bundle.json:24-39`

### 9.3 导出接口（inner_kits）

**导出库数量**：10 个

**导出目标**：
```json
"build": {
    "sub_component": ["//foundation/ability/ability_base:base_innerkits_target"],
    "inner_kits": [
        // 10 个库的头文件导出配置
    ]
}
```

**证据位置**：`bundle.json:40-161`

---

## 相关跳转

- 📁 **目录结构**：[01_Directory_Structure.md](01_Directory_Structure.md)
- 🏗️ **架构设计**：[02_Architecture.md](02_Architecture.md)
- 🔌 **Native API**：[03_Native_CPP_API.md](03_Native_CPP_API.md)
- 🔒 **安全评审**：[06_Security_Review.md](06_Security_Review.md)

---

**返回导航**：[SUMMARY.md](SUMMARY.md)
