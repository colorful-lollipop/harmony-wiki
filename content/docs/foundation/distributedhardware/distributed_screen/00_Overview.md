# 项目概览

## 目的与适用范围

本文档介绍OpenHarmony分布式屏幕组件的基本信息、项目定位和核心能力。

**适用范围**: 
- OpenHarmony开发者
- 分布式硬件子系统维护者
- 需要集成或调试分布式屏幕功能的工程师

---

## 项目定位

分布式屏幕是OpenHarmony分布式硬件子系统的核心组件之一，提供**屏幕虚拟化能力**，支持用户指定组网认证过的其他OpenHarmony设备的屏幕作为Display的显示区域。

### 核心能力

| 能力 | 说明 |
|------|------|
| 系统投屏 | 将本机屏幕内容投射到远程设备 |
| 屏幕镜像 | 实时镜像屏幕内容到对端 |
| 屏幕分割 | 支持多区域屏幕共享 |

### 运行环境

- **操作系统**: OpenHarmony Standard系统
- **组网环境**: 设备必须在同一局域网
- **语言**: C++
- **版本**: 3.1

---

## 关键概念

### 角色定义

```
┌─────────────────────────────────────────────────────────────┐
│                    分布式屏幕组网架构                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────────┐        局域网        ┌──────────────┐│
│   │   主控端         │ ◄──────────────────► │   被控端      ││
│   │   (Source)       │     软总线传输       │   (Sink)      ││
│   │                  │                      │               ││
│   │  - 捕获屏幕数据   │                      │  - 接收并解码  ││
│   │  - 编码传输      │                      │  - 显示到窗口  ││
│   │  - 控制流程      │                      │  - 反馈状态    ││
│   └──────────────────┘                      └──────────────┘│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

- **主控端 (Source)**: 控制端，通过调用分布式屏幕能力，使用被控端的屏幕用于显示主控端设备的屏幕内容
- **被控端 (Sink)**: 被控制端，通过分布式屏幕接收主控端的控制，在本地对接窗口用于接收和显示主控端设备的屏幕内容

### 架构组件

| 组件 | 职责 | 代码路径 |
|------|------|----------|
| ScreenRegionManager | 被控端显示区域管理 | `services/screenservice/sinkservice/screenregionmgr/` |
| DScreenManager | 主控端屏幕管理 | `services/screenservice/sourceservice/dscreenmgr/` |
| ScreenService | SA服务（主控/被控）| `services/screenservice/{sourceservice,sinkservice}/` |
| SoftbusAdapter | 软总线适配器 | `services/softbusadapter/` |
| ScreenTransport | 屏幕传输组件 | `services/screentransport/` |
| ScreenClient | 屏幕图像显示客户端 | `services/screenclient/` |

---

## System Ability 服务

分布式屏幕通过两个System Ability向系统提供服务：

### SA 4807 - Source端服务

**证据**: `sa_profile/4807.json:1-13`
```json
{
    "process": "dscreen",
    "systemability": [{
        "name": 4807,
        "libpath": "libdistributed_screen_source.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1
    }]
}
```

**功能**:
- 初始化Source端 (`InitSource`)
- 注册分布式硬件 (`RegisterDistributedHardware`)
- 管理屏幕连接 (`DScreenManager`)
- 编码并传输屏幕数据

### SA 4808 - Sink端服务

**证据**: `sa_profile/4808.json:1-13`
```json
{
    "process": "dscreen",
    "systemability": [{
        "name": 4808,
        "libpath": "libdistributed_screen_sink.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1
    }]
}
```

**功能**:
- 初始化Sink端 (`InitSink`)
- 订阅本地硬件 (`SubscribeLocalHardware`)
- 管理显示区域 (`ScreenRegionManager`)
- 接收并解码显示屏幕数据

---

## 工作流程

### 1. 设备开机启动

系统拉起分布式屏幕的SA服务，Source侧被初始化，相关模块被初始化。

### 2. 设备组网上线

设备上线后，分布式硬件管理框架同步到上线设备的屏幕硬件信息并使能，使能成功后在系统中会新增虚拟屏幕并通知到窗口子系统。

### 3. 屏幕数据流转

**证据**: 基于 `README_zh.md:79-89`

```
┌─────────────┐    ┌──────────────┐    ┌──────────────┐    ┌─────────────┐
│ 图形子系统   │───►│   编码器      │───►│  传输组件     │───►│   软总线     │
│  Surface    │    │  (Encoder)   │    │(SourceTrans) │    │  (SoftBus)   │
└─────────────┘    └──────────────┘    └──────────────┘    └──────┬──────┘
                                                                  │
                          网络传输                                 │
                                                                  ▼
┌─────────────┐    ┌──────────────┐    ┌──────────────┐    ┌─────────────┐
│   窗口显示   │◄───│   解码器      │◄───│  传输通道     │◄───│   软总线     │
│   Surface   │    │  (Decoder)   │    │ (DataChannel)│    │              │
└─────────────┘    └──────────────┘    └──────────────┘    └─────────────┘
```

详细流程：
1. 主控端图形子系统将需要发送的屏幕数据保存在编码器创建的输入Surface中
2. 主控端编码器将输入数据进行编码，并将编码结果返回传输组件screensourcetrans
3. 主控端传输组件将编码后的数据通过传输通道screendatachannel发送到softbusadapter
4. 软总线子系统将数据发送到被控端设备
5. 被控端设备软总线子系统收到屏幕数据后，通过softbusadapter返回给传输通道
6. 被控端传输通道将获取到的屏幕数据传递给解码器进行解码
7. 解码器将屏幕数据解码，并将解码后的数据保存到被控端代理显示窗口设置到解码器的Surface中

### 4. 设备下线

设备下线后，分布式硬件管理框架去使能下线设备的屏幕硬件，本地移除对应的虚拟屏幕并通知窗口子系统。

---

## 依赖关系

### 系统组件依赖

**证据**: `bundle.json:28-59`

核心依赖组件：
- `access_token` - 权限管理
- `device_manager` - 设备管理
- `dsoftbus` - 软总线通信
- `ipc` - IPC通信
- `samgr/safwk` - SA框架
- `av_codec` - 音视频编解码
- `media_foundation` - 媒体基础
- `window_manager` - 窗口管理
- `distributed_hardware_fwk` - 分布式硬件框架
- `graphic_2d/graphic_surface` - 图形系统

---

## 相关跳转

- [架构设计](01_Architecture.md) - 详细架构图和数据流
- [对外接口](03_Interfaces.md) - SDK接口定义
- [构建系统](05_Build_System.md) - 构建配置详解