# 01 - 项目概览

## 目的

本文档描述 OpenHarmony Cellular Call（蜂窝通话）模块的**项目定位、核心能力、系统依赖与特性标志**，帮助开发者快速理解该模块在系统中的角色与价值。

## 适用范围

- **读者对象**：OpenHarmony telephony 子系统开发者、架构师、安全审查人员
- **前置知识**：了解 Android/HarmonyOS 通话架构、IMS/VoLTE 基础概念
- **使用场景**：
  - 新人 onboarding
  - 模块功能调研
  - 系统集成分析
  - 安全评估准备

---

## 1.1 项目定位

### 1.1.1 在系统中的位置

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenHarmony 系统                           │
├─────────────────────────────────────────────────────────────────┤
│  应用层                                                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Call Manager                          │   │
│  │              (通话管理唯一入口)                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Cellular Call (SA ID: 4006)                 │   │
│  │              本模块 - 蜂窝通话核心实现                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Core Service (SA ID: 4010)            │   │
│  │                    telephony 核心服务                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      Modem / RIL                         │   │
│  │                    基带通信接口                          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.1.2 模块职责

| 职责 | 说明 | 证据位置 |
|------|------|----------|
| **CS 通话控制** | 2G/3G 电路交换通话的拨号、接听、挂断等 | `services/control/include/cs_control.h` |
| **IMS 通话控制** | 4G/5G VoLTE/VoWIFI/VoNR 通话控制 | `services/control/include/ims_control.h` |
| **域选择与切换** | CS/IMS 智能选择与无缝切换 | `services/manager/src/cellular_call_service.cpp` |
| **补充业务** | 呼叫转移、呼叫等待、呼叫限制等 | `services/utils/include/cellular_call_supplement.h` |
| **卫星通话** | 卫星通信通话支持（条件编译） | `services/control/include/satellite_control.h` |
| **紧急呼叫** | 紧急拨号与优先级处理 | `services/utils/include/emergency_utils.h` |

---

## 1.2 核心能力

### 1.2.1 支持的通话类型

```
┌────────────────────────────────────────────────────────────────┐
│                    Cellular Call 支持的通话类型                   │
├────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌─────────────────┐    ┌─────────────────────────────────┐      │
│  │   CS 通话       │    │           IMS 通话               │      │
│  │   (传统)        │    │           (现代)                 │      │
│  ├─────────────────┤    ├─────────────────────────────────┤      │
│  │ • 2G 语音通话   │    │ • VoLTE (4G 语音)               │      │
│  │ • 3G 语音通话   │    │ • VoWIFI (WIFI 通话)            │      │
│  │ • 紧急呼叫      │    │ • VoNR (5G 语音)                │      │
│  │                 │    │ • 视频通话 (Video Call)         │      │
│  │                 │    │ • 会议通话 (Conference)          │      │
│  │                 │    │ • RTT 实时文本 (条件编译)        │      │
│  └─────────────────┘    └─────────────────────────────────┘      │
│                                                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                      卫星通话 (条件编译)                      │ │
│  │              CELLULAR_CALL_SATELLITE 编译开关                │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                    │
└────────────────────────────────────────────────────────────────┘
```

### 1.2.2 补充业务支持

| 业务类型 | 功能 | 证据位置 |
|----------|------|----------|
| **呼叫转移** | 无条件/遇忙/无应答/不可及转移 | `CallTransferInfo` |
| **呼叫等待** | 呼叫等待查询与设置 | `SetCallWaiting()` |
| **呼叫限制** | 呼入/呼出限制（含密码） | `CallRestrictionInfo` |
| **呼叫保持** | 呼叫保持与恢复 | `HoldCall()` / `UnHoldCall()` |
| **呼叫切换** | 保持与激活切换 | `SwitchCall()` |
| **会议通话** | 创建/邀请/踢出会议 | `CombineConference()` 等 |
| **DTMF** | 双音多频信号发送 | `StartDtmf()` / `SendDtmf()` |
| **IMS 配置** | IMS 功能配置与查询 | `SetImsConfig()` / `GetImsConfig()` |

### 1.2.3 视频通话能力

| 功能 | 说明 | 证据位置 |
|------|------|----------|
| **Camera 控制** | 前后摄像头切换 | `ControlCamera()` |
| **窗口管理** | 预览/显示窗口设置 | `SetPreviewWindow()` |
| **缩放控制** | 视频缩放 | `SetCameraZoom()` |
| **画面旋转** | 设备方向设置 | `SetDeviceDirection()` |
| **暂停画面** | 暂停图片设置 | `SetPausePicture()` |
| **媒体模式切换** | 语音↔视频切换 | `SendUpdateCallMediaModeRequest()` |

---

## 1.3 系统依赖

### 1.3.1 外部依赖

| 依赖组件 | 用途 | 类型 |
|----------|------|------|
| **ability_base** | 基础能力框架 | 必选 |
| **ability_runtime** | 运行时能力 | 必选 |
| **access_token** | 访问令牌管理 | 必选 |
| **call_manager** | 通话管理器 | 必选 |
| **core_service** | Telephony 核心服务 (SA 4010) | 必选 |
| **ipc** | 进程间通信 | 必选 |
| **safwk** | System Ability 框架 | 必选 |
| **samgr** | 服务管理框架 | 必选 |
| **hilog** | 日志框架 | 必选 |
| **ffrt** | 高性能任务调度 | 必选 |
| **eventhandler** | 事件处理 | 必选 |
| **libphonenumber** | 电话号码处理 | 必选 |
| **json** (nlohmann) | JSON 解析 | 必选 |
| **cJSON** | JSON 处理 | 必选 |
| **common_event_service** | 公共事件服务 | 必选 |
| **data_share** | 数据共享 | 必选 |
| **graphic_surface** | 图形表面（视频通话） | 可选 |
| **hisysevent** | HiSysEvent 埋点 | 可选 |
| **hitrace** | 链路追踪 | 可选 |
| **init** | 初始化工具 | 可选 |
| **resource_management** | 资源管理 | 可选 |
| **security_guard** | 安全守护（条件） | 可选 |

**证据位置**: `bundle.json:36-63`

### 1.3.2 硬件要求

| 硬件 | 要求 | 说明 |
|------|------|------|
| **Modem** | 支持独立蜂窝通信 | 基带芯片 |
| **SIM 卡** | 支持蜂窝通信 | 至少一张 |
| **音频输出** | 扬声器或耳机 | 听筒 |
| **麦克风** | 音频输入设备 | 语音采集 |

**证据位置**: `README.md:45-46`

### 1.3.3 软件约束

- **编程语言**: C++
- **系统类型**: OpenHarmony Standard
- **必须配合**: telephony core service + Call Manager

---

## 1.4 特性标志

### 1.4.1 编译开关

| 标志 | 默认值 | 功能 | 证据位置 |
|------|--------|------|----------|
| `cellular_call_dynamic_start` | false | SA 动态启动 | `cellularcall.gni:15` |
| `cellular_call_tel_power_mode` | false | 省电模式 | `cellularcall.gni:16` |
| `cellular_call_support_UT` | true | 单元测试支持 | `cellularcall.gni:17` |
| `cellular_call_satellite` | false | 卫星通话功能 | `cellularcall.gni:18` |
| `cellular_call_support_rtt` | false | RTT 实时文本 | `cellularcall.gni:19` |

### 1.4.2 功能宏定义

| 宏 | 条件 | 功能 | 证据位置 |
|----|------|------|----------|
| `BASE_POWER_IMPROVEMENT_FEATURE` | `cellular_call_tel_power_mode` | 省电改进 | `cellularcall.gni:25` |
| `CELLULAR_CALL_SATELLITE` | `cellular_call_satellite` | 卫星通话 | `cellularcall.gni:29` |
| `SUPPORT_RTT_CALL` | `cellular_call_support_rtt` | RTT 支持 | `cellularcall.gni:33` |
| `OHOS_BUILD_ENABLE_TELEPHONY_EXT` | telephony_enhanced | 电话扩展 | `BUILD.gn:135` |
| `SECURITY_GUARDE_ENABLE` | security_guard | 安全守护 | `BUILD.gn:129` |
| `CALL_MANAGER_AUTO_START_OPTIMIZE` | device_name=="rk3568" | CallManager 启动优化 | `BUILD.gn:123` |

---

## 1.5 SysCap 与权限

### 1.5.1 System Capability

```
SystemCapability.Telephony.CellularCall
```

**证据位置**: `bundle.json:22`

### 1.5.2 权限要求

| 权限名 | 说明 | 用途 | 证据位置 |
|--------|------|------|----------|
| `CONNECT_CELLULAR_CALL_SERVICE` | 连接蜂窝通话服务 | IPC 调用权限检查 | `services/manager/src/cellular_call_stub.cpp:47` |

### 1.5.3 权限检查点

```
位置: services/manager/src/cellular_call_stub.cpp:45-50

逻辑:
  1. 获取调用方 UID
  2. 检查是否为 FOUNDATION_UID (5523)
  3. 非 FOUNDATION 调用必须检查 CONNECT_CELLULAR_CALL_SERVICE 权限
  4. 检查失败返回 TELEPHONY_ERR_PERMISSION_ERR
```

---

## 1.6 版本信息

| 属性 | 值 |
|------|-----|
| 模块版本 | 4.0 |
| 组件名 | `@ohos/cellular_call` |
| 子系统 | telephony |
| ROM 占用 | ~1MB |
| RAM 占用 | ~650KB |
| SA ID | 4006 |

---

## 相关跳转

| 目标 | 链接 |
|------|------|
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| 接口规范 | [04_Interfaces.md](./04_Interfaces.md) |
| 构建配置 | [06_GN_Build.md](./06_GN_Build.md) |

---

*最后更新：2026-02-06*
