# 目录结构与模块职责

## 顶层目录概览

```
foundation/graphic/graphic_3d/
├── 3d_widget_adapter/       # ArkUI适配层 - 连接ArkUI与引擎核心
├── 3d_scene_adapter/        # 场景适配层 - 场景管理适配
├── camera_preview_plugin/   # 相机预览插件 - 相机画面到3D纹理
├── kits/                    # API层 - JS/ETS接口实现
│   ├── ets/                 # ArkTS/ETS接口 (Taihe框架)
│   └── js/                  # JavaScript接口 (N-API)
├── lume/                    # 引擎核心 - 所有核心功能
│   ├── LumeBase/            # 基础库 - 数学、容器、工具
│   ├── LumeBinaryCompile/   # 编译工具 - Shader/资源编译
│   ├── LumeDotfield/        # 点场渲染插件
│   ├── LumeEngine/          # 引擎框架 - ECS、资源、线程、插件
│   ├── LumeFont/            # 字体渲染插件
│   ├── LumeJpg/             # JPG解码插件
│   ├── LumeMeta/            # 元对象系统
│   ├── LumePng/             # PNG解码插件
│   ├── LumeRender/          # 渲染后端 - 渲染管线、GPU抽象
│   ├── LumeScene/           # 场景管理插件
│   └── Lume_3D/             # 3D功能 - ECS组件、GLTF、动画
├── figures/                 # 文档图片
├── metadata/                # 元数据文件
└── test/                    # 测试代码 (本文档忽略)
```

## 模块详细说明

### 1. 适配层 (Adapter Layer)

#### 1.1 3d_widget_adapter
**路径**: `3d_widget_adapter/`

**职责**: 连接 ArkUI 框架与 AGP 引擎，提供Widget级别的3D渲染能力。

| 子目录 | 说明 |
|--------|------|
| `include/` | 公共头文件，包括平台抽象 |
| `core/src/` | 核心适配逻辑 |
| `src/ohos/` | OpenHarmony平台实现 |
| `src/android/` | Android平台实现(预留) |

**关键文件**:
- `widget_adapter.cpp` - 主适配器实现
- `graphics_manager.cpp` - 图形管理器
- `texture_layer.cpp` - 纹理层管理
- `BUILD.gn` - 产出 `lib3dWidgetAdapter.z.so`

**证据**: 
```gn
# 3d_widget_adapter/BUILD.gn:253
ohos_shared_library("lib3dWidgetAdapter") {
  deps = [":widget_adapter_source", "../3d_scene_adapter:scene_adapter_static"]
}
```

#### 1.2 3d_scene_adapter
**路径**: `3d_scene_adapter/`

**职责**: 场景级别的适配，提供Scene与ArkUI Node的桥接。

| 子目录 | 说明 |
|--------|------|
| `include/scene_adpater/` | 场景适配头文件 |
| `src/` | 场景桥接实现 |

**关键文件**:
- `scene_adapter.h/cpp` - 场景适配器
- `scene_bridge.h/cpp` - 场景桥接

---

### 2. API层 (Kits Layer)

#### 2.1 kits/js - JavaScript N-API接口
**路径**: `kits/js/`

**职责**: 提供 JavaScript 可调用的 N-API 接口，使JS应用能操作3D场景。

| 子目录 | 说明 |
|--------|------|
| `include/` | N-API绑定头文件 |
| `src/` | N-API绑定实现 |
| `src/napi/` | N-API工具封装 |
| `src/geometry_definition/` | 几何体定义(JS) |

**关键文件**:
- `native_module_export.cpp` - N-API模块注册入口
- `register_module.cpp` - 类注册实现
- `SceneJS.cpp` - Scene对象绑定
- `NodeJS.cpp` - Node对象绑定
- `MeshJS.cpp` - Mesh对象绑定
- `MaterialJS.cpp` - Material对象绑定

**证据**:
```cpp
// kits/js/src/native_module_export.cpp:81-89
static napi_module g_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Export,
    .nm_modname = "scene.napi",  // JS中使用的模块名
    .nm_priv = (reinterpret_cast<void *>(0)),
    .reserved = {0}
};
```

**产物**:
- `libKitHelper.z.so` - N-API辅助库
- `libscene.z.so` - 主N-API模块

#### 2.2 kits/ets - ArkTS Taihe接口
**路径**: `kits/ets/`

**职责**: 提供 ArkTS (ETS) 类型的API接口，使用Taihe框架生成。

| 子目录 | 说明 |
|--------|------|
| `include/` | ETS绑定头文件 |
| `src/` | ETS绑定实现 |
| `taihe/idl/` | Taihe IDL定义文件 |
| `taihe/src/` | Taihe实现代码 |

**关键文件**:
- `taihe/idl/SceneTH.taihe` - Scene接口定义
- `taihe/idl/SceneNodes.taihe` - Node接口定义
- `taihe/src/SceneImpl.cpp` - Scene实现
- `src/SceneETS.cpp` - ETS到Native桥接

**产物**:
- `scene_ani.z.so` - Taihe运行时库
- `graphics3d_scene.abc` - ArkTS字节码

---

### 3. 引擎核心 (Lume Core)

#### 3.1 LumeBase - 基础库
**路径**: `lume/LumeBase/`

**职责**: 提供基础数据类型、数学库、容器、工具函数。

| 子目录 | 说明 |
|--------|------|
| `api/base/` | 基础工具API |
| `api/base/math/` | 数学库 (Vector, Matrix, Quaternion) |
| `api/base/containers/` | 容器实现 (vector, map, string) |

**关键类**:
- `Math::Vector` - 向量 (Vec2, Vec3, Vec4)
- `Math::Matrix` - 矩阵 (Mat3X3, Mat4X4)
- `Math::Quaternion` - 四元数
- `string_view`, `fixed_string` - 字符串工具

**证据**:
```cpp
// lume/LumeBase/api/base/math/vector.h
struct Vec3 {
    float x, y, z;
    // ... 向量运算
};
```

#### 3.2 LumeEngine - 引擎框架
**路径**: `lume/LumeEngine/`

**职责**: ECS框架、资源管理、线程管理、插件系统、平台抽象。

| 子目录 | 说明 |
|--------|------|
| `api/core/ecs/` | ECS接口定义 |
| `api/core/plugin/` | 插件系统接口 |
| `api/core/io/` | IO/文件系统接口 |
| `api/core/threading/` | 线程池接口 |
| `src/ecs/` | ECS实现 |
| `src/threading/` | 线程管理实现 |
| `src/io/` | 文件系统实现 |
| `src/os/ohos/` | OpenHarmony平台实现 |

**关键接口**:
- `IEngine` - 引擎主接口
- `IECS` - ECS系统接口
- `IEntityManager` - 实体管理
- `IComponentManager` - 组件管理
- `ISystem` - 系统接口
- `IPlugin` - 插件接口
- `IThreadPool` - 线程池
- `IFileSystem` - 文件系统抽象

**产物**:
- `libAGPEngine.a` - 引擎静态库
- `AGPEngineApi` - 引擎API接口目标
- `AGPBaseApi` - 基础API接口目标
- `AGPEcshelperApi` - ECS助手API目标

#### 3.3 LumeRender - 渲染后端
**路径**: `lume/LumeRender/`

**职责**: 渲染管线、GPU资源管理、Shader管理、多后端抽象。

| 子目录 | 说明 |
|--------|------|
| `api/render/` | 渲染API定义 |
| `api/render/device/` | GPU设备抽象 |
| `api/render/nodecontext/` | 渲染节点上下文 |
| `api/render/datastore/` | 渲染数据存储 |
| `api/render/vulkan/` | Vulkan特定接口 |
| `api/render/gles/` | GLES特定接口 |

**关键接口**:
- `IRenderContext` - 渲染上下文
- `IRenderer` - 渲染器
- `IDevice` - GPU设备抽象
- `IGpuResourceManager` - GPU资源管理
- `IRenderNode` - 渲染节点
- `IRenderDataStore` - 渲染数据存储

**产物**:
- `libPluginAGPRender.z.so` - 渲染插件
- `AGPRenderApi` - 渲染API接口目标

#### 3.4 Lume_3D - 3D功能实现
**路径**: `lume/Lume_3D/`

**职责**: 3D渲染功能、ECS组件实现、GLTF加载、动画系统。

| 子目录 | 说明 |
|--------|------|
| `api/3d/ecs/components/` | 3D ECS组件定义 |
| `api/3d/ecs/systems/` | 3D ECS系统定义 |
| `api/3d/gltf/` | GLTF接口 |
| `api/3d/shaders/common/` | Shader头文件 |
| `src/ecs/components/` | 组件实现 |
| `src/ecs/systems/` | 系统实现 |
| `src/gltf/` | GLTF解析实现 |

**关键组件**:
- `MeshComponent` - 网格组件
- `MaterialComponent` - 材质组件
- `TransformComponent` - 变换组件
- `CameraComponent` - 相机组件
- `LightComponent` - 灯光组件
- `AnimationComponent` - 动画组件
- `PostProcessComponent` - 后处理组件

**关键系统**:
- `IRenderSystem` - 渲染系统
- `IAnimationSystem` - 动画系统
- `ISkinningSystem` - 蒙皮系统
- `IMorphingSystem` - 形变系统

**产物**:
- `libAGP3D.a` - 3D功能静态库
- `libPluginAGP3D.z.so` - 3D功能插件
- `AGP3DApi` - 3D API接口目标

#### 3.5 功能插件

| 插件 | 路径 | 功能 | 产物 |
|------|------|------|------|
| LumeFont | `lume/LumeFont/` | 字体渲染 | `libPluginFont.z.so` |
| LumePng | `lume/LumePng/` | PNG图片解码 | `libPluginAGPPng.z.so` |
| LumeJpg | `lume/LumeJpg/` | JPG图片解码 | `libPluginAGPJpg.z.so` |
| Lume_3DText | `lume/Lume_3DText/` | 3D文字渲染 | `libPluginAGP3DText.z.so` |
| LumeScene | `lume/LumeScene/` | 场景管理插件 | `libPluginSceneWidget.z.so` |
| LumeMeta | `lume/LumeMeta/` | 元对象系统 | `libPluginMetaObject.z.so` |
| LumeDotfield | `lume/LumeDotfield/` | 点场渲染 | `libPluginDotfield.z.so` |

#### 3.6 编译工具

| 工具 | 路径 | 功能 |
|------|------|------|
| LumeShaderCompiler | `lume/LumeBinaryCompile/LumeShaderCompiler/` | Shader编译 |
| lumeassetcompiler | `lume/LumeBinaryCompile/lumeassetcompiler/` | 资源编译 |

---

## 模块依赖关系

```
┌──────────────────────────────────────────────────────────────┐
│                        应用层                                 │
│         (JavaScript/ArkTS) → scene.napi / @kit.Graphics3D    │
└───────────────────────┬──────────────────────────────────────┘
                        │
┌───────────────────────▼──────────────────────────────────────┐
│                      Kits 层                                  │
│    ┌──────────────┐          ┌──────────────┐               │
│    │   kits/js    │          │   kits/ets   │               │
│    │  (N-API)     │          │  (Taihe)     │               │
│    └──────┬───────┘          └──────┬───────┘               │
└───────────┼─────────────────────────┼───────────────────────┘
            │                         │
┌───────────▼─────────────────────────▼───────────────────────┐
│                   Adapter 层                                 │
│    ┌──────────────────┐    ┌──────────────────┐            │
│    │ 3d_widget_adapter│    │ 3d_scene_adapter │            │
│    │ (Widget适配)      │    │ (Scene适配)      │            │
│    └─────────┬────────┘    └─────────┬────────┘            │
└──────────────┼──────────────────────┼──────────────────────┘
               │                      │
┌──────────────▼──────────────────────▼──────────────────────┐
│                   Engine Core (Lume)                         │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐              │
│  │ LumeEngine │ │  Lume_3D   │ │ LumeRender │              │
│  │ (ECS/资源) │ │ (3D功能)   │ │ (渲染后端) │              │
│  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘              │
│  ┌─────┴──────┐ ┌─────┴──────┐ ┌─────┴──────┐              │
│  │ LumeBase   │ │   插件     │ │ LumeBase   │              │
│  │ (基础库)   │ │(Png/Jpg等) │ │ (基础库)   │              │
│  └────────────┘ └────────────┘ └────────────┘              │
└────────────────────────────────────────────────────────────┘
```

## 下一步

- [了解架构设计 →](02_Architecture.md)
- [查看N-API接口 →](03_NAPI_Reference.md)
- [查看GN构建配置 →](05_GN_Build.md)
