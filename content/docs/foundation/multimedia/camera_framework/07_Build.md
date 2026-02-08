# 构建与产物 (Build)

> OpenHarmony Camera Framework - GN 构建系统和编译产物

---

## 构建系统概览

Camera Framework 使用 **GN (Generate Ninja)** 构建系统：

```
BUILD.gn ──→ .ninja 文件 ──→ 编译产物
   │              │              │
   │              │              ├── .so (共享库)
   │              │              ├── .a (静态库)
   │              │              └── 可执行文件
   │              │
   └── .gni 配置 ─┘
```

---

## GN Targets 清单

### 1. ohos_shared_library (运行时共享库)

| Target | 输出文件名 | 路径 | 说明 |
|--------|-----------|------|------|
| `camera_utils` | `libcamera_utils.z.so` | `common/BUILD.gn:107` | 公共工具库 |
| `camera_framework` | `libcamera_framework.z.so` | `frameworks/native/camera/base/BUILD.gn:139` | 核心框架 |
| `camera_framework_ex` | `libcamera_framework_ex.z.so` | `frameworks/native/camera/extension/BUILD.gn:52` | 扩展框架 |
| `ohcamera` | `libohcamera.z.so` | `frameworks/native/ndk/BUILD.gn:30` | NDK 库 |
| `camera_napi` | `libcamera_napi.z.so` | `interfaces/kits/js/camera_napi/BUILD.gn:35` | N-API 主库 |
| `camera_napi_base` | `libcamera_napi_base.z.so` | `interfaces/kits/js/camera_napi/BUILD.gn:186` | N-API 基础 |
| `camerapicker_napi` | `libcamerapicker_napi.z.so` | `interfaces/kits/js/camera_napi/BUILD.gn:126` | 相机选择器 N-API |
| `resourcemanager_napi` | `libresourcemanager_napi.z.so` | `interfaces/kits/js/camera_napi/BUILD.gn:293` | 资源管理 N-API |
| `camera_napi_ex` | `libcamera_napi_ex.z.so` | `interfaces/kits/js/camera_napi_for_sys/BUILD.gn:18` | 扩展 N-API |
| `camera_service` | `libcamera_service.z.so` | `services/camera_service/BUILD.gn:17` | 相机服务 |
| `deferred_processing_service` | `libdeferred_processing_service.z.so` | `services/deferred_processing_service/BUILD.gn:58` | 延迟处理服务 |
| `media_stream` | `libmedia_stream.z.so` | `mediastream/BUILD.gn:32` | 媒体流处理 |
| `movie_file` | `libmovie_file.z.so` | `moviefile/BUILD.gn:32` | 视频文件处理 |

### 2. Dynamic Libraries (9个动态加载库)

| Target | 输出文件名 | 说明 |
|--------|-----------|------|
| `camera_dynamic_medialibrary` | `libcamera_dynamic_medialibrary.z.so` | 媒体库适配 |
| `camera_dynamic_picture` | `libcamera_dynamic_picture.z.so` | 图片框架适配 |
| `camera_dynamic_avcodec` | `libcamera_dynamic_avcodec.z.so` | 编解码适配 |
| `camera_dynamic_moving_photo` | `libcamera_dynamic_moving_photo.z.so` | 动态照片处理 |
| `camera_dynamic_media_manager` | `libcamera_dynamic_media_manager.z.so` | 媒体管理 |
| `camera_dynamic_notification` | `libcamera_dynamic_notification.z.so` | 通知服务 |
| `camera_dynamic_xcomponent_controller` | `libcamera_dynamic_xcomponent_controller.z.so` | XComponent 控制 |
| `camera_dynamic_image_effect` | `libcamera_dynamic_image_effect.z.so` | 图像效果 |
| `camera_dynamic_watermark_exif_metadata` | `libcamera_dynamic_watermark_exif_metadata.z.so` | 水印/EXIF |

### 3. ohos_static_library (静态库)

| Target | 输出文件名 | 路径 |
|--------|-----------|------|
| `camera_utils_static` | `libcamera_utils_static.a` | `common/BUILD.gn` |
| `camera_framework_static` | `libcamera_framework_static.a` | `frameworks/native/camera/base/BUILD.gn` |
| `test_common` | `libtest_common.a` | `test/test_common/BUILD.gn` |

### 4. ohos_source_set (源码集合)

| Target | 路径 | 说明 |
|--------|------|------|
| `camera_idl_sa_proxy` | `services/camera_service/idls/BUILD.gn:76` | 相机服务 IDL Proxy |
| `camera_idl_sa_stub` | `services/camera_service/idls/BUILD.gn:139` | 相机服务 IDL Stub |
| `camera_deferred_idl_sa_proxy` | `services/deferred_processing_service/idls/BUILD.gn:46` | 延迟服务 IDL Proxy |
| `camera_deferred_idl_sa_stub` | `services/deferred_processing_service/idls/BUILD.gn:98` | 延迟服务 IDL Stub |

### 5. ohos_executable (测试可执行文件)

| Target | 路径 | 说明 |
|--------|------|------|
| `camera_video` | `interfaces/inner_api/native/test/BUILD.gn:28` | 视频测试 |
| `camera_capture` | `interfaces/inner_api/native/test/BUILD.gn:65` | 拍照测试 |
| `camera_capture_video` | `interfaces/inner_api/native/test/BUILD.gn:100` | 录像测试 |
| `camera_capture_mode` | `interfaces/inner_api/native/test/BUILD.gn:135` | 模式测试 |

---

## Build Groups (来自 bundle.json)

### Framework Group (fwk_group)

```json
"fwk_group": [
  "//foundation/multimedia/camera_framework/common:camera_utils",
  "//foundation/multimedia/camera_framework/frameworks/native/camera/base:camera_framework",
  "//foundation/multimedia/camera_framework/frameworks/native/camera/extension:camera_framework_ex",
  "//foundation/multimedia/camera_framework/frameworks/native/ndk:ohcamera",
  "//foundation/multimedia/camera_framework/interfaces/kits/js/camera_napi:camera_napi",
  "//foundation/multimedia/camera_framework/interfaces/kits/js/camera_napi:camerapicker_napi",
  "//foundation/multimedia/camera_framework/frameworks/cj:cj_camera_ffi"
]
```

### Service Group (service_group)

```json
"service_group": [
  "//foundation/multimedia/camera_framework/dynamic_libs:camera_dynamic_*",
  "//foundation/multimedia/camera_framework/mediastream:media_stream",
  "//foundation/multimedia/camera_framework/moviefile:movie_file",
  "//foundation/multimedia/camera_framework/sa_profile:camera_service_sa_profile",
  "//foundation/multimedia/camera_framework/services/camera_service:camera_service",
  "//foundation/multimedia/camera_framework/services/deferred_processing_service:deferred_processing_service"
]
```

---

## Feature 开关配置

### 全局配置 (multimedia_camera_framework.gni)

```gn
declare_args() {
  # 相机功能
  camera_framework_feature_camera_rotate_plugin = false
  camera_framework_feature_camera_live_scene_recognition = false
  camera_framework_feature_moving_photo = true
  camera_framework_feature_beauty_notification = true
  camera_framework_feature_movie_file = true
  camera_framework_feature_media_stream = true
  camera_framework_feature_deferred = true
  camera_framework_feature_xcomponent_toast = true
  camera_framework_feature_rotate_param_update = true
  camera_framework_feature_capture_yuv = true
  
  # 编译选项
  fwk_no_hidden = false
}
```

### 条件编译使用

```cpp
// 代码中根据 feature 开关
#ifdef CAMERA_MOVING_PHOTO_ENABLE
  // 动态照片相关代码
#endif
```

---

## 编译产物路径

### 输出目录结构

```
out/{product}/{variant}/
├── system/
│   ├── lib/
│   │   ├── libcamera_service.z.so          # 相机服务
│   │   ├── libcamera_framework.z.so        # 框架
│   │   ├── libcamera_napi.z.so             # N-API
│   │   ├── libohcamera.z.so                # NDK
│   │   └── libcamera_dynamic_*.z.so        # 动态库
│   ├── lib64/                              # 64位版本
│   └── bin/
│       └── camera_service                  # 服务可执行文件
├── system/etc/
│   └── sa_profile/
│       └── 3008.json                       # SA 配置
└── obj/
    └── foundation/multimedia/camera_framework/
        └── *.o, *.a                        # 中间文件
```

### 头文件安装

```
out/{product}/{variant}/headers/
└── foundation/multimedia/camera_framework/
    ├── interfaces/inner_api/native/camera/include/
    └── interfaces/kits/native/include/camera/
```

---

## 构建命令

### 完整编译

```bash
# 编译相机框架及其依赖
./build.sh --product {product_name} \
           --target camera_framework

# 或编译整个多媒体子系统
./build.sh --product {product_name} \
           --target multimedia
```

### 增量编译

```bash
# 只编译相机服务
./build.sh --product {product_name} \
           --target //foundation/multimedia/camera_framework/services/camera_service

# 只编译 N-API
./build.sh --product {product_name} \
           --target //foundation/multimedia/camera_framework/interfaces/kits/js/camera_napi
```

### 带 Feature 开关编译

```bash
# 禁用动态照片功能
./build.sh --product {product_name} \
           --gn-args "camera_framework_feature_moving_photo=false"

# 调试模式
./build.sh --product {product_name} \
           --gn-args "fwk_no_hidden=true"
```

---

## 依赖关系

### 依赖图

```
camera_napi (JS API)
├── camera_napi_base
├── camera_framework
│   ├── camera_utils
│   ├── camera_idl_sa_proxy
│   └── camera_deferred_idl_sa_proxy
└── camera_framework_ex

camera_service (System Service)
├── camera_utils
├── camera_idl_sa_stub
├── deferred_processing_service
│   └── camera_deferred_idl_sa_stub
└── media_stream
    └── camera_dynamic_medialibrary
```

### 外部依赖

| 组件 | 用途 |
|------|------|
| `access_token` | 权限检查 |
| `window_manager` | Surface 管理 |
| `app_manager` | 应用状态 |
| `hdf_core` | HDI 接口 |
| `ipc` | Binder 通信 |
| `graphic_surface` | Buffer 管理 |
| `media_library` | 照片存储 |
| `av_codec` | 编解码 |

---

## 安装路径

### 运行时路径

| 文件类型 | 安装路径 |
|----------|----------|
| 共享库 | `/system/lib/` 或 `/system/lib64/` |
| SA 配置 | `/system/etc/sa_profile/3008.json` |
| 动态库 | `/system/lib/` (按需加载) |

### 开发 SDK

```
sdk/
└── {version}/
    ├── sysroot/
    │   ├── usr/include/camera/      # NDK 头文件
    │   └── lib/
    │       └── libohcamera.z.so     # NDK 库
    └── toolchains/
```

---

## 常见问题

### Q: 如何添加新的 GN target?

A: 在对应目录的 BUILD.gn 中添加：

```gn
ohos_shared_library("my_new_library") {
  sources = [ "src/my_file.cpp" ]
  deps = [ "//foundation/multimedia/camera_framework/common:camera_utils" ]
  external_deps = [ "hilog:libhilog" ]
}
```

### Q: 如何条件编译代码?

A: 在 .gni 中定义 args，在代码中使用：

```gn
# multimedia_camera_framework.gni
declare_args() {
  my_feature = true
}

# BUILD.gn
if (my_feature) {
  defines = [ "MY_FEATURE_ENABLE" ]
}
```

```cpp
// 代码中
#ifdef MY_FEATURE_ENABLE
  // 功能代码
#endif
```

---

## 下一步阅读

- [内部实现细节](./08_Internals.md) - 核心类和资源生命周期
- [目录结构](./03_CodeMap.md) - 代码文件导航
