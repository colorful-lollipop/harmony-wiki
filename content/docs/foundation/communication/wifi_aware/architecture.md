# Wi-Fi Aware 架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                      Application Layer                      │
│                    (JavaScript/Native)                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Public C API Layer                        │
│           interfaces/kits/wifiaware.h (9 APIs)              │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Framework Adaptation Layer                 │
│            frameworks/source/wifiaware.c (172行)             │
│  - 服务名 SHA256 哈希                                        │
│  - 回调类型转换                                             │
│  - 热点状态验证                                             │
│  - 全局功率管理                                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   HAL Interface Layer                        │
│             hals/hal_wifiaware.h (11 函数)                  │
│  - 硬件抽象接口定义                                         │
│  - 回调类型声明                                             │
│  - 服务角色枚举                                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Board-Specific Implementation                   │
│    $ohos_board_adapter_dir/hals/communication/              │
│          wifi_lite/wifiaware/ (Hi3861 实现)                 │
└─────────────────────────────────────────────────────────────┘
```

## 组件职责

| 组件 | 文件 | 职责 |
|-----|------|-----|
| Public API | `interfaces/kits/wifiaware.h` | 对外 C 接口声明 |
| Framework | `frameworks/source/wifiaware.c` | 适配层、业务逻辑 |
| HAL | `hals/hal_wifiaware.h` | 硬件抽象接口 |
| Build | `BUILD.gn` | 构建配置 |

## 数据流

### 初始化流程
```
InitNAN()
  ↓
IsHotspotActive()  [检查热点是否启用]
  ↓
GetHotspotChannel()  [检查信道是否为6]
  ↓
GetHotspotInterfaceName()  [获取热点接口名]
  ↓
HalWifiSdpInit(ifname)  [HAL 初始化]
```

### 服务订阅流程
```
SubscribeService(svcName, localHandle, recvCB)
  ↓
strlen(svcName)  [验证服务名长度]
  ↓
HalCipherHashSha256(svcName)  [SHA256 哈希]
  ↓
HalWifiSdpStartService(shaSvcName, localHandle, recvCB, SUBSCRIBE)
  ↓
HAL 返回本地 handle 用于后续通信
```

### 数据发送流程
```
SendData(macAddr, peerHandle, localHandle, msg, len)
  ↓
HalWifiSdpSend(macAddr, peerHandle, localHandle, msg, len)
  ↓
异步返回发送结果
```

## 线程模型

**注意**: 本模块未显式声明线程模型。推测为：

- **初始化**: 主线程同步调用
- **数据接收**: 异步回调（`RecvCallback`）
- **数据发送**: 异步调用（"返回仅表示已提交到硬件驱动"）

> **证据**: `interfaces/kits/wifiaware.h:163-165` - "This is an asynchronous function. The return value only indicates whether the user data has been sent to the hardware driver."

## 状态管理

### 全局变量
| 变量 | 类型 | 用途 |
|-----|------|-----|
| `g_power` | `signed char` | 低功耗模式发射功率 |

**证据**: `frameworks/source/wifiaware.c:26`

```c
static signed char g_power = WIFIAWARE_TX_LOW_POWER;  // -61 dB
```

### 状态转换
```
DeinitNAN() ←─┬── 任意状态皆可调用
              │
InitNAN() ──┬─→ INITIALIZED ──┬─→ SubscribeService() ─→ SUBSCRIBED
           │                  │
           │                  └─→ SetLowPower() ────────────┐
           │                                              │
           ▼                                              ▼
UNINITIALIZED ←──── DeinitNAN() ←────────────── STOPPED
```

## 依赖方向

```
interfaces/kits/wifiaware.h
        │
        ▼
frameworks/source/wifiaware.c
        │
        ├── Depends on ──→ wifi_hotspot (IsHotspotActive, GetHotspotChannel)
        │                 (wifi_device_util - GetHotspotInterfaceName)
        │
        └── Depends on ──→ hals/hal_wifiaware.h
                            │
                            └── Build-time ──→ Board HAL Implementation
```

## 关键设计决策

### 1. 服务名 SHA256 哈希
- **决策**: 订阅服务前对服务名进行 SHA256 哈希
- **原因**: 统一标识符长度（32字节），提供一定混淆
- **代码**: `frameworks/source/wifiaware.c:67`

### 2. 角色枚举 (PUBLISH/SUBSCRIBE)
- **PUBLISH (0x01)**: 发布服务
- **SUBSCRIBE (0x02)**: 订阅服务
- **证据**: `hals/hal_wifiaware.h:25-29`

### 3. 全局功率管理
- **原因**: 低功耗模式需要在多次调用间保持功率设置
- **实现**: 静态变量 `g_power`
- **风险**: 线程不安全，多线程需外部同步

### 4. 无 IPC 层
- **原因**: 本模块是 lite/embedded 版本，运行在单一进程
- **影响**: 权限检查在更高层（SoftBus/IPC）处理

## 后续阅读

- [API 文档](api.md) - 完整函数签名
- [HAL 接口](hal.md) - HAL 函数定义
- [构建文档](build.md) - 编译产物
