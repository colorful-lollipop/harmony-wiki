# GN 构建文档

本文档描述 ArkXtest 各组件的 BUILD.gn 构建配置，包括 targets 列表、依赖关系和编译选项。

## 构建系统

- **构建系统**: GN (Generate Ninja)
- **配置文件**: `BUILD.gn`, `.gni`
- **根入口**: `arkxtest_config.gni`

## 全局配置

### arkxtest_config.gni

| 参数 | 值 | 说明 |
|------|-----|------|
| `arkxtest_product_feature` | `default`/`tablet`/`pc`/`phone`/`watch` | 产品特性开关 |

**产品特性宏**:

| 特性 | 宏定义 | 条件 |
|------|--------|------|
| Tablet | `ARKXTEST_TABLET_FEATURE_ENABLE` | `arkxtest_product_feature == "tablet"` |
| PC | `ARKXTEST_PC_FEATURE_ENABLE` | `arkxtest_product_feature == "pc"` |
| Watch | `ARKXTEST_WATCH_FEATURE_ENABLE` | `arkxtest_product_feature == "watch"` |
| Knuckle | `ARKXTEST_KNUCKLE_ACTION_ENABLE` | `phone` 或 `tablet` |
| AdjustWindowMode | `ARKXTEST_ADJUST_WINDOWMODE_ENABLE` | `pc`/`tablet`/`phone` |

**证据**: `arkxtest_config.gni:16-17`

---

## UiBuild 构建配置

### Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `uitest_core` | `ohos_static_library` | `libuitest_core.a` | 核心逻辑 |
| `uitest_input` | `ohos_static_library` | `libuitest_input.a` | 输入注入 |
| `uitest_record` | `ohos_static_library` | `libuitest_record.a` | 录制回放 |
| `uitest_ipc` | `ohos_static_library` | `libuitest_ipc.a` | IPC 通信 |
| `uitest_addon` | `ohos_static_library` | `libuitest_addon.a` | 扩展功能 |
| `uitest_server` | `ohos_executable` | `uitest` | 服务端守护进程 |
| `uitest_client` | `ohos_shared_library` | `libuitest.z.so` | N-API 客户端 |
| `uitest_ani` | `ohos_shared_library` | `libuitest_ani.so` | ANI 绑定 |
| `uitest_etc` | `ohos_prebuilt_etc` | `@ohos.UiTest.abc` | ABC 模块 |
| `uitest_exporter_js` | `gen_js_obj` | `.o` | JS 导出 |
| `uitest_exporter_abc` | `gen_js_obj` | `.o` | ABC 导出 |
| `gen_uitest_exporter_abc` | `es2abc_gen_abc` | `.abc` | JS 转 ABC |
| `cj_ui_test_ffi` | `ohos_shared_library` | `libcj_ui_test_ffi.z.so` | Cangjie FFI |
| `uitestkit` | `group` | - | 完整组件组 |
| `uitestkit_test` | `group` | - | 测试组 |
| `uitest_core_unittest` | `ohos_unittest` | `uitest_core_unittest` | 单元测试 |
| `uitest_ipc_unittest` | `ohos_unittest` | `uitest_ipc_unittest` | IPC 测试 |
| `uitest_extension_unittest` | `ohos_unittest` | `uitest_extension_unittest` | 扩展测试 |

**证据**: `uitest/BUILD.gn:30-449`

### uitest_core

```gn
ohos_static_library("uitest_core") {
  sources = [
    "${source_root}/core/dump_handler.cpp",
    "${source_root}/core/frontend_api_handler.cpp",
    "${source_root}/core/rect_algorithm.cpp",
    "${source_root}/core/select_strategy.cpp",
    "${source_root}/core/ui_action.cpp",
    "${source_root}/core/ui_driver.cpp",
    "${source_root}/core/ui_model.cpp",
    "${source_root}/core/widget_operator.cpp",
    "${source_root}/core/widget_selector.cpp",
    "${source_root}/core/window_operator.cpp",
  ]

  external_deps = [
    "hilog:libhilog",
    "json:nlohmann_json_static",
  ]

  defines = [
    "__OHOS__=1",
    "LOG_TAG=\"UiTestKit_Base\"",
  ]

  if (arkxtest_product_feature == "tablet") {
    defines += [ "ARKXTEST_TABLET_FEATURE_ENABLE" ]
  }
  if (arkxtest_product_feature == "pc") {
    defines += [ "ARKXTEST_PC_FEATURE_ENABLE" ]
  }
  if (arkxtest_product_feature == "phone" ||
      arkxtest_product_feature == "tablet") {
    defines += [ "ARKXTEST_KNUCKLE_ACTION_ENABLE" ]
  }
}
```

**证据**: `uitest/BUILD.gn:30-66`

### uitest_server

```gn
ohos_executable("uitest_server") {
  sources = [
    "${source_root}/server/element_node_iterator_impl.cpp",
    "${source_root}/server/server_main.cpp",
    "${source_root}/server/system_ui_controller.cpp",
  ]

  include_dirs = [
    "${source_root}/core",
    "${source_root}/connection",
    "${source_root}/record",
    "${source_root}/addon",
    "${source_root}/input",
  ]

  deps = [
    ":uitest_addon",
    ":uitest_core",
    ":uitest_input",
    ":uitest_ipc",
    ":uitest_record",
    "${source_root}/../testserver/src:test_server_client",
  ]

  external_deps = [
    "ability_runtime:ability_manager",
    "accessibility:accessibility_common",
    "accessibility:accessibleability",
    "graphic_2d:librender_service_client",
    "hisysevent:libhisysevent",
    "image_framework:image_native",
    "image_framework:image_packer",
    "init:libbegetutil",
    "input:libmmi-client",
    "ipc:ipc_single",
    "libpng:libpng",
    "window_manager:libdm",
    "window_manager:libwm",
  ]

  output_name = "uitest"
  subsystem_name = "testfwk"
  part_name = "arkxtest"
}
```

**证据**: `uitest/BUILD.gn:178-246`

### uitest_client

```gn
ohos_shared_library("uitest_client") {
  sources = [
    "${source_root}/napi/ui_event_observer_napi.cpp",
    "${source_root}/napi/uitest_napi.cpp",
  ]

  include_dirs = [
    "${source_root}/core",
    "${source_root}/connection",
  ]

  deps = [
    ":uitest_exporter_abc",
    ":uitest_exporter_js",
    ":uitest_ipc",
    "${source_root}/../testserver/src:test_server_client",
  ]

  subsystem_name = "testfwk"
  part_name = "arkxtest"
  output_name = "uitest"
  relative_install_dir = "module"
}
```

**证据**: `uitest/BUILD.gn:268-302`

---

## PerfTest 构建配置

### Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `perftest_core` | `ohos_static_library` | `libperftest_core.a` | 核心逻辑 |
| `perftest_ipc` | `ohos_static_library` | `libperftest_ipc.a` | IPC 通信 |
| `perftest_server` | `ohos_executable` | `perftest` | 服务端守护进程 |
| `perftest_client` | `ohos_shared_library` | `libperftest.z.so` | N-API 客户端 |
| `perftest_ani` | `ohos_shared_library` | `libperftest_ani.so` | ANI 绑定 |
| `perftest_etc` | `ohos_prebuilt_etc` | `@ohos.test.PerfTest.abc` | ABC 模块 |
| `perftestkit` | `group` | - | 完整组件组 |
| `perftest_unittest` | `ohos_unittest` | `perftest_unittest` | 单元测试 |

**证据**: `perftest/BUILD.gn:32-241`

### perftest_core

```gn
ohos_static_library("perftest_core") {
  sources = [
    "./core/src/frontend_api_handler.cpp",
    "./core/src/perf_test.cpp",
    "./core/src/perf_test_strategy.cpp",
    "./collection/src/data_collection.cpp",
    "./collection/src/duration_collection.cpp",
    "./collection/src/cpu_collection.cpp",
    "./collection/src/memory_collection.cpp",
    "./collection/src/app_start_time_collection.cpp",
    "./collection/src/page_switch_time_collection.cpp",
    "./collection/src/list_swipe_fps_collection.cpp",
  ]

  include_dirs = [
    "./connection/include",
    "./core/include",
    "./collection/include",
  ]

  deps = [ "../testserver/src:test_server_client" ]

  external_deps = [
    "ability_base:want",
    "common_event_service:cesfwk_innerkits",
    "ipc:ipc_core",
    "hiview:libucollection_utility",
    "hisysevent:libhisysevent",
    "hisysevent:libhisyseventmanager",
  ]
}
```

**证据**: `perftest/BUILD.gn:32-68`

---

## TestServer 构建配置

### Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `test_server_interface` | `idl_gen_interface` | Proxy/Stub 代码 | IDL 接口生成 |
| `test_server_service` | `ohos_shared_library` | `libtest_server_service.z.so` | SA 服务端 |
| `test_server_client` | `ohos_shared_library` | `libtest_server_client.z.so` | 客户端库 |

**证据**: `testserver/src/BUILD.gn:18-133`

### test_server_service

```gn
ohos_shared_library("test_server_service") {
  sources = [
    "${target_gen_dir}/test_server_interface_stub.cpp",
    "service/test_server_service.cpp",
    "${target_gen_dir}/types.cpp",
  ]

  deps_server_interface" ]

  external_depsaccess_token:lib = [ ":test = [
    "accesstoken_sdk",
    "access_token:libtokenid_sdk",
    "c_utils:utils",
    "common_event_service:cesfwk_innerkits",
    "hilog:libhilog",
    "hiview:libucollection_utility",
    "init:libbegetutil",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "soc_perf:socperf_client",
    "window_manager:session_manager_lite",
    "ability_runtime:app_manager",
  ]

  if (arkxtest_product_feature == "phone" || arkxtest_product_feature == "tablet") {
    external_deps += [ "data_share:datashare_consumer" ]
    defines += [ "ARKXTEST_KNUCKLE_ACTION_ENABLE" ]
  }

  if (defined(global_parts_info) && defined(global_parts_info.distributeddatamgr_pasteboard)) {
    external_deps += [ "pasteboard:pasteboard_client" ]
    defines += [ "ARKXTEST_PASTEBOARD_ENABLE" ]
  }
}
```

**证据**: `testserver/src/BUILD.gn:44-96`

---

## 构建命令

### UiTest 完整构建

```bash
./build.sh --product-name rk3568 --build-target uitestkit
```

### PerfTest 完整构建

```bash
./build.sh --product-name rk3568 --build-target perftestkit
```

### TestServer 构建

```bash
./build.sh --product-name rk3568 --build-target test_server_service
./build.sh --product-name rk3568 --build-target test_server_client
```

### 单元测试构建

```bash
./build.sh --product-name rk3568 --build-target uitestkit_test
./build.sh --product-name rk3568 --build-target perftest_unittest
./build.sh --product-name rk3568 --build-target testserver_unittest
```

---

## 依赖关系图

### UiTest 依赖

```
uitest_server
├── uitest_core
│   ├── hilog
│   └── json
├── uitest_ipc
│   ├── test_server_client
│   ├── ipc_core
│   └── cesfwk_innerkits
├── uitest_addon
│   ├── graphic_2d
│   ├── image_framework
│   └── window_manager
├── uitest_input
│   └── uitest_core
├── uitest_record
│   ├── uitest_core
│   └── input
└── test_server_client

uitest_client
├── uitest_ipc
│   └── test_server_client
└── test_server_client

uitest_ani
├── uitest_ipc
│   └── test_server_client
└── runtime_core
```

### PerfTest 依赖

```
perftest_server
├── perftest_core
│   └── test_server_client
└── perftest_ipc
    └── test_server_client

perftest_client
├── perftest_ipc
└── napi:ace_napi

perftest_ani
├── perftest_ipc
└── runtime_core
```

---

## 条件编译

### 产品特性配置

```gn
# Tablet 特性
if (arkxtest_product_feature == "tablet") {
  defines += [ "ARKXTEST_TABLET_FEATURE_ENABLE" ]
}

# PC 特性
if (arkxtest_product_feature == "pc") {
  defines += [ "ARKXTEST_PC_FEATURE_ENABLE" }
}

# Watch 特性（不安装）
if (arkxtest_product_feature == "watch") {
  install_enable = false
}

# Knuckle 手势（phone/tablet）
if (arkxtest_product_feature == "phone" ||
    arkxtest_product_feature == "tablet") {
  defines += [ "ARKXTEST_KNUCKLE_ACTION_ENABLE" ]
}
```

### 组件可用性配置

```gn
# DataShare（phone/tablet）
if (arkxtest_product_feature == "phone" || arkxtest_product_feature == "tablet") {
  external_deps += [ "data_share:datashare_consumer" ]
}

# Pasteboard（条件组件）
if (defined(global_parts_info.distributeddatamgr_pasteboard)) {
  external_deps += [ "pasteboard:pasteboard_client" ]
}
```

**证据**: `testserver/src/BUILD.gn:74-92`
