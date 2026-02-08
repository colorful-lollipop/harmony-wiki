# 05_API_Differences - API/接口差异

## 概述

本文档记录 **Mesa3D 在 OpenHarmony 中的 API 差异**，包括新增的 OH 特有接口、行为变更和禁用功能。

---

## 1. OH 特有 API

### 1.1 Vulkan OHOS 扩展

#### 头文件

```c
#include <vulkan/vulkan_ohos.h>
```

#### 新增结构体

```c
// Surface 创建信息
typedef struct VkSurfaceCreateInfoOHOS {
    VkStructureType               sType;
    const void*                   pNext;
    VkSurfaceCreateFlagsKHR       flags;
    OHNativeWindow*               window;
} VkSurfaceCreateInfoOHOS;
```

#### 新增枚举

```c
// Vulkan 1.0 扩展
#define VK_OHOS_surface 1
#define VK_OHOS_NATIVE_BUFFER_SPEC_VERSION 1
#define VK_OHOS_EXTERNAL_MEMORY_SPEC_VERSION 1

// Surface 填充模式
typedef enum VkOHOSSurfaceFillMode {
    VK_OHOS_SURFACE_FILL_MODE_FILL = 0,
    VK_OHOS_SURFACE_FILL_MODE_FOLLOW_DISPLAY_ROTATION = 1,
} VkOHOSSurfaceFillMode;

// 扩展结构体
#define VK_STRUCTURE_TYPE_SURFACE_CREATE_INFO_OHOS \
    VK_STRUCTURE_TYPE_SURFACE_CREATE_INFO_ANDROID_KHR + 1

#define VK_STRUCTURE_TYPE_NATIVE_BUFFER_CREATE_INFO_OHOS \
    VK_STRUCTURE_TYPE_NATIVE_BUFFER_CREATE_INFO_ANDROID_KHR + 1
```

#### 新增函数

```c
// 创建 OHOS Surface
VKAPI_ATTR VkResult vkCreateSurfaceOHOS(
    VkInstance                                  instance,
    const VkSurfaceCreateInfoOHOS*              pCreateInfo,
    const VkAllocationCallbacks*                pAllocator,
    VkSurfaceKHR*                               pSurface);

// OHOS 原生缓冲区导出
typedef struct VkExportNativeBufferInfoOHOS {
    VkStructureType            sType;
    const void*                pNext;
    OHNativeWindowBuffer*      buffer;
    uint32_t                  acquireFence;
} VkExportNativeBufferInfoOHOS;
```

### 1.2 EGL OHOS 平台扩展

#### 查询扩展

```c
// 检查 EGL OHOS 平台支持
const char* extensions = eglQueryString(EGL_NO_DISPLAY, EGL_EXTENSIONS);
if (strstr(extensions, "EGL_OHOS_platform_surface")) {
    // OHOS 平台可用
}
```

#### 平台类型

```c
// EGL 平台类型
#define _EGL_PLATFORM_OHOS 0xXXXF

// 使用 OHOS 平台获取 EGL 显示
EGLDisplay display = eglGetPlatformDisplayEXT(
    EGL_PLATFORM_OHOS_EXT,
    nativeDisplay,
    NULL
);
```

### 1.3 OH NativeWindow 集成

#### 头文件

```c
#include <surface.h>
#include <native_window.h>
```

#### 新增类型

```c
// OH NativeWindow 句柄
typedef struct OHNativeWindow OHNativeWindow;
typedef struct OHNativeWindowBuffer OHNativeWindowBuffer;
```

#### 新增函数

```c
// 创建 NativeWindow
OHNativeWindow* OH_NativeWindow_CreateNativeWindow(void* window);

// 获取窗口尺寸
int32_t OH_NativeWindow_GetNativeWindowSize(
    OHNativeWindow* window,
    int32_t* width,
    int32_t* height);

// 配置缓冲区
int32_t OH_NativeWindow_NativeWindowHandleOpt(
    OHNativeWindow* window,
    int code,
    int64_t value1,
    int64_t value2);

// 释放 NativeWindow
void OH_NativeWindow_DestroyNativeWindow(OHNativeWindow* window);
```

---

## 2. 行为变更

### 2.1 平台检测

#### 检测宏

```c
// src/util/detect_os.h
#if defined(__OHOS_FAMILY__)
#define DETECT_OS_OHOS 1
#endif
```

#### 影响范围

| 功能 | Linux | OHOS |
|------|-------|------|
| `dlopen()` 加载驱动 | ✅ 支持 | ⚠️ 路径不同 |
| `gettid()` 线程 ID | ✅ 标准 | ⚠️ musl 兼容 |
| `malloc()` 跟踪 | ✅ 可选 | ⚠️ 禁用 |

### 2.2 日志系统

#### 标准日志 vs HiLog

```c
// 上游标准日志
#include <cstdio>
printf("[MESA] info\n");

// OH 适配日志
#if DETECT_OS_OHOS
#include <hilog/log.h>
#define MESA_LOG_TAG "Mesa3D"
LOG_INFO(LOG_CORE, "[MESA] info");
#else
printf("[MESA] info\n");
#endif
```

#### 日志级别映射

| 标准级别 | OH HiLog 级别 |
|---------|--------------|
| `MESA_LOG_ALL` | `LOG_LEVEL_DEBUG` |
| `MESA_LOG_INFO` | `LOG_INFO` |
| `MESA_LOG_WARNING` | `LOG_WARN` |
| `MESA_LOG_ERROR` | `LOG_ERROR` |
| `MESA_LOG_FATAL` | `LOG_FATAL` |

### 2.3 性能追踪

```c
// OH 性能追踪集成
#if DETECT_OS_OHOS
#include <hitrace/trace.h>
#define MESA_TRACE_NAME(name) HITRACE_BEGIN(HITRACE_TAG_GRAPHIC, name)
#define MESA_TRACE_END() HITRACE_END(HITRACE_TAG_GRAPHIC)
#else
#define MESA_TRACE_NAME(name)
#define MESA_TRACE_END()
#endif
```

---

## 3. 禁用/移除的功能

### 3.1 强制禁用

| 功能 | 原因 | 替代方案 |
|------|------|---------|
| **GLX** | 无 X Server | 使用 EGL |
| **GBM (64位)** | 不适用 | 使用 Zink |
| **GLVND** | 无 NVIDIA 支持 | 原生 Mesa |
| **X11/Wayland** | 无显示服务器 | OH NativeWindow |

### 3.2 构建选项变更

| 选项 | 上游默认 | OH 值 | 原因 |
|------|---------|-------|------|
| `-Dplatforms` | auto | `ohos` | 显式指定 |
| `-Dglx` | auto | `disabled` | 无 X11 |
| `-Dtools` | true | `disabled` | 减少产物 |
| `-Dgbm` | auto | 32位启用 | 64位不适用 |

---

## 4. 新增宏定义

### 4.1 平台检测宏

```c
// 检测 OHOS 平台
#if defined(__OHOS_FAMILY__) || defined(__OHOS__)
#define MESA_HAVE_OHOS 1
#define DETECT_OS_OHOS 1
#endif

// musl libc 检测
#if defined(__MUSL__)
#define MESA_MUSL 1
#endif
```

### 4.2 编译器特性宏

```c
// OHOS 编译器信息
#define MESA_CC_CLANG 1
#define MESA_TARGET_ARCH_ARM64 1

// 交叉编译标记
#define MESA_CROSS_COMPILE 1
#define __ENDIAN_HEADER__ 1
```

### 4.3 功能开关宏

```c
// EGL 平台
#define HAVE_OHOS_PLATFORM 1
#define _EGL_PLATFORM_OHOS 0xXXXF

// Vulkan 平台
#define VK_USE_PLATFORM_OHOS 1
#define VK_OHOS_surface 1

// 日志控制
#define MESA_LOG_CONTROL_OHOS 1
```

---

## 5. 兼容性说明

### 5.1 与上游 API 兼容性

Mesa3D OH 版本 **完全兼容** 上游 OpenGL ES 2.0/3.0 和 EGL 1.5 API：

```c
// 这些 API 行为与上游完全一致
glClear(GL_COLOR_BUFFER_BIT);
glDrawArrays(GL_TRIANGLES, 0, vertexCount);
eglSwapBuffers(display, surface);
```

### 5.2 平台差异处理

```c
// 平台相关的条件编译
#if defined(DETECT_OS_OHOS)
// OHOS 特定代码
    OHNativeWindow* nativeWindow = CreateNativeWindow();
    vkCreateSurfaceOHOS(instance, &info, NULL, &surface);
#else
// 标准 Linux 代码
    Display* display = XOpenDisplay(NULL);
    VkDisplaySurfaceCreateInfoKHR createInfo = {...};
#endif
```

---

## 6. 常见问题

### Q1: 找不到 EGL 扩展

```c
// 确认 EGL 已初始化
EGLDisplay display = eglGetDisplay(EGL_DEFAULT_DISPLAY);
if (display == EGL_NO_DISPLAY) {
    // EGL 初始化失败
    return;
}

// 查询扩展
const char* exts = eglQueryString(display, EGL_EXTENSIONS);
// 检查是否包含 OHOS 扩展
```

### Q2: NativeWindow 生命周期

```c
// 错误: 过早释放
OHNativeWindow* window = CreateNativeWindow();
eglDestroySurface(display, surface);
OH_NativeWindow_DestroyNativeWindow(window);  // 错误！

// 正确: EGL Surface 销毁后释放
OHNativeWindow* window = CreateNativeWindow();
eglDestroySurface(display, surface);
if (surface == EGL_NO_SURFACE) {
    OH_NativeWindow_DestroyNativeWindow(window);  // 正确
}
```

---

## 7. API 参考

### 7.1 Vulkan OHOS 完整接口

| 函数 | 描述 |
|------|------|
| `vkCreateSurfaceOHOS` | 创建 OHOS Surface |
| `vkGetPhysicalDeviceSurfaceFormats2OHOS` | 获取 Surface 格式 |
| `vkGetPhysicalDeviceSurfacePresentModes2OHOS` | 获取 Present 模式 |

### 7.2 OH NativeWindow 完整接口

| 函数 | 描述 |
|------|------|
| `OH_NativeWindow_CreateNativeWindow` | 创建 NativeWindow |
| `OH_NativeWindow_DestroyNativeWindow` | 销毁 NativeWindow |
| `OH_NativeWindow_GetNativeWindowSize` | 获取尺寸 |
| `OH_NativeWindow_NativeWindowHandleOpt` | 配置选项 |
| `OH_NativeWindow_ReferenceBuffer` | 引用缓冲区 |
| `OH_NativeWindow_UnReferenceBuffer` | 取消引用 |
