# Ace Engine Lite 概览

## 目的

本文档提供 OpenHarmony **arkui_ace_engine_lite** 子系统的高层次概览，帮助读者快速理解项目定位、核心组件和运行环境。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 部署平台：轻量系统（mini/small）
- JS 引擎：JerryScript

---

## 项目定位

### 核心定位

Ace Engine Lite 是 OpenHarmony 为**轻量系统**提供的 **JS-UI 框架**，支持开发者使用 JavaScript 开发用户界面。

**设计目标**：
- 轻量化：ROM ~521KB，RAM ~82KB
- 高性能：基于 JerryScript 引擎的快速执行
- 跨平台：支持 LiteOS-A、LiteOS-M、Linux
- 模块化：功能通过 Feature Flags 按需启用

### 系统类型适配

| 系统类型 | 说明 | 支持状态 |
|---------|------|----------|
| LiteOS-A | 微控制器系统 | ✅ |
| LiteOS-M | 轻量级嵌入式系统 | ✅ |
| Linux | 模拟器/PC 调试环境 | ✅ |
| Standard | 标准版 OpenHarmony | ❌ |

### 能力边界

**支持**：
- 声明式 UI 组件渲染
- 数据绑定（JS-C++ 双向绑定）
- 路由和页面管理
- 样式系统（CSS 类似）
- 异步任务和定时器
- 模块化 JS API

**不支持**：
- N-API 标准（使用 JerryScript 原生 API）
- System Ability 直接交互（通过 Feature Ability 中转）
- 复杂动画系统（仅基础过渡动画）
- WebAssembly

---

## 核心组件

### 三层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                    应用层 (Application Layer)                     │
│              JavaScript 应用代码 (JS/TS)                        │
└─────────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                  JS 框架层 (JS Framework)                   │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │ 模块管理器   │  │   JSI 层    │                     │
│  │ModuleManager  │  │(JerryScript)  │                     │
│  └──────────────┘  └──────────────┘                    │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │ 核心模块     │  │   路由       │                    │
│  │   Modules    │  │   Router     │                    │
│  └──────────────┘  └──────────────┘                    │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │ 组件系统     │  │   样式管理   │                    │
│  │ Components   │  │  StyleManager  │                    │
│  └──────────────┘  └──────────────┘                    │
└─────────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│               JS 运行时层 (JS Runtime)                      │
│                   JerryScript 引擎                             │
│              (解析和执行 JavaScript 代码)                        │
└─────────────────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│              图形层 (Graphics Layer)                         │
│                   UI Lite (2D 图形框架)                      │
└─────────────────────────────────────────────────────────────────┘
```

### 组件职责

#### 1. JS Data Binding
- **位置**: `frameworks/src/core/directive/`, `frameworks/src/core/wrapper/`
- **职责**: 实现 JS 对象与 C++ 对象的双向绑定
- **关键机制**:
  - Watcher 模式：监听 JS 属性变化
  - Lazy Load：延迟加载组件
  - 数据校验：类型转换和范围检查

**证据**：`frameworks/src/core/base/js_fwk_common.h:238-269`（WatcherCallbackFunc）

#### 2. JS Runtime
- **位置**: `third_party/jerryscript`（外部依赖）
- **职责**: JavaScript 代码解析和执行
- **集成方式**: 通过 JSI 封装层

**证据**：`frameworks/native_engine/jsi/jsi.h:52`（JSIValue typedef）

#### 3. JS Framework
- **位置**: `frameworks/src/core/`
- **职责**: 提供组件、路由、样式等框架机制
- **组成**:
  - **Components**: UI 组件实现（div、text、image 等）
  - **Router**: 页面路由和状态管理
  - **StyleManager**: 样式解析和应用
  - **Modules**: JS 模块（app、router、timer 等）

**证据**：`frameworks/src/core/components/component.h`（Component 基类）

---

## 关键概念

### JSI (JavaScript Interface)

JSI 是 JerryScript 引擎的 C++ 封装层，提供统一的 JS-C++ 绑定接口。

**核心类型**：
```cpp
typedef jerry_value_t JSIValue;  // JS 值类型
typedef JSIValue (*JSIFunctionHandler)(const JSIValue thisVal, const JSIValue *args, uint8_t argsNum);
```

**关键操作**：
- 值创建：`CreateObject()`, `CreateNumber()`, `CreateString()`, `CreateUndefined()`
- 属性操作：`SetProperty()`, `GetNamedProperty()`
- 函数调用：`CallJSFunction(func, context, args, argsCount)`
- 内存管理：`ReleaseValue()`, `AcquireValue()`

**证据**：`frameworks/native_engine/jsi/jsi.h:82-325`

### 模块系统

JS 模块通过 `ModuleManager` 按需加载，模块名为 `category.name` 格式。

**加载流程**：
1. JS 代码调用 `require("system.router")`
2. `ModuleManager::RequireModule()` 解析模块名
3. 在 `OHOS_MODULES` 数组中查找
4. 调用 `InitRouterModule(exports)` 初始化
5. 返回 exports 对象给 JS

**证据**：`frameworks/module_manager/ohos_module_config.h:97-158`

### 组件生命周期

```
创建 (Create)
  ↓
解析属性 (Parse Attributes)
  ↓
应用样式 (Apply Styles)
  ↓
渲染 (Render)
  ↓
事件处理 (Handle Events)
  ↓
销毁 (Destroy)
```

**证据**：`frameworks/src/core/components/component.h`（Component 类定义）

---

## 运行环境

### 编译配置

**关键变量**：
- `ohos_kernel_type`: liteos_a / liteos_m / linux
- `ohos_build_type`: debug / release
- `build_lite_full`: 完整/精简构建

**Feature Flags**：控制功能模块（见 [06_GN_Targets.md](06_GN_Targets.md)）

### 依赖组件

**必需依赖**（`bundle.json:24-47`）：
- `jerryscript`: JavaScript 引擎
- `ui_lite`: 2D 图形框架
- `i18n_lite`: 国际化
- `resource_management_lite`: 资源管理
- `kv_store`: KV 存储
- `ability_lite`: Ability 框架

**可选依赖**（条件启用）：
- `surface_lite`: Surface 支持
- `media_lite`: 媒体播放
- `camera_lite`: 相机功能
- `battery_lite`: 电池管理

---

## 相关跳转

- [01_Positioning.md](01_Positioning.md) - 详细定位和边界
- [02_Directory_Structure.md](02_Directory_Structure.md) - 代码组织
- [03_Architecture.md](03_Architecture.md) - 深入架构设计
- [04_JS_API.md](04_JS_API.md) - JS 模块 API 详情
