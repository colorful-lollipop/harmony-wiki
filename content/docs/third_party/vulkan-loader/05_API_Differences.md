# 05 - API/接口差异

## 5.1 OH 新增的 Vulkan 扩展

### VK_OHOS_surface 扩展

**扩展名**: `VK_OHOS_surface`  
**功能**: 创建与 OHNativeWindow 绑定的 Vulkan Surface  
**头文件**: `vulkan/vulkan_ohos.h`

**新增 API**:
```cpp
// 创建 OHOS Surface
VkResult vkCreateSurfaceOHOS(
    VkInstance instance,
    const VkSurfaceCreateInfoOHOS* pCreateInfo,
    const VkAllocationCallbacks* pAllocator,
    VkSurfaceKHR* pSurface
);
```

**数据结构**:
```cpp
typedef struct VkSurfaceCreateInfoOHOS {
    VkStructureType              sType;      // VK_STRUCTURE_TYPE_SURFACE_CREATE_INFO_OHOS
    const void*                  pNext;
    VkSurfaceCreateFlagsOHOS     flags;
    struct OHNativeWindow*       window;     // OH 原生窗口指针
} VkSurfaceCreateInfoOHOS;
```

**使用场景**: 将 Vulkan 渲染与 OHOS 窗口系统对接

---

### VK_OHOS_native_buffer 扩展

**扩展名**: `VK_OHOS_native_buffer`  
**功能**: 操作 OHOS 原生图形缓冲区 (Gralloc)

**新增 API**:
```cpp
// 获取 Swapchain 的 Gralloc 使用标志
VkResult vkGetSwapchainGrallocUsageOHOS(
    VkDevice device,
    VkFormat format,
    VkImageUsageFlags imageUsage,
    uint64_t* grallocUsage
);

// 获取 Image
VkResult vkAcquireImageOHOS(
    VkDevice device,
    VkImage image,
    int nativeFenceFd,
    VkSemaphore semaphore,
    VkFence fence
);

// 队列信号释放 Image
VkResult vkQueueSignalReleaseImageOHOS(
    VkQueue queue,
    uint32_t waitSemaphoreCount,
    const VkSemaphore* pWaitSemaphores,
    VkImage image,
    int* pNativeFenceFd
);
```

**使用场景**: 与 OHOS 图形缓冲区系统高效共享内存

---

### VK_OHOS_external_memory 扩展

**扩展名**: `VK_OHOS_external_memory`  
**功能**: 导入/导出 OHOS 原生缓冲区作为 Vulkan 内存

**新增 API**:
```cpp
// 获取原生缓冲区属性
VkResult vkGetNativeBufferPropertiesOHOS(
    VkDevice device,
    const void* buffer,           // OHOS native buffer handle
    VkNativeBufferPropertiesOHOS* pProperties
);

// 从内存获取原生缓冲区
VkResult vkGetMemoryNativeBufferOHOS(
    VkDevice device,
    const VkMemoryGetNativeBufferInfoOHOS* pInfo,
    void** pBuffer                // OHOS native buffer handle
);
```

**数据结构**:
```cpp
typedef struct VkNativeBufferPropertiesOHOS {
    VkStructureType sType;
    void*           pNext;
    VkDeviceSize    allocationSize;
    uint32_t        memoryTypeBits;
} VkNativeBufferPropertiesOHOS;

typedef struct VkMemoryGetNativeBufferInfoOHOS {
    VkStructureType sType;
    const void*     pNext;
    VkDeviceMemory  memory;
} VkMemoryGetNativeBufferInfoOHOS;
```

**使用场景**: 相机、视频解码器等与 Vulkan 之间零拷贝共享数据

---

## 5.2 行为变更的 API

### 动态库加载 (dlopen)

**标准 Linux 行为**:
```cpp
handle = dlopen(libPath, RTLD_LAZY | RTLD_LOCAL);
```

**OHOS 行为**:
```cpp
// 优先从 passthrough namespace 加载
Dl_namespace ns_ps;
if (!dlns_get("passthrough", &ns_ps)) {
    handle = dlopen_ns(&ns_ps, libPath, RTLD_LAZY | RTLD_LOCAL);
}
// 回退到默认
if (!handle) {
    handle = dlopen(libPath, RTLD_LAZY | RTLD_LOCAL);
}
```

**差异说明**: OHOS 优先尝试从 `passthrough` namespace 加载，允许应用访问系统驱动库

---

### 环境变量获取

**标准 Linux 行为**:
```cpp
char* value = getenv(name);
```

**OHOS 行为**:
```cpp
CachedHandle g_Handle = CachedParameterCreate(name, "");
int changed = 0;
const char* res = CachedParameterGetChanged(g_Handle, &changed);
```

**差异说明**: OHOS 使用系统参数 (`syspara`) 替代传统环境变量，更安全且支持变更监听

**受影响的环境变量**:
- `debug.graphic.debug_layer` - 调试 Layer 名称
- `debug.graphic.debug_hap` - 调试 HAP 包名
- `debug.graphic.system_layer_flag` - 系统 Layer 标志
- `debug.graphic.vklayer_json_path` - Layer JSON 路径

---

### 日志输出

**标准 Linux 行为**:
```cpp
fputs(msg, stderr);
```

**OHOS 行为**:
```cpp
// 输出到 HiLog
HILOG_DEBUG(LOG_CORE, "%{public}s", msg);
// 或
HILOG_ERROR(LOG_CORE, "%{public}s", msg);
```

**差异说明**: OHOS 使用 HiLog 系统统一收集日志，可通过 `hilog` 命令查看

**查看日志**:
```bash
hilog | grep VulkanLoader
```

---

### 配置文件搜索路径

**标准 Linux (XDG) 行为**:
- 驱动配置: `$XDG_CONFIG_DIRS/vulkan/icd.d/`
- Layer 配置: `$XDG_DATA_DIRS/vulkan/implicit_layer.d/`

**OHOS 行为**:
- 驱动配置:
  - `/vendor/etc/vulkan/icd.d/` (推荐)
  - `/system/etc/vulkan/icd.d/`
  - `/data/vulkan/icd.d/`
  
- Layer 配置:
  - `/system/etc/vulkan/implicit_layer.d/`
  - `/system/etc/vulkan/explicit_layer.d/`
  - `/data/vulkan/implicit_layer.d/`
  - `/data/vulkan/explicit_layer.d/`

**差异说明**: OHOS 不使用 XDG 标准路径，使用固定的系统路径

---

## 5.3 废弃或禁用的功能

### 不支持的窗口系统扩展

| 扩展 | 状态 | 说明 |
|------|------|------|
| `VK_KHR_xcb_surface` | ❌ 不支持 | XCB 窗口系统 |
| `VK_KHR_xlib_surface` | ❌ 不支持 | Xlib 窗口系统 |
| `VK_KHR_wayland_surface` | ❌ 不支持 | Wayland 窗口系统 |
| `VK_KHR_win32_surface` | ❌ 不支持 | Windows 窗口系统 |

**替代方案**: 使用 `VK_OHOS_surface`

### 不支持的 Linux 特性

| 特性 | 状态 | 说明 |
|------|------|------|
| XDG 配置路径 | ❌ 不使用 | 使用固定系统路径 |
| HOME 目录配置 | ❌ 不使用 | 沙箱环境不使用 HOME |
| `secure_getenv` | ❌ 不使用 | 使用 `syspara` 替代 |

---

## 5.4 OH 特有功能

### Bundle 管理器集成

**功能**: 验证应用身份和权限  
**接口**: `openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.h`

```cpp
// 初始化 Bundle 信息
bool InitBundleInfo(char* debugHapName);

// 检查应用是否为调试版本
bool CheckAppProvisionTypeIsDebug();

// 获取调试 Layer 库路径
char* GetDebugLayerLibPath(
    const struct loader_instance* inst,
    VkSystemAllocationScope allocation_scope
);
```

**使用场景**: 调试 Layer 加载时的安全验证

---

### 调试 Layer 控制

**OHOS 特有环境变量/参数**:

| 参数名 | 用途 | 示例值 |
|--------|------|--------|
| `debug.graphic.debug_layer` | 启用指定调试 Layer | `VK_LAYER_OHOS_debug` |
| `debug.graphic.debug_hap` | 指定调试 HAP 包名 | `com.example.app` |
| `debug.graphic.system_layer_flag` | 允许使用系统 Layer | `1` |
| `debug.graphic.vklayer_json_path` | 自定义 Layer JSON 路径 | `/data/layers/` |

**使用方式**:
```bash
hdc shell param set debug.graphic.debug_layer my_layer
hdc shell param set debug.graphic.debug_hap com.example.myapp
```

---

## 5.5 扩展启用条件

### 自动启用的扩展

以下扩展在 `vkCreateInstance` 时自动可用（如果驱动支持）:

- `VK_OHOS_surface`
- `VK_OHOS_native_buffer`
- `VK_OHOS_external_memory`

### 需要显式启用的扩展

在 `VkInstanceCreateInfo::ppEnabledExtensionNames` 中指定:

```cpp
const char* extensions[] = {
    VK_OHOS_SURFACE_EXTENSION_NAME,
    VK_KHR_SURFACE_EXTENSION_NAME,
};

VkInstanceCreateInfo createInfo = {};
createInfo.enabledExtensionCount = 2;
createInfo.ppEnabledExtensionNames = extensions;
```

---

## 5.6 平台宏定义

### 编译时检测

```cpp
#if defined(VK_USE_PLATFORM_OHOS)
    // OHOS 特有代码
    VkSurfaceCreateInfoOHOS createInfo = {};
    vkCreateSurfaceOHOS(instance, &createInfo, nullptr, &surface);
#elif defined(VK_USE_PLATFORM_WIN32_KHR)
    // Windows 代码
#elif defined(VK_USE_PLATFORM_XCB_KHR)
    // Linux XCB 代码
#endif
```

### 运行时检测

```cpp
// 检查扩展是否可用
VkResult result = vkEnumerateInstanceExtensionProperties(
    nullptr, &count, nullptr
);
// 在返回的扩展列表中查找 VK_OHOS_SURFACE_EXTENSION_NAME
```

---

## 5.7 API 版本兼容性

### Vulkan API 版本

| 项目 | 值 |
|------|-----|
| Loader 版本 | 1.4.309 |
| 支持的最小版本 | 1.0 |
| 支持的最大版本 | 1.4（取决于驱动） |

### 扩展版本

| 扩展 | 版本 | 引入版本 |
|------|------|----------|
| VK_OHOS_surface | 1 | OH 3.x |
| VK_OHOS_native_buffer | 1 | OH 3.x |
| VK_OHOS_external_memory | 1 | OH 3.x |

---

## 5.8 迁移指南

### 从标准 Vulkan 迁移到 OHOS Vulkan

1. **Surface 创建**
   ```cpp
   // 标准（如 Android）
   vkCreateAndroidSurfaceKHR(instance, &createInfo, nullptr, &surface);
   
   // OHOS
   vkCreateSurfaceOHOS(instance, &createInfo, nullptr, &surface);
   ```

2. **环境变量**
   ```cpp
   // 标准
   const char* layer = getenv("VK_INSTANCE_LAYERS");
   
   // OHOS（通过系统参数）
   hdc shell param set debug.graphic.debug_layer my_layer
   ```

3. **日志查看**
   ```cpp
   // 标准 - 查看 stderr
   // OHOS - 使用 hilog
   hdc shell hilog | grep Vulkan
   ```

---

*文档版本：v1.0*
*最后更新：2026-02-07*
