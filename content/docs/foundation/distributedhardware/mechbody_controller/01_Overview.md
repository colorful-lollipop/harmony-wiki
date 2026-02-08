# 项目概述

## 项目定位

Mechbody Controller（机械设备控制器）是 OpenHarmony 分布式硬件子系统的核心服务，提供机械设备（Gimbal 云台等）的发现、控制和管理能力。

**定位**：设备厂商与上层应用之间的桥梁，将复杂的蓝牙通信和协议交互封装为标准化的 N-API/ANI 接口。

## 核心能力

| 能力 | 描述 | API 入口 |
|------|------|----------|
| **设备发现** | 发现并连接兼容的机械设备 | `getAttachedMechDevices` |
| **状态感知** | 监听设备连接/断开、追踪状态变化 | `on`, `off` |
| **运动控制** | 角度旋转、速度旋转、停止移动 | `rotate`, `rotateBySpeed` |
| **相机追踪** | 基于人脸检测的智能追踪 | `setCameraTrackingEnabled` |
| **目标搜索** | 搜索特定类型的目标 | `searchTarget` |
| **状态查询** | 查询角度、限制、轴状态 | `getCurrentAngles`, `getRotationLimits` |

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| **系统版本** | OpenHarmony Standard |
| **系统能力** | `SystemCapability.Mechanic.Core` |
| **设备类型** | 标准设备（Standard Device） |

### 硬件依赖

| 依赖 | 用途 | 必需性 |
|------|------|--------|
| **蓝牙 (BLE)** | 与机械设备通信 | 必须 |
| **相机** | 人脸检测与追踪 | 可选（追踪功能必需） |

### 软件依赖

| 依赖服务 | SA ID | 用途 |
|----------|-------|------|
| **蓝牙服务** | 1130 | BLE 通信 |
| **系统能力管理** | - | SA 生命周期 |

## 关键概念

### 设备类型

| 类型 | 值 | 说明 |
|------|-----|------|
| `GIMBAL_DEVICE` | 0 | 云台设备（目前唯一支持） |

### 旋转轴限制

| 限制类型 | 值 | 说明 |
|----------|-----|------|
| `NOT_LIMITED` | 0 | 无限制 |
| `NEGATIVE_LIMITED` | 1 | 负向限制 |
| `POSITIVE_LIMITED` | 2 | 正向限制 |

### 操作类型

| 操作 | 值 | 说明 |
|------|-----|------|
| `CONNECT` | 0 | 连接操作 |
| `DISCONNECT` | 1 | 断开操作 |

### 追踪布局

| 布局 | 值 | 说明 |
|------|-----|------|
| `DEFAULT` | 0 | 默认布局 |
| `LEFT` | 1 | 左侧追踪 |
| `MIDDLE` | 2 | 中间追踪 |
| `RIGHT` | 3 | 右侧追踪 |

### 目标类型

| 类型 | 值 | 说明 |
|------|-----|------|
| `HUMAN_FACE` | 0 | 人脸（目前唯一支持） |

### 操作结果

| 结果 | 值 | 说明 |
|------|-----|------|
| `COMPLETED` | 0 | 操作完成 |
| `INTERRUPTED` | 1 | 操作中断 |
| `LIMITED` | 2 | 受限制 |
| `TIMEOUT` | 3 | 超时 |
| `SYSTEM_ERROR` | 100 | 系统错误 |

## 相关资源

| 资源 | 链接 |
|------|------|
| 源码仓库 | `//foundation/distributedhardware/mechbody_controller` |
| N-API 模块 | `distributedHardware.mechanicManager` |
| ANI 模块 | `@ohos.mechbodyController` (ETS) |
| SA ID | 8550 |

## 目录结构

```
mechbody_controller/
├── etc/init/              # 初始化配置
│   └── mechbody.cfg       # 服务配置
├── interface/             # 接口层
│   ├── ets/              # ANI (ETS) 接口
│   └── napi/             # N-API 接口
├── sa_profile/           # SA 配置
│   └── 8550.json         # SA 8550 配置
├── services/             # 服务实现
│   ├── include/          # 头文件
│   └── src/              # 源文件
└── test/                 # 测试代码（略）
```

## 下一步

- [架构说明](02_Architecture.md) → 深入理解系统架构
- [N-API 参考](03_NAPI_Reference.md) → 学习 API 使用
