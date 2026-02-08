# GN 目标梳理

> **目的**: 梳理 Sensor 子系统的 GN 构建目标、类型、依赖、产物和配置开关
> **适用范围**: /base/sensors/sensor（排除 test/ 目录）
> **关键结论**: Sensor 子系统包含 9 个主要 BUILD.gn，生成 11 个主要产物，支持多种 feature flags
> **相关跳转**: [目录结构](01_Directory_Structure.md) | [编译产物](06_Build_Artifacts.md)

---

## 关键 BUILD.gn 文件

| BUILD.gn 文件 | 主要 Target | 说明 |
|--------------|------------|------|
| `frameworks/native/BUILD.gn` | `sensor_target`, `ohsensor` | Native 客户端库 |
| `frameworks/js/napi/BUILD.gn` | `sensor_js_target` | JS N-API 绑定库 |
| `frameworks/cj/BUILD.gn` | `cj_sensor_ffi` | Cangjie FFI 库 |
| `frameworks/ets/taihe/BUILD.gn` | `sensor_taihe` | ArkTS/ETS 绑定库 |
| `services/BUILD.gn` | `sensor_service_target` | Sensor 服务 |
| `utils/common/BUILD.gn` | `libsensor_utils` | 公共工具库 |
| `utils/ipc/BUILD.gn` | `libsensor_ipc` | IPC 工具库 |
| `utils/BUILD.gn` | `sensor_utils_target` | Utils 组目标 |
| `sa_profile/BUILD.gn` | `sensors_sa_profiles` | SA 配置文件 |

---

## 全局配置 (sensor.gni)

**文件**: `/base/sensors/sensor/sensor.gni`

### 全局变量

```gn
SUBSYSTEM_DIR = "//base/sensors/sensor"
FUZZ_MODULE_OUT_PATH = "sensor/sensor"
```

### Feature Flags

| Flag | 默认值 | 说明 | 影响范围 |
|-------|---------|------|---------|
| `hiviewdfx_hisysevent_enable` | false | HiSysEvent 日志 | 全局 |
| `hiviewdfx_hitrace_enable` | false | HiTrace 追踪 | 全局 |
| `hdf_drivers_interface_sensor` | auto | HDF 传感器驱动接口 | 服务层 |
| `sensor_memmgr_enable` | auto | 内存管理器 | 服务层 |
| `sensor_access_token_enable` | auto | Access Token 权限 | 全局 |
| `sensor_msdp_motion_enable` | auto | MSDP 运动能力 | 服务层 |
| `sensor_build_eng` | variant | 工程构建模式 | 全局 |

> **证据**: `sensor.gni:16-76`

---

## Targets 详细说明

### 1. Sensor Service (`services/BUILD.gn`)

#### Target: `libsensor_service`

**类型**: `ohos_shared_library`
**输出**: `libsensor_service.z.so`
**SA 类型**: `shlib_type = "sa"`

**Sources**:
```
src/client_info.cpp
src/fifo_cache_data.cpp
src/flush_info_record.cpp
src/sensor_common_event_subscriber.cpp
src/sensor_data_manager.cpp
src/sensor_dump.cpp
src/sensor_manager.cpp
src/sensor_observer.cpp
src/sensor_power_policy.cpp
src/sensor_service.cpp
src/sensor_shake_control_manager.cpp
src/stream_server.cpp
```

**Conditional Sources** (当 `hdf_drivers_interface_sensor`):
```
hdi_connection/adapter/src/hdi_connection.cpp
hdi_connection/adapter/src/sensor_event_callback.cpp
hdi_connection/adapter/src/sensor_plug_callback.cpp
hdi_connection/interface/src/sensor_hdi_connection.cpp
src/sensor_data_processer.cpp
```

**Engineering Builds**:
```
hdi_connection/adapter/src/compatible_connection.cpp
hdi_connection/hardware/src/hdi_service_impl.cpp
```

**Include Directories**:
```
$SUBSYSTEM_DIR/frameworks/native/include
$SUBSYSTEM_DIR/interfaces/inner_api
$SUBSYSTEM_DIR/services/include
$SUBSYSTEM_DIR/utils/common/include
$SUBSYSTEM_DIR/utils/ipc/include
$SUBSYSTEM_DIR/services/hdi_connection/interface/include (conditional)
$SUBSYSTEM_DIR/services/hdi_connection/adapter/include (conditional)
$SUBSYSTEM_DIR/services/hdi_connection/hardware/include (eng builds)
```

**Internal Dependencies**:
- `//base/sensors/sensor/frameworks/native:sensor_service_stub`
- `//base/sensors/sensor/utils/common:libsensor_utils`
- `//base/sensors/sensor/utils/ipc:libsensor_ipc`

**External Dependencies**:
```
access_token:libaccesstoken_sdk
access_token:libtokenid_sdk
c_utils:utils
common_event_service:cesfwk_innerkits
data_share:datashare_consumer
hicollie:libhicollie
hilog:libhilog
init:libbegetutil
init:libbeget_proxy
ipc:ipc_single
safwk:system_ability_fwk
samgr:samgr_proxy
os_account:os_account_innerkits
memmgr:memmgrclient (conditional: sensor_memmgr_enable)
hisysevent:libhisysevent (conditional: hiviewdfx_hisysevent_enable)
hitrace:hitrace_meter (conditional: hiviewdfx_hitrace_enable)
drivers_interface_sensor:libsensor_proxy_3.0 (conditional: hdf_drivers_interface_sensor)
drivers_interface_sensor:libsensor_convert_proxy_1.0 (conditional: hdf_drivers_interface_sensor)
```

**Security**: CFI enabled, PAC-RET branch protection, visibility hidden

> **证据**: `services/BUILD.gn:17-140`

---

#### Target: `libsensor_service_static`

**类型**: `ohos_static_library`
**说明**: 静态链接版本，用于特定场景

**Sources 和 Dependencies**: 与 `libsensor_service` 相同

> **证据**: `services/BUILD.gn:143-265`

---

### 2. Framework Native (`frameworks/native/BUILD.gn`)

#### Target: `sensor_service_interface`

**类型**: `idl_gen_interface`
**源**: `ISensorService.idl`
**日志**: LOG_DOMAIN=0xD002700, LOG_TAG=ISensorServiceIdl

> **证据**: `frameworks/native/BUILD.gn:18-24`

---

#### Target: `sensor_service_stub`

**类型**: `ohos_source_set`
**说明**: 从 IDL 生成的 Stub 代码

**Dependencies**:
- `:sensor_service_interface`
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_single`
- `samgr:samgr_proxy`

> **证据**: `frameworks/native/BUILD.gn:26-45`

---

#### Target: `libsensor_client`

**类型**: `ohos_shared_library`
**输出**: `libsensor_client.z.so`

**Sources**:
```
src/fd_listener.cpp
src/sensor_agent_proxy.cpp
src/sensor_client_stub.cpp
src/sensor_data_channel.cpp
src/sensor_event_handler.cpp
src/sensor_file_descriptor_listener.cpp
src/sensor_service_client.cpp
```

**Include Directories**:
```
${target_gen_dir} (IDL generated)
$SUBSYSTEM_DIR/frameworks/native/include
$SUBSYSTEM_DIR/interfaces/inner_api
$SUBSYSTEM_DIR/utils/common/include
$SUBSYSTEM_DIR/utils/ipc/include
```

**Dependencies**:
- Internal: `sensor_service_interface`, `libsensor_utils`, `libsensor_ipc`
- External: `c_utils:utils`, `eventhandler:libeventhandler`, `hicollie:libhicollie`, `hilog:libhilog`, `ipc:ipc_single`, `samgr:samgr_proxy`

> **证据**: `frameworks/native/BUILD.gn:47-103`

---

#### Target: `sensor_interface_native`

**类型**: `ohos_shared_library`
**输出**: `sensor_agent.z.so`
**InnerAPI Tag**: `platformsdk`

**Sources**:
```
src/geomagnetic_field.cpp
src/sensor_agent.cpp
src/sensor_algorithm.cpp
```

**Dependencies**:
- `:libsensor_client`
- External: `c_utils:utils`, `eventhandler:libeventhandler`, `hilog:libhilog`, `ipc:ipc_single`

**Security**: boundary_sanitize, integer_overflow, ubsan (增强)

> **证据**: `frameworks/native/BUILD.gn:122-155`

---

#### Target: `libsensor_ndk`

**类型**: `ohos_ndk_library`
**输出**: `sensor.so`
**NDK 描述**: `libsensor.json`
**最小版本**: "6"

> **证据**: `frameworks/native/BUILD.gn:104-108`

---

#### Target: `ohsensor`

**类型**: `ohos_shared_library`
**输出**: `ohsensor.so`
**安装路径**: `ndk/` subdirectory
**定义**: `API_EXPORT=__attribute__((visibility ("default")))`

**Dependencies**:
- `:libsensor_client`
- `:sensor_interface_native`
- `:libsensor_ipc`
- External: `c_utils:utils`, `eventhandler:libeventhandler`, `hilog:libhilog`, `ipc:ipc_single`

> **证据**: `frameworks/native/BUILD.gn:161-192`

---

### 3. Framework JS/NAPI (`frameworks/js/napi/BUILD.gn`)

#### Target: `libsensor`

**类型**: `ohos_shared_library`
**输出**: `libsensor.z.so`
**安装路径**: `module/` subdirectory
**Part**: sensor

**Sources**:
```
src/sensor_js.cpp
src/sensor_napi_error.cpp
src/sensor_napi_utils.cpp
src/sensor_system_js.cpp
```

**Defines**:
```
APP_LOG_TAG = "sensorJs"
LOG_DOMAIN = 0xD002700
```

**Dependencies**:
- Internal: `:sensor_interface_native`
- External: `bundle_framework:appexecfwk_base`, `bundle_framework:appexecfwk_core`, `c_utils:utils`, `hilog:libhilog`, `ipc:ipc_single`, `napi:ace_napi`

**Security**: CFI, PAC-RET, visibility hidden

> **证据**: `frameworks/js/napi/BUILD.gn:17-56`

---

### 4. Framework ETS/Taihe (`frameworks/ets/taihe/BUILD.gn`)

#### Target: `copy_taihe`

**类型**: `copy_taihe_idl`
**源**: `idl/ohos.sensor.taihe`

> **证据**: `frameworks/ets/taihe/BUILD.gn:28-34`

---

#### Target: `sensor_taihe_native`

**类型**: `taihe_shared_library`

**Sources**:
```
author/src/ani_constructor.cpp
author/src/ohos.sensor.impl.cpp
```

> **证据**: `frameworks/ets/taihe/BUILD.gn:51-62`

---

### 5. Framework CJ (`frameworks/cj/BUILD.gn`)

#### Target: `cj_sensor_ffi`

**类型**: `ohos_shared_library`
**InnerAPI Tag**: `platformsdk`

**Sources**:
```
src/cj_sensor_ffi.cpp
src/cj_sensor_impl.cpp
```

**Include Directories**:
```
$SUBSYSTEM_DIR/frameworks/native/include
$SUBSYSTEM_DIR/interfaces/inner_api
$SUBSYSTEM_DIR/utils/common/include
include
```

> **证据**: `frameworks/cj/BUILD.gn` (TODO: 完整读取)

---

### 6. Utils Common (`utils/common/BUILD.gn`)

#### Target: `libsensor_utils`

**类型**: `ohos_shared_library`
**InnerAPI Tag**: `platformsdk_indirect`

**Sources**:
```
src/active_info.cpp
src/motion_plugin.cpp
src/permission_util.cpp
src/print_sensor_data.cpp
src/report_data_callback.cpp
src/security_privacy_manager_plugin.cpp
src/sensor.cpp
src/sensor_basic_data_channel.cpp
src/sensor_basic_info.cpp
src/sensor_channel_info.cpp
src/sensor_xcollie.cpp
```

> **证据**: `utils/common/BUILD.gn` (TODO: 完整读取)

---

### 7. Utils IPC (`utils/ipc/BUILD.gn`)

#### Target: `libsensor_ipc`

**类型**: `ohos_shared_library`
**InnerAPI Tag**: `platformsdk_indirect`

**Sources**:
```
src/circle_stream_buffer.cpp
src/net_packet.cpp
src/stream_buffer.cpp
src/stream_session.cpp
src/stream_socket.cpp
```

> **证据**: `utils/ipc/BUILD.gn` (TODO: 完整读取)

---

### 8. SA Profile (`sa_profile/BUILD.gn`)

#### Target: `sensors_sa_profiles`

**类型**: `ohos_sa_profile`
**源**: `3601.json`

**SA 配置**:
- SA ID: 3601
- Process: "sensors"
- Library: `libsensor_service.z.so`
- Run-on-create: true
- Distributed: false
- Dump level: 1
- Min HDI proxy version: `libsensor_proxy_3.0.z.so`

> **证据**: `sa_profile/3601.json`, `sa_profile/BUILD.gn`

---

## Target 依赖图

```
最终产物依赖关系:

应用层 (JS/TS/CJ 应用)
    ↓
libsensor.z.so (NAPI)
    ├─→ sensor_agent.z.so (sensor_interface_native)
    │     ├─→ libsensor_client.z.so
    │     │     └─→ libsensor_ipc.z.so
    │     └─→ libsensor_utils.z.so
    └─→ libsensor_ipc.z.so
          └─→ libsensor_utils.z.so

服务端 (libsensor_service.z.so - SA 3601)
    ├─→ libsensor_utils.z.so
    ├─→ libsensor_ipc.z.so
    └─→ drivers_interface_sensor (HDF Proxy)

其他产物:
- ohsensor.so (NDK)
- sensor_abc.abc (ArkTS Bytecode)
- cj_sensor_ffi.z.so (Cangjie FFI)
- sensor_taihe_native.z.so (ArkTS Native)
```

---

## Bundle 配置目标

从 `bundle.json` 中提取的构建组：

### fwk_group

```
//base/sensors/sensor/frameworks/js/napi:sensor_js_target
//base/sensors/sensor/frameworks/cj:cj_sensor_ffi
//base/sensors/sensor/frameworks/native:sensor_target
//base/sensors/sensor/frameworks/native:ohsensor
//base/sensors/sensor/frameworks/ets/taihe:sensor_taihe
//base/sensors/sensor/utils:sensor_utils_target
```

> **证据**: `bundle.json:50-56`

### service_group

```
//base/sensors/sensor/services:sensor_service_target
//base/sensors/sensor/sa_profile:sensors_sa_profiles
```

> **证据**: `bundle.json:58-61`

### Inner Kits

```
//base/sensors/sensor/frameworks/ets/taihe:copy_taihe
//base/sensors/sensor/frameworks/native:sensor_interface_native (with headers)
//base/sensors/sensor/frameworks/cj:cj_sensor_ffi
```

> **证据**: `bundle.json:63-84`

---

## 编译选项

### 全局选项

| 选项 | 默认值 | 说明 |
|------|---------|------|
| CFI | true | 控制流完整性保护 |
| CFI Cross DSO | true | 跨 DSO CFI 保护 |
| Visibility | hidden | 符号隐藏 |
| Function Sections | -ffunction-sections | 函数分段 |
| Data Sections | -fdata-sections | 数据分段 |
| Optimization | -Oz | 大小优化 |

> **证据**: 各 BUILD.gn 文件中的 `cflags`, `sanitize`, `branch_protector_ret`

### 条件编译宏

| 宏 | 触发条件 | 影响范围 |
|----|----------|---------|
| HDF_DRIVERS_INTERFACE_SENSOR | hdf_drivers_interface_sensor | HDI 连接代码 |
| MEMMGR_ENABLE | sensor_memmgr_enable | 内存管理器集成 |
| ACCESS_TOKEN_ENABLE | sensor_access_token_enable | Access Token 权限支持 |
| MSDP_MOTION_ENABLE | sensor_msdp_motion_enable | MSDP 运动能力 |
| HIVIEWDFX_HISYSEVENT_ENABLE | hiviewdfx_hisysevent_enable | HiSysEvent 日志 |
| HIVIEWDFX_HITRACE_ENABLE | hiviewdfx_hitrace_enable | HiTrace 追踪 |
| BUILD_VARIANT_ENG | build_variant == "root" | 工程构建模式 |

> **证据**: `services/BUILD.gn:62,92,95,96,99-105`

---

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 查看目录组织
- [编译产物](06_Build_Artifacts.md) - 查看产物详细列表和加载关系
