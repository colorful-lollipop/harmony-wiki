# 全站导航（SUMMARY）

本文档提供 `graphic_utils_lite` Wiki 的完整导航结构。

## 文档索引

| 编号 | 文档名称 | 说明 | 目标读者 |
|------|----------|------|----------|
| 00 | [概览](00_Overview.md) | 项目定位、边界、核心能力、运行环境 | 所有开发者 |
| 01 | [架构说明](01_Architecture.md) | 组件图、数据流、线程模型、关键时序 | 架构师、核心开发者 |
| 02 | [N-API 接口](02_NAPI.md) | JS/TS 接口清单（本项目无 N-API） | JS/TS 开发者 |
| 03 | [内部 API](03_InnerAPI.md) | 模块接口、依赖方向、稳定性标注 | C++ 开发者 |
| 04 | [GN 构建配置](04_Build.md) | targets 列表、依赖、编译产物 | 构建工程师 |
| 05 | [安全风险评估](05_Security.md) | 攻击面、信任边界、风险清单 | 安全工程师 |
| 06 | [常见问题](06_FAQ.md) | 构建、运行、调试问题与解决方案 | 所有开发者 |

## 新人阅读路线

### 路线一：C++ 开发者

```
1. 阅读 [概览](00_Overview.md) 了解项目定位
2. 阅读 [架构说明](01_Architecture.md) 理解模块划分
3. 阅读 [内部 API](03_InnerAPI.md) 掌握接口调用方式
4. 参考 [GN 构建配置](04_Build.md) 进行编译调试
5. 查阅 [安全评估](05_Security.md) 了解安全注意事项
```

### 路线二：构建工程师

```
1. 阅读 [概览](00_Overview.md) 了解项目边界
2. 直接阅读 [GN 构建配置](04_Build.md) 掌握编译流程
3. 查阅 [常见问题](06_FAQ.md) 解决构建问题
```

### 路线三：安全审计

```
1. 阅读 [概览](00_Overview.md) 了解攻击面范围
2. 阅读 [安全风险评估](05_Security.md) 获取风险清单
3. 参考 [内部 API](03_InnerAPI.md) 追溯代码证据
```

## 模块快速跳转

### Utils 模块（公共工具）

| 功能 | 头文件 | 实现文件 |
|------|--------|----------|
| 颜色处理 | `color.h` | `color.cpp` |
| 2D 几何 | `geometry2d.h` | `geometry2d.cpp` |
| 变换 | `transform.h` | `transform.cpp` |
| 仿射变换 | `trans_affine.h` | `trans_affine.cpp` |
| 内存管理 | `mem_api.h` | `mem_api.cpp` |

### Diagram 模块（2D 图形引擎）

| 子模块 | 功能 | 头文件目录 |
|--------|------|------------|
| depiction | 曲线绘制 | `diagram/depiction/` |
| rasterizer | 光栅化 | `diagram/rasterizer/` |
| vertexgenerate | 顶点生成 | `diagram/vertexgenerate/` |
| vertexprimitive | 图元处理 | `diagram/vertexprimitive/` |
| common | 公共组件 | `diagram/common/` |

### Hals 模块（硬件抽象层）

| 功能 | 头文件 | 实现文件 |
|------|--------|----------|
| GFX 引擎 | `gfx_engines.h` | `gfx_engines.cpp` |
| FrameBuffer | `hi_fbdev.h` | `hi_fbdev.cpp` |

## API 层级说明

```
┌─────────────────────────────────────┐
|           应用层 (ArkUI)            │
├─────────────────────────────────────┤
|       窗口管理层 (WindowMgr)        │
├─────────────────────────────────────┤
|       表面层 (Surface)              │
├─────────────────────────────────────┤
|      graphic_utils_lite (本文档)     │
│  ┌─────────────┬─────────────┐     │
│  │   Utils    │  Diagram    │     │
│  │  公共工具   │  2D 图形引擎 │     │
│  └─────────────┴─────────────┘     │
│  ┌─────────────┬─────────────┐     │
│  │   Hals    │   HALs      │     │
│  │ 硬件抽象层  │   驱动适配  │     │
│  └─────────────┴─────────────┘     │
├─────────────────────────────────────┤
|     驱动层 (HDI/Framebuffer)        │
└─────────────────────────────────────┘
```

## 版本历史

| 版本 | 更新内容 | 日期 |
|------|----------|------|
| 1.0 | 初始版本 | 2026-02-06 |

## 相关链接

- [OpenHarmony 图形子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/graphics.md)
- [窗口管理器](https://gitee.com/openharmony/window_window_manager_lite)
- [图形表面](https://gitee.com/openharmony/graphic_surface_lite)
- [UI 框架](https://gitee.com/openharmony/arkui_ui_lite)
