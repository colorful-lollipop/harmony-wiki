# GN 构建 Targets

> 本文档描述 Form Fwk 的 GN 构建配置，包括关键 Targets、依赖关系、产物清单

## 根目录 BUILD.gn 核心 Targets

**文件路径**: `/Volumes/lexar/code/d/work/oh/foundation/ability/form_fwk/BUILD.gn`

### 服务层核心 Targets

| Target | 类型 | 产物 | 行号 | 说明 |
|--------|------|------|------|------|
| `libfms` | ohos_shared_library | libfms.so | L143 | Form Manager Service (SA) |
| `form_manager` | ohos_shared_library | libform_manager.so | L520 | Form 管理器 IPC 接口 |
| `fmskit_native` | ohos_shared_library | libfmskit_native.so | L399 | Native SDK |
| `fmskit_provider_client` | ohos_shared_library | libfmskit_provider_client.so | L471 | Provider Client |
| `form_config` | ohos_prebuilt_etc | form_config.xml | L604 | 配置文件 |
| `formrender_service_hap` | ohos_hap | Form_Render_Service.hap | (services/) | Form 渲染服务 |

### 关键 Configs

| Config | 用途 |
|--------|------|
| `formmgr_log_config` | 日志标签定义 (FMS_LOG_TAG) |
| `fms_idl_config` | IDL 编译配置 |
| `formmgr_config` | 服务内部配置 |

## Frameworks/js/napi BUILD.gn Targets

**文件路径**: `/Volumes/lexar/code/d/work/oh/foundation/ability/form_fwk/frameworks/js/napi/BUILD.gn`

### N-API 模块 Targets

| Target | 类型 | 产物 | 安装路径 | 行号 |
|--------|------|------|----------|------|
| `formbindingdata_napi` | ohos_shared_library | libformbindingdata_napi.so | module/application | L24 |
| `formbindingdata` | ohos_shared_library | libformbindingdata.so | module/app/form | L55 |
| `forminfo_napi` | ohos_shared_library | libforminfo_napi.so | module/application | L224 |
| `forminfo` | ohos_shared_library | libforminfo.so | module/app/form | L265 |
| `formhost_napi` | ohos_shared_library | libformhost_napi.so | module/application | L308 |
| `formhost` | ohos_shared_library | libformhost.so | module/app/form | L350 |
| `formobserver` | ohos_shared_library | libformobserver.so | module/app/form | L401 |
| `formprovider_napi` | ohos_shared_library | libformprovider_napi.so | module/application | L452 |
| `formprovider` | ohos_shared_library | libformprovider.so | module/app/form | L494 |
| `formagent` | ohos_shared_library | libformagent.so | module/app/form | L546 |
| `formutil_napi` | ohos_shared_library | libformutil_napi.so | - | L593 |
| `formerror_napi` | ohos_shared_library | libformerror_napi.so | module/application | L627 |

### Stage 模型扩展 Targets

| Target | 类型 | 产物 | 安装路径 | 行号 |
|--------|------|------|----------|------|
| `formextension_napi` | ohos_shared_library | libformextension_napi.so | module/application | L115 |
| `formextensionability` | ohos_shared_library | libformextensionability.so | module/app/form | L158 |
| `formextensioncontext_napi` | ohos_shared_library | libformextensioncontext_napi.so | module/application | L201 |
| `formeditextensionability_napi` | ohos_shared_library | libformeditextensionability_napi.so | module/app/form | L685 |
| `formeditextensioncontext_napi` | ohos_shared_library | libformeditextensioncontext_napi.so | module/application | L730 |
| `liveformextensionability_napi` | ohos_shared_library | libliveformextensionability_napi.so | module/app/form | L918 |
| `liveformextensioncontext_napi` | ohos_shared_library | libliveformextensioncontext_napi.so | module/application | L963 |

### JS/ABC 编译 Targets

| Target | 类型 | 输入 | 输出 |
|--------|------|------|------|
| `gen_form_extension_abc` | es2abc_gen_abc | form_extension.js | form_extension.abc |
| `gen_form_extension_ability_abc` | es2abc_gen_abc | form_extension_ability.js | form_extension_ability.abc |
| `gen_form_edit_extension_ability_abc` | es2abc_gen_abc | form_edit_extension_ability.js | form_edit_extension_ability.abc |
| `gen_live_form_extension_ability_abc` | es2abc_gen_abc | live_form_extension_ability.js | live_form_extension_ability.abc |

## GN Feature Flags

**文件路径**: `/Volumes/lexar/code/d/work/oh/foundation/ability/form_fwk/form_fwk.gni`

### 可配置开关

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `form_fwk_form_dimension_2_3` | false | 启用 2x3 卡片尺寸 |
| `form_fwk_form_dimension_3_3` | false | 启用 3x3 卡片尺寸 |
| `form_fwk_watch_api_disable` | false | 禁用 Watch API |
| `form_fwk_dynamic_support` | false | 启用动态 SA 支持 |
| `device_usage_statistics` | true | 设备使用统计 |
| `cite_memmgr` | false | 内存管理引用 |
| `res_schedule_service` | false | 资源调度服务 |
| `theme_mgr_enable` | false | 主题管理器 |
| `form_runtime_power` | true | 电源管理支持 |
| `hiappevent_global_part_enabled` | false | HiAppEvent 全局 |

## 核心 Target 依赖关系

```
libfms (SA)
├── :form_config
├── :form_manager
│   ├── :form_host_delegate_proxy
│   ├── :form_host_delegate_stub
│   ├── :form_provider_delegate_proxy
│   └── :form_provider_delegate_stub
└── external_deps (48+)

form_manager
├── :form_host_delegate_proxy
├── :form_host_delegate_stub
├── :form_provider_delegate_proxy
├── :form_provider_delegate_stub
└── external_deps

fmskit_native
├── :form_manager
└── external_deps (10+)

form_napi_packages
├── formbindingdata_napi/bindingdata
├── forminfo_napi/info
├── formhost_napi/host
├── formobserver
├── formprovider_napi/provider
├── formagent
└── form_edit_extension/live_form_extension

form_ani_packages (ETS ANI)
├── form_agent_ani
├── formHost_ani
├── formProvider_ani
├── formBindingData_ani
├── formobserver_ani
└── form_edit/live_form_extension_ani
```

## 构建命令示例

```bash
# 构建所有 Form Fwk
ninja -C out/default libfms
ninja -C out/default formrender_service_hap
ninja -C out/default form_napi_packages

# 构建特定模块
ninja -C out/default //foundation/ability/form_fwk:fmskit_native
ninja -C out/default //foundation/ability/form_fwk/frameworks/js/napi:formprovider
ninja -C out/default //foundation/ability/form_fwk/frameworks/js/napi:formhost

# 查看依赖
gn desc out/default //foundation/ability/form_fwk:libfms deps
gn desc out/default //foundation/ability/form_fwk:form_manager deps
```
