# GN 构建系统

## 根目录构建

### BUILD.gn

```gn
# 文件: BUILD.gn
import("//build/config/ohos/rules.gni")
import("//build/ohos.gni")

group("media_packages") {
  public_deps = [
    "interfaces/inner_api/native:media_client",
    "services:media_service",
  ]
}
```

### bundle.json

定义部件元数据：
- 部件名称: multimedia_player_framework
- 描述: 媒体播放框架
- 依赖: aafwk, graphic, etc.

### config.gni

构建配置：
- 编译选项
- 头文件路径
- 依赖配置

## frameworks 构建

### JS N-API 构建

```
frameworks/js/
├── media/BUILD.gn
├── player/BUILD.gn
├── recorder/BUILD.gn
├── avplayer/BUILD.gn
├── avrecorder/BUILD.gn
├── soundpool/BUILD.gn
├── metadatahelper/BUILD.gn
├── avscreen_capture/BUILD.gn
├── audio_haptic/BUILD.gn
├── system_sound_manager/BUILD.gn
└── ...
```

### CJ FFI 构建

```
frameworks/cj/
├── avplayer/BUILD.gn
├── avrecorder/BUILD.gn
├── avscreen_capture/BUILD.gn
├── avtranscoder/BUILD.gn
├── audio_haptic/BUILD.gn
├── metadatahelper/BUILD.gn
└── soundpool/BUILD.gn
```

## services 构建

### 服务构建

```
services/
├── services/BUILD.gn          # 媒体服务主构建
├── engine/BUILD.gn            # 引擎构建
├── engine/histreamer/BUILD.gn # HiStreamer 构建
├── utils/BUILD.gn             # 工具库构建
├── seccomp_policy/BUILD.gn    # 安全策略
└── dfx/BUILD.gn               # 诊断构建
```

## interfaces 构建

### kits 构建

```
interfaces/kits/
├── js/BUILD.gn    # JS N-API 头文件
└── c/BUILD.gn     # C API 头文件
```

## 关键 targets

### 服务端 targets

| target | 产物 | 职责 |
|--------|-----|------|
| `:media_service` | libmedia_service.z.so | 媒体服务主进程 |
| `:player_service` | libplayer_service.z.so | 播放服务 |
| `:recorder_service` | librecorder_service.z.so | 录制服务 |
| `:screen_capture_service` | libscreen_capture_service.z.so | 屏幕捕获服务 |

### 客户端 targets

| target | 产物 | 职责 |
|--------|-----|------|
| `:media_client` | libmedia_client.z.so | 媒体客户端库 |
| `:avplayer_napi` | libavplayer_napi.z.so | AVPlayer N-API |
| `:soundpool_napi` | libsoundpool_napi.z.so | SoundPool N-API |

## 构建配置

### 条件编译

```gn
if (is_standard_system) {
  # 标准系统构建配置
} else {
  # 轻量系统构建配置
}
```

### 依赖配置

```gn
deps = [
  "//foundation/aafwk:aafwk_core",
  "//foundation/graphic/graphic_2d:graphic_2d",
  "//third_party/cJSON:cJSON",
]
```

## 产物路径

| 产物类型 | 路径 |
|---------|------|
| 系统服务 | `/system/lib64/media_service/` |
| N-API 库 | `/system/lib64/` |
| 头文件 | `/include/multimedia/` |

## 相关文档

- [编译产物说明](12_Build_Artifacts.md)
