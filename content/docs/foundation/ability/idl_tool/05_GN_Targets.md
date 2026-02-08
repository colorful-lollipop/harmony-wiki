# OpenHarmony IDL Tool - GN 构建目标

**目的**：详细说明 GN 构建系统的 targets、类型、依赖关系和编译产物。

---

## 适用范围

本文档适用于：
- 需要理解 IDL Tool 构建配置的开发者
- 需要集成 IDL 工具到 GN 构建系统的工程师
- 需要调试构建问题的工程师

---

## GN 构建配置文件

### 主构建文件：BUILD.gn

**位置**: `/Volumes/lexar/code/d/work/oh/foundation/ability/idl_tool/BUILD.gn`

**关键配置**:
```gn
import("//build/ohos.gni")

config("idl_config") {
  include_dirs = [ "./idl_tool_2" ]
}
```

---

## 构建目标（Targets）

### 1. idl 可执行文件

**证据**: `BUILD.gn:377-392`

```gn
ohos_executable("idl") {
  sources = [
    "idl_tool_2/main.cpp",
  ] + common_sources

  configs = [ ":idl_config" ]
  use_exceptions = true

  external_deps = [ "bounds_checking_function:libsec_static" ]

  if (is_arkui_x) {
    deps = [ "//third_party/bounds_checking_function:libsec_static" ]
  }

  install_enable = false
  part_name = "idl_tool"
  subsystem_name = "ability"
}
```

**说明**:
- **类型**: `ohos_executable` - 可执行文件
- **主程序**: `idl_tool_2/main.cpp`
- **源文件**: `common_sources` 宏（包含所有 AST, Parser, Codegen, Metadata, Util 模块）
- **配置**: `:idl_config` - 包含 idl_tool_2 头文件路径
- **异常**: 启用 C++ 异常
- **外部依赖**: `bounds_checking_function` - 边界检查函数
- **安装**: 不安装到系统（`install_enable = false`）
- **组件**: `ability_idl_tool`

### 2. idl_group 目标组

**证据**: `BUILD.gn:394-399`

```gn
group("idl_group") {
  deps = [
    ":idl",
    ":idl($host_toolchain)",
  ]
}
```

**说明**:
- **类型**: `group` - 目标组
- **依赖**: `:idl` - 主可执行文件
- **host 版本**: `:idl($host_toolchain)` - 宿主机版本（用于交叉编译）

---

## common_sources 宏定义

**证据**: `BUILD.gn:20-376`

`common_sources` 包含以下源文件：

### AST 模块源文件
```gn
# 基础类型（20 个文件）
"idl_tool_2/ast/base/ast_boolean_type.cpp",
"idl_tool_2/ast/base/ast_byte_type.cpp",
"idl_tool_2/ast/base/ast_char_type.cpp",
"idl_tool_2/ast/base/ast_cstring_type.cpp",
"idl_tool_2/ast/base/ast_double_type.cpp",
"idl_tool_2/ast/base/ast_float_type.cpp",
"idl_tool_2/ast/base/ast_integer_type.cpp",
"idl_tool_2/ast/base/ast_long_type.cpp",
"idl_tool_2/ast/base/ast_short_type.cpp",
"idl_tool_2/ast/base/ast_string_type.cpp",
"idl_tool_2/ast/base/ast_string16_type.cpp",
"idl_tool_2/ast/base/ast_uchar_type.cpp",
"idl_tool_2/ast/base/ast_uint_type.cpp",
"idl_tool_2/ast/base/ast_ulong_type.cpp",
"idl_tool_2/ast/base/ast_ushort_type.cpp",
"idl_tool_2/ast/base/ast_u16string_type.cpp",

# 复合类型（12 个文件）
"idl_tool_2/ast/ast_array_type.cpp",
"idl_tool_2/ast/ast_attribute.cpp",
"idl_tool_2/ast/ast_enum_type.cpp",
"idl_tool_2/ast/ast_expr.cpp",
"idl_tool_2/ast/ast_fd_type.cpp",
"idl_tool_2/ast/ast_fdsan_type.cpp",
"idl_tool_2/ast/ast_interface_type.cpp",
"idl_tool_2/ast/ast_map_type.cpp",
"idl_tool_2/ast/ast_method.cpp",
"idl_tool_2/ast/ast_namespace.cpp",
"idl_tool_2/ast/ast_parameter.cpp",
"idl_tool_2/ast/ast_ptr_type.cpp",
"idl_tool_2/ast/ast_rawdata_type.cpp",
"idl_tool_2/ast/ast_sequenceable_type.cpp",
"idl_tool_2/ast/ast_set_type.cpp",
"idl_tool_2/ast/ast_smq_type.cpp",
"idl_tool_2/ast/ast_struct_type.cpp",
"idl_tool_2/ast/ast_union_type.cpp",

# 根节点（6 个文件）
"idl_tool_2/ast/ast.cpp",
"idl_tool_2/ast/ast_node.cpp",
"idl_tool_2/ast/ast_type.cpp",
"idl_tool_2/ast/ast_void_type.cpp",
"idl_tool_2/ast/ast_native_buffer_type.cpp",
"idl_tool_2/ast/ast_pointer_type.cpp",
"idl_tool_2/ast/ast_orderedmap_type.cpp",
"idl_tool_2/ast/ast_ptr_type.cpp",
```

### Codegen 模块源文件
```gn
# HDI 后端（20 个文件）
"idl_tool_2/codegen/HDI/hdi_code_emitter.cpp",
"idl_tool_2/codegen/HDI/hdi_code_generator.cpp",
"idl_tool_2/codegen/HDI/hdi_type_emitter.cpp",

# HDI C 后端（10 个文件）
"idl_tool_2/codegen/HDI/c/c_client_proxy_code_emitter.cpp",
"idl_tool_2/codegen/HDI/c/c_custom_types_code_emitter.cpp",
"idl_tool_2/codegen/HDI/c/c_interface_code_emitter.cpp",
"idl_tool_2/codegen/HDI/c/c_service_driver_code_emitter.cpp",
"idl_tool_2/codegen/HDI/c/c_service_stub_code_emitter.cpp",

# HDI C++ 后端（10 个文件）
"idl_tool_2/codegen/HDI/cpp/cpp_client_proxy_code_emitter.cpp",
"idl_tool_2/codegen/HDI/cpp/cpp_custom_types_code_emitter.cpp",
"idl_tool_2/codegen/HDI/cpp/cpp_interface_code_emitter.cpp",
"idl_tool_2/codegen/HDI/cpp/cpp_service_driver_code_emitter.cpp",
"idl_tool_2/codegen/HDI/cpp/cpp_service_stub_code_emitter.cpp",

# HDI Java 后端（2 个文件）
"idl_tool_2/codegen/HDI/java/java_client_interface_code_emitter.cpp",
"idl_tool_2/codegen/HDI/java/java_client_proxy_code_emitter.cpp",

# SA 后端（16 个文件）
"idl_tool_2/codegen/SA/sa_code_emitter.cpp",
"idl_tool_2/codegen/SA/sa_code_generator.cpp",
"idl_tool_2/codegen/SA/sa_type_emitter.cpp",

# SA C++ 后端（4 个文件）
"idl_tool_2/codegen/SA/cpp/sa_cpp_client_code_emitter.cpp",
"idl_tool_2/codegen/SA/cpp/sa_cpp_custom_types_code_emitter.cpp",
"idl_tool_2/codegen/SA/cpp/sa_cpp_interface_code_emitter.cpp",
"idl_tool_2/codegen/SA/cpp/sa_cpp_service_stub_code_emitter.cpp",

# SA TS 后端（2 个文件）
"idl_tool_2/codegen/SA/ts/sa_ts_client_proxy_code_emitter.cpp",
"idl_tool_2/codegen/SA/ts/sa_ts_service_stub_code_emitter.cpp",

# SA Rust 后端（2 个文件）
"idl_tool_2/codegen/SA/rust/sa_rust_interface_code_emitter.cpp",
"idl_tool_2/codegen/SA/rust/sa_rust_type_emitter.cpp",

# 基础框架（2 个文件）
"idl_tool_2/codegen/code_emitter.cpp",
"idl_tool_2/codegen/code_generator.cpp",
```

### 其他模块源文件
```gn
# Lexer 模块（3 个文件）
"idl_tool_2/lexer/lexer.cpp",
"idl_tool_2/lexer/token.cpp",

# Parser 模块（2 个文件）
"idl_tool_2/parser/parser.cpp",
"idl_tool_2/parser/intf_type_check.cpp",

# Preprocessor 模块（1 个文件）
"idl_tool_2/preprocessor/preprocessor.cpp",

# Metadata 模块（6 个文件）
"idl_tool_2/metadata/metadata_builder.cpp",
"idl_tool_2/metadata/metadata_dumper.cpp",
"idl_tool_2/metadata/metadata_reader.cpp",
"idl_tool_2/metadata/metadata_serializer.cpp",

# Hash 模块（2 个文件）
"idl_tool_2/hash/hash.cpp",
"idl_tool_2/hash/hash.h",

# Util 模块（10 个文件）
"idl_tool_2/util/autoptr.cpp",
"idl_tool_2/util/file.cpp",
"idl_tool_2/util/light_refcount_base.cpp",
"idl_tool_2/util/logger.cpp",
"idl_tool_2/util/options.cpp",
"idl_tool_2/util/string.cpp",
"idl_tool_2/util/string_builder.cpp",
"idl_tool_2/util/string_pool.cpp",

# 主程序（1 个文件）
"idl_tool_2/main.cpp"
```

---

## 外部依赖

**证据**: `bundle.json:22-31`

### OpenHarmony 组件依赖

| 组件 | 用途 | GN 依赖 |
|--------|------|---------|
| **hilog** | 日志记录 | `external_deps = [ "hilog:..." ]` 或 `deps` |
| **ipc** | IPC 框架（生成代码依赖） | `deps` |
| **samgr** | 系统能力管理器 | `deps` |
| **safwk** | 系统服务框架 | `deps` |
| **c_utils** | C 工具库 | `deps` |

**使用位置**：
- 生成的 C++ 代码包含 `#include <iremote_stub.h>`（IPC）
- 生成的 C++ 代码包含 `#include <iservice_registry.h>`（SAMGR）

### 第三方依赖

| 库 | 用途 | GN 依赖 |
|------|------|---------|
| **bounds_checking_function** | 边界检查函数 | `external_deps = [ "bounds_checking_function:libsec_static" ]` |

---

## Bundle 配置

**位置**: `/Volumes/lexar/code/d/work/oh/foundation/ability/idl_tool/bundle.json`

**关键配置**:
```json
{
  "name": "@ohos/idl_tool",
  "description": "提供自动生成Extension 服务端及客户端接口文件的能力",
  "version": "3.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "foundation/ability/idl_tool"
  },
  "component": {
    "name": "idl_tool",
    "subsystem": "ability",
    "syscap": [],
    "features": [],
    "adapted_system_type": [ "standard" ],
    "deps": {
      "components": [
        "hilog", "ipc", "samgr", "safwk", "c_utils", "bounds_checking_function"
      ],
      "third_party": []
    },
    "build": {
      "sub_component": [
        "//foundation/ability/idl_tool:idl",
        "//foundation/ability/idl_tool:idl_group"
      ],
      "inner_kits": [
        {
          "name": "//foundation/ability/idl_tool:idl"
        }
      ],
      "test": [
        "//foundation/ability/idl_tool/test/rust/moduletest:moduletest",
        "//foundation/ability/idl_tool/test/rust/unittest:unittest",
        "//foundation/ability/idl_tool/test/ts/moduletest:moduletest",
        "//foundation/ability/idl_tool/test/ts/unittest:unittest",
        "//foundation/ability/idl_tool/test/unittest:unittest",
        "//foundation/ability/idl_tool/idl_tool_2/test/unittest:unittest"
      ]
    }
  }
}
```

---

## 编译选项

### 条件编译

**证据**: `BUILD.gn:385-387`

```gn
if (is_arkui_x) {
    deps = [ "//third_party/bounds_checking_function:libsec_static" ]
}
```

**说明**:
- `is_arkui_x` - ArkUI X 平台特殊处理
- 使用系统内置的 `bounds_checking_function` 而非外部依赖

---

## 目标与产物映射

### idl 可执行文件

| 配置 | 产物路径 | 说明 |
|------|---------|------|
| Debug | `out/riscv64/ability/idl_tool/clang_x64/` | Debug 版本 |
| Release | `out/riscv64/ability/idl_tool/clang_x64/` | Release 版本 |
| host_toolchain | `out/host/linux-x86_64/...` | 宿主机版本（用于交叉编译） |

**安装**: `install_enable = false` - 不安装到系统镜像

---

## 关键 GN 变量与宏

### 预定义宏

| 宏 | 说明 | 证据 |
|------|------|------|
| `is_arkui_x` | ArkUI X 平台标识 | BUILD.gn:385 |
| `is_ohos` | OpenHarmony 系统标识 | 默认 GN 变量 |
| `target_cpu` | 目标 CPU 架构 | riscv64 等 |
| `host_toolchain` | 宿主机工具链 | 用于交叉编译 |

---

## 构建命令示例

### 构建可执行文件

```bash
# 在 OpenHarmony 源码树中构建
./build.sh --product-name rk3568 --build-target ohos_sdk

# 直接使用 GN 构建
gn gen out/Debug --args="target_cpu=\"riscv64\""
ninja -C out/Debug
```

### 交叉编译

```bash
# 为宿主机构建（用于 IDL 工具本身）
gn gen out/host --args="target_cpu=\"x86_64\" ohos_build_type=\"\""
ninja -C out/host
```

---

## 关键结论

1. **单目标产出**：主要产出 `idl` 可执行文件
2. **大源文件集**：`common_sources` 包含 100+ 个源文件
3. **依赖系统组件**：依赖 OpenHarmony 基础组件（hilog, ipc, samgr, safwk）
4. **条件编译支持**：支持 `is_arkui_x` 等平台特定配置
5. **不安装到系统**：`install_enable = false`，作为开发工具使用

---

## 相关文档

- [01_Directory_Structure.md](01_Directory_Structure.md) - 模块职责
- [02_Architecture.md](02_Architecture.md) - 架构设计
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物说明

---

**最后更新**: 2026-02-06
