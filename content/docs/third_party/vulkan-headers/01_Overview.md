# Vulkan-Headers 库概览

## 1.1 基础信息

| 属性 | 值 |
|------|-----|
| **库名称** | Vulkan-Headers |
| **上游版本** | v1.4.309 |
| **OH 版本** | 3.2 |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/KhronosGroup/Vulkan-Headers |
| **所属子系统** | thirdparty |
| **适配状态** | 已适配 OpenHarmony |

## 1.2 原始库功能

Vulkan-Headers 是 Khronos Group 维护的 Vulkan 图形 API 头文件库，提供以下核心功能：

### 核心 Vulkan API 定义

- **Vulkan 核心规范**：包含 Vulkan 1.4 版本的完整 API 声明
- **类型定义**：VkInstance、VkDevice、VkQueue 等核心类型
- **结构体定义**：创建、查询、操作 Vulkan 对象的结构体
- **枚举常量**：扩展标识、特性标志、错误码等
- **函数指针类型**：所有 Vulkan API 的函数原型声明

### 平台扩展支持

- **Android**：`VK_USE_PLATFORM_ANDROID_KHR`
- **iOS**：`VK_USE_PLATFORM_IOS_MVK`
- **macOS**：`VK_USE_PLATFORM_MACOS_MVK`
- **Windows**：`VK_USE_PLATFORM_WIN32_KHM**  
- **Wayland**：`VK_USE_PLATFORM_WAYLAND_KHR`
- **X11**：`VK_USE_PLATFORM_XCB_KHR`
- **Fuchsia**：`VK_USE_PLATFORM_FUCHSIA`
- **Metal**：`VK_USE_PLATFORM_METAL_EXT`
- **QNX**：`VK_QNX_screen_surface`

### 视频编解码扩展

- **AV1**：vulkan_video_codec_av1std.h
- **H.264**：vulkan_video_codec_h264std.h
- **H.265/HEVC**：vulkan_video_codec_h265std.h

## 1.3 在 OpenHarmony 中的定位

### 图形栈位置

```
┌─────────────────────────────────────┐
│           应用层 (Applications)       │
├─────────────────────────────────────┤
│      图形框架 (Graphic Framework)     │
│    (ACE Engine, Rosen, etc.)         │
├─────────────────────────────────────┤
│      Vulkan-Loader (加载层)           │
├─────────────────────────────────────┤
│    vulkan-headers (API 定义层)        │  ← 本库位置
├─────────────────────────────────────┤
│       Vulkan Driver (GPU 驱动)        │
└─────────────────────────────────────┘
```

### 核心作用

1. **API 契约定义**
   - 定义 Vulkan 应用与驱动之间的接口契约
   - 确保不同厂商驱动的 API 一致性
   - 提供类型安全和编译时检查

2. **OH 特有扩展**
   - 定义 OpenHarmony 平台专用的 Vulkan 扩展
   - 实现与 OH Native Window 系统的互操作
   - 支持 OH Native Buffer 内存管理

3. **标准化基础**
   - 提供 Khronos 官方标准的 Vulkan 规范
   - 确保 OpenHarmony Vulkan 实现符合行业标准
   - 支持 Vulkan CTS (Conformance Test Suite) 测试

### 与其他 OH 模块的关系

| 依赖关系 | 模块名称 | 说明 |
|---------|---------|------|
| **被依赖** | vulkan-loader | Vulkan 加载程序依赖头文件定义 |
| **被依赖** | skia | 2D/3D 图形渲染库依赖 Vulkan API |
| **被依赖** | graphic_2d | SDK 图形接口层 |
| **被依赖** | vk-gl-cts | Vulkan 合规测试套件 |

## 1.4 OH 适配概述

### 适配策略

Vulkan-Headers 采用**零 Patch** 适配策略，主要通过以下方式实现 OpenHarmony 支持：

1. **上游源码直接使用**
   - 不修改上游任何源文件
   - 使用 Khronos 官方发布的 Vulkan-Headers v1.4.309
   - 保持与上游的兼容性

2. **新增 OH 特有头文件**
   - `vulkan_ohos.h`：OHOS Surface 和 Native Buffer 扩展
   - `vulkan_screen.h`：QNX Screen 平台支持
   - `vk_ohos_native_buffer.h`：专用 Native Buffer 操作接口

3. **构建系统适配**
   - 在 BUILD.gn 中添加 `VK_USE_PLATFORM_OHOS` 宏定义
   - 配置 OH 特有的头文件包含路径
   - 导出 OH 特有源文件

### 适配优势

- **维护简单**：无需维护 Patch 合并流程
- **升级便捷**：上游版本更新时只需同步文件
- **标准化**：OH 扩展遵循 Vulkan 扩展设计规范
- **隔离性好**：OH 特有代码与上游代码分离

## 1.5 版本信息

### OH 版本历史

| OH 版本 | 库版本 | 更新内容 |
|---------|--------|---------|
| 3.2 | v1.4.309 | 当前版本 |
| 3.1 | v1.3.x | 需确认具体版本 |
| 3.0 | v1.3.x | 需确认具体版本 |

### 上游版本特性（v1.4.309）

- **Vulkan 1.4**：完整支持 Vulkan 1.4 Core 和所有已发布的扩展
- **视频扩展**：AV1、H.264、H.265 视频编解码支持
- **新扩展**：包含 2024 年发布的新扩展定义

## 1.6 相关资源

### 官方资源

- **上游仓库**：https://github.com/KhronosGroup/Vulkan-Headers
- **Vulkan Registry**：https://registry.khronos.org/vulkan/
- **Vulkan 规范**：https://www.khronos.org/vulkan/

### OpenHarmony 资源

- **Vulkan-Loader**：[README](../vulkan-loader/README_OpenHarmony.md)
- **图形子系统**：参考 OpenHarmony 图形架构文档
- **驱动接口**：[LoaderDriverInterface](https://gitee.com/openharmony/third_party_vulkan-loader/blob/master/docs/LoaderDriverInterface.md)

### 测试资源

- **Vulkan CTS**：third_party/vk-gl-cts
- **Vulkan SDK**：https://www.lunarg.com/vulkan-sdk/
