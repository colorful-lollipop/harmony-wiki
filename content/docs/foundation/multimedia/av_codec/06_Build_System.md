# 06_GN 构建系统

## 构建配置

### 根配置文件

| 文件 | 用途 | 证据 |
|------|------|------|
| `BUILD.gn` | 根构建入口 | `BUILD.gn:17-22` |
| `config.gni` | GN 编译选项和开关 | `config.gni:16-62` |
| `bundle.json` | 部件描述和依赖声明 | `bundle.json:1-330` |

### 根 BUILD.gn

**证据**: `BUILD.gn:14-22`
```gn
import("//build/config/ohos/config.gni")
import("//build/ohos.gni")

group("av_codec_packages") {
  public_deps = [
    "interfaces/inner_api/native:av_codec_client",
    "services/services:av_codec_service",
  ]
}
```

## 编译开关

### 功能开关 (config.gni)

| 开关 | 默认值 | 用途 |
|------|--------|------|
| `av_codec_support_capi` | `true` | C-API 支持 |
| `av_codec_support_codec` | `true` | 编解码支持 |
| `av_codec_support_codeclist` | `true` | 能力列表 |
| `av_codec_support_hcodec` | `true` | 硬件编解码 |
| `av_codec_support_demuxer` | `true` | 解封装 |
| `av_codec_support_source` | `true` | 媒体源 |
| `av_codec_support_muxer` | `true` | 封装 |
| `av_codec_support_drm` | `false` | DRM 支持 |
| `av_codec_support_fcodec` | `true` | 软件编解码 |
| `av_codec_support_avc_encoder` | `true` | AVC 编码器 |
| `av_codec_support_hevc_decoder` | `true` | HEVC 解码器 |
| `av_codec_support_av1_decoder` | `false` | AV1 解码器 |

**证据**: `config.gni:16-62`

### 安全编译选项

```gn
// config.gni:63-70
av_codec_sanitize = {
  boundary_sanitize = true      # 边界检查
  cfi = true                    # 控制流完整性
  cfi_cross_dso = true          # 跨 DSO CFI
  integer_overflow = true       # 整数溢出检测
  ubsan = true                  # 未定义行为检测
  debug = false
}
```

## 关键 Targets

### 对外 C-API Targets

| Target | 类型 | 源文件 | 输出 | 证据 |
|--------|------|--------|------|------|
| `native_media_avdemuxer` | shared_library | `native_avdemuxer.cpp` | `libnative_media_avdemuxer.so` | `interfaces/kits/c/BUILD.gn:133-165` |
| `native_media_avsource` | shared_library | `native_avsource.cpp` | `libnative_media_avsource.so` | `interfaces/kits/c/BUILD.gn:167-196` |
| `native_media_codecbase` | shared_library | `native_avcodec_base.cpp` | `libnative_media_codecbase.so` | `interfaces/kits/c/BUILD.gn:198-226` |
| `native_media_acodec` | shared_library | `native_audio_codec.cpp` | `libnative_media_acodec.so` | `interfaces/kits/c/BUILD.gn:228-264` |
| `native_media_adec` | shared_library | `native_audio_decoder.cpp` | `libnative_media_adec.so` | `interfaces/kits/c/BUILD.gn:266-291` |
| `native_media_aenc` | shared_library | `native_audio_encoder.cpp` | `libnative_media_aenc.so` | `interfaces/kits/c/BUILD.gn:293-318` |
| `native_media_vdec` | shared_library | `native_video_decoder.cpp` | `libnative_media_vdec.so` | `interfaces/kits/c/BUILD.gn:320-352` |
| `native_media_venc` | shared_library | `native_video_encoder.cpp` | `libnative_media_venc.so` | `interfaces/kits/c/BUILD.gn:354-379` |
| `native_media_avcencinfo` | shared_library | `native_cencinfo.cpp` | `libnative_media_avcencinfo.so` | `interfaces/kits/c/BUILD.gn:381-409` |

### Inner API Targets

| Target | 类型 | 头文件 | 证据 |
|--------|------|--------|------|
| `av_codec_client` | shared_library | `avcodec_audio_decoder.h` 等 | `bundle.json:134-158` |
| `av_codec_suspend_client` | shared_library | `avcodec_suspend.h` 等 | `bundle.json:123-130` |

### 服务 Targets

| Target | 类型 | 描述 | 证据 |
|--------|------|------|------|
| `av_codec_service` | executable | SA 服务主程序 | `services/BUILD.gn:17-49` |
| `av_codec_service_utils` | static_library | 服务工具库 | `services/utils/BUILD.gn` |
| `av_codec_service_dfx` | static_library | DFX 功能 | `services/dfx/BUILD.gn` |

### 引擎 Targets

| Target | 类型 | 条件 | 证据 |
|--------|------|------|------|
| `fcodec` | shared_library | `av_codec_support_fcodec` | `services/BUILD.gn:23-24` |
| `avc_encoder` | shared_library | `av_codec_support_avc_encoder` | `services/BUILD.gn:26-27` |
| `av1_decoder` | shared_library | `av_codec_support_av1_decoder` | `services/BUILD.gn:29-30` |
| `hevc_decoder` | shared_library | `av_codec_support_hevc_decoder` | `services/BUILD.gn:32-33` |
| `hcodec_group` | shared_library | `av_codec_support_hcodec` | `services/BUILD.gn:35-36` |
| `vpx_decoder` | shared_library | `av_codec_support_vp8_decoder \|\| av_codec_support_vp9_decoder` | `services/BUILD.gn:38-39` |

## 编译产物

### 产物清单

| 产物 | 类型 | 安装路径 | 用途 |
|------|------|----------|------|
| `libnative_media_*.so` | 动态库 | `/system/lib/` | 对外 C-API |
| `av_codec_service` | 可执行文件 | `/system/bin/` | SA 服务 |
| `*.a` | 静态库 | 构建中间目录 | 内部链接 |

### 运行时加载关系

```
应用进程
  │
  ├── dlopen("libnative_media_vdec.so")    // 视频解码
  ├── dlopen("libnative_media_venc.so")    // 视频编码
  ├── dlopen("libnative_media_avdemuxer.so") // 解封装
  ├── dlopen("libnative_media_avmuxer.so")   // 封装
  └── dlopen("libnative_media_avsource.so")  // 媒体源
         │
         └── IPC 调用 ──> av_codec_service (SA 3011)
```

## 编译命令

### 编译整个部件

```bash
# 32位 ARM
./build.sh --product-name {product_name} --ccache --build-target av_codec

# 64位 ARM
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target av_codec
```

### 单独编译某个模块

```bash
# 编译 C-API
./build.sh --product-name {product_name} --build-target av_codec_capi_packages

# 编译服务
./build.sh --product-name {product_name} --build-target av_codec_services_package
```

## 依赖关系

### 外部依赖

| 依赖组件 | 用途 | 证据 |
|---------|------|------|
| `ipc` | IPC 通信 | `bundle.json:82` |
| `hilog` | 日志 | `bundle.json:78` |
| `safwk` | SA 框架 | `bundle.json:84` |
| `samgr` | 服务管理 | `bundle.json:85` |
| `media_foundation` | 媒体基础 | `bundle.json:87` |
| `audio_framework` | 音频框架 | `bundle.json:88` |
| `drm_framework` | DRM 解密 | `bundle.json:89` |
| `graphic_surface` | Surface | `bundle.json:73` |

**证据**: `bundle.json:64-104`

### 内部依赖

```
C-API 库 ──> Inner API (av_codec_client) ──> 服务 (av_codec_service)
   │
   └──> DFX (av_codec_service_dfx)
```

---

**相关文档**: [对外 C-API](04_C_API.md) | [安全风险评审](07_Security_Review.md) | [故障排查指南](08_Troubleshooting.md)
