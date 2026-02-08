# GN 构建配置与编译产物

## 构建系统概述

ImageEffect 模块使用 OpenHarmony 的 GN（Generate Ninja）构建系统。项目配置通过 `BUILD.gn` 和 `.gni` 文件定义，支持组件化编译和产物分发。

### 构建入口

**根构建文件**：`BUILD.gn`

```gn
group("image_effect") {
  deps = [
    "frameworks/native:image_effect",
    "frameworks/native:image_effect_impl",
  ]
}
```

**证据**：`BUILD.gn:16-21`

### 项目配置

**配置文件**：`config.gni`

```gni
image_effect_root_dir = "//foundation/multimedia/image_effect"
image_effect_colorspace_convertor_enable = false
image_effect_sanitize = {
  integer_overflow = true
  ubsan = true
  boundary_sanitize = true
  cfi = true
  cfi_cross_dso = true
  cfi_vcall_icall_only = true
}
```

**证据**：`config.gni`

## GN Targets 详解

### 框架层 Targets

#### image_effect_impl

**类型**：`ohos_shared_library`

**用途**：内部实现库，提供核心功能。

**源文件**（部分）：
```
effect/base/effect.cpp
effect/base/effect_context.cpp
effect/base/effect_surface_adapter.cpp
effect/base/external_loader.cpp
effect/base/image_effect_inner.cpp
effect/manager/colorspace_manager/*.cpp
effect/manager/memory_manager/*.cpp
effect/pipeline/core/*.cpp
effect/pipeline/factory/*.cpp
effect/pipeline/filters/sink/image_sink_filter.cpp
effect/pipeline/filters/source/image_source_filter.cpp
efilter/base/*.cpp
efilter/custom/*.cpp
efilter/filterimpl/brightness/*.cpp
efilter/filterimpl/contrast/*.cpp
efilter/filterimpl/crop/*.cpp
render_environment/core/*.cpp
render_environment/graphic/*.cpp
render_environment/render_environment.cpp
utils/common/*.cpp
utils/dfx/*.cpp
utils/format/*.cpp
```

**外部依赖**：
| 依赖组件 | 用途 |
|----------|------|
| ability_base:zuri | 基础能力接口 |
| bounds_checking_function:libsec_shared | 边界检查函数 |
| cJSON:cjson | JSON 解析 |
| c_utils:utils | C 工具库 |
| drivers_interface_display:display_commontype_idl_headers | 显示驱动接口 |
| graphic_2d:EGL | 2D 图形 EGL |
| graphic_2d:GLESv3 | OpenGL ES 3.x |
| graphic_2d:color_manager | 颜色管理器 |
| graphic_2d:librender_service_client | 渲染服务客户端 |
| graphic_surface:surface | 图形表面 |
| graphic_surface:sync_fence | 同步栅栏 |
| hilog:libhilog | 日志库 |
| hisysevent:libhisysevent | 性能事件 |
| hitrace:hitrace_meter | 性能追踪 |
| image_framework:image_native | 图像框架 |
| libexif:libexif | EXIF 元数据 |
| napi:ace_napi | N-API 运行时 |
| qos_manager:qos | QoS 管理 |
| skia:skia_canvaskit | Skia 2D 图形 |
| video_processing_engine:videoprocessingengine | 视频处理引擎（条件启用） |

**公共配置**：
```gn
public_configs = [":image_effect_impl_public_config"]

config("image_effect_impl_public_config") {
  defines = [
    "HST_ANY_WITH_NO_RTTI",
    "IMAGE_COLORSPACE_FLAG",
  ]
  include_dirs = [
    "interfaces/inner_api/native/*",
    "frameworks/native/effect/pipeline/include/*",
    "frameworks/native/effect/base",
    "frameworks/native/render_environment/*",
  ]
}
```

**输出产物**：`image_effect_impl.so`

**内 API 标签**：`platformsdk`, `sasdk`

**安全加固**：已启用 sanitize（integer_overflow、ubsan、boundary_sanitize、cfi）

**证据**：`frameworks/native/BUILD.gn`

#### image_effect

**类型**：`ohos_shared_library`

**用途**：NDK 对外接口库，封装 C API。

**源文件**：
```
efilter/custom/filter_delegate.cpp
capi/image_effect.cpp
capi/image_effect_filter.cpp
capi/native_common_utils.cpp
utils/common/any.cpp
```

**依赖**：
```gn
deps = [":image_effect_impl"]
external_deps = [
  "cJSON:cjson",
  "c_utils:utils",
  "graphic_2d:librender_service_client",
  "graphic_surface:surface",
  "hilog:libhilog",
  "image_framework:image_native",
  "image_framework:picture",
  "image_framework:pixelmap",
  "napi:ace_napi",
]
public_configs = [":image_effect_ndk_public_config"]
```

**输出产物**：`image_effect.so`

**内 API 标签**：`ndk`

**证据**：`frameworks/native/BUILD.gn`

### 接口层 Targets

#### libimage_effect（NDK 库）

**类型**：`ohos_ndk_library`

**用途**：NDK 规范库，定义对外接口规范。

**NDK 描述文件**：`.ndk.json`

**最小兼容版本**：`12`

**系统能力**：`SystemCapability.Multimedia.ImageEffect.Core`

**头文件**：
```
multimedia/image_effect/image_effect_errors.h
multimedia/image_effect/image_effect_filter.h
multimedia/image_effect/image_effect.h
```

**证据**：`interfaces/kits/native/BUILD.gn`

#### libimage_effect_header（NDK 头文件）

**类型**：`ohos_ndk_headers`

**用途**：NDK 头文件打包。

**输出目录**：`$ndk_headers_out_dir/multimedia/image_effect`

**证据**：`interfaces/kits/native/BUILD.gn`

### 测试 Targets

#### image_effect_unittest

**类型**：`ohos_unittest`

**用途**：单元测试可执行文件。

**测试源文件**（部分）：
```
test/unittest/TestEffectColorSpaceManager.cpp
test/unittest/TestEffectMemoryManager.cpp
test/unittest/TestEffectPipeline.cpp
test/unittest/TestImageEffect.cpp
test/unittest/TestJsonHelper.cpp
test/unittest/TestPort.cpp
test/unittest/TestRenderEnvironment.cpp
test/unittest/TestUtils.cpp
test/unittest/image_effect_capi_unittest.cpp
test/unittest/image_effect_inner_unittest.cpp
```

**依赖**：
```gn
deps = [
  "frameworks/native:image_effect",
  "frameworks/native:image_effect_impl",
]
external_deps = [
  "ability_base:zuri",
  "cJSON:cjson",
  "c_utils:utils",
  "graphic_2d:EGL",
  "graphic_2d:GLESv3",
  "graphic_surface:surface",
  "hilog:libhilog",
  "image_framework:*",
  "ipc:ipc_single",
  "napi:ace_napi",
  "googletest:gmock_main",
  "googletest:gtest_main",
]
```

**模块输出路径**：`image_effect/image_effect/test`

**证据**：`test/unittest/BUILD.gn`

## 编译产物清单

### 产物列表

| 产物名称 | 类型 | 构建目标 | 用途 | 安装路径 |
|----------|------|----------|------|----------|
| `image_effect.so` | 共享库 | frameworks/native:image_effect | NDK 对外接口 | system/lib64/module/ |
| `image_effect_impl.so` | 共享库 | frameworks/native:image_effect_impl | 内部实现 | system/lib64/ |
| `image_effect_unittest` | 可执行文件 | test/unittest:image_effect_unittest | 单元测试 | system/bin/ |
| `libimage_effect.ndk.json` | NDK 描述 | interfaces/kits/native:libimage_effect | NDK 规范 | prebuilts/ndk/ |

### 运行时加载关系

```
应用进程
    │
    ▼
dlopen("libimage_effect.so")  ──► 依赖 ──► dlopen("libimage_effect_impl.so")
    │                                        │
    │                                        ▼
    │                              dlopen("libimage_effect_ext.so")
    │                              (外部扩展库，动态加载)
    │
    ▼
EXPORT: OH_ImageEffect_Create()
        OH_EffectFilter_*
```

### 动态库依赖

**image_effect.so 依赖**：
```
libace_napi.so          (N-API 运行时)
libhilog.so             (日志)
librender_service_client.so (渲染服务)
libsurface.so            (图形表面)
libimage_native.so       (图像框架)
libcjson.so              (JSON 解析)
```

**image_effect_impl.so 依赖**：
```
libEGL.so                (OpenGL EGL)
libGLESv3.so             (OpenGL ES 3.x)
libskia_canvaskit.so     (Skia 图形库)
libcolor_manager.so       (颜色管理)
libexif.so               (EXIF 处理)
libhisysevent.so         (性能事件)
libimage_effect_ext.so    (外部扩展，可选)
```

## 构建命令

### 完整构建

```bash
./build.sh --product-name {product-name} --build-target foundation/multimedia/image_effect:image_effect
```

**产品名称示例**：rk3568、hi3516dv300

### 仅构建 image_effect_impl

```bash
./build.sh --product-name {product-name} --build-target foundation/multimedia/image_effect:image_effect_impl
```

### 运行测试

```bash
./build.sh --product-name {product-name} --build-target foundation/multimedia/image_effect:image_effect_test
```

### 构建配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `image_effect_colorspace_convertor_enable` | false（默认） | 是否启用色彩空间转换器 |
| `image_effect_sanitize.integer_overflow` | true | 整数溢出检测 |
| `image_effect_sanitize.ubsan` | true | 未定义行为检测 |
| `image_effect_sanitize.boundary_sanitize` | true | 边界检查 |
| `image_effect_sanitize.cfi` | true | 控制流完整性 |

## 产物验证

### 检查 SO 导出符号

```bash
# 查看 image_effect.so 导出符号
nm -D out/{product}/libs/image_effect.so | grep " T "

# 关键导出符号
000000000000XXXX T OH_ImageEffect_Create
000000000000XXXX T OH_ImageEffect_AddFilter
000000000000XXXX T OH_EffectFilter_Create
000000000000XXXX T OH_EffectFilter_SetValue
```

### 检查 SO 依赖

```bash
# 查看 image_effect.so 依赖
ldd out/{product}/libs/image_effect.so

# 查看 image_effect_impl.so 依赖
ldd out/{product}/libs/image_effect_impl.so
```

## 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览 |
| [01_N-API_Reference.md](./01_N-API_Reference.md) | N-API 接口 |
| [02_Architecture.md](./02_Architecture.md) | 内部架构 |
| [04_Security_Review.md](./04_Security_Review.md) | 安全评估 |
