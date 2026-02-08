# API 差异分析

## 5.1 OH 特有 API

### 新增扩展概览

| 扩展名称 | 头文件 | 版本 | 功能描述 |
|---------|--------|------|---------|
| `VK_OHOS_surface` | vulkan_ohos.h | 1 | OHOS NativeWindow Surface 创建 |
| `VK_OHOS_native_buffer` | vulkan_ohos.h | 1 | OH Native Buffer 操作 |
| `VK_OHOS_external_memory` | vulkan_ohos.h | 1 | 外部内存共享扩展 |
| `VK_QNX_screen_surface` | vulkan_screen.h | 1 | QNX Screen Surface |
| `VK_QNX_external_memory_screen_buffer` | vulkan_screen.h | 1 | QNX Screen Buffer |

### VK_OHOS_surface 扩展详解

#### 新增结构体

```c
typedef struct VkSurfaceCreateInfoOHOS {
    VkStructureType             sType;        // 必须是 VK_STRUCTURE_TYPE_SURFACE_CREATE_INFO_OHOS
    const void*                 pNext;         // 扩展链
    VkSurfaceCreateFlagsOHOS    flags;         // 保留，必须为 0
    OHNativeWindow*             window;        // OH NativeWindow 句柄
} VkSurfaceCreateInfoOHOS;

typedef VkFlags VkSurfaceCreateFlagsOHOS;  // 标志类型，目前为空
```

#### 新增枚举

```c
#define VK_OHOS_surface 1                    // 扩展启用标志
#define VK_OHOS_SURFACE_SPEC_VERSION 1       // 扩展版本号
#define VK_OHOS_SURFACE_EXTENSION_NAME "VK_OHOS_surface"  // 扩展名称字符串
```

#### 新增函数

```c
// 函数指针类型
typedef VkResult (VKAPI_PTR *PFN_vkCreateSurfaceOHOS)(
    VkInstance                                  instance,
    const VkSurfaceCreateInfoOHOS*              pCreateInfo,
    const VkAllocationCallbacks*                pAllocator,
    VkSurfaceKHR*                               pSurface);

// 函数声明
VKAPI_ATTR VkResult VKAPI_CALL vkCreateSurfaceOHOS(
    VkInstance                                  instance,
    const VkSurfaceCreateInfoOHOS*              pCreateInfo,
    const VkAllocationCallbacks*                pAllocator,
    VkSurfaceKHR*                               pSurface);
```

**使用示例**：
```c
// 创建 OHOS Surface
OHNativeWindow* nativeWindow = GetNativeWindowFromOHApp();
VkSurfaceCreateInfoOHOS createInfo = {
    .sType = VK_STRUCTURE_TYPE_SURFACE_CREATE_INFO_OHOS,
    .pNext = NULL,
    .flags = 0,
    .window = nativeWindow,
};

VkSurfaceKHR surface;
VkResult result = vkCreateSurfaceOHOS(instance, &createInfo, NULL, &surface);
```

### VK_OHOS_native_buffer 扩展详解

#### 新增枚举

```c
// Swapchain 图像使用标志
typedef enum VkSwapchainImageUsageFlagBitsOHOS {
    VK_SWAPCHAIN_IMAGE_USAGE_SHARED_BIT_OHOS = 0x00000001,
    VK_SWAPCHAIN_IMAGE_USAGE_FLAG_BITS_MAX_ENUM_OHOS = 0x7FFFFFFF
} VkSwapchainImageUsageFlagBitsOHOS;

typedef VkFlags VkSwapchainImageUsageFlagsOHOS;
```

#### 新增结构体

```c
// Native Buffer 信息
typedef struct VkNativeBufferOHOS {
    VkStructureType    sType;
    const void*        pNext;
    struct OHBufferHandle*      handle;  // OH Buffer 句柄
} VkNativeBufferOHOS;

// Swapchain 图像创建信息
typedef struct VkSwapchainImageCreateInfoOHOS {
    VkStructureType                   sType;
    const void*                       pNext;
    VkSwapchainImageUsageFlagsOHOS    usage;  // 图像使用标志
} VkSwapchainImageCreateInfoOHOS;

// 物理设备展示属性
typedef struct VkPhysicalDevicePresentationPropertiesOHOS {
    VkStructureType    sType;
    void*              pNext;
    VkBool32           sharedImage;  // 是否支持共享图像
} VkPhysicalDevicePresentationPropertiesOHOS;
```

#### 新增函数

```c
// 查询 Gralloc 使用
typedef VkResult (VKAPI_PTR *PFN_vkGetSwapchainGrallocUsageOHOS)(
    VkDevice                                    device,
    VkFormat                                    format,
    VkImageUsageFlags                           imageUsage,
    uint64_t*                                   grallocUsage);

// 获取图像（带栅栏）
typedef VkResult (VKAPI_PTR *PFN_vkAcquireImageOHOS)(
    VkDevice                                    device,
    VkImage                                     image,
    int32_t                                     nativeFenceFd,
    VkSemaphore                                 semaphore,
    VkFence                                     fence);

// 释放图像（带栅栏）
typedef VkResult (VKAPI_PTR *PFN_vkQueueSignalReleaseImageOHOS)(
    VkQueue                                     queue,
    uint32_t                                    waitSemaphoreCount,
    const VkSemaphore*                          pWaitSemaphores,
    VkImage                                     image,
    int32_t*                                    pNativeFenceFd);
```

### VK_OHOS_external_memory 扩展详解

#### 新增结构体

```c
// Native Buffer 使用信息
typedef struct VkNativeBufferUsageOHOS {
    VkStructureType    sType;
    void*              pNext;
    uint64_t           OHOSNativeBufferUsage;  // OH NativeBuffer 使用标志
} VkNativeBufferUsageOHOS;

// Native Buffer 属性
typedef struct VkNativeBufferPropertiesOHOS {
    VkStructureType    sType;
    void*              pNext;
    VkDeviceSize       allocationSize;         // 分配大小
    uint32_t           memoryTypeBits;         // 支持的内存类型
} VkNativeBufferPropertiesOHOS;

// Native Buffer 格式属性
typedef struct VkNativeBufferFormatPropertiesOHOS {
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
} VkNativeBufferFormatPropertiesOHOS;

// 导入 Native Buffer 信息
typedef struct VkImportNativeBufferInfoOHOS {
    VkStructureType            sType;
    const void*                pNext;
    struct OH_NativeBuffer*    buffer;  // OH NativeBuffer 指针
} VkImportNativeBufferInfoOHOS;

// 获取内存的 Native Buffer
typedef struct VkMemoryGetNativeBufferInfoOHOS {
    VkStructureType    sType;
    const void*        pNext;
    VkDeviceMemory     memory;  // Vulkan 内存对象
} VkMemoryGetNativeBufferInfoOHOS;

// 外部格式
typedef struct VkExternalFormatOHOS {
    VkStructureType    sType;
    void*              pNext;
    uint64_t           externalFormat;
} VkExternalFormatOHOS;
```

#### 新增函数

```c
// 获取 Native Buffer 属性
typedef VkResult (VKAPI_PTR *PFN_vkGetNativeBufferPropertiesOHOS)(
    VkDevice                                    device,
    const struct OH_NativeBuffer*               buffer,
    VkNativeBufferPropertiesOHOS*               pProperties);

// 从内存获取 Native Buffer
typedef VkResult (VKAPI_PTR *PFN_vkGetMemoryNativeBufferOHOS)(
    VkDevice                                    device,
    const VkMemoryGetNativeBufferInfoOHOS*      pInfo,
    struct OH_NativeBuffer**                    pBuffer);
```

## 5.2 行为变更的 API

### 条件编译行为

#### VK_USE_PLATFORM_OHOS 宏

当 `VK_USE_PLATFORM_OHOS` 被定义时：

| 行为 | 未定义时 | 定义时 |
|------|---------|--------|
| `VK_OHOS_surface` | 未定义 | 启用 |
| `VK_OHOS_native_buffer` | 未定义 | 启用 |
| `VK_OHOS_external_memory` | 未定义 | 启用 |
| `OHNativeWindow` 类型 | 未定义 | 已定义 |
| `OH_NativeBuffer` 类型 | 未定义 | 已定义 |

### OHNativeWindow 类型

```c
// vulkan_ohos.h 中定义
typedef struct NativeWindow OHNativeWindow;
```

**说明**：
- `NativeWindow` 是 OH Native Window 系统的句柄类型
- 该类型在 OH 系统库中定义
- Vulkan 使用该句柄创建 Surface

### OH_NativeBuffer 类型

```c
// vulkan_ohos.h 中声明
struct OH_NativeBuffer;

// 引用类型
typedef struct OH_NativeBuffer OH_NativeBuffer;
```

**说明**：
- `OH_NativeBuffer` 是 OH Native Buffer 系统的缓冲区类型
- 用于 Vulkan 与 OH 图形系统之间的内存共享

## 5.3 废弃或待废弃功能

### vk_ohos_native_buffer.h 中的废弃声明

```c
/**
 * @brief move to vk_ohos_native_buffer.h 
 * @since 10
 * @deprecated since 23
 */
typedef enum VkSwapchainImageUsageFlagBitsOHOS { ... } VkSwapchainImageUsageFlagBitsOHOS;

/**
 * @brief move to vk_ohos_native_buffer.h 
 * @since 10
 * @deprecated since 23
 */
typedef struct VkNativeBufferOHOS { ... } VkNativeBufferOHOS;

/**
 * @brief move to vk_ohos_native_buffer.h 
 * @since 10
 * @deprecated since 23
 */
typedef struct VkSwapchainImageCreateInfoOHOS { ... } VkSwapchainImageCreateInfoOHOS;

/**
 * @brief move to vk_ohos_native_buffer.h 
 * @since 10
 * @deprecated since 23
 */
typedef struct VkPhysicalDevicePresentationPropertiesOHOS { ... } VkPhysicalDevicePresentationPropertiesOHOS;
```

**废弃函数**：
```c
/**
 * @brief move to vk_ohos_native_buffer.h 
 * @since 10
 * @deprecated since 23
 */
VKAPI_ATTR VkResult VKAPI_CALL vkGetSwapchainGrallocUsageOHOS(...);

/**
 * @brief move to vk_ohos_native_buffer.h 
 * @since 10
 * @deprecated since 23
 */
VKAPI_ATTR VkResult VKAPI_CALL vkAcquireImageOHOS(...);

/**
 * @brief move to vk_ohos_native_buffer.h 
 * @since 10
 * @deprecated since 23
 */
VKAPI_ATTR VkResult VKAPI_CALL vkQueueSignalReleaseImageOHOS(...);
```

### 废弃时间线

| API | 从版本 | 废弃版本 | 说明 |
|-----|--------|---------|------|
| `vkGetSwapchainGrallocUsageOHOS` | OH 10 | OH 23 | 建议使用 `vkGetNativeBufferPropertiesOHOS` |
| `vkAcquireImageOHOS` | OH 10 | OH 23 | 建议使用标准 Vulkan 图像获取 |
| `vkQueueSignalReleaseImageOHOS` | OH 10 | OH 23 | 建议使用标准 Vulkan 队列操作 |
| `VkNativeBufferOHOS` | OH 10 | OH 23 | 建议使用 `VkImportNativeBufferInfoOHOS` |

### 迁移建议

**旧代码（OH 10 风格）**：
```c
// 已废弃的 API
VkNativeBufferOHOS bufferInfo = {
    .sType = VK_STRUCTURE_TYPE_NATIVE_BUFFER_OHOS,
    .handle = ohBufferHandle,
};
vkAcquireImageOHOS(device, image, fenceFd, semaphore, fence);
```

**新代码（OH 23+ 风格）**：
```c
// 推荐的新 API
VkImportNativeBufferInfoOHOS importInfo = {
    .sType = VK_STRUCTURE_TYPE_IMPORT_NATIVE_BUFFER_INFO_OHOS,
    .buffer = ohNativeBuffer,
};
// 使用标准 Vulkan 图像获取机制
```

## 5.4 与上游 API 的对比

### 平台扩展对比

| 平台 | 扩展名称 | Surface 创建函数 | 窗口句柄类型 |
|------|---------|----------------|-------------|
| **OpenHarmony** | `VK_OHOS_surface` | `vkCreateSurfaceOHOS` | `OHNativeWindow` |
| **Android** | `VK_ANDROID_surface` | `vkCreateAndroidSurfaceKH` | `ANativeWindow` |
| **Windows** | `VK_WIN32_surface` | `vkCreateWin32SurfaceKH` | `HWND` |
| **macOS** | `VK_MACOS_surface` | `vkCreateMacOSSurfaceMVK` | `NSView*` |
| **iOS** | `VK_IOS_surface` | `vkCreateIOSSurfaceMVK` | `UIView*` |

### 设计模式差异

**OpenHarmony 扩展设计**：
- 使用独立函数 `vkCreateSurfaceOHOS`
- 明确的 OHOS 命名空间
- 结构体包含 OH 特定类型

**上游标准设计**：
- 大部分平台使用 `vkCreate*SurfaceKH` 命名模式
- 使用联合体/扩展链机制
- 窗口句柄作为 `void*` 传递

## 5.5 API 使用注意事项

### 类型安全

1. **OHNativeWindow 转换**
   ```c
   // 确保使用正确的类型
   OHNativeWindow* window = reinterpret_cast<OHNativeWindow*>(nativeWindowPtr);
   ```

2. **结构体 sType 字段**
   ```c
   // 必须正确设置 sType
   VkSurfaceCreateInfoOHOS info = {
       .sType = VK_STRUCTURE_TYPE_SURFACE_CREATE_INFO_OHOS,  // 必须正确
       // ...
   };
   ```

### 内存管理

1. **AllocationCallbacks**
   ```c
   // 使用默认分配器
   vkCreateSurfaceOHOS(instance, &info, NULL, &surface);
   
   // 或使用自定义分配器
   VkAllocationCallbacks* allocator = GetCustomAllocator();
   vkCreateSurfaceOHOS(instance, &info, allocator, &surface);
   ```

2. **Surface 销毁**
   ```c
   vkDestroySurfaceKH(instance, surface, allocator);
   // 注意：使用标准 Vulkan 销毁函数
   ```

### 错误处理

```c
VkResult result = vkCreateSurfaceOHOS(instance, &info, NULL, &surface);
switch (result) {
    case VK_SUCCESS:
        // 成功
        break;
    case VK_ERROR_OUT_OF_HOST_MEMORY:
        // 主机内存不足
        break;
    case VK_ERROR_OUT_OF_DEVICE_MEMORY:
        // 设备内存不足
        break;
    case VK_ERROR_NATIVE_WINDOW_IN_USE_KHR:
        // NativeWindow 已被使用
        break;
    default:
        // 其他错误
        break;
}
```
