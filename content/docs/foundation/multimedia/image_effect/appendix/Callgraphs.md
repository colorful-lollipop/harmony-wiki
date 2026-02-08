# 关键调用链

## 概述

本文档记录 ImageEffect 模块中的关键调用链，以入口→核心逻辑的形式呈现，便于理解代码流程和进行安全审计。

## ImageEffect 创建流程

```
OH_ImageEffect_Create()
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/capi/image_effect.cpp                    │
│  ImageEffect* OH_ImageEffect_Create()                       │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/effect/base/image_effect_inner.cpp      │
│  ImageEffect* ImageEffect::Create()                        │
│  ├── new ImageEffect()                                     │
│  ├── new EffectContext()                                   │
│  │   ├── EffectMemoryManager::Create()                     │
│  │   │   └── Evidence: effect_memory.cpp:52               │
│  │   └── RenderStrategy::Create()                          │
│  │       └── Evidence: render_strategy.cpp                 │
│  └── return ImageEffect*                                   │
└─────────────────────────────────────────────────────────────┘
```

**证据**：`frameworks/native/capi/image_effect.cpp`（OH_ImageEffect_Create 实现）

## 滤镜添加流程

```
OH_ImageEffect_AddFilter(effect, filterName)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/capi/image_effect.cpp                    │
│  ImageEffect_ErrorCode OH_ImageEffect_AddFilter()           │
│  ├── CHECK_AND_RETURN_RET_LOG (空值检查)                   │
│  │   └── Evidence: image_effect.cpp:65                    │
│  └── AddFilter(filterName)                                 │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/effect/base/image_effect_inner.cpp        │
│  ImageEffect::AddFilter()                                   │
│  ├── EffectContext::GetFilterChain()                        │
│  └── EFilterFactory::Create(filterName)                    │
│      ├── Evidence: efilter_factory.cpp                      │
│      ├── REGISTER_EFILTER_FACTORY 自动注册                   │
│      │   └── Evidence: brightness_efilter.cpp              │
│      └── return EFilter*                                   │
└─────────────────────────────────────────────────────────────┘
```

## 输入设置流程

### PixelMap 输入

```
OH_ImageEffect_SetInputPixelmap(effect, pixelmap)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/capi/image_effect.cpp                    │
│  ImageEffect_ErrorCode OH_ImageEffect_SetInputPixelmap()    │
│  ├── CHECK_AND_RETURN_RET_LOG (空值检查)                   │
│  └── NativeCommonUtils::GetPixelMapFromOHPixelmap()         │
│      └── Evidence: native_common_utils.cpp:191            │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/effect/base/image_effect_inner.cpp       │
│  ImageEffect::SetInputPixelMap()                            │
│  ├── PixelMap::GetImageInfo()                               │
│  └── EffectBuffer::Init(info)                               │
└─────────────────────────────────────────────────────────────┘
```

### NativeBuffer 输入

```
OH_ImageEffect_SetInputNativeBuffer(effect, buffer)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/capi/image_effect.cpp                    │
│  ImageEffect_ErrorCode OH_ImageEffect_SetInputNativeBuffer()│
│  ├── CHECK_AND_RETURN_RET_LOG (空值检查)                   │
│  └── ImageEffect::SetInputSurfaceBuffer()                   │
│      └── Evidence: image_effect.cpp:407                    │
└─────────────────────────────────────────────────────────────┘
```

## 流水线执行流程

```
OH_ImageEffect_Start(effect)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/effect/base/image_effect_inner.cpp       │
│  ImageEffect::Start()                                       │
│  ├── EffectContext::GetRenderStrategy()                     │
│  └── PipelineCore::Render(buffer)                          │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/effect/pipeline/core/pipeline_core.cpp   │
│  PipelineCore::Render()                                     │
│  ├── SourceFilter::Process()                                │
│  │   └── Evidence: image_source_filter.cpp                 │
│  ├── NegotiateCapabilities()                                │
│  │   └── CapabilityNegotiate::Negotiate()                  │
│  │       └── Evidence: capability_negotiate.cpp            │
│  ├── for each filter in filterChain                        │
│  │   └── Filter::Process()                                 │
│  └── SinkFilter::Process()                                 │
│      └── Evidence: image_sink_filter.cpp                   │
└─────────────────────────────────────────────────────────────┘
```

## 滤镜渲染流程

```
EFilter::Render(buffer)
    │
    ├── GPU 路径（如启用）
    │   └── RenderEnvironment::BindEGLContext()
    │       └── Evidence: render_environment.cpp
    │       └── GLES 渲染调用
    │
    └── CPU 路径
        └── EFilter::OnRender(buffer)
            │
            ├── EFilter::ProcessBefore()
            ├── [子滤镜处理]
            └── EFilter::ProcessAfter()
```

## 自定义滤镜回调流程

```
OH_EffectFilter_Register(delegate)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/efilter/custom/filter_delegate.cpp       │
│  FilterDelegate::Register()                                  │
│  ├── delegate_->setValue = user_setValue                    │
│  ├── delegate_->render = user_render                        │
│  ├── delegate_->save = user_save                            │
│  └── delegate_->restore = user_restore                      │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  运行时回调调用                                               │
│  FilterDelegate::InvokeRender(buffer)                       │
│  ├── delegate_->render != nullptr?                          │
│  └── delegate_->render(filter_, buffer)                     │
│      └── Evidence: filter_delegate.cpp                     │
└─────────────────────────────────────────────────────────────┘
```

## 外部扩展加载流程

```
ExternalLoader::LoadExtLibrary()
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/effect/base/external_loader.cpp          │
│  void* ExternalLoader::LoadExtLibrary()                     │
│  ├── dlopen("libimage_effect_ext.so", RTLD_NOW)            │
│  │   └── Evidence: external_loader.cpp:39                 │
│  ├── dlsym(handle, "Init")                                  │
│  │   └── Evidence: external_loader.cpp:49                 │
│  ├── dlsym(handle, "Deinit")                                │
│  ├── dlsym(handle, "InitModule")                            │
│  └── dlsym(handle, "DeinitModule")                          │
└─────────────────────────────────────────────────────────────┘
```

## 内存分配流程

```
EffectMemoryManager::Allocate(size)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/effect/manager/memory_manager/           │
│  effect_memory.cpp                                          │
│  EffectMemory::Allocate()                                   │
│  ├── CHECK_AND_RETURN_RET_LOG (size 检查)                  │
│  │   └── Evidence: effect_memory.cpp:52                  │
│  │       MAX_RAM_SIZE = 256 * 1024 * 1024                  │
│  ├── malloc(size)                                           │
│  │   └── Evidence: effect_memory.cpp:54                   │
│  └── return EffectMemory*                                   │
└─────────────────────────────────────────────────────────────┘
```

## 资源释放流程

```
OH_ImageEffect_Release(effect)
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  frameworks/native/capi/image_effect.cpp                    │
│  ImageEffect_ErrorCode OH_ImageEffect_Release()            │
│  └── ImageEffect::~ImageEffect()                           │
│      ├── delete EffectContext                               │
│      │   ├── delete RenderStrategy                         │
│      │   └── delete EffectMemoryManager                    │
│      └── for each filter in filterChain                    │
│          └── delete EFilter                                 │
└─────────────────────────────────────────────────────────────┘
```

## 调用深度统计

| 调用链 | 最大深度 | 关键路径 |
|--------|----------|----------|
| Create → Context → MemoryManager | 3 | image_effect_inner.cpp |
| AddFilter → Factory | 2 | efilter_factory.cpp |
| Start → Pipeline → Filter | 3 | pipeline_core.cpp |
| Render → GPU/EGL | 2 | render_environment.cpp |
| Register → Callback | 2 | filter_delegate.cpp |

## 相关文档

| 文档 | 说明 |
|------|------|
| [02_Architecture.md](./02_Architecture.md) | 内部架构 |
| [01_N-API_Reference.md](./01_N-API_Reference.md) | N-API 接口 |
| [04_Security_Review.md](./04_Security_Review.md) | 安全评估 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 配置宏与 Feature Flags |
