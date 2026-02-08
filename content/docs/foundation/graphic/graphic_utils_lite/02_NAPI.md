# N-API 接口说明

本文档说明 `graphic_utils_lite` 的 N-API 接口情况。

## 重要说明

### 本项目不提供传统 N-API

经过全面扫描确认，**`graphic_utils_lite` 项目不提供 JavaScript/TypeScript 接口（N-API）**。

**扫描证据**：
- 项目根目录下无 `napi*.cpp` 文件
- 代码中未发现 `NAPI_MODULE`、`napi_define_properties`、`napi_create_*` 等宏使用
- 所有 API 以 C++ 头文件形式导出（`interfaces/kits/` 和 `interfaces/innerkits/`）

### 原因分析

本项目定位为**系统基础设施库**，面向以下调用方提供 C++ 接口：

| 调用方 | 接口类型 | 说明 |
|--------|----------|------|
| `arkui_ui_lite` | 内部 C++ API | UI 框架直接链接使用 |
| `window_manager_lite` | 内部 C++ API | 窗口管理器直接链接使用 |
| `graphic_surface_lite` | 内部 C++ API | 表面管理器直接链接使用 |

## 接口访问方式

### 方式一：应用框架层

```
JS 应用
    ↓
ArkUI 框架 (arkui_ui_lite)  [JS → N-API 转换在此层]
    ↓
graphic_utils_lite  [C++ 接口]
```

ArkUI 框架本身提供了 N-API，开发者通过 ArkUI 组件间接使用图形能力。

### 方式二：Native 开发

对于需要直接使用 `graphic_utils_lite` 的 Native 应用，可以通过以下方式：

1. **直接链接库**：在 `BUILD.gn` 中添加依赖
2. **包含头文件**：从 `interfaces/kits/` 和 `interfaces/innerkits/` 包含所需头文件

```gn
# 示例 BUILD.gn 依赖
deps += [ "//foundation/graphic/graphic_utils_lite:graphic_utils_lite" ]
```

## 替代方案

如需在 JS/TS 中使用本项目的图形功能，建议：

| 需求 | 推荐方案 |
|------|----------|
| 2D 绘图 | 使用 ArkUI `<canvas>` 组件 |
| 图像处理 | 使用 ArkUI Image API |
| 自定义渲染 | 使用 ArkUI Native Rendering 接口 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [概览](00_Overview.md) | 项目定位与能力说明 |
| [架构说明](01_Architecture.md) | 内部架构与模块划分 |
| [内部 API](03_InnerAPI.md) | C++ 接口详细说明 |

---

*最后更新时间：2026-02-06*
