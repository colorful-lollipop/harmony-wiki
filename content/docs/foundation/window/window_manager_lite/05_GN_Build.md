# GN 构建配置

## 构建概述

window_manager_lite 使用 **GN (Generate Ninja)** 构建系统，构建产物包括：

| 产物 | 类型 | 说明 |
|------|------|------|
| `wms_server` | 可执行文件 | WMS/IMS 服务端 |
| `libwms_client.so` | 共享库 | 客户端 IPC 代理库 |

**构建命令**:

```bash
hb build window_manager_lite
```

## 构建入口

**文件**: `BUILD.gn`

### 根目标 (lite_component)

```gn
// BUILD.gn:15-21
lite_component("window_manager_lite") {
  features = [
    ":wms_server",
    ":wms_client",
  ]
  public_deps = [ ":wms_client" ]
}
```

### NDK 目标 (ndk_lib)

```gn
// BUILD.gn:23-27
ndk_lib("window_manager_lite_ndk") {
  lib_extension = ".so"
  deps = [ ":wms_client" ]
  head_files = []
}
```

## 编译目标详解

### 1. wms_client - 客户端库

**类型**: `shared_library`

**输出**: `libwms_client.so`

```gn
// BUILD.gn:38-55
shared_library("wms_client") {
  sources = [
    "frameworks/ims/input_event_listener_proxy.cpp",
    "frameworks/wms/iwindows_manager.cpp",
    "frameworks/wms/lite_proxy_surface.cpp",
    "frameworks/wms/lite_proxy_window.cpp",
    "frameworks/wms/lite_proxy_windows_manager.cpp",
    "frameworks/wms/lite_win_requestor.cpp",
    "frameworks/wms/lite_wm_requestor.cpp",
    "frameworks/wms/lite_wms_client.cpp",
  ]

  deps = commonDeps
  public_deps = [ "//foundation/graphic/surface_lite:surface_lite" ]
  public_configs = [ ":wms_public_config" ]
  ldflags = [ "-lstdc++" ]
  cflags = [ "-Wall" ]
  cflags_cc = cflags
}
```

#### 依赖 (deps)

| 依赖项 | 说明 |
|--------|------|
| `//foundation/systemabilitymgr/samgr_lite/samgr:samgr` | SAMGR 框架 |
| `//foundation/systemabilitymgr/samgr_lite/communication/broadcast:broadcast` | 广播通信 |
| `//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single` | IPC 单例 |
| `//foundation/graphic/surface_lite:surface` | Surface 库 |
| `//foundation/graphic/graphic_utils_lite:utils_lite` | 图形工具库 |
| `//third_party/bounds_checking_function:libsec_shared` | 安全函数 |

#### 编译配置 (configs)

```gn
// BUILD.gn:57-59
config("wms_public_config") {
  include_dirs = [ "interfaces/innerkits" ]
}
```

### 2. wms_server - 服务端

**类型**: `executable`

**输出**: `wms_server` (可执行文件)

```gn
// BUILD.gn:71-108
executable("wms_server") {
  sources = [
    "services/wms/lite_win.cpp",
    "services/wms/lite_wm.cpp",
    "services/wms/lite_wms.cpp",
    "services/wms/samgr_wms.cpp",
    "services/wms/wms.cpp",
    # IMS sources
    "services/ims/input_event_distributer.cpp",
    "services/ims/input_event_hub.cpp",
    "services/ims/input_manager_service.cpp",
    "services/ims/input_event_client_proxy.cpp",
    "services/ims/samgr_ims.cpp",
  ]

  include_dirs = [
    "frameworks/ims",
    "interfaces/innerkits",
    "//base/security/permission_lite/services/pms_client/include",
    "//base/security/permission_lite/interfaces/innerkits",
    "//base/security/permission_lite/interfaces/kits",
    "//drivers/peripheral/input/interfaces/include",
    "//third_party/FreeBSD/sys/dev/evdev",
  ]

  ldflags = [
    "-lstdc++",
    "-lpthread",
    "-rpath-link=$ohos_root_path/$root_out_dir",
    "-ldisplay_gfx",
    "-ldisplay_gralloc",
    "-ldisplay_layer",
  ]

  deps = [
    "//base/security/permission_lite/services/pms_client:pms_client",
    "//drivers/peripheral/input/hal:hdi_input",
    "//foundation/graphic/graphic_utils_lite:lite_graphic_hals",
  ]

  deps += commonDms
  cflags = [ "-Wall" ]
  cflags_cc = cflags
}
```

#### 服务端额外依赖

| 依赖项 | 说明 |
|--------|------|
| `//base/security/permission_lite/services/pms_client:pms_client` | 权限管理客户端 |
| `//drivers/peripheral/input/hal:hdi_input` | 输入设备 HDI |
| `//foundation/graphic/graphic_utils_lite:lite_graphic_hals` | 图形 HAL 工具 |

#### 链接标志 (ldflags)

| 标志 | 说明 |
|------|------|
| `-ldisplay_gfx` | 显示图形库 |
| `-ldisplay_gralloc` | 显示内存分配库 |
| `-ldisplay_layer` | 显示图层库 |

## 公共依赖定义

```gn
// BUILD.gn:29-36
commonDeps = [
  "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  "//foundation/systemabilitymgr/samgr_lite/communication/broadcast:broadcast",
  "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
  "//foundation/graphic/surface_lite:surface",
  "//foundation/graphic/graphic_utils_lite:utils_lite",
  "//third_party/bounds_checking_function:libsec_shared",
]
```

## 产物映射

| 源文件 | 所属 Target | 输出产物 |
|--------|-------------|----------|
| `frameworks/wms/*.cpp` | wms_client | libwms_client.so |
| `frameworks/ims/*.cpp` | wms_client | libwms_client.so |
| `services/wms/*.cpp` | wms_server | wms_server |
| `services/ims/*.cpp` | wms_server | wms_server |

## 头文件配置

| 配置 | 路径 | 用途 |
|------|------|------|
| `wms_public_config` | `interfaces/innerkits` | 客户端公共头文件路径 |
| 服务端 include_dirs | 多路径 | 服务端头文件路径 |

## 构建产物安装位置

```
out/{product}/
├── lib/
│   └── libwms_client.so          # 客户端库
└── bin/
    └── wms_server                 # 服务端可执行文件
```

**注意**: 具体路径取决于产品配置。

## 编译配置开关

### IMS 编译配置

```gn
// BUILD.gn:61-69
imsSources = [
  "services/ims/input_event_distributer.cpp",
  "services/ims/input_event_hub.cpp",
  "services/ims/input_manager_service.cpp",
  "services/ims/input_event_client_proxy.cpp",
  "services/ims/samgr_ims.cpp",
]
imsInclude = [ "services/ims" ]
imsDeps = [ "//drivers/hdf_core/adapter/uhdf/posix:hdf_posix_osal" ]
```

## 组件配置 (bundle.json)

```json
{
  "name": "@ohos/window_manager_lite",
  "version": "3.1",
  "component": {
    "name": "window_manager_lite",
    "subsystem": "window",
    "adapted_system_type": [ "small" ],
    "rom": "110KB",
    "ram": "~50KB",
    "deps": {
      "third_party": [ "bounds_checking_function" ],
      "components": [
        "samgr_lite",
        "surface_lite",
        "drivers_peripheral_input",
        "ipc",
        "graphic_utils_lite",
        "hdf_core",
        "permission_lite"
      ]
    }
  }
}
```
