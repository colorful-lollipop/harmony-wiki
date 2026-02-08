# GN 构建系统与编译产物

## 构建配置概览

### 根配置文件

| 文件 | 用途 |
|------|------|
| `bundle.json` | OpenHarmony 组件描述文件 |
| `distributedcamera.gni` | GN 构建变量定义 |
| `BUILD.gn` | (无，各模块独立) |

### 关键路径变量 (distributedcamera.gni)

```gn
distributedcamera_path = "//foundation/distributedhardware/distributed_camera"
common_path = "${distributedcamera_path}/common"
services_path = "${distributedcamera_path}/services"
innerkits_path = "${distributedcamera_path}/interfaces/inner_kits"
```

---

## 构建目标 (Targets)

### 核心模块 (9 个)

| Target | 类型 | 输出产物 | BUILD.gn 位置 |
|--------|------|----------|---------------|
| `distributed_camera_utils` | ohos_shared_library | libdistributed_camera_utils.so | `common/BUILD.gn` |
| `distributed_camera_sink_sdk` | ohos_shared_library | libdistributed_camera_sink_sdk.so | `interfaces/inner_kits/native_cpp/camera_sink/BUILD.gn` |
| `distributed_camera_source_sdk` | ohos_shared_library | libdistributed_camera_source_sdk.so | `interfaces/inner_kits/native_cpp/camera_source/BUILD.gn` |
| `distributed_camera_client` | ohos_shared_library | libdistributed_camera_client.so | `services/cameraservice/cameraoperator/client/BUILD.gn` |
| `distributed_camera_handler` | ohos_shared_library | libdistributed_camera_handler.so | `services/cameraservice/cameraoperator/handler/BUILD.gn` |
| `distributed_camera_sink` | ohos_shared_library | libdistributed_camera_sink.so | `services/cameraservice/sinkservice/BUILD.gn` |
| `distributed_camera_source` | ohos_shared_library | libdistributed_camera_source.so | `services/cameraservice/sourceservice/BUILD.gn` |
| `distributed_camera_data_process` | ohos_shared_library | libdistributed_camera_data_process.so | `services/data_process/BUILD.gn` |
| `distributed_camera_channel` | ohos_shared_library | libdistributed_camera_channel.so | `services/channel/BUILD.gn` |

### SA 配置模块

| Target | 类型 | 输出 | BUILD.gn 位置 |
|--------|------|------|---------------|
| `dcamera_sa_profile` | ohos_sa_profile | SA 配置 (4803.json, 4804.json) | `sa_profile/BUILD.gn` |
| `dcamera.cfg` | ohos_prebuilt_etc | /init/dcamera.cfg | `sa_profile/BUILD.gn` |

---

## Target 详情

### 1. distributed_camera_utils

**类型**: `ohos_shared_library`  
**输出**: `libdistributed_camera_utils.so`  
**位置**: `common/BUILD.gn`

**Sources**:
```gn
sources = [
    "src/utils/anonymous_string.cpp",
    "src/utils/data_buffer.cpp",
    "src/utils/dcamera_buffer_handle.cpp",
    "src/utils/dcamera_hidumper.cpp",
    "src/utils/dcamera_hisysevent_adapter.cpp",
    "src/utils/dcamera_hitrace_adapter.cpp",
    "src/utils/dcamera_radar.cpp",
    "src/utils/dcamera_utils_tools.cpp",
    "src/utils/dh_log.cpp",
]
```

**Deps**: 无内部 deps  
**External Deps**:
```
c_utils:utils, distributed_hardware_fwk:distributedhardwareutils,
dsoftbus:softbus_client, ffrt:libffrt, hdf_core:libhdi,
hilog:libhilog, hisysevent:libhisysevent, hitrace:hitrace_meter,
init:libbegetutil, ipc:ipc_core, safwk:system_ability_fwk,
samgr:samgr_proxy
```

**Defines**:
```
HI_LOG_ENABLE, DH_LOG_TAG="distributedcamerautils",
LOG_DOMAIN=0xD004150
```

---

### 2. distributed_camera_source_sdk

**类型**: `ohos_shared_library`  
**输出**: `libdistributed_camera_source_sdk.so`  
**位置**: `interfaces/inner_kits/native_cpp/camera_source/BUILD.gn`

**Sources**:
```gn
sources = [
    "src/callback/dcamera_source_callback.cpp",
    "src/callback/dcamera_source_callback_stub.cpp",
    "src/dcamera_hdf_operate.cpp",
    "src/dcamera_source_handler.cpp",
    "src/dcamera_source_handler_ipc.cpp",
    "src/dcamera_source_load_callback.cpp",
    "src/distributed_camera_source_proxy.cpp",
]
```

**Deps**: `common:distributed_camera_utils`  
**External Deps**:
```
c_utils:utils, distributed_hardware_fwk:distributedhardwareutils,
drivers_interface_distributed_camera:libdistributed_camera_provider_proxy_1.1,
hdf_core:libhdf_ipc_adapter, hdf_core:libhdi, hdf_core:libpub_utils,
hilog:libhilog, ipc:ipc_core, samgr:samgr_proxy
```

---

### 3. distributed_camera_sink_sdk

**类型**: `ohos_shared_library`  
**输出**: `libdistributed_camera_sink_sdk.so`  
**位置**: `interfaces/inner_kits/native_cpp/camera_sink/BUILD.gn`

**Sources**:
```gn
sources = [
    "src/callback/dcamera_sink_callback.cpp",
    "src/callback/dcamera_sink_callback_stub.cpp",
    "src/dcamera_sink_handler.cpp",
    "src/dcamera_sink_handler_ipc.cpp",
    "src/dcamera_sink_load_callback.cpp",
    "src/distributed_camera_sink_proxy.cpp",
]
```

**Deps**: `common:distributed_camera_utils`  
**External Deps**:
```
c_utils:utils, distributed_hardware_fwk:distributedhardwareutils,
hilog:libhilog, ipc:ipc_core, samgr:samgr_proxy
```

---

### 4. distributed_camera_source

**类型**: `ohos_shared_library`  
**输出**: `libdistributed_camera_source.so`  
**位置**: `services/cameraservice/sourceservice/BUILD.gn`

**Sources** (30+ 文件):
```gn
sources = [
    "${innerkits_path}/native_cpp/camera_sink/src/distributed_camera_sink_proxy.cpp",
    "${services_path}/cameraservice/base/src/dcamera_*.cpp",
    "src/distributedcamera/*.cpp",
    "src/distributedcameramgr/dcamera_*.cpp",
    "src/distributedcameramgr/dcameracontrol/*.cpp",
    "src/distributedcameramgr/dcameradata/*.cpp",
    "src/distributedcameramgr/dcameradata/feedingsmoother/**/*.cpp",
    "src/distributedcameramgr/dcamerahdf/*.cpp",
    "src/distributedcameramgr/dcamerastate/*.cpp",
]
```

**Deps**:
```
common:distributed_camera_utils,
innerkits/native_cpp/camera_sink:distributed_camera_sink_sdk,
services/cameraservice/cameraoperator/handler:distributed_camera_handler,
services/channel:distributed_camera_channel,
services/data_process:distributed_camera_data_process
```

**External Deps**:
```
access_token:libaccesstoken_sdk,
access_token:libtokenid_sdk,
access_token:libtokensetproc_shared,
av_codec:av_codec_client, cJSON:cjson, c_utils:utils,
camera_framework:camera_framework, device_manager:devicemanagersdk,
distributed_hardware_fwk:distributed_av_receiver,
distributed_hardware_fwk:distributedhardwareutils,
distributed_hardware_fwk:libdhfwk_sdk,
drivers_interface_camera:metadata,
drivers_interface_distributed_camera:libdistributed_camera_provider_proxy_1.1,
dsoftbus:softbus_client, eventhandler:libeventhandler,
ffrt:libffrt, graphic_surface:surface,
hdf_core:libhdf_ipc_adapter, hdf_core:libhdi, hdf_core:libpub_utils,
hicollie:libhicollie, hilog:libhilog, hitrace:hitrace_meter,
ipc:ipc_core, media_foundation:media_foundation,
safwk:system_ability_fwk, samgr:samgr_proxy
```

**条件编译**:
```gn
if (build_variant == "root") {
    defines += [ "DUMP_DCAMERA_FILE" ]
}
if (os_account_camera) {
    defines += [ "OS_ACCOUNT_ENABLE" ]
}
if (!distributed_camera_common) {
    cflags = [ "-DDCAMERA_MMAP_RESERVE" ]
}
```

---

### 5. distributed_camera_sink

**类型**: `ohos_shared_library`  
**输出**: `libdistributed_camera_sink.so`  
**位置**: `services/cameraservice/sinkservice/BUILD.gn`

**Sources** (20+ 文件):
```gn
sources = [
    "${innerkits_path}/native_cpp/camera_source/src/distributed_camera_source_proxy.cpp",
    "${services_path}/cameraservice/base/src/dcamera_*.cpp",
    "src/distributedcamera/*.cpp",
    "src/distributedcameramgr/callback/*.cpp",
    "src/distributedcameramgr/dcamera_*.cpp",
    "src/distributedcameramgr/listener/*.cpp",
]
```

**Deps**:
```
common:distributed_camera_utils,
services/cameraservice/cameraoperator/client:distributed_camera_client,
services/cameraservice/cameraoperator/handler:distributed_camera_handler,
services/channel:distributed_camera_channel,
services/data_process:distributed_camera_data_process
```

**条件编译**:
```gn
if (build_variant == "root") {
    defines += [ "DUMP_DCAMERA_FILE" ]
}
if (os_account_camera) {
    defines += [ "OS_ACCOUNT_ENABLE" ]
}
if (!distributed_camera_common) {
    defines += [ "SECURITY_LEVEL_CHECK_ENABLE" ]
}
if (distributed_camera_open_stabile) {
    cflags = [ "-DDCAMERA_OPEN_STABILE" ]
}
if (device_security_level_camera) {
    external_deps += [ "device_security_level:dslm_sdk" ]
    defines += [ "DEVICE_SECURITY_LEVEL_ENABLE" ]
}
```

---

### 6. distributed_camera_channel

**类型**: `ohos_shared_library`  
**输出**: `libdistributed_camera_channel.so`  
**位置**: `services/channel/BUILD.gn`

**Sources**:
```gn
sources = [
    "${services_path}/cameraservice/base/src/dcamera_info_cmd.cpp",
    "${services_path}/cameraservice/base/src/dcamera_sink_frame_info.cpp",
    "${services_path}/cameraservice/base/src/dcamera_event_cmd.cpp",
    "src/allconnect/distributed_camera_allconnect_manager.cpp",
    "src/dcamera_channel_sink_impl.cpp",
    "src/dcamera_channel_source_impl.cpp",
    "src/dcamera_low_latency.cpp",
    "src/dcamera_softbus_adapter.cpp",
    "src/dcamera_softbus_latency.cpp",
    "src/dcamera_softbus_session.cpp",
]
```

**条件编译**:
```gn
if (distributed_camera_wakeup_enabled) {
    cflags = [ "-DDCAMERA_WAKEUP" ]
}
```

---

### 7. distributed_camera_data_process

**类型**: `ohos_shared_library`  
**输出**: `libdistributed_camera_data_process.so`  
**位置**: `services/data_process/BUILD.gn`

**Sources**:
```gn
sources = [
    "${services_path}/cameraservice/base/src/dcamera_sink_frame_info.cpp",
    "src/pipeline/abstract_data_process.cpp",
    "src/pipeline/dcamera_pipeline_sink.cpp",
    "src/pipeline/dcamera_pipeline_source.cpp",
    "src/pipeline_node/fpscontroller/fps_controller_process.cpp",
    "src/pipeline_node/multimedia_codec/decoder/*.cpp",
    "src/pipeline_node/multimedia_codec/encoder/*.cpp",
    "src/utils/image_common_type.cpp",
    "src/utils/property_carrier.cpp",
]
```

**条件编译**:
```gn
if (!distributed_camera_common) {
    sources += [
        "src/pipeline_node/multimedia_codec/decoder/decode_data_process.cpp",
        "src/pipeline_node/scale_conversion/scale_convert_process.cpp",
    ]
} else {
    sources += [
        "src/pipeline_node/multimedia_codec/decoder/decode_data_process_common.cpp",
        "src/pipeline_node/scale_conversion/scale_convert_process_common.cpp",
    ]
}
if (distributed_camera_common) {
    cflags += [ "-DDCAMERA_SUPPORT_FFMPEG" ]
} else {
    cflags += [ "-DDCAMERA_MMAP_RESERVE" ]
}
```

---

## 条件编译开关

### GNI 参数 (distributedcamera.gni)

| 参数 | 默认值 | 用途 |
|------|--------|------|
| `distributed_camera_common` | true | 控制公共代码路径 |
| `device_security_level_camera` | true | 设备安全级别检查 |
| `distributed_camera_filter_front` | false | 前置摄像头过滤 |
| `distributed_camera_wakeup_enabled` | false | 唤醒功能 |
| `distributed_camera_open_stabile` | false | 相机打开稳定性优化 |
| `os_account_camera` | 自动检测 | OS 账户支持 |

### 宏定义

| 宏 | 触发条件 | 用途 |
|----|----------|------|
_DCAMERA_FILE| `DUMP` | `build_variant == "root"` | 调试文件转储 |
| `OS_ACCOUNT_ENABLE` | `os_account_camera` | OS 账户功能 |
| `SECURITY_LEVEL_CHECK_ENABLE` | `!distributed_camera_common` | 安全级别检查 |
| `DEVICE_SECURITY_LEVEL_ENABLE` | `device_security_level_camera` | 设备安全级别 |
| `DCAMERA_SUPPORT_FFMPEG` | `distributed_camera_common` | FFmpeg 支持 |
| `DCAMERA_WAKEUP` | `distributed_camera_wakeup_enabled` | 唤醒功能 |
| `DCAMERA_OPEN_STABILE` | `distributed_camera_open_stabile` | 打开稳定性 |
| `DCAMERA_MMAP_RESERVE` | `!distributed_camera_common` | MMAP 保留 |
| `DCAMERA_FRONT` | `distributed_camera_filter_front` | 前置过滤 |

---

## 安全编译选项

所有 `ohos_shared_library` target 均启用：

```gn
sanitize = {
  cfi = true              # 控制流完整性
  cfi_cross_dso = true    # 跨 DSO CFI
  boundary_sanitize = true
  integer_overflow = true
  ubsan = true            # 未定义行为检测
}
stack_protector_ret = true
branch_protector_ret = "pac_ret"  # ARM PAC 返回地址保护

ldflags = [
  "-fpie",               # 位置无关可执行文件
  "-Wl,-z,relro",       # 只读重定位
  "-Wl,-z,now",         # 立即绑定
]
```

---

## 产物安装路径

| 产物 | 目标路径 |
|------|----------|
| `libdistributed_camera_*.so` | `/system/lib/` |
| `dcamera_sa_profile` | `/system/profile/` |
| `dcamera.cfg` | `/init/` |

---

## 依赖关系图

```
                    ┌────────────────────────┐
                    │  distributed_camera_utils  │
                    └───────────┬────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│distributed_camera│   │distributed_camera│   │distributed_camera│
│   _channel       │   │_data_process    │   │_cameraoperator/*│
└────────┬────────┘   └────────┬────────┘   └────────┬────────┘
         │                      │                      │
         └──────────────────────┼──────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │ distributed_camera    │
                    │    sourceservice      │
                    └───────────────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │ distributed_camera    │
                    │     sinkservice       │
                    └───────────────────────┘
```
