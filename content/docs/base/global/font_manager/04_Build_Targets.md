# 构建目标

## 概述

font_manager 使用 GN (Generate Ninja) 作为构建系统，本章节描述所有关键的构建 targets、依赖关系和编译配置。

## 构建配置概览

### 根构建文件

| 文件 | 用途 |
|------|------|
| `bundle.json` | 组件配置，定义子系统、组件、依赖 |
| `service/BUILD.gn` | 服务端和客户端构建配置 |
| `interfaces/js/kits/BUILD.gn` | N-API 构建配置 |
| `frameworks/fontmgr/fontmgr.gni` | 框架层共享配置 |

### 组件配置

```json
// bundle.json:38-80
{
    "component": {
        "name": "font_manager",
        "subsystem": "global",
        "syscap": [ "SystemCapability.Global.FontManager" ],
        "adapted_system_type": [ "standard" ],
        "deps": {
            "components": [
                "ability_base", "ability_runtime", "access_token",
                "bounds_checking_function", "c_utils", "cJSON",
                "eventhandler", "hilog", "hisysevent", "hitrace",
                "ipc", "napi", "safwk", "samgr", "graphic_2d",
                "runtime_core", "os_account"
            ]
        },
        "build": {
            "sub_component": [
                "//base/global/font_manager/interfaces/ani:ani_package_font_manager",
                "//base/global/font_manager/interfaces/js/kits:fontmanager",
                "//base/global/font_manager/sa_profile:font_server_profile",
                "//base/global/font_manager/service:font_service_ability"
            ]
        }
    }
}
```

## Targets 详解

### 1. fontmanager (N-API)

**文件**: `interfaces/js/kits/BUILD.gn`

| 属性 | 值 |
|------|-----|
| 类型 | ohos_shared_library |
| 输出 | libfontmanager.z.so |
| part_name | font_manager |
| subsystem_name | global |
| relative_install_dir | module |

#### Sources

```gn
sources = [
    "src/font_manager_addon.cpp",
    "src/font_manager_napi.cpp",
    "src/js_data_migration_listener.cpp",
    "src/js_func_ref_holder.cpp",
]
```

#### Dependencies

```gn
deps = [
    "../../../service:font_manager_client",
]

external_deps = [
    "ability_runtime:runtime",
    "hilog:libhilog",
    "ipc:ipc_core",
    "napi:ace_napi",
]
```

#### Public Configs

```gn
config("fontmgr_napi_config") {
    include_dirs = [
        "include",
        "../../../common/include",
    ]
}
```

### 2. font_manager_client

**文件**: `service/BUILD.gn`

| 属性 | 值 |
|------|-----|
| 类型 | ohos_shared_library |
| 输出 | libfont_manager_client.z.so |
| part_name | font_manager |
| subsystem_name | global |

#### Sources

```gn
sources = [
    "client/src/font_manager_client.cpp",
    "client/src/font_service_load_manager.cpp",
    "client/src/font_sa_load_callback.cpp",
    "client/src/data_migration_cb_agent.cpp",
    "inner_api/src/font_manager_kits.cpp",
]

# IDL 生成的代码
sources += filter_include(output_values, [ "*_proxy.cpp" ])
sources += filter_include(output_values, [ "*_callback_stub.cpp" ])
sources += filter_include(output_values, [ "*_callback_event.cpp" ])
```

#### Dependencies

```gn
deps = [ ":font_service_interface" ]

external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "hitrace:hitrace_meter",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
]
```

### 3. font_manager_server

**文件**: `service/BUILD.gn`

| 属性 | 值 |
|------|-----|
| 类型 | ohos_shared_library (sa 类型) |
| 输出 | libfont_manager_server.z.so |
| part_name | font_manager |
| subsystem_name | global |

#### Sources

```gn
sources = [ "server/src/font_manager_server.cpp" ]

# IDL 生成的代码
sources += filter_include(output_values, [ "*_stub.cpp" ])
sources += filter_include(output_values, [ "*_callback_proxy.cpp" ])
sources += filter_include(output_values, [ "*_callback_event.cpp" ])

# 框架层代码
sources += fontmgr_src
```

#### Dependencies

```gn
deps = [
    ":font_manager_client",
    ":font_service_interface",
]

external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "c_utils:utils",
    "eventhandler:libeventhandler",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
]

external_deps += fontmgr_external_deps

# 条件依赖
if (os_account_enable) {
    external_deps += [ "os_account:os_account_innerkits" ]
    defines = [ "ACCOUNT_ENABLE" ]
}
```

#### 安全加固

```gn
branch_protector_ret = "pac_ret"

sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    integer_overflow = true
    ubsan = true
}
```

### 4. font_service_interface (IDL)

**文件**: `service/BUILD.gn`

| 属性 | 值 |
|------|-----|
| 类型 | idl_gen_interface |
| part_name | font_manager |
| subsystem_name | global |

#### Sources

```gn
sources = [ "IFontService.idl" ]
sources_callback = [ "IDataMigrationCallback.idl" ]
sources_common = [
    "IDataMigrationCallbackEvent.idl",
]
```

#### 生成文件

IDL 工具将生成以下文件：
- `IFontService.h`
- `IFontService.cpp`
- `IFontService_proxy.cpp`
- `IFontService_stub.cpp`
- `IDataMigrationCallback.h`
- `IDataMigrationCallback.cpp`
- `IDataMigrationCallbackStub.cpp`
- `IDataMigrationCallbackProxy.cpp`
- `IDataMigrationCallbackEvent.h`
- `IDataMigrationCallbackEvent.cpp`

### 5. font_service_ability (Group)

**文件**: `service/BUILD.gn`

```gn
group("font_service_ability") {
    deps = [
        ":font_manager_server",
        ":font_manager_client",
        "./etc:font_sa_etc",
    ]
}
```

### 6. font_sa_etc

**文件**: `service/etc/BUILD.gn`

```gn
ohos_prebuilt_etc("font_sa_etc") {
    source = "font_manager_server.cfg"
    depfile = "font_manager_server.cfg"
    part_name = "font_manager"
    subsystem_name = "global"
}
```

### 7. font_server_profile

**文件**: `sa_profile/BUILD.gn`

```gn
ohos_prebuilt_etc("font_server_profile") {
    source = "66262.json"
    part_name = "font_manager"
    subsystem_name = "global"
}
```

### 8. ani_package_font_manager

**文件**: `interfaces/ani/BUILD.gn`

```gn
# ANI 接口构建目标
```

## 框架层配置 (fontmgr.gni)

**文件**: `frameworks/fontmgr/fontmgr.gni`

```gn
root_path = "//base/global/font_manager/frameworks/fontmgr"

fontmgr_include = [
    "$root_path/include"
]

fontmgr_src = [
    "$root_path/src/font_manager_utils.cpp",
    "$root_path/src/font_config.cpp",
    "$root_path/src/font_manager.cpp",
    "$root_path/src/font_event_publish.cpp",
    "$root_path/src/hisysevent_adapter.cpp",
    "$root_path/src/data_migration_manager.cpp"
]

fontmgr_external_deps = [
    "ability_base:want",
    "c_utils:utils",
    "cJSON:cjson",
    "common_event_service:cesfwk_innerkits",
    "hilog:libhilog",
    "graphic_2d:2d_graphics",
    "graphic_2d:rosen_text",
]
```

## 依赖关系图

```
fontmanager (N-API)
    ├── font_manager_client
    │   ├── font_service_interface (IDL)
    │   │   └── (无外部依赖)
    │   ├── c_utils:utils
    │   ├── hilog:libhilog
    │   ├── hitrace:hitrace_meter
    │   ├── ipc:ipc_core
    │   └── samgr:samgr_proxy
    ├── ability_runtime:runtime
    ├── napi:ace_napi
    └── ipc:ipc_core

font_manager_server
    ├── font_manager_client
    │   └── (同上)
    ├── font_service_interface
    │   └── (同上)
    ├── access_token:libaccesstoken_sdk
    ├── access_token:libtokenid_sdk
    ├── eventhandler:libeventhandler
    ├── hilog:libhilog
    ├── hisysevent:libhisysevent
    ├── ipc:ipc_core
    ├── safwk:system_ability_fwk
    ├── samgr:samgr_proxy
    ├── os_account:os_account_innerkits (条件)
    └── fontmgr (框架层)
        ├── ability_base:want
        ├── c_utils:utils
        ├── cJSON:cjson
        ├── common_event_service:cesfwk_innerkits
        ├── graphic_2d:2d_graphics
        └── graphic_2d:rosen_text
```

## 编译选项

### 通用编译选项

```gn
cflags_cc = [
    "-fdata-sections",
    "-ffunction-sections",
    "-fno-asynchronous-unwind-tables",
    "-fno-unwind-tables",
    "-Os",
]
```

### 服务端安全选项

```gn
# PACRET (指针认证)
branch_protector_ret = "pac_ret"

#  sanitizer 配置
sanitize = {
    boundary_sanitize = true   # 边界检查
    cfi = true                  # 控制流完整性
    cfi_cross_dso = true        # 跨 DSO CFI
    integer_overflow = true     # 整数溢出检查
    ubsan = true                # 未定义行为检查
}
```

## 相关文档

- [编译产物](05_Artifacts.md)
- [架构说明](01_Architecture.md)
- [故障排查](07_Troubleshooting.md)
