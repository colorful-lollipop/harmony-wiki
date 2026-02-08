# SmartPerf GN 构建配置

## 概述

SmartPerf 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统。配置文件分布在多个目录，包括根 `bundle.json`、各模块 `BUILD.gn` 和 `.gni` 配置变量文件。

**构建入口**:
- 设备端: `smartperf_device/BUILD.gn`
- Host 端: `smartperf_host/BUILD.gn`
- Trace Streamer: `smartperf_host/trace_streamer/BUILD.gn`

## bundle.json 配置

**代码位置**: `bundle.json`

```json
{
  "name": "@ohos/smartperf_host",
  "component": {
    "name": "smartperf_host",
    "subsystem": "developtools",
    "adapted_system_type": ["standard"],
    "features": ["smartperf_host_device"],
    "rom": "188KB",
    "ram": "2000KB",
    "build": {
      "sub_component": [
        "//developtools/smartperf_host/smartperf_device/device_command/:SP_daemon",
        "//developtools/smartperf_host/smartperf_device/:SmartPerf",
        "//developtools/smartperf_host/smartperf_device/device_command:smartperf_daemon"
      ],
      "inner_kits": [
        {
          "header": {
            "header_base": "//developtools/smartperf_host/smartperf_device/device_command/interface",
            "header_files": [
              "GameServicePlugin.h",
              "GameEventCallback.h",
              "GpuCounterCallback.h"
            ]
          },
          "name": "//developtools/smartperf_host/smartperf_device/device_command:smartperf_daemon"
        }
      ],
      "test": [
        "//developtools/smartperf_host/smartperf_device/device_command/test:unittest"
      ]
    }
  }
}
```

## 设备端构建配置

### 根 BUILD.gn

**代码位置**: `smartperf_device/BUILD.gn`

```gn
import("//build/ohos.gni")
import("./build/config.gni")

group("SmartPerf") {
  deps = []
  if (support_jsapi && smartperf_host_device) {
    deps += [
      "//developtools/smartperf_host/smartperf_device/device_ui/:SmartPerf",
    ]
  }
}
```

### device_command BUILD.gn

**代码位置**: `smartperf_device/device_command/BUILD.gn`

#### SP_daemon 可执行文件

```gn
ohos_executable("SP_daemon") {
  sources = [
    "collector/src/AI_schedule.cpp",
    "collector/src/ByTrace.cpp",
    "collector/src/CPU.cpp",
    "collector/src/Capture.cpp",
    "collector/src/DDR.cpp",
    "collector/src/Dubai.cpp",
    "collector/src/FPS.cpp",
    "collector/src/FileDescriptor.cpp",
    "collector/src/GPU.cpp",
    "collector/src/GameEvent.cpp",
    "collector/src/GpuCounter.cpp",
    "collector/src/GpuCounterCallback.cpp",
    "collector/src/Network.cpp",
    "collector/src/Power.cpp",
    "collector/src/RAM.cpp",
    "collector/src/Temperature.cpp",
    "collector/src/Threads.cpp",
    "collector/src/cpu_info.cpp",
    "collector/src/hiperf.cpp",
    "collector/src/lock_frequency.cpp",
    "collector/src/effective.cpp",
    "collector/src/navigation.cpp",
    "collector/src/parse_slide_fps_trace.cpp",
    "collector/src/sdk_data_recv.cpp",
    "cmds/src/client_control.cpp",
    "cmds/src/control_call_cmd.cpp",
    "cmds/src/editor_command.cpp",
    "cmds/src/smartperf_command.cpp",
    "utils/src/GetLog.cpp",
    "utils/src/service_plugin.cpp",
    "utils/src/sp_log.cpp",
    "utils/src/sp_utils.cpp",
    "utils/src/startup_delay.cpp",
    "utils/src/sp_profiler_factory.cpp",
    "scenarios/src/parse_click_complete_trace.cpp",
    "scenarios/src/parse_click_response_trace.cpp",
    "scenarios/src/parse_radar.cpp",
    "scenarios/src/stalling_rate_trace.cpp",
    "services/ipc/src/sp_server_socket.cpp",
    "services/ipc/src/sp_thread_socket.cpp",
    "services/task_mgr/src/sp_task.cpp",
    "heartbeat.cpp",
    "smartperf_main.cpp",
  ]

  sources += [
    "services/task_mgr/src/argument_parser.cpp",
    "services/task_mgr/src/task_manager.cpp",
    "services/task_mgr/src/thread_pool.cpp",
  ]

  cflags = [
    "-O2",
    "-ffunction-sections",
    "-fdata-sections",
    "-fvisibility=hidden",
    "-flto",
  ]
  ldflags = [
    "-Wl,--gc-sections",
    "-flto",
  ]
  public_configs = [ ":public_config" ]
  configs = [ ":config" ]
  deps = [
    ":smartperf_daemon"
  ]
  subsystem_name = "${OHOS_SMARTPERF_DEVICE_SUBSYS_NAME}"
  part_name = "${OHOS_SMARTPERF_DEVICE_PART_NAME}"
  
  external_deps = [
    "ability_base:want",
    "c_utils:utils",
    "common_event_service:cesfwk_innerkits",
    "graphic_2d:librender_service_base",
    "graphic_2d:librender_service_client",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hiview:libucollection_utility",
    "image_framework:image_native",
    "init:libbegetutil",
    "ipc:ipc_core",
    "libpng:libpng",
    "samgr:samgr_proxy",
    "window_manager:libdm",
    "window_manager:libwm",
  ]
  defines = [
    "HI_LOG_ENABLE",
    "LOG_DOMAIN = 0xD004100",
  ]
  if (smartperf_arkxtest_able) {
    external_deps += [ "arkxtest:test_server_client" ]
    defines += [ "ARKTEST_ENABLE" ]
  }
}
```

#### smartperf_daemon 共享头文件

```gn
ohos_shared_headers("smartperf_daemon") {
  include_dirs = [
    "interface"
  ]
  subsystem_name = "${OHOS_SMARTPERF_DEVICE_SUBSYS_NAME}"
  part_name = "${OHOS_SMARTPERF_DEVICE_PART_NAME}"
}
```

#### 公共配置

```gn
config("public_config") {
  include_dirs = [
    ".",
    "include",
    "interface",
    "cmds",
    "cmds/include",
    "collector",
    "collector/include",
    "scenarios",
    "scenarios/include",
    "services/ipc",
    "services/ipc/include",
    "services/task_mgr",
    "services/task_mgr/include",
    "utils",
    "utils/include",
  ]
}

config("config") {
  visibility = [ ":*" ]
  cflags = [
    "-Wall",
    "-Werror",
    "-g3",
    "-Wunused-variable",
    "-Wno-unused-but-set-variable",
  ]
  cflags_cc = [ "-fexceptions" ]
}
```

### device_ui BUILD.gn

**代码位置**: `smartperf_device/device_ui/BUILD.gn`

```gn
ohos_hap("SmartPerf") {
  hap_profile = "entry/src/main/module.json"
  certificate_profile = "signature/openharmony_smartperf.p7b"
  module_install_dir = "app/com.ohos.gameperceptio"
  js_build_mode = "debug"
  
  deps = [
    ":smartperf_js_assets",
    ":smartperf_resources",
  ]
}

ohos_js_assets("smartperf_js_assets") {
  source_dir = "entry/src/main/ets"
  ets2abc = true
}

ohos_app_scope("smartperf_app_profile") {
  app_profile = "AppScope/app.json"
  sources = [ "AppScope/resources" ]
}

ohos_resources("smartperf_resources") {
  sources = [ "entry/src/main/resources" ]
  deps = [ ":smartperf_app_profile" ]
  hap_profile = "entry/src/main/module.json"
}
```

### config.gni 变量

**代码位置**: `smartperf_device/build/config.gni`

```gn
OHOS_SMARTPERF_HOST_DIR = get_path_info("../..", "abspath")
OHOS_SMARTPERF_DEVICE_SUBSYS_NAME = "developtools"
OHOS_SMARTPERF_DEVICE_PART_NAME = "smartperf_host"
OHOS_SMARTPERF_DEVICE_TEST_MODULE_OUTPUT_PATH = "smartperf_host/smartperf_device"

declare_args() {
  smartperf_host_device = false
  smartperf_arkxtest_able = false
}
```

## Trace Streamer 构建配置

### 主 BUILD.gn

**代码位置**: `smartperf_host/trace_streamer/BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/templates/libs/libs.gni")
import("build/ts.gni")

group("trace_streamer") {
  if (is_test) {
    deps = [ "test:unittest" ]
  } else if (is_fuzz) {
    deps = [ "test:fuzztest" ]
  } else if (use_wasm) {
    deps = [ "src:trace_streamer_builtin" ]
  } else {
    deps = [ "src:trace_streamer" ]
  }
}
```

### src BUILD.gn

**代码位置**: `smartperf_host/trace_streamer/src/BUILD.gn`

#### trace_streamer 可执行文件

```gn
ohos_executable("trace_streamer") {
  deps = [ ":trace_streamer_source" ]
}
```

#### trace_streamer_source 源集合

```gn
ohos_source_set("trace_streamer_source") {
  sources = [
    "cfg/trace_streamer_config.cpp",
    "rpc/ffrt_converter.cpp",
    "rpc/rpc_server.cpp",
    "trace_streamer/trace_streamer_filters.cpp",
    "trace_streamer/trace_streamer_selector.cpp",
    "version.cpp",
  ]
  if (!is_test && !is_fuzz) {
    sources += [ "main.cpp" ]
  }
  if (use_wasm) {
    sources += [ "rpc/wasm_func.cpp" ]
  }
  
  deps = [
    ":ts_sqlite",
    "base:base",
    "filter:filter",
    "metrics:metrics_parser",
    "parser:parser",
    "parser/hiperf_parser:libsec_static",
    "proto_reader:proto_reader",
    "table:table",
    "trace_data:trace_data",
  ]
}
```

### ts.gni 配置变量

**代码位置**: `smartperf_host/trace_streamer/build/ts.gni`

```gn
declare_args() {
  is_independent_compile = false
  hiperf_debug = true
  enable_hiperf = true
  enable_ebpf = true
  enable_native_hook = true
  enable_hilog = true
  enable_hisysevent = true
  enable_arkts = true
  enable_bytrace = true
  enable_rawtrace = true
  enable_htrace = true
  enable_ffrt = true
  enable_memory = true
  enable_hidump = true
  enable_cpudata = true
  enable_network = true
  enable_diskio = true
  enable_process = true
  enable_xpower = true
  enable_stream_extend = false
}

OHOS_PROFILER_SUBSYS_NAME = "developtools"
OHOS_PROFILER_PART_NAME = "smartperf_host"
```

## Targets 汇总

| Target | 类型 | 路径 | 依赖 | 输出 |
|--------|------|------|------|------|
| `SP_daemon` | ohos_executable | `device_command/` | `:smartperf_daemon` + 15 external_deps | 可执行文件 |
| `smartperf_daemon` | ohos_shared_headers | `device_command/` | 无 | 头文件 |
| `SmartPerf` | ohos_hap | `device_ui/` | `:smartperf_js_assets`, `:smartperf_resources` | `.hap` |
| `trace_streamer` | ohos_executable | `trace_streamer/src/` | `:trace_streamer_source` | 可执行文件 |
| `sp_daemon_ut` | ohos_unittest | `device_command/test/` | `:SP_daemon` | 测试可执行文件 |

## 依赖关系图

```
smartperf_device
├── SmartPerf (group)
│   └── device_ui:SmartPerf (ohos_hap)
│       ├── smartperf_js_assets (ohos_js_assets)
│       └── smartperf_resources (ohos_resources)
│
└── device_command/
    ├── SP_daemon (ohos_executable)
    │   ├── :smartperf_daemon
    │   ├── ability_base:want
    │   ├── common_event_service:cesfwk_innerkits
    │   ├── graphic_2d:*
    │   ├── hilog:libhilog
    │   ├── hisysevent:libhisysevent
    │   ├── ipc:ipc_core
    │   ├── samgr:samgr_proxy
    │   └── window_manager:*
    │
    └── smartperf_daemon (ohos_shared_headers)
        └── interface/* (头文件)

smartperf_host
└── trace_streamer
    └── trace_streamer (group)
        └── trace_streamer_source (ohos_source_set)
            ├── ts_sqlite
            ├── base:base
            ├── filter:filter
            ├── metrics:metrics_parser
            ├── parser:parser
            ├── proto_reader:proto_reader
            ├── table:table
            └── trace_data:trace_data
```

## 条件编译

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `smartperf_host_device` | `false` | 是否构建设备端 |
| `smartperf_arkxtest_able` | `false` | 是否启用 ArkXtest |
| `support_jsapi` | - | 是否支持 JS API（HAP 构建） |
| `use_wasm` | `false` | 是否编译 WASM |
| `is_test` | `false` | 是否构建测试 |
| `is_fuzz` | `false` | 是否构建模糊测试 |

## 相关文档

- [项目概览](00_Overview.md)
- [系统架构](01_Architecture.md)
- [编译产物](05_BuildArtifacts.md)
- [安全风险评审](06_Security.md)
