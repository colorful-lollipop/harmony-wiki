# GN 构建配置与编译产物

## 根构建配置

### BUILD.gn (`/`)

| Target | 类型 | 描述 | 产物 |
|--------|------|------|------|
| `default` | group | 主构建组 | bootanimation (可选) |
| `graphic_common_test` | group | 测试聚合 | test/*.so |
| `graphic.rc` | ohos_prebuilt_etc | 初始化配置 | `/system/etc/init/` |
| `libvulkan` | group | Vulkan 包装 | - |
| `libnative_image` | group | Native Image | libnative_image.so |
| `libbootanimation_utils` | group | 开机动画工具 | - |

**证据来源**: `BUILD.gn`

---

## Feature Flags

### graphic_config.gni 关键配置

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `graphic_2d_feature_enable_vulkan` | false | 启用 Vulkan 后端 |
| `graphic_2d_feature_enable_opengl` | true | 启用 OpenGL 后端 |
| `graphic_2d_feature_rs_enable_uni_render` | false | 统一渲染模式 |
| `graphic_2d_feature_enable_dvsync` | false | 动态 VSync |
| `graphic_2d_feature_bootanimation_enable` | true | 开机动画 |
| `rs_enable_gpu` | true | GPU 加速 |
| `graphic_2d_feature_pgo` | false | PGO 优化 |
| `graphic_2d_feature_enable_codemerge` | false | 代码合并 |
| `graphic_2d_feature_upgrade_skia` | true | 升级 Skia |
| `is_cross_platform` | false | 跨平台支持 |

**产品相关开关** (phone/pc/tablet/wearable):

```gn
if (product == "phone" || "pc" || "tablet" || "wearable") {
  graphic_2d_feature_enable_vulkan = true
  graphic_2d_feature_enable_stack_culling = true
}
```

**证据来源**: `graphic_config.gni:15-100`

---

## 核心模块构建 Targets

### Render Service 模块

| Target | 类型 | 产物 | 关键 Sources | 关键依赖 |
|--------|------|------|--------------|----------|
| **librender_service** | ohos_shared_library | `.so` | MainThread, RenderThread, HWC | base, ipc, ffrt, skia:skia_canvaskit |
| **librender_service_base** | ohos_shared_library | `.so` | Animation, Modifier, Drawable, Transaction | platform, color_manager, skia |
| **librender_service_client** | ohos_shared_library | `.so` | UI Nodes, Animation Client, Modifier NG | base, 2d_graphics, ace_napi |

**证据来源**: `rosen/modules/render_service/BUILD.gn`

---

### 2D 图形模块

| Target | 类型 | 产物 | 关键 Sources | 依赖 |
|--------|------|------|--------------|------|
| **2d_graphics** | ohos_shared_library | `2d_graphics.so` | Canvas, Paint, Path, Brush, Pen | skia, icu, hilog |
| **native_drawing_ndk** | ohos_shared_library | `libnative_drawing_ndk.so` | Drawing NDK 函数 (40+) | 2d_graphics |

**产物符号链接**: `lib2d_graphics.so` → `2d_graphics.so`

**证据来源**: `rosen/modules/2d_graphics/BUILD.gn`

---

### 显示合成模块

| Target | 类型 | 产物 | 关键 Sources | 依赖 |
|--------|------|------|--------------|------|
| **libcomposer** | ohos_shared_library | `libcomposer.so` | HDI Backend, Screen | display HDI (1.0-1.4) |
| **libvsync** | ohos_shared_library | `libvsync.so` | VSync Generator/Distributor | - |

**证据来源**: `rosen/modules/composer/BUILD.gn`

---

### 效果模块

| Target | 类型 | 产物 | 依赖 |
|--------|------|------|------|
| **color_picker** | shared_library | `libcolor_picker.so` | effect_common |
| **skeffectchain** | shared_library | `libskeffectchain.so` | skia |
| **effect_common** | shared_library | `libeffect_common.so` | skia |
| **native_effect_ndk** | shared_library | `libnative_effect_ndk.so` | effect_common |

**符号链接**: `libnative_effect.so` → `libnative_effect_ndk.so`

**证据来源**: `rosen/modules/effect/*/BUILD.gn`

---

### OpenGL 封装库

| Target | 类型 | 产物 | API 级别 |
|--------|------|------|----------|
| **EGL** | shared_library | `libEGL.so` | EGL 1.4 |
| **GLESv1** | shared_library | `libGLESv1.so` | OpenGL ES 1.1 |
| **GLESv2** | shared_library | `libGLESv2.so` | OpenGL ES 2.0 |
| **GLESv3** | shared_library | `libGLESv3.so` | OpenGL ES 3.x |
| **GLv4** | shared_library | `libGLv4.so` | OpenGL 4.x (可选) |

**标签**: `llndk` (Low-level NDK)

**证据来源**: `frameworks/opengl_wrapper/BUILD.gn`

---

### 工具库

| Target | 类型 | 产物 | 依赖 |
|--------|------|------|------|
| **color_manager** | shared_library | `libcolor_manager.so` | - |
| **libgraphic_utils** | shared_library | `libgraphic_utils.so` | - |
| **socketpair** | source_set | - | - |
| **scoped_bytrace** | source_set | - | - |

**证据来源**: `utils/BUILD.gn`

---

## N-API 构建 Targets

### 包组

| Target | 描述 | 包含模块 |
|--------|------|----------|
| **napi_packages** | N-API 包组 | drawingnapi, effectkit, libhgmnapi 等 |
| **ffi_packages** | Cangjie FFI 包 | cj_color_manager, cj_effect_kit |
| **ani_packages** | ArkNative 包 | ani_color_space, ani_drawing |

### 独立 N-API 模块

| Target | 产物 | 依赖 |
|--------|------|------|
| **drawingnapi** | `libdrawing_napi.so` | 2d_graphics, ace_napi |
| **effectkit** | `libeffectkit.so` | effect_common, color_picker |
| **libhgmnapi** | `libhgmnapi.so` | hyper_graphic_manager |
| **textnapi** | `libtextnapi.so` | text |
| **windowanimationmanager_napi** | `libwindowanimationmanager_napi.so` | animation |
| **colorspacemanager_napi** | `libcolorspacemanager_napi.so` | color_manager |
| **libwebglnapi** | `libwebglnapi.so` | opengl_wrapper |

**证据来源**: `interfaces/kits/napi/BUILD.gn`

---

## 产物安装路径

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `librender_service.so` | `/system/lib64/` | 渲染服务库 |
| `librender_service_base.so` | `/system/lib64/` | 基础库 |
| `librender_service_client.so` | `/system/lib64/` | 客户端库 |
| `2d_graphics.so` | `/system/lib64/` | 2D 绘图引擎 |
| `libnative_drawing_ndk.so` | `/system/lib64/` | NDK 绘图 API |
| `libEGL.so` | `/system/lib64/` | EGL |
| `libGLESv2.so` | `/system/lib64/` | OpenGL ES 2 |
| `libGLESv3.so` | `/system/lib64/` | OpenGL ES 3 |
| `libcomposer.so` | `/system/lib64/` | 显示合成 |
| `libvsync.so` | `/system/lib64/` | VSync |
| `libcolor_picker.so` | `/system/lib64/` | 颜色选择器 |
| `libnative_effect_ndk.so` | `/system/lib64/` | NDK 效果 API |
| `graphic.rc` | `/system/etc/init/` | 初始化配置 |

---

## 构建命令

### 构建整个 graphic_2d

```bash
./build.sh --product-name <product> --ccache --build-target graphic_2d
```

### 构建特定模块

```bash
# 渲染服务
./build.sh --product-name <product> --build-target librender_service

# 客户端库
./build.sh --product-name <product> --build-target librender_service_client

# 2D 图形
./build.sh --product-name <product> --build-target 2d_graphics

# NDK 绘图
./build.sh --product-name <product> --build-target native_drawing_ndk

# OpenGL 库
./build.sh --product-name <product> --build-target //frameworks/opengl_wrapper:EGL
./build.sh --product-name <product> --build-target //frameworks/opengl_wrapper:GLESv2
```

### 运行测试

```bash
# 所有测试
./build.sh --product-name <product> --build-target graphic_common_test

# 特定测试
./build.sh --product-name <product> --build-target //foundation/graphic/graphic_2d/rosen/test/render_service:test
./build.sh --product-name <product> --build-target //foundation/graphic/graphic_2d/rosen/test/2d_graphics:test
```

---

## 关键 Defines

| Define | 作用域 | 说明 |
|--------|--------|------|
| `MODULE_RS` | render_service | 渲染服务模块 |
| `MODULE_RSB` | render_service_base | 基础模块 |
| `RS_ENABLE_GPU` | GPU 特性启用 | GPU 渲染 |
| `USE_ROSEN_DRAWING` | 绘图引擎 | Rosen 绘图 |
| `IS_OHOS` | OpenHarmony | OS 目标 |
| `OHOS_TEXT_ENABLE` | 文本支持 | 启用文本 |
| `SUPPORT_OHOS_PIXMAP` | PixelMap 支持 | 图像支持 |

---

## Sanitizer 配置

大多数共享库包含以下 sanitizer 配置：

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  cfi_vcall_icall_only = true
  boundary_sanitize = true
  integer_overflow = true
  ubsan = true
}
```

---

## 相关文档

- [目录结构](02_Directory_Structure.md) - 模块目录
- [架构说明](03_Architecture.md) - 组件交互
- [N-API 接口](04_N-API.md) - API 构建
