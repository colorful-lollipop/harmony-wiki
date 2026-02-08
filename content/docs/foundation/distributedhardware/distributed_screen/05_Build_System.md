# GN构建系统

## 目的与适用范围

本文档详细描述分布式屏幕的GN构建系统配置。

---

## 构建文件清单

### 核心构建文件

| 文件类型 | 数量 | 路径 |
|---------|------|------|
| BUILD.gn | 12个核心文件 | 见下表 |
| .gni | 1个 | `distributedscreen.gni` |
| bundle.json | 1个 | `bundle.json` |
| SA Profile JSON | 2个 | `sa_profile/4807.json`, `sa_profile/4808.json` |

### BUILD.gn文件列表

| 路径 | 说明 |
|------|------|
| `common/BUILD.gn` | 公共工具库 |
| `interfaces/innerkits/native_cpp/screen_source/BUILD.gn` | Source SDK |
| `interfaces/innerkits/native_cpp/screen_sink/BUILD.gn` | Sink SDK |
| `screenhandler/BUILD.gn` | 硬件处理器 |
| `sa_profile/BUILD.gn` | SA配置 |
| `services/screenservice/sourceservice/BUILD.gn` | Source服务 |
| `services/screenservice/sinkservice/BUILD.gn` | Sink服务 |
| `services/screentransport/screensourcetrans/BUILD.gn` | Source传输 |
| `services/screentransport/screensinktrans/BUILD.gn` | Sink传输 |
| `services/screenclient/BUILD.gn` | 屏幕客户端 |
| `services/screendemo/BUILD.gn` | Demo示例 |

---

## 构建变量配置

### distributedscreen.gni

**文件**: `distributedscreen.gni:14-26`

```gn
distributedscreen_path = "//foundation/distributedhardware/distributed_screen"
fuzz_test_path = "distributed_screen/distributed_screen"
common_path = "${distributedscreen_path}/common"
services_path = "${distributedscreen_path}/services"
interfaces_path = "${distributedscreen_path}/interfaces"

declare_args() {
  need_same_account = true
  if (!defined(global_parts_info) || !defined(
          global_parts_info.distributedhardware_distributed_hardware_adapter)) {
    need_same_account = false
  }
}
```

**变量说明**:
| 变量 | 说明 |
|------|------|
| `distributedscreen_path` | 项目根路径 |
| `common_path` | 公共模块路径 |
| `services_path` | 服务实现路径 |
| `interfaces_path` | 接口定义路径 |
| `need_same_account` | 是否支持同账号检查（特性开关）|

---

## Targets详解

### 1. distributed_screen_utils

**文件**: `common/BUILD.gn:19-66`

```gn
ohos_shared_library("distributed_screen_utils") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
    boundary_sanitize = true
    integer_overflow = true
    ubsan = true
  }
  branch_protector_ret = "pac_ret"
  
  sources = [
    "src/dscreen_hisysevent.cpp",
    "src/dscreen_json_util.cpp",
    "src/dscreen_util.cpp",
  ]
  
  external_deps = [
    "c_utils:utils",
    "distributed_hardware_fwk:distributedhardwareutils",
    "dsoftbus:softbus_client",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "init:libbegetutil",
    "ipc:ipc_core",
    "json:nlohmann_json_static",
  ]
  
  defines = [
    "HI_LOG_ENABLE",
    "DH_LOG_TAG=\"dscreenutil\"",
    "LOG_DOMAIN=0xD004140",
  ]
  
  cflags = [ "-fstack-protector-strong" ]
  
  subsystem_name = "distributedhardware"
  part_name = "distributed_screen"
}
```

| 属性 | 值 |
|------|-----|
| **Target类型** | `ohos_shared_library` |
| **输出产物** | `libdistributed_screen_utils.z.so` |
| **类型** | 基础工具库 |
| **依赖** | 12个外部组件 |
| **安全选项** | CFI、边界检查、整数溢出检查、PAC-RET、栈保护 |

---

### 2. distributed_screen_source_sdk

**文件**: `interfaces/innerkits/native_cpp/screen_source/BUILD.gn:19-67`

```gn
ohos_shared_library("distributed_screen_source_sdk") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
    boundary_sanitize = true
    integer_overflow = true
    ubsan = true
  }
  branch_protector_ret = "pac_ret"
  
  sources = [
    "src/callback/dscreen_source_callback.cpp",
    "src/callback/dscreen_source_callback_stub.cpp",
    "src/callback/dscreen_source_load_callback.cpp",
    "src/dscreen_source_handler.cpp",
    "src/dscreen_source_proxy.cpp",
  ]
  
  deps = [ "${common_path}:distributed_screen_utils" ]
  
  external_deps = [
    "c_utils:utils",
    "distributed_hardware_fwk:distributedhardwareutils",
    "hilog:libhilog",
    "hitrace:hitrace_meter",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
  ]
  
  defines = [
    "HI_LOG_ENABLE",
    "DH_LOG_TAG=\"dscreensourcesdk\"",
    "LOG_DOMAIN=0xD004140",
  ]
  
  cflags = [ "-fstack-protector-strong" ]
  
  subsystem_name = "distributedhardware"
  part_name = "distributed_screen"
}
```

| 属性 | 值 |
|------|-----|
| **Target类型** | `ohos_shared_library` |
| **输出产物** | `libdistributed_screen_source_sdk.z.so` |
| **类型** | Source端SDK |
| **deps** | distributed_screen_utils |
| **外部依赖** | 6个 |

---

### 3. distributed_screen_sink_sdk

**文件**: `interfaces/innerkits/native_cpp/screen_sink/BUILD.gn:19-66`

```gn
ohos_shared_library("distributed_screen_sink_sdk") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
    boundary_sanitize = true
    integer_overflow = true
    ubsan = true
  }
  branch_protector_ret = "pac_ret"
  
  sources = [
    "src/callback/dscreen_sink_load_callback.cpp",
    "src/dscreen_sink_handler.cpp",
    "src/dscreen_sink_proxy.cpp",
  ]
  
  deps = [ "${common_path}:distributed_screen_utils" ]
  
  external_deps = [
    "c_utils:utils",
    "distributed_hardware_fwk:distributedhardwareutils",
    "hilog:libhilog",
    "hitrace:hitrace_meter",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
  ]
  
  defines = [
    "HI_LOG_ENABLE",
    "DH_LOG_TAG=\"dscreensinksdk\"",
    "LOG_DOMAIN=0xD004140",
  ]
  
  cflags = [ "-fstack-protector-strong" ]
  
  subsystem_name = "distributedhardware"
  part_name = "distributed_screen"
}
```

| 属性 | 值 |
|------|-----|
| **Target类型** | `ohos_shared_library` |
| **输出产物** | `libdistributed_screen_sink_sdk.z.so` |
| **类型** | Sink端SDK |

---

### 4. distributed_screen_source

**文件**: `services/screenservice/sourceservice/BUILD.gn:19-118`

```gn
ohos_shared_library("distributed_screen_source") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
    boundary_sanitize = true
    integer_overflow = true
    ubsan = true
  }
  branch_protector_ret = "pac_ret"
  
  include_dirs = [
    "dscreenservice/include",
    "dscreenservice/include/callback",
    "${services_path}/screenservice/sourceservice/dscreenmgr",
    "${services_path}/screenservice/sourceservice/dscreenmgr/common/include",
    "${interfaces_path}/innerkits/native_cpp/screen_sink/include",
    "${interfaces_path}/innerkits/native_cpp/screen_sink/include/callback",
    "${interfaces_path}/innerkits/native_cpp/screen_source/include",
    "${interfaces_path}/innerkits/native_cpp/screen_source/include/callback",
    "${common_path}/include",
    "${services_path}/common/utils/include",
    "${services_path}/common/databuffer/include",
    "${services_path}/common/screen_channel/include",
    "${services_path}/screentransport/screensourceprocessor/include",
    "${services_path}/screentransport/screensourceprocessor/encoder/include",
    "${services_path}/screentransport/screensourcetrans/include",
    "${services_path}/common/imageJpeg/include",
    "${services_path}/common/decision_center/include",
  ]
  
  sources = [
    "${interfaces_path}/innerkits/native_cpp/screen_sink/src/dscreen_sink_proxy.cpp",
    "${interfaces_path}/innerkits/native_cpp/screen_source/src/dscreen_source_proxy.cpp",
    "${services_path}/common/utils/src/dscreen_fwkkit.cpp",
    "${services_path}/common/utils/src/dscreen_hidumper.cpp",
    "${services_path}/common/utils/src/dscreen_maprelation.cpp",
    "${services_path}/common/utils/src/video_param.cpp",
    "dscreenmgr/1.0/src/dscreen.cpp",
    "dscreenmgr/1.0/src/dscreen_manager.cpp",
    "dscreenmgr/2.0/src/av_sender_engine_adapter.cpp",
    "dscreenmgr/2.0/src/dscreen.cpp",
    "dscreenmgr/2.0/src/dscreen_manager.cpp",
    "dscreenmgr/common/src/screen_manager_adapter.cpp",
    "dscreenservice/src/callback/dscreen_source_callback_proxy.cpp",
    "dscreenservice/src/dscreen_source_service.cpp",
    "dscreenservice/src/dscreen_source_stub.cpp",
  ]
  
  deps = [
    "${common_path}:distributed_screen_utils",
    "${services_path}/screentransport/screensourcetrans:distributed_screen_sourcetrans",
  ]
  
  defines = [
    "HI_LOG_ENABLE",
    "DH_LOG_TAG=\"dscreensource\"",
    "LOG_DOMAIN=0xD004140",
  ]
  
  if (build_variant == "root") {
    defines += [ "DUMP_DSCREEN_FILE" ]
  }
  
  external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "av_codec:av_codec_client",
    "cJSON:cjson",
    "c_utils:utils",
    "distributed_hardware_fwk:distributed_av_sender",
    "distributed_hardware_fwk:distributedhardwareutils",
    "distributed_hardware_fwk:libdhfwk_sdk",
    "eventhandler:libeventhandler",
    "graphic_2d:2d_graphics",
    "graphic_2d:GLESv3",
    "graphic_2d:libcomposer",
    "graphic_2d:librender_service_base",
    "graphic_2d:librender_service_client",
    "graphic_surface:surface",
    "hicollie:libhicollie",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "ipc:ipc_core",
    "json:nlohmann_json_static",
    "libjpeg-turbo:turbojpeg",
    "media_foundation:media_foundation",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "window_manager:libdm",
  ]
  
  cflags = [ "-fstack-protector-strong" ]
  
  subsystem_name = "distributedhardware"
  part_name = "distributed_screen"
}
```

| 属性 | 值 |
|------|-----|
| **Target类型** | `ohos_shared_library` |
| **输出产物** | `libdistributed_screen_source.z.so` |
| **类型** | Source端SA服务 |
| **源代码数** | 15个cpp文件 |
| **Include路径** | 16个 |
| **Deps** | 2个内部target |
| **外部依赖** | 29个 |
| **条件编译** | `DUMP_DSCREEN_FILE` (root变体) |

---

### 5. distributed_screen_sink

**文件**: `services/screenservice/sinkservice/BUILD.gn:19-109`

```gn
ohos_shared_library("distributed_screen_sink") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
    boundary_sanitize = true
    integer_overflow = true
    ubsan = true
  }
  branch_protector_ret = "pac_ret"
  
  sources = [
    "${interfaces_path}/innerkits/native_cpp/screen_sink/src/dscreen_sink_proxy.cpp",
    "${interfaces_path}/innerkits/native_cpp/screen_source/src/dscreen_source_proxy.cpp",
    "${services_path}/common/utils/src/dscreen_fwkkit.cpp",
    "${services_path}/common/utils/src/dscreen_hidumper.cpp",
    "${services_path}/common/utils/src/dscreen_maprelation.cpp",
    "${services_path}/common/utils/src/video_param.cpp",
    "dscreenservice/src/dscreen_sink_service.cpp",
    "dscreenservice/src/dscreen_sink_stub.cpp",
    "screenregionmgr/1.0/src/screenregion.cpp",
    "screenregionmgr/1.0/src/screenregionmgr.cpp",
    "screenregionmgr/2.0/src/av_receiver_engine_adapter.cpp",
    "screenregionmgr/2.0/src/screenregion.cpp",
    "screenregionmgr/2.0/src/screenregionmgr.cpp",
  ]
  
  deps = [
    "${common_path}:distributed_screen_utils",
    "${services_path}/screenclient:distributed_screen_client",
    "${services_path}/screentransport/screensinktrans:distributed_screen_sinktrans",
  ]
  
  external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "av_codec:av_codec_client",
    "cJSON:cjson",
    "c_utils:utils",
    "distributed_hardware_fwk:distributed_av_receiver",
    "distributed_hardware_fwk:distributedhardwareutils",
    "distributed_hardware_fwk:libdhfwk_sdk",
    "graphic_2d:libgraphic_utils",
    "graphic_2d:librender_service_base",
    "graphic_2d:librender_service_client",
    "graphic_surface:surface",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "ipc:ipc_core",
    "json:nlohmann_json_static",
    "media_foundation:media_foundation",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "window_manager:libdm",
  ]
  
  if (build_variant == "root") {
    defines += [ "DUMP_DSCREENREGION_FILE" ]
  }
  
  subsystem_name = "distributedhardware"
  part_name = "distributed_screen"
}
```

| 属性 | 值 |
|------|-----|
| **Target类型** | `ohos_shared_library` |
| **输出产物** | `libdistributed_screen_sink.z.so` |
| **类型** | Sink端SA服务 |
| **源代码数** | 13个cpp文件 |
| **Deps** | 3个内部target |
| **条件编译** | `DUMP_DSCREENREGION_FILE` (root变体) |

---

### 6. dscreen_sa_profile

**文件**: `sa_profile/BUILD.gn:16-30`

```gn
ohos_sa_profile("dscreen_sa_profile") {
  sources = [
    "4807.json",
    "4808.json",
  ]
  part_name = "distributed_screen"
}

ohos_prebuilt_etc("dscreen.cfg") {
  relative_install_dir = "init"
  source = "dscreen.cfg"
  part_name = "distributed_screen"
  subsystem_name = "distributedhardware"
}
```

| 属性 | 值 |
|------|-----|
| **Target类型** | `ohos_sa_profile`, `ohos_prebuilt_etc` |
| **输出产物** | SA配置文件, `init/dscreen.cfg` |
| **安装路径** | `/etc/init/dscreen.cfg` |

---

## 主构建入口

### bundle.json 配置

**文件**: `bundle.json:60-91`

```json
{
  "build": {
    "sub_component": [
      "//foundation/distributedhardware/distributed_screen/common:distributed_screen_utils",
      "//foundation/distributedhardware/distributed_screen/interfaces/innerkits/native_cpp/screen_sink:distributed_screen_sink_sdk",
      "//foundation/distributedhardware/distributed_screen/interfaces/innerkits/native_cpp/screen_source:distributed_screen_source_sdk",
      "//foundation/distributedhardware/distributed_screen/services/screenclient:distributed_screen_client",
      "//foundation/distributedhardware/distributed_screen/screenhandler:distributed_screen_handler",
      "//foundation/distributedhardware/distributed_screen/services/screentransport/screensinktrans:distributed_screen_sinktrans",
      "//foundation/distributedhardware/distributed_screen/services/screentransport/screensourcetrans:distributed_screen_sourcetrans",
      "//foundation/distributedhardware/distributed_screen/services/screenservice/sinkservice:distributed_screen_sink",
      "//foundation/distributedhardware/distributed_screen/services/screenservice/sourceservice:distributed_screen_source",
      "//foundation/distributedhardware/distributed_screen/sa_profile:dscreen_sa_profile",
      "//foundation/distributedhardware/distributed_screen/sa_profile:dscreen.cfg"
    ],
    "inner_kits": [
      {
        "type": "so",
        "name": "//.../screen_sink:distributed_screen_sink_sdk",
        "header": {
          "header_base": "//.../screen_sink/include",
          "header_files": [ "idscreen_sink.h" ]
        }
      },
      {
        "type": "so",
        "name": "//.../screen_source:distributed_screen_source_sdk",
        "header": {
          "header_base": "//.../screen_source/include",
          "header_files": [ "idscreen_source.h" ]
        }
      }
    ]
  }
}
```

---

## 安全编译选项

### 所有核心Target统一配置

| 选项 | 值 | 说明 |
|------|-----|------|
| `cfi` | true | 控制流完整性 |
| `cfi_cross_dso` | true | 跨DSO的CFI |
| `boundary_sanitize` | true | 边界检查 |
| `integer_overflow` | true | 整数溢出检查 |
| `ubsan` | true | 未定义行为检查 |
| `branch_protector_ret` | "pac_ret" | ARM指针认证 |
| `cflags` | "-fstack-protector-strong" | 栈保护 |

**证据**: `common/BUILD.gn:19-27`

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
  boundary_sanitize = true
  integer_overflow = true
  ubsan = true
}
branch_protector_ret = "pac_ret"
cflags = [ "-fstack-protector-strong" ]
```

---

## 条件编译开关

### need_same_account

**定义**: `distributedscreen.gni:20-26`

```gn
declare_args() {
  need_same_account = true
  if (!defined(global_parts_info) || !defined(
          global_parts_info.distributedhardware_distributed_hardware_adapter)) {
    need_same_account = false
  }
}
```

**使用** (`screensourcetrans/BUILD.gn:67-69`):
```gn
if (need_same_account) {
  defines += [ "SUPPORT_SAME_ACCOUNT" ]
}
```

### build_variant (root)

**使用** (`sourceservice/BUILD.gn:78-80`):
```gn
if (build_variant == "root") {
  defines += [ "DUMP_DSCREEN_FILE" ]
}
```

**影响**: 在root版本中启用DUMP功能，用于调试

---

## Targets依赖关系图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              构建依赖关系                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │ distributed_screen_utils (common)                               │      │
│   │ └── 被所有其他target依赖                                        │      │
│   └────────────────────────────┬────────────────────────────────────┘      │
│                                │                                            │
│        ┌───────────────────────┼───────────────────────┐                    │
│        │                       │                       │                    │
│        ▼                       ▼                       ▼                    │
│   ┌─────────────┐        ┌─────────────┐        ┌─────────────┐            │
│   │source_sdk   │        │sink_sdk     │        │screenhandler│            │
│   └─────────────┘        └─────────────┘        └─────────────┘            │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │ distributed_screen_sourcetrans / distributed_screen_sinktrans   │      │
│   │ └── 依赖 utils                                                  │      │
│   └────────────────────────────┬────────────────────────────────────┘      │
│                                │                                            │
│        ┌───────────────────────┴───────────────────────┐                    │
│        │                                               │                    │
│        ▼                                               ▼                    │
│   ┌─────────────────────────────────┐      ┌─────────────────────┐         │
│   │ distributed_screen_source       │      │ distributed_screen  │         │
│   │ └── deps: utils + sourcetrans   │      │ _sink               │         │
│   │                                 │      │ └── deps: utils +   │         │
│   │                                 │      │     sinktrans +     │         │
│   │                                 │      │     client          │         │
│   └─────────────────────────────────┘      └─────────────────────┘         │
│                                │                       │                    │
│                                ▼                       ▼                    │
│                        ┌─────────────────────────────────────┐             │
│                        │ distributed_screen_client           │             │
│                        └─────────────────────────────────────┘             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 相关跳转

- [编译产物](06_Products.md) - 产物清单和安装路径
- [目录结构](02_Directory_Structure.md) - 模块职责说明
- [附录/配置参数](appendix/Config_Flags.md) - 详细配置说明