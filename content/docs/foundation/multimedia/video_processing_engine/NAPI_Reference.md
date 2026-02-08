# VPE API 参考手册

本文档提供 VPE 视频处理引擎的完整 API 参考，包括 N-API（JS/TS）和 C API 接口。

---

## 1 JS/TS N-API

### 1.1 模块注册

#### 1.1.1 主模块 multimedia.videoProcessingEngine

**模块名**：`multimedia.videoProcessingEngine`

**注册文件**：`interfaces/kits/js/native_module_ohos_imageprocessing.cpp:38-54`

```cpp
static napi_module videoProcessingModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Export,
    .nm_modname = "multimedia.videoProcessingEngine",
    .nm_priv = nullptr,
    .reserved = {0},
};

extern "C" __attribute__((constructor)) void VideoProcessingModule(void)
{
    napi_module_register(&videoProcessingModule);
}
```

#### 1.1.2 细节增强模块 multimedia.detailEnhancer

**模块名**：`multimedia.detailEnhancer`

**注册文件**：`framework/capi/image_processing/detail_enhance_napi.cpp:314-327`

```cpp
static napi_module detailEnhanceModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "multimedia.detailEnhancer",
    .nm_priv = ((void *)0),
    .reserved = {0},
};

extern "C" __attribute__((constructor)) void DetailEnhanceRegisterModule(void)
{
    napi_module_register(&detailEnhanceModule);
}
```

### 1.2 N-API 方法列表

#### 1.2.1 multimedia.detailEnhancer 模块

| JS 方法名 | C++ 实现函数 | 功能 | 同步/异步 | 证据位置 |
|-----------|-------------|------|----------|---------|
| `init()` | `DetailEnhanceNapi::Init` | 初始化细节增强模块 | 同步 | `detail_enhance_napi.cpp:91` ✅ |
| `process(width, height, pixelmap)` | `DetailEnhanceNapi::Process` | 执行细节增强处理 | 异步 | `detail_enhance_napi.cpp:253` ✅ |
| `destroy()` | `DetailEnhanceNapi::Destroy` | 销毁细节增强模块 | 同步 | `detail_enhance_napi.cpp:119` ✅ |

**N-API 导出定义**（`detail_enhance_napi.cpp:304-307`）：
```cpp
static napi_property_descriptor desc[] = {
    DECLARE_NAPI_FUNCTION("init", DetailEnhanceNapi::Init),
    DECLARE_NAPI_FUNCTION("process", DetailEnhanceNapi::Process),
    DECLARE_NAPI_FUNCTION("destroy", DetailEnhanceNapi::Destroy),
};
```

#### 1.2.2 multimedia.videoProcessingEngine 模块

| JS 方法名 | C++ 实现函数 | 功能 | 同步/异步 | 证据位置 |
|-----------|-------------|------|----------|---------|
| `init()` | `VpeNapi::Init` | 初始化处理引擎 | 同步 | `detail_enhance_napi_formal.h:32` ✅ |
| `create(type)` | `VpeNapi::Create` | 创建处理实例 | 同步 | `detail_enhance_napi_formal.h:33` ✅ |
| `enhanceDetail(instance, src, dst)` | `VpeNapi::EnhanceDetail` | 细节增强 | 异步 | `detail_enhance_napi_formal.h:34` ✅ |
| `enhanceDetailSync(instance, src, dst)` | `VpeNapi::EnhanceDetailSync` | 细节增强（同步） | 同步 | `detail_enhance_napi_formal.h:35` ✅ |
| `setDetailImage(instance, image)` | `VpeNapi::SetDetailImage` | 设置细节图像 | 同步 | `detail_enhance_napi_formal.h:36` ✅ |
| `setLcdImage(instance, image)` | `VpeNapi::SetLcdImage` | 设置 LCD 图像 | 同步 | `detail_enhance_napi_formal.h:37` ✅ |
| `registerCallback(instance, callback)` | `VpeNapi::RegisterCallback` | 注册回调 | 同步 | `detail_enhance_napi_formal.h:38` ✅ |
| `initializeEnvironment()` | `VpeNapi::InitializeEnvironment` | 初始化环境 | 同步 | `detail_enhance_napi_formal.h:39` ✅ |
| `deinitializeEnvironment()` | `VpeNapi::DeinitializeEnvironment` | 反初始化环境 | 同步 | `detail_enhance_napi_formal.h:40` ✅ |

### 1.3 N-API 使用示例

```typescript
// 方式一：使用 multimedia.detailEnhancer
import detailEnhancer from '@ohos.multimedia.detailEnhancer';

async function enhanceImage() {
    // 初始化
    await detailEnhancer.init();
    
    // 处理图像
    const result = await detailEnhancer.process(width, height, pixelMap);
    
    // 销毁
    await detailEnhancer.destroy();
}

// 方式二：使用 multimedia.videoProcessingEngine
import videoProcessingEngine from '@ohos.multimedia.videoProcessingEngine';

async function useVideoProcessing() {
    // 初始化环境
    await videoProcessingEngine.initializeEnvironment();
    
    // 创建实例
    const instance = await videoProcessingEngine.create(
        videoProcessingEngine.ImageProcessingType.DETAIL_ENHANCER
    );
    
    // 注册回调
    await videoProcessingEngine.registerCallback(instance, {
        onError: (error) => { /* 错误处理 */ },
        onComplete: (result) => { /* 完成处理 */ }
    });
    
    // 执行处理
    await videoProcessingEngine.enhanceDetail(instance, srcPixelMap, dstPixelMap);
}
```

---

## 2 图像处理 C API

### 2.1 图像处理 API 清单

| 行号 | API 名称 | 功能描述 | 同步/异步 | 证据位置 |
|------|----------|---------|----------|---------|
| 63 | `OH_ImageProcessing_InitializeEnvironment` | 初始化全局环境 | 同步 | `image_processing.h:63` ✅ |
| 79 | `OH_ImageProcessing_DeinitializeEnvironment` | 反初始化环境 | 同步 | `image_processing.h:79` ✅ |
| 90 | `OH_ImageProcessing_IsColorSpaceConversionSupported` | 查询色彩空间转换支持 | 同步 | `image_processing.h:90` ✅ |
| 104 | `OH_ImageProcessing_IsCompositionSupported` | 查询图像合成支持 | 同步 | `image_processing.h:104` ✅ |
| 119 | `OH_ImageProcessing_IsDecompositionSupported` | 查询图像分解支持 | 同步 | `image_processing.h:119` ✅ |
| 132 | `OH_ImageProcessing_IsMetadataGenerationSupported` | 查询元数据生成支持 | 同步 | `image_processing.h:132` ✅ |
| 150 | `OH_ImageProcessing_Create` | 创建图像处理实例 | 同步 | `image_processing.h:150` ✅ |
| 161 | `OH_ImageProcessing_Destroy` | 销毁图像处理实例 | 同步 | `image_processing.h:161` ✅ |
| 178 | `OH_ImageProcessing_SetParameter` | 设置参数 | 同步 | `image_processing.h:178` ✅ |
| 193 | `OH_ImageProcessing_GetParameter` | 获取参数 | 同步 | `image_processing.h:193` ✅ |
| 216 | `OH_ImageProcessing_ConvertColorSpace` | 色彩空间转换 | 同步 | `image_processing.h:216` ✅ |
| 239 | `OH_ImageProcessing_Compose` | HDR 图像合成 | 同步 | `image_processing.h:239` ✅ |
| 262 | `OH_ImageProcessing_Decompose` | HDR 图像分解 | 同步 | `image_processing.h:262` ✅ |
| 283 | `OH_ImageProcessing_GenerateMetadata` | 生成元数据 | 同步 | `image_processing.h:283` ✅ |
| 307 | `OH_ImageProcessing_EnhanceDetail` | 细节增强 | 同步 | `image_processing.h:307` ✅ |

### 2.2 环境管理 API

#### 2.2.1 OH_ImageProcessing_InitializeEnvironment

```c
/**
 * @brief 初始化图像处理的全局环境
 * 
 * 此函数为可选函数。通常在进程启动时调用一次，以初始化图像处理的全局环境，
 * 可减少后续 OH_ImageProcessing_Create 的调用时间。
 * 
 * @return {@link IMAGE_PROCESSING_SUCCESS} 初始化成功
 *         {@link IMAGE_PROCESSING_ERROR_INITIALIZE_FAILED} 初始化失败
 * 
 * @since 13
 */
ImageProcessing_ErrorCode OH_ImageProcessing_InitializeEnvironment(void);
```

**证据位置**：`image_processing.h:50-63` ✅

**前置条件**：
- 进程首次使用图像处理功能前调用
- 无未销毁的处理实例存在

**后置条件**：
- 全局初始化完成，后续 Create 调用更快

#### 2.2.2 OH_ImageProcessing_DeinitializeEnvironment

```c
/**
 * @brief 反初始化图像处理的全局环境
 * 
 * 如果调用了 OH_ImageProcessing_InitializeEnvironment，则在进程退出前应调用此函数。
 * 调用此函数时，不能存在未销毁的图像处理实例。
 * 
 * @return {@link IMAGE_PROCESSING_SUCCESS} 反初始化成功
 *         {@link IMAGE_PROCESSING_ERROR_OPERATION_NOT_PERMITTED} 
 *             - 存在未销毁的处理实例
 *             - 未调用 InitializeEnvironment
 * 
 * @since 13
 */
ImageProcessing_ErrorCode OH_ImageProcessing_DeinitializeEnvironment(void);
```

**证据位置**：`image_processing.h:65-79` ✅

### 2.3 能力查询 API

#### 2.3.1 OH_ImageProcessing_IsColorSpaceConversionSupported

```c
/**
 * @brief 查询指定的色彩空间转换是否支持
 * 
 * @param sourceImageInfo 输入图像色彩空间信息指针
 * @param destinationImageInfo 输出图像色彩空间信息指针
 * @return <b>true</b> 支持该色彩空间转换
 *         <b>false</b> 不支持该色彩空间转换
 * 
 * @since 13
 */
bool OH_ImageProcessing_IsColorSpaceConversionSupported(
    const ImageProcessing_ColorSpaceInfo* sourceImageInfo,
    const ImageProcessing_ColorSpaceInfo* destinationImageInfo);
```

**证据位置**：`image_processing.h:82-92` ✅

### 2.4 实例管理 API

#### 2.4.1 OH_ImageProcessing_Create

```c
/**
 * @brief 创建图像处理实例
 * 
 * @param imageProcessor 输出参数，指向新创建的图像处理对象
 * @param type 使用 IMAGE_PROCESSING_TYPE_XXX 指定处理类型
 * @return {@link IMAGE_PROCESSING_SUCCESS} 创建成功
 *         {@link IMAGE_PROCESSING_ERROR_UNSUPPORTED_PROCESSING} 不支持的类型
 *         {@link IMAGE_PROCESSING_ERROR_CREATE_FAILED} 创建失败
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_PARAMETER} 类型参数无效
 * 
 * @since 13
 */
ImageProcessing_ErrorCode OH_ImageProcessing_Create(
    OH_ImageProcessing** imageProcessor, 
    int32_t type);
```

**证据位置**：`image_processing.h:136-150` ✅

#### 2.4.2 OH_ImageProcessing_Destroy

```c
/**
 * @brief 销毁图像处理实例
 * 
 * @param imageProcessor 图像处理实例指针，销毁后建议置为 null
 * @return {@link IMAGE_PROCESSING_SUCCESS} 销毁成功
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 * 
 * @since 13
 */
ImageProcessing_ErrorCode OH_ImageProcessing_Destroy(OH_ImageProcessing* imageProcessor);
```

**证据位置**：`image_processing.h:152-161` ✅

### 2.5 参数配置 API

#### 2.5.1 OH_ImageProcessing_SetParameter

```c
/**
 * @brief 设置图像处理参数
 * 
 * @param imageProcessor 图像处理实例指针
 * @param parameter 图像处理参数（OH_AVFormat 格式）
 * @return {@link IMAGE_PROCESSING_SUCCESS} 设置成功
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_PARAMETER} 参数为空
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_VALUE} 参数值无效
 *         {@link IMAGE_PROCESSING_ERROR_NO_MEMORY} 内存分配失败
 * 
 * @since 13
 */
ImageProcessing_ErrorCode OH_ImageProcessing_SetParameter(
    OH_ImageProcessing* imageProcessor,
    const OH_AVFormat* parameter);
```

**证据位置**：`image_processing.h:164-179` ✅

### 2.6 图像处理 API

#### 2.6.1 OH_ImageProcessing_EnhanceDetail

```c
/**
 * @brief 图像细节增强
 * 
 * 对源图像执行细节增强处理，包括超分辨率和锐化算法。
 * 
 * @param imageProcessor 图像处理实例（类型为 IMAGE_PROCESSING_TYPE_DETAIL_ENHANCER）
 * @param sourceImage 输入图像指针
 * @param destinationImage 输出图像指针
 * @return {@link IMAGE_PROCESSING_SUCCESS} 处理成功
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_PARAMETER} 图像为空
 *         {@link IMAGE_PROCESSING_ERROR_INVALID_VALUE} 图像属性无效
 *         {@link IMAGE_PROCESSING_ERROR_UNSUPPORTED_PROCESSING} 不支持的处理
 *         {@link IMAGE_PROCESSING_ERROR_PROCESS_FAILED} 处理失败
 *         {@link IMAGE_PROCESSING_ERROR_NO_MEMORY} 内存分配失败
 * 
 * @since 13
 */
ImageProcessing_ErrorCode OH_ImageProcessing_EnhanceDetail(
    OH_ImageProcessing* imageProcessor,
    OH_PixelmapNative* sourceImage, 
    OH_PixelmapNative* destinationImage);
```

**证据位置**：`image_processing.h:287-308` ✅

---

## 3 视频处理 C API

### 3.1 视频处理 API 清单

| 行号 | API 名称 | 功能描述 | 同步/异步 | 证据位置 |
|------|----------|---------|----------|---------|
| 63 | `OH_VideoProcessing_InitializeEnvironment` | 初始化全局环境 | 同步 | `video_processing.h:63` ✅ |
| 79 | `OH_VideoProcessing_DeinitializeEnvironment` | 反初始化环境 | 同步 | `video_processing.h:79` ✅ |
| 90 | `OH_VideoProcessing_IsColorSpaceConversionSupported` | 查询色彩空间转换支持 | 同步 | `video_processing.h:90` ✅ |
| 102 | `OH_VideoProcessing_IsMetadataGenerationSupported` | 查询元数据生成支持 | 同步 | `video_processing.h:102` ✅ |
| 120 | `OH_VideoProcessing_Create` | 创建视频处理实例 | 同步 | `video_processing.h:120` ✅ |
| 134 | `OH_VideoProcessing_Destroy` | 销毁视频处理实例 | 同步 | `video_processing.h:134` ✅ |
| 150 | `OH_VideoProcessing_RegisterCallback` | 注册回调对象 | 同步 | `video_processing.h:150` ✅ |
| 165 | `OH_VideoProcessing_SetSurface` | 设置输出 Surface | 同步 | `video_processing.h:165` ✅ |
| 183 | `OH_VideoProcessing_GetSurface` | 获取输入 Surface | 同步 | `video_processing.h:183` ✅ |
| 200 | `OH_VideoProcessing_SetParameter` | 设置参数 | 同步 | `video_processing.h:200` ✅ |
| 215 | `OH_VideoProcessing_GetParameter` | 获取参数 | 同步 | `video_processing.h:215` ✅ |
| 230 | `OH_VideoProcessing_Start` | 启动处理 | 同步 | `video_processing.h:230` ✅ |
| 244 | `OH_VideoProcessing_Stop` | 停止处理 | 同步 | `video_processing.h:244` ✅ |
| 261 | `OH_VideoProcessing_RenderOutputBuffer` | 渲染输出缓冲区 | 同步 | `video_processing.h:261` ✅ |
| 273 | `OH_VideoProcessingCallback_Create` | 创建回调对象 | 同步 | `video_processing.h:273` ✅ |
| 286 | `OH_VideoProcessingCallback_Destroy` | 销毁回调对象 | 同步 | `video_processing.h:286` ✅ |
| 297 | `OH_VideoProcessingCallback_BindOnError` | 绑定错误回调 | 同步 | `video_processing.h:297` ✅ |
| 309 | `OH_VideoProcessingCallback_BindOnState` | 绑定状态回调 | 同步 | `video_processing.h:309` ✅ |
| 321 | `OH_VideoProcessingCallback_BindOnNewOutputBuffer` | 绑定新输出缓冲区回调 | 同步 | `video_processing.h:321` ✅ |

### 3.2 环境管理 API

#### 3.2.1 OH_VideoProcessing_InitializeEnvironment

```c
/**
 * @brief 初始化视频处理的全局环境
 * 
 * 此函数为可选函数。通常在进程启动时调用一次，以初始化视频处理的全局环境，
 * 可减少后续 OH_VideoProcessing_Create 的调用时间。
 * 
 * @return {@link VIDEO_PROCESSING_SUCCESS} 初始化成功
 *         {@link VIDEO_PROCESSING_ERROR_INITIALIZE_FAILED} 初始化失败
 * 
 * @since 12
 */
VideoProcessing_ErrorCode OH_VideoProcessing_InitializeEnvironment(void);
```

**证据位置**：`video_processing.h:50-63` ✅

### 3.3 Surface 管理 API

#### 3.3.1 OH_VideoProcessing_GetSurface

```c
/**
 * @brief 创建输入 Surface
 * 
 * 在开始视频处理前调用，获取输入 Surface。
 * 
 * @param videoProcessor 视频处理实例指针
 * @param window 输入 Surface 指针（例如视频解码器的输出 Surface）
 * @return {@link VIDEO_PROCESSING_SUCCESS} 操作成功
 *         {@link VIDEO_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 *         {@link VIDEO_PROCESSING_ERROR_INVALID_PARAMETER} 参数无效
 *         {@link VIDEO_PROCESSING_ERROR_OPERATION_NOT_PERMITTED} 操作不允许
 * 
 * @since 12
 */
VideoProcessing_ErrorCode OH_VideoProcessing_GetSurface(
    OH_VideoProcessing* videoProcessor, 
    OHNativeWindow** window);
```

**证据位置**：`video_processing.h:169-183` ✅

#### 3.3.2 OH_VideoProcessing_SetSurface

```c
/**
 * @brief 设置输出 Surface
 * 
 * 在开始视频处理前调用，设置输出 Surface。
 * 
 * @param videoProcessor 视频处理实例指针
 * @param window 输出 Surface 指针
 * @return {@link VIDEO_PROCESSING_SUCCESS} 操作成功
 *         {@link VIDEO_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 *         {@link VIDEO_PROCESSING_ERROR_INVALID_PARAMETER} 参数无效
 * 
 * @since 12
 */
VideoProcessing_ErrorCode OH_VideoProcessing_SetSurface(
    OH_VideoProcessing* videoProcessor,
    const OHNativeWindow* window);
```

**证据位置**：`video_processing.h:153-166` ✅

### 3.4 生命周期 API

#### 3.4.1 OH_VideoProcessing_Start

```c
/**
 * @brief 启动视频处理实例
 * 
 * 启动后，通过回调报告状态变化。
 * 
 * @param videoProcessor 视频处理实例指针
 * @return {@link VIDEO_PROCESSING_SUCCESS} 操作成功
 *         {@link VIDEO_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 *         {@link VIDEO_PROCESSING_ERROR_OPERATION_NOT_PERMITTED} 操作不允许
 *             - 未设置输出 Surface
 *             - 未创建输入 Surface
 *             - 实例已在运行
 * 
 * @since 12
 */
VideoProcessing_ErrorCode OH_VideoProcessing_Start(OH_VideoProcessing* videoProcessor);
```

**证据位置**：`video_processing.h:218-230` ✅

#### 3.4.2 OH_VideoProcessing_Stop

```c
/**
 * @brief 停止视频处理实例
 * 
 * 停止后，通过回调报告 VIDEO_PROCESSING_STATE_STOPPED 状态。
 * 
 * @param videoProcessor 视频处理实例指针
 * @return {@link VIDEO_PROCESSING_SUCCESS} 操作成功
 *         {@link VIDEO_PROCESSING_ERROR_INVALID_INSTANCE} 实例无效
 *         {@link VIDEO_PROCESSING_ERROR_OPERATION_NOT_PERMITTED} 实例已停止
 * 
 * @since 12
 */
VideoProcessing_ErrorCode OH_VideoProcessing_Stop(OH_VideoProcessing* videoProcessor);
```

**证据位置**：`video_processing.h:232-244` ✅

### 3.5 回调 API

#### 3.5.1 回调绑定函数

| API 名称 | 功能 | 证据位置 |
|----------|------|---------|
| `OH_VideoProcessingCallback_BindOnError` | 绑定错误回调 | `video_processing.h:297` ✅ |
| `OH_VideoProcessingCallback_BindOnState` | 绑定状态回调 | `video_processing.h:309` ✅ |
| `OH_VideoProcessingCallback_BindOnNewOutputBuffer` | 绑定新输出缓冲区回调 | `video_processing.h:321` ✅ |

**回调类型定义**：
```c
/**
 * @brief 错误回调
 */
typedef void (*OH_VideoProcessingCallback_OnError)(
    OH_VideoProcessing* videoProcessor, 
    VideoProcessing_ErrorCode error, 
    void* userData);

/**
 * @brief 状态回调
 */
typedef void (*OH_VideoProcessingCallback_OnState)(
    OH_VideoProcessing* videoProcessor, 
    VideoProcessing_State state, 
    void* userData);

/**
 * @brief 新输出缓冲区回调
 */
typedef void (*OH_VideoProcessingCallback_OnNewOutputBuffer)(
    OH_VideoProcessing* videoProcessor, 
    uint32_t index, 
    void* userData);
```

---

## 4 错误码

### 4.1 图像处理错误码

| 错误码 | 定义 | 说明 |
|--------|------|------|
| `IMAGE_PROCESSING_SUCCESS` | 0 | 成功 |
| `IMAGE_PROCESSING_ERROR` | -1 | 通用错误 |
| `IMAGE_PROCESSING_ERROR_INVALID_INSTANCE` | -2 | 无效实例 |
| `IMAGE_PROCESSING_ERROR_INVALID_PARAMETER` | -3 | 无效参数 |
| `IMAGE_PROCESSING_ERROR_INVALID_VALUE` | -4 | 无效值 |
| `IMAGE_PROCESSING_ERROR_UNSUPPORTED_PROCESSING` | -5 | 不支持的处理 |
| `IMAGE_PROCESSING_ERROR_PROCESS_FAILED` | -6 | 处理失败 |
| `IMAGE_PROCESSING_ERROR_NO_MEMORY` | -7 | 内存不足 |
| `IMAGE_PROCESSING_ERROR_INITIALIZE_FAILED` | -8 | 初始化失败 |
| `IMAGE_PROCESSING_ERROR_OPERATION_NOT_PERMITTED` | -9 | 操作不允许 |

**证据位置**：`image_processing_types.h` ✅

### 4.2 视频处理错误码

| 错误码 | 定义 | 说明 |
|--------|------|------|
| `VIDEO_PROCESSING_SUCCESS` | 0 | 成功 |
| `VIDEO_PROCESSING_ERROR` | -1 | 通用错误 |
| `VIDEO_PROCESSING_ERROR_INVALID_INSTANCE` | -2 | 无效实例 |
| `VIDEO_PROCESSING_ERROR_INVALID_PARAMETER` | -3 | 无效参数 |
| `VIDEO_PROCESSING_ERROR_INVALID_VALUE` | -4 | 无效值 |
| `VIDEO_PROCESSING_ERROR_UNSUPPORTED_PROCESSING` | -5 | 不支持的处理 |
| `VIDEO_PROCESSING_ERROR_PROCESS_FAILED` | -6 | 处理失败 |
| `VIDEO_PROCESSING_ERROR_NO_MEMORY` | -7 | 内存不足 |
| `VIDEO_PROCESSING_ERROR_INITIALIZE_FAILED` | -8 | 初始化失败 |
| `VIDEO_PROCESSING_ERROR_OPERATION_NOT_PERMITTED` | -9 | 操作不允许 |

**证据位置**：`video_processing_types.h` ✅

---

## 5 API 调用链

### 5.1 图像处理调用链

```
┌─────────────────────────────────────────────────────────────────┐
│ 应用层 (JS/TS)                                                   │
└────────────────────────────┬────────────────────────────────────┘
                             │ require('@ohos.multimedia.detailEnhancer')
┌────────────────────────────▼────────────────────────────────────┐
│ N-API 层 (detail_enhance_napi.cpp)                              │
│   ├─ init()     → DetailEnhanceNapi::Init()                    │
│   ├─ process()   → DetailEnhanceNapi::Process()                │
│   └─ destroy()   → DetailEnhanceNapi::Destroy()                │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ CAPI 层 (image_processing_impl.cpp)                             │
│   └─ OH_ImageProcessing_XXX()  调用框架接口                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 框架层 (framework/algorithm/detail_enhancer/)                    │
│   └─ DetailEnhancerImage::Create() / Process()                  │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 服务层 (SA 进程)                                                 │
│   └─ VideoProcessingServer::Process()                          │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 算法层 (plugins/Skia/EVE/AISR)                                  │
│   └─ 具体算法实现                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**证据位置**：`detail_enhance_napi.cpp` → `image_processing_impl.cpp` → `detail_enhancer_image_fwk.cpp` ✅

### 5.2 视频处理调用链

```
┌─────────────────────────────────────────────────────────────────┐
│ 应用层 (Native C++)                                             │
└────────────────────────────┬────────────────────────────────────┘
                             │ dlopen(libvideo_processing.so)
┌────────────────────────────▼────────────────────────────────────┐
│ CAPI 层 (video_processing_impl.cpp)                             │
│   ├─ OH_VideoProcessing_Create()                               │
│   ├─ OH_VideoProcessing_SetSurface() / GetSurface()           │
│   ├─ OH_VideoProcessing_RegisterCallback()                     │
│   ├─ OH_VideoProcessing_Start() / Stop()                       │
│   └─ OH_VideoProcessing_RenderOutputBuffer()                   │
└────────────────────────────┬────────────────────────────────────┘
                             │ IPC (Binder)
┌────────────────────────────▼────────────────────────────────────┐
│ IPC 代理层 (video_processing_service_manager_proxy.cpp)          │
│   └─ IVideoProcessingServiceManager 接口                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ IPC 存根层 (video_processing_service_manager_stub.cpp)           │
│   └─ VideoProcessingServer::OnRequest()                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 服务层 (video_processing_server.cpp)                            │
│   ├─ Create() / Destroy()                                       │
│   ├─ SetParameter() / GetParameter()                            │
│   └─ Process()                                                  │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 算法层 (plugins/Skia/EVE/AISR/HDR)                               │
│   └─ 具体算法实现                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**证据位置**：`video_processing_impl.cpp` → IDL Proxy/Stub → `video_processing_server.cpp` ✅

---

## 6 相关文档链接

| 文档 | 说明 |
|------|------|
| [Architecture.md](./Architecture.md) | 架构设计 |
| [Inner_API.md](./Inner_API.md) | Inner API |
| [Build_System.md](./Build_System.md) | 构建系统 |
| [Security_Review.md](./Security_Review.md) | 安全评审 |

---

## 7 更新日志

| 版本 | 日期 | 变更内容 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本，包含完整 API 参考 |
