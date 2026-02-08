# 项目定位与边界

> 详细说明 window_cangjie_wrapper 的边界条件、运行环境和核心概念

---

## 目的

本文档详细说明 `window_cangjie_wrapper` 的项目边界、适用场景和设计决策，帮助开发者理解何时使用以及如何正确使用。

---

## 项目边界

### 功能边界

#### 支持的功能

| 功能模块 | 能力描述 | 证据 |
|----------|----------|--------|
| **窗口管理** | 创建、销毁、属性设置、生命周期管理 | `window.cj` (895 行) |
| **显示设备管理** | Display 查询、折叠屏管理、事件监听 | `display.cj` (726 行) |
| **WindowStage 管理** | 主窗口、子窗口、内容加载 | `window_stage.cj` (157 行) |
| **回调机制** | 窗口事件、尺寸变化、键盘高度等 14 种回调类型 | `cj_window_enum.cj:924-1051` |

#### 不支持的功能

| 功能类别 | 说明 | 证据 |
|----------|--------|--------|
| **画中画窗口** | 画中画基础功能，包括判断系统支持、创建控制器 | `README_zh.md:57-59` |
| **闪控球窗口** | 闪控球基础功能，包括判断设备支持、创建控制器 | `README_zh.md:60-61` |
| **屏幕截图** | 屏幕截图能力 | `README_zh.md:62` |

#### 技术边界

| 边界类型 | 限制 | 证据 |
|----------|--------|--------|
| **设备类型** | 仅支持 standard 设备，不支持 small/mini/wearable 等 | `bundle.json:16` |
| **API Level** | 要求 API Level 22+，更低版本无法使用 | 所有 API `@!APILevel[since: "22"]` |
| **语言** | 仅支持仓颉语言，不支持 ArkTS/TS/JS | 仓颉语法 |
| **上下文** | 仅支持 UIAbilityContext，不支持 ServiceExtension 等 | `window.cj:193-196` |

---

## 运行环境

### 系统要求

| 要求项 | 详细说明 | 证据 |
|--------|----------|--------|
| **操作系统** | OpenHarmony API Level 22+ | `bundle.json:4` (version: "6.1") |
| **设备类型** | standard（标准设备，如手机、平板） | `bundle.json:16` |
| **CPU 架构** | 与 OpenHarmony 标准设备一致（通常为 ARM64） | 未明确指定，依赖系统 |

### 资源占用

根据 `bundle.json:17-18`：

| 资源 | 占用量 | 说明 |
|--------|---------|--------|
| **ROM** | 435KB | 静态库大小 |
| **RAM** | 401KB | 运行时内存占用 |

---

## 权限模型

### 必需权限

| 权限 | 使用场景 | 检查位置 | 证据 |
|--------|----------|----------|--------|
| `ohos.permission.SYSTEM_FLOAT_WINDOW` | 创建 TYPE_FLOAT 类型窗口时需要 | `createWindow()` 前检查 | `window.cj:187` |
| `ohos.permission.PRIVACY_WINDOW` | 设置隐私窗口模式时需要 | `setWindowPrivacyMode()` 前检查 | `window.cj:527` |

### Syscap 要求

| Syscap | 覆盖范围 | 示例 API |
|--------|----------|----------|
| `SystemCapability.WindowManager.WindowManager.Core` | 核心窗口管理能力（窗口创建、属性设置、Display 查询） | `createWindow()`, `getDefaultDisplaySync()` |
| `SystemCapability.Window.SessionManager` | 会话管理能力（窗口状态、折叠屏管理） | `shiftAppWindowFocus()`, `getFoldStatus()` |

**权限检查实现**：通过 Native 层（window_manager 子系统）的权限验证系统进行检查，仓颉封装层仅传递权限注解。

---

## 核心概念

### Window（窗口）

**定义**：应用程序的可视化界面单元，管理内容显示和用户交互。

**类型**：
- `TypeApp` - 应用窗口（默认）
- `TypeMain` - 主窗口（Ability 关联）
- `TypeFloat` - 浮动窗口（需 SYSTEM_FLOAT_WINDOW 权限）
- `TypeDialog` - 对话框窗口

**证据**：`cj_window_enum.cj:32-320`

### WindowStage（窗口阶段）

**定义**：窗口的生命周期阶段，管理主窗口和子窗口。

**方法**：
- `getMainWindow()` - 获取主窗口
- `createSubWindow()` - 创建子窗口
- `getSubWindow()` - 获取所有子窗口
- `loadContent()` - 加载页面内容

**证据**：`window_stage.cj:86-156`

### Display（显示设备）

**定义**：物理显示设备，管理显示属性和事件。

**属性**：
- `id` - 显示设备 ID
- `name` - 显示设备名称
- `state` - 显示状态（On/Off/Doze）
- `refreshRate` - 刷新率（Hz）
- `width/height` - 分辨率
- `densityDpi` - 密度（DPI）

**证据**：`display.cj:496-681`

### Orientation（方向）

**定义**：窗口或显示内容的旋转方向。

**类型**：
- `Portrait` - 竖屏
- `Landscape` - 横屏
- `AutoRotation` - 自动旋转（跟随传感器）
- `Locked` - 锁定当前方向

**证据**：`cj_window_enum.cj:330-535`

---

## 使用场景

### 场景 1：标准应用窗口管理

**适用**：普通 UIAbility 应用创建主窗口和子窗口

**流程**：
1. 获取 AbilityContext
2. 调用 `getLastWindow(ctx)` 获取当前窗口
3. 设置窗口属性（颜色、亮度、方向等）
4. 注册回调（尺寸变化、焦点变化等）

**证据**：`window.cj:224-236`

### 场景 2：浮动窗口

**适用**：系统应用创建浮层窗口（如悬浮球、系统提示）

**要求**：
- 需 `ohos.permission.SYSTEM_FLOAT_WINDOW` 权限
- `windowType = WindowType.TypeFloat`

**证据**：`window.cj:185-209`、`cj_window_enum.cj:52-59`

### 场景 3：折叠屏适配

**适用**：可折叠屏设备上的应用

**流程**：
1. 调用 `isFoldable()` 检测是否支持折叠
2. 调用 `getFoldStatus()` 获取当前折叠状态
3. 调用 `getFoldDisplayMode()` 获取显示模式
4. 注册 `on(FoldStatusChange, callback)` 监听折叠变化
5. 根据状态调整 UI 布局

**证据**：`display.cj:180-214`

### 场景 4：多显示设备管理

**适用**：连接多个显示设备（如折叠屏展开/折叠）

**流程**：
1. 调用 `getAllDisplays()` 获取所有显示设备
2. 调用 `on(ListenerTypeAdd, callback)` 监听显示设备添加
3. 调用 `on(ListenerTypeRemove, callback)` 监听显示设备移除
4. 根据显示设备列表选择合适的显示设备

**证据**：`display.cj:155-169`、`display.cj:295-301`

---

## 设计决策

### 为什么使用 FFI

**决策**：通过仓颉 `foreign` 块调用 Native C/C++ 函数

**原因**：
1. 窗口管理子系统（window_manager）已有成熟的 C++ 实现
2. 避免重复开发，复用现有能力
3. 保证与 ArkTS 版本的功能对等性

**证据**：`window.cj:32-146` - `foreign` 块定义

### 为什么使用 RemoteDataLite

**决策**：所有数据类继承 `RemoteDataLite`

**原因**：
1. 统一管理 Native 资源生命周期
2. 自动处理 FFI 指针的释放
3. 避免资源泄露

**证据**：`window.cj:269-297`、`display.cj:485-690`

### 为什么使用 Mutex 同步

**决策**：使用 `std.sync.Mutex` 保护回调注册

**原因**：
1. 多线程环境下，回调注册/注销可能并发
2. 避免竞态条件
3. 保证回调 Map 的一致性

**证据**：`window.cj:148`、`display.cj:96`

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| 项目边界明确：支持窗口管理和显示设备管理，不支持画中画、闪控球、屏幕截图 | `README_zh.md:57-62` |
| 适用环境：OpenHarmony API Level 22+，仅 standard 设备 | `bundle.json:4,16` |
| 权限模型：SYSTEM_FLOAT_WINDOW（创建浮窗）、PRIVACY_WINDOW（隐私窗口） | `window.cj:187,527` |
| 核心概念：Window（窗口）、WindowStage（窗口阶段）、Display（显示设备）、Orientation（方向） | `cj_window_enum.cj`、`cj_display_enum.cj` |

---

**生成时间**: 2025-02-06
