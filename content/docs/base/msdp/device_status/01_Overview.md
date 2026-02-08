# 项目概览

## 目的

本文档为 OpenHarmony MSDP 设备状态感知框架（device_status）提供完整的项目概览，帮助新人快速理解项目定位、核心能力和运行环境。

---

## 项目定位

### 1. 项目简介

**MSDP 设备状态感知框架** 是 OpenHarmony 多模态感知子系统（MSDP - Multi-Sensory Data Processing）的核心组件，负责识别和传递设备状态给订阅者。

**核心价值**：
- 提供设备状态感知能力（静止、运动、屏幕感知等）
- 支持跨设备协同（鼠标键盘输入共享）
- 提供拖拽交互功能
- 提供元数据绑定能力（Boomerang）
- 提供距离测量能力

### 2. 在系统中的位置

- **子系统**: `msdp`
- **组件名**: `device_status`
- **路径**: `/base/msdp/device_status`
- **SystemAbility ID**: 2902

### 3. 适用场景

| 场景 | 说明 |
|--------|------|
| **设备状态感知** | 识别设备是否静止、处于水平/垂直姿态、皮套开合状态 |
| **跨设备协同** | 在多个设备间共享鼠标和键盘输入，实现跨设备操作 |
| **拖拽交互** | 支持应用内的拖拽操作，包括跨设备拖拽 |
| **屏幕感知** | 识别用户是否在看屏幕，支持页面内容获取 |
| **距离测量** | 基于 BLE/WIFI/UWB 技术测量设备间距离 |
| **元数据绑定** | 支持应用间元数据传递和绑定 |
| **运动感知** | 识别用户操作手、站立拍照、握持状态 |

---

## 核心能力

### 1. 设备状态感知（Stationary）

基于 MSDP 算法库和 SensorHDI 组件识别设备状态：

| 能力 | 说明 |
|------|------|
| **绝对静止** | 利用加速度、陀螺仪等传感器信息识别设备处于绝对静止状态 |
| **水平/垂直姿态** | 利用加速度、陀螺仪等传感器信息识别设备处于水平或垂直状态 |
| **皮套开合事件** | 基于霍尔传感器识别皮套的开合状态 |

### 2. 跨设备协同（Cooperate）

支持在多个设备间共享输入设备：

| 能力 | 说明 |
|------|------|
| **鼠标共享** | 跨设备共享鼠标输入 |
| **键盘共享** | 跨设备共享键盘输入 |
| **虚拟触控板** | 提供虚拟触控板能力 |
| **协同状态管理** | 协同状态的激活、去激活、断开 |

### 3. 拖拽交互（Drag）

提供完整的拖拽交互功能：

| 能力 | 说明 |
|------|------|
| **拖拽启动** | 应用可发起拖拽操作 |
| **拖拽监听** | 应用可注册拖拽事件监听 |
| **数据摘要** | 获取拖拽数据摘要 |
| **预览动画** | 支持拖拽预览和动画 |
| **跨设备拖拽** | 支持跨设备拖拽 |
| **拦截器** | 可选的拖拽拦截功能 |

### 4. 屏幕感知（On-Screen Awareness）

识别用户与屏幕的交互：

| 能力 | 说明 |
|------|------|
| **控制事件** | 发送控制事件到应用 |
| **页面内容获取** | 获取当前屏幕页面内容 |
| **感知触发** | 触发屏幕感知能力（AI 算法、OCR 等） |
| **事件订阅** | 支持屏幕事件订阅 |

### 5. 距离测量（Distance Measurement）

基于多种技术测量设备间距离：

| 能力 | 说明 |
|------|------|
| **BLE RSSI** | 基于 BLE 信号强度测量 |
| **WIFI RSSI** | 基于 WIFI 信号强度测量 |
| **UWB（超宽带）** | 使用 UWB 技术的精确测量 |
| **室内/室外识别** | 识别设备是在室内还是室外 |

### 6. 元数据绑定（Boomerang）

支持应用间元数据传递：

| 能力 | 说明 |
|------|------|
| **图片编码/解码** | 支持图片的元数据绑定 |
| **一步绑定** | 优化的单步绑定流程 |
| **HDR 支持** | 支持 HDR 格式 |

### 7. 运动感知（Motion）

识别用户运动状态：

| 能力 | 说明 |
|------|------|
| **操作手检测** | 识别当前使用的是左手还是右手操作 |
| **站立检测** | 检测用户是否站立 |
| **远程拍照检测** | 检测远程拍照场景 |
| **握持状态** | 识别是否握持设备及握持方式（左手、右手、双手） |

---

## 运行环境

### 1. 系统要求

- **OpenHarmony 版本**: 4.0+
- **依赖子系统**:
  - `accessibility` - 无障碍服务
  - `bundle_framework` - 应用框架
  - `safwk` - 系统能力框架
  - `samgr` - 系统服务管理
  - `ipc` - IPC 通信
  - `sensor` - 传感器子系统
  - `eventhandler` - 事件处理
  - `napi` - N-API 框架
  - `ace_engine` - ArkUI 引擎
  - `image_framework` - 图像框架
  - `window_manager` - 窗口管理
  - `input` - 输入子系统
  - `device_manager` - 设备管理器
  - `dsoftbus` - 分布式软总线
  - `hisysevent` - 系统事件
  - `hitrace` - 性能追踪

### 2. 依赖的外部库

- **cJSON** - JSON 解析
- **jsoncpp** - JSON 处理
- **libxml2** - XML 解析

### 3. 硬件要求

- **传感器**：加速度传感器、陀螺仪、霍尔传感器（可选）
- **输入设备**：鼠标、键盘（用于协同）
- **网络**：支持 BLE/WIFI/UWB 的距离测量

### 4. 内存和存储

| 类型 | 大小 |
|------|------|
| ROM | 2048 KB |
| RAM | ~4096 KB |

---

## System Capabilities (syscap)

本模块导出以下系统能力：

```json
[
  "SystemCapability.MultimodalAwareness.DistanceMeasurement",
  "SystemCapability.MultimodalAwareness.MetadataBinding",
  "SystemCapability.MultimodalAwareness.Motion",
  "SystemCapability.Msdp.DeviceStatus.Stationary",
  "SystemCapability.Msdp.DeviceStatus.Cooperate",
  "SystemCapability.Msdp.DeviceStatus.Drag",
  "SystemCapability.MultimodalAwareness.OnScreenAwareness",
  "SystemCapability.MultimodalAwareness.UserStatus",
  "SystemCapability.MultimodalAwareness.DeviceStatus"
]
```

---

## Feature Flags

项目支持以下特性开关（定义在 `device_status.gni`）：

| Flag | 默认值 | 说明 |
|------|---------|------|
| `device_status_intention_framework` | true | 启用 Intention 框架（核心插件系统） |
| `device_status_rust_enabled` | false | 启用 Rust 实现替代方案 |
| `device_status_interaction_coordination` | false | 启用协同交互功能 |
| `device_status_drag_enable_monitor` | true | 启用拖拽监控 |
| `device_status_drag_enable_interceptor` | false | 启用拖拽拦截器 |
| `device_status_drag_enable_animation` | false | 启用拖拽动画 |
| `device_status_performance_check` | true | 启用性能检查 |
| `device_status_sensor_enable` | true | 启用传感器支持 |
| `device_status_memmgr_enable` | false | 启用内存管理 |
| `device_status_motion_enable` | false | 启用运动感知 |
| `device_status_enable_universal_drag` | false | 启用通用拖拽 |
| `device_status_enable_internal_drop_animation` | false | 启用内部下放动画 |
| `device_status_pullthrow_enable` | false | 启用投掷拖拽 |
| `device_status_boomerang_onestep` | false | 启用 Boomerang 单步模式 |
| `device_status_boomerang_support_hdr` | false | 启用 Boomerang HDR 支持 |
| `device_status_phone_standard_lite` | false | 启用轻量级手机标准模式 |

---

## 关键概念

### 1. Intention 框架

**Intention 框架** 是本项目的核心插件系统，提供：

- **插件化架构**：通过 IPlugin 接口统一管理各种功能插件
- **任务调度**：支持异步和同步任务执行
- **定时器管理**：提供定时任务能力
- **设备管理**：统一管理输入设备的连接和状态
- **跨进程通信**：支持 Socket 和 IPC 通信

### 2. SystemAbility 架构

- **SA ID**: 2902
- **进程名**: "msdp"
- **服务类型**: 非分布式服务
- **启动模式**: 创建时自动启动（`run-on-create: true`）

### 3. IPC 通信模型

- **Binder IPC**：主 IPC 通道，用于服务间通信
- **Unix Domain Socket**：高性能数据传输通道
- **Proxy/Stub 模式**：客户端使用 Proxy，服务端使用 Stub

### 4. 安全模型

- **基于 AccessToken**：使用 OpenHarmony 的访问令牌机制
- **权限验证**：调用方需具有相应权限才能访问敏感功能
- **令牌类型检查**：区分 Native/HAP/Shell 类型
- **系统应用验证**：验证调用方是否为系统应用

---

## 技术栈

| 层次 | 技术 |
|--------|------|
| **应用层** | ArkTS (ETS) / JavaScript |
| **框架层** | N-API (Node-API） |
| **服务层** | C++ SystemAbility |
| **IPC 层** | Binder / Unix Domain Socket |
| **硬件抽象** | Sensor HDI / MMI HDI |
| **算法库** | MSDP 算法库（可选 Rust 实现） |

---

## 相关文档

- **[02_Directory_Structure](02_Directory_Structure.md)** - 详细目录结构和模块职责
- **[03_Architecture](03_Architecture.md)** - 完整架构设计说明
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 参考文档
- **[05_Inner_API](05_Inner_API.md)** - 内部 API 接口定义
- **[06_GN_Targets](06_GN_Targets.md)** - GN 构建目标梳理
- **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物说明
- **[08_Security_Review](08_Security_Review.md)** - 安全风险评审
- **[09_FAQ](09_FAQ.md)** - 常见问题

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
