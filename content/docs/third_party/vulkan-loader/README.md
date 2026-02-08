# Vulkan-Loader Wiki

## 简介

本文档是 **OpenHarmony 第三方库 Vulkan-Loader** 的详细 Wiki，专注于分析和记录该库在 OpenHarmony 中的集成与适配情况。

## 文档导航

### 基础文档
- [01_Overview.md](01_Overview.md) - 原始库简介与 OH 定位
- [02_Patches.md](02_Patches.md) - Patch 与适配分析（**核心文档**）
- [03_Build_Integration.md](03_Build_Integration.md) - OH 构建适配
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系与使用场景
- [05_API_Differences.md](05_API_Differences.md) - API/接口差异
- [06_Security.md](06_Security.md) - 安全风险分析

### 工作文档
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估结果
- [_work/NOTES.md](_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](_work/PLAN.md) - 任务进度

## 快速开始

### 关键信息摘要

| 项目 | 内容 |
|------|------|
| **原始库** | Khronos Vulkan-Loader v1.4.309 |
| **OH 组件** | @ohos/vulkan-loader v4.1 |
| **许可证** | Apache-2.0 |
| **适配方式** | 代码级集成（无传统 Patch） |
| **输出** | libvulkan.so |
| **子系统** | thirdparty |

### OH 特有适配亮点

1. **平台适配**：通过 `VK_USE_PLATFORM_OHOS` 宏实现完整的 OHOS 平台支持
2. **WSI 扩展**：支持 `VK_OHOS_surface`、`VK_OHOS_native_buffer` 等 OH 特有扩展
3. **Bundle 集成**：与应用框架集成，支持调试 Layer 动态加载
4. **日志系统**：桥接到 OpenHarmony HiLog 系统
5. **Namespace 隔离**：支持 OHOS 的库 namespace 机制

## 适用读者

- **图形开发者**：了解 Vulkan 在 OH 中的加载机制
- **驱动开发者**：实现 GPU 驱动与 Vulkan-Loader 对接
- **系统开发者**：维护升级 vulkan-loader 组件
- **安全工程师**：评估 Vulkan 相关安全风险

## 相关资源

- [Khronos Vulkan-Loader 上游](https://github.com/KhronosGroup/Vulkan-Loader)
- [Vulkan 规范文档](https://www.vulkan.org/)
- [OpenHarmony 图形子系统文档](https://gitee.com/openharmony/graphic_graphic_2d)

---

*最后更新：2026-02-07*
