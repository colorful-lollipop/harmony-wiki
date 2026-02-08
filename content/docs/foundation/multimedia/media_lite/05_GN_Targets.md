# GN Targets

## 目的

本文档介绍 media_lite 项目的 GN 构建系统，包括关键 targets、类型、依赖关系和产物映射。

## 适用范围

- 适用于构建工程师
- 适用于需要修改构建配置的开发者
- 包含所有核心 targets 和编译产物

---

## 关键构建文件

| 文件路径 | 说明 |
|----------|------|
| `config.gni` | 全局配置文件（enable_multimedia_camera_lite 开关） |
| `services/BUILD.gn` | 媒体服务主入口和组件聚合 |
| `services/player_lite/BUILD.gn` | 播放器服务端和实现 |
| `services/recorder_lite/BUILD.gn` | 录音器服务端和实现 |
| `frameworks/player_lite/BUILD.gn` | 播放器框架层 |
| `frameworks/recorder_lite/BUILD.gn` | 录音器框架层 |
| `interfaces/kits/player_lite/js/builtin/BUILD.gn` | JSI 音频 API |

---

## Targets 分类与详情

### Executable（可执行文件）

#### media_server

| 属性 | 值 |
|------|-----|
| 所在文件 | `services/BUILD.gn:18-71` |
| 类型 | executable |
| 源文件 | `media_main.cpp` |
| 输出名 | `media_server` |
| 输出目录 | `$root_out_dir` |
| 条件 defines | `ENABLE_PASSTHROUGH_MODE`, `SUPPORT_CAMERA_LITE` |
| 依赖 | `camera_server`, `samgr`, `audio_capturer_impl`, `audio_capturer_server`, `player_server`, `recorder_server` |
| 说明 | 媒体服务主入口，初始化各子服务 |

#### test_play_file_h265

| 属性 | 值 |
|------|-----|
| 所在文件 | `test/BUILD.gn:24-35` |
| 类型 | executable |
| 源文件 | `test_play_file_h265.cpp` |
| 输出名 | `test_play_file_h265` |
| 输出目录 | `$root_out_dir` |
| 说明 | H265 解码测试程序 |

---

### Shared Library（共享库）

#### player_impl

| 属性 | 值 |
|------|-----|
| 所在文件 | `services/player_lite/BUILD.gn:18-53` |
| 类型 | shared_library |
| 源文件 | `player_impl.cpp`, `player_control/`, `source/`, `decoder/`, `sink/`, `player/`, `buffersource/` |
| 输出名 | `libplayer_impl.so` |
| 依赖 | `hilog_shared`, `surface_lite`, `media_common`, `libsec_shared` |
| 外部依赖 | `hardware_media_sdk`（histreamer 播放器） |
| configs | `player_impl_external_library_config` |
| 说明 | 播放器核心实现库，包含播放控制、解码、同步等 |

#### player_server

| 属性 | 值 |
|------|-----|
| 所在文件 | `services/player_lite/BUILD.gn:55-72` |
| 类型 | shared_library |
| 源文件 | `player_server.cpp`, `factory/` |
| 输出名 | `libplayer_server.so` |
| 依赖 | `surface_lite`, `media_engine_histreamer`, `samgr`, `libbegetutil`, `player_impl`, `media_common` |
| 说明 | 播放器服务端，处理 IPC 请求和回调 |

#### recorder_impl

| 属性 | 值 |
|------|-----|
| 所在文件 | `services/recorder_lite/BUILD.gn:18-54` |
| 类型 | shared_library |
| 源文件 | `recorder_impl.cpp`, `recorder_video_source.cpp`, `recorder_audio_source.cpp`, `recorder_sink.cpp`, `recorder_data_source.cpp` |
| 条件 defines | `ENABLE_PASSTHROUGH_MODE` |
| 依赖 | `hardware_media_sdk`, `audio_capturer_impl`, `media_common`, `libsec_shared` |
| 说明 | 录音器实现库，包含音视频源、编码、封装 |

#### recorder_server

| 属性 | 值 |
|------|-----|
| 所在文件 | `services/recorder_lite/BUILD.gn:56-85` |
| 类型 | static_library |
| 源文件 | `recorder_service.cpp` |
| 输出名 | `librecorder_server.a` |
| 依赖 | `pms_client`, `ipc_single`, `surface_lite`, `audio_capturer_impl`, `recorder_impl`, `media_common`, `samgr`, `libsec_shared` |
| 说明 | 录音器服务端，处理 IPC 请求分发 |

#### player_lite（框架）

| 属性 | 值 |
|------|-----|
| 所在文件 | `frameworks/player_lite/BUILD.gn:17-82` |
| 类型 | shared_library |
| 条件 | `if (ohos_kernel_type != "liteos_m")` |
| 源文件（Binder 模式） | `binder/player.cpp`, `binder/player_client.cpp` |
| 源文件（Passthrough 模式） | `passthrough/liteplayer/player.cpp`, `passthrough/liteplayer/player_client.cpp` |
| 输出名 | `libplayer_lite.so` |
| 依赖 | IPC/SAMGR（Binder 模式）、`player_server`（Passthrough 模式）、`media_common`, `surface_lite`, `pms_client` |
| configs | `player_external_library_config` |
| 说明 | 播放器框架层，支持两种运行模式 |

#### recorder_lite（框架）

| 属性 | 值 |
|------|-----|
| 所在文件 | `frameworks/recorder_lite/BUILD.gn:17-72` |
| 类型 | shared_library |
| 源文件 | `recorder.cpp`, `binder/recorder_client.cpp`, `passthrough/recorder_client.cpp` |
| 输出名 | `librecorder_lite.so` |
| 条件 sources | `if (enable_media_passthrough_mode == false)` | `binder/recorder_client.cpp` |
| 条件 sources | `if (enable_media_passthrough_mode == true)` | `passthrough/recorder_client.cpp` |
| 依赖 | `media_common`, `surface_lite`, `pms_client`, `recorder_server`（Passthrough 模式） |
| configs | `recorder_external_library_config` |
| 说明 | 录音器框架层，支持两种运行模式 |

#### audio_lite_api

| 属性 | 值 |
|------|-----|
| 所在文件 | `interfaces/kits/player_lite/js/builtin/BUILD.gn:24-55` |
| 类型 | lite_library（条件） |
| 条件 | `if (ohos_kernel_type == "liteos_m")` → static_library |
| 条件 | `if (ohos_kernel_type != "liteos_m")` → shared_library |
| 源文件 | `audio_module.cpp`, `audio_player.cpp` |
| 输出名 | `libaudio_lite_api.so/.a` |
| 依赖 | `player_lite`, `media_utils_lite` |
| 说明 | JSI 音频 API，根据内核类型选择库格式 |

---

### Static Library（静态库）

#### player_lite（LiteOS-M）

| 属性 | 值 |
|------|-----|
| 所在文件 | `frameworks/player_lite/BUILD.gn:98-119` |
| 类型 | static_library |
| 条件 | `if (ohos_kernel_type == "liteos_m")` |
| 源文件 | `passthrough/histreamer/player.cpp` |
| 输出名 | `libplayer_lite.a` |
| 依赖 | `hilog_static`, `media_foundation:histreamer` |
| 说明 | LiteOS-M 内核播放器，静态链接 histreamer |

---

### Group（目标组）

#### lite_medialite_test

| 属性 | 值 |
|------|-----|
| 所在文件 | `test/unittest/BUILD.gn` |
| 类型 | group |
| 条件 | `if (is_debug == true)` |
| 子 targets | `lite_player_unittest`, `lite_recorder_unittest` |
| 说明 | 单元测试组（仅 debug 构建） |

#### media_lite（组件）

| 属性 | 值 |
|------|-----|
| 所在文件 | `services/BUILD.gn:90-100` |
| 类型 | lite_component |
| 子 targets | `media_server`, `media_common`, `camera_lite`, `player_lite`, `recorder_lite`, `player_server`, `audio_capturer_lite` |
| 说明 | 媒体组件聚合，用于 bundle.json 构建 |

---

### NDK Library

#### media_ndk

| 属性 | 值 |
|------|-----|
| 所在文件 | `services/BUILD.gn:102-119` |
| 类型 | ndk_lib |
| lib_extension | `.so` |
| 头文件 | player.h, recorder.h, media_info.h 等 |
| 依赖 | `audio_capturer_lite`, `camera_lite`, `player_lite`, `recorder_lite`, `media_common` |
| 输出名 | `libmedia_ndk.so` |
| 说明 | 多媒体 NDK 库，暴露核心接口给应用 |

---

### Unit Test（单元测试）

#### lite_player_unittest

| 属性 | 值 |
|------|-----|
| 所在文件 | `test/unittest/BUILD.gn` |
| 类型 | unittest |
| 子 targets | `lite_player_unittest` → `player_lite` |
| 输出扩展名 | `.bin` |
| 输出路径 | `$root_out_dir/test/unittest/playerlite/` |
| 说明 | 播放器单元测试 |

#### lite_recorder_unittest

| 属性 | 值 |
|------|-----|
| 所在文件 | `test/unittest/BUILD.gn` |
| 类型 | unittest |
| 子 targets | `lite_recorder_unittest` → `recorder_lite` |
| 输出扩展名 | `.bin` |
| 输出路径 | `$root_out_dir/test/unittest/recorder/` |
| 说明 | 录音器单元测试 |

---

## 关键 Defines

### 条件编译开关

| Define | 所在文件 | 条件 | 说明 |
|--------|----------|------|------|
| ENABLE_PASSTHROUGH_MODE | 多个 BUILD.gn | enable_media_passthrough_mode == true | 启用透传模式，减少 IPC 开销 |
| SUPPORT_CAMERA_LITE | services/BUILD.gn | enable_multimedia_camera_lite == true | 支持相机轻量版 |
| `__OHOS_KERNEL__LITEOS_M__` | services/BUILD.gn | ohos_kernel_type == "liteos_m" | LiteOS-M 内核标记 |

### 全局配置变量

| 变量 | 所在文件 | 类型 | 说明 |
|------|----------|------|------|
| enable_multimedia_camera_lite | config.gni:14-20 | boolean | 是否启用 camera_lite 支持 |

---

## 依赖关系图

### media_server 完整依赖

```
media_server (executable)
├── camera_server (external)
├── samgr (external)
├── audio_capturer_impl (external)
├── audio_capturer_server (external)
├── player_server (shared_library)
│   ├── surface_lite (external)
│   ├── media_engine_histreamer (external)
│   ├── samgr (external)
│   ├── libbegetutil (external)
│   ├── player_impl (shared_library)
│   │   ├── hilog_shared (external)
│   │   ├── surface_lite (external)
│   │   ├── media_common (external)
│   │   └── libsec_shared (external)
│   └── media_common (external)
└── recorder_server (static_library)
    ├── pms_client (external)
    ├── ipc_single (external)
    ├── surface_lite (external)
    ├── audio_capturer_impl (external)
    ├── recorder_impl (shared_library)
    │   ├── hardware_media_sdk (external)
    │   ├── audio_capturer_impl (external)
    │   ├── media_common (external)
    │   └── libsec_shared (external)
    ├── media_common (external)
    ├── samgr (external)
    └── libsec_shared (external)
```

### player_lite 依赖

```
player_lite (shared_library)
├── IPC/SAMGR (deps, non-passthrough)
│   ├── ipc_single
│   └── samgr
├── player_server (passthrough mode)
│   └── player_impl
├── media_common (public_deps)
├── surface_lite (public_deps)
└── pms_client (deps, non-passthrough)
```

---

## Target 与产物映射

### 主要产物

| Target | 输出文件 | 安装路径 | 运行时加载关系 |
|--------|------------|----------|----------------|
| `media_server` | `media_server` | `/usr/bin/` 或 `/system/bin/` | 独立进程，服务入口 |
| `libplayer_impl.so` | `libplayer_impl.so` | `/usr/lib/` 或 `/system/lib/` | 被 player_server 加载 |
| `libplayer_server.so` | `libplayer_server.so` | `/usr/lib/` 或 `/system/lib/` | 被 media_server 加载 |
| `librecorder_impl.so` | `librecorder_impl.so` | `/usr/lib/` 或 `/system/lib/` | 被 recorder_server 加载 |
| `librecorder_server.a` | `librecorder_server.a` | 静态链接到 media_server |
| `libplayer_lite.so` | `libplayer_lite.so` | `/usr/lib/` 或 `/system/lib/` | 被应用动态链接 |
| `librecorder_lite.so` | `librecorder_lite.so` | `/usr/lib/` 或 `/system/lib/` | 被应用动态链接 |
| `libaudio_lite_api.so` | `libaudio_lite_api.so` | `/usr/lib/` 或 `/system/lib/` | 被 JS 引擎动态链接 |
| `libmedia_ndk.so` | `libmedia_ndk.so` | `/usr/lib/` 或 `/system/lib/` | NDK 库，供应用链接 |

### 特殊产物

| Target | 输出文件 | 说明 |
|--------|------------|------|
| `test_play_file_h265` | `test_play_file_h265` | H265 解码测试程序 |
| `lite_player_unittest.bin` | 单元测试二进制 | 播放器单元测试 |
| `lite_recorder_unittest.bin` | 单元测试二进制 | 录音器单元测试 |

---

## 构建配置

### 两种运行模式

**Binder 模式** (`enable_media_passthrough_mode = false`):
- 服务独立运行在 media_server 进程
- 客户端通过 IPC (SAMGR) 通信
- 适合多进程架构

**Passthrough 模式** (`enable_media_passthrough_mode = true`):
- 客户端直接链接实现库
- 减少 IPC 开销
- 适合单进程、性能敏感场景

### 内核类型支持

| 内核类型 | 支持的功能 |
|---------|----------|
| **LiteOS-A** | 全部功能（Player、Recorder、JSI） |
| **LiteOS-M** | Player 仅支持 histreamer，Recorder 有限支持，无 JSI（无 JS 引擎） |

---

## 相关跳转

- [项目概览](00_Overview.md)
- [目录结构与模块职责](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [内部 API](04_Inner_API.md)
- [编译产物](06_Build_Artifacts.md)
