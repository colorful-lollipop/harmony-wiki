# 目录结构

## 目的

本文档说明 Image Framework 的目录组织结构，帮助开发者快速定位代码。

## 结构总览

```
foundation/multimedia/image_framework/
├── BUILD.gn                    # 根构建文件
├── bundle.json                 # 组件元数据
├── README.md                   # 项目说明
├── hisysevent.yaml             # 系统事件配置
│
├── frameworks/                 # 框架实现
│   ├── innerkitsimpl/         # Native API 实现
│   │   ├── accessor/          # 元数据访问器
│   │   ├── common/            # 通用实现 (PixelMap, ImageSource, ImagePacker)
│   │   ├── egl_image/         # EGL/GPU 图像处理
│   │   ├── pixelconverter/    # 像素格式转换
│   │   ├── receiver/          # ImageReceiver 实现
│   │   ├── stream/            # 数据源流实现
│   │   └── utils/             # 工具类
│   │
│   ├── kits/                  # 多语言绑定
│   │   ├── ani/               # ANI (ArkUI Native Interface)
│   │   ├── cj/                # Cangjie FFI
│   │   ├── js/common/         # JS/N-API 绑定
│   │   │   ├── ndk/           # NDK 实现
│   │   │   ├── pixelmap_ndk/  # PixelMap NDK
│   │   │   ├── picture_ndk/   # Picture NDK
│   │   │   └── sendable/      # Sendable 变体
│   │   ├── native/common/     # Native NDK
│   │   └── taihe/             # Taihe 绑定
│   │
│   └── jni/                   # JNI (Android 兼容)
│
├── interfaces/                 # API 接口
│   ├── innerkits/             # C++ 内部 API
│   │   └── include/           # 头文件
│   │       ├── image_source.h
│   │       ├── image_packer.h
│   │       ├── pixel_map.h
│   │       ├── picture.h
│   │       ├── image_type.h
│   │       └── ...
│   │
│   └── kits/                  # 公开 API
│       ├── js/common/         # JS N-API 头文件
│       │   └── include/
│       │       ├── image_source_napi.h
│       │       ├── pixel_map_napi.h
│       │       └── ...
│       └── native/            # Native NDK 头文件
│           └── include/
│
├── plugins/                    # 图像编解码插件
│   ├── common/libs/           # 插件库
│   │   ├── image/
│   │   │   ├── libbmpplugin/      # BMP 插件
│   │   │   ├── libextplugin/      # 扩展插件 (HEIF/HDR/ASTC)
│   │   │   ├── libgifplugin/      # GIF 插件
│   │   │   ├── libjpegplugin/     # JPEG 插件
│   │   │   ├── libpngplugin/      # PNG 插件
│   │   │   ├── librawplugin/      # RAW 插件
│   │   │   ├── libtiffplugin/     # TIFF 插件
│   │   │   ├── libwebpplugin/     # WebP 插件
│   │   │   ├── libsvgplugin/      # SVG 插件
│   │   │   └── formatagentplugin/ # 格式检测代理
│   │   └── BUILD.gn
│   │
│   ├── cross/                 # 跨平台配置
│   │   ├── image_native_ios.gni
│   │   └── image_native_android.gni
│   │
│   └── manager/               # 插件管理器
│       ├── include/           # 头文件
│       ├── src/               # 实现
│       └── BUILD.gn
│
├── mock/                       # Mock 实现
├── ide/                        # IDE 配置
└── test/                       # 测试代码 (忽略)
```

## 模块职责

### frameworks/innerkitsimpl/

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `common/` | PixelMap、ImageSource、ImagePacker 实现 | `src/pixel_map.cpp`, `src/image_source.cpp` |
| `accessor/` | 元数据访问器 (EXIF/ICC/DNG) | `include/metadata_accessor.h` |
| `egl_image/` | EGL/GPU 加速处理 | `include/pixel_map_from_surface.h` |
| `pixelconverter/` | 像素格式转换 | `include/pixel_convert_adapter.h` |
| `receiver/` | ImageReceiver 实现 | `include/image_receiver.h` |
| `stream/` | 数据源抽象 | `include/source_stream.h` |
| `utils/` | 工具类 | `include/image_utils.h`, `include/pixel_yuv_utils.h` |

### frameworks/kits/js/common/

N-API 绑定实现：

| 文件 | 职责 |
|------|------|
| `native_module_ohos_image.cpp` | 主模块注册 (`multimedia.image`) |
| `image_source_napi.cpp` | ImageSource JS 绑定 |
| `image_packer_napi.cpp` | ImagePacker JS 绑定 |
| `pixel_map_napi.cpp` | PixelMap JS 绑定 |
| `image_receiver_napi.cpp` | ImageReceiver JS 绑定 |
| `picture_napi.cpp` | Picture JS 绑定 |
| `metadata_napi.cpp` | Metadata JS 绑定 |
| `image_napi_utils.cpp` | 工具函数与权限检查 |

### interfaces/innerkits/include/

C++ 核心 API 头文件：

| 头文件 | 说明 |
|--------|------|
| `image_source.h` | ImageSource 类定义，解码入口 |
| `image_packer.h` | ImagePacker 类定义，编码入口 |
| `pixel_map.h` | PixelMap 类定义，像素数据容器 |
| `picture.h` | Picture 类定义，HDR 容器 |
| `image_type.h` | 枚举类型与结构体定义 |
| `auxiliary_picture.h` | AuxiliaryPicture 定义 |
| `metadata.h` | 元数据基类定义 |

### plugins/

#### 插件管理器 (plugins/manager/)

| 组件 | 文件 | 职责 |
|------|------|------|
| PluginServer | `include/plugin_server.h` | 插件系统单例 |
| AbsImageDecoder | `include/image/abs_image_decoder.h` | 解码器插件接口 |
| AbsImageEncoder | `include/image/abs_image_encoder.h` | 编码器插件接口 |
| AbsImageFormatAgent | `include/pluginbase/abs_image_format_agent.h` | 格式检测接口 |

#### 编解码插件 (plugins/common/libs/image/)

| 插件 | 说明 | 格式 |
|------|------|------|
| `libjpegplugin` | JPEG 编解码 | JPEG, JPG |
| `libpngplugin` | PNG 编解码 | PNG |
| `libgifplugin` | GIF 编解码 | GIF |
| `libbmpplugin` | BMP 解码 | BMP, WBMP |
| `libtiffplugin` | TIFF 编解码 | TIFF |
| `librawplugin` | RAW 解码 | DNG, CR3, RAW |
| `libwebpplugin` | WebP 编解码 | WebP |
| `libsvgplugin` | SVG 解码 | SVG |
| `libextplugin` | 扩展插件 | HEIF/HEIC, ASTC, HDR |
| `formatagentplugin` | 格式检测 | 所有格式 |

## 关键文件索引

### 入口文件

| 类型 | 路径 |
|------|------|
| 主模块注册 | `frameworks/kits/js/common/native_module_ohos_image.cpp` |
| 根构建文件 | `BUILD.gn` |
| 组件配置 | `bundle.json` |
| 构建配置 | `ide/image_decode_config.gni` |

### 核心类实现

| 类 | 头文件 | 实现文件 |
|----|--------|----------|
| ImageSource | `interfaces/innerkits/include/image_source.h` | `frameworks/innerkitsimpl/common/src/image_source.cpp` |
| ImagePacker | `interfaces/innerkits/include/image_packer.h` | `frameworks/innerkitsimpl/common/src/image_packer.cpp` |
| PixelMap | `interfaces/innerkits/include/pixel_map.h` | `frameworks/innerkitsimpl/common/src/pixel_map.cpp` |
| Picture | `interfaces/innerkits/include/picture.h` | `frameworks/innerkitsimpl/common/src/picture.cpp` |
| PluginServer | `plugins/manager/include/plugin_server.h` | `plugins/manager/src/plugin_server.cpp` |

## 相关文档

- [架构说明](02_Architecture.md) - 组件关系与数据流
- [GN 构建目标](05_GN_Targets.md) - 构建系统详情
