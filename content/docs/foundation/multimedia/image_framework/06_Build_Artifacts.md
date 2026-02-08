# 编译产物

## 目的

本文档描述 Image Framework 构建后的输出文件及其安装路径。

## 产物清单

### 核心库

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| `libimage_native.so` | 共享库 | `/system/lib64/` | 核心图像框架 |
| `libimage_utils.so` | 共享库 | `/system/lib64/` | 图像工具库 |
| `libpluginmanager.so` | 共享库 | `/system/lib64/` | 插件管理器 |
| `libimage_napi.so` | 共享库 | `/system/lib64/module/multimedia/` | JS NAPI 接口 |
| `libimage.so` | 共享库 | `/system/lib64/module/multimedia/` | NAPI 模块 |
| `libsendableimage.so` | 共享库 | `/system/lib64/module/multimedia/` | Sendable 变体 |

### NDK 库

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| `libimage_ndk.so` | 共享库 | `/system/lib64/` | 基础 NDK |
| `libimage_receiver_ndk.so` | 共享库 | `/system/lib64/` | ImageReceiver NDK |
| `libimage_source_ndk.so` | 共享库 | `/system/lib64/` | ImageSource NDK |
| `libimage_packer_ndk.so` | 共享库 | `/system/lib64/` | ImagePacker NDK |
| `libimage_source.so` | 共享库 | `/system/lib64/` | 原生 ImageSource |
| `libimage_packer.so` | 共享库 | `/system/lib64/` | 原生 ImagePacker |
| `libpixelmap.so` | 共享库 | `/system/lib64/` | PixelMap NDK |
| `libpicture.so` | 共享库 | `/system/lib64/` | Picture NDK |
| `libohimage.so` | 共享库 | `/system/lib64/` | 主 NDK 库 |
| `libimage_receiver.so` | 共享库 | `/system/lib64/` | 原生 ImageReceiver |

### 插件库

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| `libjpegplugin.so` | 共享库 | `/system/lib64/` | JPEG 编解码 |
| `libpngplugin.so` | 共享库 | `/system/lib64/` | PNG 编解码 |
| `libgifplugin.so` | 共享库 | `/system/lib64/` | GIF 编解码 |
| `libbmpplugin.so` | 共享库 | `/system/lib64/` | BMP 解码 |
| `libwebpplugin.so` | 共享库 | `/system/lib64/` | WebP 编解码 |
| `librawplugin.so` | 共享库 | `/system/lib64/` | RAW 解码 |
| `libtiffplugin.so` | 共享库 | `/system/lib64/` | TIFF 编解码 |
| `libsvgplugin.so` | 共享库 | `/system/lib64/` | SVG 解码 |
| `libextplugin.so` | 共享库 | `/system/lib64/` | 扩展插件 |
| `libheifparser.so` | 共享库 | `/system/lib64/` | HEIF 解析器 |
| `libheifimpl.so` | 共享库 | `/system/lib64/` | HEIF 实现 |
| `libformatagentplugin.so` | 共享库 | `/system/lib64/` | 格式检测 |
| `libtextureEncoderCL.so` | 共享库 | `/system/lib64/` | OpenCL 纹理编码 |

### 辅助库

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| `libegl_image.so` | 共享库 | `/system/lib64/` | EGL/GPU 图像 |
| `libpixelconvertadapter.so` | 共享库 | `/system/lib64/` | 像素转换 |
| `libimage_accessor.so` | 共享库 | `/system/lib64/` | 元数据访问器 |
| `libimage_error_convert.so` | 共享库 | `/system/lib64/` | 错误转换 |

### 插件元数据

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| `jpegplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | JPEG 插件配置 |
| `pngplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | PNG 插件配置 |
| `gifplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | GIF 插件配置 |
| `bmpplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | BMP 插件配置 |
| `webpplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | WebP 插件配置 |
| `rawplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | RAW 插件配置 |
| `tiffplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | TIFF 插件配置 |
| `svgplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | SVG 插件配置 |
| `extplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | 扩展插件配置 |
| `formatagentplugin.pluginmeta` | 配置文件 | `/system/etc/image/` | 格式检测配置 |

## 运行时加载关系

### 应用启动时

```
应用加载 multimedia.image 模块
    ↓
加载 libimage_napi.so
    ↓
加载 libimage_native.so
    ↓
加载 libpluginmanager.so
    ↓
扫描 /system/lib64/*.pluginmeta
    ↓
按需加载插件库 (*.so)
```

### 解码流程加载

```
调用 createImageSource()
    ↓
加载 libimage_native.so
    ↓
根据文件格式检测
    ↓
加载对应格式插件 (如 libjpegplugin.so)
    ↓
执行解码
```

### 插件加载机制

```cpp
// PluginServer 加载流程
PluginServer::Register() {
    // 1. 扫描配置路径
    // 2. 读取 .pluginmeta 文件
    // 3. dlopen() 加载 .so
    // 4. 注册插件接口
}
```

## 文件路径汇总

### 系统目录

| 目录 | 用途 |
|------|------|
| `/system/lib64/` | 64位共享库 |
| `/system/lib/` | 32位共享库（如有） |
| `/system/lib64/module/multimedia/` | NAPI 模块 |
| `/system/etc/image/` | 插件配置文件 |

### 运行时数据

| 目录 | 用途 |
|------|------|
| `/data/misc/receiver/` | ImageReceiver 临时文件 |
| `/data/misc/creator/` | ImageCreator 临时文件 |

## 依赖关系

### libimage_native.so 依赖

```
libimage_native.so
├── libimage_utils.so
├── libpluginmanager.so
├── libskia.so
├── libcolor_manager.so
├── libsurface.so
├── libjpeg-turbo.so
├── libpng.so
└── libhilog.so
```

### libimage_napi.so 依赖

```
libimage_napi.so
├── libimage_native.so
├── libimage_utils.so
└── libace_napi.z.so
```

### 插件依赖

```
libjpegplugin.so
├── libpluginmanager.so
├── libimage_utils.so
└── libturbojpeg.so

libpngplugin.so
├── libpluginmanager.so
└── libpng.so
```

## 构建输出示例

```
out/ohos-arm64-release/
├── system/
│   ├── lib64/
│   │   ├── libimage_native.so
│   │   ├── libimage_utils.so
│   │   ├── libpluginmanager.so
│   │   ├── libimage_napi.so
│   │   ├── libimage.so
│   │   ├── libsendableimage.so
│   │   ├── libimage_ndk.so
│   │   ├── libpixelmap.so
│   │   ├── libpicture.so
│   │   ├── libjpegplugin.so
│   │   ├── libpngplugin.so
│   │   ├── libgifplugin.so
│   │   ├── libextplugin.so
│   │   └── ...
│   ├── lib64/module/multimedia/
│   │   └── libimage.so -> /system/lib64/libimage.so
│   └── etc/image/
│       ├── jpegplugin.pluginmeta
│       ├── pngplugin.pluginmeta
│       └── ...
└── ...
```

## 相关文档

- [GN 构建目标](05_GN_Targets.md) - 构建系统详情
- [架构说明](02_Architecture.md) - 运行时架构
