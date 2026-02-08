# 配置开关与宏定义

## 概述

本文档列出 graphic_2d 中所有关键配置开关 (Feature Flags) 和编译宏。

---

## Feature Flags (graphic_config.gni)

### 产品相关开关

| 开关 | 默认值 | 产品默认值 | 说明 |
|------|--------|------------|------|
| `graphic_2d_feature_product` | `"default"` | phone/pc/tablet/wearable | 产品类型 |
| `is_cross_platform` | `false` | `false` | 跨平台支持 |

### GPU/图形 API 开关

| 开关 | 默认值 | 产品默认值 | 说明 |
|------|--------|------------|------|
| `graphic_2d_feature_enable_vulkan` | `false` | `true` | 启用 Vulkan 后端 |
| `graphic_2d_feature_enable_opengl` | `true` | `true` | 启用 OpenGL 后端 |
| `graphic_2d_feature_enable_opinc` | `false` | phone/tablet/wearable | OpenGL Incircle 优化 |

### 渲染特性开关

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `graphic_2d_feature_rs_enable_uni_render` | `false` | 统一渲染模式 |
| `graphic_2d_feature_parallel_render_enable` | `true` | 并行渲染 |
| `graphic_2d_feature_enable_stack_culling` | `false` | `true` (phone/pc/tablet/wearable) | 堆叠剔除优化 |
| `graphic_2d_feature_rs_enable_eglimage` | `false` | EGLImage 支持 |
| `graphic_2d_feature_enable_afbc` | `false` | ARM Frame Buffer Compression |
| `graphic_2d_feature_freemem_enable` | `false` | 空闲内存释放 |
| `graphic_2d_feature_color_gamut_enable` | `false` | 色域管理 |
| `graphic_2d_feature_enable_sdf` | `false` | SDF (Signed Distance Field) |

### 动画与特效开关

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `enable_text_gine` | `true` | TextGine 文本引擎 |
| `use_skia_txt` | `true` | Skia 文本渲染 |
| `graphic_2d_feature_use_texgine` | `false` | TextureGine |

### 系统特性开关

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `graphic_2d_feature_bootanimation_enable` | `true` | 开机动画 |
| `graphic_2d_feature_bootanimation_ext_enable` | `"default"` | 开机动画扩展 |
| `graphic_2d_feature_rs_enable_profiler` | `true` | Profiler 支持 |
| `graphic_2d_feature_enable_chipset_vsync` | `false` | 芯片级 VSync |
| `graphic_2d_feature_tp_switch_enbale` | `false` | 触摸板切换 |
| `graphic_2d_feature_overlay_display_enable` | `false` | 叠加显示 |
| `graphic_2d_feature_screenless_enable` | `false` | 无屏模式 |
| `graphic_2d_feature_tv_metadata_enable` | `false` | TV 元数据 |

### 性能优化开关

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `graphic_2d_feature_enable_pgo` | `false` | PGO (Profile-Guided Optimization) |
| `graphic_2d_feature_enable_codemerge` | `false` | 代码合并 |
| `graphic_2d_feature_pgo_path` | `""` | PGO 数据路径 |
| `graphic_2d_feature_enable_prefetch` | `true` | 预取优化 |
| `graphic_2d_feature_rs_enable_rspipeline` | `true` | RS Pipeline 优化 |
| `graphic_2d_feature_subtree_parallel_enable` | `false` | 子树并行处理 |
| `graphic_2d_feature_enable_filter_cache` | `true` | 滤镜缓存 |

### 内存与安全开关

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `graphic_2d_support_access_token` | `true` | AccessToken 支持 |
| `graphic_2d_feature_enable_memory_info_manager` | `false` | 内存信息管理 |
| `graphic_2d_feature_enable_memory_downtree` | `false` | 内存向下树优化 |
| `graphic_2d_feature_enable_rdo` | `false` | RDO (Render Decision Optimization) |
| `graphic_2d_feature_enable_dvsync` | `false` | 动态 VSync |

### HDR 与显示开关

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `graphic_2d_feature_wuji_enable` | `false` | 无界显示 |
| `graphic_2d_feature_hetero_hdr_enable` | `false` | 异构 HDR |
| `graphic_2d_feature_mhc_enable` | `false` | MHC (Multi-HDR Composition) |

### 调试与分析开关

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `logger_enable_scope` | `false` | 范围日志 |
| `enable_full_screen_recongnize` | `false` | 全屏识别 |

**证据来源**: `graphic_config.gni:15-100`

---

## 编译宏定义

### 模块宏

| 宏 | 作用域 | 说明 |
|-----|--------|------|
| `MODULE_RS` | render_service | 渲染服务模块 |
| `MODULE_RSB` | render_service_base | 基础模块 |
| `MODULE_RSC` | render_service_client | 客户端模块 |

### 功能宏

| 宏 | 作用域 | 说明 |
|-----|--------|------|
| `RS_ENABLE_GPU` | GPU 特性 | GPU 渲染启用 |
| `USE_ROSEN_DRAWING` | 绘图引擎 | Rosen 绘图 |
| `IS_OHOS` | OS 目标 | OpenHarmony 系统 |
| `OHOS_TEXT_ENABLE` | 文本 | 文本支持 |
| `SUPPORT_OHOS_PIXMAP` | 图像 | PixelMap 支持 |

### 图形 API 宏

| 宏 | 作用域 | 说明 |
|-----|--------|------|
| `USE_ACE_SKIA` | Skia | ACE Skia 后端 |
| `GL_GLEXT_PROTOTYPES` | OpenGL | OpenGL 原型声明 |
| `EGL_EGLEXT_PROTOTYPES` | EGL | EGL 原型声明 |

### 平台宏

| 宏 | 作用域 | 说明 |
|-----|--------|------|
| `IS_ENABLE_DRM` | DRM | DRM 支持 |
| `IS_OHOS_SURFACE` | Surface | OHOS Surface |

---

## IPC 安全宏

| 宏 | 默认值 | 说明 |
|-----|--------|------|
| `ENABLE_IPC_SECURITY` | 启用 | IPC 安全检查 |

**注意**: 禁用此宏会完全绕过所有 IPC 安全检查！

**证据来源**: `rs_ipc_interface_code_access_verifier_base.cpp:268-299`

---

## 传感器与输入宏

| 宏 | 默认值 | 说明 |
|-----|--------|------|
| `ENABLE_MAGIC_CURSOR` | 可选 | 魔法光标功能 |

---

## 构建配置

### 产物类型

| 类型 | 宏 | 说明 |
|------|-----|------|
| `ohos_shared_library` | - | 动态库 (.so) |
| `ohos_static_library` | - | 静态库 (.a) |
| `ohos_prebuilt_etc` | - | 预制配置文件 |
| `source_set` | - | 源文件组 |

### 标签 (Tags)

| 标签 | 用途 |
|------|------|
| `ndk` | NDK API |
| `llndk` | Low-level NDK |
| `innerapi` | Inner API |

**证据来源**: `06_Build.md` - 构建配置

---

## 代码路径条件编译

### 平台相关

```cpp
#if defined(IS_OHOS) && defined(USE_ROSEN_DRAWING)
// OpenHarmony + Rosen 绘图
#elif defined(USE_DEFAULT_DRAWING)
// 默认绘图实现
#endif
```

### GPU 后端选择

```cpp
#if defined(RS_ENABLE_GPU)
    #if defined(USE_VULKAN)
        // Vulkan 后端
    #elif defined(USE_OPENGL)
        // OpenGL 后端
    #endif
#endif
```

### 特性开关

```cpp
#ifdef ENABLE_IPC_SECURITY
    // IPC 安全检查代码
#else
    // 绕过安全检查
#endif
```

---

## 关键配置示例

### phone 产品配置

```gn
graphic_2d_feature_product = "phone"

graphic_2d_feature_enable_vulkan = true
graphic_2d_feature_enable_opengl = true
graphic_2d_feature_enable_stack_culling = true
graphic_2d_feature_enable_opinc = true
enable_full_screen_recongnize = true
```

### wearable 产品配置

```gn
graphic_2d_feature_product = "wearable"

graphic_2d_feature_enable_vulkan = true
graphic_2d_feature_enable_opengl = true
graphic_2d_feature_enable_stack_culling = true
graphic_2d_feature_enable_opinc = true
```

### 跨平台配置

```gn
is_cross_platform = true

graphic_2d_feature_enable_vulkan = false  # 根据平台调整
graphic_2d_feature_enable_opengl = true
```

---

## 相关文档

- [构建文档](../06_Build.md) - GN 构建配置
- [架构说明](../03_Architecture.md) - 功能模块
- [安全评审](../07_Security.md) - 安全相关开关
