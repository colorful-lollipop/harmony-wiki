# 04_Usage_in_OH - 依赖关系与使用

## 概述

本文档描述 Mesa3D 在 OpenHarmony 系统中的 **依赖关系** 和 **使用方式**。

---

## 依赖关系图

### Mesa3D 依赖栈

```
Mesa3D (third_party/mesa3d)
│
├── OpenHarmony 系统依赖
│   ├── graphic_surface:surface
│   │   └── 提供: OH NativeWindow 接口
│   ├── hilog:libhilog
│   │   └── 提供: 日志输出 (HiLog)
│   ├── hitrace:hitrace_meter
│   │   └── 提供: 性能追踪
│   ├── init:libbegetutil
│   │   └── 提供: 工具库
│   └── c_utils (内置)
│
├── 第三方依赖
│   ├── zlib (third_party)
│   ├── libdrm (third_party)
│   ├── libwayland-* (third_party)
│   └── expat (third_party)
│
└── 工具链
    ├── clang/llvm (prebuilt)
    ├── meson (Python)
    └── pkg-config
```

### OH 模块对 Mesa3D 的依赖

```
┌─────────────────────────────────────────────────────────────────┐
│                      上层依赖模块                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  ace_engine (UI 框架)                    │   │
│  │                  media_foundation (媒体框架)             │   │
│  │                  3D 游戏引擎                              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
└──────────────────────────────┼──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Mesa3D (OpenGL ES 提供者)                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  libEGL_mesa.so ──────────────────────────────────────▶│   │
│  │  libgallium-25.0.1.so                                  │   │
│  │  libGLESv2.so.2.0.0                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      底层支撑模块                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              libsurface (NativeWindow)                  │   │
│  │              libdrm (设备资源管理)                       │   │
│  │              device/vulkan (GPU 驱动)                   │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 依赖者分析

### 主要依赖模块

| 模块 | 用途 | 依赖方式 |
|------|------|---------|
| **ace_engine** | ArkUI 框架 | EGL + OpenGL ES |
| **media_foundation** | 媒体播放 | OpenGL ES 渲染 |
| **graphic_2d** | 2D 图形子系统 | EGL Platform |
| **Rockchip 产品** | RK3568 等设备 | GPU 驱动集成 |

### GN 依赖声明

```gn
# foundation/arkui/ace_engine/BUILD.gn 示例
ohos_shared_library("libace") {
  deps = [
    "//third_party/mesa3d:mesa3d_libEGL",
    "//third_party/mesa3d:mesa3d_libgallium",
  ]
}
```

---

## 使用方式

### 1. EGL 初始化

```c
// EGL 初始化 (OHOS 平台自动检测)
#include <EGL/egl.h>
#include <EGL/eglext.h>

EGLDisplay display;
EGLContext context;
EGLSurface surface;
EGLConfig config;

// 1. 获取 EGL 显示句柄
display = eglGetDisplay(EGL_DEFAULT_DISPLAY);
if (display == EGL_NO_DISPLAY) {
    // 处理错误
}

// 2. 初始化 EGL
EGLBoolean success = eglInitialize(display, NULL, NULL);
if (!success) {
    // 初始化失败
}

// 3. 选择配置
EGLint numConfigs;
eglChooseConfig(display, attribs, &config, 1, &numConfigs);

// 4. 创建上下文
context = eglCreateContext(display, config, EGL_NO_CONTEXT, contextAttribs);

// 5. 创建 surface (绑定到 NativeWindow)
surface = eglCreateWindowSurface(display, config, nativeWindow, NULL);
```

### 2. OpenGL ES 渲染

```c
// OpenGL ES 2.0 渲染循环
#include <GLES2/gl2.h>
#include <GLES2/gl2ext.h>

void render_frame() {
    // 清屏
    glClearColor(0.0f, 0.0f, 1.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    // 绑定 shader program
    glUseProgram(program);

    // 设置 uniforms
    glUniformMatrix4fv(uMatrix, 1, GL_FALSE, matrix);

    // 绘制
    glDrawElements(GL_TRIANGLES, indexCount, GL_UNSIGNED_SHORT, indices);

    // 交换缓冲区
    eglSwapBuffers(display, surface);
}
```

### 3. NativeWindow 绑定

```c
// 获取 OH NativeWindow
#include <surface.h>
#include <native_window.h>

OHNativeWindow* nativeWindow = OH_NativeWindow_CreateNativeWindow(&window);

// 配置 NativeWindow 属性
int32_t width, height;
OH_NativeWindow_GetNativeWindowSize(nativeWindow, &width, &height);

// 使用后释放
OH_NativeWindow_DestroyNativeWindow(nativeWindow);
```

---

## API 使用示例

### 1. EGL Platform 扩展

```c
// 查询 EGL OHOS 扩展
const char* extensions = eglQueryString(EGL_NO_DISPLAY, EGL_EXTENSIONS);
if (strstr(extensions, "EGL_OHOS_platform_surface")) {
    // 使用 OHOS 平台扩展
}
```

### 2. Vulkan OHOS Surface

```c
// 使用 Vulkan OHOS Surface 扩展
#include <vulkan/vulkan.h>
#include <vulkan/vulkan_ohos.h>

VkSurfaceCreateInfoOHOS createInfo = {
    .sType = VK_STRUCTURE_TYPE_SURFACE_CREATE_INFO_OHOS,
    .pNext = NULL,
    .flags = 0,
    .window = nativeWindow
};

VkSurfaceKHR surface;
vkCreateSurfaceOHOS(instance, &createInfo, NULL, &surface);
```

---

## 链接方式

### 动态链接

```gn
# BUILD.gn 中声明
ohos_prebuilt_shared_library("mesa3d_libEGL") {
  source = "${root_build_dir}/thirdparty/mesa3d/lib/libEGL_mesa.so"
}

ohos_prebuilt_shared_library("mesa3d_libgallium") {
  source = "${root_build_dir}/thirdparty/mesa3d/lib/libgallium-25.0.1.so"
}
```

### 链接声明

```c
// 链接标志
LDFLAGS += -lEGL -lGLESv2 -lgallium-25.0.1
```

---

## 配置项说明

### 特性开关

| 配置项 | 默认值 | 说明 |
|-------|-------|------|
| `mesa3d_feature_upgrade_skia` | false | 是否使用新版 Skia (m133) |

### 环境变量

```bash
# Mesa 调试输出
export MESA_LOG=debug
export MESA_DEBUG=1

# Gallium 驱动选择
export GALLIUM_DRIVER=zink
```

---

## 常见使用场景

### 场景 1: 2D/3D UI 渲染

```
用户界面
    │
    ▼
Ace Engine (UI 框架)
    │
    ├── OpenGL ES 2.0/3.0 绘制
    │
    ▼
Mesa3D (Zink 驱动)
    │
    ▼
Vulkan 驱动
    │
    ▼
GPU 硬件
```

### 场景 2: 游戏渲染

```
游戏引擎
    │
    ├── OpenGL ES API 调用
    │
    ▼
Mesa3D (OpenGL 实现)
    │
    ├── EGL 上下文管理
    ├── OpenGL 命令转换
    │
    ▼
Zink + Kopper
    │
    ▼
Vulkan 命令缓冲区
    │
    ▼
GPU 执行
```

### 场景 3: 视频播放

```
视频解码器
    │
    ├── 纹理上传
    │
    ▼
OpenGL ES (视频后处理)
    │
    ▼
Mesa3D
    │
    ▼
显示 (EGL + NativeWindow)
```

---

## 性能优化建议

### 1. 着色器缓存

```c
// 启用着色器缓存
glEnable(GL_SHADER_CACHE);
```

### 2. 批处理渲染

```c
// 合并多个绘制调用
for (i = 0; i < count; i++) {
    draw_object(i);  // 改为批处理
}
```

### 3. 纹理优化

```c
// 使用合适的纹理格式
GLenum format = GL_COMPRESSED_RGBA8_ETC2_EAC;

// Mipmap
glGenerateMipmap(GL_TEXTURE_2D);
```

---

## 调试技巧

### EGL 调试

```c
// 启用 EGL 调试输出
eglDebugMessageControlKHR(egl_callback, NULL);
```

### OpenGL 调试

```c
// OpenGL 调试回调
glEnable(GL_DEBUG_OUTPUT);
glDebugMessageCallback(gl_debug_callback, NULL);
```

### 日志查看

```bash
# 查看 Mesa 日志
hilog | grep -i mesa

# 查看 EGL 错误
hilog | grep -i egl
```

---

## 已知限制

| 限制 | 说明 | 解决方案 |
|------|------|---------|
| **GLX 不可用** | 桌面端 GLX 协议禁用 | 使用 EGL |
| **GBM 64位禁用** | 32位 GBM 库不适用于 64位 | 使用 Zink |
| **Vulkan 依赖** | Zink 需要 Vulkan 驱动 | 确保 GPU 驱动正确 |

---

## 相关资源

| 资源 | 链接 |
|------|------|
| EGL 规范 | https://www.khronos.org/registry/EGL/ |
| GLES 规范 | https://www.khronos.org/registry/OpenGL/ |
| Vulkan 规范 | https://www.vulkan.org/specification |
| Mesa 文档 | https://docs.mesa3d.org/ |
