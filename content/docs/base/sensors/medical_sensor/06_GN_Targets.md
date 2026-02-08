# GN Targets 文档

## 目的

本文档系统说明 Medical_Sensor 的 GN 构建目标、依赖关系和编译产物。

---

## 适用范围

本文档适用于需要：
- 理解构建系统结构
- 分析模块依赖关系
- 定位编译问题和产物

---

## 所有 BUILD.gn 文件

### 非 test 目录的 BUILD.gn 文件（6 个）

| 路径 | 说明 |
|------|------|
| `sa_profile/BUILD.gn` | SA 配置文件目标 |
| `utils/BUILD.gn` | 工具库目标 |
| `frameworks/native/medical_sensor/BUILD.gn` | 客户端框架目标 |
| `services/medical_sensor/BUILD.gn` | 服务端目标 |
| `interfaces/native/BUILD.gn` | Native 接口目标 |
| `interfaces/plugin/BUILD.gn` | JS/NAPI 插件目标 |

---

## 目标定义详解

### 1. SA 配置目标（sa_profile/BUILD.gn）

**目标类型**：`ohos_sa_profile`

**输出**：`3605.xml`

**GN 配置**：

```gn
ohos_sa_profile("medical_sa_profiles") {
  sources = [ "3605.xml" ]
  part_name = "medical_sensor"
}
```

**证据**：
- 配置文件：`sa_profile/BUILD.gn:16-19`

---

### 2. 工具库目标（utils/BUILD.gn）

**目标类型**：`ohos_shared_library`

**输出**：`libmedical_utils.so`

**关键配置**：

```gn
ohos_shared_library("libmedical_utils") {
  sources = [
    "src/dmd_report.cpp",
    "src/report_data_cache.cpp",
    "src/medical_basic_info.cpp",
    "src/permission_util.cpp",
    "src/medical_basic_data_channel.cpp",
    "src/medical.cpp",
    "src/medical_channel_info.cpp"
  ]

  include_dirs = [
    "$SUBSYSTEM_DIR/utils/include",
  ]
}
```

**依赖**：无内部依赖（仅依赖外部组件）

---

### 3. 客户端框架目标（frameworks/native/medical_sensor/BUILD.gn）

**目标类型**：`ohos_shared_library`

**输出**：`libmedical_native.so`

**关键配置**：

```gn
ohos_shared_library("libmedical_native") {
  sources = [
    "src/medical_service_proxy.cpp",
    "src/medical_service_client.cpp",
    "src/my_file_descriptor_listener.cpp"
  ]

  deps = [
    "$SUBSYSTEM_DIR/services/medical_sensor:libmedical_service",
    "$SUBSYSTEM_DIR/utils:libmedical_utils"
  ]

  external_deps = [
    "c_utils:utils",
    "eventhandler:libeventhandler",
    "hilog:libhilog",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy"
  ]
}
```

**依赖关系**：
- 依赖 `libmedical_service`（服务端）
- 依赖 `libmedical_utils`（工具库）

---

### 4. 服务端目标（services/medical_sensor/BUILD.gn）

**目标类型**：`ohos_shared_library`

**输出**：`libmedical_service.z.so`（带 .z 后缀表示 SA 库）

**关键配置**：

```gn
ohos_shared_library("libmedical_service") {
  sources = [
    "hdi_connection/adapter/src/compatible_connection.cpp",
    "hdi_connection/adapter/src/hdi_connection.cpp",
    "hdi_connection/adapter/src/sensor_event_callback.cpp",
    "hdi_connection/hardware/src/hdi_service_impl.cpp",
    "hdi_connection/interface/src/sensor_hdi_connection.cpp",
    "src/client_info.cpp",
    "src/fifo_cache_data.cpp",
    "src/medical_data_processer.cpp",
    "src/medical_dump.cpp",
    "src/medical_manager.cpp",
    "src/medical_service.cpp",
    "src/medical_service_stub.cpp"
  ]

  deps = [ "$SUBSYSTEM_DIR/utils:libmedical_utils" ]

  external_deps = [
    "access_token:libaccesstoken_sdk",
    "c_utils:utils",
    "drivers_interface_sensor:libsensor_proxy_1.0",
    "drivers_peripheral_sensor:hdi_sensor",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy"
  ]

  cflags = [ "-Wno-error=inconsistent-missing-override" ]
}
```

**依赖关系**：
- 内部依赖 `libmedical_utils`
- 外部依赖 HDI 接口、访问控制、IPC、SA 框架

**证据**：
- 服务端构建：`services/medical_sensor/BUILD.gn:17-62`

---

### 5. Native 接口目标（interfaces/native/BUILD.gn）

#### NDK 目标

**目标类型**：`ohos_ndk_library`

**输出**：`medical.so`（NDK 符号）

**关键配置**：

```gn
ohos_ndk_library("libmedical_ndk") {
  output_name = "medical"
  ndk_description_file = "./libmedical.json"
  min_compact_version = "6"
}
```

#### Native 接口目标

**目标类型**：`ohos_shared_library`

**输出**：`medical_agent.so`

**关键配置**：

```gn
ohos_shared_library("medical_interface_native") {
  output_name = "medical_agent"
  sources = [ "src/medical_native_impl.cpp" ]

  deps = [
    "$SUBSYSTEM_DIR/frameworks/native/medical_sensor:libmedical_native",
    "$SUBSYSTEM_DIR/interfaces/native:libmedical_ndk",
    "$SUBSYSTEM_DIR/utils:libmedical_utils"
  ]

  external_deps = [
    "c_utils:utils",
    "eventhandler:libeventhandler",
    "hilog:libhilog",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy"
  ]
}
```

**依赖关系**：
- 依赖 `libmedical_native`（客户端框架）
- 依赖 `libmedical_ndk`（NDK）
- 依赖 `libmedical_utils`（工具库）

**证据**：
- Native 接口：`interfaces/native/BUILD.gn:23-50`

---

### 6. JS 插件目标（interfaces/plugin/BUILD.gn）

#### 动态库目标

**目标类型**：`ohos_shared_library`

**输出**：`medical.so`

**关键配置**：

```gn
ohos_shared_library("medical") {
  sources = [
    "src/medical_js.cpp",
    "src/medical_napi_utils.cpp"
  ]

  include_dirs = [
    "$SUBSYSTEM_DIR/interfaces/native/include",
    "$SUBSYSTEM_DIR/interfaces/plugin/include",
    "//third_party/libuv/include",
    "//third_party/node/src"
  ]

  defines = [
    "APP_LOG_TAG = \"medicalJs\"",
    "LOG_DOMAIN = 0xD002701"
  ]

  deps = [ "$SUBSYSTEM_DIR/interfaces/native:medical_interface_native" ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "napi:ace_napi"
  ]

  relative_install_dir = "module"
}
```

#### 静态库目标

**目标类型**：`ohos_static_library`

**输出**：`medical_static.a`

**关键配置**：

```gn
ohos_static_library("medical_static") {
  sources = [
    "src/medical_js.cpp",
    "src/medical_napi_utils.cpp"
  ]

  include_dirs = [
    "$SUBSYSTEM_DIR/interfaces/native/include",
    "$SUBSYSTEM_DIR/interfaces/plugin/include",
    "//third_party/libuv/include",
    "//third_party/node/src"
  ]

  defines = [
    "APP_LOG_TAG = \"medicalJs\"",
    "LOG_DOMAIN = 0xD002701"
  ]

  deps = [ "$SUBSYSTEM_DIR/interfaces/native:medical_interface_native" ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "napi:ace_napi"
  ]
}
```

**依赖关系**：
- 依赖 `medical_interface_native`（Native 接口）
- 外部依赖 N-API、日志、工具

**证据**：
- JS 插件：`interfaces/plugin/BUILD.gn:17-66`

---

## 依赖关系图

### 完整依赖树

```
bundle.json (组件入口)
├── sa_profile:medical_sa_profiles
│   └── 输出: 3605.xml
│
├── utils:medical_utils_target
│   └── libmedical_utils.so
│       └── [无内部依赖]
│
├── services:medical_service_target
│   └── libmedical_service.z.so
│       ├── deps: utils:libmedical_utils
│       └── external_deps: access_token, c_utils, drivers_interface_sensor,
│                           drivers_peripheral_sensor, hilog, hisysevent, ipc, safwk, samgr
│
├── frameworks:medical_native_target
│   └── libmedical_native.so
│       ├── deps: services:libmedical_service
│       │          utils:libmedical_utils
│       └── external_deps: c_utils, eventhandler, hilog, ipc, safwk, samgr
│
├── interfaces/native:medical_ndk_target
│   ├── libmedical_ndk (NDK)
│   │   └── 输出: medical.so (NDK 符号)
│   └── medical_interface_native (medical_agent.so)
│       ├── deps: frameworks:libmedical_native
│       │          interfaces/native:libmedical_ndk
│       │          utils:libmedical_utils
│       └── external_deps: c_utils, eventhandler, hilog, ipc, safwk, samgr
│
└── interfaces/plugin:medical_js_target
    └── medical (medical.so - JS 模块)
        ├── deps: interfaces/native:medical_interface_native
        └── external_deps: c_utils, hilog, napi:ace_napi
```

---

## 关键 Defines

### 日志域

| 模块 | 宏 | 值 | 说明 |
|------|-----|-----|------|
| N-API | `APP_LOG_TAG` | "medicalJs" |
| N-API | `LOG_DOMAIN` | 0xD002701 |
| 服务端 | 日志域 | 0xD002786 |

**证据**：
- N-API 日志：`interfaces/plugin/BUILD.gn:24-27`

---

## 全局变量

### SUBSYSTEM_DIR

**定义位置**：`medical_sensor.gni:16`

```gn
SUBSYSTEM_DIR = "//base/sensors/medical_sensor"
```

**用途**：所有子模块的构建配置都使用此变量引用源代码根目录。

---

## 内部接口导出

### inner_kits 定义（bundle.json）

```json
"inner_kits": [
  {
    "name": "//base/sensors/medical_sensor/interfaces/native:medical_interface_native",
    "header": {
      "header_files": [
        "medical_native_type.h",
        "medical_native_impl.h"
      ],
      "header_base": "//base/sensors/medical_sensor/interfaces/native/include"
    }
  }
]
```

**证据**：
- 内部接口：`bundle.json:43-53`

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [目录结构](01_Directory_Structure.md) - 代码组织详解
- [架构说明](02_Architecture.md) - 系统架构和数据流
- [编译产物](06_Build_Artifacts.md) - 输出文件和安装路径
