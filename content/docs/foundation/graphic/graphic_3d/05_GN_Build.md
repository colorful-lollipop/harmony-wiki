# GN构建系统

## 根配置

**文件**: `lume/lume_config.gni`

### 全局变量

| 变量名 | 值 | 说明 |
|--------|-----|------|
| `BUILDTYPE` | "Release" | 构建类型 |
| `LUME_OHOS_BUILD` | true | OHOS平台构建开关 |
| `RENDER_BUILD_VULKAN` | true | Vulkan后端启用 |
| `RENDER_BUILD_GLES` | true | GLES后端启用 |
| `USE_LIB_PNG_JPEG_DYNAMIC_PLUGIN` | true | PNG/JPG动态插件 |
| `LIB_ENGINE_CORE` | "libAGPDLL" | 引擎核心库名 |
| `LIB_RENDER` | "libPluginAGPRender" | 渲染插件库名 |
| `LIB_CORE3D` | "libPluginAGP3D" | 3D插件库名 |

### 路径配置

```gni
LUME_ROOT = "//foundation/graphic/graphic_3d/lume/"
LUME_CORE_PATH = "${LUME_ROOT}/LumeEngine"
LUME_RENDER_PATH = "${LUME_ROOT}/LumeRender"
LUME_CORE3D_PATH = "${LUME_ROOT}/Lume_3D"
LUME_PNG_PATH = "${LUME_ROOT}/LumePng"
LUME_JPG_PATH = "${LUME_ROOT}/LumeJpg"
```

### 自定义模板

| 模板 | 功能 |
|------|------|
| `lume_rofs` | 资源打包模板 |
| `lume_compile_shader` | 着色器编译模板 |
| `lume_binary_complile` | 二进制编译模板 |

---

## 关键Targets

### 1. 适配层

#### 3d_widget_adapter

**文件**: `3d_widget_adapter/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `widget_adapter_source` | ohos_source_set | - | 适配器源码集合 |
| `3dWidgetAdapterInterface` | group | - | 接口配置组 |
| `lib3dWidgetAdapter` | ohos_shared_library | `lib3dWidgetAdapter.z.so` | **主适配器库** |

**lib3dWidgetAdapter 配置**:
```gn
ohos_shared_library("lib3dWidgetAdapter") {
  deps = [
    ":widget_adapter_source",
    "../3d_scene_adapter:scene_adapter_static",
  ]
  
  external_deps = [
    "graphic_surface:surface",
    "vulkan-headers:vulkan_headers",
    "vulkan-loader:vulkan_loader",
    "graphic_2d:EGL",
    "graphic_2d:GLESv3",
    "ability_runtime:ability_manager",
    "ability_runtime:napi_common",
    "ipc:ipc_single",
    "napi:ace_napi",
    # ... 更多依赖
  ]
  
  defines = [
    "PLATFORM_CORE_ROOT_PATH=/system/lib64/",
    "PLATFORM_CORE_PLUGIN_PATH=/system/lib64/graphics3d/",
  ]
}
```

#### 3d_scene_adapter

**文件**: `3d_scene_adapter/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `scene_adapter_static` | ohos_static_library | - | 场景适配静态库 |
| `sceneAdapterInterface` | group | - | 接口配置组 |

---

### 2. 引擎核心

#### LumeEngine

**文件**: `lume/LumeEngine/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `lume_engine_src` | ohos_source_set | - | 引擎核心源码 |
| `libAGPEngine` | ohos_static_library | `libAGPEngine.a` | **引擎静态库** |
| `AGPBaseApi` | ohos_shared_library | - | 基础API接口 |
| `AGPEngineApi` | ohos_shared_library | - | 引擎API接口 |
| `AGPEcshelperApi` | ohos_shared_library | - | ECS辅助API |
| `libComponentHelper` | ohos_static_library | `libComponentHelper.a` | 组件帮助库 |

**lume_engine_src 关键配置**:
```gn
ohos_source_set("lume_engine_src") {
  sources = [
    "src/ecs/ecs.cpp",
    "src/ecs/entity_manager.cpp",
    "src/engine.cpp",
    "src/image/image_loader_manager.cpp",
    "src/io/filesystem_api.cpp",
    "src/plugin_registry.cpp",
    "src/threading/dispatcher_impl.cpp",
    # ... 共50+源文件
  ]
  
  external_deps = [
    "c_utils:utils",
    "resource_management:global_resmgr",
    "qos_manager:qos",
    "resource_schedule_service:ressched_client",
  ]
}
```

**编译选项** (lume_default config):
```gn
cflags = [
  "-Wno-unused-function",
  "-Wno-unused-parameter",
  "-Wno-sign-compare",
  "-fno-rtti",
  "-fvisibility=hidden",
  "-ffunction-sections",
  "-fdata-sections",
]

cflags_cc = [
  "-std=c++17",
  "-fno-rtti",
  "-fvisibility=hidden",
]

ldflags = [
  "-fuse-ld=lld",
  "-flto=thin",
  "-Wl,--gc-sections",
]
```

#### LumeEngine DLL

**文件**: `lume/LumeEngine/DLL/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libAGPDLL` | ohos_shared_library | `libAGPDLL.z.so` | **引擎核心DLL** |

```gn
ohos_shared_library("libAGPDLL") {
  deps = [
    ":lume_engine_dynamic_src",
    "${LUME_CORE_PATH}:libAGPEngine",
  ]
  relative_install_dir = "graphics3d"
}
```

---

### 3. 渲染后端

#### LumeRender

**文件**: `lume/LumeRender/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `lume_render_src` | ohos_source_set | - | 渲染核心源码 |
| `libPluginAGPRender` | ohos_shared_library | `libPluginAGPRender.z.so` | **渲染插件库** |
| `AGPRenderApi` | ohos_shared_library | - | 渲染API接口 |

**lume_render_src 条件编译**:
```gn
ohos_source_set("lume_render_src") {
  sources = [
    "src/plugin/static_plugin.cpp",
    "src/device/gpu_resource_manager.cpp",
    "src/renderer.cpp",
    "src/render_context.cpp",
    # ... 共180+源文件
  ]
  
  # GLES后端
  if (RENDER_BUILD_GLES) {
    sources += [
      "src/gles/device_gles.cpp",
      "src/gles/render_backend_gles.cpp",
    ]
    external_deps += ["graphic_2d:EGL", "graphic_2d:GLESv3"]
  }
  
  # Vulkan后端
  if (RENDER_BUILD_VULKAN) {
    sources += [
      "src/vulkan/device_vk.cpp",
      "src/vulkan/render_backend_vk.cpp",
    ]
    external_deps += [
      "vulkan-headers:vulkan_headers",
      "vulkan-loader:vulkan_loader"
    ]
  }
}
```

---

### 4. 3D功能

#### Lume_3D

**文件**: `lume/Lume_3D/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `lume_3d_src` | ohos_source_set | - | 3D核心源码 |
| `AGP3DApi` | ohos_shared_library | - | 3D API接口 |
| `libAGP3D` | ohos_static_library | `libAGP3D.a` | **3D静态库** |

**lume_3d_src 配置**:
```gn
ohos_source_set("lume_3d_src") {
  sources = [
    "src/ecs/components/animation_component_manager.cpp",
    "src/ecs/components/camera_component_manager.cpp",
    "src/ecs/systems/animation_system.cpp",
    "src/ecs/systems/render_system.cpp",
    "src/gltf/gltf2.cpp",
    "src/gltf/gltf2_importer.cpp",
    "src/gltf/gltf2_loader.cpp",
    # ... 共100+源文件
  ]
  
  deps = ["${LUME_CORE_PATH}/ecshelper:libAGPEcshelper"]
}
```

#### Lume_3D DLL

**文件**: `lume/Lume_3D/DLL/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libPluginAGP3D` | ohos_shared_library | `libPluginAGP3D.z.so` | **3D插件DLL** |

---

### 5. Kits层

#### JS Kits (N-API)

**文件**: `kits/js/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `napi_source` | ohos_source_set | - | NAPI源码 |
| `libKitHelper` | ohos_shared_library | `libKitHelper.z.so` | NAPI辅助库 |
| `libscene` | ohos_shared_library | `libscene.z.so` | **Scene NAPI库** |

**napi_source 配置**:
```gn
ohos_source_set("napi_source") {
  sources = [
    "src/AnimationJS.cpp",
    "src/CameraJS.cpp",
    "src/GeometryJS.cpp",
    "src/SceneJS.cpp",
    "src/NodeJS.cpp",
    "src/MaterialJS.cpp",
    "src/register_module.cpp",
    "src/napi/array.cpp",
    "src/napi/env.cpp",
    # ... 共60+源文件
  ]
  
  deps = ["../../3d_widget_adapter:lib3dWidgetAdapter"]
  
  external_deps = [
    "graphic_2d:EGL",
    "graphic_2d:GLESv3",
    "graphic_surface:surface",
    "hilog:libhilog",
    "hitrace:hitrace_meter",
    "napi:ace_napi",
  ]
}
```

**libscene 配置**:
```gn
ohos_shared_library("libscene") {
  deps = [":napi_entry"]
  relative_install_dir = "module/graphics"
}
```

#### ETS Kits (Taihe)

**文件**: `kits/ets/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `run_taihe` | ohos_taihe | - | Taihe代码生成 |
| `scene_ani` | taihe_shared_library | `scene_ani.z.so` | **ANI场景库** |
| `graphics3d_scene` | generate_static_abc | `graphics3d_scene.abc` | Scene字节码 |
| `graphics3d_scene_nodes` | generate_static_abc | `graphics3d_scene_nodes.abc` | Nodes字节码 |
| `graphics3d_scene_resources` | generate_static_abc | `graphics3d_scene_resources.abc` | Resources字节码 |
| `graphics3d_scene_types` | generate_static_abc | `graphics3d_scene_types.abc` | Types字节码 |
| `graphics3d_scene_post_process_settings` | generate_static_abc | `graphics3d_scene_post_process_settings.abc` | PostProcess字节码 |
| `graphics_3d_taihe` | group | - | Taihe组合目标 |

**scene_ani 配置**:
```gn
taihe_shared_library("scene_ani") {
  sources = [
    # Taihe生成
    "$taihe_generated_file_path/src/SceneTH.ani.cpp",
    "$taihe_generated_file_path/src/SceneNodes.ani.cpp",
    # 手写实现
    "taihe/src/SceneImpl.cpp",
    "taihe/src/NodeImpl.cpp",
    "taihe/src/CameraImpl.cpp",
    "taihe/src/MeshImpl.cpp",
    # ... 共40+文件
  ]
  
  deps = [
    ":run_taihe",
    "../../3d_widget_adapter:lib3dWidgetAdapter",
    "../js:libKitHelper",
  ]
}
```

---

### 6. 功能插件

| 模块 | Target | 类型 | 输出 | 安装路径 |
|------|--------|------|------|----------|
| LumePng | `libPluginAGPPng` | ohos_shared_library | `libPluginAGPPng.z.so` | graphics3d/ |
| LumeJpg | `libPluginAGPJpg` | ohos_shared_library | `libPluginAGPJpg.z.so` | graphics3d/ |
| LumeFont | `libPluginFont` | ohos_shared_library | `libPluginFont.z.so` | graphics3d/ |
| Lume_3DText | `libPluginAGP3DText` | ohos_shared_library | `libPluginAGP3DText.z.so` | graphics3d/ |
| LumeScene | `libPluginSceneWidget` | ohos_shared_library | `libPluginSceneWidget.z.so` | graphics3d/ |
| LumeMeta | `libPluginMetaObject` | ohos_shared_library | `libPluginMetaObject.z.so` | graphics3d/ |
| camera_preview_plugin | `libPluginCamPreview` | ohos_shared_library | `libPluginCamPreview.z.so` | graphics3d/ |
| LumeDotfield | `libPluginDotfield` | ohos_shared_library | `libPluginDotfield.z.so` | graphics3d/ |

---

## 依赖关系图

```
                         ┌──────────────────┐
                         │ lib3dWidgetAdapter│
                         │   (主适配器库)    │
                         └────────┬─────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         │                        │                        │
         ▼                        ▼                        ▼
┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ scene_adapter_static│    │    libKitHelper    │    │  3d_particles_app │
│  (场景适配静态库)   │    │   (NAPI 辅助库)    │    │   (粒子效果扩展)   │
└────────┬────────┘    └────────┬─────────┘    └──────────────────┘
         │                        │
         │                        ▼
         │              ┌──────────────────┐
         │              │    napi_source    │
         │              │  (NAPI 接口实现)  │
         │              └────────┬─────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌──────────────────┐
│   libAGPEngine  │◄───│     libscene      │
│   (引擎静态库)   │    │   (Scene NAPI)    │
└────────┬────────┘    └──────────────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌────────┐  ┌──────────────┐
│libAGPDLL│  │libComponentHelper│
│(引擎DLL)│  │  (组件帮助库)     │
└────┬───┘  └──────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────┐
│              插件层 (graphics3d/ 目录)                │
├─────────────┬─────────────┬─────────────┬────────────┤
│libPluginAGPRender│libPluginAGP3D │libPluginFont │libPlugin... │
│ (渲染插件)   │  (3D 插件)   │ (字体插件)   │ (其他插件)  │
└─────────────┴─────────────┴─────────────┴────────────┘
```

---

## 编译产物清单

### 共享库 (.z.so)

| 产物文件名 | 安装路径 | 说明 |
|-----------|----------|------|
| `lib3dWidgetAdapter.z.so` | /system/lib64/ | ArkUI适配库 |
| `libAGPDLL.z.so` | /system/lib64/ | 引擎核心DLL |
| `libscene.z.so` | /system/lib64/module/graphics/ | JS N-API模块 |
| `libKitHelper.z.so` | /system/lib64/ | NAPI辅助库 |
| `scene_ani.z.so` | /system/lib64/ | ETS ANI模块 |
| `libPluginAGPRender.z.so` | /system/lib64/graphics3d/ | 渲染插件 |
| `libPluginAGP3D.z.so` | /system/lib64/graphics3d/ | 3D插件 |
| `libPluginAGPPng.z.so` | /system/lib64/graphics3d/ | PNG解码插件 |
| `libPluginAGPJpg.z.so` | /system/lib64/graphics3d/ | JPG解码插件 |
| `libPluginFont.z.so` | /system/lib64/graphics3d/ | 字体插件 |
| `libPluginAGP3DText.z.so` | /system/lib64/graphics3d/ | 3D文字插件 |
| `libPluginSceneWidget.z.so` | /system/lib64/graphics3d/ | 场景控件插件 |
| `libPluginMetaObject.z.so` | /system/lib64/graphics3d/ | 元对象插件 |
| `libPluginCamPreview.z.so` | /system/lib64/graphics3d/ | 相机预览插件 |
| `libPluginDotfield.z.so` | /system/lib64/graphics3d/ | 点场插件 |

### 静态库 (.a)

| 产物文件名 | 说明 |
|-----------|------|
| `libAGPEngine.a` | 引擎核心静态库 |
| `libAGP3D.a` | 3D功能静态库 |
| `libComponentHelper.a` | 组件帮助静态库 |

### ArkTS字节码 (.abc)

| 产物文件名 | 安装路径 | 说明 |
|-----------|----------|------|
| `graphics3d_scene.abc` | /system/framework/ | Scene模块 |
| `graphics3d_scene_nodes.abc` | /system/framework/ | Nodes模块 |
| `graphics3d_scene_resources.abc` | /system/framework/ | Resources模块 |
| `graphics3d_scene_types.abc` | /system/framework/ | Types模块 |
| `graphics3d_scene_post_process_settings.abc` | /system/framework/ | PostProcess模块 |

---

## 构建命令

### 全模块构建

```bash
hb build graphic_3d
```

### 单Target构建

```bash
# 构建适配器库
hb build //foundation/graphic/graphic_3d/3d_widget_adapter:lib3dWidgetAdapter

# 构建引擎DLL
hb build //foundation/graphic/graphic_3d/lume/LumeEngine/DLL:libAGPDLL

# 构建渲染插件
hb build //foundation/graphic/graphic_3d/lume/LumeRender:libPluginAGPRender

# 构建JS N-API模块
hb build //foundation/graphic/graphic_3d/kits/js:libscene

# 构建ETS模块
hb build //foundation/graphic/graphic_3d/kits/ets:graphics_3d_taihe
```

---

## bundle.json构建入口

**文件**: `bundle.json` (行 398-416)

```json
"sub_component": [
  "//foundation/graphic/graphic_3d/lume/LumeEngine:libAGPEngine",
  "//foundation/graphic/graphic_3d/lume/LumeEngine/DLL:libAGPDLL",
  "//foundation/graphic/graphic_3d/lume/LumeRender:libPluginAGPRender",
  "//foundation/graphic/graphic_3d/lume/Lume_3D/DLL:libPluginAGP3D",
  "//foundation/graphic/graphic_3d/lume/Lume_3D:libAGP3D",
  "//foundation/graphic/graphic_3d/lume/LumePng:libPluginAGPPng",
  "//foundation/graphic/graphic_3d/lume/LumeJpg:libPluginAGPJpg",
  "//foundation/graphic/graphic_3d/3d_widget_adapter:lib3dWidgetAdapter",
  "//foundation/graphic/graphic_3d/kits/js:libscene",
  "//foundation/graphic/graphic_3d/kits/ets:graphics_3d_taihe",
  "//foundation/graphic/graphic_3d/lume/LumeFont:libPluginFont",
  "//foundation/graphic/graphic_3d/lume/Lume_3DText:libPluginAGP3DText",
  "//foundation/graphic/graphic_3d/camera_preview_plugin:libPluginCamPreview"
]
```

---

## 运行时加载关系

```
1. 应用加载 lib3dWidgetAdapter.z.so
   └─> 初始化图形系统
   
2. 应用加载 libscene.z.so (scene.napi)
   └─> 调用 Graphics3dKitRegisterModule
       └─> 加载 libAGPDLL.z.so
           └─> 扫描并加载 graphics3d/ 目录下插件
               ├─> libPluginAGPRender.z.so
               ├─> libPluginAGP3D.z.so
               ├─> libPluginFont.z.so
               └─> ...其他插件

3. ETS应用加载 scene_ani.z.so
   └─> 依赖 lib3dWidgetAdapter.z.so
   └─> 依赖 libKitHelper.z.so
```

---

## 下一步

- [查看安全风险 →](06_Security.md)
- [查看架构设计 →](02_Architecture.md)
- [查看常见问题 →](07_FAQ.md)
