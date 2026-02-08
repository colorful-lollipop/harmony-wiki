# 分布式硬件管理框架概览

**英文名**: Distributed Hardware Framework
**模块名**: `@ohos/distributed_hardware_fwk`
**版本**: 4.0
**源码路径**: `foundation/distributedhardware/distributed_hardware_fwk`

---

## 项目定位

分布式硬件管理框架是 OpenHarmony 分布式硬件子系统的**核心基础设施**，提供统一的硬件接入、查询、使能和版本管理等能力。

> **核心职责**: 为分布式硬件子系统（如分布式相机、分布式屏幕、分布式音频）提供统一的管理平面。

---

## 核心能力

### 1. 硬件接入管理 (AccessManager)
- 对接 DeviceManager 子系统
- 处理设备上下线事件响应
- 维护设备在线状态

**证据**: `README_zh.md:11-12`
```markdown
**硬件接入管理(AccessManager)**：硬件接入管理模块对接设备管理（DeviceManger）子系统，
用于处理设备的上下线事件响应。
```

### 2. 硬件资源管理 (ResourceManager)
- 对接分布式数据服务
- 存储信任体系内的设备硬件信息
- 同步本机和周边设备的硬件信息

**证据**: `README_zh.md:13-14`
```markdown
**硬件资源管理(ResourceManager)**：对接分布式数据服务，用于存储信任体系内，
本机和周边设备同步过来的设备硬件信息。
```

### 3. 分布式硬件部件管理 (ComponentManager)
- 对接各分布式硬件实例化的部件
- 动态加载和使能/去使能部件驱动
- 管理部件生命周期

### 4. 本地硬件信息管理 (LocalHardwareManager)
- 采集本地硬件信息
- 感知本地硬件插拔事件
- 将动态硬件纳入分布式管理

### 5. 部件加载管理 (ComponentLoader)
- 解析部件配置文件
- 按需加载部件驱动 so
- 获取驱动接口函数句柄

### 6. 版本管理 (VersionManager)
- 管理超级终端内各设备的平台版本号
- 管理分布式硬件部件的版本号
- 版本兼容性检查

---

## 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层                                   │
│  ┌─────────────────┐                                           │
│  │   DHardware_UI  │  (系统应用)                                │
│  └────────┬────────┘                                           │
│           │                                                    │
│  ┌────────▼────────┐    N-API    ┌────────────────────────┐   │
│  │ hardwaremanager │────────────▶│ DistributedHardwareService│  │
│  │   (JS Binding) │             │     (SA ID: 4801)       │   │
│  └────────┬────────┘             └───────────┬────────────┘   │
│           │                                  │                 │
│           │                            ┌─────▼─────┐          │
│           │                            │ IPC Stub  │          │
│           │                            │ & Proxy   │          │
│           │                            └─────┬─────┘          │
│  ┌────────▼────────┐                          │                │
│  │   Inner Kit     │◀─────────────────────────┘                │
│  │   (libdhfwk_sdk)│                                          │
│  └─────────────────┘                                          │
└─────────────────────────────────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
   ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
   │   设备管理   │ │  分布式数据  │ │  软总线通信  │
   │  DeviceMgr  │ │   KV Store  │ │  SoftBus    │
   └─────────────┘ └─────────────┘ └─────────────┘
```

**证据**: `README_zh.md:7-9`
```markdown
其系统架构图如下图所示：
![](figures/distributedhardwarefwk_arch.png)
```

---

## 运行环境

| 条件 | 要求 |
|------|------|
| **语言限制** | C++ |
| **组网环境** | 设备必须在同一局域网中 |
| **操作系统** | OpenHarmony |

**证据**: `README_zh.md:41-44`
```markdown
**语言限制**：C++语言。  
**组网环境**：必须确保设备在同一个局域网中。  
**操作系统限制**：OpenHarmony操作系统。
```

---

## 关键概念

### 硬件类型 (DistributedHardwareType)

| 类型 | 说明 |
|------|------|
| `ALL` | 所有硬件类型 |
| `CAMERA` | 分布式相机 |
| `SCREEN` | 分布式屏幕 |
| `MODEM_MIC` | Modem 麦克风 |
| `MODEM_SPEAKER` | Modem 扬声器 |
| `MIC` | 本地麦克风 |
| `SPEAKER` | 本地扬声器 |

### 设备状态

| 状态 | 说明 |
|------|------|
| `ENABLED` | 硬件已使能 |
| `DISABLED` | 硬件已去使能 |
| `OFFLINE` | 设备离线 |
| `ONLINE` | 设备在线 |

---

## 目录速览

| 目录 | 职责 |
|------|------|
| `services/` | 核心 SA 服务实现 |
| `interfaces/kits/napi/` | N-API JavaScript 绑定 |
| `interfaces/inner_kits/` | Inner Kit SDK |
| `av_transport/` | AV 音视频传输 |
| `application/` | DHardware_UI 系统应用 |
| `common/` | 公共接口 |
| `utils/` | 工具类 |
| `sa_profile/` | SA 配置文件 |

---

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [distributedhardware_device_manager](https://gitee.com/openharmony/distributedhardware_device_manager) | 设备管理 |
| [distributedhardware_distributed_camera](https://gitee.com/openharmony/distributedhardware_distributed_camera) | 分布式相机 |
| [distributedhardware_distributed_screen](https://gitee.com/openharmony/distributedhardware_distributed_screen) | 分布式屏幕 |

---

## 下一步

- 了解模块结构 → [01_Directory_Structure.md](01_Directory_Structure.md)
- 理解系统架构 → [02_Architecture.md](02_Architecture.md)
- 查看 N-API 接口 → [03_NAPI_Reference.md](03_NAPI_Reference.md)
