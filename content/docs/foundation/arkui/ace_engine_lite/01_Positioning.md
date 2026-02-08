# 项目定位与边界

## 目的

本文档详细说明 ace_engine_lite 项目的**定位、边界、核心能力**和**运行环境**，帮助理解其适用场景和限制。

## 适用范围

- **子系统**：arkui.ace_engine_lite
- **版本**：3.1
- **平台**：轻量系统（mini/small）

---

## 项目定位

### 核心定位

Ace Engine Lite 是 OpenHarmony 为**轻量级设备**（IoT、可穿戴、智能家居）提供的 **JavaScript UI 开发框架**。

**设计理念**：
- **轻量化优先**：最小化 ROM 和 RAM 占用
- **性能优先**：使用 JerryScript 轻量级 JS 引擎
- **渐进式增强**：通过 Feature Flags 按需启用功能
- **开发者友好**：提供声明式 UI 和数据绑定

### 与完整版差异

| 特性 | Ace Engine Lite | Ace Engine (完整版) |
|------|----------------|---------------------|
| 目标平台 | 轻量系统 (mini/small) | 标准系统 (standard/large) |
| JS 引擎 | JerryScript | V8 + ArkTS |
| 编程语言 | JavaScript | ArkTS (TypeScript 扩展) |
| 组件数量 | 基础组件集 | 完整组件集 |
| 动画系统 | 基础过渡动画 | 完整动画系统 |
| N-API 支持 | ❌ | ✅ |
| 预计 ROM | ~521KB | 更大（具体值未确认） |
| 预计 RAM | ~82KB | 更大（具体值未确认） |

**证据**：`bundle.json:22-23`（adapted_system_type: ["mini","small"]）

---

## 能力边界

### 支持能力

#### 1. 声明式 UI 渲染

**支持**：
- 基础 UI 组件（div、text、image、list、slider 等）
- 嵌套组件结构
- 属性绑定（data-*）
- 事件绑定（onclick、ontouch 等）

**证据**：`frameworks/src/core/components/` 目录（20+ 组件实现）

#### 2. 数据绑定

**支持**：
- JS 对象属性监听（watch）
- 自动 UI 更新（属性变化触发重新渲染）
- 类型转换（JS → C++）
- 数据校验（类型、范围）

**证据**：`frameworks/src/core/base/js_fwk_common.h:238-269`（Watcher 机制）

#### 3. 路由与页面管理

**支持**：
- 页面替换（router.replace）
- 页面参数传递
- 页面状态管理（页面栈）

**限制**：
- 不支持复杂路由（如嵌套路由）
- 不支持路由守卫（navigation guards）

**证据**：`frameworks/src/core/modules/router_module.cpp:26-30`

#### 4. 样式系统

**支持**：
- 内联样式（style 属性）
- 样式表（.css 文件）
- ID/Class 选择器
- 动态样式绑定
- 媒体查询（@media）

**限制**：
- 不支持 CSS 预处理器（Sass/Less）
- 不支持 CSS 模块（@import）

**证据**：`frameworks/src/core/stylemgr/` 目录

#### 5. JS 模块系统

**支持**：
- 模块按需加载（require）
- 内置模块列表（19+ 模块）
- 条件模块（Feature Flags）
- 异步操作（Promise/Callback）

**内置模块**：
- app - 应用信息管理
- router - 路由
- timer - 定时器
- console - 日志输出
- fetch - HTTP 请求（可选）
- storage - KV 存储（可选）
- 等等

**证据**：`frameworks/module_manager/ohos_module_config.h:97-158`

### 不支持能力

#### 1. N-API (Node-API)

项目不使用标准的 N-API，而是使用 **JerryScript 原生 API** + JSI 封装层。

**影响**：
- 无法直接复用 N-API 模块
- 需要专门为 JerryScript 编写原生模块
- API 稳定性依赖于 JerryScript 版本

**证据**：`interfaces/inner_api/builtin/jsi/jsi.h`（JSI 定义）

#### 2. System Ability 直接交互

不支持直接与 System Ability 通信，需要通过 Feature Ability 模块中转。

**限制**：
- 跨进程通信复杂度增加
- 需要 Feature Ability 适配层

**证据**：`frameworks/src/core/modules/presets/feature_ability_module.cpp`（FeatureAbility 封装）

#### 3. 复杂动画

仅支持基础过渡动画（transition），不支持：
- 关键帧动画
- 路径动画
- 物理动画

**证据**：`frameworks/src/core/animation/transition_impl.cpp`

#### 4. WebAssembly

不支持 WebAssembly，无法加载 .wasm 模块。

---

## 运行环境要求

### 硬件要求

| 资源类型 | 最小要求 | 推荐配置 |
|---------|---------|----------|
| ROM | 521KB | 1MB+ |
| RAM | 82KB | 256KB+ |
| Flash | - | 512KB+ |

**证据**：`bundle.json:22-23`（rom: "521KB", ram: "~82KB"）

### 软件要求

**必需**：
- JerryScript JavaScript 引擎
- UI Lite 图形框架
- Ability Lite 框架
- Bundle Framework Lite

**可选**（条件启用）：
- Surface Lite
- Media Lite
- Camera Lite
- Battery Lite
- KV Store

**证据**：`bundle.json:24-47`（deps 列表）

### 编译器支持

- **ICCARM**（IAR 编译器）：LiteOS-M 平台
- **GCC/Clang**：LiteOS-A/Linux 平台

**证据**：`frameworks/BUILD.gn:31-36`（cflags 配置）

---

## 应用场景

### 适合场景

1. **可穿戴设备**：智能手表、健康手环
2. **智能家居**：温湿度计、智能插座
3. **物联网网关**：边缘计算节点
4. **轻量工业**：HMI 显示终端

### 不适合场景

1. **手机应用**：需要完整版 Ace Engine
2. **平板应用**：需要完整版 Ace Engine
3. **高性能计算**：需要 N-API 和复杂动画
4. **Web 应用**：不支持 WebAssembly

---

## 系统能力

### SystemCapability

```
SystemCapability.ArkUI.ArkUI.Lite
```

**能力等级**：Lite（轻量级）

**证据**：`bundle.json:16`（syscap: ["SystemCapability.ArkUI.ArkUI.Lite"]）

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 概览和核心组件
- [02_Directory_Structure.md](02_Directory_Structure.md) - 代码组织
- [04_JS_API.md](04_JS_API.md) - JS 模块 API
