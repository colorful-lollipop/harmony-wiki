# Wi-Fi Aware HAL 接口

## 概述

HAL（Hardware Abstraction Layer）接口定义位于 `hals/hal_wifiaware.h`，共 **11 个函数**。

这些是**抽象接口**，由具体芯片实现。当前 Hi3861 芯片的实现位于：
```
$ohos_board_adapter_dir/hals/communication/wifi_lite/wifiaware/
```

## HAL 函数清单

| 函数 | 签名 | 功能 |
|-----|------|-----|
| `HalWifiSdpInit` | `int HalWifiSdpInit(const char* ifname)` | 初始化 SDP |
| `HalWifiSdpDeinit` | `int HalWifiSdpDeinit(void)` | 去初始化 SDP |
| `HalWifiSdpStartService` | `int HalWifiSdpStartService(const char* svcName, unsigned char localHandle, HalRecvCallback recvCB, unsigned char role)` | 启动服务 |
| `HalWifiSdpStopService` | `int HalWifiSdpStopService(unsigned char localHandle, unsigned char role)` | 停止服务 |
| `HalWifiSdpSend` | `int HalWifiSdpSend(unsigned char* macAddr, unsigned char peerHandle, unsigned char localHandle, unsigned char* msg, int len)` | 发送数据 |
| `HalCipherHashSha256` | `unsigned int HalCipherHashSha256(const char* input, unsigned int inputLen, unsigned char* hash, unsigned hashLen)` | SHA256 哈希 |
| `HalWifiSdpAdjustTxPower` | `int HalWifiSdpAdjustTxPower(const char* ifname, signed char power)` | 调整发射功率 |
| `HalWifiSdpRestoreTxPower` | `int HalWifiSdpRestoreTxPower(const char* ifname)` | 恢复默认功率 |
| `HalWifiSdpBeaconSwitch` | `int HalWifiSdpBeaconSwitch(const char* ifname, unsigned char enable)` | Beacon 开关 |
| `HalWifiSdpSetRetryTimes` | `int HalWifiSdpSetRetryTimes(unsigned int retries)` | 设置重试次数 |
| `HalWifiSdpGetSyncMode` | `int HalWifiSdpGetSyncMode(void)` | 获取同步模式 |

## 服务角色枚举

```c
enum {
    HAL_WIFI_SDP_PUBLISH   = 0x01,  // 发布服务
    HAL_WIFI_SDP_SUBSCRIBE = 0x02,  // 订阅服务
    HAL_WIFI_SDP_BUTT               // 上界标记
};
```

## 回调类型

### HalRecvCallback

```c
typedef int (*HalRecvCallback)(unsigned char* macAddr, 
                                unsigned char peerHandle, 
                                unsigned char localHandle,
                                unsigned char* msg, 
                                unsigned char len);
```

**说明**: HAL 层接收数据回调，与框架层 `RecvCallback` 类型兼容。

## HAL 详细说明

### HalWifiSdpInit

```c
int HalWifiSdpInit(const char* ifname);
```

**参数**: `ifname` - Wi-Fi 热点接口名称

**功能**: 初始化 Wi-Fi SDP（Service Discovery Protocol）硬件驱动

---

### HalWifiSdpDeinit

```c
int HalWifiSdpDeinit(void);
```

**功能**: 去初始化 Wi-Fi SDP 硬件驱动

---

### HalWifiSdpStartService

```c
int HalWifiSdpStartService(const char* svcName, unsigned char localHandle, 
                           HalRecvCallback recvCB, unsigned char role);
```

**参数**:
- `svcName`: 服务名（通常是 SHA256 哈希后的 32 字节）
- `localHandle`: 本地实例 ID
- `recvCB`: 接收数据回调
- `role`: 服务角色 (`PUBLISH` 或 `SUBSCRIBE`)

**功能**: 启动 NAN 服务

---

### HalWifiSdpStopService

```c
int HalWifiSdpStopService(unsigned char localHandle, unsigned char role);
```

**参数**:
- `localHandle`: 本地实例 ID
- `role`: 服务角色

**功能**: 停止 NAN 服务

---

### HalWifiSdpSend

```c
int HalWifiSdpSend(unsigned char* macAddr, unsigned char peerHandle, 
                   unsigned char localHandle, unsigned char* msg, int len);
```

**参数**:
- `macAddr`: 目标 MAC 地址（6 字节）
- `peerHandle`: 目标实例 ID
- `localHandle`: 本地实例 ID
- `msg`: 待发送数据
- `len`: 数据长度

**功能**: 通过 Wi-Fi Aware 发送数据

---

### HalCipherHashSha256

```c
unsigned int HalCipherHashSha256(const char* input, unsigned int inputLen, 
                                 unsigned char* hash, unsigned hashLen);
```

**参数**:
- `input`: 输入数据
- `inputLen`: 输入数据长度
- `hash`: 输出哈希缓冲区（32字节）
- `hashLen`: 哈希缓冲区长度

**返回**: 0 表示成功，非 0 表示失败

**功能**: 计算输入数据的 SHA256 哈希值

---

### HalWifiSdpAdjustTxPower

```c
int HalWifiSdpAdjustTxPower(const char* ifname, signed char power);
```

**参数**:
- `ifname`: 接口名称
- `power`: 发射功率（dB）

**功能**: 调整 Wi-Fi 发射功率（低功耗模式）

---

### HalWifiSdpRestoreTxPower

```c
int HalWifiSdpRestoreTxPower(const char* ifname);
```

**参数**: `ifname` - 接口名称

**功能**: 恢复默认发射功率

---

### HalWifiSdpBeaconSwitch

```c
int HalWifiSdpBeaconSwitch(const char* ifname, unsigned char enable);
```

**参数**:
- `ifname`: 接口名称
- `enable`: 启用/禁用标志

**功能**: 控制 Beacon 可见性

---

### HalWifiSdpSetRetryTimes

```c
int HalWifiSdpSetRetryTimes(unsigned int retries);
```

**参数**: `retries` - 重试次数

**功能**: 设置数据包重传次数

---

### HalWifiSdpGetSyncMode

```c
int HalWifiSdpGetSyncMode(void);
```

**返回**: NAN 同步模式枚举值

**功能**: 获取支持的同步模式

## 框架层调用映射

| Public API | HAL 函数 | 路径 |
|------------|----------|------|
| `InitNAN()` | `HalWifiSdpInit()` | `frameworks/source/wifiaware.c:46` |
| `DeinitNAN()` | `HalWifiSdpDeinit()` | `frameworks/source/wifiaware.c:99` |
| `SubscribeService()` | `HalWifiSdpStartService()` | `frameworks/source/wifiaware.c:71` |
| `SendData()` | `HalWifiSdpSend()` | `frameworks/source/wifiaware.c:81` |
| `StopSubscribe()` | `HalWifiSdpStopService()` | `frameworks/source/wifiaware.c:90` |
| `SetLowPower()` | `HalWifiSdpAdjustTxPower()` / `RestoreTxPower()` | `frameworks/source/wifiaware.c:118-124` |
| `NanBeaconSwitch()` | `HalWifiSdpBeaconSwitch()` | `frameworks/source/wifiaware.c:152` |
| `NanSetRetryTimes()` | `HalWifiSdpSetRetryTimes()` | `frameworks/source/wifiaware.c:161` |
| `NanGetSyncMode()` | `HalWifiSdpGetSyncMode()` | `frameworks/source/wifiaware.c:170` |

## 板级实现要求

要支持新芯片，需实现 `hal_wifiaware.h` 中声明的 **11 个函数**。

实现位置：
```
device/{芯片厂商}/hals/communication/wifi_lite/wifiaware/
```

## 后续阅读

- [架构文档](architecture.md) - 层次结构
- [API 文档](api.md) - C API 参考
- [构建文档](build.md) - 编译配置
