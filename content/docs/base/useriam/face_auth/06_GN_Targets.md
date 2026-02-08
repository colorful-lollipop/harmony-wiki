# GN 构建目标

## Targets 汇总

| # | 路径 | Target 名称 | 类型 | 输出产物 | 主要 Sources |
|---|------|-------------|------|----------|--------------|
| 1 | common/ | `iam_log_config` | config | - | - |
| 2 | common/ | `iam_utils_config` | config | - | - |
| 3 | frameworks/js/napi/ | `faceauth` | ohos_shared_library | libfaceauth.so | face_auth_napi.cpp |
| 4 | frameworks/ipc/ | `faceauth_framework` | ohos_shared_library | libfaceauth_framework.so | client_impl.cpp, proxy.cpp |
| 5 | frameworks/ipc/ | `faceauth_framework_stub` | ohos_source_set | - | face_auth_stub.cpp |
| 6 | frameworks/ets/ani/ | `face_auth_copy_taihe_idl` | copy_taihe_idl | - | ohos.userIAM.faceAuth.taihe |
| 7 | frameworks/ets/ani/ | `face_auth_taihe` | ohos_taihe | 生成代码 | - |
| 8 | frameworks/ets/ani/ | `face_auth_taihe_abc` | generate_static_abc | ohos.userIAM.faceAuth.abc | @ohos.userIAM.faceAuth.ets |
| 9 | frameworks/ets/ani/ | `face_auth_taihe_abc_etc` | ohos_prebuilt_etc | - | ohos.userIAM.faceAuth.abc |
| 10 | frameworks/ets/ani/ | `faceauth_ani` | taihe_shared_library | libfaceauth_ani.so | 生成代码 + ani_constructor.cpp |
| 11 | frameworks/ets/ani/ | `face_auth_ani` | group | - | face_auth_taihe_abc_etc, faceauth_ani |
| 12 | services/ | `faceauthservice_source_set` | ohos_source_set | - | 8 个 cpp 文件 |
| 13 | services/ | `faceauthservice` | ohos_shared_library | libfaceauthservice.so | - |
| 14 | services_ex/ | `faceauthservice_ex_source_set` | ohos_source_set | - | 4 个 cpp 文件 |
| 15 | services_ex/ | `faceauthservice_ex` | ohos_shared_library | libfaceauthservice_ex.so | - |
| 16 | sa_profile/ | `faceauth_sa_profile` | ohos_sa_profile | 942.json | - |

## 关键 Target 详情

### 3. faceauth (JS N-API)

**BUILD.gn**: `frameworks/js/napi/BUILD.gn`

```gn
ohos_shared_library("faceauth") {
  sources = [ "src/face_auth_napi.cpp" ]

  deps = [ "../../../frameworks/ipc:faceauth_framework" ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "napi:ace_napi",
  ]

  include_dirs = [
    "../../../common/logs",
    "../../../common/inc",
    "../../../common/utils",
  ]

  relative_install_dir = "module/useriam"
  subsystem_name = "useriam"
  part_name = "face_auth"
}
```

**Sanitize 配置**:

```gn
sanitize = {
  integer_overflow = true
  ubsan = true
  boundary_sanitize = true
  cfi = true
  cfi_cross_dso = true
  debug = false
  blocklist = "../../../cfi_blocklist.txt"
}
branch_protector_ret = "pac_ret"
```

### 4. faceauth_framework (IPC)

**BUILD.gn**: `frameworks/ipc/BUILD.gn`

```gn
ohos_shared_library("faceauth_framework") {
  sources = [
    "src/face_auth_client_impl.cpp",
    "src/face_auth_proxy.cpp",
  ]

  public_configs = [ ":faceauth_framework_public_config" ]
  configs = [ "../../common:iam_log_config" ]

  include_dirs = [
    "../../common/utils",
    "../../common/inc",
  ]

  external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "access_token:libtokensetproc_shared",
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "samgr:samgr_proxy",
  ]

  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "useriam"
  part_name = "face_auth"
}
```

### 5. faceauth_framework_stub (IPC Stub)

**BUILD.gn**: `frameworks/ipc/BUILD.gn`

```gn
ohos_source_set("faceauth_framework_stub") {
  sources = [ "src/face_auth_stub.cpp" ]

  configs = [ "../../common:iam_log_config" ]
  public_configs = [ ":faceauth_framework_public_config" ]

  include_dirs = [
    "../../common/utils",
    "../../common/inc",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
  ]

  subsystem_name = "useriam"
  part_name = "face_auth"
}
```

### 12-13. faceauthservice (SA)

**BUILD.gn**: `services/BUILD.gn`

```gn
ohos_source_set("faceauthservice_source_set") {
  sources = [
    "src/face_auth_all_in_one_executor_hdi.cpp",
    "src/face_auth_driver_hdi.cpp",
    "src/face_auth_executor_callback_hdi.cpp",
    "src/face_auth_interface_adapter.cpp",
    "src/face_auth_service.cpp",
    "src/sa_command_manager.cpp",
    "src/screen_brightness_manager.cpp",
    "src/service_ex_manager.cpp",
  ]

  include_dirs = [
    "inc",
    "../common/inc",
    "../common/logs",
    "../common/utils",
  ]

  public_configs = [ ":faceauthservice_config" ]
  deps = [ "../frameworks/ipc:faceauth_framework_stub" ]

  external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "c_utils:utils",
    "drivers_interface_camera:libbuffer_producer_sequenceable_1.0",
    "drivers_interface_face_auth:libface_auth_proxy_2.0",
    "hdf_core:libhdf_utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "miscdevice:vibrator_interface_native",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "user_auth_framework:userauth_executors",
  ]

  subsystem_name = "useriam"
  part_name = "face_auth"
}

ohos_shared_library("faceauthservice") {
  deps = [ ":faceauthservice_source_set" ]
  external_deps = [ "hilog:libhilog" ]

  subsystem_name = "useriam"
  part_name = "face_auth"
}
```

## 依赖关系图

```
face_auth.gni (全局配置)
    │
    ├── common/BUILD.gn
    │   ├── iam_log_config ──────────────┐
    │   └── iam_utils_config ────────────┤
    │                                     │
    ├── frameworks/ipc/BUILD.gn           │
    │   ├── faceauth_framework ───────────┼──┐
    │   └── faceauth_framework_stub ─────┼──┤
    │                                     │  │
    ├── frameworks/js/napi/BUILD.gn       │  │
    │   └── faceauth ◄───────────────────┘  │
    │                                        │
    ├── frameworks/ets/ani/BUILD.gn          │
    │   ├── face_auth_taihe ────────────────┤
    │   ├── face_auth_taihe_abc            │
    │   ├── faceauth_ani ◄─────────────────┤
    │   └── face_auth_ani (group)          │
    │                                        │
    ├── services/BUILD.gn                    │
    │   ├── faceauthservice_source_set ◄────┘
    │   └── faceauthservice ◄──────────────┐
    │                                        │
    ├── services_ex/BUILD.gn                 │
    │   ├── faceauthservice_ex_source_set ◄─┘
    │   └── faceauthservice_ex              │
    │                                        │
    └── sa_profile/BUILD.gn
        └── faceauth_sa_profile
```

## 全局配置参数

**文件**: `face_auth.gni`

```gn
declare_args() {
  face_use_display_manager_component = true
  face_use_power_manager_component = true
  face_use_sensor_component = true
  face_auth_path = "//base/useriam/face_auth"
}
```

| 参数 | 默认值 | 条件 |
|------|--------|------|
| `face_use_display_manager_component` | true | global_parts_info 包含 display_manager |
| `face_use_power_manager_component` | true | global_parts_info 包含 power_manager |
| `face_use_sensor_component` | true | global_parts_info 包含 sensors |
