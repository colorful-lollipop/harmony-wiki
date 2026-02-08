# Ace Engine 项目概览

> **文档版本**: v1.0  
> **更新时间**: 2026-02-06  
> **源码版本**: OpenHarmony ace_engine

---

## 📋 目录

1. [项目定位](#项目定位)
2. [核心能力](#核心能力)
3. [模块边界](#模块边界)
4. [运行环境](#运行环境)
5. [关键概念](#关键概念)

---

## 项目定位

### 1.1 项目概述

**Ace Engine** (`@ohos/ace_engine`) 是 OpenHarmony **ArkUI** 框架的核心执行引擎，负责从组件描述到最终渲染的完整管线。

> **证据来源**: `bundle.json:3`
> ```json
> {
>   "name": "@ohos/ace_engine",
>   "description": "ArkUI Cross-Platform Engine for UI layout measure and paint"
> }
> ```

### 1.2 在 OpenHarmony 中的位置

```
OpenHarmony 系统
├── 应用层 (Applications)
├── 框架层 (Framework)
│   ├── ArkUI (声明式 UI)
│   │   ├── ace_engine (核心引擎) ← 本项目
│   │   └── arkui_napi (N-API 接口)
│   └── 其他子系统
├── 系统服务层 (System Services)
└── 内核层 (Kernel)
```

### 1.3 核心职责

| 职责 | 描述 | 证据 |
|------|------|------|
| 组件解析 | 解析声明式 UI 描述，构建组件树 | `frameworks/bridge/` |
| 布局计算 | 测量尺寸、计算位置 | `frameworks/core/components_ng/layout/` |
| 渲染绘制 | 生成渲染命令并绘制 | `frameworks/core/components_ng/render/` |
| 事件处理 | 手势识别、用户交互 | `frameworks/core/components_ng/gestures/` |

---

## 核心能力

### 2.1 前端支持

Ace Engine 支持多种前端范式：

| 前端类型 | 语言 | 用途 | 证据 |
|---------|------|------|------|
| **Declarative Frontend** | ArkTS/TypeScript | 推荐使用，现代声明式 UI | `frameworks/bridge/declarative_frontend/` |
| **ArkTS Frontend** | ArkTS static | 基于 ArkTS 的增量引擎 | `frameworks/bridge/arkts_frontend/` |
| **JavaScript Frontend** | JavaScript | 传统 Web 风格开发 | `frameworks/bridge/js_frontend/` |
| **Cangjie Frontend** | Cangjie | Cangjie 语言支持 | `frameworks/bridge/cj_frontend/` |

### 2.2 组件系统

支持丰富的 UI 组件类型：

| 类型 | 组件示例 | 证据 |
|------|----------|------|
| **基础组件** | Button, Text, Image, TextInput | `frameworks/core/components_ng/pattern/` |
| **容器组件** | Column, Row, Grid, List, Stack | `frameworks/core/components_ng/pattern/` |
| **选择器** | DatePicker, TimePicker, Slider | `frameworks/core/components_ng/pattern/` |
| **形状组件** | Rect, Circle, Path, Polygon | `frameworks/core/components_ng/svg/` |
| **媒体组件** | Video, Canvas, ImageAnimator | `frameworks/core/components_ng/` |
| **高级组件** | Menu, Dialog, Navigation | `frameworks/core/components_ng/pattern/` |

### 2.3 状态管理

提供完整的状态管理框架：

| 能力 | 用途 | 证据 |
|------|------|------|
| **AppStorage** | 应用级状态管理 | `frameworks/bridge/declarative_frontend/state_mgmt/` |
| **LocalStorage** | 页面级状态管理 | `frameworks/bridge/declarative_frontend/state_mgmt/` |
| **@Watch** | 属性观察与响应 | `frameworks/bridge/declarative_frontend/state_mgmt/` |
| **@Link/@Prop** | 父子组件数据绑定 | `frameworks/bridge/declarative_frontend/state_mgmt/` |

---

## 模块边界

### 3.1 目录结构概览

```
ace_engine/
├── adapter/              # 平台适配层 (Platform Adapter)
│   ├── ohos/            # OpenHarmony 平台实现
│   └── preview/         # 预览工具支持
├── frameworks/          # 框架代码
│   ├── base/            # 基础工具和公共库
│   ├── bridge/          # 前端-后端桥接层
│   │   ├── declarative_frontend/  # ArkTS/TS 声明式UI
│   │   ├── arkts_frontend/        # ArkTS 语言支持
│   │   ├── js_frontend/           # JavaScript 前端
│   │   └── cj_frontend/          # Cangjie 前端
│   └── core/            # 核心组件和渲染
│       ├── components/    # 遗留组件实现
│       └── components_ng/ # 新一代组件 (推荐)
├── interfaces/           # 公共 API 接口
│   ├── inner_api/       # 内部 API
│   └── native/          # NDK/N-API 接口
├── test/                # 测试 (不作为文档证据来源)
└── build/               # 构建配置
```

### 3.2 模块职责

| 模块 | 职责 | 边界 |
|------|------|------|
| `adapter/ohos/` | OpenHarmony 平台抽象 (窗口、输入、渲染) | 与系统服务交互 |
| `frameworks/bridge/` | 语言解析、状态管理、组件树构建 | 前端语言 → 组件树 |
| `frameworks/core/` | 组件逻辑、布局算法、渲染管线 | 组件树 → 渲染命令 |
| `interfaces/native/` | N-API 导出接口 | C++ → JS/ArkTS 绑定 |

### 3.3 依赖方向

```
JS/ArkTS 应用
    ↓
frameworks/bridge/ (解析 + 状态管理)
    ↓
frameworks/core/components_ng/ (组件 + 布局 + 渲染)
    ↓
adapter/ (平台抽象)
    ↓
OpenHarmony 系统服务
```

---

## 运行环境

### 4.1 目标平台

| 平台 | 支持状态 | 证据 |
|------|----------|------|
| **OpenHarmony Standard** | ✅ 支持 | `bundle.json:46` |
| **OpenHarmony Lite** | ✅ 支持 | `bundle.json:47` |

### 4.2 系统依赖

Ace Engine 依赖以下 OpenHarmony 子系统：

| 依赖子系统 | 用途 | 证据 |
|-----------|------|------|
| `ability_runtime` | 能力运行时 | `bundle.json:60` |
| `window_manager` | 窗口管理 | `bundle.json:69` |
| `input` | 输入事件 | `bundle.json:96` |
| `accessibility` | 无障碍支持 | `bundle.json:58` |
| `hiview` | 日志与诊断 | `bundle.json:67` |
| `ipc` / `samgr` | IPC 通信 | `bundle.json:61-62` |
| `bundle_framework` | 包管理 | `bundle.json:72` |
| `resource_management` | 资源管理 | `bundle.json:85` |
| `image_framework` | 图片处理 | `bundle.json:92` |
| `access_token` | 权限管理 | `bundle.json:95` |

> **完整依赖列表**: 参见 `bundle.json:54-126`

### 4.3 系统能力 (SysCap)

| 能力标识 | 描述 |
|---------|------|
| `SystemCapability.ArkUI.ArkUI.Full` | 完整 ArkUI 能力 |
| `SystemCapability.ArkUI.ArkUI.Lite` | 轻量 ArkUI 能力 |
| `SystemCapability.ArkUI.ArkUI.Circle` | 圆形 UI 能力 |

> **证据来源**: `bundle.json:15-19`

---

## 关键概念

### 5.1 组件生命周期

Ace Engine 组件经历以下阶段：

```
┌─────────────────────────────────────────────────────────┐
│  组件生命周期 (Component Lifecycle)                     │
│                                                          │
│  1. Parsing (解析)                                      │
│     Frontend 解析 ArkTS/JS 代码 → 组件描述              │
│                                                          │
│  2. Building (构建)                                      │
│     创建 FrameNode → 初始化 Pattern + Properties        │
│                                                          │
│  3. Layout (布局)                                        │
│     Measure → Layout → Position Calculation             │
│                                                          │
│  4. Render (渲染)                                        │
│     RenderNode → Drawing → Display                      │
└─────────────────────────────────────────────────────────┘
```

### 5.2 Four-Layer Architecture

Ace Engine 采用四层架构：

| 层级 | 位置 | 职责 |
|------|------|------|
** | ArkTS 应用层 | 声明式 UI 描述 |
| **| **Application LayerBridge Layer** | `frameworks/bridge/` | 语言解析、状态管理、组件树构建 |
| **Component Framework** | `frameworks/core/` | 组件逻辑、布局算法、渲染 |
| **Platform Adapter** | `adapter/` | 平台抽象 (图形、输入、窗口) |

### 5.3 Pattern-Model-View

Components NG 架构遵循 Pattern-Model 分离：

| 概念 | 位置 | 职责 |
|------|------|------|
| **Pattern** | `*_pattern.h/cpp` | 业务逻辑与生命周期 |
| **Model** | `*_model.h/cpp` | 数据模型接口 |
| **Layout Property** | `*_layout_property.h/cpp` | 布局相关属性 |
| **Paint Property** | `*_paint_property.h/cpp` | 渲染相关属性 |
| **Event Hub** | `*_event_hub.h/cpp` | 事件处理 |

> **证据来源**: `CLAUDE.md` - "Component Creation Flow (NG Architecture)"

### 5.4 修饰器模式 (Modifier Pattern)

属性更新使用修饰器模式：

```typescript
// 示例
Text()
  .width(100)
  .height(50)
  .fontSize(16)
  .fontColor(Color.Red)
```

> **证据来源**: `CLAUDE.md` - "Property System"

---

## 🔗 相关文档

- 架构详解: [01_Architecture](01_Architecture.md)
- N-API 接口: [02_NAPI](02_NAPI.md)
- 构建系统: [03_Build](03_Build.md)
