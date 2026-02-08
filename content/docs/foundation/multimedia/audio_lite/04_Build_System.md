# 04_Build_System - 构建系统

本文档描述 audio_lite 的 GN 构建配置、编译产物和依赖关系。

## 4.1 构建系统概述

### 4.1.1 构建工具

| 工具 | 版本要求 | 说明 |
|------|----------|------|
| GN | 最新版本 | 生成 Ninja 构建文件 |
| Ninja | 最新版本 | 执行构建 |
| hb (OpenHarmony Build) | 配套版本 | 构建入口工具 |

### 4.1.2 构建命令

```bash
# 1. 设置开发板
hb set

# 2. 构建 audio_lite
hb build audio_lite

# 3. 清理构建
hb build audio_lite --clean
```

## 4.2 构建配置文件

### 4.2.1 配置文件清单

| 文件 | 类型 | 说明 |
|------|------|------|
| `frameworks/BUILD.gn` | BUILD.gn | 框架层构建配置 |
| `services/BUILD.gn` | BUILD.gn | 服务层构建配置 |
| `bundle.json` | bundle.json | 组件元数据配置 |
| `config.gni` | .gni | 编译开关配置 |

### 4.2.2 引用的外部配置

| 配置文件 | 来源 | 用途 |
|----------|------|------|
| `//build/lite/config/component/lite_component.gni` | 构建系统 | 组件配置模板 |
| `//build/lite/ndk/ndk.gni` | 构建系统 | NDK 配置 |
| `//foundation/multimedia/media_utils_lite/config.gni` | 子系统 | 编译开关定义 |

## 4.3 Target 清单

### 4.3.1 frameworks/BUILD.gn

**文件位置**：`frameworks/BUILD.gn`

| Target | 类型 | Sources | 输出产物 | 说明 |
|--------|------|---------|----------|------|
| `audio_capturer_lite` | `shared_library` | 见下方 | `libaudio_capturer_lite.so` | **主对外动态库** |
| `audio_capturer_public_config` | `config` | - | - | 公共头文件配置 |

#### audio_capturer_lite 详细配置

**证据**：`frameworks/BUILD.gn:17-56`

```gn
shared_library("audio_capturer_lite") {
  sources = [ "audio_capturer.cpp" ]

  if (enable_media_passthrough_mode == true) {
    # Passthrough 模式
    sources += [ "passthrough/audio_capturer_client.cpp" ]
    include_dirs = [
      "//foundation/multimedia/audio_lite/frameworks/passthrough",
      "//foundation/multimedia/audio_lite/services/impl",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    ]
    deps = [
      "//foundation/multimedia/audio_lite/services:audio_capturer_impl",
      "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
    ]
  } else {
    # Binder IPC 模式
    sources += [ "binder/audio_capturer_client.cpp" ]
    include_dirs = [
      "//base/security/permission_lite/interfaces/kits",
     /audio_lite/services "//foundation/multimedia/server/include",
      "//foundation/multimedia/audio_lite/services/impl",
      "//foundation/multimedia/audio_lite/frameworks/binder",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    ]
    deps = [
      "//base/security/permission_lite/services/pms_client:pms_client",
      "//foundation/graphic/surface_lite:surface_lite",
      "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
    ]
  }
  public_configs = [ ":audio_capturer_public_config" ]
  public_deps = [
    "//foundation/multimedia/media_utils_lite:media_common",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}
```

**Sources 清单**：
| 文件 | 模式 | 说明 |
|------|------|------|
| `audio_capturer.cpp` | 通用 | AudioCapturer 主类实现 |
| `binder/audio_capturer_client.cpp` | IPC | Binder 模式客户端 |
| `passthrough/audio_capturer_client.cpp` | Passthrough | 直通模式客户端 |

**Public Deps**：
| 依赖项 | 用途 |
|--------|------|
| `//foundation/multimedia/media_utils_lite:media_common` | 公共媒体类型 |
| `//third_party/bounds_checking_function:libsec_shared` | 安全字符串函数 |

---

### 4.3.2 services/BUILD.gn

**文件位置**：`services/BUILD.gn`

| Target | 类型 | Sources | 输出产物 | 说明 |
|--------|------|---------|----------|------|
| `audio_capturer_server` | `static_library` | 见下方 | `libaudio_capturer_server.a` | **服务端静态库** |
| `audio_capturer_impl` | `shared_library` | 见下方 | `libaudio_capturer_impl.so` | **实现动态库** |
| `audio_external_library_config` | `config` | - | - | 外部库配置 |

#### audio_capturer_server 详细配置

**证据**：`services/BUILD.gn:16-46`

```gn
static_library("audio_capturer_server") {
  sources = [
    "server/src/audio_capturer_samgr.cpp",
    "server/src/audio_capturer_server.cpp",
  ]
  cflags = [ "-fPIC" ]
  cflags += [ "-Werror" ]
  cflags_cc = cflags
  include_dirs = [
    "//base/security/permission_lite/interfaces/kits",
    "//foundation/multimedia/audio_lite/services/server/include",
    "//foundation/multimedia/audio_lite/interfaces/kits",
    "//foundation/multimedia/audio_lite/services/impl",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc/include",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast",
    "//foundation/multimedia/media_utils_lite/interfaces/kits",
    "//third_party/bounds_checking_function/include",
  ]

  deps = [
    "//base/security/permission_lite/services/pms_client:pms_client",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "//foundation/graphic/surface_lite:surface_lite",
    "//foundation/multimedia/audio_lite/services:audio_capturer_impl",
    "//foundation/multimedia/media_utils_lite:media_common",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}
```

#### audio_capturer_impl 详细配置

**证据**：`services/BUILD.gn:48-83`

```gn
shared_library("audio_capturer_impl") {
  sources = [
    "impl/audio_capturer_impl.cpp",
    "impl/audio_encoder/audio_encoder.cpp",
    "impl/audio_source/audio_source.cpp",
  ]
  cflags = [ "-fPIC" ]
  cflags += [ "-Werror" ]
  cflags_cc = cflags
  include_dirs = [
    "//foundation/multimedia/audio_lite/services/impl/audio_encoder/include",
    "//foundation/multimedia/audio_lite/services/impl/audio_source/include",
    "//foundation/multimedia/audio_lite/frameworks/binder",
    "//foundation/multimedia/audio_lite/interfaces/kits",
    "//drivers/peripheral/audio/interfaces/include",
    "//drivers/peripheral/codec/interfaces/include",
    "//drivers/peripheral/display/interfaces/include",
    "//drivers/peripheral/base",
    "//foundation/multimedia/media_utils_lite/interfaces/kits",
  ]

  outdir = rebase_path("$root_out_dir")
  public_configs = [ ":audio_external_library_config" ]
  ldflags = [
    "-L$outdir",
    "-lcodec",
    "-laudio_hw",
    "-lpthread",
  ]
  deps = [
    "//device/soc/hisilicon/common/hal/media:hardware_media_sdk",
    "//foundation/graphic/surface_lite:surface_lite",
    "//foundation/multimedia/media_utils_lite:media_common",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}
```

**Sources 清单**：
| 文件 | 说明 |
|------|------|
| `impl/audio_capturer_impl.cpp` | 核心实现 |
| `impl/audio_encoder/audio_encoder.cpp` | 音频编码器 |
| `impl/audio_source/audio_source.cpp` | 音频源 |

---

## 4.4 编译开关

### 4.4.1 全局开关

| 开关 | 定义位置 | 默认值 | 影响 |
|------|----------|--------|------|
| `enable_media_passthrough_mode` | `media_utils_lite/config.gni` | `false` | 通信模式切换 |
| `ohos_build_type` | 全局 | `release` | 构建类型 |

### 4.4.2 模式差异

| 模式 | 开关值 | 通信方式 | 适用场景 |
|------|--------|----------|----------|
| Binder IPC | `false` | Samgr IPC + Surface | 多进程、安全隔离 |
| Passthrough | `true` | 直接函数调用 | 单进程、性能优先 |

## 4.5 编译产物

### 4.5.1 产物清单

| 产物文件 | Target | 路径 | 类型 | 说明 |
|---------|--------|------|------|------|
| `libaudio_capturer_lite.so` | `audio_capturer_lite` | `out/.../frameworks` | 动态库 | **主对外 API 库** |
| `libaudio_capturer_impl.so` | `audio_capturer_impl` | `out/.../services` | 动态库 | 实现层动态库 |
| `libaudio_capturer_server.a` | `audio_capturer_server` | `out/.../services` | 静态库 | 服务端静态库 |

### 4.5.2 安装路径

| 产物 | 预计安装路径 |
|------|--------------|
| `libaudio_capturer_lite.so` | `/system/lib/` 或 `/system/lib64/` |
| `libaudio_capturer_impl.so` | `/system/lib/` 或 `/system/lib64/` |

### 4.5.3 运行时加载关系

```
┌─────────────────────────────────────────────────────────────┐
│                     应用进程                                 │
│                                                             │
│  dlopen("libaudio_capturer_lite.so")                       │
│         │                                                    │
│         ▼                                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  AudioCapturer                                      │   │
│  │    - Binder 模式: IClientProxy (IPC)                │   │
│  │    - Passthrough 模式: 直接调用 AudioCapturerImpl   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                          │
                          │ IPC / 直接调用
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     服务进程                                 │
│                                                             │
│  ┌──────────────────────┐  ┌────────────────────────────┐ │
│  │ libaudio_capturer_   │  │ libaudio_capturer_impl.so  │ │
│  │ server.a (静态链接)   │  │                            │ │
│  └──────────────────────┘  └────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## 4.6 依赖关系图

### 4.6.1 内部依赖

```
frameworks/BUILD.gn
│
├── audio_capturer_lite ────────┬──────────────────────────────┐
│       │                       │                              │
│       │ (enable_media_        │ (enable_media_               │
│       │  passthrough_mode?)    │  passthrough_mode?)         │
│       │                       │                              │
│       ▼                       ▼                              ▼
│  passthrough/            binder/                      services/BUILD.gn
│  audio_capturer_         audio_capturer_                    │
│  client.cpp              client.cpp                          │
│                            │                                  │
│                            ▼                                  │
│                   ┌────────────────────┐                    │
│                   │ samgr (IPC 框架)   │                    │
│                   │ pms_client (权限)  │                    │
│                   │ surface_lite        │                    │
│                   └────────────────────┘                    │
│                                                           │
│                                                           ▼
│                                           audio_capturer_server (静态库)
│                                           audio_capturer_impl (动态库)
│                                           │
│                                           └──► hardware_media_sdk (HAL)
```

### 4.6.2 外部依赖

| 依赖项 | 类型 | 用途 |
|--------|------|------|
| `samgr` | 系统组件 | 服务管理、IPC |
| `surface_lite` | 系统组件 | 共享内存 |
| `pms_client` | 安全组件 | 权限管理 |
| `ipc_single` | 通信组件 | IPC |
| `media_utils_lite:media_common` | 子系统库 | 公共类型 |
| `bounds_checking_function` | 第三方 | 安全函数 |
| `hardware_media_sdk` | HAL | 硬件接口 |
| `libaudio_hw` | HAL | 音频硬件 |
| `libcodec` | HAL | 编解码器 |

## 4.7 bundle.json 配置

**证据**：`bundle.json:29-35`

```json
{
    "build": {
        "sub_component": [
            "//foundation/multimedia/audio_lite/frameworks:audio_capturer_lite"
        ],
        "inner_kits": [],
        "test": []
    }
}
```

**关键配置**：
- 子系统入口：`//foundation/multimedia/audio_lite/frameworks:audio_capturer_lite`
- 无内部 kits（不对其他子系统暴露内部接口）
- 无测试套件引用

---

**上一章**：[03_API_Reference](03_API_Reference.md) | **下一章**：[05_Security_Review](05_Security_Review.md)
