# Wi-Fi Aware C API 参考

## 重要说明

**本模块不包含 N-API（JavaScript）绑定**，是纯 C 原生库。

如需 JavaScript API，请查阅 OpenHarmony 其他仓库（可能位于 `communication_wifi` 或 JS API bundle 中）。

本文档描述 **C API**，位于 `interfaces/kits/wifiaware.h`。

## API 清单

| 函数 | 签名 | 版本 | 描述 |
|-----|------|-----|-----|
| `InitNAN` | `int InitNAN(void)` | 1.0 | 初始化 Wi-Fi Aware 模块 |
| `DeinitNAN` | `int DeinitNAN(void)` | 1.0 | 去初始化模块 |
| `SubscribeService` | `int SubscribeService(const char* svcName, unsigned char localHandle, RecvCallback recvCB)` | 1.0 | 启动订阅服务 |
| `SendData` | `int SendData(unsigned char* macAddr, unsigned char peerHandle, unsigned char localHandle, unsigned char* msg, int len)` | 1.0 | 发送数据到对端 |
| `StopSubscribe` | `int StopSubscribe(unsigned char localHandle)` | 1.0 | 停止订阅服务 |
| `SetLowPower` | `int SetLowPower(int powerSwitch)` | 1.0 | 低功耗模式开关 |
| `SetPower` | `int SetPower(signed char value)` | 2.2 | 设置发射功率 |
| `NanBeaconSwitch` | `int NanBeaconSwitch(unsigned char enable)` | 2.2 | Beacon 开关 |
| `NanSetRetryTimes` | `int NanSetRetryTimes(unsigned int retries)` | 2.2 | 设置重试次数 |
| `NanGetSyncMode` | `int NanGetSyncMode(void)` | 2.2 | 获取同步模式 |

## 错误码

| 常量 | 值 | 说明 |
|-----|---|-----|
| `WIFIAWARE_SUCCESS` | 0 | 操作成功 |
| `WIFIAWARE_FAIL` | -1 | 操作失败 |

## 常量定义

| 常量 | 值 | 说明 |
|-----|---|-----|
| `WIFIAWARE_LOW_POWER_SWITCH_ON` | 1 | 低功耗开启 |
| `WIFIAWARE_LOW_POWER_SWITCH_OFF` | 0 | 低功耗关闭 |
| `WIFIAWARE_DEFAULT_CHANNEL` | 6 | 默认信道 |

## 枚举

### NanSyncMode

```c
enum NanSyncMode {
    NAN_SYNC_MODE_PRIVATE,   // 华为私有 NAN 同步
    NAN_SYNC_MODE_STANDARD,  // 标准 NAN 同步
    NAN_SYNC_MODE_BOTH,      // 两者都支持
    NAN_SYNC_MODE_BUTT       // 枚举上界（不计）
};
```

## 回调类型

### RecvCallback

```c
typedef int (*RecvCallback)(unsigned char* macAddr, 
                            unsigned char peerHandle, 
                            unsigned char localHandle,
                            unsigned char* msg, 
                            unsigned char len);
```

**说明**: 当订阅服务收到消息时触发。此回调**仅处理时间敏感的事务**，耗时操作需创建独立任务处理。

## API 详细说明

### InitNAN

```c
int InitNAN(void);
```

**功能**: 初始化 Wi-Fi Aware 模块

**前置条件**:
- 设备热点 AP 接口已启用
- 使用信道 6（`WIFIAWARE_DEFAULT_CHANNEL`）

**返回**:
- `WIFIAWARE_SUCCESS`: 初始化成功
- `WIFIAWARE_FAIL`: 初始化失败（热点未启用或信道不正确）

**示例**:
```c
int ret = InitNAN();
if (ret != WIFIAWARE_SUCCESS) {
    // 处理错误
}
```

---

### DeinitNAN

```c
int DeinitNAN(void);
```

**功能**: 去初始化 Wi-Fi Aware 模块

**说明**: 无论模块是否已初始化，调用均生效

**返回**:
- `WIFIAWARE_SUCCESS`: 去初始化成功
- `WIFIAWARE_FAIL`: 去初始化失败

---

### SubscribeService

```c
int SubscribeService(const char* svcName, unsigned char localHandle, RecvCallback recvCB);
```

**功能**: 启动 Wi-Fi Aware 订阅服务

**参数**:
- `svcName`: 服务名称字符串，非 NULL
- `localHandle`: 本地实例 ID，范围 1-255
- `recvCB`: 收到消息时的回调函数

**前置条件**: 必须先调用 `InitNAN()`

**返回**:
- `WIFIAWARE_SUCCESS`: 订阅服务启动成功
- `WIFIAWARE_FAIL`: 订阅服务启动失败

**处理流程**:
1. 验证 `svcName` 非空且长度 > 0
2. 对服务名进行 SHA256 哈希
3. 调用 HAL 启动订阅服务

**代码证据**: `frameworks/source/wifiaware.c:53-76`

---

### SendData

```c
int SendData(unsigned char* macAddr, unsigned char peerHandle, 
             unsigned char localHandle, unsigned char* msg, int len);
```

**功能**: 发送用户数据到对端设备

**参数**:
- `macAddr`: 对端设备 MAC 地址（6 字节）
- `peerHandle`: 对端实例 ID（1-255）
- `localHandle`: 本地实例 ID（SubscribeService 时指定）
- `msg`: 用户数据，最大 255 字节
- `len`: 用户数据长度

**说明**: 
- **异步函数**。返回值仅表示数据是否已提交到硬件驱动
- 调用方需确保 MAC 地址和实例 ID 的有效性

**返回**:
- `WIFIAWARE_SUCCESS`: 提交成功
- `WIFIAWARE_FAIL`: 提交失败

**代码证据**: `interfaces/kits/wifiaware.h:161-179`

---

### StopSubscribe

```c
int StopSubscribe(unsigned char localHandle);
```

**功能**: 停止 Wi-Fi Aware 订阅服务

**参数**:
- `localHandle`: SubscribeService 时指定的本地实例 ID

**前置条件**: 必须先调用 `InitNAN()`

**说明**: 无论订阅服务是否已启动，调用均生效

**返回**:
- `WIFIAWARE_SUCCESS`: 停止成功
- `WIFIAWARE_FAIL`: 停止失败

---

### SetLowPower

```c
int SetLowPower(int powerSwitch);
```

**功能**: 设置是否启用短距离传输（低功耗模式）

**参数**:
- `powerSwitch`: `WIFIAWARE_LOW_POWER_SWITCH_ON` 或 `WIFIAWARE_LOW_POWER_SWITCH_OFF`

**说明**: 
- 启用后，消息可发送到本地设备 30cm 范围内的设备
- 需先确保热点已启用

**返回**:
- `WIFIAWARE_SUCCESS`: 设置成功
- `WIFIAWARE_FAIL`: 设置失败

---

### SetPower

```c
int SetPower(signed char value);
```

**功能**: 设置低功耗模式下的发射功率

**参数**:
- `value`: 发射功率（dB），推荐范围 -70 到 -42，默认 -61

**前置条件**: 先调用 `SetLowPower(WIFIAWARE_LOW_POWER_SWITCH_ON)`

**返回**:
- `WIFIAWARE_SUCCESS`: 设置成功
- `WIFIAWARE_FAIL`: 设置失败

**代码证据**: `frameworks/source/wifiaware.c:134-138`

---

### NanBeaconSwitch

```c
int NanBeaconSwitch(unsigned char enable);
```

**功能**: 设置 AP 是否可被发现

**参数**:
- `enable`: `true` 表示可被发现，`false` 表示不可见

**默认值**: 默认可见

**前置条件**: 必须先启用热点

**返回**:
- `WIFIAWARE_SUCCESS`: 设置成功
- `WIFIAWARE_FAIL`: 设置失败

---

### NanSetRetryTimes

```c
int NanSetRetryTimes(unsigned int retries);
```

**功能**: 设置低功耗模式下 NAN 数据包重传次数

**参数**:
- `retries`: 重传次数，推荐范围 0-200，默认 200

**说明**: 当 `SendData()` 发送失败时，重传次数达到后返回失败

**前置条件**: 必须先启用热点

**返回**:
- `WIFIAWARE_SUCCESS`: 设置成功
- `WIFIAWARE_FAIL`: 设置失败

---

### NanGetSyncMode

```c
int NanGetSyncMode(void);
```

**功能**: 获取 Wi-Fi Aware 模块支持的 NAN 同步模式

**返回**: `NanSyncMode` 枚举值之一:
- `NAN_SYNC_MODE_PRIVATE`: 华为私有 NAN 同步
- `NAN_SYNC_MODE_STANDARD`: 标准 NAN 同步
- `NAN_SYNC_MODE_BOTH`: 两者都支持

**代码证据**: `interfaces/kits/wifiaware.h:262-269`

## 使用示例

```c
#include "wifiaware.h"

// 接收回调
int OnDataReceived(unsigned char* mac, unsigned char peerHandle, 
                   unsigned char localHandle, unsigned char* msg, unsigned char len)
{
    // 处理接收到的数据
    // 注意：仅处理时间敏感操作，耗时任务创建独立线程
    return WIFIAWARE_SUCCESS;
}

void Example(void)
{
    int ret;
    
    // 初始化
    ret = InitNAN();
    if (ret != WIFIAWARE_SUCCESS) {
        return;
    }
    
    // 订阅服务
    ret = SubscribeService("HelloService", 1, OnDataReceived);
    if (ret != WIFIAWARE_SUCCESS) {
        DeinitNAN();
        return;
    }
    
    // 发送数据 (示例 MAC 地址)
    unsigned char mac[6] = {0x11, 0x22, 0x33, 0x44, 0x55, 0x66};
    unsigned char msg[] = "Hello!";
    ret = SendData(mac, 1, 1, msg, strlen(msg));
    
    // 清理
    StopSubscribe(1);
    DeinitNAN();
}
```

## 后续阅读

- [架构文档](architecture.md) - 架构与数据流
- [HAL 接口](hal.md) - HAL 函数定义
- [构建文档](build.md) - 编译说明
