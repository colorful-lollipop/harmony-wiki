# 构建配置与编译产物

## 构建系统

sensor_lite 使用 **GN (Generate Ninja)** 构建系统，配合 OpenHarmony 的 **HPM (HarmonyOS Package Manager)**。

## 组件配置

### bundle.json

**文件**: `bundle.json`

```json
{
  "name": "@ohos/sensor_lite",
  "version": "3.1",
  "component": {
    "name": "sensor_lite",
    "subsystem": "sensors",
    "adapted_system_type": ["mini"],
    "rom": "92KB",
    "ram": "~200KB",
    "deps": {
      "components": [
        "drivers_peripheral_sensor",
        "hilog_lite",
        "ipc",
        "samgr_lite",
        "utils_lite"
      ],
      "third_party": ["bounds_checking_function"]
    },
    "build": {
      "sub_component": [
        "//base/sensors/sensor_lite/services:sensor_service",
        "//base/sensors/sensor_lite/frameworks:sensor_lite"
      ]
    }
  }
}
```

**证据**: `bundle.json:1-35`

## GN Targets

### 1. sensor_service (可执行文件)

**位置**: `services/BUILD.gn`

```gn
executable("sensor_service") {
  sources = [
    "./src/proc.c",
    "./src/sensor_service.c",
    "./src/sensor_service_impl.c",
  ]
  
  include_dirs = [
    "./include",
    "//third_party/bounds_checking_function/include",
    "//commonlibrary/utils_lite/include",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast",
    "//foundation/systemabilitymgr/samgr_lite/samgr/source",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc/include",
    "//foundation/communication/ipc/ipc/native/c/manager/include",
    "//foundation/communication/ipc/services/dbinder/c/include",
    "//base/sensors/sensor_lite/frameworks/include",
    "//base/sensors/sensor_lite/interfaces/kits/native/include",
  ]
  
  defines = sensor_default_defines
  
  deps = [
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]
  
  if (has_drivers_peripheral_sensor_part) {
    deps += ["//drivers/peripheral/sensor/hal:hdi_sensor"]
    include_dirs += ["//drivers/peripheral/sensor/interfaces/include"]
  }
}
```

**证据**: `services/BUILD.gn:17-51`

| 配置项 | 值 |
|-------|-----|
| 产物类型 | 可执行文件 |
| 源码文件 | proc.c, sensor_service.c, sensor_service_impl.c |
| 依赖 | ipc_single, samgr, hdi_sensor (条件) |

### 2. sensor_lite (Lite 组件)

**位置**: `frameworks/BUILD.gn`

```gn
lite_component("sensor_lite") {
  features = ["src:sensor_client"]
}
```

**证据**: `frameworks/BUILD.gn:16-18`

### 3. sensor_client (共享库)

**位置**: `frameworks/src/BUILD.gn`

```gn
shared_library("sensor_client") {
  if (ohos_kernel_type == "liteos_riscv") {
    sources = [
      "sensor_agent.c",
      "sensor_agent_client.c",
    ]
  }
  if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
    sources = [
      "sensor_agent.c",
      "sensor_agent_proxy.c",
    ]
  }
  
  include_dirs = [
    "../include",
    "//third_party/bounds_checking_function/include",
    "//commonlibrary/utils_lite/include",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast",
    "//foundation/systemabilitymgr/samgr_lite/samgr/source",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc/include",
    "//base/sensors/sensor_lite/interfaces/kits/native/include",
    "//base/sensors/sensor_lite/services/include",
  ]
  
  deps = ["//foundation/systemabilitymgr/samgr_lite/samgr:samgr"]
}
```

**证据**: `frameworks/src/BUILD.gn:16-46`

**条件编译**:

| 条件 | 源文件 |
|-----|-------|
| `liteos_riscv` | sensor_agent.c + sensor_agent_client.c |
| `liteos_a` / `linux` | sensor_agent.c + sensor_agent_proxy.c |

### 4. sensor_lite.gni (配置)

**位置**: `sensor_lite.gni`

```gni
sensor_default_defines = []

if (!defined(global_parts_info) ||
    defined(global_parts_info.hdf_drivers_peripheral_sensor)) {
  has_drivers_peripheral_sensor_part = true
  sensor_default_defines += ["HAS_HDI_SENSOR_LITE_PRAT"]
} else {
  has_drivers_peripheral_sensor_part = false
}
```

**证据**: `sensor_lite.gni:16-24`

## 编译产物

### 产物清单

| 产物名称 | 类型 | 路径模式 | 说明 |
|---------|------|---------|------|
| `sensor_service` | 可执行文件 | `out/[product]/sensor_lite/services/` | Sensor 服务进程 |
| `libsensor_client.so` | 共享库 | `out/[product]/sensor_lite/frameworks/src/` | 客户端库 |

### 安装路径

| 产物 | 典型安装路径 |
|-----|-------------|
| sensor_service | `/system/bin/sensor_service` |
| libsensor_client.so | `/system/lib/libsensor_client.z.so` |

### 运行时加载关系

```
┌─────────────────────────────────────────────────────────────────┐
│                    Application                                   │
│   链接: libsensor_client.so                                     │
│   dlopen: 自动加载 (由系统加载器处理)                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│               libsensor_client.so                                │
│   依赖: libsamgr_lite.so, libipc.so                             │
│   职责: 封装 IPC 调用、管理回调                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│               sensor_service (独立进程)                           │
│   启动: 系统 init 进程                                            │
│   依赖: libsamgr_lite.so, libipc.so, libhdi_sensor.so           │
└─────────────────────────────────────────────────────────────────┘
```

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           sensor_lite 依赖图                                  │
└─────────────────────────────────────────────────────────────────────────────┘

//base/sensors/sensor_lite:sensor_service
│
├── //foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single
│   └── libipc.so
├── //foundation/systemabilitymgr/samgr_lite/samgr:samgr
│   └── libsamgr_lite.so
└── //drivers/peripheral/sensor/hal:hdi_sensor (条件)
    └── libhdi_sensor.so

//base/sensors/sensor_lite/frameworks:sensor_lite (lite_component)
│
└── //base/sensors/sensor_lite/frameworks/src:sensor_client
    ├── //foundation/systemabilitymgr/samgr_lite/samgr:samgr
    └── 条件编译:
        ├── liteos_riscv: sensor_agent_client.c
        └── liteos_a/linux: sensor_agent_proxy.c
```

## 构建命令

### 全量构建

```bash
# 使用 hb (HarmonyOS Build) 工具
hb set -p [product]
hb build -f

# 或使用 GN + Ninja
gn gen out/[product]
ninja -C out/[product] sensor_lite
```

### 增量构建

```bash
ninja -C out/[product] //base/sensors/sensor_lite/services:sensor_service
ninja -C out/[product] //base/sensors/sensor_lite/frameworks:sensor_lite
```

### 子模块构建

```bash
# 构建客户端库
ninja -C out/[product] //base/sensors/sensor_lite/frameworks/src:sensor_client

# 构建服务
ninja -C out/[product] //base/sensors/sensor_lite/services:sensor_service
```

## 条件编译说明

### HAS_HDI_SENSOR_LITE_PRAT

当定义了 `HAS_HDI_SENSOR_LITE_PRAT` 宏时：

- `sensor_service_impl.c` 会调用 `g_sensorDevice->GetAllSensors()`
- `sensor_service_impl.c` 会调用 `g_sensorDevice->Enable()` / `Disable()`
- `sensor_service_impl.c` 会调用 `g_sensorDevice->Register()` / `Unregister()`

**证据**: `services/src/sensor_service_impl.c:36-48`

### has_drivers_peripheral_sensor_part

当 `has_drivers_peripheral_sensor_part = true` 时：

- `services/BUILD.gn` 依赖 `//drivers/peripheral/sensor/hal:hdi_sensor`
- `sensor_lite.gni` 定义 `HAS_HDI_SENSOR_LITE_PRAT`

**证据**: `sensor_lite.gni:18-21`

---

## 相关文档

- [项目概览](01_Overview.md)
- [API 参考](02_API_Reference.md)
- [架构设计](03_Architecture.md)
- [安全评审](05_Security.md)
