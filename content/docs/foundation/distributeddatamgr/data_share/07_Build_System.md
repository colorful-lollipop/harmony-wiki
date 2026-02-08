# Data Share 构建系统

## 目的

本文档描述 Data Share 项目的 GN 构建配置，包括 targets、依赖关系和编译产物。

## 适用范围

- 构建工程师
- 需要理解模块依赖的开发者
- 集成 Data Share 到其他项目的开发者

## 构建配置概览

### 主要构建文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `datashare.gni` | `//foundation/distributeddatamgr/data_share/datashare.gni` | 路径变量定义 |
| `BUILD.gn` (Inner API) | `//foundation/distributeddatamgr/data_share/interfaces/inner_api/BUILD.gn` | 核心库定义 |
| `BUILD.gn` (NAPI) | `//foundation/distributeddatamgr/data_share/frameworks/js/napi/BUILD.gn` | NAPI 包组合 |
| `BUILD.gn` (DataShare) | `//foundation/distributeddatamgr/data_share/frameworks/js/napi/dataShare/BUILD.gn` | DataShare NAPI |

**证据**: `datashare.gni:14` - 基础路径定义

## GN Targets 清单

### interfaces/inner_api/BUILD.gn

**路径变量** (来自 `datashare.gni`):
```gn
datashare_base_path = "//foundation/distributeddatamgr/data_share"
datashare_common_native_path = "${datashare_base_path}/frameworks/native/common"
datashare_native_consumer_path = "${datashare_base_path}/frameworks/native/consumer"
datashare_native_provider_path = "${datashare_base_path}/frameworks/native/provider"
datashare_native_permission_path = "${datashare_base_path}/frameworks/native/permission"
datashare_native_dfx_path = "${datashare_base_path}/frameworks/native/dfx"
datashare_native_proxy_path = "${datashare_base_path}/frameworks/native/proxy"
```

#### 1. datashare_consumer (共享库)

```gn
ohos_shared_library("datashare_consumer") {
    # 安全选项
    branch_protector_ret = "pac_ret"
    sanitize = {
        ubsan = true
        boundary_sanitize = true
        cfi = true
        cfi_cross_dso = true
        debug = false
    }
    
    # 源文件 (22个)
    sources = [
        # Common
        "${datashare_common_native_path}/src/call_reporter.cpp",
        "${datashare_common_native_path}/src/datashare_string_utils.cpp",
        "${datashare_common_native_path}/src/datashare_uri_utils.cpp",
        # Consumer Controller Provider
        "${datashare_native_consumer_path}/controller/provider/src/ext_special_controller.cpp",
        "${datashare_native_consumer_path}/controller/provider/src/general_controller_provider_impl.cpp",
        # Consumer Controller Service
        "${datashare_native_consumer_path}/controller/service/src/general_controller_service_impl.cpp",
        "${datashare_native_consumer_path}/controller/service/src/persistent_data_controller.cpp",
        "${datashare_native_consumer_path}/controller/service/src/published_data_controller.cpp",
        # Consumer Src
        "${datashare_native_consumer_path}/src/datashare_connection.cpp",
        "${datashare_native_consumer_path}/src/datashare_helper.cpp",
        "${datashare_native_consumer_path}/src/datashare_helper_impl.cpp",
        "${datashare_native_consumer_path}/src/dataproxy_handle.cpp",
        "${datashare_native_consumer_path}/src/datashare_proxy.cpp",
        # Proxy Src
        "${datashare_native_proxy_path}/src/ams_mgr_proxy.cpp",
        "${datashare_native_proxy_path}/src/data_proxy_observer_stub.cpp",
        "${datashare_native_proxy_path}/src/data_share_manager_impl.cpp",
        "${datashare_native_proxy_path}/src/data_share_service_proxy.cpp",
        "${datashare_native_proxy_path}/src/idata_share_client_death_observer.cpp",
        "${datashare_native_proxy_path}/src/published_data_subscriber_manager.cpp",
        "${datashare_native_proxy_path}/src/rdb_subscriber_manager.cpp",
        "${datashare_native_proxy_path}/src/proxy_data_subscriber_manager.cpp",
    ]
    
    # 导出符号控制
    version_script = "consumer/libdatashare_consumer.map"
    
    # InnerAPI 标签
    innerapi_tags = [ "platformsdk", "sasdk" ]
    
    # 依赖
    deps = [ "${datashare_innerapi_path}/common:datashare_common" ]
    
    # 外部依赖
    external_deps = [
        "ability_base:want",
        "ability_base:zuri",
        "ability_runtime:ability_connect_callback_stub",
        "ability_runtime:app_context",
        "ability_runtime:extension_manager",
        "bundle_framework:appexecfwk_core_headers",
        "c_utils:utils",
        "common_event_service:cesfwk_innerkits",
        "hilog:libhilog",
        "hisysevent:libhisysevent",
        "hitrace:hitrace_meter",
        "hitrace:libhitracechain",
        "ipc:ipc_single",
        "ipc:rpc",
        "samgr:samgr_proxy",
    ]
    
    public_external_deps = [
        "ability_runtime:dataobs_manager",
        "kv_store:distributeddata_inner",
    ]
}
```

**输出**: `libdatashare_consumer.so`

#### 2. datashare_permission (共享库)

```gn
ohos_shared_library("datashare_permission") {
    branch_protector_ret = "pac_ret"
    sanitize = { ... }  # 同上
    
    sources = [
        "${datashare_common_native_path}/src/datashare_string_utils.cpp",
        "${datashare_common_native_path}/src/serializable.cpp",
        "${datashare_native_permission_path}/src/data_share_called_config.cpp",
        "${datashare_native_permission_path}/src/data_share_permission.cpp",
        "${datashare_native_permission_path}/src/data_share_config.cpp",
        "${datashare_native_dfx_path}/src/hiview_datashare.cpp",
    ]
    
    version_script = "permission/libdatashare_permission.map"
    innerapi_tags = [ "platformsdk" ]
    
    deps = [ "${datashare_innerapi_path}/common:datashare_common" ]
    
    external_deps = [
        "ability_base:zuri",
        "ability_runtime:app_context",
        "ability_runtime:runtime",
        "access_token:libaccesstoken_sdk",
        "bundle_framework:appexecfwk_base",
        "bundle_framework:appexecfwk_core",
        "bundle_framework:libappexecfwk_common",
        "c_utils:utils",
        "common_event_service:cesfwk_innerkits",
        "hisysevent:libhisysevent",
        "kv_store:distributeddata_inner",
        "hilog:libhilog",
        "ipc:ipc_single",
        "samgr:samgr_proxy",
    ]
}
```

**输出**: `libdatashare_permission.so`

#### 3. datashare_provider (共享库)

```gn
ohos_shared_library("datashare_provider") {
    branch_protector_ret = "pac_ret"
    sanitize = { ... }  # 同上
    
    sources = [
        "${datashare_common_native_path}/src/serializable.cpp",
        "${datashare_native_provider_path}/src/datashare_ext_ability.cpp",
        "${datashare_native_provider_path}/src/datashare_ext_ability_context.cpp",
        "${datashare_native_provider_path}/src/datashare_stub.cpp",
        "${datashare_native_provider_path}/src/datashare_stub_impl.cpp",
        "${datashare_native_provider_path}/src/datashare_uv_queue.cpp",
        "${datashare_native_provider_path}/src/js_datashare_ext_ability.cpp",
        "${datashare_native_provider_path}/src/js_datashare_ext_ability_context.cpp",
        "${datashare_native_permission_path}/src/data_share_config.cpp",
        "${datashare_native_dfx_path}/src/hiview_datashare.cpp",
        "${datashare_native_provider_path}/src/sts_datashare_ext_ability.cpp",
        "${datashare_native_provider_path}/src/sts_datashare_ext_ability_context.cpp",
    ]
    
    include_dirs = [
        "${datashare_common_napi_path}/include",
        "${datashare_native_dfx_path}/include",
        "${datashare_native_permission_path}/include",
        "${datashare_base_path}/frameworks/ets/ani/include",
        "${target_gen_dir}/../../frameworks/ets/ani/src",
    ]
    
    version_script = "provider/libdatashare_provider.map"
    innerapi_tags = [ "platformsdk" ]
    
    deps = [
        "${datashare_base_path}/frameworks/ets/ani:datashare_ani_cxx",
        "${datashare_base_path}/frameworks/ets/ani:datashare_ani_rs",
        "${datashare_innerapi_path}/common:datashare_common",
        "${datashare_napi_path}/dataShare:datashare_jscommon",
    ]
    
    external_deps = [
        "ability_base:want",
        "ability_base:zuri",
        "ability_runtime:ability_connect_callback_stub",
        "ability_runtime:ability_context_native",
        "ability_runtime:abilitykit_native",
        "ability_runtime:abilitykit_utils",
        "ability_runtime:app_context",
        "ability_runtime:extensionkit_native",
        "ability_runtime:napi_common",
        "ability_runtime:ani_common",
        "ability_runtime:runtime",
        "access_token:libaccesstoken_sdk",
        "access_token:libtokenid_sdk",
        "c_utils:utils",
        "common_event_service:cesfwk_innerkits",
        "eventhandler:libeventhandler",
        "hilog:libhilog",
        "hisysevent:libhisysevent",
        "ipc:ipc_napi",
        "ipc:ipc_single",
        "kv_store:distributeddata_inner",
        "napi:ace_napi",
        "runtime_core:ani",
        "rust_cxx:cxx_cppdeps",
        "samgr:samgr_proxy",
    ]
    
    public_external_deps = [ "ability_runtime:dataobs_manager" ]
}
```

**输出**: `libdatashare_provider.so`

#### 4. datashare_ext_ability_module (共享库)

```gn
ohos_shared_library("datashare_ext_ability_module") {
    branch_protector_ret = "pac_ret"
    sanitize = { ... }  # 同上
    
    sources = [
        "${datashare_native_provider_path}/src/datashare_ext_ability_module_loader.cpp"
    ]
    
    deps = [ ":datashare_provider" ]
    
    external_deps = [
        "ability_base:want",
        "ability_runtime:abilitykit_native",
        "ability_runtime:runtime",
        "c_utils:utils",
        "common_event_service:cesfwk_innerkits",
        "hilog:libhilog",
        "ipc:ipc_napi",
        "ipc:ipc_single",
        "napi:ace_napi",
    ]
    
    relative_install_dir = "extensionability/"
}
```

**输出**: `extensionability/libdatashare_ext_ability_module.so`

#### 5. datashare_consumer_static (静态库)

```gn
ohos_static_library("datashare_consumer_static") {
    branch_protector_ret = "pac_ret"
    sanitize = { ... }  # 同上
    
    sources = datashare_consumer_sources  # 同 datashare_consumer
    
    deps = [ "${datashare_innerapi_path}/common:datashare_common_static" ]
    
    external_deps = datashare_consumer_external_deps
    
    public_external_deps = [ "kv_store:distributeddata_inner" ]
}
```

**输出**: `libdatashare_consumer_static.a`

### frameworks/js/napi/dataShare/BUILD.gn

#### 1. datashare_jscommon (共享库)

```gn
ohos_shared_library("datashare_jscommon") {
    branch_protector_ret = "pac_ret"
    sanitize = { ... }
    
    sources = [
        "${datashare_common_napi_path}/src/datashare_js_utils.cpp",
        "${datashare_common_napi_path}/src/datashare_predicates_proxy.cpp",
        "${datashare_common_napi_path}/src/datashare_result_set_proxy.cpp",
        "${datashare_common_napi_path}/src/napi_datashare_values_bucket.cpp",
        "${datashare_common_native_path}/src/datashare_string_utils.cpp",
    ]
    
    public_configs = [ ":datashare_jscommon_public_config" ]
    
    deps = [
        "${datashare_innerapi_path}/common:datashare_common",
        "${datashare_innerapi_path}:datashare_consumer",
    ]
    
    external_deps = [
        "ability_base:zuri",
        "c_utils:utils",
        "hilog:libhilog",
        "hisysevent:libhisysevent",
        "hitrace:hitrace_meter",
        "hitrace:libhitracechain",
        "ipc:ipc_napi",
        "ipc:ipc_single",
        "kv_store:distributeddata_inner",
        "napi:ace_napi",
    ]
}
```

**输出**: `libdatashare_jscommon.so`

#### 2. datashare (共享库)

```gn
ohos_shared_library("datashare") {
    branch_protector_ret = "pac_ret"
    sanitize = { ... }
    
    sources = [
        "${datashare_common_napi_path}/src/datashare_error_impl.cpp",
        "${datashare_common_native_path}/src/datashare_string_utils.cpp",
        "src/async_call.cpp",
        "src/napi_datashare_const_properties.cpp",
        "src/napi_datashare_helper.cpp",
        "src/napi_dataproxy_handle.cpp",
        "src/napi_datashare_inner_observer.cpp",
        "src/napi_datashare_observer.cpp",
        "src/native_datashare_module.cpp",
        "${datashare_napi_path}/observer/src/napi_observer.cpp",
        "${datashare_napi_path}/observer/src/napi_subscriber_manager.cpp",
    ]
    
    deps = [
        ":datashare_jscommon",
        "${datashare_innerapi_path}:datashare_consumer",
        "${datashare_innerapi_path}/common:datashare_common",
    ]
    
    external_deps = [
        "ability_base:base",
        "ability_base:want",
        "ability_base:zuri",
        "ability_runtime:abilitykit_native",
        "ability_runtime:dataobs_manager",
        "ability_runtime:extensionkit_native",
        "ability_runtime:napi_base_context",
        "ability_runtime:napi_common",
        "access_token:libtokenid_sdk",
        "c_utils:utils",
        "common_event_service:cesfwk_innerkits",
        "hilog:libhilog",
        "hitrace:hitrace_meter",
        "hitrace:libhitracechain",
        "ipc:ipc_single",
        "kv_store:distributeddata_inner",
        "libuv:uv",
        "napi:ace_napi",
    ]
    
    relative_install_dir = "module/data"
}
```

**输出**: `module/data/libdatashare.so`

#### 3. datasharepredicates (共享库)

```gn
ohos_shared_library("datasharepredicates") {
    branch_protector_ret = "pac_ret"
    sanitize = { ... }
    
    sources = [ "src/native_datashare_predicates_module.cpp" ]
    
    deps = [
        ":datashare_jscommon",
        "${datashare_innerapi_path}:datashare_consumer",
        "${datashare_innerapi_path}/common:datashare_common",
    ]
    
    external_deps = [
        "c_utils:utils",
        "hilog:libhilog",
        "ipc:ipc_single",
        "kv_store:distributeddata_inner",
        "napi:ace_napi",
    ]
    
    relative_install_dir = "module/data"
}
```

**输出**: `module/data/libdatasharepredicates.so`

## Target 汇总表

| Target | 类型 | 输出路径 | 依赖 | 说明 |
|--------|------|----------|------|------|
| datashare_consumer | shared_library | `libdatashare_consumer.so` | datashare_common | 客户端库 |
| datashare_permission | shared_library | `libdatashare_permission.so` | datashare_common | 权限库 |
| datashare_provider | shared_library | `libdatashare_provider.so` | datashare_common, datashare_jscommon, ANI | 服务端库 |
| datashare_ext_ability_module | shared_library | `extensionability/libdatashare_ext_ability_module.so` | datashare_provider | 扩展能力模块 |
| datashare_consumer_static | static_library | `libdatashare_consumer_static.a` | datashare_common_static | 客户端静态库 |
| datashare_jscommon | shared_library | `libdatashare_jscommon.so` | datashare_common, datashare_consumer | JS 公共库 |
| datashare | shared_library | `module/data/libdatashare.so` | datashare_jscommon, datashare_consumer | 主 NAPI 模块 |
| datasharepredicates | shared_library | `module/data/libdatasharepredicates.so` | datashare_jscommon | 谓词 NAPI 模块 |

## 关键编译选项

### 安全加固选项

所有共享库和静态库都启用了以下安全选项：

```gn
branch_protector_ret = "pac_ret"
sanitize = {
    ubsan = true              # 未定义行为检测
    boundary_sanitize = true  # 边界检查
    cfi = true                # 控制流完整性
    cfi_cross_dso = true      # 跨 DSO CFI
    debug = false
}
```

**证据**: `interfaces/inner_api/BUILD.gn:111-119`

### ARM 32位特殊处理

```gn
if (target_cpu == "arm") {
    cflags += [ "-DBINDER_IPC_32BIT" ]
}
```

**证据**: `interfaces/inner_api/BUILD.gn:29-31`

## 依赖关系图

```
datashare_napi_packages (group)
├── dataShare:datashare
│   ├── :datashare_jscommon
│   │   ├── //interfaces/inner_api/common:datashare_common
│   │   └── //interfaces/inner_api:datashare_consumer
│   ├── //interfaces/inner_api:datashare_consumer
│   └── //interfaces/inner_api/common:datashare_common
│
├── dataShare:datasharepredicates
│   ├── :datashare_jscommon
│   ├── //interfaces/inner_api:datashare_consumer
│   └── //interfaces/inner_api/common:datashare_common
│
├── datashare_ext_ability:*
└── datashare_ext_ability_context:*

//interfaces/inner_api:datashare_consumer
├── //interfaces/inner_api/common:datashare_common
└── external: ability_runtime, access_token, ipc, ...

//interfaces/inner_api:datashare_provider
├── //interfaces/inner_api/common:datashare_common
├── //frameworks/ets/ani:datashare_ani_cxx
├── //frameworks/ets/ani:datashare_ani_rs
└── external: ability_runtime, napi, runtime_core, ...
```

## 产物安装路径

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `libdatashare_consumer.so` | `/system/lib/` 或 `/system/lib64/` | 客户端库 |
| `libdatashare_permission.so` | `/system/lib/` 或 `/system/lib64/` | 权限库 |
| `libdatashare_provider.so` | `/system/lib/` 或 `/system/lib64/` | 服务端库 |
| `libdatashare_ext_ability_module.so` | `/system/lib/extensionability/` | 扩展能力模块 |
| `libdatashare.so` | `/system/lib/module/data/` | 主 NAPI |
| `libdatasharepredicates.so` | `/system/lib/module/data/` | 谓词 NAPI |

## bundle.json 构建配置

**文件**: `bundle.json`

```json
{
    "build": {
        "group_type": {
            "fwk_group": [
                "//foundation/distributeddatamgr/data_share/interfaces/inner_api:datashare_consumer",
                "//foundation/distributeddatamgr/data_share/interfaces/inner_api:datashare_permission",
                "//foundation/distributeddatamgr/data_share/interfaces/inner_api:datashare_provider",
                "//foundation/distributeddatamgr/data_share/interfaces/inner_api/common:datashare_common",
                "//foundation/distributeddatamgr/data_share/interfaces/inner_api:datashare_ext_ability_module",
                "//foundation/distributeddatamgr/data_share/frameworks/js/napi:datashare_napi_packages",
                "//foundation/distributeddatamgr/data_share/frameworks/cj/ffi:datashare_cj_ffi_packages",
                "//foundation/distributeddatamgr/data_share/frameworks/ets/ani:datashare_ani_rs",
                "//foundation/distributeddatamgr/data_share/frameworks/ets/ani:datashare_ani_group"
            ]
        }
    }
}
```

**证据**: `bundle.json:77-90`

## 关键结论

1. **分层构建** - 从 Inner API → Native → NAPI，层次分明
2. **安全加固** - 所有库启用 PAC-RET、UBSan、边界检查和 CFI
3. **双模式支持** - datashare_consumer 同时包含 Silent 和 Non-Silent 控制器
4. **多语言支持** - NAPI (JS)、ANI (ArkTS)、FFI (Cangjie) 分别构建
5. **符号版本控制** - 使用 `.map` 文件控制导出符号

## 相关文档

- [目录结构](02_Directory_Structure.md) - 代码组织
- [内部 API](06_Inner_API.md) - 接口定义
- [架构设计](05_Architecture.md) - 模块关系
