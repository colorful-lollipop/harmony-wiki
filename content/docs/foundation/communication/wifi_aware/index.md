# Wi-Fi Aware 项目概览

## 项目定位

Wi-Fi Aware（Neighbor Awareness Networking, NAN）是 OpenHarmony 的**近场通信模块**，使设备能够在**不连接 Wi-Fi 网络**的情况下快速发现和交互。

**核心能力**:
- 设备发现（Service Discovery）
- 近场数据传输（Data Transfer）
- 低功耗通信（Low Power Mode）

## 适用范围

| 维度 | 范围 |
|-----|------|
| **适用系统** | small, standard |
| **支持芯片** | Hi3861（当前） |
| **实现语言** | C |
| **模块规模** | 3 文件，约 500 行 |

> **注意**: 如需支持其他芯片，需在 `device` 目录实现 HAL 集成 API。

## 核心能力

### 1. 模块初始化与去初始化
```c
int InitNAN(void);     // 初始化，需热点启用 + 信道6
int DeinitNAN(void);   // 去初始化
```

### 2. 服务订阅与发现
```c
int SubscribeService(const char* svcName, unsigned char localHandle, 
                     RecvCallback recvCB);  // 启动订阅服务
int StopSubscribe(unsigned char localHandle); // 停止订阅
```

### 3. 数据传输
```c
int SendData(unsigned char* macAddr, unsigned char peerHandle, 
             unsigned char localHandle, 
             unsigned char* msg, int len); // 发送数据到对端
```

### 4. 功耗控制
```c
int SetLowPower(int powerSwitch); // 低功耗开关
int SetPower(signed char value);  // 设置发射功率
```

## 依赖关系

### 内部依赖
| 依赖模块 | 用途 |
|---------|------|
| `wifi_hotspot` | 热点状态查询 |
| `wifi_device_util` | WiFi 工具函数 |

### 外部依赖
- `communication_softbus_lite` - Intelligent Soft Bus 子系统
- `communication_ipc_lite` - IPC 通信（README 提及）
- `hal_wifiaware` - HAL 实现（构建时注入）

## 目录结构

```
wifi_aware/
├── frameworks/
│   └── source/wifiaware.c      # 框架实现（172行）
├── hals/
│   └── hal_wifiaware.h         # HAL 接口（63行）
├── interfaces/
│   └── kits/wifiaware.h       # 对外 API（278行）
├── BUILD.gn                    # 构建配置
├── bundle.json                 # 组件配置
└── wiki/                       # 文档目录
```

## 关键概念

| 概念 | 说明 |
|-----|------|
| **NAN** | Neighbor Awareness Networking，近场感知网络 |
| **Service Name** | 服务名称，SHA256 哈希后用于标识 |
| **localHandle** | 本地实例 ID（1-255），用于设备认证 |
| **peerHandle** | 对端实例 ID（1-255） |
| **MAC Address** | 6 字节对端设备 MAC 地址 |
| **Sync Mode** | NAN 同步模式（私有/标准/两者） |

## 后续阅读

- [架构文档](architecture.md) - 详细架构分析
- [API 文档](api.md) - 完整 API 参考
- [HAL 接口](hal.md) - 硬件抽象层定义
- [安全评审](security.md) - 安全风险分析
