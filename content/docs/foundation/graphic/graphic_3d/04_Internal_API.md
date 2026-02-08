# 内部API

## 概述

本文档描述AGP引擎内部模块接口、依赖关系和稳定性。

## 模块接口分层

```
┌─────────────────────────────────────────────────────────────┐
│                     Public API (稳定)                        │
│     lume/*/api/ 目录下的接口，保证向后兼容                     │
├─────────────────────────────────────────────────────────────┤
│                   Internal API (不稳定)                      │
│        lume/*/src/ 内部实现，可能随时变更                      │
├─────────────────────────────────────────────────────────────┤
│                   Private API (内部使用)                     │
│            模块私有实现，不对外暴露                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Public API清单

### LumeBase API

**位置**: `lume/LumeBase/api/`

| 头文件 | 接口 | 稳定性 |
|--------|------|--------|
| `base/math/vector.h` | Vector2/3/4 | 稳定 |
| `base/math/matrix.h` | Matrix3X3, Matrix4X4 | 稳定 |
| `base/math/quaternion.h` | Quaternion | 稳定 |
| `base/containers/vector.h` | vector | 稳定 |
| `base/containers/string.h` | string | 稳定 |

### LumeEngine API

**位置**: `lume/LumeEngine/api/`

#### ECS接口

| 头文件 | 接口 | 稳定性 | 说明 |
|--------|------|--------|------|
| `core/ecs/intf_ecs.h` | `IEcs` | 稳定 | ECS主接口 |
| `core/ecs/intf_entity_manager.h` | `IEntityManager` | 稳定 | 实体管理 |
| `core/ecs/intf_component_manager.h` | `IComponentManager` | 稳定 | 组件管理 |
| `core/ecs/intf_system.h` | `ISystem` | 稳定 | 系统接口 |
| `core/ecs/entity.h` | `Entity` | 稳定 | 实体标识符 |

#### 插件接口

| 头文件 | 接口 | 稳定性 | 说明 |
|--------|------|--------|------|
| `core/plugin/intf_plugin.h` | `IPlugin` | 稳定 | 插件定义 |
| `core/plugin/intf_plugin_register.h` | `IPluginRegister` | 稳定 | 插件注册器 |
| `core/plugin/intf_class_factory.h` | `IClassFactory` | 稳定 | 类工厂 |
| `core/plugin/intf_interface.h` | `IInterface` | 稳定 | 接口基类 |

#### 资源接口

| 头文件 | 接口 | 稳定性 | 说明 |
|--------|------|--------|------|
| `core/resources/intf_resource_manager.h` | `IResourceManager` | 稳定 | 资源管理器 |
| `core/resources/intf_resource.h` | `IResource` | 稳定 | 资源基类 |

#### IO接口

| 头文件 | 接口 | 稳定性 | 说明 |
|--------|------|--------|------|
| `core/io/intf_file_manager.h` | `IFileManager` | 稳定 | 文件管理器 |
| `core/io/intf_filesystem_api.h` | `IFilesystemApi` | 稳定 | 文件系统API |
| `core/io/intf_file.h` | `IFile` | 稳定 | 文件接口 |

#### 线程接口

| 头文件 | 接口 | 稳定性 | 说明 |
|--------|------|--------|------|
| `core/threading/intf_thread_pool.h` | `IThreadPool` | 稳定 | 线程池 |

### LumeRender API

**位置**: `lume/LumeRender/api/`

| 头文件 | 接口 | 稳定性 | 说明 |
|--------|------|--------|------|
| `render/intf_render_context.h` | `IRenderContext` | 稳定 | 渲染上下文 |
| `render/intf_renderer.h` | `IRenderer` | 稳定 | 渲染器 |
| `render/nodecontext/intf_render_node.h` | `IRenderNode` | 稳定 | 渲染节点 |
| `render/nodecontext/intf_render_node_graph_manager.h` | `IRenderNodeGraphManager` | 稳定 | 节点图管理 |
| `render/datastore/intf_render_data_store.h` | `IRenderDataStore` | 稳定 | 渲染数据存储 |
| `render/device/intf_gpu_resource_manager.h` | `IGpuResourceManager` | 稳定 | GPU资源管理 |
| `render/device/intf_device.h` | `IDevice` | 稳定 | 设备抽象 |

### Lume_3D API

**位置**: `lume/Lume_3D/api/`

| 头文件 | 接口 | 稳定性 | 说明 |
|--------|------|--------|------|
| `3d/intf_graphics_context.h` | `IGraphicsContext` | 稳定 | 3D图形上下文 |
| `3d/util/intf_scene_util.h` | `ISceneUtil` | 稳定 | 场景工具 |
| `3d/util/intf_mesh_util.h` | `IMeshUtil` | 稳定 | 网格工具 |
| `3d/util/intf_picking.h` | `IPicking` | 稳定 | 拾取接口 |
| `3d/loaders/intf_scene_loader.h` | `ISceneLoader` | 稳定 | 场景加载器 |

### ECS组件API

**位置**: `lume/Lume_3D/api/3d/ecs/components/`

所有组件定义头文件均为**稳定接口**：

| 组件 | 头文件 |
|------|--------|
| NodeComponent | `node_component.h` |
| TransformComponent | `transform_component.h` |
| MeshComponent | `mesh_component.h` |
| MaterialComponent | `material_component.h` |
| CameraComponent | `camera_component.h` |
| LightComponent | `light_component.h` |
| SkinComponent | `skin_component.h` |
| AnimationComponent | `animation_component.h` |
| EnvironmentComponent | `environment_component.h` |
| PostProcessComponent | `post_process_component.h` |

---

## 模块依赖关系

### 依赖图

```
                    ┌─────────────┐
                    │   Kits      │
                    │(JS/ETS API) │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ LumeScene   │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌─────▼─────┐      ┌────▼────┐
   │Lume_3D  │       │LumeRender │      │LumeMeta │
   └────┬────┘       └─────┬─────┘      └────┬────┘
        │                  │                 │
        └──────────────────┼─────────────────┘
                           │
                    ┌──────▼──────┐
                    │ LumeEngine  │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  LumeBase   │
                    └─────────────┘
```

### 依赖方向

| 模块 | 依赖 | 说明 |
|------|------|------|
| LumeBase | 无 | 基础库，不依赖其他模块 |
| LumeEngine | LumeBase | 使用基础数学和容器 |
| LumeRender | LumeBase, LumeEngine | 使用ECS和资源管理 |
| Lume_3D | LumeBase, LumeEngine, LumeRender | 完整的3D功能 |
| LumeScene | LumeBase, LumeEngine, Lume_3D, LumeRender | 场景管理 |
| Kits | 所有底层模块 | 封装Native API |

---

## 接口稳定性标注

### 稳定性等级

| 等级 | 标识 | 说明 | 变更策略 |
|------|------|------|----------|
| **稳定** | `STABLE` | 公开API，向后兼容 | 主版本变更时可 breaking |
| **实验性** | `EXPERIMENTAL` | 预览API，可能变更 | 次版本变更时可 breaking |
| **内部** | `INTERNAL` | 内部使用，不保证 | 随时可能变更 |
| **弃用** | `DEPRECATED` | 已弃用，将被移除 | 下个主版本移除 |

### 当前稳定性状态

#### 稳定接口

- 所有 `api/` 目录下的接口
- ECS核心接口 (`IEcs`, `IEntityManager`, `IComponentManager`, `ISystem`)
- 插件接口 (`IPlugin`, `IPluginRegister`, `IClassFactory`)
- 资源接口 (`IResourceManager`, `IResource`)
- 渲染接口 (`IRenderContext`, `IRenderNode`, `IGpuResourceManager`)
- 数学库接口 (Vector, Matrix, Quaternion)

#### 内部接口（不稳定）

- 具体实现类（`src/` 目录）
- 平台相关代码 (`os/ohos/`, `os/android/`)
- 渲染后端具体实现 (`gles/`, `vulkan/`)
- GLTF解析器内部细节

---

## 可替换点

### 可替换组件

| 组件 | 接口 | 替换方式 |
|------|------|----------|
| **文件系统** | `IFileManager` | 实现自定义文件系统 |
| **渲染后端** | `IDevice` | 实现新的图形API后端 |
| **资源加载器** | `IResourceLoader` | 添加新的资源格式支持 |
| **渲染节点** | `IRenderNode` | 添加自定义渲染效果 |
| **ECS系统** | `ISystem` | 添加自定义游戏逻辑系统 |

### 替换示例：自定义文件系统

```cpp
#include <core/io/intf_file_manager.h>

class MyCustomFileManager : public IFilesystemApi {
public:
    // 实现 IFileManager 接口
    IDirectory::Ptr OpenDirectory(const string_view path) override {
        // 自定义目录打开逻辑
    }
    
    IFile::Ptr OpenFile(const string_view path, 
                        FileOpenMode mode) override {
        // 自定义文件打开逻辑
    }
};

// 注册到引擎
auto filesystem = IFilesystemApi::Ptr(new MyCustomFileManager());
engine->RegisterFilesystem(filesystem);
```

### 替换示例：自定义渲染节点

```cpp
#include <render/nodecontext/intf_render_node.h>

class MyCustomEffectNode : public IRenderNode {
public:
    void InitNode(IRenderNodeContextManager& contextMgr) override {
        // 初始化节点
    }
    
    void ExecuteFrame(IRenderCommandList& cmdList) override {
        // 执行自定义渲染逻辑
    }
};

// 注册到渲染管线
auto node = IRenderNode::Ptr(new MyCustomEffectNode());
renderContext->AddRenderNode(node);
```

---

## 下一步

- [查看架构设计 →](02_Architecture.md)
- [查看N-API接口 →](03_NAPI_Reference.md)
- [查看GN构建 →](05_GN_Build.md)
