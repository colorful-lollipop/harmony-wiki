# 02 - Patch 与适配分析

## 2.1 Patch 文件清单

### 传统 Patch 文件

**结论：本项目没有传统的 `.patch` 补丁文件。**

搜索范围：整个仓库  
搜索命令：`find . -name "*.patch" -o -name "patches" -type d`  
搜索结果：无匹配

### 适配方式说明

Vulkan-Loader 采用**代码级集成**的方式进行 OpenHarmony 适配，而非传统的外部 Patch 文件。这种方式的优点：

1. **维护简单**：升级上游版本时无需重新应用 Patch
2. **结构清晰**：OH 特有代码集中在 `openharmony/` 目录
3. **版本控制友好**：与上游代码变更易于区分

---

## 2.2 OH 特有代码分析

### 2.2.1 Bundle 管理器帮助类

**文件位置**：
- `openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.h`
- `openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.cpp`

**原始问题**：
OpenHarmony 应用运行在沙箱环境中，需要验证应用身份和权限才能加载调试 Layer。传统的环境变量方式无法满足安全需求。

**修改内容**：

| 函数 | 功能 |
|------|------|
| `InitBundleInfo()` | 初始化 Bundle 信息，验证调试 HAP 包名是否匹配 |
| `CheckAppProvisionTypeIsDebug()` | 检查应用是否为调试版本（非 release 版本） |
| `GetDebugLayerLibPath()` | 获取调试 Layer 库的完整路径 |
| `VKBundleMgrHelper::Connect()` | 连接到 BundleManager 系统服务 |
| `VKBundleMgrHelper::GetBundleInfoForSelf()` | 获取当前应用的 Bundle 信息 |

**关键代码**：
```cpp
// 使用 OHOS Bundle 框架服务验证应用身份
auto vkBundleMgrHelper = OHOS::DelayedSingleton<OHOS::AppExecFwk::VKBundleMgrHelper>::GetInstance();
if (vkBundleMgrHelper->GetBundleInfoForSelf(
    OHOS::AppExecFwk::GetBundleInfoFlag::GET_BUNDLE_INFO_WITH_APPLICATION, 
    vkBundleMgrHelper->g_bundleInfo) == OHOS::ERR_OK) {
    if (vkBundleMgrHelper->g_bundleInfo.name == debugHap) {
        return true;  // 包名匹配，允许加载调试 Layer
    }
}
```

**OH 价值**：
- ✅ 实现应用层调试 Layer 的安全加载
- ✅ 防止 release 版本应用加载调试 Layer
- ✅ 验证 HAP 包名，防止恶意应用伪造调试 Layer

**回归风险**：
- 升级时需检查 BundleManager 接口变更
- `GetBundleInfoForSelf` 等接口签名变化会影响编译

---

### 2.2.2 日志系统适配

**文件位置**：
- `openharmony/loader_hilog.h`

**原始问题**：
上游代码使用 `stderr` 输出日志，在 OpenHarmony 中需要集成到系统日志服务 HiLog 中。

**修改内容**：
```cpp
#include <hilog/log.h>

#undef LOG_DOMAIN
#undef LOG_TAG
#define LOG_DOMAIN 0xD001405  // OH 分配的日志域
#define LOG_TAG "VulkanLoader"

#define VKHILOGD(fmt, ...) HILOG_DEBUG(LOG_CORE, fmt, ##__VA_ARGS__)
#define VKHILOGE(fmt, ...) HILOG_ERROR(LOG_CORE, fmt, ##__VA_ARGS__)

// 桥接函数：将 loader 日志发送到 HiLog
void OpenHarmonyLog(VkFlags log_type, char *log_msg) {
    if (log_type & VULKAN_LOADER_ERROR_BIT) {
        VKHILOGE("%{public}s", log_msg);
    } else {
        VKHILOGD("%{public}s", log_msg);
    }
}
```

**使用位置**：
- `loader/log.c` (line 277-282)：替换 `stderr` 输出

**OH 价值**：
- ✅ 日志统一收集到系统 HiLog
- ✅ 可通过 `hilog` 命令查看
- ✅ 支持日志级别过滤

**回归风险**：低（仅日志输出方式变更）

---

### 2.2.3 平台抽象层修改

**文件位置**：
- `loader/vk_loader_platform.h`

**修改 1：OHOS 特有 dlopen 实现**

**原始问题**：
OpenHarmony 使用 namespace 机制隔离系统库和应用库，需要优先从 `passthrough` namespace 加载 GPU 驱动。

**修改内容**：
```cpp
#if defined(VK_USE_PLATFORM_OHOS)
static inline loader_platform_dl_handle loader_platform_open_library(const char *libPath) {
    void *handle = NULL;
    Dl_namespace ns_ps;
    // 优先尝试从 passthrough namespace 加载（访问系统驱动）
    if (!dlns_get("passthrough", &ns_ps)) {
        handle = dlopen_ns(&ns_ps, libPath, LOADER_DLOPEN_MODE);
    }
    // 回退到默认 dlopen
    if (!handle) {
        handle = dlopen(libPath, LOADER_DLOPEN_MODE);
    }
    return handle;
}
```

**OH 价值**：
- ✅ 支持 OHOS namespace 机制
- ✅ 应用可以访问系统 GPU 驱动
- ✅ 安全隔离与功能兼容并存

**修改 2：定义 VK_USE_PLATFORM_OHOS 宏**

在 `BUILD.gn` 中定义：
```gn
defines += [
    "VK_USE_PLATFORM_OHOS",
]
```

此宏启用所有 OHOS 特有的代码路径。

---

### 2.2.4 环境变量处理

**文件位置**：
- `loader/loader_environment.c`

**原始问题**：
OpenHarmony 出于安全考虑，限制了传统环境变量的使用，改用 `syspara` (系统参数) 机制。

**修改内容**：
```cpp
#if defined(__OHOS__)
char *loader_getenv(const char *name, const struct loader_instance *inst) {
    // 使用 OHOS 系统参数 API
    CachedHandle g_Handle = CachedParameterCreate(name, "");
    int changed = 0;
    const char *res = CachedParameterGetChanged(g_Handle, &changed);
    loader_log(inst, VULKAN_LOADER_DEBUG_BIT | VULKAN_LOADER_INFO_BIT, 0, 
               "loader_getenv name:%s, res:%s", name, res);
    if (res == NULL || res[0] == '\0') {
        return NULL;
    }
    return (char *)res;
}
#else
    return getenv(name);
#endif
```

**OH 价值**：
- ✅ 符合 OHOS 安全模型
- ✅ 使用系统参数替代环境变量
- ✅ 支持参数变更监听

**回归风险**：
- 需确保 `libbegetutil` (提供 syspara) 可用

---

### 2.2.5 调试 Layer 加载逻辑

**文件位置**：
- `loader/loader.c` (line 3204-3444)

**原始问题**：
需要支持应用开发者在 OHOS 上动态加载调试 Layer，同时确保安全（仅调试版本应用可用）。

**修改内容**：

**1. OHOS 特有环境变量定义**：
```cpp
#if defined(__OHOS__)
    char *debug_layer_name = loader_secure_getenv("debug.graphic.debug_layer", inst);
    char *debug_hap_name = loader_secure_getenv("debug.graphic.debug_hap", inst);
    char *system_debug_hap_name = loader_secure_getenv("debug.graphic.system_layer_flag", inst);
    char *debug_layer_json_path = NULL;
```

**2. 调试 Layer 搜索路径**：
```cpp
    const char default_json_path[] = "/data/storage/el2/base/haps/entry/files/";
    const char external_system_json_path[] = "/vendor/etc/vulkan/debuglayer/";
```

**3. Bundle 验证**：
```cpp
    if (NULL != debug_layer_name && '\0' != debug_layer_name[0] && InitBundleInfo(debug_hap_name)) {
        currentProcessEnableDebugLayer = true;
        // 根据应用类型选择路径
        if (use_system_layer && CheckAppProvisionTypeIsDebug()) {
            strncpy(debug_layer_json_path, external_system_json_path, debug_layer_json_path_len);
        } else {
            strncpy(debug_layer_json_path, default_json_path, debug_layer_json_path_len);
        }
    }
```

**OH 价值**：
- ✅ 支持开发者调试 Vulkan 应用
- ✅ 仅允许调试版本应用加载 Layer
- ✅ 包名验证防止滥用

---

### 2.2.6 WSI (Window System Integration) 扩展

**文件位置**：
- `loader/wsi.c` (line 1233-1279)
- `loader/generated/vk_loader_extensions.c` (多处)
- `loader/trampoline.c` (line 1130-1151)

**扩展列表**：

| 扩展名 | 功能 |
|--------|------|
| `VK_OHOS_surface` | 创建 OHOS 平台 Surface |
| `VK_OHOS_native_buffer` | 原生缓冲区操作 (Gralloc) |
| `VK_OHOS_external_memory` | 外部内存导入/导出 |

**核心实现**：
```cpp
#if defined(VK_USE_PLATFORM_OHOS)

// Trampoline 函数：应用层调用
LOADER_EXPORT VKAPI_ATTR VkResult VKAPI_CALL vkCreateSurfaceOHOS(
    VkInstance instance,
    const VkSurfaceCreateInfoOHOS *pCreateInfo,
    const VkAllocationCallbacks *pAllocator,
    VkSurfaceKHR *pSurface) {
    // 转发到 ICD 或 Layer
    return loader_inst->disp->layer_inst_disp.CreateSurfaceOHOS(...);
}

// Terminator 函数：Layer 链终点
VKAPI_ATTR VkResult VKAPI_CALL terminator_CreateSurfaceOHOS(
    VkInstance instance,
    const VkSurfaceCreateInfoOHOS *pCreateInfo,
    const VkAllocationCallbacks *pAllocator,
    VkSurfaceKHR *pSurface) {
    // 检查扩展是否启用
    if (!loader_inst->wsi_ohos_surface_enabled) {
        return VK_ERROR_EXTENSION_NOT_PRESENT;
    }
    // 创建 ICD Surface
    VkIcdSurfaceOHOS *pIcdSurface = loader_instance_heap_alloc(...);
    pIcdSurface->base.platform = VK_ICD_WSI_PLATFORM_OHOS;
    pIcdSurface->window = pCreateInfo->window;
    *pSurface = (VkSurfaceKHR)(uintptr_t)pIcdSurface;
    return VK_SUCCESS;
}

#endif  // VK_USE_PLATFORM_OHOS
```

**OH 价值**：
- ✅ Vulkan 应用可以在 OHOS 窗口系统上渲染
- ✅ 与 swapchain_layer 配合实现图像呈现
- ✅ 支持原生缓冲区 (NativeBuffer) 高效内存共享

---

### 2.2.7 排除标准 Linux 路径

**文件位置**：
- `loader/loader.c` (line 3145, 3154)

**修改内容**：
```cpp
// 不使用 XDG 标准路径（OHOS 不支持）
#if !defined(__Fuchsia__) && !defined(__QNX__) && !defined(__OHOS__)
    if (NULL == xdg_config_dirs || '\0' == xdg_config_dirs[0]) {
        xdg_config_dirs = FALLBACK_CONFIG_DIRS;
    }
#endif

// 不使用 HOME 目录（OHOS 沙箱机制不同）
#if !defined(__Fuchsia__) && !defined(__QNX__) && !defined(__OHOS__)
    if (NULL == xdg_data_dirs || '\0' == xdg_data_dirs[0]) {
        xdg_data_dirs = FALLBACK_DATA_DIRS;
    }
#endif
```

**OH 价值**：
- ✅ 避免在非标准路径查找配置
- ✅ 符合 OHOS 沙箱安全模型

---

## 2.3 按功能分类的 OH 适配

### 驱动加载相关

| 文件 | 修改点 | 目的 |
|------|--------|------|
| `loader/loader.c` | 排除 XDG 路径 | 使用 OHOS 标准路径 |
| `loader/vk_loader_platform.h` | namespace dlopen | 支持系统驱动访问 |

### Layer 调试相关

| 文件 | 修改点 | 目的 |
|------|--------|------|
| `openharmony/bundle_mgr_helper/*` | Bundle 验证 | 安全加载调试 Layer |
| `loader/loader.c` | 调试 Layer 逻辑 | 应用层调试支持 |

### 系统集成相关

| 文件 | 修改点 | 目的 |
|------|--------|------|
| `loader/loader_environment.c` | syspara 集成 | 系统参数替代环境变量 |
| `openharmony/loader_hilog.h` | HiLog 桥接 | 系统日志集成 |
| `loader/log.c` | 日志输出 | 替换 stderr |

### WSI 相关

| 文件 | 修改点 | 目的 |
|------|--------|------|
| `loader/wsi.c` | VK_OHOS_surface | Surface 创建 |
| `loader/generated/*` | 扩展分发 | 扩展函数指针管理 |

---

## 2.4 升级建议

### 可以推向上游的修改

以下修改具有通用性，理论上可以推向上游（需与 Khronos 协商）：

1. **代码结构优化**：将平台相关代码进一步抽象
2. **文档完善**：增加平台适配指南

### OH 特有的修改（保留）

以下修改与 OHOS 强相关，必须保留：

1. **Bundle 管理器集成** - OHOS 应用框架特有
2. **HiLog 日志系统** - OHOS 特有
3. **SysParam 环境变量** - OHOS 安全模型特有
4. **VK_OHOS_* 扩展** - OHOS WSI 特有
5. **namespace dlopen** - OHOS 库加载机制特有

### 升级检查清单

升级上游版本时，检查以下 OH 特有代码是否受影响：

- [ ] `openharmony/` 目录文件是否完整
- [ ] `__OHOS__` / `VK_USE_PLATFORM_OHOS` 条件编译是否正常
- [ ] 生成代码中 VK_OHOS_* 扩展是否保留
- [ ] BUILD.gn 是否需要同步更新
- [ ] 测试用例是否能通过

---

## 2.5 总结

| 统计项 | 数值 |
|--------|------|
| 传统 Patch 文件 | 0 个 |
| OH 特有源文件 | 4 个（openharmony/ 目录） |
| 修改的原始文件 | 5+ 个 |
| 条件编译块 | 20+ 处 |
| WSI 扩展 | 3 个 |

**核心适配价值**：
1. 完整的 WSI 支持（VK_OHOS_surface 等）
2. 安全的调试 Layer 加载机制
3. 深度系统集成（日志、Bundle、SysParam）
4. 符合 OHOS 安全模型

---

*文档版本：v1.0*
*最后更新：2026-02-07*
