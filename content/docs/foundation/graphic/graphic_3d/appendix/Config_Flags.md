# 附录：配置宏与Feature Flags

## 根配置 (lume_config.gni)

### 构建类型配置

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `BUILDTYPE` | "Release" | 构建类型 (Release/Debug/MinSizeRel/RelWithDebInfo) |
| `LUME_OHOS_BUILD` | true | OHOS平台构建开关 |

### 渲染后端配置

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `RENDER_BUILD_VULKAN` | true | 启用Vulkan后端 |
| `RENDER_BUILD_GLES` | true | 启用OpenGL ES后端 |
| `RENDER_BUILD_GL` | false | 启用桌面OpenGL |
| `RENDER_BUILD_2D` | false | 启用2D渲染 |

### 库名称配置

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `LIB_ENGINE_CORE` | "libAGPDLL" | 引擎核心DLL名称 |
| `LIB_RENDER` | "libPluginAGPRender" | 渲染插件名称 |
| `LIB_CORE3D` | "libPluginAGP3D" | 3D插件名称 |
| `LIB_PNG` | "libPluginAGPPng" | PNG插件名称 |
| `LIB_JPG` | "libPluginAGPJpg" | JPG插件名称 |

### 功能开关

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `USE_LIB_PNG_JPEG_DYNAMIC_PLUGIN` | true | 使用动态PNG/JPG插件 |
| `USE_STB_IMAGE` | false | 使用STB图片库 |
| `CORE3D_EMBEDDED_ASSETS_ENABLED` | 2 | 3D资源嵌入模式 |

---

## LumeEngine 编译宏

### 日志配置 (lume_engine_api)

| 宏 | 条件 | 说明 |
|----|------|------|
| `CORE_LOG_NO_DEBUG` | BUILDTYPE相关 | 禁用调试日志 |
| `CORE_LOG_TO_CONSOLE` | 1 | 输出到控制台 |
| `CORE_LOG_TO_DEBUG_OUTPUT` | 1 | 输出到调试器 |
| `CORE_LOG_TO_FILE` | 0 | 输出到文件 |
| `CORE_LOG_DISABLED` | CORE_LOG_ENABLED | 完全禁用日志 |

### 构建配置

| 宏 | 值 | 说明 |
|----|----|----|
| `CORE_BUILD_BASE` | 1 | 构建基础库 |
| `CORE_HIDE_SYMBOLS` | 1 | 隐藏符号 |
| `CORE_PERF_ENABLED` | 0 | 性能分析开关 |
| `CORE_VALIDATION_ENABLED` | 0 | 验证层开关 |
| `CORE_TESTS_ENABLED` | 0 | 测试开关 |

### 平台宏

| 宏 | 条件 | 说明 |
|----|------|------|
| `__OHOS_PLATFORM__` | LUME_OHOS_BUILD | OHOS平台标识 |
| `VK_USE_PLATFORM_OHOS` | 1 | Vulkan OHOS平台 |

---

## Widget/Kits 编译宏

### 3d_widget_adapter 配置

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `MULTI_ECS_UPDATE_AT_ONCE` | "0" | 多ECS同时更新 |
| `UNIFY_RENDER` | "1" | 统一渲染 |
| `WIDGET_TRACE_ENABLE` | "1" | Widget追踪 |
| `DBG_DRAW_PIXEL` | "0" | 像素调试绘制 |

### 渲染后端支持

| 宏 | 值 | 说明 |
|----|----|----|
| `CORE_HAS_GLES_BACKEND` | 1 | 支持GLES后端 |
| `CORE_HAS_VULKAN_BACKEND` | 1 | 支持Vulkan后端 |
| `RENDER_HAS_GLES_BACKEND` | 1 | GLES后端启用 |
| `RENDER_HAS_VULKAN_BACKEND` | 1 | Vulkan后端启用 |

### N-API配置

| 宏 | 条件 | 说明 |
|----|------|------|
| `__OHOS_PLATFORM__` | current_os == "ohos" | OHOS平台 |
| `__SCENE_ADAPTER__` | 1 | 场景适配 |
| `__NAPI_CALL_TF_WITH_PRIORITY__` | 1 | N-API优先级调用 |
| `__PHYSICS_MODULE__` | 1 | 物理模块开关 |

---

## LumeRender 编译宏

### 调试配置

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `CORE_GL_DEBUG` | 0 | OpenGL调试 |
| `CORE_VALIDATION_ENABLED` | 0 | 验证层 |
| `CORE_VULKAN_VALIDATION_ENABLED` | 0 | Vulkan验证层 |
| `CORE_DEBUG_GPU_RESOURCE_IDS` | 0 | GPU资源ID调试 |
| `CORE_DEBUG_COMMAND_MARKERS_ENABLED` | 0 | 命令标记调试 |
| `CORE_DEBUG_MARKERS_ENABLED` | 0 | 标记调试 |

### 性能配置

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `CORE_ENABLE_GPU_QUERIES` | 0 | GPU查询 |
| `CORE_EMBEDDED_ASSETS_ENABLED` | 2 | 嵌入资源模式 |

### 编译器配置

| 宏 | 值 | 说明 |
|----|----|----|
| `CORE_PUBLIC` | `__attribute__((visibility("default")))` | 符号可见性 |
| `CORE_DYNAMIC` | 1 | 动态链接 |
| `CORE_BUILD_GLES` | 1 | 构建GLES |
| `CORE_BUILD_VULKAN` | 1 | 构建Vulkan |

---

## 路径配置宏

### 安装路径

| 宏 | 值 (arm64/x64) | 值 (arm/x86) |
|----|----------------|--------------|
| `PLATFORM_CORE_ROOT_PATH` | "/system/lib64/" | "/system/lib/" |
| `PLATFORM_CORE_PLUGIN_PATH` | "/system/lib64/graphics3d/" | "/system/lib/graphics3d/" |
| `PLATFORM_APP_ROOT_PATH` | "/system/lib64/" | "/system/lib/" |
| `PLATFORM_APP_PLUGIN_PATH` | "/system/lib64/graphics3d/" | "/system/lib/graphics3d/" |

---

## 特性开关使用示例

### 禁用Vulkan后端

```gn
# 在 BUILD.gn 或 .gni 中
RENDER_BUILD_VULKAN = false
```

或添加编译定义：
```gn
cflags = [
  "-DCORE_BUILD_VULKAN=0",
  "-DRENDER_HAS_VULKAN_BACKEND=0",
]
```

### 启用调试日志

```gn
# lume/LumeEngine/BUILD.gn
defines = [
  "CORE_LOG_NO_DEBUG=0",
  "CORE_LOG_DEBUG=1",
]
```

### 禁用插件动态加载

```gn
# 在根配置中
USE_LIB_PNG_JPEG_DYNAMIC_PLUGIN = false
```

---

## 相关文档

- [GN构建 →](../05_GN_Build.md)
- [架构设计 →](../02_Architecture.md)
