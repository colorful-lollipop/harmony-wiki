# 项目概览

**目的**: 概述 `telephony_call_manager` 模块的定位、核心能力和运行环境

---

## 模块定位

### 核心职责

`call_manager` 是 OpenHarmony Telephony 子系统的核心模块，主要负责：

| 职责 | 说明 |
|------|------|
| **通话管理** | 管理 CS（电路交换）、IMS（IP多媒体子系统）、OTT（Over-The-Top）三类通话 |
| **资源管理** | 申请/释放通话所需的音视频资源 |
| **冲突解决** | 处理多通道通话的资源冲突和状态管理 |
| **UI 交互** | 与通话 UI 层（CallServiceAbility）交互 |

### 模块边界

```
上游依赖:
- telephony_core_service - 电话核心服务
- cellular_call - 蜂窝通话模块
- audio_framework - 音频框架
- multimedia - 多媒体子系统

下游输出:
- @ohos.telephony.call - JS API 接口
- ICallManagerService - 内部 IPC 接口
```

---

## 六大核心组件

### 1. CallServiceAbility
- **职责**: UI 交互（拨打键盘、来电状态上报）
- **位置**: `services/call/`

### 2. CallManagerService
- **职责**: 服务启动、初始化、SA 注册
- **证据**: `services/call_manager_service/src/call_manager_service.cpp:81`
- **SA ID**: 4005

### 3. Call Manager（核心）
- **职责**: 下行通话操作（拨号、接听、挂断）、上行状态处理、冲突解决
- **位置**: `services/call/`

### 4. Audio Manager
- **职责**: 音频资源申请/释放、音频状态管理
- **依赖**: `audio_framework`
- **位置**: `services/audio/`

### 5. Video Manager
- **职责**: 视频资源申请/释放、视频状态管理
- **位置**: `services/video/`

### 6. Bluetooth Manager
- **职责**: 蓝牙通话资源管理、蓝牙设备操作
- **依赖**: Bluetooth 框架
- **位置**: `services/bluetooth/`

---

## 运行环境

### 软件依赖

| 依赖项 | 用途 | 证据 |
|--------|------|------|
| Security subsystem | 权限校验、访问控制 | `call_manager_service.cpp:18-19` |
| Multimedia subsystem | 音视频资源 | `callmanager.gni:226-230` |
| Intelligent Soft Bus | 分布式通信 | `callmanager.gni:204-205` |
| telephony_core_service | 核心服务交互 | `bundle.json:49` |

### 硬件要求

- 扬声器或耳机
- 麦克风设备
- 支持语音通话的基带

---

## 关键概念

### 通话类型

| 类型 | 说明 | 缩写 |
|------|------|------|
| CS Call | 电路交换通话 | CS |
| IMS Call | IP多媒体子系统通话 | IMS |
| OTT Call | 第三方互联网通话 | OTT |
| VoIP Call | IP语音通话 | VoIP |

### 通话状态

| 状态 | 说明 |
|------|------|
| IDLE | 空闲 |
| RINGING | 响铃中 |
| OFFHOOK | 摘机/通话中 |
| HOLDING | 保持中 |

---

## 目录结构

```
telephony_call_manager/
├── figures/                 # README 图表
├── frameworks/              # 框架层
│   ├── js/                  # JS 框架（N-API）
│   └── native/             # Native 框架（IPC）
├── interfaces/              # 接口层
│   ├── innerkits/           # 内部 API
│   └── kits/                # 外部 API（JS API）
├── sa_profile/              # SA 配置
├── services/                # 服务层
│   ├── audio/               # 音频管理
│   ├── bluetooth/           # 蓝牙通话
│   ├── call/                # 通话核心
│   ├── call_manager_service/# SA 服务
│   ├── call_report/         # 状态上报
│   ├── call_setting/        # 通话设置
│   ├── telephony_interaction/# 核心服务交互
│   └── video/               # 视频管理
└── utils/                   # 工具类
```

---

## 相关文档

- [架构说明](01_Architecture.md)
- [API 参考](02_API_Reference.md)
- [构建系统](03_Build_System.md)
