# GN 构建配置

## 概述

HiStreamer 使用 **GN (Generate Ninja)** 作为构建系统，配置文件位于：
- 根目录: `BUILD.gn`, `config.gni`
- 模块目录: 各子目录的 `BUILD.gn`

## Feature Flags

### 插件开关 (Plugin Features)

| Feature Flag | 默认值 | Standard | Small | Mini | 说明 |
|--------------|--------|----------|-------|------|------|
| `media_foundation_enable_plugin_ffmpeg_adapter` | false | true | true | - | FFmpeg 适配器 |
| `media_foundation_enable_plugin_codec_adapter` | false | true | - | - | HDI Codec 适配器 |
| `media_foundation_enable_plugin_file_source` | false | - | true | true | 文件源 |
| `media_foundation_enable_plugin_file_fd_source` | false | - | true | true | FD 文件源 |
| `media_foundation_enable_plugin_http_source` | false | - | true | true | HTTP 源 |
| `media_foundation_enable_plugin_hdi_adapter` | false | - | true | true | HDI 音频适配器 |
| `media_foundation_enable_plugin_minimp3_adapter` | false | - | - | - | Minimp3 适配器 |
| `media_foundation_enable_plugin_minimp4_demuxer` | false | - | - | - | Minimp4 Demuxer |
| `media_foundation_enable_plugin_aac_demuxer` | false | - | - | - | AAC Demuxer |
| `media_foundation_enable_plugin_wav_demuxer` | false | - | - | - | WAV Demuxer |
| `media_foundation_enable_plugin_lite_aac_decoder` | false | - | - | - | Lite AAC 解码器 |
| `media_foundation_enable_plugin_std_audio_capture` | false | true | - | - | 标准音频采集 |
| `media_foundation_enable_plugin_audio_server_sink` | false | true | - | - | 音频服务器 Sink |
| `media_foundation_enable_plugin_std_video_surface_sink` | false | true | - | - | 视频 Surface Sink |
| `media_foundation_enable_plugin_std_video_capture` | false | - | - | - | 标准视频采集 |

### 功能开关 (Core Features)

| Feature Flag | 默认值 | 说明 |
|--------------|--------|------|
| `media_foundation_enable_recorder` | false | 录制功能 |
| `media_foundation_enable_video` | false | 视频功能 |
| `media_foundation_enable_avs3da` | false | AVS3DA 音频 |
| `media_foundation_enable_ffrt` | false | 使用 FFRT 替代 pthread |
| `media_foundation_enable_rm_demuxer` | true | RM Demuxer |
| `media_foundation_enable_cook_audio_decoder` | false | Cook 音频解码 |
| `media_foundation_enable_eac3_audio_decoder` | false | EAC3 音频解码 |
| `media_foundation_enable_lrc_demuxer` | false | LRC 歌词 Demuxer |
| `media_foundation_enable_sami_demuxer` | false | SAMI 字幕 Demuxer |
| `media_foundation_enable_ass_demuxer` | false | ASS 字幕 Demuxer |
| `media_foundation_enable_eac3_demuxer` | false | EAC3 Demuxer |
| `media_foundation_enable_dtshd_demuxer` | false | DTS-HD Demuxer |
| `media_foundation_enable_truehd_demuxer` | false | TrueHD Demuxer |

### 系统类型变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `hst_is_lite_sys` | bool | Lite 系统（Mini/Small） |
| `hst_is_standard_sys` | bool | Standard 系统 |
| `hst_is_small_sys` | bool | Small 系统 |
| `hst_is_mini_sys` | bool | Mini 系统 (L0) |

## 根级 Targets

### BUILD.gn

**路径**: `BUILD.gn`

```gn
config("histreamer_presets") {
    include_dirs = [
        "//foundation/multimedia/media_foundation/engine",
    ]
    defines = [
        "HST_ANY_WITH_NO_RTTI",
        "MEDIA_OHOS",
    ]
    # 根据 feature flags 添加 defines
}

# Standard/L1 系统
group("histreamer") {
    deps = [
        "engine/plugin/plugins:histreamer_plugin_store",
    ]
}

group("media_foundation") {
    deps = [
        "src:media_foundation",
        "src/capi:capi_packages",
    ]
}

# Small/Lite 系统
lite_library("media_engine_histreamer") {
    target_type = "shared_library"
    output_name = "media_foundation"
    deps = [
        "engine/scene/player:histreamer_player",
    ]
}

# Mini/L0 系统
lite_library("histreamer") {
    target_type = "static_library"
    complete_static_lib = true
    deps = [
        "engine/plugin/plugins:histreamer_plugin_store",
        "engine/scene/player:histreamer_player",
    ]
}
```

## 核心 Targets

### src/BUILD.gn

```gn
ohos_shared_library("media_foundation") {
    sources = [
        # Buffer
        "buffer/av_hardware_memory.cpp",
        "buffer/avbuffer.cpp",
        # ...
    ]

    deps = [
        "//third_party/cjson:cjson",
        "//third_party/libpng:png",
        "//foundation/multimedia/media_utils:media_utils",
        "//foundation/graphic/graphic_2d:graphic_2d",
        "//base/hiviewdfx/hisysevent/interfaces/native/kits:hisysevent_kits",
        "//base/startup/init/services:init_services",
        "//drivers/interface/display/v1.1:display_hdi_call",
        "//drivers/interface/codec/v3_0:codec_hdi_call",
        "//foundation/afms:afms_client",
        "//foundation/multimedia/ffmpeg_kit:ffmpeg_kit",
        "//foundation/multimedia/media_lite:media_lite",
        "//utils/native/base:utils_nbase",
        "//foundation/multimedia/media_utils:media_utils",
        "//third_party/crypto:openssl_crypto",
        "//third_party/curl:curl",
        "//foundation/systemabilityguressor/samgr/samgr_lite:samgr_lite",
        "//foundation/multimedia/media_lite/audio_lite:audio_lite",
        "//foundation/window_manager/window_manager_service:window_managerservice",
    ]

    external_deps = [
        "c_utils:utils",
        "hilog_native:hilog",
        "ipc:ipc_single",
        "safwk:safwk",
        "samgr:samgr",
        "appexecfwk:appexecfwk_innerkit",
        "graphic_surface:surface",
        "bounds_checking_function:sec_shared_buffers",
    ]
}
```

### src/capi/BUILD.gn

```gn
ohos_shared_library("native_media_core") {
    sources = [
        "native_avbuffer.cpp",
        "native_avformat.cpp",
        "native_avmemory.cpp",
        "common/native_mfmagic.cpp",
    ]

    deps = [
        ":media_foundation",
        "//utils/native/base:utils_nbase",
    ]

    external_deps = [
        "hilog_native:hilog_inner",
        "ipc:ipc_single",
    ]
}
```

### engine/pipeline/BUILD.gn

```gn
ohos_shared_library("libhistreamer_base") {
    sources = [
        "core/pipeline_core.cpp",
        "core/filter_base.cpp",
        "core/port.cpp",
    ]

    deps = [
        ":pipeline_base",
    ]
}

ohos_shared_library("libhistreamer_codec_filters") {
    sources = [
        "filters/codec/*.cpp",
    ]

    deps = [
        ":codec_filters",
    ]
}
```

## 插件 Targets

### engine/plugin/plugins/BUILD.gn

```gn
group("histreamer_plugin_store") {
    deps = [
        ":gen_plugin_static_header",
        # 根据 feature flags 条件编译
        ":plugin_ffmpeg_adapter",
        ":plugin_codec_adapter",
        # ...
    ]
}

# FFmpeg 适配器
ohos_shared_library("histreamer_plugin_FFmpegDemuxer") {
    output_name = "libFFmpegDemuxer"
    sources = [
        "ffmpeg_adapter/demuxer/*.cpp",
    ]

    deps = [
        "//foundation/multimedia/media_foundation/engine/plugin:ffmpeg_convert",
    ]

    install_install = true
    install_deps = [
        "$install_output_path/media/histreamer_plugins",
    ]
}
```

## 依赖配置

### external_deps

| 依赖 | 用途 |
|------|------|
| `hilog_native:hilog` | 日志 |
| `ipc:ipc_single` | 进程间通信 |
| `safwk:safwk` | 系统能力框架 |
| `samgr:samgr` | 服务管理 |
| `graphic_surface:surface` | 图形表面 |
| `c_utils:utils` | C 工具库 |
| `bounds_checking_function:*` | 边界检查 |

### third_party 依赖

| 依赖 | 用途 |
|------|------|
| `ffmpeg` | 编解码、格式处理 |
| `curl` | HTTP 下载 |
| `libpng` | PNG 图像 |
| `openssl` | SSL/TLS |
| `cjson` | JSON 解析 |

## 构建配置变量

### config.gni

```gn
declare_args() {
    histreamer_root_dir = "//foundation/multimedia/media_foundation"
    multimedia_root_dir = "//foundation/multimedia"

    # 插件开关
    media_foundation_enable_plugin_ffmpeg_adapter = false
    media_foundation_enable_plugin_codec_adapter = false
    # ...

    # 功能开关
    media_foundation_enable_recorder = false
    media_foundation_enable_video = false

    # 线程配置
    config_ohos_histreamer_stack_size = 0  # 0 = 系统默认
}

# 根据 ohos_lite 配置系统类型
if (!defined(ohos_lite) || !ohos_lite) {
    hst_is_standard_sys = true
} else {
    hst_is_lite_sys = true
    if (ohos_kernel_type == "liteos_m") {
        hst_is_mini_sys = true
    } else {
        hst_is_small_sys = true
    }
}
```

## 构建命令示例

### Standard 系统

```bash
# 启用所有 Standard 功能
./build.sh --product-name rk3568 --build-target histreamer,media_foundation
```

### Small 系统

```bash
# 启用 Small 功能
./build.sh --product-name w800 --build-target media_foundation
```

### Mini 系统

```bash
# 启用 Mini 功能
./build.sh --product-name hispark_aries --build-target histreamer
```

### 自定义 Feature

```bash
# 启用 FFmpeg 插件
build --gn-args "media_foundation_enable_plugin_ffmpeg_adapter=true"
```
