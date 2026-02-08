# 项目定位 (Project Positioning)

## 目的

本文档描述 distributed_input 模块的项目定位、功能边界、核心能力、运行环境和关键概念。

## 适用范围

本文档适用于以下场景：
- 理解分布式输入模块在整个 OpenHarmony 生态中的位置
- 了解模块的功能边界和能力范围
- 掌握运行环境要求和约束条件

## 关键结论

1. **功能定位**: 提供跨设备的输入外设控制能力，不直接面向应用开发者
2. **架构定位**: 纯 native C++ 模块，使用 IPC/SA 机制，无 JavaScript/N-API 层
3. **消费方**: 主要消费方为多模输入子系统（multimodalinput_input）
4. **依赖框架**: 依赖分布式硬件管理框架（distributed_hardware_fwk）和设备管理（device_manager）
5. **运行约束**: 必须在局域网中组网，需要 OpenHarmony 标准系统

## 功能边界

### 提供的功能

分布式输入模块提供以下核心能力：

| 能力 | 描述 | 实现方式 |
|------|------|----------|
| **跨设备输入** | 一台设备可以使用另一台设备的输入外设（鼠标、键盘、触摸板等）在本设备进行输入操作 | 通过虚拟输入驱动 + SoftBus 事件传输 |
| **外设共享** | 被控端提供本地外设供主控端使用 | 分布式硬件框架同步外设规格信息 |
| **事件过滤** | 根据白名单过滤组合键，限制某些键只能在本地生效 | EventFilter + 白名单配置 |
| **状态同步** | 跨设备同步外设状态（ THROUGH_IN/THROUGH_OUT） | SoftBus 消息传输 |
| **仿真事件** | 支持仿真事件监听和注入 | SimulationEventListener 接口 |

### 不提供的功能

| 功能 | 说明 | 真实位置 |
|------|------|----------|
| **JavaScript API** | 不提供面向应用的 JS API | 位于 multimodalinput_input 模块 |
| **直接设备管理** | 不直接管理设备发现和组网 | 由 device_manager 模块负责 |
| **网络传输** | 不直接提供网络传输能力 | 依赖 dsoftbus（软总线） |
| **本地输入处理** | 不处理本地输入设备的常规输入 | 由多模输入子系统负责 |

## 核心能力

### 1. Source 侧能力

主控端设备（Source）提供以下能力：

| 能力 | 接口 | 说明 |
|------|------|------|
| **管理虚拟驱动** | `RegisterDistributedHardware` / `UnregisterDistributedHardware` | 为远程外设创建/销毁虚拟输入驱动节点 |
| **准备远程输入** | `PrepareRemoteInput` | 准备跨设备输入，建立 SoftBus 会话 |
| **启动远程输入** | `StartRemoteInput` | 启动跨设备输入，远程事件开始在本地生效 |
| **停止远程输入** | `StopRemoteInput` | 停止跨设备输入，远程事件停止生效 |
| **取消准备** | `UnprepareRemoteInput` | 释放跨设备输入资源，断开会话 |
| **注册事件监听** | `RegisterSimulationEventListener` | 注册仿真事件监听器 |

### 2. Sink 侧能力

被控端设备（Sink）提供以下能力：

| 能力 | 接口 | 说明 |
|------|------|------|
| **采集输入事件** | `StartCollectionThread` | 从本地输入驱动采集原始输入事件 |
| **发送输入事件** | SoftBus 传输 | 将采集的输入事件发送到主控端 |
| **事件过滤** | `IsNeedFilterOut` | 根据白名单过滤敏感事件（如锁屏键） |
| **屏幕信息同步** | `NotifyStartDScreen` / `NotifyStopDScreen` | 同步屏幕信息用于坐标映射 |

### 3. 框架集成能力

与分布式硬件管理框架的集成能力：

| 能力 | 接口 | 说明 |
|------|------|------|
| **设备能力查询** | `QueryMeta` / `Query` | 查询本地输入外设的能力和元数据 |
| **组件接入** | `InitSource` / `InitSink` | 接入分布式硬件框架，响应设备上线/下线 |
| **配置下发** | `ConfigDistributedHardware` / `SubscribeLocalHardware` | 接收分布式硬件管理框架的配置 |

## 运行环境

### 系统要求

| 要求项 | 说明 | 证据 |
|--------|------|------|
| **操作系统** | OpenHarmony 标准系统 | [bundle.json:19-21](../bundle.json:19-21) |
| **子系统** | distributedhardware（分布式硬件子系统） | [bundle.json:16](../bundle.json:16) |
| **架构支持** | 标准系统（standard） | [bundle.json:19-21](../bundle.json:19-21) |

### 网络要求

| 要求项 | 说明 | 证据 |
|--------|------|------|
| **组网环境** | 设备必须在同一个局域网中 | [README_zh.md:97](../README_zh.md:97) |
| **SoftBus 支持** | 需要支持软总线协议 | [bundle.json:39](../bundle.json:39) |
| **帐号认证** | 部分操作需要同一帐号 | [distributedinput.gni:51-56](../distributedinput.gni:51-56) |

### 硬件要求

| 设备类型 | 说明 |
|---------|------|
| **输入外设** | 键盘、鼠标、触摸板等 |
| **虚拟驱动** | 主控端支持虚拟输入驱动节点 |
| **本地驱动** | 被控端有实际的本地输入驱动节点 |

## 关键概念

### 设备角色

| 角色 | 英文名称 | 说明 |
|------|---------|------|
| **主控端** | Source | 向被控端设备发送指令，使用其外设输入的能力 |
| **被控端** | Sink | 接受主控端发送的指令并且完成对应操作，提供本地外设供主控端设备使用 |

### 输入类型

根据 [constants_dinput.h](../common/include/constants_dinput.h:44-76) 定义：

| 输入类型 | 常量值 | 说明 |
|---------|---------|------|
| **MOUSE** | 0x1 | 鼠标输入 |
| **KEYBOARD** | 0x2 | 键盘输入 |
| **TOUCHSCREEN** | 0x4 | 触摸屏输入 |

### 设备状态

根据 [dinput_sink_state.h](../services/state/include/dinput_sink_state.h:32-39) 定义：

| 状态 | 常量值 | 说明 |
|------|---------|------|
| **THROUGH_IN** | 0 | 设备在跨设备输入状态，事件穿透到远程 |
| **THROUGH_OUT** | 1 | 设备在本地输入状态，事件在本地生效 |

### 会话状态

根据 [constants_dinput.h](../common/include/constants_dinput.h:262-270) 定义：

| 状态 | 常量值 | 说明 |
|------|---------|------|
| **SESSION_STATE_INIT** | 0 | 会话初始化中 |
| **SESSION_STATE_PREPARED** | 1 | 会话已准备 |
| **SESSION_STATE_STARTED** | 2 | 会话已启动 |
| **SESSION_STATE_STOPPED** | 3 | 会话已停止 |
| **SESSION_STATE_UNPREPARED** | 4 | 会话已取消准备 |

## 性能指标

### 资源占用

根据 [bundle.json](../bundle.json:22-23)：

| 资源类型 | 占用量 |
|---------|-------|
| **ROM** | 16384KB |
| **RAM** | 15360KB |

### 延迟要求

- **低延迟**: 实时输入传输要求低延迟（基于 dsoftbus）
- **实时性**: 输入事件需要实时传输，避免明显延迟

## 设计约束

### 安全约束

1. **权限要求**: 所有操作都需要相应权限（详见[安全评审](08_Security_Review.md)）
2. **访问控制**: 需要通过设备管理的 ACL 检查
3. **同账号限制**: 部分操作受 check_same_account 标志控制

### 技术约束

1. **无 N-API**: 不提供 JavaScript 接口，仅提供 C++ Inner SDK
2. **依赖框架**: 必须依赖分布式硬件管理框架和设备管理
3. **SoftBus 依赖**: 跨设备通信必须通过 SoftBus

## 相关跳转

- [架构设计](03_Architecture.md) - 详细的架构图和组件关系
- [目录结构](02_Directory_Structure.md) - 代码组织结构
- [公共 API](04_Public_API.md) - Inner SDK 接口说明
- [安全评审](08_Security_Review.md) - 权限和安全机制

---

*更新时间: 2026-02-06 15:08:55*
