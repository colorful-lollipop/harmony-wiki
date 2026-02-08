# 项目概览

## 目的

本文档提供 OpenHarmony wifi-iot 示例应用的总体概览，帮助新人快速理解项目定位、核心能力和适用场景。

## 适用范围

- OpenHarmony 4.0.2 及以上版本
- 基于 LiteOS-M 的轻量级系统（mini 系统类型）
- Wi-Fi IoT 设备开发场景
- 需要学习 SAMGR_Lite、GPIO 操作的开发者

## 关键结论

1. **项目定位**: 这是一个示例应用，展示如何在 OpenHarmony mini 系统上使用 SAMGR_Lite 服务框架和 IoT 硬件接口
2. **技术栈**: 纯 C 代码 + LiteOS-M + SAMGR_Lite，无 JS/N-API 绑定
3. **核心能力**:
   - SAMGR_Lite 服务/特性注册与调用
   - 广播/发布订阅消息机制
   - GPIO 硬件操作（LED 控制）
   - 线程任务管理（CMSIS-OS2）
4. **运行环境**: 轻量级物联网设备（如 HiSpark Pegasus 开发板）
5. **代码规模**: 约 1765 行 C 代码，结构清晰，适合学习参考

## 项目定位与边界

### 项目定位

**wifi-iot 示例应用** 是 OpenHarmony 应用层的一个示例项目，位于 `applications/sample/wifi-iot/` 目录。

证据：
- `bundle.json:2-3` 定义了应用名称和描述：
  ```json
  "name": "@ohos/wifi_iot_sample_app",
  "description": "Samples of wifi_iot"
  ```
- `bundle.json:20` 指定系统类型为 `mini`（轻量级系统）

### 项目边界

**包含**:
- SAMGR_Lite 服务框架示例代码
- IoT 硬件操作示例（GPIO）
- Demo SDK 示例代码
- 基础启动代码

**不包含**:
- 完整的业务逻辑实现
- JS/TypeScript 绑定层（无 N-API）
- 传统 IPC/ServiceAbility（轻量级系统不支持）
- 复杂的权限管理机制

## 核心能力

### 1. SAMGR_Lite 服务框架示例

SAMGR（System Ability Manager Lite）是 OpenHarmony 轻量级系统的服务管理框架。

**提供的示例**:
- 服务（Service）注册与发现
- 特性（Feature）注册与调用
- 默认特性 API（Default Feature API）
- 广播/发布订阅消息机制
- 服务启动顺序控制
- 服务恢复机制

证据：
- `app/samgr/service_example.c:92-93` 注册服务和默认特性 API
- `app/samgr/feature_example.c:188-189` 注册特性和特性 API
- `app/samgr/broadcast_example.c:88` 注册广播服务

### 2. IoT 硬件操作示例

提供 GPIO（通用输入输出）操作示例，用于控制 LED。

**提供的示例**:
- GPIO 初始化
- GPIO 方向设置（输入/输出）
- GPIO 输出值设置
- 线程任务创建与运行

证据：
- `app/iothardware/led_example.c:65-66` 初始化 GPIO 并设置方向
- `app/iothardware/led_example.c:76` 创建任务控制 LED

### 3. Demo SDK 集成示例

展示如何创建和运行自定义 SDK。

**提供的示例**:
- SDK 入口函数
- 任务创建与管理
- 线程优先级与栈大小配置

证据：
- `app/demolink/demosdk.c:33-48` DemoSdkEntry 函数实现

## 运行环境

### 硬件平台

- **目标设备**: Wi-Fi IoT 开发板（如 HiSpark Pegasus）
- **处理器**: 轻量级 IoT 芯片（基于 LiteOS-M）

### 软件依赖

证据：`bundle.json:23-30` 定义的依赖组件

| 组件名称 | 说明 | 证据位置 |
|---------|------|---------|
| utils_lite | 轻量级工具库 | bundle.json:25 |
| liteos_m | LiteOS-M 实时操作系统内核 | bundle.json:26 |
| peripheral | 外设驱动接口 | bundle.json:27 |
| acts | Ability Component Test Suite（测试套件） | bundle.json:28 |
| samgr_lite | 服务管理框架 | bundle.json:29 |

### 构建系统

- GN（Generate Ninja）：OpenHarmony 官方构建系统
- Ninja：实际执行编译的后端工具

## 关键概念

### SAMGR_Lite 框架核心概念

#### Service（服务）
服务是系统功能的抽象单元，具有独立的任务和消息队列。

证据：
- `app/samgr/service_example.c:80-88` 定义 ExampleService 结构

#### Feature（特性）
特性是服务的具体实现，可以按需注册。

证据：
- `app/samgr/feature_example.c:64-76` 定义 DemoFeature 结构

#### IUnknown / IUnknownEntry
接口查询与引用计数机制，用于跨模块调用。

证据：
- `app/samgr/feature_example.c:42-48` 定义 DemoApi 接口

#### Identity
服务/特性的唯一标识，包含 serviceId、featureId、queueId。

证据：
- `app/samgr/feature_example.c:53` 定义 Identity 成员

#### Broadcast（广播）
发布-订阅模式的消息机制，用于模块间异步通信。

证据：
- `app/samgr/broadcast_example.c:31-45` 定义回调函数

### LiteOS-M 相关概念

#### CMSIS-OS2
Cortex Microcontroller Software Interface Standard，线程管理接口。

证据：
- `app/iothardware/led_example.c:19` 包含 cmsis_os2.h

#### osThreadNew
创建新任务的 API。

证据：
- `app/iothardware/led_example.c:76` 使用 osThreadNew 创建任务

### IoT 硬件操作相关概念

#### GPIO（General Purpose Input/Output）
通用输入输出引脚，用于控制硬件设备。

证据：
- `app/iothardware/led_example.c:20` 包含 iot_gpio.h
- `app/iothardware/led_example.c:65` 使用 IoTGpioInit
- `app/iothardware/led_example.c:40` 使用 IoTGpioSetOutputVal

## 适用场景

### 学习与参考

1. **学习 SAMGR_Lite 框架**
   - 了解如何注册服务和特性
   - 掌握跨模块接口调用机制
   - 理解广播消息机制

2. **学习 IoT 硬件操作**
   - GPIO 基础操作
   - 硬件控制与线程结合

3. **了解 OpenHarmony 构建系统**
   - GN 构建配置
   - 组件依赖管理

### 基础开发参考

1. **开发 IoT 应用**
   - 基于此项目模板进行二次开发
   - 参考服务组织结构

2. **集成自定义 SDK**
   - 参考 demolink 模块的集成方式

## 技术约束

### 语言与规范
- **编程语言**: 纯 C 语言
- **编码规范**: 遵循 Apache 2.0 许可证的代码规范
- **线程模型**: CMSIS-OS2（非 pthread）
- **构建系统**: GN + Ninja

### 系统限制
- **无 JS 运行环境**: 不支持 Node.js、N-API
- **无传统 IPC**: 不支持 Binder、ServiceAbility
- **轻量级系统**: 适用于小型 IoT 设备

## 相关跳转链接

- [目录结构与模块职责](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [对外 API 文档](04_N-API_External.md)
- [GN 构建系统](06_GN_Build.md)
- [常见问题与调试](09_QA_Troubleshooting.md)

## 参考文献

1. [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
2. [SAMGR_Lite 源码](https://gitee.com/openharmony/systemabilitymgr_samgr_lite)
3. [LiteOS-M 内核文档](https://gitee.com/openharmony/kernel_liteos_m)
