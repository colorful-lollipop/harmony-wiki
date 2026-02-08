# GN 构建产物

## 构建配置

### 构建参数文件

**文件**：`mechbody.gni`

```gn
declare_args() {
  mechbody_controller_feature_L1 = false      # L1 特性开关
  mechbody_controller_feature_product = true # 产品特性开关
  mechbody_controller_extended = false       # 扩展特性开关
}
```

### 条件编译

| Define | 条件 | 用途 |
|--------|------|------|
| `MECHBODY_CONTROLLER_EXTENDED` | `mechbody_controller_extended == true` | 扩展功能 |
| `MECHBODY_CONTROLLER_L1` | `mechbody_controller_feature_L1 == true` | L1 变体 |

## Targets 清单

### 核心库 Targets

| Target | 类型 | 输出文件 | 路径 |
|--------|------|---------|------|
| **mechbody_service** | ohos_shared_library | libmechbody_service.z.so | services/BUILD.gn:17 |
| **mechanicmanager_napi** | ohos_shared_library | libmechanicmanager_napi.so | interface/napi/mech_manager/BUILD.gn:17 |
| **mechanic_manager_ani** | taihe_shared_library | taihe 模块 | interface/ets/mech_manager/BUILD.gn:63 |

### 配置 Targets

| Target | 类型 | 输出文件 | 路径 |
|--------|------|---------|------|
| **mechbody_etc** | ohos_preInstalledConfig | mechbody.cfg | etc/init/BUILD.gn:18 |
| **mechbody_sa_profile** | ohos_sa_profile | 8550.json | sa_profile/BUILD.gn:17 |

## mechbody_service 详解

### Target 定义

**文件**：`services/BUILD.gn`

```gn
ohos_shared_library("mechbody_service") {
  name = "mechbody_service"

  sources = [
    # Controller
    "src/controller/mc_controller_manager.cpp",
    "src/controller/mc_camera_tracking_controller.cpp",
    "src/controller/mc_controller_ipc_death_listener.cpp",
    # Connect
    "src/connect/mc_connect_manager.cpp",
    "src/connect/bluetooth_state_adapter.cpp",
    "src/connect/bluetooth_state_listener.cpp",
    # Motion
    "src/motion/mc_motion_manager.cpp",
    # Transport
    "src/transport/mc_send_adapter.cpp",
    "src/transport/mc_data_buffer.cpp",
    "src/transport/mc_protocol_convertor.cpp",
    "src/transport/mc_subscription_center.cpp",
    "src/transport/mc_event_listener.cpp",
    # Command 0x01
    "src/transport/command/0x01/mc_command_base_v1.cpp",
    "src/transport/command/0x01/mc_camera_tracking_command.cpp",
    # ... 21 files
    # Command 0x02
    "src/transport/command/0x02/mc_command_base_v2.cpp",
    # ... 21 files
    # BLE
    "src/ble_send_manager.cpp",
    # Core
    "src/mechbody_controller_service.cpp",
    "src/mechbody_controller_stub.cpp",
    "src/mechbody_controller_utils.cpp",
    # Utils
    "src/utils/load_mechbody_adapter.cpp",
    # DotReport
    "src/dotReport/hisysevent_utils.cpp",
  ]

  cflags = [
    "-frtti",
    "-fpie",
  ]
  cflags_cc = [ "-fstack-protector-strong" ]
  ldflags = [
    "-Wl,-z,relro",
    "-Wl,-z,now",
  ]

  sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    integer_overflow = true
   ubsan = true
  }

  external_deps = [
    "ability_base:base",
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "access_token:libtokensetproc_shared",
    "bluetooth:btframework",
    "bluetooth:btcommon",
    "camera_framework:camera_framework",
    "cJSON:cjson",
    "c_utils:utils",
    "eventhandler:libeventhandler",
    "graphic_surface:surface",
    "hilog:libhilog",
    "init:libbegetutil",
    "input:libmmi-client",
    "input:oh_input_manager",
    "ipc:ipc_core",
    "os_account:libaccountkits",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "sensor:sensor_interface_native",
    "window_manager:libdm_lite",
    "hisysevent:libhisysevent",
    "drivers_interface_camera:metadata",
  ]

  include_dirs = [
    "include",
    "include/dotReport",
    "include/connect",
    "include/controller",
    "include/motion",
    "include/transport",
    "include/transport/command",
    "include/transport/command/0x01",
    "include/transport/command/0x02",
    "include/utils",
    "//foundation/distributedhardware/mechbody_controller/interface/inner_api",
  ]

  install_enable = true
  install_images = [ system_base_dir ]
}
```

### Sources 文件统计

| 类别 | 文件数 |
|------|--------|
| Controller | 3 |
| Connect | 3 |
| Motion | 1 |
| Transport (Base) | 5 |
| Command 0x01 | 21 |
| Command 0x02 | 21 |
| BLE | 1 |
| Core | 3 |
| Utils | 1 |
| DotReport | 1 |
| **总计** | **60+** |

## mechanicmanager_napi 详解

### Target 定义

**文件**：`interface/napi/mech_manager/BUILD.gn`

```gn
ohos_shared_library("mechanicmanager_napi") {
  name = "mechanicmanager_napi"

  sources = [
    "js_mech_manager.cpp",
    "js_mech_manager_client.cpp",
    "js_mech_manager_service.cpp",
    "js_mech_manager_stub.cpp",
  ]

  external_deps = [
    "ability_base:base",
    "access_token:libaccesstoken_sdk",
    "bluetooth:btframework",
    "bluetooth:btcommon",
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
  ]

  defines = []
  if (mechbody_controller_feature_L1) {
    defines += [ "MECHBODY_CONTROLLER_L1" ]
  }
  if (target_cpu == "arm") {
    defines += [ "BINDER_IPC_32BIT" ]
  }

  relative_install_dir = "module/distributedhardware"
}
```

## 产物清单

### 编译产物

| 产物 | 类型 | 描述 |
|------|------|------|
| `libmechbody_service.z.so` | 共享库 | 核心服务 |
| `libmechanicmanager_napi.so` | 共享库 | N-API 接口 |
| `mechbody.cfg` | 配置文件 | 服务初始化配置 |
| `8550.json` | SA 配置 | SystemAbility 配置 |

### 产物大小

| 产物 | ROM | RAM (估计) |
|------|-----|-------------|
| mechbody_service | ~900KB | ~10MB |
| mechanicmanager_napi | ~50KB | ~1MB |
| **总计** | **~950KB** | **~11MB** |

## 安装路径

| 产物 | 安装路径 |
|------|----------|
| `libmechbody_service.z.so` | `/system/lib64/` 或 `/system/lib/` |
| `libmechanicmanager_napi.so` | `/system/app/Nxxxxx/distributedhardware/` |
| `mechbody.cfg` | `/system/etc/init/` |
| `8550.json` | `/system/sa_profile/` |

### 产物 → Target 映射

```
libmechbody_service.z.so  →  services:mechbody_service
libmechanicmanager_napi.so → interface/napi/mech_manager:mechanicmanager_napi
mechbody.cfg               → etc/init:mechbody_etc
8550.json                  → sa_profile:mechbody_sa_profile
```

## 运行时加载关系

```
应用进程
    │
    ▼ dlopen
libmechanicmanager_napi.so  (N-API)
    │
    │ IPC 调用
    ▼
system/ sa/ mechbody (进程)
    │
    ▼ dlopen
libmechbody_service.z.so
    │
    │ dlopen (动态)
    ▼
libmech_adapter.z.so  (vendor 南向适配)
```

### 动态加载

**文件**：`services/src/utils/load_mechbody_adapter.cpp`

```cpp
void* handle = dlopen("/system/lib64/libmech_adapter.z.so", RTLD_LAZY);
void* symbol = dlsym(handle, "MechAdapterCreate");
```

## 依赖关系

### 外部依赖

| 依赖 | 用途 |
|------|------|
| **ability_base:base** | 能力框架基础 |
| **access_token** | 权限管理 |
| **bluetooth** | 蓝牙通信 |
| **camera_framework** | 相机交互 |
| **ipc:ipc_core** | IPC 框架 |
| **safwk** | SystemAbility 框架 |
| **samgr** | 服务管理 |
| **hilog** | 日志 |
| **cJSON** | JSON 解析 |
| **sensor** | 传感器接口 |

### 内部依赖

| Consumer | Target | 说明 |
|----------|--------|------|
| test/fuzztest/* | services:mechbody_service | 模糊测试 |
| interface/ets/* | (IDL 生成) | ANI 接口 |

## 安全加固

### 编译时加固

| 选项 | 值 | 说明 |
|------|-----|------|
| `-fstack-protector-strong` | cflags_cc | 栈保护 |
| `-Wl,-z,relro,-z,now` | ldflags | RELRO 加固 |
| **Sanitizers** | | |
| boundary_sanitize | true | 边界检查 |
| cfi | true | 控制流完整性 |
| cfi_cross_dso | true | 跨 DSO CFI |
| integer_overflow | true | 整数溢出检查 |
| ubsan | true | 未定义行为检查 |

### SELinux

```
Context: u:r:mechbody:s0
Process: mechbody (uid)
```

## 构建命令

### 全量构建

```bash
hb build mechbody_controller
```

### 单个 Target

```bash
hb build //foundation/distributedhardware/mechbody_controller/services:mechbody_service
hb build //foundation/distributedhardware/mechbody_controller/interface/napi/mech_manager:mechanicmanager_napi
```

### 带配置构建

```bash
hb build mechbody_controller --gn-args "mechbody_controller_feature_product=true"
```

## 相关文档

- [架构说明](02_Architecture.md) → 构建在架构中的位置
- [N-API 参考](03_NAPI_Reference.md) → N-API 模块
- [安全评审](06_Security_Review.md) → 构建安全
