# 06_GN_Targets - GN 构建目标

> 本文档详细说明 Cast+ Stream 模块的 GN 构建目标、依赖关系和配置选项。

---

## 1. 构建系统概述

### 1.1 GN 文件分布

```
castengine_cast_plus_stream/
├── BUILD.gn                    # 根构建文件（主目标定义）
├── src/
│   ├── channel/
│   │   └── BUILD.gn            # 通道模块构建
│   ├── mirror/
│   │   └── BUILD.gn            # 镜像模块构建
│   ├── rtsp/
│   │   └── BUILD.gn            # RTSP 模块构建
│   ├── stream/
│   │   └── BUILD.gn            # 流媒体模块构建
│   └── utils/
│       └── BUILD.gn            # 工具模块构建
```

### 1.2 构建目标概览

| 目标名 | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cast_session` | static_library | libcast_session.a | 主目标（聚合所有子模块） |
| `cast_session_channel` | static_library | libcast_session_channel.a | 通道管理 |
| `cast_session_utils` | static_library | libcast_session_utils.a | 工具库 |
| `cast_session_rtsp` | static_library | libcast_session_rtsp.a | RTSP 协议 |
| `cast_session_stream` | static_library | libcast_session_stream.a | 流媒体 |
| `cast_session_mirror` | static_library | libcast_session_mirror.a | 镜像播放 |

---

## 2. 根构建文件

### 2.1 BUILD.gn 完整内容

```gn
# Copyright (c) 2024-2024 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//foundation/CastEngine/castengine_cast_framework/cast_engine.gni")

config("cast_session_config") {
  include_dirs = [
    "include",
    "//third_party/jsoncpp/include",
  ]
}

ohos_static_library("cast_session") {
  sources = [
    "src/cast_session_impl.cpp",
    "src/cast_session_impl_stub.cpp",
    "src/cast_session_listener_impl_proxy.cpp",
    "src/cast_session_listeners.cpp",
    "src/cast_session_state.cpp",
  ]

  configs = [
    ":cast_session_config",
    "${cast_engine_root}:cast_engine_default_config",
  ]

  public_configs = [
    ":cast_session_config",
    "src/channel:cast_session_channel_config",
    "src/utils:cast_session_utils_config",
    "src/rtsp:cast_session_rtsp_config",
    "src/mirror:cast_session_mirror_config",
    "src/stream:cast_session_stream_config",
  ]

  deps = [
    "${cast_engine_common}:cast_engine_common_sources",
    "${cast_engine_service}/src/device_manager:cast_discovery",
    "${cast_engine_service}/src/session/src/utils:cast_session_utils",
    "src/channel:cast_session_channel",
    "src/mirror:cast_session_mirror",
    "src/stream:cast_session_stream",
    "src/rtsp:cast_session_rtsp",
    "src/utils:cast_session_utils",
    "//third_party/openssl:libcrypto_shared",
    "//third_party/jsoncpp:jsoncpp",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "player_framework:media_client",
    "audio_framework:audio_client",
    "device_manager:devicemanagersdk",
    "input:libmmi-client",
    "graphic_surface:surface",
    "ability_runtime:app_manager",
    "image_framework:image_native",
  ]

  subsystem_name = "castplus"
  part_name = "cast_engine"
}
```

### 2.2 关键配置解析

#### Config: cast_session_config

| 属性 | 值 | 说明 |
|------|-----|------|
| `include_dirs` | `include` | 本模块公共头文件目录 |
| `include_dirs` | `//third_party/jsoncpp/include` | JSON 库头文件 |

#### Target: cast_session

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `ohos_static_library` | 静态库 |
| `sources` | 5 个 cpp 文件 | 核心会话实现 |
| `configs` | 2 个 config | 编译配置 |
| `public_configs` | 6 个 config | 暴露给依赖方的头文件配置 |
| `deps` | 10 个 dep | 内部依赖 |
| `external_deps` | 10 个 dep | 外部系统依赖 |

---

## 3. 子模块构建文件

### 3.1 通道模块 (src/channel/BUILD.gn)

```gn
config("cast_session_channel_config") {
  include_dirs = [ "include", "src/softbus" ]
}

ohos_static_library("cast_session_channel") {
  sources = [
    "src/channel_manager.cpp",
    "src/softbus/softbus_connection.cpp",
    "src/softbus/softbus_wrapper.cpp",
    "src/tcp/tcp_connection.cpp",
    "src/tcp/tcp_socket.cpp",
  ]

  include_dirs = [
    "src/softbus",
    "src/tcp",
    # ... SoftBus SDK 路径
  ]

  configs = [ ":cast_session_channel_config", "${cast_engine_root}:cast_engine_default_config" ]
  public_configs = [ ":cast_session_channel_config" ]

  deps = [
    "${cast_engine_common}:cast_engine_common_sources",
    "${cast_engine_service}/src/device_manager:cast_discovery",
  ]

  external_deps = [
    "c_utils:utils",
    "dsoftbus:softbus_client",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "device_manager:devicemanagersdk",
  ]

  subsystem_name = "castplus"
  part_name = "cast_engine"
}
```

**关键依赖**:
- `dsoftbus:softbus_client` - SoftBus 客户端库
- `device_manager:devicemanagersdk` - 设备管理 SDK

### 3.2 RTSP 模块 (src/rtsp/BUILD.gn)

```gn
config("cast_session_rtsp_config") {
  include_dirs = [ "include" ]
}

ohos_static_library("cast_session_rtsp") {
  sources = [
    "src/rtsp_channel_manager.cpp",
    "src/rtsp_controller.cpp",
    "src/rtsp_package.cpp",
    "src/rtsp_param_info.cpp",
    "src/rtsp_parse.cpp",
  ]

  include_dirs = [ "src" ]

  configs = [ ":cast_session_rtsp_config", "${cast_engine_root}:cast_engine_default_config" ]
  public_configs = [ ":cast_session_rtsp_config" ]

  deps = [
    "${cast_engine_common}:cast_engine_common_sources",
    "${cast_engine_service}/src/device_manager:cast_discovery",
    "${cast_engine_service}/src/session/src/channel:cast_session_channel",
    "${cast_engine_service}/src/session/src/utils:cast_session_utils",
    "//third_party/openssl:libcrypto_shared",
  ]

  external_deps = [
    "hilog:libhilog",
    "ipc:ipc_core",
    "device_manager:devicemanagersdk",
  ]

  subsystem_name = "castplus"
  part_name = "cast_engine"
}
```

**关键依赖**:
- `//third_party/openssl:libcrypto_shared` - AES 加密
- `src/channel:cast_session_channel` - 通道传输

### 3.3 流媒体模块 (src/stream/BUILD.gn)

```gn
config("cast_session_stream_config") {
  include_dirs = [
    "include",
    "${cast_engine_service}/src/device_manager/include",
    "${cast_engine_service}/src/session/include",
    "${cast_engine_service}/src/session/src/utils/include",
    "${cast_engine_service}/src/session/src/stream/include",
    "${cast_engine_service}/src/session/src/stream/src/local/include",
    "${cast_engine_service}/src/session/src/stream/src/player/include",
    "//third_party/json/single_include/nlohmann",
  ]
}

ohos_static_library("cast_session_stream") {
  sources = [
    "src/cast_stream_manager_client.cpp",
    "src/cast_stream_manager_server.cpp",
    "src/i_cast_stream_manager.cpp",
    "src/local/src/cast_local_file_channel_client.cpp",
    "src/local/src/cast_local_file_channel_common.cpp",
    "src/local/src/cast_local_file_channel_server.cpp",
    "src/local/src/local_data_source.cpp",
    "src/player/src/cast_stream_player.cpp",
    "src/player/src/cast_stream_player_manager.cpp",
    "src/player/src/remote_player_controller.cpp",
    "src/player/src/stream_player_impl_stub.cpp",
    "src/player/src/stream_player_listener_impl_proxy.cpp",
    "src/player/src/cast_stream_player_utils.cpp",
  ]

  configs = [ ":cast_session_stream_config", "${cast_engine_root}:cast_engine_default_config" ]
  public_configs = [ ":cast_session_stream_config" ]

  deps = [
    "${cast_engine_common}:cast_engine_common_sources",
    "${cast_engine_service}/src/device_manager:cast_discovery",
    "${cast_engine_service}/src/session/src/channel:cast_session_channel",
    "${cast_engine_service}/src/session/src/utils:cast_session_utils",
  ]

  external_deps = [
    "audio_framework:audio_client",
    "c_utils:utils",
    "graphic_surface:surface",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "player_framework:media_client",
    "image_framework:image_native",
  ]

  subsystem_name = "castplus"
  part_name = "cast_engine"
}
```

**关键依赖**:
- `player_framework:media_client` - 系统播放器
- `audio_framework:audio_client` - 系统音频

### 3.4 镜像模块 (src/mirror/BUILD.gn)

```gn
config("cast_session_mirror_config") {
  include_dirs = [
    "include",
    "${cast_engine_service}/src/session/src/mirror/include",
    "${cast_engine_service}/src/session/include",
  ]
}

ohos_static_library("cast_session_mirror") {
  sources = [
    "src/mirror_player_impl.cpp",
    "src/mirror_player_impl_stub.cpp",
  ]

  configs = [ ":cast_session_mirror_config", "${cast_engine_root}:cast_engine_default_config" ]
  public_configs = [ ":cast_session_mirror_config" ]

  deps = [
    "${cast_engine_common}:cast_engine_common_sources",
    "${cast_engine_service}/src/session/src/channel:cast_session_channel",
    "${cast_engine_service}/src/session/src/rtsp:cast_session_rtsp",
    "${cast_engine_service}/src/session/src/utils:cast_session_utils",
    "${cast_engine_service}/src/session/src/stream:cast_session_stream",
  ]

  external_deps = [
    "audio_framework:audio_capturer",
    "audio_framework:audio_client",
    "audio_framework:audio_renderer",
    "av_codec:av_codec_client",
    "c_utils:utils",
    "graphic_2d:librender_service_client",
    "graphic_surface:surface",
    "window_manager:libdm",
    "window_manager:libwm",
    "hitrace:hitrace_meter",
    "hilog:libhilog",
    "ipc:ipc_core",
    "init:libbegetutil",
    "input:libmmi-client",
    "ability_runtime:extension_manager",
    "ability_base:want",
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "ability_runtime:app_manager",
  ]

  subsystem_name = "castplus"
  part_name = "cast_engine"
}
```

**关键依赖**:
- `av_codec:av_codec_client` - 音视频编解码
- `graphic_2d:librender_service_client` - 图形渲染
- `window_manager:libwm` - 窗口管理

### 3.5 工具模块 (src/utils/BUILD.gn)

```gn
config("cast_session_utils_config") {
  include_dirs = [
    "include",
    "//third_party/glib/glib",
    "//third_party/glib",
  ]
}

ohos_static_library("cast_session_utils") {
  sources = [
    "src/encrypt_decrypt.cpp",
    "src/handler.cpp",
    "src/message.cpp",
    "src/permission.cpp",
    "src/state_machine.cpp",
    "src/cast_timer.cpp",
    "src/utils.cpp",
  ]

  configs = [ ":cast_session_utils_config", "${cast_engine_root}:cast_engine_default_config" ]
  public_configs = [ ":cast_session_utils_config" ]

  deps = [
    "${cast_engine_common}:cast_engine_common_sources",
    "//third_party/openssl:libcrypto_shared",
    "//third_party/glib:glib_packages",
  ]

  external_deps = [
    "access_token:libaccesstoken_sdk",
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "wifi:wifi_sdk",
  ]

  subsystem_name = "castplus"
  part_name = "cast_engine"
}
```

**关键依赖**:
- `//third_party/openssl:libcrypto_shared` - 加密
- `access_token:libaccesstoken_sdk` - 权限管理

---

## 4. 依赖关系图

### 4.1 内部依赖

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           内部依赖关系图                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   cast_session (根目标)                                                      │
│   ├── cast_session_channel                                                   │
│   │   └── cast_discovery (device_manager)                                   │
│   ├── cast_session_utils                                                     │
│   │   └── cast_engine_common_sources                                        │
│   ├── cast_session_rtsp                                                      │
│   │   ├── cast_session_channel                                               │
│   │   ├── cast_session_utils                                                 │
│   │   └── cast_discovery                                                     │
│   ├── cast_session_stream                                                    │
│   │   ├── cast_session_channel                                               │
│   │   ├── cast_session_utils                                                 │
│   │   └── cast_discovery                                                     │
│   └── cast_session_mirror                                                    │
│       ├── cast_session_channel                                               │
│       ├── cast_session_rtsp                                                  │
│       ├── cast_session_stream                                                │
│       ├── cast_session_utils                                                 │
│       └── cast_engine_common_sources                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 外部依赖

| 模块 | 外部依赖 | 说明 |
|------|----------|------|
| **全部** | `c_utils:utils` | C++ 工具库 |
| **全部** | `hilog:libhilog` | 日志系统 |
| **全部** | `ipc:ipc_core` | IPC 核心 |
| **channel** | `dsoftbus:softbus_client` | SoftBus 客户端 |
| **channel** | `device_manager:devicemanagersdk` | 设备管理 |
| **utils** | `access_token:libaccesstoken_sdk` | 权限管理 |
| **utils** | `wifi:wifi_sdk` | WiFi 管理 |
| **stream** | `player_framework:media_client` | 媒体播放器 |
| **stream** | `audio_framework:audio_client` | 音频客户端 |
| **mirror** | `av_codec:av_codec_client` | 音视频编解码 |
| **mirror** | `graphic_2d:librender_service_client` | 图形渲染 |
| **mirror** | `window_manager:libwm` | 窗口管理 |

### 4.3 第三方依赖

| 库 | 目标 | 用途 |
|-----|------|------|
| **openssl** | `libcrypto_shared` | AES 加密/解密 |
| **jsoncpp** | `jsoncpp` | JSON 解析 |
| **glib** | `glib_packages` | Base64、工具函数 |

---

## 5. 构建配置

### 5.1 编译选项

```gn
# 默认配置 (来自 cast_engine.gni)
cast_engine_default_config = {
  # C++ 标准
  cflags_cc = [ "-std=c++17" ]
  
  # 警告级别
  cflags = [
    "-Wall",
    "-Wextra",
    "-Werror",
  ]
  
  # 宏定义
  defines = [
    "CAST_ENGINE_LOG_TAG=\"CastEngine\"",
    "OHOS_PLATFORM",
  ]
}
```

### 5.2 包含路径

| 路径 | 说明 |
|------|------|
| `include/` | 公共头文件 |
| `src/*/include/` | 模块头文件 |
| `//third_party/jsoncpp/include` | JSON 库 |
| `//third_party/glib/glib` | GLib 库 |
| `//foundation/communication/dsoftbus/...` | SoftBus SDK |

---

## 6. 构建命令

### 6.1 编译命令

```bash
# 编译整个 CastEngine
cd /path/to/openharmony
hb build cast

# 编译特定目标
hb build cast_session
hb build cast_session_channel
hb build cast_session_rtsp
hb build cast_session_stream
hb build cast_session_mirror
hb build cast_session_utils
```

### 6.2 输出目录

```
out/{product}/
├── obj/
│   └── foundation/CastEngine/castengine_cast_plus_stream/
│       ├── libcast_session.a
│       ├── src/
│       │   ├── channel/libcast_session_channel.a
│       │   ├── mirror/libcast_session_mirror.a
│       │   ├── rtsp/libcast_session_rtsp.a
│       │   ├── stream/libcast_session_stream.a
│       │   └── utils/libcast_session_utils.a
│       └── ... (object files)
```

---

## 7. 相关文档

- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [05_Internal_API.md](./05_Internal_API.md) - 内部 API
- [07_Build_Artifacts.md](./07_Build_Artifacts.md) - 编译产物
