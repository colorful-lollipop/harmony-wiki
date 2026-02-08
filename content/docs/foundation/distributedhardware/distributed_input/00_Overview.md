# 概览 (Overview)

## 目的

本文档提供 OpenHarmony distributed_input（分布式输入）模块的全面概览，包括项目定位、核心概念、架构设计和主要功能。

## 适用范围

本文档适用于以下读者：
- 新加入项目的开发者 - 理解整体架构和概念
- 功能开发者 - 了解分布式输入能力如何使用
- 架构师 - 理解组件关系和设计模式
- 安全审计员 - 了解安全机制和风险点

## 关键结论

1. **模块定位**: distributed_input 是纯 native C++ 模块，提供跨设备输入外设控制能力
2. **无 N-API**: 本模块不提供 JavaScript/N-API 接口，JS API 位于独立的 multimodalinput_input 模块
3. **双角色架构**: Source（主控端）和 Sink（被控端）两个角色协同工作
4. **基于 SA**: 使用 System Ability（SA 4809 和 4810）和 IPC 机制实现跨进程通信
5. **权限控制**: 使用 OpenHarmony 的权限系统（ENABLE_DISTRIBUTED_HARDWARE、ACCESS_DISTRIBUTED_HARDWARE）进行访问控制

## 核心概念

### Source（主控端）

**定义**: 分布式输入控制端设备，向被控端设备发送指令，使用其外设输入的能力。

**职责**:
- 接收来自被控端的输入事件
- 将事件注入到虚拟输入驱动
- 管理虚拟输入设备的生命周期
- 响应多模输入模块的分布式输入请求

**关键组件**:
- `DistributedInputSourceManager` (SA 4809) - Source 侧 SA 实现
- `EventReceiver` - 接收被控端发送的输入事件
- `EventInject` - 将事件注入到虚拟输入驱动
- `DInputDriverMgr` - 管理分布式输入驱动

### Sink（被控端）

**定义**: 分布式输入被控制端设备，接受主控端发送的指令并且完成对应操作，提供本地外设供主控端设备使用。

**职责**:
- 从本地输入驱动采集输入外设原始事件
- 将事件发送到主控端
- 响应主控端的业务调用
- 管理本地输入设备状态

**关键组件**:
- `DistributedInputSinkManager` (SA 4810) - Sink 侧 SA 实现
- `EventCollector` - 从输入驱动采集输入外设原始事件
- `EventSender` - 将事件采集模块采集到的原始事件发送到主控端
- `EventFilter` - 提供组合键过滤能力

### DistributedInputSDK（Inner SDK）

**定义**: 为多模输入模块调用分布式输入能力提供的内部接口。

**职责**:
- 提供统一的 C++ API 接口
- 管理 Source 和 Sink SA 的连接
- 实现跨设备输入的准备、启动、停止操作
- 提供事件过滤和状态查询接口

**关键接口**:
- `PrepareRemoteInput()` - 准备分布式输入
- `StartRemoteInput()` - 启动分布式输入
- `StopRemoteInput()` - 停止分布式输入
- `UnprepareRemoteInput()` - 取消准备
- `RegisterSimulationEventListener()` - 注册仿真事件监听器

### DistributedInputFwkImpl（框架南向扩展实现）

**定义**: 实现了分布式硬件管理框架定义的南向外设扩展接口，供分布式硬件管理框架调度分布式输入能力。

**职责**:
- 实现分布式硬件框架的组件接入接口
- 提供输入外设能力查询
- 响应分布式硬件管理框架的设备上线/下线事件

**关键组件**:
- `DistributedInputSourceHandler` - Source 侧组件接入接口
- `DistributedInputSinkHandler` - Sink 侧组件接入接口
- `DistributedInputHandler` - 设备能力查询接口

## 架构概览

### 整体架构

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        多模输入模块                                   │
│                    (Multimodal Input)                              │
└────────────────────────────┬─────────────────────────────────────────────┘
                         │
        ┌────────────────▼────────────────────────────────────────┐
        │          DistributedInputSDK (Inner SDK)            │
        │          - C++ API 接口                          │
        │          - SA 连接管理                             │
        └────────────┬───────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │             │             │
        ▼             ▼             ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Source SA      │ │    Sink SA        │ │ DH Fwk         │
│ (4809)         │ │    (4810)          │ │               │
│                │ │                   │ │ 设备能力查询    │
│ - Source业务  │ │ - Sink业务      │ │ Source接入      │
│   逻辑         │ │   逻辑            │ │ Sink接入        │
│ - 外设注入    │ │ - 事件采集        │ │               │
│ - 事件接收    │ │ - 事件发送        │ │               │
└────────┬─────────┘ └────────┬─────────┘ └────────┬─────────┘
         │                    │                    │
         │                    │                    │
    ┌────▼────────┐      ┌────▼────────┐      ┌────▼────────┐
    │ 虚拟驱动     │      │ 本地驱动    │      │ SoftBus     │
    │ Virtual      │      │ Local       │      │ (跨设备    │
    │ Input Driver │      │ Input Driver │      │  传输)      │
    └─────────────┘      └─────────────┘      └─────────────┘
```

### 角色关系

```
设备 A（Source/主控端）              设备 B（Sink/被控端）
         │                                    │
         │ 1. 使用设备B的外设                │
         │    - 虚拟设备在A中创建           │
         │    - 虚拟设备与B的物理设备对应     │
         │                                    │
         │ ←──────────────────────────────────── │
         │   2. 跨设备输入事件传输              │
         │   （通过 SoftBus）                 │
         │                                    │
         │ 3. 事件在A中生效                │
         ▼                                    ▼
    [用户操作设备A]                    [用户操作设备B的输入设备]
```

## 主要功能

### 1. 设备发现与准备

- **设备组网**: 设备通过 SoftBus 组网并认证
- **能力同步**: 分布式硬件管理框架同步输入外设信息
- **虚拟驱动注册**: 主控端为被控端设备创建虚拟输入驱动节点
- **自动发现**: 多模输入模块自动发现虚拟输入设备

### 2. 分布式输入控制

- **准备输入**: `PrepareRemoteInput()` - 准备跨设备输入
- **启动输入**: `StartRemoteInput()` - 启动键鼠外设跨设备输入
- **停止输入**: `StopRemoteInput()` - 停止跨设备输入
- **取消准备**: `UnprepareRemoteInput()` - 释放跨设备输入资源

### 3. 事件过滤

- **组合键白名单**: 根据系统预置的组合键白名单进行组合键过滤
- **本地生效限制**: 白名单上的组合键只在被控端设备生效（如锁屏键等）
- **触摸屏过滤**: 支持触摸屏事件过滤

### 4. 状态管理

- **设备状态**: THROUGH_IN（穿透中）/ THROUGH_OUT（本地生效）
- **按键状态**: 跟踪按键按下/释放状态
- **会话管理**: SoftBus 会话生命周期管理

## 技术栈

### 语言和框架

- **语言**: C++
- **系统**: OpenHarmony 标准系统
- **子系统**: distributedhardware（分布式硬件子系统）
- **版本**: 3.2

### 关键依赖

| 依赖组件 | 用途 |
|---------|------|
| distributed_hardware_fwk | 分布式硬件管理框架 |
| device_manager | 设备管理和 ACL |
| dsoftbus | 跨设备通信（软总线） |
| safwk / samgr | System Ability 框架 |
| ipc | 进程间通信 |
| libevdev | 输入设备驱动 |
| eventhandler | 事件处理 |
| hilog / hisysevent | 日志和事件 |
| access_token | 权限令牌 |
| graphic_surface / window_manager | 屏幕和窗口管理 |

## 运行环境

### 硬件要求

- 输入外设支持：键盘、鼠标、触摸板等
- 局域网连接：确保设备在同一个局域网中
- 系统版本：OpenHarmony 标准系统

### 网络要求

- SoftBus 支持：设备间需要通过 SoftBus 连接
- 帐号要求：部分操作需要同一帐号（check_same_account）
- 带宽：低延迟要求（实时输入传输）

## 相关跳转

- [项目定位](01_Project_Positioning.md) - 详细的功能边界和能力说明
- [目录结构](02_Directory_Structure.md) - 代码组织结构
- [架构设计](03_Architecture.md) - 详细的架构图和数据流
- [公共 API](04_Public_API.md) - Inner SDK API 详细说明
- [安全评审](08_Security_Review.md) - 权限和安全机制

---

*更新时间: 2026-02-06 15:08:55*
