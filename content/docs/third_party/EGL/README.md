# OpenHarmony EGL 第三方库 Wiki

## 库概览

| 项目 | 内容 |
|------|------|
| **库名称** | Khronos Group EGL-Registry |
| **版本** | 1.5（上游）/ 3.1（OH 包版本） |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/KhronosGroup/EGL-Registry.git |
| **OH 组件** | `egl`（子系统：thirdparty） |

## 什么是 EGL

EGL（EGL-Registry）是 Khronos Group 维护的 **EGL API 头文件和扩展注册表**。EGL 是一个接口规范，定义了 Khronos 渲染 API（如 OpenGL ES、OpenVG）与底层原生平台窗口系统之间的通信标准。

**在 OpenHarmony 中的定位**：EGL 库为 OH 的图形渲染子系统提供标准化的 EGL API 定义和扩展规范，是图形驱动与渲染框架之间的接口契约。

## OpenHarmony 适配概述

本库是 **纯头文件注册表**，OpenHarmony 对其的适配主要体现在：

### 1. 构建系统适配
- 使用 GN 构建系统（`BUILD.gn`）
- 定义 `OHOS_PLATFORM` 宏用于条件编译
- 导出 `api/` 目录头文件

### 2. 平台类型定义
- 在 `eglplatform.h` 中定义 OHOS 原生窗口类型
- `EGLNativeWindowType` 映射到 `NativeWindow` 结构体

### 3. OH 特有扩展
- `EGL_OHOS_image_native_buffer`：支持 OHOS 原生缓冲区与 EGLImage 互操作

## 文档导航

### 必读文档

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议 | ⭐⭐⭐ |
| [01_Overview.md](./01_Overview.md) | 库功能与 OH 定位 | ⭐⭐ |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配详解 | ⭐⭐⭐ |
| [02_Patches.md](./02_Patches.md) | OH 特有扩展说明 | ⭐⭐ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | ⭐ |

### 特殊说明

**本库没有代码 Patch**：作为纯头文件注册表，EGL 库不包含任何可编译的源代码实现。所有的 OH 适配都通过构建配置、平台类型定义和扩展规范来实现。

## 快速索引

### 关键文件路径

```
third_party/EGL/
├── BUILD.gn                    # OH 构建配置
├── api/EGL/
│   ├── egl.h                  # EGL 核心 API 头文件
│   ├── eglplatform.h           # 平台类型定义（含 OHOS）
│   └── eglext.h               # EGL 扩展声明
├── extensions/
│   └── OH/
│       └── EGL_OHOS_image_native_buffer.txt  # OH 特有扩展
└── bundle.json                 # OH 组件配置
```

### 关键配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `defines` | `ENABLE_EGL`, `OHOS_PLATFORM` | 编译宏定义 |
| `include_dirs` | `api/` | 头文件包含路径 |
| `inner_kit` | `//third_party/EGL:libEGL` | OH 内部kit |

## 相关资源

- [Khronos EGL 官方规范](https://www.khronos.org/registry/egl/)
- [EGL-Registry 上游仓库](https://github.com/KhronosGroup/EGL-Registry)
- [OpenHarmony 图形架构](../graphics/README.md)
