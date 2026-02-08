# 04 - 构建系统

> Camera_Lite GN构建配置详解

---

## 构建概述

Camera_Lite 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统。

### 构建入口

```bash
# 在OpenHarmony源码根目录下
hb set  # 选择开发板
hb build camera_lite  # 构建camera_lite组件
```

### 构建文件位置

| 文件 | 路径 | 说明 |
|------|------|------|
| 组件配置 | `bundle.json` | 组件元数据 |
| 客户端构建 | `frameworks/BUILD.gn` | libcamera_lite.so |
| 服务端构建 | `services/BUILD.gn` | libcamera_server.so |

---

## GN Targets

### frameworks/BUILD.gn

**文件**: `frameworks/BUILD.gn`

#### Target: camera_lite (shared_library)

**产物**: `libcamera_lite.so`

**源码文件** (基础):
```gn
sources = [
  "camera_ability.cpp",
  "camera_client.cpp",
  "camera_config.cpp",
  "camera_impl.cpp",
  "camera_info_impl.cpp",
  "camera_kit.cpp",
  "camera_manager.cpp",
  "event_handler.cpp",
  "frame_config.cpp",
]
```

**模式相关源码**:

**Passthrough模式** (`enable_media_passthrough_mode == true`):
```gn
sources += [
  "../services/impl/src/camera_device.cpp",
  "../services/impl/src/camera_service.cpp",
  "passthrough/src/camera_device_client.cpp",
  "passthrough/src/camera_service_client.cpp",
]

include_dirs += [
  "//drivers/peripheral/display/interfaces/include",
  "//drivers/peripheral/codec/interfaces/include",
  "//foundation/multimedia/camera_lite/frameworks/passthrough/include",
  "//foundation/multimedia/media_utils_lite/hals",
  "//drivers/peripheral/base",
]

ldflags = [ "-lhdi_camera" ]
defines = [ "ENABLE_PASSTHROUGH_MODE" ]
```

**Binder模式** (默认):
```gn
sources += [
  "binder/src/camera_device_client.cpp",
  "binder/src/camera_service_client.cpp",
]

include_dirs += [
  "//foundation/multimedia/camera_lite/frameworks/binder/include",
  "//commonlibrary/utils_lite/include",
  "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
  "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
  "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast",
  "//foundation/communication/ipc/interfaces/innerkits/c/ipc/include/",
]
```

**公共配置**:
```gn
include_dirs += [
  "//foundation/multimedia/camera_lite/frameworks",
  "//foundation/multimedia/camera_lite/interfaces/kits",
  "//foundation/multimedia/camera_lite/services/impl/include",
  "//foundation/multimedia/camera_lite/services/server/include",
  "//foundation/multimedia/media_utils_lite/interfaces/kits",
  "//base/security/permission_lite/interfaces/kits",
]

cflags = [ "-fPIC", "-Wall" ]

deps = [
  "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
  "//base/security/permission_lite/services/pms_client:pms_client",
  "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
  "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  "//third_party/bounds_checking_function:libsec_shared",
]

public_deps = [
  "//foundation/graphic/surface_lite:surface_lite",
  "//foundation/multimedia/media_lite/frameworks/recorder_lite:recorder_lite",
  "//foundation/multimedia/media_utils_lite:media_common",
]
```

**板级特定配置** (hispark_taurus / hispark_aries):
```gn
if (board_name == "hispark_taurus" || board_name == "hispark_aries") {
  ldflags += [
    "-lcodec",
    "-lhdi_videodisplayer",
  ]
  deps += [
    "//device/soc/hisilicon/common/hal/media:hardware_media_sdk",
    "//device/soc/hisilicon/common/hal/middleware:middleware_source_sdk",
  ]
}
```

**证据**: `frameworks/BUILD.gn:14-104`

---

### services/BUILD.gn

**文件**: `services/BUILD.gn`

#### Target: camera_server (shared_library)

**产物**: `libcamera_server.so`

**源码文件**:
```gn
sources = [
  "impl/src/camera_device.cpp",
  "impl/src/camera_service.cpp",
  "server/src/camera_server.cpp",
  "server/src/samgr_camera.cpp",
]
```

**包含路径**:
```gn
include_dirs = [
  "//foundation/multimedia/camera_lite/services/impl/include",
  "//foundation/multimedia/camera_lite/services/server/include",
]
```

**外部库配置**:
```gn
public_configs = [ ":external_camera_server_library" ]
```

**链接标志**:
```gn
ldflags = [
  "-lstdc++",
  "-lcodec",
  "-lhdi_camera",
  "-lhdi_videodisplayer",
  "-lpthread",
  "-Wl,-rpath-link=$ohos_root_path/$root_out_dir",
]
```

**编译选项**:
```gn
cflags = [ "-Wall", "-fPIC" ]
```

**依赖**:
```gn
deps = [
  "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
  "//base/security/permission_lite/services/pms_client:pms_client",
  "//device/soc/hisilicon/common/hal/media:hardware_media_sdk",
  "//device/soc/hisilicon/common/hal/middleware:middleware_source_sdk",
  "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  "//third_party/bounds_checking_function:libsec_shared",
]

public_deps = [
  "//foundation/graphic/surface_lite:surface_lite",
  "//foundation/multimedia/camera_lite/frameworks:camera_lite",
  "//foundation/multimedia/media_utils_lite:media_common",
]
```

**证据**: `services/BUILD.gn:14-52`

---

## 编译产物

### 输出文件

| 产物 | 类型 | 路径 | 说明 |
|------|------|------|------|
| libcamera_lite.so | 共享库 | `out/{board}/libs/` | 客户端库 |
| libcamera_server.so | 共享库 | `out/{board}/libs/` | 服务端库 |

### 安装路径

根据系统类型，产物安装到不同位置：

**Mini系统**:
```
/system/lib/libcamera_lite.so
/system/lib/libcamera_server.so
```

**Small系统**:
```
/system/lib/libcamera_lite.so
/system/lib/libcamera_server.so
```

**证据**: 产物通过 `bundle.json` 中的 `sub_component` 指定:
```json
"build": {
  "sub_component": [
    "//foundation/multimedia/camera_lite/frameworks:camera_lite"
  ]
}
```

---

## 依赖关系

### 外部依赖组件

| 组件 | 用途 | 依赖类型 |
|------|------|----------|
| hilog_lite | 日志输出 | deps |
| permission_lite | 权限检查 | deps |
| surface_lite | 图形Surface | public_deps |
| media_utils_lite | 媒体工具 | public_deps |
| media_lite | 录制器 | public_deps |
| ipc | IPC通信 (Binder模式) | deps |
| samgr_lite | 系统能力管理 | deps |
| bounds_checking_function | 安全函数 | deps |

### HAL依赖

| 库 | 用途 |
|----|------|
| hdi_camera | 相机HAL接口 |
| hdi_videodisplayer | 视频显示HAL |
| codec | 编解码器 |

---

## Feature Flags

### 编译开关

| 宏定义 | 定义位置 | 说明 |
|--------|----------|------|
| ENABLE_PASSTHROUGH_MODE | `frameworks/BUILD.gn:43` | 启用直通模式 |
| __LINUX__ | 系统定义 | Linux平台标识 |

### 运行时配置

| 配置项 | 取值 | 说明 |
|--------|------|------|
| enable_media_passthrough_mode | true/false | 选择运行模式 |
| board_name | hispark_taurus/hispark_aries/... | 目标开发板 |

---

## 构建命令详解

### 完整构建流程

```bash
# 1. 进入OpenHarmony源码根目录
cd /path/to/openharmony

# 2. 设置构建环境
source build/envsetup.sh

# 3. 选择开发板
hb set
# 选择: hispark_taurus 或其他支持的开发板

# 4. 构建camera_lite
hb build camera_lite

# 5. 构建产物位置
ls out/hispark_taurus/libs/libcamera_*.so
```

### 单仓构建

```bash
# 在camera_lite目录下
cd foundation/multimedia/camera_lite

# 使用gn直接构建 (需要配置好环境)
gn gen out/Default
ninja -C out/Default camera_lite
gn gen out/Default
ninja -C out/Default camera_server
```

---

## 产物验证

### 检查产物

```bash
# 检查库文件
file out/hispark_taurus/libs/libcamera_lite.so

# 查看符号表
nm -D out/hispark_taurus/libs/libcamera_lite.so | grep CameraKit

# 查看依赖
readelf -d out/hispark_taurus/libs/libcamera_lite.so | grep NEEDED
```

### 运行时加载

```bash
# 查看运行时库加载路径
cat /system/etc/ld.so.conf

# 或设置环境变量
export LD_LIBRARY_PATH=/system/lib:$LD_LIBRARY_PATH
```

---

## 常见问题

### 编译错误

**问题**: `ERROR: target not found: camera_lite`

**解决**: 确保 `bundle.json` 正确配置，并在正确目录执行 `hb set`

---

**问题**: `undefined reference to 'HalCameraXXX'`

**解决**: 确保HAL库已编译并链接，检查 `board_name` 配置

---

**问题**: `cannot find -lhdi_camera`

**解决**: 检查驱动HAL层是否已编译，确保 `ohos_root_path` 设置正确

---

### 链接问题

**问题**: 运行时找不到 `libcamera_lite.so`

**解决**: 
1. 检查产物是否正确安装到 `/system/lib/`
2. 检查 `LD_LIBRARY_PATH` 是否包含 `/system/lib/`
3. 检查文件权限

---

## 下一步

- [安全风险](05_Security.md) - 安全问题分析
- [常见问题](06_Troubleshooting.md) - 更多问题排查
