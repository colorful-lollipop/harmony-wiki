# 构建系统

本文档详细描述划词服务子系统的 GN 构建配置、Targets 清单、编译产物以及运行时加载关系。

## 1. 构建系统概述

### 1.1 构建工具链

划词服务子系统使用 OpenHarmony 标准 GN 构建系统：

- **构建工具**: GN (Generate Ninja)
- **构建配置**: `build/ohos.gni` + `selection_service.gni`
- **编译标准**: C++17 (`-std=c++17`)
- ** sanitizer**: 支持 CFI、UBSan、Integer Overflow、Boundary Sanitize

**证据来源**: `selection_service.gni:16-33`

```gni
declare_args() {
  word_selection_feature_test_one = true
  selectionfwk_support_pass_windowId = false
}

selection_fwk_root_path = "//foundation/systemabilitymgr/selectionfwk"
```

### 1.2 构建入口配置

| 文件路径 | 用途 |
|---------|------|
| `bundle组件配置.json` | ，声明子系统归属、依赖 |
| `selection、产物_service.gni` | 项目参数配置 |
|级构建 `BUILD.gn构建` | 各模块定义 |

**证据来源**: `bundle.json:1-96`

## 2. Targets 清单

### 2.1 服务端 Targets

| Target 名称 | 类型 | 输出文件 | 依赖组件 |
|------------|------|---------|----------|
| `selection_service` | ohos_shared_library | libselection_service.z.so | ipc, safwk, samgr, hilog, ability_runtime, etc. |
| `selection_service_cfg` | ohos_prebuilt_etc | selection_service.cfg | - |
| `selection_service_sa_profile` | ohos_prebuilt_sa_profile | 8500.json | - |
| `selection_para` | ohos_prebuilt_etc | selection_para | - |
| `selection_para_dac` | ohos_prebuilt_etc | selection_para_dac | - |

**证据来源**: `service/BUILD.gn:25-108`, `etc/init/BUILD.gn:15-20`

#### selection_service Target 详解

```gn
ohos_shared_library("selection_service") {
  # 安全加固配置
  branch_protector_ret = "pac_ret"
  sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    cfi_vcall_icall_only = true
    integer_overflow = true
    ubsan = true
  }
  
  # 编译选项
  cflags_cc = [
    "-fdata-sections",
    "-ffunction-sections",
    "-Oz",
    "-std=c++17",
  ]
  ldflags = [
    "-Wl,--as-needed",
    "-Wl,--gc-sections",
  ]
  
  # 源码文件
  sources = [
    "src/db_selection_config_repository.cpp",
    "src/selection_service.cpp",
    "src/selection_input_monitor.cpp",
    "focus_monitor/src/focus_change_listener.cpp",
    "focus_monitor/src/focus_monitor_manager.cpp",
    "src/selection_app_validator.cpp",
    "src/selection_config.cpp",
    "src/selection_config_database.cpp",
    "src/sys_selection_config_repository.cpp",
    "src/selection_config_comparator.cpp",
    "src/selection_common.cpp",
    "src/system_ability_status_change_listener.cpp",
    "../sysevent/hisysevent_adapter.cpp",
    "../utils/src/selection_timer.cpp",
  ]
  
  # 内部依赖
  deps = [
    "${selection_fwk_root_path}/interfaces/idl:selection_service_interface",
    "${selection_fwk_root_path}/common:selection_common",
    "${selection_fwk_root_path}/interfaces/idl:selection_service_stub",
    "${selection_fwk_root_path}/interfaces/idl:selection_listener_proxy",
  ]
  
  # 外部依赖
  external_deps = [
    "ability_base:want",
    "ability_runtime:ability_connect_callback_stub",
    "ability_runtime:ability_manager",
    "access_token:libaccesstoken_sdk",
    "c_utils:utils",
    "common_event_service:cesfwk_core",
    "hilog:libhilog",
    "ipc:ipc_single",
    "init:libbeget_proxy",
    "input:libmmi-client",
    "pasteboard:pasteboard_client",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "os_account:os_account_innerkits",
    "relational_store:native_rdb",
    "hisysevent:libhisysevent",
  ]
  
  # 条件依赖
  if (window_manager_use_sceneboard) {
    external_deps += [ "window_manager:libwm_lite" ]
    defines += [ "SCENE_BOARD_ENABLE" ]
  } else {
    external_deps += [ "window_manager:libwm" ]
  }
}
```

### 2.2 公共库 Targets

| Target 名称 | 类型 | 输出文件 | 用途 |
|------------|------|---------|------|
| `selection_common` | ohos_static_library | libselection_common.a | 公共工具代码 |

**证据来源**: `common/BUILD.gn`

```gn
ohos_static_library("selection_common") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    cfi_vcall_icall_only = true
  }
  cflags_cc = [
    "-fdata-sections",
    "-ffunction-sections",
    "-Os",
  ]
  
  sources = [
    "callback_handler.cpp",
    "callback_object.cpp",
    "event_checker.cpp",
    "selectionfwk_js_utils.cpp",
    "selectionmethod_trace.cpp",
    "util.cpp",
  ]
  
  public_deps = [
    "//foundation/ability/ability_runtime:ability_runtime_inner_api",
    "//foundation/systemabilitymgr/sa/profile:sa_profile",
    "//third_party/bounds_checking_function:git_commit_sha1",
  ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
  ]
}
```

### 2.3 N-API Targets

| Target 名称 | 类型 | 输出文件 | 对应 JS 模块 |
|------------|------|---------|-------------|
| `selectionmanager_napi` | ohos_shared_library | libselectionmanager_napi.so | selectionInput.SelectionManager |
| `selectionpanel_napi` | ohos_shared_library | libselectionpanel_napi.so | selectionInput.SelectionPanel |
| `selectionextensionability_napi` | ohos_shared_library | libselectionextensionability_napi.so | selectionInput.SelectionExtensionAbility |
| `selectionextensioncontext_napi` | ohos_shared_library | libselectionextensioncontext_napi.so | selectionInput.SelectionExtensionContext |

**证据来源**: `frameworks/js/napi/*/BUILD.gn`

#### selectionmanager_napi 详解

```gn
ohos_shared_library("selectionmanager_napi") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    cfi_vcall_icall_only = true
  }
  
  cflags_cc = [
    "-fdata-sections",
    "-ffunction-sections",
    "-Os",
  ]
  
  sources = [
    "js_panel.cpp",
    "js_panel.h",
    "js_selection_ability.cpp",
    "js_selection_engine_setting.cpp",
    "panel_listener_impl.cpp",
    "selection_engine_module.cpp",
  ]
  
  deps = [
    "${selection_fwk_root_path}/common:selection_common",
    "${selection_fwk_root_path}/frameworks/js/napi/selection_client:selection_client_napi",
  ]
  
  external_deps = [
    "ability_runtime:ability_runtime_core",
    "ability_runtime:napi",
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "napi:native_common",
  ]
}
```

### 2.4 客户端库 Targets

| Target 名称 | 类型 | 输出文件 | 用途 |
|------------|------|---------|------|
| `selection_client` | ohos_shared_library | libselection_client.so | Inner API 客户端库 |
| `selection_client_napi` | ohos_shared_library | libselection_client_napi.so | N-API 异步调用封装 |

**证据来源**: `interfaces/inner_kits/selection_client/BUILD.gn`

```gn
ohos_shared_library("selection_client") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    cfi_vcall_icall_only = true
    integer_overflow = true
    ubsan = true
  }
  
  version_script = "selection_client.versionscript"
  innerapi_tags = [ "platformsdk" ]
  
  sources = [
    "../../../frameworks/native/selection_client/selection_client.cpp",
  ]
  
  deps = [
    "${selection_fwk_root_path}/interfaces/idl:selection_service_proxy",
    "${selection_fwk_root_path}/interfaces/idl:selection_listener_proxy",
  ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "ipc:ipc_single",
    "ffrt:libffrt",
    "samgr:samgr_proxy",
  ]
}
```

### 2.5 IDL 接口 Targets

| Target 名称 | 类型 | 输出文件 | 说明 |
|------------|------|---------|------|
| `selection_listener_interface` | idl_gen_interface | *_proxy.cpp, *_stub.cpp | ISelectionListener 接口 |
| `selection_service_interface` | idl_gen_interface | *_proxy.cpp, *_stub.cpp | ISelectionService 接口 |
| `selection_listener_proxy` | ohos_source_set | libselection_listener_proxy.a | Listener 代理实现 |
| `selection_listener_stub` | ohos_source_set | libselection_listener_stub.a | Listener 存根实现 |
| `selection_service_proxy` | ohos_source_set | libselection_service_proxy.a | Service 代理实现 |
| `selection_service_stub` | ohos_source_set | libselection_service_stub.a | Service 存根实现 |

**证据来源**: `interfaces/idl/BUILD.gn`

### 2.6 Native Framework Targets

| Target 名称 | 类型 | 输出文件 | 说明 |
|------------|------|---------|------|
| `selection_ability` | ohos_shared_library | libselection_ability.z.so | Native SelectionAbility |
| `selection_extension_ability_native` | ohos_shared_library | libselection_extension_ability_native.z.so | Native Extension |

**证据来源**: `frameworks/native/*/BUILD.gn`

### 2.7 ETS/ArkTS Targets

| Target 名称 | 类型 | 输出文件 | 说明 |
|------------|------|---------|------|
| `selection_taihe_group` | - | - | Taihe UI 框架集成 |
| `selection_extension_ability_etc` | - | - | ETS Extension |

**证据来源**: `frameworks/ets/*/BUILD.gn`

## 3. 组件配置

### 3.1 bundle.json 配置

**证据来源**: `bundle.json:1-96`

```json
{
  "name": "@ohos/selectionfwk",
  "version": "1.0",
  "component": {
    "name": "selectionfwk",
    "subsystem": "systemabilitymgr",
    "syscap": [
      "SystemCapability.SelectionInput.Selection"
    ],
    "adapted_system_type": [
      "mini",
      "small",
      "standard"
    ],
    "rom": "5831KB",
    "ram": "5831KB",
    "deps": {
      "components": [
        "c_utils",
        "eventhandler",
        "ipc",
        "safwk",
        "hilog",
        "hitrace",
        "samgr",
        "init",
        "input",
        "napi",
        "ability_base",
        "ability_runtime",
        "access_token",
        "window_manager",
        "pasteboard",
        "relational_store",
        "resource_management",
        "graphic_2d",
        "bundle_framework",
        "ffrt",
        "config_policy",
        "os_account",
        "cJSON",
        "common_event_service",
        "hicollie",
        "hisysevent",
        "memmgr",
        "resource_schedule_service",
        "hiappevent",
        "runtime_core",
        "bounds_checking_function"
      ]
    }
  }
}
```

## 4. 编译产物清单

### 4.1 产物映射表

| Target | 产物类型 | 输出路径 | 安装路径 |
|--------|---------|---------|---------|
| selection_service | .so | out/.../libselection_service.z.so | /system/lib64/ |
| selectionmanager_napi | .so | out/.../libselectionmanager_napi.so | /system/lib64/ |
| selectionpanel_napi | .so | out/.../libselectionpanel_napi.so | /system/lib64/ |
| selectionextensionability_napi | .so | out/.../libselectionextensionability_napi.so | /system/lib64/ |
| selectionextensioncontext_napi | .so | out/.../libselectionextensioncontext_napi.so | /system/lib64/ |
| selection_client | .so | out/.../libselection_client.so | /system/lib64/ |
| selection_service_cfg | .cfg | out/.../selection_service.cfg | /system/etc/init/ |
| selection_para | - | out/.../selection_para | /system/etc/para/ |
| selection_para_dac | - | out/.../selection_para_dac | /system/etc/para/ |

### 4.2 符号导出

**selection_client.versionscript**:

```text
{
    global:
        SelectionClient*;
        GetInstance*;
    local:
        *;
};
```

**证据来源**: `interfaces/inner_kits/selection_client/selection_client.versionscript`

## 5. 运行时加载关系

### 5.1 服务加载流程

```
系统启动
    │
    ▼
Init 进程
    │
    ├── 读取 /system/etc/init/selection_service.cfg
    │
    ├── 根据配置启动 selection_service 进程
    │
    └── 加载 libselection_service.z.so
                │
                ▼
        SAMgr 注册 SA (ID: 8500)
                │
                ▼
        SelectionService::OnStart()
```

**证据来源**: `sa_profile/8500.json`

```json
{
    "process": "selection_service",
    "systemability": [{
        "name": 8500,
        "libpath": "libselection_service.z.so",
        "run-on-create": false,
        "bootphase": "BootStartPhase",
        "auto-restart": true
    }]
}
```

### 5.2 应用加载 N-API 流程

```
JS/ArkTS 应用启动
        │
        ▼
    import selectionInput
        │
        ▼
DLOpen libselectionmanager_napi.so
        │
        ▼
napi_module_register()
        │
        ▼
exports JS API
```

### 5.3 客户端调用 IPC 流程

```
JS API (getSelectionContent)
        │
        ▼
N-API (selection_client_napi)
        │
        ▼
AsyncCall (FFRT)
        │
        ▼
IPC Proxy (libselection_client.so)
        │
        ▼
Binder Driver
        │
        ▼
SelectionService (Stub)
```

## 6. 编译命令

### 6.1 全量编译

```bash
# 修改 BUILD.gn 后
./build.sh --product-name rk3568 --ccache

# 未修改 BUILD.gn
./build.sh --product-name rk3568 --ccache --fast-rebuild
```

### 6.2 单独编译

```bash
./build.sh --product-name rk3568 --ccache --build-target selectionfwk
```

### 6.3 子组件编译

```bash
# 编译特定模块
./build.sh --product-name rk3568 --build-target selection_service
./build.sh --product-name rk3568 --build-target selectionmanager_napi
```

## 7. 构建配置项

### 7.1 Feature Flags

**证据来源**: `selection_service.gni`

| 配置项 | 默认值 | 说明 |
|-------|-------|------|
| `word_selection_feature_test_one` | true | 功能测试开关 |
| `selectionfwk_support_pass_windowId` | false | 是否支持传递窗口 ID |

### 7.2 条件编译

**证据来源**: `service/BUILD.gn:99-104`

```gn
if (window_manager_use_sceneboard) {
  external_deps += [ "window_manager:libwm_lite" ]
  defines += [ "SCENE_BOARD_ENABLE" ]
} else {
  external_deps += [ "window_manager:libwm" ]
}
```

## 8. 产物验证

### 8.1 符号检查

```bash
# 检查 selection_client 导出符号
nm -D out/.../libselection_client.so | grep "Selection"
```

预期输出:
```
00000000 T SelectionClient* SelectionClient::GetInstance()
00000000 T SelectionClient* SelectionClient::IsCurrentSelectionApp(int)
...
```

### 8.2 依赖检查

```bash
# 检查 N-API 库依赖
ldd out/.../libselectionmanager_napi.so
```

预期依赖:
```
libhilog.so
libnapi_native.so
libability_runtime_core.so
libselection_client.so
...
```

---

**相关链接**:

- [返回 SUMMARY](./SUMMARY.md)
- [N-API 参考](./03_NAPI_Reference.md)
- [架构说明](./02_Architecture.md)
- [安全评审](./05_Security_Review.md)
