# 附录：关键符号索引

## 按模块索引

### LumeBase (基础库)

| 符号 | 类型 | 文件 | 说明 |
|------|------|------|------|
| `BASE_NS::vector` | 类模板 | `api/base/containers/vector.h` | 动态数组 |
| `BASE_NS::string` | 类 | `api/base/containers/string.h` | 字符串 |
| `BASE_NS::Math::Vec3` | 结构体 | `api/base/math/vector.h` | 3D向量 |
| `BASE_NS::Math::Mat4X4` | 结构体 | `api/base/math/matrix.h` | 4x4矩阵 |
| `BASE_NS::Math::Quat` | 结构体 | `api/base/math/quaternion.h` | 四元数 |
| `BASE_NS::uid` | 类 | `api/base/util/uid.h` | 唯一标识符 |

### LumeEngine (引擎框架)

| 符号 | 类型 | 文件 | 说明 |
|------|------|------|------|
| `IEcs` | 接口 | `api/core/ecs/intf_ecs.h` | ECS主接口 |
| `IEntityManager` | 接口 | `api/core/ecs/intf_entity_manager.h` | 实体管理器 |
| `IComponentManager` | 接口 | `api/core/ecs/intf_component_manager.h` | 组件管理器 |
| `ISystem` | 接口 | `api/core/ecs/intf_system.h` | 系统接口 |
| `Entity` | 结构体 | `api/core/ecs/entity.h` | 实体标识符 |
| `IPlugin` | 接口 | `api/core/plugin/intf_plugin.h` | 插件接口 |
| `IPluginRegister` | 接口 | `api/core/plugin/intf_plugin_register.h` | 插件注册器 |
| `IClassFactory` | 接口 | `api/core/plugin/intf_class_factory.h` | 类工厂 |
| `IInterface` | 接口 | `api/core/plugin/intf_interface.h` | 接口基类 |
| `IResourceManager` | 接口 | `api/core/resources/intf_resource_manager.h` | 资源管理器 |
| `IFileManager` | 接口 | `api/core/io/intf_file_manager.h` | 文件管理器 |
| `IThreadPool` | 接口 | `api/core/threading/intf_thread_pool.h` | 线程池 |

### LumeRender (渲染后端)

| 符号 | 类型 | 文件 | 说明 |
|------|------|------|------|
| `IRenderContext` | 接口 | `api/render/intf_render_context.h` | 渲染上下文 |
| `IRenderer` | 接口 | `api/render/intf_renderer.h` | 渲染器 |
| `IRenderNode` | 接口 | `api/render/nodecontext/intf_render_node.h` | 渲染节点 |
| `IRenderNodeGraphManager` | 接口 | `api/render/nodecontext/intf_render_node_graph_manager.h` | 节点图管理 |
| `IRenderDataStore` | 接口 | `api/render/datastore/intf_render_data_store.h` | 渲染数据存储 |
| `IGpuResourceManager` | 接口 | `api/render/device/intf_gpu_resource_manager.h` | GPU资源管理 |
| `IDevice` | 接口 | `api/render/device/intf_device.h` | 设备抽象 |
| `RenderHandle` | 类型别名 | `api/render/resource_handle.h` | 资源句柄 |

### Lume_3D (3D功能)

| 符号 | 类型 | 文件 | 说明 |
|------|------|------|------|
| `IGraphicsContext` | 接口 | `api/3d/intf_graphics_context.h` | 3D图形上下文 |
| `ISceneUtil` | 接口 | `api/3d/util/intf_scene_util.h` | 场景工具 |
| `IMeshUtil` | 接口 | `api/3d/util/intf_mesh_util.h` | 网格工具 |
| `IPicking` | 接口 | `api/3d/util/intf_picking.h` | 拾取接口 |
| `GLTF2` | 命名空间 | `api/3d/gltf/gltf.h` | GLTF解析 |
| `NodeComponent` | 结构体 | `api/3d/ecs/components/node_component.h` | 节点组件 |
| `TransformComponent` | 结构体 | `api/3d/ecs/components/transform_component.h` | 变换组件 |
| `MeshComponent` | 结构体 | `api/3d/ecs/components/mesh_component.h` | 网格组件 |
| `MaterialComponent` | 结构体 | `api/3d/ecs/components/material_component.h` | 材质组件 |
| `CameraComponent` | 结构体 | `api/3d/ecs/components/camera_component.h` | 相机组件 |
| `LightComponent` | 结构体 | `api/3d/ecs/components/light_component.h` | 灯光组件 |
| `SkinComponent` | 结构体 | `api/3d/ecs/components/skin_component.h` | 蒙皮组件 |
| `AnimationComponent` | 结构体 | `api/3d/ecs/components/animation_component.h` | 动画组件 |

### Kits (N-API)

| 符号 | 类型 | 文件 | 说明 |
|------|------|------|------|
| `SceneJS` | 类 | `src/SceneJS.cpp` | Scene JS绑定 |
| `NodeJS` | 类 | `src/NodeJS.cpp` | Node JS绑定 |
| `CameraJS` | 类 | `src/CameraJS.cpp` | Camera JS绑定 |
| `MaterialJS` | 类 | `src/MaterialJS.cpp` | Material JS绑定 |
| `MeshJS` | 类 | `src/MeshJS.cpp` | Mesh JS绑定 |
| `AnimationJS` | 类 | `src/AnimationJS.cpp` | Animation JS绑定 |
| `NapiApi::FunctionContext` | 类模板 | `include/napi/function_context.h` | 函数上下文 |
| `Promise` | 类 | `src/Promise.cpp` | Promise异步实现 |

---

## 按类别索引

### ECS相关

| 符号 | 说明 |
|------|------|
| `IEcs` | ECS主接口 |
| `IEntityManager` | 实体管理器 |
| `IComponentManager` | 组件管理器 |
| `ISystem` | 系统接口 |
| `Entity` | 实体标识符 (id + version) |
| `ComponentManagerTypeInfo` | 组件管理器类型信息 |
| `SystemTypeInfo` | 系统类型信息 |

### 插件相关

| 符号 | 说明 |
|------|------|
| `IPlugin` | 插件定义接口 |
| `IEnginePlugin` | 引擎插件接口 |
| `IEcsPlugin` | ECS插件接口 |
| `IPluginRegister` | 插件注册器 |
| `IClassFactory` | 类工厂 |
| `IInterface` | 接口基类 (引用计数) |
| `IInterfaceHelper` | 接口辅助 |
| `GetPluginRegister()` | 获取全局插件注册器 |

### 渲染相关

| 符号 | 说明 |
|------|------|
| `IRenderContext` | 渲染上下文 |
| `IRenderer` | 渲染器 |
| `IRenderNode` | 渲染节点基类 |
| `IRenderDataStore` | 渲染数据存储 |
| `IGpuResourceManager` | GPU资源管理器 |
| `IDevice` | GPU设备抽象 |
| `IShaderManager` | Shader管理器 |
| `RenderHandle` | 渲染资源句柄 |
| `GpuBufferDesc` | GPU缓冲区描述 |
| `GpuImageDesc` | GPU图像描述 |

### 资源相关

| 符号 | 说明 |
|------|------|
| `IResourceManager` | 资源管理器 |
| `IResource` | 资源基类 |
| `IResourceType` | 资源类型 |
| `ResourceId` | 资源ID |
| `IFileManager` | 文件管理器 |
| `IFile` | 文件接口 |
| `IDirectory` | 目录接口 |
| `IImageLoaderManager` | 图片加载管理器 |

### 数学相关

| 符号 | 说明 |
|------|------|
| `Vec2` | 2D向量 |
| `Vec3` | 3D向量 |
| `Vec4` | 4D向量 |
| `Mat3X3` | 3x3矩阵 |
| `Mat4X4` | 4x4矩阵 |
| `Quat` | 四元数 |
| `Math::PI` | 圆周率 |
| `Math::Deg2Rad` | 度转弧度 |
| `Math::Rad2Deg` | 弧度转度 |

---

## 按文件名索引

### A

| 文件 | 关键符号 |
|------|----------|
| `animation_component.h` | `AnimationComponent` |
| `animation_system.h` | `IAnimationSystem` |
| `allocator.h` | `allocator` |

### C

| 文件 | 关键符号 |
|------|----------|
| `camera_component.h` | `CameraComponent` |
| `component_manager.h` | `IComponentManager` |

### E

| 文件 | 关键符号 |
|------|----------|
| `entity.h` | `Entity`, `EntityReference` |
| `entity_manager.h` | `IEntityManager` |
| `ecs.h` | `IEcs` |

### G

| 文件 | 关键符号 |
|------|----------|
| `gltf.h` | `GLTF2` |
| `gpu_resource_manager.h` | `IGpuResourceManager` |

### I

| 文件 | 关键符号 |
|------|----------|
| `intf_class_factory.h` | `IClassFactory` |
| `intf_component_manager.h` | `IComponentManager` |
| `intf_ecs.h` | `IEcs` |
| `intf_entity_manager.h` | `IEntityManager` |
| `intf_graphics_context.h` | `IGraphicsContext` |
| `intf_plugin.h` | `IPlugin`, `IEnginePlugin`, `IEcsPlugin` |
| `intf_plugin_register.h` | `IPluginRegister` |
| `intf_render_context.h` | `IRenderContext` |
| `intf_render_node.h` | `IRenderNode` |
| `intf_resource_manager.h` | `IResourceManager` |
| `intf_system.h` | `ISystem` |

### L

| 文件 | 关键符号 |
|------|----------|
| `light_component.h` | `LightComponent` |

### M

| 文件 | 关键符号 |
|------|----------|
| `material_component.h` | `MaterialComponent` |
| `matrix.h` | `Mat3X3`, `Mat4X4` |
| `mesh_component.h` | `MeshComponent` |

### N

| 文件 | 关键符号 |
|------|----------|
| `node_component.h` | `NodeComponent` |

### P

| 文件 | 关键符号 |
|------|----------|
| `plugin_registry.cpp` | `GetPluginRegister()` |

### Q

| 文件 | 关键符号 |
|------|----------|
| `quaternion.h` | `Quat` |

### R

| 文件 | 关键符号 |
|------|----------|
| `render_node.h` | `IRenderNode` |

### S

| 文件 | 关键符号 |
|------|----------|
| `SceneJS.cpp` | `SceneJS` |
| `skin_component.h` | `SkinComponent` |
| `string.h` | `string` |
| `system.h` | `ISystem` |

### T

| 文件 | 关键符号 |
|------|----------|
| `transform_component.h` | `TransformComponent` |

### V

| 文件 | 关键符号 |
|------|----------|
| `vector.h` | `vector`, `Vec2/3/4` |

---

## 相关文档

- [内部API →](../04_Internal_API.md)
- [架构设计 →](../02_Architecture.md)
