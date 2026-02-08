# Patch 分析

## 2.1 Patch 清单

### Patch 状态总结

| 状态 | 说明 |
|------|------|
| **Patch 文件数** | 0 |
| **Patch 目录数** | 0 |
| **适配方式** | 零 Patch（原生头文件 + 新增文件） |

### 分析结论

**Vulkan-Headers 库在 OpenHarmony 中没有使用任何 Patch 文件**。

这是因为：
1. Vulkan-Headers 是纯头文件库（header-only library）
2. Khronos 官方源码已支持多平台（Android、iOS、Windows、Mac 等）
3. OpenHarmony 采用与上游一致的平台适配方式
4. 适配通过新增 OH 特有头文件和构建配置实现

## 2.2 无 Patch 适配机制

### 适配原理

```
上游 Vulkan-Headers 源码 (无修改)
           │
           ▼
┌────────────────────────┐
│    OH 特有头文件        │
│  vulkan_ohos.h         │  ← 新增
│  vulkan_screen.h       │  ← 新增  
│  vk_ohos_native_buffer.h│ ← 新增
└────────────────────────┘
           │
           ▼
┌────────────────────────┐
│    BUILD.gn 配置        │
│  VK_USE_PLATFORM_OHOS  │  ← 新增定义
└────────────────────────┘
           │
           ▼
    OpenHarmony 适配版本
```

### 适配优势

| 优势 | 说明 |
|------|------|
| **维护简单** | 无需维护 Patch 合并冲突 |
| **升级便捷** | 上游版本更新时直接同步即可 |
| **兼容性** | 保持与上游完全一致 |
| **标准化** | OH 扩展遵循 Vulkan 扩展规范 |

## 2.3 OH 特有文件

### 新增头文件清单

| 文件名 | 路径 | 功能 |
|--------|------|------|
| `vulkan_ohos.h` | include/vulkan/vulkan_ohos.h | OHOS Surface 和 Native Buffer 扩展 |
| `vulkan_screen.h` | include/vulkan/vulkan_screen.h | QNX Screen 平台扩展 |
| `vk_ohos_native_buffer.h` | include/vulkan/vk_ohos_native_buffer.h | 专用 Native Buffer 操作 |

### vulkan_ohos.h 详细分析

**文件路径**: `include/vulkan/vulkan_ohos.h`

**修改目的**: 定义 OpenHarmony 平台专用的 Vulkan 扩展接口

**包含的扩展**:

#### 1. VK_OHOS_surface 扩展

```c
#define VK_OHOS_surface 1
#define VK_OHOS_SURFACE_SPEC_VERSION      1
#define VK_OHOS_SURFACE_EXTENSION_NAME    "VK_OHOS_surface"

typedef struct VkSurfaceCreateInfoOHOS {
    VkStructureType             sType;
    const void*                 pNext;
    VkSurfaceCreateFlagsOHOS    flags;
    OHNativeWindow*             window;  // OH NativeWindow 句柄
} VkSurfaceCreateInfoOHOS;

VkResult vkCreateSurfaceOHOS(
    VkInstance                                  instance,
    const VkSurfaceCreateInfoOHOS*              pCreateInfo,
    const VkAllocationCallbacks*                pAllocator,
    VkSurfaceKHR*                               pSurface);
```

**用途**: 创建与 OH NativeWindow 关联的 Vulkan Surface

#### 2. VK_OHOS_native_buffer 扩展

```c
#define VK_OHOS_native_buffer 1
#define VK_OHOS_NATIVE_BUFFER_SPEC_VERSION 1
#define VK_OHOS_NATIVE_BUFFER_EXTENSION_NAME "VK_OHOS_native_buffer"

// Gralloc 使用查询
typedef VkResult (VKAPI_PTR *PFN_vkGetSwapchainGrallocUsageOHOS)(
    VkDevice device, VkFormat format, VkImageUsageFlags imageUsage, 
    uint64_t* grallocUsage);

// 图像获取
typedef VkResult (VKAPI_PTR *PFN_vkAcquireImageOHOS)(
    VkDevice device, VkImage image, int32_t nativeFenceFd, 
    VkSemaphore semaphore, VkFence fence);

// 图像释放
typedef VkResult (VKAPI_PTR *PFN_vkQueueSignalReleaseImageOHOS)(
    VkQueue queue, uint32_t waitSemaphoreCount, const VkSemaphore* pWaitSemaphores, 
    VkImage image, int32_t* pNativeFenceFd);
```

**用途**: 与 OH Gralloc 系统互操作，管理共享内存

#### 3. VK_OHOS_external_memory 扩展

```c
#define VK_OHOS_external_memory 1
#define VK_OHOS_EXTERNAL_MEMORY_SPEC_VERSION 1
#define VK_OHOS_EXTERNAL_MEMORY_EXTENSION_NAME "VK_OHOS_external_memory"

// OH NativeBuffer 导入
typedef struct VkImportNativeBufferInfoOHOS {
    VkStructureType            sType;
    const void*                pNext;
    struct OH_NativeBuffer*    buffer;
} VkImportNativeBufferInfoOHOS;

// 获取 Native Buffer 属性
typedef VkResult (VKAPI_PTR *PFN_vkGetNativeBufferPropertiesOHOS)(
    VkDevice device, const struct OH_NativeBuffer* buffer, 
    VkNativeBufferPropertiesOHOS* pProperties);

// 从内存获取 Native Buffer
typedef VkResult (VKAPI_PTR *PFN_vkGetMemoryNativeBufferOHOS)(
    VkDevice device, const VkMemoryGetNativeBufferInfoOHOS* pInfo, 
    struct OH_NativeBuffer** pBuffer);
```

**用途**: 支持 Vulkan 与 OH Native Buffer 系统之间的内存共享

### vk_ohos_native_buffer.h 详细分析

**文件路径**: include/vulkan/vk_ohos_native_buffer.h

**版权**: Huawei Device Co., Ltd.

**功能**: 与 vulkan_ohos.h 中的 VK_OHOS_native_buffer 扩展重复定义

**说明**: 该文件与 vulkan_ohos.h 中的 native_buffer 扩展内容相同，可能是历史遗留或为了特定用途的独立文件。

### vulkan_screen.h 详细分析

**文件路径**: include/vulkan/vulkan_screen.h

**平台**: QNX Screen

**包含的扩展**:

| 扩展名称 | 功能 |
|---------|------|
| `VK_QNX_screen_surface` | QNX Screen Surface 创建 |
| `VK_QNX_external_memory_screen_buffer` | QNX Screen Buffer 外部内存 |

**说明**: 该文件定义的是 QNX 平台扩展，不是 OpenHarmony 特有扩展。在 OH 中可能用于特定的嵌入式场景或测试环境。

## 2.4 构建配置适配

### BUILD.gn 中的适配

```gn
config("vulkan_headers_config") {
  include_dirs = [ "include" ]
  defines = []

  if (is_ohos) {
    defines += [ "VK_USE_PLATFORM_OHOS" ]
  }
  // ... 其他平台配置
}
```

**适配说明**:
- `is_ohos` 变量标识 OpenHarmony 平台
- `VK_USE_PLATFORM_OHOS` 宏启用 OHOS 平台支持
- 当该宏被定义时，vulkan_ohos.h 中的扩展接口被激活

### 源文件配置

```gn
ohos_static_library("vulkan_headers") {
  sources = [
    // ... 上游标准头文件
    "include/vulkan/vulkan_ohos.h",      // OH 特有
    "include/vulkan/vulkan_screen.h",     // QNX 平台
  ]
  public_configs = [ ":vulkan_headers_config" ]
}
```

## 2.5 升级建议

### 上游版本同步策略

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1 | 检查上游 Release | 访问 https://github.com/KhronosGroup/Vulkan-Headers/releases |
| 2 | 同步上游文件 | 复制新版本的头文件到 OH 仓库 |
| 3 | 保留 OH 特有文件 | vulkan_ohos.h, vulkan_screen.h, vk_ohos_native_buffer.h |
| 4 | 更新 BUILD.gn | 如有新头文件，添加到 sources 列表 |
| 5 | 验证编译 | 编译相关依赖模块 |
| 6 | 运行测试 | 执行 Vulkan CTS 测试 |

### OH 特有文件维护

**vulkan_ohos.h**:
- 该文件由 OH 团队维护
- 如上游发布类似扩展，考虑合并或保持独立
- 确保与 OH Native Window/Native Buffer API 兼容性

**vulkan_screen.h**:
- 该文件来自上游 Khronos 标准
- 确认在 OH 中的实际使用场景
- 如无需 QNX 支持，可考虑移除（需评估影响）

**vk_ohos_native_buffer.h**:
- 与 vulkan_ohos.h 内容重复
- 建议统一，避免维护两份相同代码

### 回归风险评估

| 项目 | 风险等级 | 说明 |
|------|---------|------|
| 上游版本同步 | 低 | 纯头文件库，无逻辑变更 |
| OH 扩展兼容性 | 中 | 需验证 OH 特有扩展与新版本的兼容性 |
| 构建配置 | 低 | BUILD.gn 通常无需修改 |
| 依赖模块 | 中 | 需验证 skia、graphic_2d 等模块的兼容性 |

## 2.6 与其他库的 Patch 对比

### 适配方式对比

| 库名称 | Patch 数量 | 适配方式 | 复杂度 |
|-------|-----------|---------|-------|
| vulkan-headers | 0 | 新增头文件 + 构建配置 | 低 |
| vulkan-loader | 多 | 大量 Patch | 高 |
| curl | 多 | 大量 Patch | 高 |
| openssl | 多 | 大量 Patch | 高 |

### 结论

Vulkan-Headers 是 OH 第三方库中**适配最简单的库之一**，主要因为：
1. 纯头文件库，无编译复杂性
2. 上游已支持多平台
3. OH 扩展遵循 Vulkan 标准规范
