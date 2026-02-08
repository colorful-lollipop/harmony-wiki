# GN Targets 与编译产物

## 目的

本文档梳理 hiperf 的 GN 构建目标、配置选项和编译产物。

## 适用范围

- 构建系统开发者
- 需要定制 hiperf 的开发者
- 系统集成人员

## 构建入口

### 主构建文件

**文件**: `BUILD.gn` (673行)

**导入的配置**:
```gn
import("//build/ohos.gni")
import("./hiperf.gni")
```

### 构建配置定义

**文件**: `hiperf.gni` (69行)

**关键变量**:
```gn
hiperf_path = "//developtools/hiperf"
innerkits_path = "${hiperf_path}/interfaces/innerkits"
kits_path = "${hiperf_path}/interfaces/kits"
```

## Targets 列表

### 1. 可执行文件

#### hiperf (设备端主程序)

**定义**: `BUILD.gn:427-467`

```gn
ohos_executable("hiperf") {
  install_enable = true
  sources = [ "./src/main.cpp" ]
  deps = [
    ":adapt_mingw_sourceset",
    ":hiperf_etc",
    ":hiperf_platform_common",
    ":hiperf_platform_linux",
  ]
  external_deps = [
    "abseil-cpp:absl_container",
    "abseil-cpp:absl_cord",
    "abseil-cpp:absl_log",
    "abseil-cpp:absl_strings",
    "bounds_checking_function:libsec_shared",
    "c_utils:utils",
    "faultloggerd:libunwinder",
    "hilog:libhilog",
    "ipc:ipc_single",
  ]
  subsystem_name = "developtools"
  part_name = "hiperf"
}
```

**输出**: `hiperf`

**安装路径**: `/system/bin/hiperf`

#### hiperf_host (Host 端程序)

**定义**: `BUILD.gn:469-495`

```gn
ohos_executable("hiperf_host") {
  install_enable = false
  sources = [ "./src/main.cpp" ]
  deps = [
    ":adapt_mingw_sourceset",
    ":hiperf_platform_common",
  ]
  external_deps = [
    "bounds_checking_function:libsec_shared",
    "c_utils:utils",
  ]
  subsystem_name = "developtools"
  part_name = "hiperf"
}
```

**输出**: 
- Linux: `hiperf_host`
- Windows: `hiperf_host.exe`

### 2. 动态库

#### hiperf_client

**定义**: `interfaces/innerkits/native/hiperf_client/BUILD.gn:26-43`

```gn
ohos_shared_library("hiperf_client") {
  install_enable = true
  public_configs = [ ":hiperf_client_config" ]
  sources = [ "src/hiperf_client.cpp" ]
  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "developtools"
  part_name = "hiperf"
}
```

**输出**: `libhiperf_client.so`

**安装路径**: `/system/lib*/`

#### hiperf_local

**定义**: `interfaces/innerkits/native/hiperf_local/BUILD.gn:25-41`

```gn
ohos_shared_library("hiperf_local") {
  install_enable = true
  public_configs = [ ":hiperf_local_config" ]
  sources = [ "src/lperf.cpp" ]
  external_deps = [
    "c_utils:utils",
    "faultloggerd:libdfx_dumpcatcher",
  ]
  version_script = "hiperf_local.map"
  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "developtools"
  part_name = "hiperf"
}
```

**输出**: `libhiperf_local.so`

**安装路径**: `/system/lib*/`

#### hiperf_host_lib

**定义**: `BUILD.gn:550-560`

```gn
ohos_shared_library("hiperf_host_lib") {
  install_enable = false
  output_name = "hiperf_report"
  subsystem_name = "developtools"
  part_name = "hiperf"
}
```

**输出**:
- Linux: `libhiperf_report.so`
- Windows: `libhiperf_report.dll`

### 3. Source Sets

#### hiperf_platform_common

**定义**: `BUILD.gn:212-261`

**职责**: 平台通用代码

**Sources**:
- `command.cpp`
- `command_reporter.cpp`
- `ipc_utilities.cpp`
- `report_json_file.cpp`
- `subcommand_dump.cpp`
- `subcommand_help.cpp`
- `subcommand_report.cpp`
- `dwarf_encoding.cpp`
- `option.cpp`
- `perf_event_record.cpp`
- `perf_file_format.cpp`
- `perf_file_reader.cpp`
- `register.cpp`
- `report.cpp`
- `subcommand.cpp`
- `symbols_file.cpp`
- `unique_stack_table.cpp`
- `utilities.cpp`
- `virtual_runtime.cpp`
- `virtual_thread.cpp`

#### hiperf_platform_linux

**定义**: `BUILD.gn:267-300`

**职责**: Linux 平台专用代码

**Sources**:
- `perf_events.cpp`
- `tracked_command.cpp`
- `ring_buffer.cpp`
- `perf_file_writer.cpp`
- `subcommand_stat.cpp`
- `subcommand_record.cpp`
- `subcommand_list.cpp`
- `spe_decoder.cpp`
- `perf_pipe.cpp`

#### adapt_mingw_sourceset

**定义**: `BUILD.gn:194-210`

**职责**: Windows MinGW 适配

**Sources**: `mingw_adapter.cpp`

### 4. Group Targets

| Target | 说明 | 依赖 |
|--------|------|------|
| `hiperf_target` | 设备端构建目标 | `:hiperf` |
| `hiperf_target_all` | 全平台构建 | 设备端 + Host 端 + API |
| `hiperf_all` | 完整构建（含测试） | 所有 targets + 测试 |
| `hiperf_etc` | 配置文件 | `hiperf.cfg`, `hiperf.para`, `hiperf.para.dac` |
| `hiperf_demo` | Demo 程序 | `demo/cpp:hiperf_demo` |
| `hiperf_example_cmd` | 示例命令 | `demo/cpp:hiperf_example_cmd` |

## Feature Flags

### 定义位置

**文件**: `hiperf.gni:22-47`

### Flags 列表

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `hiperf_target_host` | bool | false | 编译 Host 版本 |
| `hiperf_target_static` | bool | false | 静态链接 |
| `hiperf_test_coverage` | bool | false | 测试覆盖率 |
| `hiperf_test_fuzz` | bool | true | Fuzz 测试 |
| `hiperf_sanitize` | bool | false | Sanitizer |
| `hiperf_check_time` | bool | false | 时间检查 |
| `hiperf_use_libunwind` | bool | false | 使用 libunwind |
| `hiperf_use_libunwinder` | bool | true | 使用 libunwinder |
| `hiperf_debug` | bool | true | 调试模式 |
| `hiperf_code_analyze` | bool | false | 代码分析 |
| `hiperf_use_syspara` | bool | true | 使用系统参数 |
| `hiperf_independent_compilation` | bool | true | 独立编译 |
| `bundle_framework_enable` | bool | false | Bundle 框架支持 |
| `ability_base_enable` | bool | false | Ability Base 支持 |
| `hiperf_sandbox_log_path_mapping` | bool | false | 沙箱路径映射 |
| `hiperf_feature_support_usr_symlink` | bool | false | /usr 符号链接支持 |

### 自动检测的 Flags

```gni
if (defined(global_parts_info) &&
    defined(global_parts_info.bundlemanager_bundle_framework)) {
  bundle_framework_enable = true
}
if (defined(global_parts_info) &&
    defined(global_parts_info.ability_ability_base)) {
  ability_base_enable = true
}
```

## 编译命令

### 基础构建

```bash
# 只编译设备端
--build-target hiperf_target

# 编译所有平台（含 Host 端）
--build-target hiperf_target_all

# 编译所有组件（含测试）
--build-target hiperf_all

# 编译单元测试
--build-target hiperf_unittest

# 编译接口测试
--build-target hiperf_interfacetest
```

### 带参数的构建

```bash
# 编译 Host 端工具（x86_64 Linux）
--gn-args "hiperf_target_host=true"

# 静态链接
--gn-args "hiperf_target_static=true"

# 开启调试
--gn-args "hiperf_debug=true"

# 代码分析
--gn-args "hiperf_code_analyze=true"
```

## 产物映射

### 设备端产物

| Target | 输出 | 安装路径 |
|--------|------|----------|
| hiperf | `hiperf` | `/system/bin/hiperf` |
| hiperf_client | `libhiperf_client.so` | `/system/lib*/` |
| hiperf_local | `libhiperf_local.so` | `/system/lib*/` |
| hiperf.para | `hiperf.para` | `/system/etc/param/` |
| hiperf.para.dac | `hiperf.para.dac` | `/system/etc/param/` |
| hiperf.cfg | `hiperf.cfg` | `/system/etc/init/` |

### Host 端产物

| Target | Linux 输出 | Windows 输出 |
|--------|------------|--------------|
| hiperf_host | `hiperf_host` | `hiperf_host.exe` |
| hiperf_host_lib | `libhiperf_report.so` | `libhiperf_report.dll` |

### 输出路径

```
out/ohos-arm-release/
├── developtools/hiperf/
│   ├── hiperf                    # 设备端主程序
│   └── libhiperf_client.so       # 客户端库
├── clang_x64/developtools/hiperf/
│   ├── hiperf_host               # Linux Host 工具
│   └── libhiperf_report.so       # Linux Host 库
└── mingw_x86_64/developtools/hiperf/
    ├── hiperf_host.exe           # Windows Host 工具
    └── libhiperf_report.dll      # Windows Host 库
```

## 依赖关系

### 外部依赖

```
hiperf
├── ability_base (可选)
├── abseil-cpp
├── bounds_checking_function
├── bundle_framework (可选)
├── cJSON
├── c_utils
├── config_policy
├── faultloggerd
├── hilog
├── hisysevent
├── init
├── ipc
├── napi (系统依赖)
├── protobuf
├── samgr
└── zlib
```

### 内部依赖

```
hiperf
├── hiperf_platform_common
│   ├── support_elf
│   └── support_protobuf
├── hiperf_platform_linux
│   └── hiperf_platform_common
├── hiperf_client
│   └── (独立)
├── hiperf_local
│   └── faultloggerd:libdfx_dumpcatcher
└── adapt_mingw_sourceset
    └── faultloggerd:unwinder_host
```

## 关键结论

1. **多平台支持**: 支持设备端（ARM）和 Host 端（Linux/Windows）
2. **模块化设计**: 使用 source_set 实现代码复用
3. **Feature Flags**: 丰富的编译选项支持功能裁剪
4. **Inner API**: 通过 `innerapi_tags = ["platformsdk"]` 标记平台 API
5. **版本控制**: hiperf_local 使用 version_script 控制符号导出

## 相关跳转

- [编译产物](07_Build_Artifacts.md) - 产物详细说明
- [配置 Flags](appendix/Config_Flags.md) - 所有配置选项
- [目录结构](02_Directory_Structure.md) - 代码组织
