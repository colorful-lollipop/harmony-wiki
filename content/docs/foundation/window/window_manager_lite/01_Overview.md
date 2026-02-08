# 项目概述

## 项目定位

`window_manager_lite` 是 OpenHarmony 图形子系统的核心组件，提供**窗口管理服务（WMS）**和**输入管理服务（IMS）**两大能力。

## 核心能力

### 1. 窗口管理服务 (WMS)

| 能力 | 说明 |
|------|------|
| 窗口生命周期 | 创建、销毁、显示、隐藏 |
| 窗口几何管理 | 调整大小、移动位置、层级控制 |
| 截图功能 | 屏幕截图（需权限） |
| 输入事件分发 | 接收 RawEvent 并分发到目标窗口 |

**证据**: `services/wms/lite_wms.h:24-55` - 定义了 WMS 的所有请求处理器

### 2. 输入管理服务 (IMS)

| 能力 | 说明 |
|------|------|
| 输入设备管理 | 打开/关闭输入设备 |
| 事件读取 | 从设备读取 RawEvent |
| 事件分发 | 将事件分发给目标窗口 |

**证据**: `services/ims/input_manager_service.h:26-95` - InputManagerService 定义

## 运行环境

| 属性 | 值 |
|------|-----|
| 适配系统类型 | small（轻量系统） |
| ROM 占用 | ~110KB |
| RAM 占用 | ~50KB |
| 依赖子系统 | samgr_lite, surface_lite, graphic_utils_lite |

**证据**: `bundle.json:17-18` - ROM/RAM 规格

## 关键概念

### 1. 窗口与 Surface 绑定

每个 `IWindow` 对象关联一个 `ISurface`，用于图形渲染：

```
IWindow → ISurface → SurfaceBuffer
```

**证据**: `interfaces/innerkits/iwindow.h:78` - `GetSurface()` 接口

### 2. 窗口 ID 管理

- 最大窗口数：**32 个** (`lite_wm.cpp:95`)
- 使用位图存储可用 ID (`lite_wm.cpp:96`)

### 3. 窗口合成模式

| 模式 | 说明 |
|------|------|
| `COPY` | 完全覆盖模式下层窗口 |
| `BLEND` | 混合模式（支持透明度） |

**证据**: `interfaces/innerkits/lite_wm_type.h:24-27`

## 技术架构概览

```
┌─────────────────────────────────────────────────────┐
│                    Application                     │
│  ┌─────────────────────────────────────────────┐  │
│  │            libwms_client.so                  │  │
│  │  ┌─────────────┐  ┌─────────────────────┐  │  │
│  │  │ LiteWMClient│  │  LiteProxyWindow    │  │  │
│  │  └─────────────┘  └─────────────────────┘  │  │
│  └─────────────────────────────────────────────┘  │
└────────────────────────┬──────────────────────────┘
                         │ IPC (SAMGR)
                         ▼
┌─────────────────────────────────────────────────────┐
│                 wms_server (进程)                   │
│  ┌─────────────────────────────────────────────┐  │
│  │            LiteWMS                          │  │
│  │  ┌───────────┐  ┌──────────────────────┐  │  │
│  │  │ LiteWM    │  │  InputManagerService │  │  │
│  │  │ (窗口管理) │  │  (输入管理)           │  │  │
│  │  └───────────┘  └──────────────────────┘  │  │
│  └─────────────────────────────────────────────┘  │
│           │                    │                   │
│           ▼                    ▼                   │
│  ┌──────────────┐   ┌──────────────────┐        │
│  │ Display HAL   │   │ Input Device HAL  │        │
│  └──────────────┘   └──────────────────┘        │
└─────────────────────────────────────────────────────┘
```

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [graphic_surface_lite](https://gitee.com/openharmony/graphic_surface_lite) | Surface 内存管理 |
| [arkui_ui_lite](https://gitee.com/openharmony/arkui_ui_lite) | UI 组件框架 |
| [graphic_graphic_utils_lite](https://gitee.com/openharmony/graphic_graphic_utils_lite) | 图形工具库 |
