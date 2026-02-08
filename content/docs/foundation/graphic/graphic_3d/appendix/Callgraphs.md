# 附录：关键调用链

## 1. 场景加载调用链

```mermaid
sequenceDiagram
    participant JS as JS: Scene.load(uri)
    participant NAPI as SceneJS::Load()
    participant Adapter as SceneAdapter
    participant Engine as AGP Engine
    participant GLTF as GLTF2Importer
    participant ECS as ECS Framework
    
    JS->>NAPI: 调用 load 静态方法
    NAPI->>NAPI: FunctionContext参数校验
    NAPI->>Adapter: LoadPluginsAndInit()
    Adapter->>Engine: CreateGraphicsContext()
    Engine->>Engine: 初始化ECS
    Engine->>Engine: 加载插件
    NAPI->>Adapter: LoadScene(uri)
    Adapter->>GLTF: Parse(uri)
    GLTF->>GLTF: 解析GLTF/GLB文件头
    GLTF->>GLTF: 加载Buffers/Images
    GLTF->>ECS: 创建Entity
    GLTF->>ECS: 添加Components
    ECS->>ECS: 初始化Systems
    ECS-->>Adapter: 返回Scene Handle
    Adapter-->>NAPI: 返回SceneJS对象
    NAPI-->>JS: 返回JS Scene对象
```

**关键代码路径**:
1. `kits/js/src/SceneJS.cpp:180-186` - SceneJS::Load()
2. `3d_scene_adapter/src/scene_adapter.cpp` - SceneAdapter::LoadScene()
3. `lume/Lume_3D/src/gltf/gltf2_importer.cpp` - GLTF2Importer::Import()
4. `lume/LumeEngine/src/ecs/ecs.cpp` - ECS初始化

---

## 2. 渲染帧调用链

```mermaid
sequenceDiagram
    participant RS as RenderService
    participant Adapter as WidgetAdapter
    participant Engine as AGP Engine
    participant ECS as ECS Systems
    participant Render as RenderSystem
    participant GPU as GPU Backend
    
    loop VSync
        RS->>Adapter: OnVSync()
        Adapter->>Engine: RenderFrame()
        Engine->>ECS: Update Systems
        ECS->>ECS: AnimationSystem.Update()
        ECS->>ECS: NodeSystem.Update()
        ECS->>Render: RenderSystem.Update()
        Render->>Render: Build RenderNodes
        Render->>Render: Update RenderDataStores
        Render->>GPU: ExecuteFrame(cmdList)
        GPU->>GPU: Submit GPU Commands
        GPU-->>Adapter: Frame Complete
        Adapter-->>RS: Present
    end
```

**关键代码路径**:
1. `3d_widget_adapter/src/widget_adapter.cpp` - WidgetAdapter渲染入口
2. `lume/LumeEngine/src/ecs/ecs.cpp` - ECS系统更新
3. `lume/Lume_3D/src/ecs/systems/render_system.cpp` - 渲染系统
4. `lume/LumeRender/src/renderer.cpp` - 渲染器执行
5. `lume/LumeRender/src/gles/render_backend_gles.cpp` 或 `vulkan/render_backend_vk.cpp` - 后端提交

---

## 3. N-API调用链示例

### createCamera Promise调用链

```mermaid
sequenceDiagram
    participant JS as JS: scene.createCamera(opts)
    participant NAPI as SceneJS::CreateCamera()
    participant Promise as Promise类
    participant Async as napi_async_work
    participant Factory as RenderResourceFactory
    participant ECS as ECS
    
    JS->>NAPI: 调用createCamera
    NAPI->>Promise: new Promise(env, worker)
    Promise->>Async: napi_create_async_work
    Async->>Factory: 在工作线程执行
    Factory->>ECS: CreateEntity()
    Factory->>ECS: AddComponent<CameraComponent>()
    ECS-->>Factory: 返回Entity
    Factory-->>Async: 返回Camera句柄
    Async->>Promise: complete回调
    Promise->>NAPI: napi_resolve_deferred
    NAPI->>NAPI: CameraJS对象包装
    NAPI-->>JS: Promise.resolve(camera)
```

**关键代码路径**:
1. `kits/js/src/SceneJS.cpp` - SceneJS::CreateCamera()
2. `kits/js/src/Promise.cpp` - Promise异步实现
3. `lume/Lume_3D/src/util/intf_scene_util.h` - 场景工具接口
4. `lume/LumeEngine/src/ecs/entity_manager.cpp` - 实体创建

---

## 4. 插件加载调用链

```mermaid
sequenceDiagram
    participant Engine as AGP Engine
    participant PluginReg as PluginRegistry
    participant Lib as LibraryOHOS
    participant DL as dlopen/dlsym
    participant SO as Plugin .so
    participant Factory as IClassFactory
    
    Engine->>PluginReg: LoadPlugins()
    PluginReg->>PluginReg: Scan plugin directory
    loop For each plugin
        PluginReg->>Lib: LibraryOHOS(filename)
        Lib->>DL: dlopen(path)
        DL->>SO: 加载共享库
        Lib->>DL: dlsym("gPluginData")
        DL->>SO: 获取插件入口
        SO-->>Lib: 返回IPlugin*
        Lib->>Lib: 注册接口
        PluginReg->>Factory: RegisterClass()
    end
    PluginReg-->>Engine: 加载完成
```

**关键代码路径**:
1. `lume/LumeEngine/src/plugin_registry.cpp:648-654` - 插件加载
2. `lume/LumeEngine/src/os/ohos/library_ohos.cpp:28-31` - dlopen
3. `lume/LumeEngine/api/core/plugin/intf_plugin_register.h` - 注册接口

---

## 5. GLTF解析调用链

```mermaid
sequenceDiagram
    participant Importer as GLTF2Importer
    participant Loader as GLTF2Loader
    participant Parser as GLTF2Parser
    participant ECS as ECS Framework
    participant ResMgr as ResourceManager
    
    Importer->>Loader: LoadGLTF(uri)
    Loader->>Loader: 读取文件
    Loader->>Parser: ParseJSON/GLB()
    Parser->>Parser: 解析scene/node/mesh
    loop For each node
        Parser->>ECS: CreateEntity()
        Parser->>ECS: AddComponent<NodeComponent>()
        Parser->>ECS: AddComponent<TransformComponent>()
        opt Has Mesh
            Parser->>ECS: AddComponent<MeshComponent>()
            Parser->>ResMgr: CreateGpuBuffer()
        end
        opt Has Material
            Parser->>ECS: AddComponent<MaterialComponent>()
            Parser->>ResMgr: CreateMaterial()
        end
    end
    Parser-->>Loader: 返回解析结果
    Loader-->>Importer: 返回Scene
```

**关键代码路径**:
1. `lume/Lume_3D/src/gltf/gltf2_importer.cpp` - GLTF导入器
2. `lume/Lume_3D/src/gltf/gltf2_loader.cpp` - GLTF加载器
3. `lume/Lume_3D/src/gltf/gltf2.cpp` - GLTF解析器
4. `lume/LumeEngine/src/ecs/entity_manager.cpp` - 实体管理

---

## 6. 资源加载调用链

```mermaid
sequenceDiagram
    participant App as Application
    participant NAPI as ImageJS
    participant Adapter as SceneAdapter
    participant ResMgr as ResourceManager
    participant Loader as ImageLoaderManager
    participant Cache as ResourceCache
    participant FS as FileSystem
    
    App->>NAPI: scene.createImage(uri)
    NAPI->>Adapter: CreateImage(uri)
    Adapter->>ResMgr: GetResource(uri)
    ResMgr->>Cache: Find in cache
    alt Cache miss
        ResMgr->>Loader: LoadImage(uri)
        Loader->>FS: OpenFile(uri)
        FS->>FS: Read file
        FS-->>Loader: Return data
        Loader->>Loader: Decode image
        Loader-->>ResMgr: Return image
        ResMgr->>Cache: Add to cache
    end
    Cache-->>ResMgr: Return cached resource
    ResMgr-->>Adapter: Return Image handle
    Adapter-->>NAPI: Return ImageJS
    NAPI-->>App: Promise<Image>
```

**关键代码路径**:
1. `lume/LumeEngine/api/core/resources/intf_resource_manager.h` - 资源管理器接口
2. `lume/LumeEngine/src/image/image_loader_manager.cpp` - 图片加载管理
3. `lume/LumeEngine/src/io/file_manager.cpp` - 文件管理

---

## 7. 内存分配调用链

```mermaid
sequenceDiagram
    participant Container as vector/string
    participant Allocator as BASE_NS::allocator
    participant Malloc as malloc/calloc
    participant Memory as Heap Memory
    
    Container->>Allocator: alloc(size)
    Allocator->>Malloc: malloc(size)
    Malloc->>Memory: 分配内存
    Memory-->>Malloc: 返回指针
    Malloc-->>Allocator: 返回ptr
    Allocator-->>Container: 返回ptr
    Note over Container: 使用内存...
    Container->>Allocator: free(ptr)
    Allocator->>Malloc: free(ptr)
    Malloc->>Memory: 释放内存
```

**关键代码路径**:
1. `lume/LumeBase/api/base/containers/vector.h` - 容器实现
2. `lume/LumeBase/api/base/containers/allocator.h` - 分配器

---

## 8. 动画更新调用链

```mermaid
sequenceDiagram
    participant System as AnimationSystem
    participant Comp as AnimationComponent
    participant Track as AnimationTrackComponent
    participant Output as AnimationOutputComponent
    participant Target as Target Component
    
    loop Update Frame
        System->>System: Get current time
        System->>Comp: For each animation
        Comp->>Comp: Calculate normalized time
        Comp->>Track: Sample keyframes
        Track->>Track: Interpolate values
        Track-->>Output: Write output values
        Output->>Target: Apply to target component
    end
```

**关键代码路径**:
1. `lume/Lume_3D/api/3d/ecs/systems/intf_animation_system.h` - 动画系统接口
2. `lume/Lume_3D/api/3d/ecs/components/animation_component.h` - 动画组件
3. `lume/Lume_3D/api/3d/ecs/components/animation_track_component.h` - 动画轨道

---

## 相关文档

- [架构设计 →](../02_Architecture.md)
- [N-API接口 →](../03_NAPI_Reference.md)
- [内部API →](../04_Internal_API.md)
