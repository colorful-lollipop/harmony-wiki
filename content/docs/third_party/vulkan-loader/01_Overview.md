# 01 - 原始库简介与 OpenHarmony 定位

## 1.1 原始库信息

### Vulkan-Loader 是什么

Vulkan-Loader 是 Khronos Group 官方提供的 Vulkan 加载器，是 Vulkan 图形 API 的核心组件之一。

**核心功能**：
1. **ICD (Installable Client Driver) 加载** - 加载 GPU 驱动程序
2. **Layer 管理** - 加载和管理 Vulkan Layer（如验证层、调试层）
3. **入口点分发** - 将 Vulkan API 调用分发到正确的驱动或 Layer
4. **多 GPU 支持** - 支持系统中存在多个 GPU 和驱动的场景

**架构位置**：
```
┌─────────────────────────────────────────┐
│         Vulkan 应用程序                  │
├─────────────────────────────────────────┤
│      libvulkan.so (Vulkan-Loader)       │
├─────────────────────────────────────────┤
│   Vulkan Layer 1    │   Vulkan Layer 2  │
├─────────────────────────────────────────┤
│         GPU Driver (ICD)                │
├─────────────────────────────────────────┤
│              GPU 硬件                    │
└─────────────────────────────────────────┘
```

### 上游版本信息

| 项目 | 内容 |
|------|------|
| **库名称** | Khronos Group - Vulkan-Loader |
| **版本** | v1.4.309 |
| **上游地址** | https://github.com/KhronosGroup/Vulkan-Loader.git |
| **许可证** | Apache-2.0 |
| **维护者** | LunarG, Inc. (主要)，Khronos Group |

### 支持的平台

- ✅ Linux
- ✅ Windows  
- ✅ macOS / iOS
- ✅ Fuchsia
- ✅ QNX
- ✅ OpenHarmony (本项目适配)
- ❌ Android (使用 Google 维护的独立版本)

---

## 1.2 在 OpenHarmony 中的定位

### 组件信息

| 项目 | 内容 |
|------|------|
| **OH 组件名** | @ohos/vulkan-loader |
| **OH 版本** | 4.1 |
| **子系统** | thirdparty |
| **所属部件** | graphic_graphic_2d |
| **API 级别** | llndk (提供给应用层使用) |

### 在 OpenHarmony 中的角色

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (HAP)                            │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐   │
│  │ 图形引擎      │  │ Skia         │  │ XComponent NAPI │   │
│  │ (Unity/Unreal)│  │ (2D图形库)    │  │ (原生窗口)       │   │
│  └──────┬───────┘  └──────┬───────┘  └────────┬────────┘   │
└─────────┼─────────────────┼───────────────────┼────────────┘
          │                 │                   │
          └─────────────────┴───────────────────┘
                            │
              ┌─────────────▼──────────────┐
              │   NDK (libvulkan.so)       │
              │   Vulkan-Loader            │
              └─────────────┬──────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
   ┌──────▼──────┐  ┌───────▼───────┐  ┌──────▼──────┐
   │ GPU Driver  │  │ Swapchain     │  │ Debug Layer │
   │ (ICD)       │  │ Layer         │  │             │
   └──────┬──────┘  └───────┬───────┘  └──────┬──────┘
          │                 │                 │
   ┌──────▼─────────────────▼─────────────────▼──────┐
   │            OHNativeWindow / GPU 硬件            │
   └─────────────────────────────────────────────────┘
```

### 主要功能在 OH 中的体现

1. **GPU 驱动加载**
   - 扫描 `/vendor/etc/vulkan/icd.d/` 等路径
   - 读取 GPU 驱动的 JSON 清单文件
   - 加载驱动共享库 (`.so` 文件)

2. **Swapchain Layer 加载**
   - 加载 `VK_LAYER_OHOS_surface` Layer
   - 实现 Vulkan 与 OHNativeWindow 的对接
   - 代码位置：[graphic_graphic_2d/frameworks/vulkan_layers/swapchain_layer](https://gitee.com/openharmony/graphic_graphic_2d/tree/master/frameworks/vulkan_layers/swapchain_layer)

3. **调试 Layer 支持**
   - 支持开发者动态加载自定义 Layer
   - 通过环境变量控制
   - 详见 README_OpenHarmony.md

### 在 OH 中的安装位置

```
/system/lib[64]/libvulkan.so
```

### 驱动配置文件路径

```
/vendor/etc/vulkan/icd.d/      ← 推荐放置 GPU 驱动配置
/system/etc/vulkan/icd.d/      ← 系统自带驱动配置
/data/vulkan/icd.d/            ← 用户自定义配置
```

### Layer 配置文件路径

```
/system/etc/vulkan/implicit_layer.d/   ← 默认加载的 Layer
/system/etc/vulkan/explicit_layer.d/   ← 需要显式启用的 Layer
/data/vulkan/implicit_layer.d/         ← 用户隐式 Layer
/data/vulkan/explicit_layer.d/         ← 用户显式 Layer
```

---

## 1.3 与上游的差异概述

### 无传统 Patch 文件

与其他许多 OpenHarmony 第三方库不同，vulkan-loader **没有独立的 `.patch` 文件**。适配通过以下方式实现：

1. **代码级集成**：OH 特有代码直接集成在源码中（`openharmony/` 目录）
2. **条件编译**：大量使用 `#ifdef __OHOS__` / `#ifdef VK_USE_PLATFORM_OHOS` 宏
3. **BUILD.gn 构建**：完全替换上游的 CMake 构建系统

### OH 特有功能

| 功能 | 上游支持 | OH 适配 | 说明 |
|------|---------|---------|------|
| ICD 加载 | ✅ | ✅ | 标准实现 |
| Layer 加载 | ✅ | ✅+ | 增加 Bundle 管理器集成 |
| WSI 扩展 | 平台相关 | ✅ | VK_OHOS_surface 等 |
| 日志系统 | stderr | ✅ | HiLog 桥接 |
| 环境变量 | getenv | ✅ | SysParam 系统参数 |
| Namespace | ❌ | ✅ | passthrough namespace 支持 |

---

## 1.4 相关文档

### 上游文档
- [LoaderInterfaceArchitecture.md](../docs/LoaderInterfaceArchitecture.md) - 加载器架构
- [LoaderDriverInterface.md](../docs/LoaderDriverInterface.md) - 驱动接口
- [LoaderLayerInterface.md](../docs/LoaderLayerInterface.md) - Layer 接口
- [BUILD.md](../BUILD.md) - 上游构建说明

### OpenHarmony 特有文档
- [README_OpenHarmony.md](../README_OpenHarmony.md) - OH 使用指南（中文）
- [swapchain_layer 代码](https://gitee.com/openharmony/graphic_graphic_2d/tree/master/frameworks/vulkan_layers/swapchain_layer) - OH WSI Layer 实现

---

*文档版本：v1.0*
*最后更新：2026-02-07*
