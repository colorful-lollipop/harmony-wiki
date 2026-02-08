# 05_GN_Targets - GN 构建目标

## 目的

本文档详细介绍 `syscap_codec` 的 GN 构建目标、编译产物和构建配置。

## 适用范围

- 维护构建系统的工程师
- 需要理解编译产物的开发者

## 构建文件概览

| 文件 | 路径 | 说明 |
|------|------|------|
| BUILD.gn | 根目录 | 主构建文件 |
| config.gni | 根目录 | 构建配置参数 |
| napi/BUILD.gn | napi/ | N-API模块构建 |
| taihe/BUILD.gn | taihe/ | Taihe组构建 |
| taihe/syscap/BUILD.gn | taihe/syscap/ | Taihe模块构建 |

## 主构建目标 (BUILD.gn)

### 1. syscap_tool_bin - 命令行工具

**目标类型**: `ohos_executable`

**定义位置**: `BUILD.gn:35-67`

**输出名称**: `syscap_tool`

**源文件**:
```
src/main.c
src/syscap_tool.c
src/create_pcid.c
src/endian_internal.c
src/context_tool.c
src/common_method.c
```

**依赖**:
- `bounds_checking_function:libsec_static`
- `cJSON:cjson_static` (标准系统) 或 `//build/lite/config/component/cJSON:cjson_static` (Lite系统)

**条件编译**:
- `is_mingw`: 定义 `_POSIX_`
- `ohos_lite && ohos_kernel_type == "liteos_m"`: 定义 `PATH_MAX=1024`

**安装**: 启用 (`install_enable = true`)

---

### 2. syscap_interface_shared - 内部API动态库

**目标类型**: 
- Lite系统: `shared_library`
- 标准系统: `ohos_shared_library`

**定义位置**: `BUILD.gn:73-140`

**输出名称**: `libsyscap_interface_shared.so`

**版本脚本**: `libsyscap_interface_shared.versionscript`

**源文件**:
```
interfaces/inner_api/syscap_interface.c
src/context_tool.c
src/endian_internal.c
src/syscap_tool.c
src/common_method.c
```

**依赖**:
- `bounds_checking_function:libsec_shared`
- `cJSON:cjson_static`

**链接选项** (Lite系统):
```
-rdynamic
-Wl,--version-script=${_version_script}
```

**分支保护** (标准系统): `branch_protector_ret = "pac_ret"`

---

### 3. generate_pcid - PCID生成动作

**目标类型**: `build_ext_component`

**定义位置**: `BUILD.gn:146-162`

**输出**: `$root_out_dir/pcid.sc`

**依赖**: `syscap_tool_bin_linux`

**执行逻辑**:
1. 设置 `syscap_tool` 可执行权限
2. 执行编码: `syscap_tool -P -e -i ${preload_path}/system/etc/SystemCapability.json`
3. (Lite系统) 复制到 `system/etc/pcid.sc`

---

### 4. pcid.sc - PCID预构建产物

**目标类型**: `ohos_prebuilt_etc`

**定义位置**: `BUILD.gn:164-169`

**源文件**: `$root_out_dir/pcid.sc`

**安装位置**: `system/etc/`

---

### 5. gen_syscap_define_custom - 自定义Syscap定义生成

**目标类型**: `action`

**定义位置**: `BUILD.gn:171-189`

**条件**: `syscap_codec_config_extern_path != ""`

**脚本**: `tools/syscap_config_merge.py`

**输入**:
- `include/codec_config/syscap_define.h`
- `${syscap_codec_config_extern_path}`

**输出**: `${root_build_dir}/syscap_define_custom.h`

**用途**: 支持扩展系统能力定义

---

### 6. syscap_codec - 主组

**目标类型**: `group`

**定义位置**: `BUILD.gn:195-203`

**依赖**:
- `pcid_sc`
- `syscap_interface_shared`
- `napi:systemcapability` (条件: `support_jsapi && is_standard_system`)

---

## N-API构建目标 (napi/BUILD.gn)

### systemcapability - N-API动态库

**目标类型**: `ohos_shared_library`

**定义位置**: `napi/BUILD.gn:34-67`

**输出名称**: `libsystemcapability.so`

**安装目录**: `module/` (`relative_install_dir = "module"`)

**源文件**:
```
napi/napi_query_syscap.cpp
src/syscap_tool.c
src/create_pcid.c
src/endian_internal.c
src/context_tool.c
interfaces/inner_api/syscap_interface.c
```

**依赖**:
- `query_syscap_js` (JS对象文件)
- `bounds_checking_function:libsec_static`
- `napi:ace_napi`
- `cJSON:cjson_static`

**条件编译**:
- `syscap_codec_config_extern_path != ""`: 添加 `SYSCAP_DEFINE_EXTERN_ENABLE`

---

### query_syscap_js - JS对象文件

**目标类型**: `gen_js_obj`

**定义位置**: `napi/BUILD.gn:21-24`

**输入**: `napi/query_syscap.js`

**输出**: `${target_out_dir}/query_syscap.o`

---

## Taihe构建目标 (taihe/)

### taihe_group - Taihe组

**目标类型**: `group`

**定义位置**: `taihe/BUILD.gn:16-23`

**条件**: `support_jsapi`

**依赖**:
- `syscap:systemCapability_etc`
- `syscap:systemCapability_taihe_native`

---

### copy_systemCapability - IDL复制

**目标类型**: `copy_taihe_idl`

**定义位置**: `taihe/syscap/BUILD.gn:17-19`

**源文件**: `idl/ohos.systemCapability.taihe`

---

### run_taihe - Taihe代码生成

**目标类型**: `ohos_taihe`

**定义位置**: `taihe/syscap/BUILD.gn:24-31`

**依赖**: `copy_systemCapability`

**输出**:
```
${taihe_generated_file_path}/src/ohos.systemCapability.ani.cpp
${taihe_generated_file_path}/src/ohos.systemCapability.abi.c
```

---

### systemCapability_taihe_native - Taihe动态库

**目标类型**: `taihe_shared_library`

**定义位置**: `taihe/syscap/BUILD.gn:33-70`

**输出名称**: `systemCapability_taihe_native.z.so`

**源文件**:
```
${taihe_generated_file_path}/src/ohos.systemCapability.ani.cpp
${taihe_generated_file_path}/src/ohos.systemCapability.abi.c
src/ani_constructor.cpp
src/ohos.systemCapability.impl.cpp
../../src/syscap_tool.c
../../src/create_pcid.c
../../src/endian_internal.c
../../src/common_method.c
../../src/context_tool.c
../../interfaces/inner_api/syscap_interface.c
```

**依赖**:
- `run_taihe`
- `bounds_checking_function:libsec_shared`
- `cJSON:cjson_static`

**CFI保护**:
```
cfi = true
cfi_cross_dso = true
debug = false
```

---

### systemCapability - ABC文件

**目标类型**: `generate_static_abc`

**定义位置**: `taihe/syscap/BUILD.gn:72-78`

**输入**: `${taihe_generated_file_path}/@ohos.systemCapability.ets`

**输出**: `systemCapability.abc`

**设备路径**: `/system/framework/systemCapability.abc`

**引导ABC**: `is_boot_abc = "True"`

---

### systemCapability_etc - ABC预构建

**目标类型**: `ohos_prebuilt_etc`

**定义位置**: `taihe/syscap/BUILD.gn:80-86`

**源文件**: `${target_out_dir}/systemCapability.abc`

**安装目录**: `framework/`

---

## 编译产物清单

### 可执行文件

| 产物 | 路径 | 说明 |
|------|------|------|
| `syscap_tool` | `toolchains/` | 命令行工具 |

### 动态库

| 产物 | 路径 | 说明 |
|------|------|------|
| `libsyscap_interface_shared.so` | `system/lib/` | 内部API动态库 |
| `libsystemcapability.so` | `system/lib/module/` | N-API模块 |
| `systemCapability_taihe_native.z.so` | `system/lib/` | Taihe/ANI模块 |

### 配置文件

| 产物 | 路径 | 说明 |
|------|------|------|
| `pcid.sc` | `system/etc/` | 设备系统能力描述 |
| `systemCapability.abc` | `system/framework/` | Taihe字节码 |

## 构建配置参数 (config.gni)

### 可配置参数

| 参数名 | 默认值 | 说明 |
|--------|--------|------|
| `syscap_codec_config_path` | `//developtools/syscap_codec/include/codec_config` | Syscap定义头文件路径 |
| `syscap_codec_config_extern_path` | `""` | 外部Syscap定义路径（扩展用）|

### 使用示例

在产品的 `config.gni` 中覆盖配置:

```gn
# 使用自定义的syscap定义
decalre_args() {
  syscap_codec_config_path = "//vendor/my_vendor/syscap_config"
  syscap_codec_config_extern_path = "//vendor/my_vendor/syscap_extern.h"
}
```

## 运行时加载关系

```
设备启动
    │
    ├──▶ 加载 libsyscap_interface_shared.so
    │       └── 被 N-API / ANI 模块依赖
    │
    ├──▶ 加载 libsystemcapability.so (N-API)
    │       └── 依赖 libsyscap_interface_shared.so
    │       └── 读取 /system/etc/pcid.sc
    │
    ├──▶ 加载 systemCapability_taihe_native.z.so (ANI)
    │       └── 依赖 libsyscap_interface_shared.so
    │       └── 读取 /system/etc/pcid.sc
    │
    └──▶ 加载 systemCapability.abc
            └── 由 ArkCompiler 加载
```

## 构建命令示例

### 构建完整模块

```bash
# 构建整个 syscap_codec
hb build //developtools/syscap_codec:syscap_codec

# 构建命令行工具
hb build //developtools/syscap_codec:syscap_tool_bin

# 构建内部API动态库
hb build //developtools/syscap_codec:syscap_interface_shared

# 构建N-API模块
hb build //developtools/syscap_codec/napi:systemcapability

# 构建Taihe模块
hb build //developtools/syscap_codec/taihe:taihe_group
```

### 生成PCID

```bash
# 生成 pcid.sc
hb build //developtools/syscap_codec:generate_pcid
```

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 代码组织
- [架构说明](02_Architecture.md) - 模块依赖
- [内部API](04_Inner_API.md) - 接口说明
- [配置标志附录](appendix/Config_Flags.md) - 编译宏说明
