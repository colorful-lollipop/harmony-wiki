# Patch 详细分析

## 概述

**重要说明**：本库（EGL-Registry）是 **纯头文件注册表**，不包含任何可编译的源代码实现。因此，**没有传统的代码 Patch 文件**。

OpenHarmony 对 EGL 的适配主要通过以下方式实现：
1. **构建配置适配**（`BUILD.gn`）
2. **平台类型定义**（`eglplatform.h`）
3. **OH 特有扩展**（`extensions/OH/`）

本章节重点说明 OH 特有的扩展定义和平台适配，这些内容在功能上等同于其他库中的 Patch。

---

## OH 特有扩展清单

### 扩展列表

| 扩展名称 | 文件路径 | 功能描述 | 状态 |
|----------|----------|----------|------|
| `EGL_OHOS_image_native_buffer` | `extensions/OH/EGL_OHOS_image_native_buffer.txt` | 支持 OHOS 原生缓冲区与 EGLImage 互操作 | 已发布 |

---

## 扩展详细分析

### EGL_OHOS_image_native_buffer

**修改文件**：
- `extensions/OH/EGL_OHOS_image_native_buffer.txt`（新增）

**功能概述**：
该扩展定义了 `EGL_NATIVE_BUFFER_OHOS` 令牌，允许使用 OHOS 原生缓冲区（HNativeWindowBuffer）作为 `eglCreateImageKHR()` 的 source，创建可用于 GPU 渲染的 EGLImage 对象。

**原始需求**：
OHOS 需要一种机制，将系统原生的图形缓冲区（HNativeWindowBuffer）与 GPU 可访问的 EGLImage 进行绑定，以实现：
- 零拷贝的图形数据传输
- 多媒体渲染管线的 GPU 加速
- 显示子系统与渲染子系统的无缝集成

**关键代码变更**：

```c
// 新增令牌定义
#define EGL_NATIVE_BUFFER_OHOS 0x34E1

// 使用示例
EGLImageKHR eglImage = eglCreateImageKHR(
    display,
    EGL_NO_CONTEXT,
    EGL_NATIVE_BUFFER_OHOS,
    (EGLClientBuffer)nativeWindowBuffer,
    attribs
);
```

**扩展规格摘要**：

| 项目 | 内容 |
|------|------|
| **扩展编号** | #148 |
| **版本** | 1 |
| **发布日期** | 2021年12月14日 |
| **作者** | Xindong Shi, Zheng Li (Huawei) |
| **依赖** | EGL 1.2, EGL_KHR_image_base |

**新增令牌**：

| 令牌 | 值 | 用途 |
|------|-----|------|
| `EGL_NATIVE_BUFFER_OHOS` | `0x34E1` | `eglCreateImageKHR()` 的 target 参数 |

**错误码定义**：

| 错误条件 | 错误码 |
|----------|--------|
| `<target>` 是 `EGL_NATIVE_BUFFER_OHOS` 且 `<buffer>` 无效 | `EGL_BAD_PARAMETER` |
| `<target>` 是 `EGL_NATIVE_BUFFER_OHOS` 且 `<ctx>` 非 `EGL_NO_CONTEXT` | `EGL_BAD_CONTEXT` |
| `<target>` 是 `EGL_NATIVE_BUFFER_OHOS` 且 `<buffer>属性不支持 | `EGL_BAD_PARAMETER` |

**OH 价值**：
- 实现 OHOS 原生缓冲区与 GPU 的无缝对接
- 为图形驱动提供标准化的缓冲区绑定接口
- 支持零拷贝渲染管线

**升级建议**：
- 此扩展为 OHOS 特有功能，上游无对应实现
- 升级上游版本时，需保留 `extensions/OH/` 目录
- 确认新版本 `egl.xml` 中的枚举值不与 `0x34E1` 冲突

---

## 平台类型适配

### eglplatform.h 适配

**文件位置**：`api/EGL/eglplatform.h`

**适配代码**：

```c
#elif defined(OHOS_PLATFORM)

struct NativeWindow;

typedef void*                   EGLNativeDisplayType;
typedef void*                   EGLNativePixmapType;
typedef struct NativeWindow*    EGLNativeWindowType;
```

**适配说明**：

| 类型 | OHOS 定义 | 用途 |
|------|----------|------|
| `EGLNativeDisplayType` | `void*` | EGL Display 类型 |
| `EGLNativePixmapType` | `void*` | 像素图类型 |
| `EGLNativeWindowType` | `NativeWindow*` | OHOS 原生窗口句柄 |

**设计决策**：
- `EGLNativeWindowType` 直接映射到 OHOS 的 `NativeWindow` 结构体指针
- 这种设计确保 EGL 实现能够直接操作 OHOS 窗口系统

**升级注意事项**：
- 保留 `OHOS_PLATFORM` 分支定义
- 确认 `NativeWindow` 结构体定义兼容性

---

## 构建配置适配

### BUILD.gn 适配详情

**文件位置**：`BUILD.gn`

**完整配置**：

```gn
# Copyright (c) 2021-2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");

import("//build/ohos.gni")

config("libEGL_public_config") {
  # 头文件包含路径
  include_dirs = [ "api" ]
  
  # 全局宏定义
  defines = [ "ENABLE_EGL" ]
  
  # OHOS 平台特定宏
  if (current_os == "ohos") {
    defines += [ "OHOS_PLATFORM" ]
  }
}

ohos_static_library("libEGL") {
  public_configs = [ ":libEGL_public_config" ]
}
```

**配置说明**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `include_dirs` | `["api"]` | 导出 EGL 头文件目录 |
| `defines` | `["ENABLE_EGL"]` | 启用 EGL 功能的全局宏 |
| `defines` (OHOS) | `["OHOS_PLATFORM"]` | OHOS 平台条件编译宏 |

**设计决策**：
- 使用 `ohos_static_library` 模板（虽为纯头文件库，保持库类型一致性）
- 通过 `public_configs` 导出配置，确保依赖方自动获取头文件路径

**升级建议**：
- 构建配置相对稳定，可直接复用
- 检查 `import("//build/ohos.gni")` 路径兼容性

---

## 适配总结

### 适配类型分布

| 适配类型 | 文件数量 | 变更性质 |
|----------|----------|----------|
| OH 特有扩展 | 1 | 新增功能 |
| 平台类型定义 | 1 | 条件编译 |
| 构建配置 | 1 | GN 适配 |

### 适配影响范围

| 影响项 | 状态 | 说明 |
|--------|------|------|
| API 兼容性 | ✅ 兼容 | 遵循 EGL 1.5 规范 |
| 扩展注册 | ✅ 完整 | OH 扩展已注册 |
| 平台支持 | ✅ 完整 | OHOS_PLATFORM 已定义 |
| 构建集成 | ✅ 完整 | GN 配置可用 |

### 维护建议

1. **扩展管理**：新 OHOS EGL 扩展应在 `extensions/OH/` 目录下添加
2. **版本同步**：上游更新时，检查 `eglplatform.h` 平台分支变化
3. **枚举值协调**：添加新扩展时，确认枚举值不与上游冲突
4. **文档同步**：扩展规格更新时，同步更新本文档
