# VK-GL-CTS OpenHarmony 特有 API 差异

## 概述

OpenHarmony 为 VK-GL-CTS 添加了**OHOS 特有的 Vulkan 扩展**，用于支持 OpenHarmony 原生图形系统（Rosen）与 Vulkan 的集成。

这些扩展遵循 Vulkan 扩展命名规范，使用 `VK_OpenHarmony_` 前缀。

## OHOS 特有 Vulkan 扩展

### 1. VK_OpenHarmony_OHOS_surface

**扩展名称**: `VK_OpenHarmony_OHOS_surface`  
**扩展编号**: 451  
**规范版本**: 1

#### 功能说明

该扩展允许 Vulkan 应用程序在 OpenHarmony 的 `OH_NativeWindow` 上创建 Vulkan Surface。

#### 新增结构体

```c
// 创建 OHOS Surface 的信息结构体
typedef struct VkOHOSSurfaceCreateInfoOpenHarmony {
    VkStructureType                        sType;
    const void*                            pNext;
    VkOHOSSurfaceCreateFlagsOpenHarmony    flags;
    struct OH_NativeWindow*                window;  // OHOS 原生窗口
} VkOHOSSurfaceCreateInfoOpenHarmony;
```

#### 新增枚举

```c
// 结构体类型枚举值
VK_STRUCTURE_TYPE_OHOS_SURFACE_CREATE_INFO_OPENHARMONY = 1000451000;
```

#### 新增函数

```c
// 创建 OHOS Surface
VkResult vkCreateOHOSSurfaceOpenHarmony(
    VkInstance instance,
    const VkOHOSSurfaceCreateInfoOpenHarmony* pCreateInfo,
    const VkAllocationCallbacks* pAllocator,
    VkSurfaceKHR* pSurface);
```

#### 使用示例

```cpp
// 使用 VK_OpenHarmony_OHOS_surface 扩展

// 1. 检查扩展支持
VkBool32 presentSupport = VK_FALSE;
vkGetPhysicalDeviceSurfaceSupportKHR(physicalDevice, queueFamilyIndex, surface, &presentSupport);

// 2. 创建 OHOS Surface
VkOHOSSurfaceCreateInfoOpenHarmony surfaceCreateInfo = {};
surfaceCreateInfo.sType = VK_STRUCTURE_TYPE_OHOS_SURFACE_CREATE_INFO_OPENHARMONY;
surfaceCreateInfo.window = ohNativeWindow;  // 从 OHOS 获取

VkSurfaceKHR surface;
VkResult result = vkCreateOHOSSurfaceOpenHarmony(instance, &surfaceCreateInfo, nullptr, &surface);
```

### 2. VK_OpenHarmony_external_memory_OHOS_native_buffer

**扩展名称**: `VK_OpenHarmony_external_memory_OHOS_native_buffer`  
**扩展编号**: 452  
**规范版本**: 1

#### 功能说明

该扩展允许 Vulkan 导入和导出 OpenHarmony 的 `OH_NativeBuffer`，实现 Vulkan 与 OpenHarmony 图形系统的零拷贝内存共享。

#### 新增句柄类型

```c
// 外部内存句柄类型 - OHOS Native Buffer
VK_EXTERNAL_MEMORY_HANDLE_TYPE_OHOS_NATIVE_BUFFER_BIT_OPENHARMONY = 0x00004000;
```

#### 新增结构体

```c
// OHOS Native Buffer 使用信息
typedef struct VkOHOSNativeBufferUsageOpenHarmony {
    VkStructureType    sType;
    const void*        pNext;
    deUint64           OHOSNativeBufferUsage;  // OHOS Buffer 使用标志
} VkOHOSNativeBufferUsageOpenHarmony;

// OHOS Native Buffer 属性
typedef struct VkOHOSNativeBufferPropertiesOpenHarmony {
    VkStructureType    sType;
    void*              pNext;
    VkDeviceSize       allocationSize;
    uint32_t           memoryTypeBits;
} VkOHOSNativeBufferPropertiesOpenHarmony;

// OHOS Native Buffer 格式属性
typedef struct VkOHOSNativeBufferFormatPropertiesOpenHarmony {
    VkStructureType                  sType;
    void*                            pNext;
    VkFormat                         format;
    uint64_t                         externalFormat;
    VkFormatFeatureFlags             formatFeatures;
    VkComponentMapping               samplerYcbcrConversionComponents;
    VkSamplerYcbcrModelConversion    suggestedYcbcrModel;
    VkSamplerYcbcrRange              suggestedYcbcrRange;
    VkChromaLocation                 suggestedXChromaOffset;
    VkChromaLocation                 suggestedYChromaOffset;
} VkOHOSNativeBufferFormatPropertiesOpenHarmony;

// 导入 OHOS Native Buffer 信息
typedef struct VkImportOHOSNativeBufferInfoOpenHarmony {
    VkStructureType            sType;
    const void*                pNext;
    struct OH_NativeBuffer*    buffer;       // 要导入的 OHOS Buffer
} VkImportOHOSNativeBufferInfoOpenHarmony;

// 获取 OHOS Native Buffer 信息
typedef struct VkMemoryGetOHOSNativeBufferInfoOpenHarmony {
    VkStructureType    sType;
    const void*        pNext;
    VkDeviceMemory     memory;               // Vulkan 设备内存
    VkDeviceSize       handleType;           // 句柄类型
} VkMemoryGetOHOSNativeBufferInfoOpenHarmony;
```

#### 新增枚举

```c
VK_STRUCTURE_TYPE_OHOS_NATIVE_BUFFER_USAGE_OPENHARMONY          = 1000452000;
VK_STRUCTURE_TYPE_OHOS_NATIVE_BUFFER_PROPERTIES_OPENHARMONY     = 1000452001;
VK_STRUCTURE_TYPE_OHOS_NATIVE_BUFFER_FORMAT_PROPERTIES_OPENHARMONY = 1000452002;
VK_STRUCTURE_TYPE_IMPORT_OHOS_NATIVE_BUFFER_INFO_OPENHARMONY    = 1000452003;
VK_STRUCTURE_TYPE_MEMORY_GET_OHOS_NATIVE_BUFFER_INFO_OPENHARMONY = 1000452004;
```

#### 新增函数

```c
// 获取 OHOS Native Buffer 的属性
VkResult vkGetOHOSNativeBufferPropertiesOpenHarmony(
    VkDevice device,
    const struct OH_NativeBuffer* buffer,
    VkOHOSNativeBufferPropertiesOpenHarmony* pProperties);

// 从 Vulkan 内存获取 OHOS Native Buffer
VkResult vkGetMemoryOHOSNativeBufferOpenHarmony(
    VkDevice device,
    const VkMemoryGetOHOSNativeBufferInfoOpenHarmony* pInfo,
    struct OH_NativeBuffer** pBuffer);
```

#### 使用示例

```cpp
// 导入 OHOS Native Buffer 到 Vulkan

// 1. 获取 OHOS Buffer 属性
VkOHOSNativeBufferPropertiesOpenHarmony bufferProps = {};
bufferProps.sType = VK_STRUCTURE_TYPE_OHOS_NATIVE_BUFFER_PROPERTIES_OPENHARMONY;

vkGetOHOSNativeBufferPropertiesOpenHarmony(device, ohNativeBuffer, &bufferProps);

// 2. 创建 Vulkan 图像 (使用外部内存)
VkExternalMemoryImageCreateInfo externalMemInfo = {};
externalMemInfo.sType = VK_STRUCTURE_TYPE_EXTERNAL_MEMORY_IMAGE_CREATE_INFO;
externalMemInfo.handleTypes = VK_EXTERNAL_MEMORY_HANDLE_TYPE_OHOS_NATIVE_BUFFER_BIT_OPENHARMONY;

VkImageCreateInfo imageCreateInfo = {};
imageCreateInfo.sType = VK_STRUCTURE_TYPE_IMAGE_CREATE_INFO;
imageCreateInfo.pNext = &externalMemInfo;
// ... 其他参数

VkImage image;
vkCreateImage(device, &imageCreateInfo, nullptr, &image);

// 3. 分配并绑定内存 (导入 OHOS Buffer)
VkImportMemoryAllocateInfo importAllocInfo = {};
importAllocInfo.sType = VK_STRUCTURE_TYPE_IMPORT_MEMORY_ALLOCATE_INFO;
importAllocInfo.handleTypes = VK_EXTERNAL_MEMORY_HANDLE_TYPE_OHOS_NATIVE_BUFFER_BIT_OPENHARMONY;

VkMemoryAllocateInfo allocInfo = {};
allocInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
allocInfo.pNext = &importAllocInfo;
allocInfo.allocationSize = bufferProps.allocationSize;
allocInfo.memoryTypeIndex = /* 根据 bufferProps.memoryTypeBits 选择 */;

VkDeviceMemory memory;
vkAllocateMemory(device, &allocInfo, nullptr, &memory);
vkBindImageMemory(device, image, memory, 0);
```

### 3. Swapchain 相关扩展 (OpenHarmony 特有)

#### 功能说明

用于 Swapchain 与 OpenHarmony Gralloc 系统的集成。

#### 新增函数

```c
// 获取 Swapchain Gralloc 使用方式
VkResult vkGetSwapchainGrallocUsageOpenHarmony(
    VkDevice device,
    VkFormat format,
    VkImageUsageFlags imageUsage,
    int* grallocUsage);

// 设置原生 Fence FD
VkResult vkSetNativeFenceFdOpenHarmony(
    VkDevice device,
    int32_t nativeFenceFd,
    VkSemaphore semaphore,
    VkFence fence);

// 获取原生 Fence FD
VkResult vkGetNativeFenceFdOpenHarmony(
    VkQueue queue,
    uint32_t waitSemaphoreCount,
    const VkSemaphore* pWaitSemaphores,
    VkImage image,
    int32_t* pNativeFenceFd);
```

#### 使用示例

```cpp
// Swapchain 与 OHOS Gralloc 集成

// 1. 查询 Gralloc 使用标志
int grallocUsage = 0;
vkGetSwapchainGrallocUsageOpenHarmony(device, VK_FORMAT_B8G8R8A8_UNORM, 
                                       VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT, &grallocUsage);

// 2. 在 Queue Submit 时处理 Fence
// 设置 Fence
vkSetNativeFenceFdOpenHarmony(device, nativeFenceFd, semaphore, fence);

// 获取 Fence
int32_t outFenceFd = -1;
vkGetNativeFenceFdOpenHarmony(queue, waitSemaphoreCount, pWaitSemaphores, image, &outFenceFd);
```

## 代码分布

这些扩展的定义分布在以下生成代码文件中：

### Vulkan 类型定义

| 文件 | 内容 |
|------|------|
| `build/external/vulkancts/framework/vulkan/vkBasicTypes.inl` | 基本类型和枚举 |
| `build/external/vulkancts/framework/vulkan/vkStructTypes.inl` | 结构体定义 |
| `build/external/vulkancts/framework/vulkan/vkFunctionPointerTypes.inl` | 函数指针类型 |

### 接口实现

| 文件 | 内容 |
|------|------|
| `build/external/vulkancts/framework/vulkan/vkConcreteInstanceInterface.inl` | 实例接口 |
| `build/external/vulkancts/framework/vulkan/vkConcreteDeviceInterface.inl` | 设备接口 |
| `build/external/vulkancts/framework/vulkan/vkInstanceDriverImpl.inl` | 实例驱动实现 |
| `build/external/vulkancts/framework/vulkan/vkDeviceDriverImpl.inl` | 设备驱动实现 |
| `build/external/vulkancts/framework/vulkan/vkNullDriverImpl.inl` | Null 驱动实现 |

### 扩展依赖

| 文件 | 内容 |
|------|------|
| `build/external/vulkancts/framework/vulkan/vkApiExtensionDependencyInfo.inl` | 扩展依赖关系 |

## 扩展依赖关系

### VK_OpenHarmony_OHOS_surface 依赖

```
VK_OpenHarmony_OHOS_surface requires:
- VK_KHR_surface (核心 Surface 扩展)
```

### VK_OpenHarmony_external_memory_OHOS_native_buffer 依赖

```
VK_OpenHarmony_external_memory_OHOS_native_buffer requires:
- VK_KHR_sampler_ycbcr_conversion
- VK_KHR_external_memory
- VK_EXT_queue_family_foreign
- VK_KHR_dedicated_allocation
```

## 与上游的差异

### 标准 Vulkan 扩展 vs OHOS 扩展

| 特性 | 标准 Vulkan 扩展 | OHOS 扩展 |
|------|------------------|-----------|
| **前缀** | `VK_KHR_`, `VK_EXT_` | `VK_OpenHarmony_` |
| **组织** | Khronos 官方 | OpenHarmony 特有 |
| **移植性** | 跨平台 | 仅 OHOS |
| **编号** | 官方分配 | 使用预留范围 |

### 与 Android 扩展对比

Android 有类似的扩展：
- `VK_ANDROID_external_memory_android_hardware_buffer`
- `VK_ANDROID_native_buffer`

OpenHarmony 的设计参考了 Android，但使用了独立的命名空间和数据结构：

| Android | OpenHarmony |
|---------|-------------|
| `AHardwareBuffer` | `OH_NativeBuffer` |
| `ANativeWindow` | `OH_NativeWindow` |
| `VK_ANDROID_external_memory_android_hardware_buffer` | `VK_OpenHarmony_external_memory_OHOS_native_buffer` |

## 实现细节

### 平台层实现

这些扩展的实际实现在 `framework/platform/ohos/` 目录下：

```cpp
// tcuOhosPlatform.cpp 中实现 WSI 接口

class OhosWindowInterface : public vk::wsi::WindowInterface {
public:
    // 实现 createSurface 等接口
    // 内部调用 vkCreateOHOSSurfaceOpenHarmony
};
```

### Rosen 集成

Rosen 图形框架提供了底层实现：

```cpp
// ohos_context_i.cpp 中实现

namespace OHOS {
    // 创建 OHOS Surface 的底层实现
    VkResult OhosContextI::CreateSurface(VkInstance instance, 
                                          OH_NativeWindow* window,
                                          VkSurfaceKHR* surface) {
        // 调用驱动层的 vkCreateOHOSSurfaceOpenHarmony
    }
}
```

## 测试覆盖

VK-GL-CTS 中包含对这些扩展的测试：

| 测试模块 | 说明 |
|----------|------|
| `vktWsiTests.cpp` | WSI (Window System Integration) 测试 |
| `vktExternalMemoryTests.cpp` | 外部内存测试 |

## 升级注意事项

### Vulkan 版本升级

1. **检查扩展编号冲突**：确保 `VK_OpenHarmony_*` 编号未与新版本 Vulkan 冲突
2. **更新生成代码**：重新生成 `build/external/vulkancts/` 下的代码
3. **验证功能**：测试扩展在新版本下的兼容性

### 驱动支持

GPU 驱动需要实现这些扩展的入口点：

```c
// 驱动需要实现的函数
VKAPI_ATTR VkResult VKAPI_CALL vkCreateOHOSSurfaceOpenHarmony(...);
VKAPI_ATTR VkResult VKAPI_CALL vkGetOHOSNativeBufferPropertiesOpenHarmony(...);
VKAPI_ATTR VkResult VKAPI_CALL vkGetMemoryOHOSNativeBufferOpenHarmony(...);
```

## 参考文档

- [Vulkan Extension Registry](https://registry.khronos.org/vulkan/)
- [OpenHarmony Native API](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/napi)
- [Rosen 图形框架文档](https://gitee.com/openharmony/graphic_graphic_2d)
