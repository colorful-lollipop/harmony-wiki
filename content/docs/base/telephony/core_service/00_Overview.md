# 项目概览

## 目的与适用范围

本文档介绍 OpenHarmony `telephony_core_service` 模块的项目定位、核心能力和运行环境。

---

## 项目定位

`telephony_core_service` 是 OpenHarmony 电话子系统的核心服务模块，负责：

1. **SIM 卡管理** - SIM 卡初始化、文件读写、状态通知、PIN/PUK 管理
2. **网络搜索** - 网络注册、信号强度、运营商信息、小区信息
3. **RIL 通信** - 与 RIL Adapter 服务通信，转发 modem 指令
4. **IMS 状态** - IMS 网络注册状态上报

## 核心能力

| 能力域 | 功能描述 |
|--------|----------|
| SIM 管理 | 卡状态查询、ICCID/IMSI 读取、联系人存储、SMS 存储、STK |
| 网络搜索 | 网络注册、制式查询、信号强度、小区管理、时区更新 |
| 设备信息 | IMEI/MEID 获取、基带版本、设备唯一标识 |
| eSIM 支持 | Profile 下载、切换、删除（条件编译）|
| 卫星通信 | 卫星服务交互（条件编译）|

## 运行环境

### 硬件约束

- 设备必须配备独立的蜂窝通信 Modem
- 支持 SIM 卡槽（单卡/双卡）
- eSIM 功能需要硬件支持

### 软件依赖

```
telephony_core_service
├── drivers_interface_ril (RIL 驱动接口)
├── ril_adapter (RIL 适配服务)
├── state_registry (状态注册服务)
├── samgr (系统能力管理)
├── safwk (系统能力框架)
├── hilog (日志服务)
├── hisysevent (系统事件)
└── ... (详见 bundle.json)
```

### 权限要求

| Syscap | 说明 |
|--------|------|
| SystemCapability.Telephony.CoreService | 核心服务能力 |
| SystemCapability.Telephony.CoreService.Esim | eSIM 能力 |

## 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                     Application (JS)                        │
├─────────────────────────────────────────────────────────────┤
│  @ohos.telephony.sim  │  @ohos.telephony.radio              │
├─────────────────────────────────────────────────────────────┤
│  N-API (sim.z.so)     │  N-API (radio.z.so)                 │
├─────────────────────────────────────────────────────────────┤
│            CoreServiceClient (tel_core_service_api)         │
├─────────────────────────────────────────────────────────────┤
│  CoreService (SA:4010)  │  ImsCoreServiceClient             │
├─────────────────────────────────────────────────────────────┤
│  SimManager │ NetworkSearchManager │ TelRilManager          │
├─────────────────────────────────────────────────────────────┤
│                    RIL Adapter (HDF)                        │
├─────────────────────────────────────────────────────────────┤
│                      Modem Driver                           │
└─────────────────────────────────────────────────────────────┘
```

## 关键概念

### SlotId vs SimId

- **SlotId**: 物理卡槽索引，从 0 开始（0=卡槽1, 1=卡槽2）
- **SimId**: SIM 卡逻辑标识，用于区分不同的 SIM 卡

### 网络制式

| 制式 | 说明 |
|------|------|
| GSM | 2G |
| WCDMA | 3G |
| LTE | 4G |
| NR | 5G |

### 回调机制

- **同步回调**: 直接返回结果
- **异步回调**: 通过 `INetworkSearchCallback` / `IRawParcelCallback` 回调

## 相关链接

- [目录结构](./01_Directory_Structure.md)
- [架构设计](./02_Architecture.md)
- [README.md](../README.md)
