# API 参考

## 概述

本文档描述 `wifi_lite` 提供的所有 C 语言接口，包括 Station 模式、Hotspot 模式及相关数据结构。

**重要说明**: 本仓库仅为接口定义层，函数实现位于 Wi-Fi 服务进程中。调用这些 API 需要链接实际的 Wi-Fi 服务实现库。

---

## Station 模式 API

Station 模式使设备作为 Wi-Fi 客户端连接到无线网络。

### 头文件

```c
#include "wifi_device.h"
```

**证据**: [wifi_device.h:37-44](interfaces/wifiservice/wifi_device.h#L37-L44)

### API 清单

| 返回类型 | 函数名 | 功能描述 | 行号 |
|----------|--------|----------|------|
| `WifiErrorCode` | `EnableWifi()` | 启用 Station 模式 | [57](interfaces/wifiservice/wifi_device.h#L57) |
| `WifiErrorCode` | `DisableWifi()` | 禁用 Station 模式 | [66](interfaces/wifiservice/wifi_device.h#L66) |
| `int` | `IsWifiActive()` | 检查 Station 是否启用 | [75](interfaces/wifiservice/wifi_device.h#L75) |
| `WifiErrorCode` | `Scan()` | 开始 Wi-Fi 扫描 | [84](interfaces/wifiservice/wifi_device.h#L84) |
| `WifiErrorCode` | `GetScanInfoList(WifiScanInfo *result, unsigned int *size)` | 获取扫描结果 | [98](interfaces/wifiservice/wifi_device.h#L98) |
| `WifiErrorCode` | `AddDeviceConfig(const WifiDeviceConfig *config, int *result)` | 添加网络配置 | [111](interfaces/wifiservice/wifi_device.h#L111) |
| `WifiErrorCode` | `GetDeviceConfigs(WifiDeviceConfig *result, unsigned int *size)` | 获取所有配置 | [125](interfaces/wifiservice/wifi_device.h#L125) |
| `WifiErrorCode` | `RemoveDevice(int networkId)` | 删除网络配置 | [135](interfaces/wifiservice/wifi_device.h#L135) |
| `WifiErrorCode` | `DisableDeviceConfig(int networkId)` | 禁用指定配置 | [146](interfaces/wifiservice/wifi_device.h#L146) |
| `WifiErrorCode` | `EnableDeviceConfig(int networkId)` | 启用指定配置 | [157](interfaces/wifiservice/wifi_device.h#L157) |
| `WifiErrorCode` | `ConnectTo(int networkId)` | 连接指定网络 | [169](interfaces/wifiservice/wifi_device.h#L169) |
| `WifiErrorCode` | `ConnectToDevice(const WifiDeviceConfig *config)` | 直接连接网络 | [179](interfaces/wifiservice/wifi_device.h#L179) |
| `WifiErrorCode` | `Disconnect()` | 断开当前连接 | [188](interfaces/wifiservice/wifi_device.h#L188) |
| `WifiErrorCode` | `GetLinkedInfo(WifiLinkedInfo *result)` | 获取连接信息 | [198](interfaces/wifiservice/wifi_device.h#L198) |
| `WifiErrorCode` | `GetDeviceMacAddress(unsigned char *result)` | 获取本机 MAC 地址 | [208](interfaces/wifiservice/wifi_device.h#L208) |
| `WifiErrorCode` | `AdvanceScan(WifiScanParams *params)` | 带参数扫描 | [221](interfaces/wifiservice/wifi_device.h#L221) |
| `WifiErrorCode` | `GetIpInfo(IpInfo *info)` | 获取 IP 信息 | [230](interfaces/wifiservice/wifi_device.h#L230) |
| `int` | `GetSignalLevel(int rssi, int band)` | 获取信号等级 | [244](interfaces/wifiservice/wifi_device.h#L244) |
| `WifiErrorCode` | `RegisterWifiEvent(WifiEvent *event)` | 注册事件回调 | [256](interfaces/wifiservice/wifi_device.h#L256) |
| `WifiErrorCode` | `UnRegisterWifiEvent(const WifiEvent *event)` | 注销事件回调 | [266](interfaces/wifiservice/wifi_device.h#L266) |

---

## Hotspot 模式 API

Hotspot 模式使设备作为 Wi-Fi 热点供其他设备连接。

### 头文件

```c
#include "wifi_hotspot.h"
```

**证据**: [wifi_hotspot.h:37-46](interfaces/wifiservice/wifi_hotspot.h#L37-L46)

### API 清单

| 返回类型 | 函数名 | 功能描述 | 行号 |
|----------|--------|----------|------|
| `WifiErrorCode` | `EnableHotspot()` | 启用 Hotspot 模式 | [63](interfaces/wifiservice/wifi_hotspot.h#L63) |
| `WifiErrorCode` | `DisableHotspot()` | 禁用 Hotspot 模式 | [72](interfaces/wifiservice/wifi_hotspot.h#L72) |
| `WifiErrorCode` | `SetHotspotConfig(const HotspotConfig *config)` | 设置热点配置 | [86](interfaces/wifiservice/wifi_hotspot.h#L86) |
| `WifiErrorCode` | `GetHotspotConfig(HotspotConfig *result)` | 获取热点配置 | [98](interfaces/wifiservice/wifi_hotspot.h#L98) |
| `int` | `IsHotspotActive()` | 检查热点是否启用 | [107](interfaces/wifiservice/wifi_hotspot.h#L107) |
| `WifiErrorCode` | `GetStationList(StationInfo *result, unsigned int *size)` | 获取已连接站点列表 | [121](interfaces/wifiservice/wifi_hotspot.h#L121) |
| `WifiErrorCode` | `DisassociateSta(unsigned char *mac, int macLen)` | 断开指定站点 | [132](interfaces/wifiservice/wifi_hotspot.h#L132) |
| `WifiErrorCode` | `AddTxPowerInfo(int power)` | 添加发射功率信息 | [145](interfaces/wifiservice/wifi_hotspot.h#L145) |

---

## 数据结构

### WifiDeviceConfig（Station 连接配置）

**头文件**: [wifi_device_config.h](interfaces/wifiservice/wifi_device_config.h)

用于配置 Wi-Fi 连接参数：

```c
typedef struct {
    char ssid[WIFI_MAX_SSID_LEN];       // 网络名称
    char preSharedKey[WIFI_MAX_KEY_LEN]; // 预共享密钥
    char bssid[WIFI_MAC_LEN];           // BSSID（可选）
    int securityType;                   // 安全类型
    int netId;                          // 网络ID
    int isWepPsk;                       // WEP 标识
    int isHiddenSSID;                   // 隐藏SSID
} WifiDeviceConfig;
```

**证据**: [wifi_device_config.h](interfaces/wifiservice/wifi_device_config.h)（完整结构体定义）

### HotspotConfig（热点配置）

**头文件**: [wifi_hotspot_config.h](interfaces/wifiservice/wifi_hotspot_config.h)

用于配置热点参数：

```c
typedef struct {
    char ssid[WIFI_MAX_SSID_LEN];       // 热点名称
    char preSharedKey[WIFI_MAX_KEY_LEN]; // 连接密钥
    int securityType;                   // 安全类型
    int band;                           // 频段
    int channel;                        // 通道
    int maxConn;                        // 最大连接数
} HotspotConfig;
```

**证据**: [wifi_hotspot_config.h](interfaces/wifiservice/wifi_hotspot_config.h)（完整结构体定义）

### WifiEvent（事件回调）

**头文件**: [wifi_event.h](interfaces/wifiservice/wifi_event.h)

用于注册 Wi-Fi 状态变化回调：

```c
typedef struct {
    WifiEventStateChangedCallback onWifiStateChanged;
    WifiConnectionChangedCallback onConnectionChanged;
    WifiScanStateChangedCallback onScanResult;
} WifiEvent;
```

**证据**: [wifi_event.h](interfaces/wifiservice/wifi_event.h)（完整函数指针类型定义）

### WifiScanInfo（扫描信息）

**头文件**: [wifi_scan_info.h](interfaces/wifiservice/wifi_scan_info.h)

包含扫描发现的热点信息：

```c
typedef struct {
    char ssid[WIFI_MAX_SSID_LEN];       // 网络名称
    char bssid[WIFI_MAC_LEN];           // BSSID
    int securityType;                    // 安全类型
    int rssi;                           // 信号强度
    int band;                           // 频段
    int frequency;                      // 频率
    char flags[WIFI_SCAN_FLAGS_LEN];   // 扫描标志
} WifiScanInfo;
```

**证据**: [wifi_scan_info.h](interfaces/wifiservice/wifi_scan_info.h)（完整结构体定义）

### WifiLinkedInfo（连接信息）

**头文件**: [wifi_linked_info.h](interfaces/wifiservice/wifi_linked_info.h)

包含当前连接状态信息。

**证据**: [wifi_linked_info.h](interfaces/wifiservice/wifi_linked_info.h)

### StationInfo（站点信息）

**头文件**: [station_info.h](interfaces/wifiservice/station_info.h)

包含已连接热点/站点的信息。

**证据**: [station_info.h](interfaces/wifiservice/station_info.h)

---

## 错误码

### 头文件

```c
#include "wifi_error_code.h"
```

**证据**: [wifi_error_code.h](interfaces/wifiservice/wifi_error_code.h)

### 错误码清单

| 错误码 | 定义 | 描述 |
|--------|------|------|
| `WIFI_SUCCESS` | 操作成功 | - |
| `WIFI_FAILED` | 操作失败 | 通用失败 |
| `WIFI_ERROR_INVALID_PARAMS` | 参数无效 | 输入参数错误 |
| `WIFI_ERROR_NOT_SUPPORTED` | 不支持 | 功能不支持 |
| `WIFI_ERROR_NOT_OPENED` | 未开启 | Wi-Fi 未启用 |
| ... | ... | 完整列表见头文件 |

**注意**: 完整错误码列表请参考 [wifi_error_code.h](interfaces/wifiservice/wifi_error_code.h)。

---

## 事件回调注册

### 示例代码

```c
#include "wifi_device.h"

void OnWifiStateChanged(int state) {
    // state: WIFI_STATE_ENABLED 或 WIFI_STATE_DISABLED
}

void OnConnectionChanged(int state, WifiLinkedInfo* info) {
    // state: 连接状态变化
    // info: 连接信息
}

void OnScanResult() {
    // 扫描完成
}

int main() {
    WifiEvent event = {
        .onWifiStateChanged = OnWifiStateChanged,
        .onConnectionChanged = OnConnectionChanged,
        .onScanResult = OnScanResult,
    };
    
    RegisterWifiEvent(&event);
    EnableWifi();
    return 0;
}
```

---

## 注意事项

1. **线程安全**: 多线程环境下需自行加锁保护 API 调用
2. **资源释放**: 回调函数中不要执行阻塞操作
3. **错误处理**: 每次 API 调用后应检查返回值
4. **MAC 地址**: `GetDeviceMacAddress()` 返回的 MAC 地址可能为随机化地址

---

[返回 SUMMARY.md](SUMMARY.md) | [概览](01_Overview.md) | [构建系统](03_Build_System.md)
