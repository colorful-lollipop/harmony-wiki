# OpenCL-Headers OpenHarmony Wiki

## 库概述

OpenCL-Headers 是 Khronos Group 维护的 OpenCL API 头文件库，为异构计算提供标准化的编程接口。本 Wiki 文档记录了 OpenCL-Headers 在 OpenHarmony 系统中的集成与适配信息。

### 原始库信息

| 项目 | 内容 |
|------|------|
| 库名称 | Khronos Group OpenCL Headers |
| 版本 | v2024.05.08 |
| 许可证 | Apache-2.0 |
| 上游地址 | https://github.com/KhronosGroup/OpenCL-Headers |
| 描述 | OpenCL 标准 API 头文件集合 |

### OpenHarmony 适配概述

OpenCL-Headers 在 OpenHarmony 中的适配主要体现在以下几个方面：

**适配策略**：由于 OpenCL-Headers 是纯头文件库，不包含实现代码，OpenHarmony 采用了动态加载包装器的方式进行适配。这种方式避免了静态链接 OpenCL 驱动带来的依赖问题，同时提供了灵活的运行时绑定能力。

**核心适配组件**：

| 组件 | 类型 | 用途 |
|------|------|------|
| opencl_wrapper.h | 新增头文件 | 定义动态加载包装器的函数指针类型和初始化接口 |
| opencl_wrapper.cpp | 新增源文件 | 实现动态加载 OpenCL 驱动的包装器功能 |
| BUILD.gn | 适配文件 | OpenHarmony 构建系统配置 |
| opencl-headers-CL | 适配目录 | OH 公共头文件包含路径 |

**适配特点**：

- 采用 `dlopen` 和 `dlsym` 实现运行时动态加载 OpenCL 驱动
- 支持多平台库路径自动探测（Apple、Mali GPU、HVGR GPU 等）
- 线程安全的初始化机制
- 完整的 OpenCL 3.0 API 支持
- ARM 特有扩展支持（`clImportMemoryARM`）

## 文档导航

### 必读文档

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [README](README.md) | 本文档，提供概览和导航 | 所有开发者 |
| [01_概述](01_Overview.md) | 库功能详细介绍和 OH 定位 | 需要了解库功能的开发者 |
| [02_Patch 分析](02_Patches.md) | OH 特有适配说明 | 需要理解适配细节的开发者 |
| [03_构建适配](03_Build_Integration.md) | BUILD.gn 配置说明 | 需要进行构建配置的开发者 |
| [04_使用说明](04_Usage_in_OH.md) | 在 OH 中的使用方式 | 需要集成该库的开发者 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告，包含详细分析过程 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录（待创建） |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度跟踪（待创建） |

## 快速入门

### 基本信息

- **组件名称**：`@ohos/opencl-headers`
- **所属子系统**：`thirdparty`
- **OH 版本**：3.2
- **构建目标**：`//third_party/opencl-headers:libcl`

### 包含的头文件

```cpp
// 标准 OpenCL 头文件
#include <CL/opencl.h>

// OH 包装器头文件（需要动态加载时使用）
#include <opencl_wrapper.h>
```

### 初始化 OpenCL

```cpp
#include <opencl_wrapper.h>

// 初始化 OpenCL（自动加载驱动）
bool success = OHOS::InitOpenCL();
if (!success) {
    // 处理初始化失败
}

// 使用 OpenCL API
cl_platform_id platform;
clGetPlatformIDs(1, &platform, nullptr);
```

## 关键技术特性

### 动态加载机制

OpenHarmony 采用动态加载机制来处理 OpenCL 驱动的加载。这种设计带来了以下优势：

1. **运行时绑定**：应用启动时动态加载 OpenCL 驱动，避免编译时依赖特定厂商的实现。

2. **多实现支持**：自动探测并加载可用的 OpenCL 实现，支持多种 GPU 厂商。

3. **延迟初始化**：OpenCL 驱动在首次使用时才加载，减少启动开销。

### 库路径探测

默认的库路径搜索顺序如下：

| 优先级 | 路径 | 平台 |
|--------|------|------|
| 1 | `/vendor/lib64/chipsetsdk/libGLES_mali.so` | Mali GPU |
| 2 | `/system/lib64/libGLES_mali.so` | Mali GPU |
| 3 | `libGLES_mali.so` | Mali GPU |
| 4 | `/vendor/lib64/chipsetsdk/libhvgr_v200.so` | HVGR GPU |
| 5 | `/vendor/lib64/passthrough/libhvgr_v200.so` | HVGR GPU |
| 6 | `libhvgr_v200.so` | HVGR GPU |

### 版本兼容性

- **目标 OpenCL 版本**：3.0（`CL_TARGET_OPENCL_VERSION = 300`）
- **向后兼容**：支持 OpenCL 1.2、2.0 API 的条件编译
- **扩展支持**：包含 ARM 特有扩展和 Intel 扩展

## 相关资源

### 外部链接

- [Khronos OpenCL 官方主页](https://www.khronos.org/opencl/)
- [OpenCL-Headers 上游仓库](https://github.com/KhronosGroup/OpenCL-Headers)
- [OpenCL 规范文档](https://www.khronos.org/registry/OpenCL/)

### OpenHarmony 相关

- [OpenHarmony 第三方组件规范](../docs/third_party_guidelines.md)
- [构建系统文档](https://gitee.com/openharmony/build)
- [图形子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn subsystem/graphics/README.md)

## 贡献指南

### 文档改进

本 Wiki 文档托管在与代码相同的仓库中。如需改进文档，请：

1.Fork 仓库并创建分支
2.修改 `wiki/` 目录下的相应文件
3.提交 Pull Request

### 问题反馈

关于 OpenCL-Headers 适配的问题，请：

1.检查本文档是否已有相关说明
2.在 [OpenHarmony 问题跟踪系统](https://gitee.com/openharmony/issues) 中搜索类似问题
3.创建新问题并标记相关组件

## 版本历史

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| 1.0 | 2024-05-08 | 初始版本，跟踪上游 v2024.05.08 |
| 1.1 | 2024-XX-XX | 补充动态加载机制说明（待更新） |

---

*本文档最后更新于 2024-05-08*
