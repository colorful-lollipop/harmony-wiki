# AGP引擎 - 首页概览

## 项目定位

**AGP (Ark Graphics Platform)** 是 OpenHarmony 操作系统内置的跨平台高性能实时3D渲染引擎，为应用开发者提供声明式的3D图形编程能力。

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 应用层                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   ArkTS/JS  │  │   ArkUI-X   │  │   原生应用 (C++)    │  │
│  │   应用代码   │  │   跨平台    │  │                     │  │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
└─────────┼────────────────┼────────────────────┼─────────────┘
          │                │                    │
          ▼                ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                      AGP 引擎 (本仓库)                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  JS/ETS API │  │  ECS框架    │  │   渲染后端          │  │
│  │  (Kits层)   │  │  (Lume_3D)  │  │  (LumeRender)       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ 场景适配器  │  │  资源管理   │  │   插件系统          │  │
│  │(Scene/Widget│  │(LumeEngine) │  │                     │  │
│  │   Adapter)  │  │             │  │                     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
          │                │                    │
          ▼                ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                      系统服务层                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  graphic_2d │  │   Vulkan    │  │    OpenGL ES        │  │
│  │  (合成显示) │  │   驱动      │  │    驱动             │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力

### 1. 3D渲染能力
| 能力 | 说明 | 关键文件 |
|------|------|----------|
| **GLTF模型加载** | 支持GLTF 2.0标准格式 | `lume/Lume_3D/src/gltf/` |
| **PBR材质系统** | 基于物理的渲染材质 | `lume/Lume_3D/api/3d/ecs/components/material_component.h` |
| **光照系统** | 定向光/点光源/聚光源 | `lume/Lume_3D/api/3d/ecs/components/light_component.h` |
| **后处理特效** | Bloom/HDR/ToneMapping/FXAA | `lume/Lume_3D/api/3d/ecs/components/post_process_component.h` |
| **动画系统** | 骨骼动画/形变动画 | `lume/Lume_3D/api/3d/ecs/systems/intf_animation_system.h` |
| **粒子系统** | 基础粒子效果 | `lume/LumeDotfield/` |

### 2. ECS架构特性
```cpp
// 伪代码示例
Entity entity = ecs.CreateEntity();
ecs.AddComponent<TransformComponent>(entity, position, rotation, scale);
ecs.AddComponent<MeshComponent>(entity, meshHandle);
ecs.AddComponent<MaterialComponent>(entity, materialHandle);
```

**核心概念**:
- **Entity**: 轻量级标识符，代表游戏对象
- **Component**: 纯数据结构，存储属性
- **System**: 逻辑处理器，遍历处理Component

### 3. 多后端渲染
| 后端 | 状态 | 配置文件 |
|------|------|----------|
| OpenGL ES 3.0+ | ✅ 主要支持 | `kits/*/BUILD.gn` 中 `CORE_BUILD_GLES=1` |
| Vulkan 1.0+ | ✅ 支持 | `kits/*/BUILD.gn` 中 `CORE_BUILD_VULKAN=1` |
| OpenGL (Desktop) | ❌ 禁用 | `CORE_BUILD_GL=0` |

## 运行环境

### 系统要求
- **操作系统**: OpenHarmony 3.1+ (standard系统)
- **硬件**: 支持OpenGL ES 3.0+ 或 Vulkan 的移动/嵌入式GPU
- **内存**: 最低8MB ROM + 8MB RAM (bundle.json声明)

### 依赖组件
```json
// 来自 bundle.json
deps: {
  "c_utils",          // 基础工具库
  "hilog",            // 日志系统
  "graphic_2d",       // 2D图形/合成
  "graphic_surface",  // 图形缓冲区
  "hitrace",          // 性能追踪
  "ipc",              // IPC通信
  "napi",             // N-API运行时
  "ability_runtime",  // Ability框架
  "vulkan-loader",    // Vulkan加载器
  "skia",             // Skia图形库
  "meshoptimizer"     // 网格优化
}
```

## 关键概念

### 1. Scene (场景)
3D世界的顶层容器，包含所有Entity、灯光、相机。
```cpp
// 来自 kits/js/include/SceneJS.h
class SceneJS : public BaseObjectJS {
    // 管理场景内所有3D对象
};
```

### 2. Node (节点)
场景中的基本对象，具有变换矩阵，可形成层级结构。
```cpp
// 来自 lume/Lume_3D/api/3d/ecs/components/node_component.h
struct NodeComponent {
    Entity parent;
    vector<Entity> children;
};
```

### 3. Mesh (网格)
3D几何数据，包含顶点、索引、子网格。
```cpp
// 来自 lume/Lume_3D/api/3d/ecs/components/mesh_component.h
struct MeshComponent {
    RenderHandleReference mesh;
    vector<RenderHandleReference> subMeshes;
};
```

### 4. Material (材质)
定义表面外观属性，支持PBR工作流。
```cpp
// 来自 lume/Lume_3D/api/3d/ecs/components/material_component.h
struct MaterialComponent {
    RenderHandleReference material;
    MaterialType type;  // PBR, Unlit, etc.
};
```

### 5. Plugin (插件)
引擎功能扩展机制，动态加载的模块。
```cpp
// 来自 lume/LumeEngine/api/core/plugin/intf_plugin.h
class IPlugin : public IInterface {
    virtual string_view GetName() const = 0;
    virtual uint32_t GetVersion() const = 0;
};
```

## 代码规模

| 指标 | 数值 | 备注 |
|------|------|------|
| C++源文件 | ~1000+ | 排除测试目录 |
| API头文件 | ~200+ | `api/` 和 `include/` 目录 |
| GN构建文件 | 34 | BUILD.gn 文件 |
| JS/ETS绑定 | ~60+ | kits/js/src, kits/ets/src |
| Shader文件 | ~50+ | `.h` 格式的GLSL源码 |

## 快速开始

### 应用开发者（JS/ArkTS）
```javascript
// 导入3D场景模块
import { Scene, Node, Mesh, Material } from '@kit.Graphics3D';

// 创建场景
const scene = new Scene();

// 创建节点
const node = new Node();
node.position = { x: 0, y: 0, z: 0 };
scene.addNode(node);
```

### 系统开发者（C++）
```cpp
// 包含引擎API
#include <3d/intf_graphics_context.h>
#include <core/intf_engine.h>

// 初始化引擎
auto engine = CreateEngine(engineCreateInfo);
auto graphicsContext = engine->CreateGraphicsContext();
```

## 下一步

- [了解目录结构 →](01_Directory_Structure.md)
- [深入架构设计 →](02_Architecture.md)
- [查看API文档 →](03_NAPI_Reference.md)
