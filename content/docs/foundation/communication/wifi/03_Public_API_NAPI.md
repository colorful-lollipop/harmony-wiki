# 对外 N-API（JavaScript API）文档

**目的**: 描述 WLAN 组件的所有对外 N-API（JavaScript API）、导出符号、权限、参数、错误码

**适用范围**: JavaScript/ArkTS 应用开发者、TypeScript 开发者

**生成时间**: 2026-02-06

---

## N-API 模块概览

WLAN 组件提供 2 个 N-API 模块：

| 模块名 | 导出名称 | 模块文件 | 库文件 | 说明 |
|---------|---------|---------|---------|------|
| Main WiFi | `wifi` / `wifiManager` | `wifi_napi_entry.cpp` | `wifi.z.so` | 主 WiFi 功能模块 |
| Extension | `wifiext` / `wifiManagerExt` | `wifi_ext_napi_entry.cpp` | `wifiext.z.so` | 扩展功能模块（如功率模型） |

### 模块命名规则
- `ENABLE_NAPI_WIFI_MANAGER` 宏定义时：
  - 模块名：`wifiManager` / `wifiManagerExt`
  - 导出类名使用 "Manager" 后缀
- 未定义时：
  - 模块名：`wifi` / `wifiext`
  - 使用简短命名

### 模块注册点
**证据**: `wifi/frameworks/js/napi/src/wifi_napi_entry.cpp:460-477`

```cpp
static napi_module wifiJsModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = NULL,
    .nm_register_func = Init,
#ifdef ENABLE_NAPI_WIFI_MANAGER
    .nm_modname = "wifiManager",
#else
    .nm_modname = "wifi",
#endif
    .nm_priv = ((void *)0),
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void RegisterModule(void) {
    napi_module_register(&wifiJsModule);
}
```

---

## STA（Station）模式 API

### WiFi 设备管理

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `enableWifi()` | `EnableWifi()` | 同步 | GET_WIFI_INFO, SET_WIFI_INFO | - | `boolean` |
| `disableWifi()` | `DisableWifi()` | 同步 | GET_WIFI_INFO, SET_WIFI_INFO | - | `boolean` |
| `enableSemiWifi()` | `EnableSemiWifi()` | 同步 | GET_WIFI_INFO, SET_WIFI_INFO, WIFI_CONNECTION | - | `boolean` |
| `isWifiActive()` | `IsWifiActive()` | 同步 | GET_WIFI_INFO | - | `boolean` |
| `getWifiDetailState()` | `GetWifiDetailState()` | 同步 | GET_WIFI_INFO | - | `number` |
| `getDeviceMacAddress()` | `GetDeviceMacAddress()` | 同步 | GET_WIFI_INFO, GET_WIFI_LOCAL_MAC | - | `string` |

### 扫描功能

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `scan()` | `Scan()` | 同步 | SET_WIFI_INFO, GET_WIFI_INFO | - | `boolean` |
| `startScan()` | `StartScan()` | 同步 | SET_WIFI_INFO, GET_WIFI_INFO | - | `boolean` |
| `getScanInfos()` | `GetScanInfoResults()` | 异步（Promise） | GET_WIFI_INFO | - | `Promise<Array<WifiScanInfo>>` |
| `getScanInfosSync()` | `GetScanResults()` | 同步 | GET_WIFI_INFO | - | `Array<WifiScanInfo>` |
| `getScanResults()` | `GetScanInfos()` | 异步（Callback） | GET_WIFI_INFO | `callback: AsyncCallback<Array<WifiScanInfo>>` | `void` |
| `getScanResultsSync()` | `GetScanInfos()` | 同步 | GET_WIFI_INFO | - | `Array<WifiScanInfo>` |
| `getScanInfoList()` | `GetScanInfoList()` | 同步 | GET_WIFI_INFO | - | `Array<WifiScanInfo>` |

### 配置管理

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `addDeviceConfig()` | `AddDeviceConfig()` | 异步（Promise） | SET_WIFI_INFO, SET_WIFI_CONFIG | `config: WifiDeviceConfig` | `Promise<number>` |
| `addDeviceConfig(callback)` | `AddDeviceConfig()` | 异步（Callback） | SET_WIFI_INFO, SET_WIFI_CONFIG | `config: WifiDeviceConfig, callback: AsyncCallback<number>` | `void` |
| `addUntrustedConfig()` | `AddUntrustedConfig()` | 异步 | SET_WIFI_INFO, SET_WIFI_CONFIG | `config: WifiDeviceConfig` | `Promise<number>` |
| `removeUntrustedConfig()` | `RemoveUntrustedConfig()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | `networkId: number` | `boolean` |
| `addCandidateConfig()` | `AddCandidateConfig()` | 异步 | SET_WIFI_INFO, SET_WIFI_CONFIG | `config: WifiDeviceConfig` | `Promise<number>` |
| `removeCandidateConfig()` | `RemoveCandidateConfig()` | 同步 | SET_WIFI_INFO | `networkId: number` | `boolean` |
| `connectToCandidateConfig()` | `ConnectToCandidateConfig()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | `networkId: number` | `boolean` |
| `connectToCandidateConfigWithUserAction()` | `ConnectToCandidateConfigWithUserAction()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | `networkId: number, action: number` | `boolean` |
| `getCandidateConfigs()` | `GetCandidateConfigs()` | 同步 | GET_WIFI_CONFIG | - | `Array<WifiDeviceConfig>` |
| `removeDevice()` | `RemoveDevice()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | `networkId: number` | `boolean` |
| `removeDeviceConfig()` | 同上 | - | - | - |
| `removeAllNetwork()` | `RemoveAllNetwork()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | - | `boolean` |
| `removeAllDeviceConfigs()` | 同上 | - | - | - |
| `allowAutoConnect()` | `AllowAutoConnect()` | 同步 | SET_WIFI_INFO | `networkId: number, allow: boolean` | `boolean` |
| `disableNetwork()` | `DisableNetwork()` | 同步 | SET_WIFI_INFO | `networkId: number` | `boolean` |
| `updateNetwork()` | `UpdateNetwork()` | 同步 | SET_WIFI_INFO, SET_WIFI_CONFIG | `networkId: number, config: WifiDeviceConfig` | `boolean` |
| `getDeviceConfigs()` | `GetDeviceConfigs()` | 同步 | GET_WIFI_CONFIG | - | `Array<WifiDeviceConfig>` |

### 连接管理

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `connectToNetwork()` | `ConnectToNetwork()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | `networkId: number` | `boolean` |
| `connectToDevice()` | `ConnectToDevice()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION, SET_WIFI_CONFIG | `config: WifiDeviceConfig` | `boolean` |
| `isConnected()` | `IsConnected()` | 同步 | GET_WIFI_INFO | - | `boolean` |
| `disconnect()` | `Disconnect()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | - | `boolean` |
| `reconnect()` | `ReConnect()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | - | `boolean` |
| `reassociate()` | `ReAssociate()` | 同步 | SET_WIFI_INFO, WIFI_CONNECTION | - | `boolean` |

### 信息查询

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `getLinkedInfo()` | `GetLinkedInfo()` | 异步（Promise） | GET_WIFI_INFO, GET_WIFI_LOCAL_MAC, GET_WIFI_PEERS_MAC | - | `Promise<WifiLinkedInfo>` |
| `getLinkedInfoSync()` | `GetLinkedInfoSync()` | 同步 | GET_WIFI_INFO, GET_WIFI_LOCAL_MAC, GET_WIFI_PEERS_MAC | - | `WifiLinkedInfo` |
| `getMultiLinkedInfo()` | `GetMultiLinkedInfo()` | 异步（Promise） | GET_WIFI_INFO | - | `Promise<Array<WifiLinkedInfo>>` |
| `getIpInfo()` | `GetIpInfo()` | 异步（Promise） | GET_WIFI_INFO | - | `Promise<IpInfo>` |
| `getIpv6Info()` | `GetIpv6Info()` | 异步（Promise） | GET_WIFI_INFO | - | `Promise<Ipv6Info>` |
| `getDisconnectedReason()` | `GetDisconnectedReason()` | 同步 | GET_WIFI_INFO | - | `number` |
| `isMeteredHotspot()` | `IsMeteredHotspot()` | 同步 | GET_WIFI_INFO | - | `boolean` |
| `getCountryCode()` | `GetCountryCode()` | 异步（Promise） | GET_WIFI_INFO | - | `Promise<string>` |
| `getSignalLevel()` | `GetSignalLevel()` | 同步 | GET_WIFI_INFO | `rssi: number, band: number` | `number` |
| `getSupportedFeatures()` | `GetSupportedFeatures()` | 同步 | GET_WIFI_INFO | - | `Array<number>` |
| `isFeatureSupported()` | `IsFeatureSupported()` | 同步 | GET_WIFI_INFO | `featureId: number` | `boolean` |

### 高级功能

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `setScanAlwaysAllowed()` | `SetScanOnlyAvailable()` | 同步 | SET_WIFI_INFO | `always: boolean` | `boolean` |
| `getScanAlwaysAllowed()` | `GetScanOnlyAvailable()` | 同步 | GET_WIFI_INFO | - | `boolean` |
| `isBandTypeSupported()` | `IsBandTypeSupported()` | 同步 | GET_WIFI_INFO | `bandType: number` | `boolean` |
| `get5GChannelList()` | `Get5GHzChannelList()` | 同步 | GET_WIFI_INFO | - | `Array<number>` |
| `startPortalCertification()` | `StartPortalCertification()` | 同步 | SET_WIFI_CONFIG | `url: string` | `boolean` |
| `getWifiProtect()` | `GetWifiProtectRef()` | 同步 | GET_WIFI_INFO | - | `number` |
| `putWifiProtect()` | `PutWifiProtectRef()` | 同步 | SET_WIFI_INFO | `ref: number` | `boolean` |
| `factoryReset()` | `FactoryReset()` | 同步 | SET_WIFI_INFO, SET_WIFI_CONFIG | - | `boolean` |
| `startWifiDetection()` | `StartWifiDetection()` | 同步 | SET_WIFI_INFO | - | `boolean` |
| `enableHiLinkHandshake()` | `EnableHiLinkHandshake()` | 同步 | SET_WIFI_INFO | - | `boolean` |
| `isRandomMacDisabled()` | `IsRandomMacDisabled()` | 同步 | GET_WIFI_INFO | - | `boolean` |

---

## AP（热点）模式 API

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `isHotspotActive()` | `IsHotspotActive()` | 同步 | GET_WIFI_INFO | - | `boolean` |
| `isHotspotDualBandSupported()` | `IsHotspotDualBandSupported()` | 同步 | GET_WIFI_INFO, MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `isOpenSoftApAllowed()` | `IsOpenSoftApAllowed()` | 同步 | GET_WIFI_INFO, MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `enableHotspot()` | `EnableHotspot()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `disableHotspot()` | `DisableHotspot()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `setHotspotConfig()` | `SetHotspotConfig()` | 同步 | MANAGE_WIFI_HOTSPOT, SET_WIFI_CONFIG | `config: HotspotConfig` | `boolean` |
| `getHotspotConfig()` | `GetHotspotConfig()` | 同步 | GET_WIFI_INFO, MANAGE_WIFI_HOTSPOT | - | `HotspotConfig` |
| `getStations()` | `GetStations()` | 异步（Promise） | MANAGE_WIFI_HOTSPOT | - | `Promise<Array<StationInfo>>` |
| `getHotspotStations()` | 同上 | - | - | - |
| `addHotspotBlockList()` | `AddHotspotBlockedList()` | 同步 | MANAGE_WIFI_HOTSPOT | `list: Array<string>` | `boolean` |
| `delHotspotBlockList()` | `DelHotspotBlockedList()` | 同步 | MANAGE_WIFI_HOTSPOT | `list: Array<string>` | `boolean` |
| `getHotspotBlockList()` | `GetHotspotBlockedList()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `Array<string>` |
| `setHotspotIdleTimeout()` | `SetHotspotIdleTimeout()` | 同步 | MANAGE_WIFI_HOTSPOT | `timeout: number` | `boolean` |

---

## P2P（WiFi Direct）API

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|---------|----------|----------|------|------|--------|
| `getP2pLinkedInfo()` | `GetP2pLinkedInfo()` | 同步 | GET_WIFI_INFO | - | `P2pLinkedInfo` |
| `getCurrentGroup()` | `GetCurrentGroup()` | 同步 | GET_WIFI_INFO | - | `P2pGroupInfo` |
| `getP2pPeerDevices()` | `GetP2pDevices()` | 同步 | GET_WIFI_INFO | - | `Array<P2pDevice>` |
| `getP2pLocalDevice()` | `GetP2pLocalDevice()` | 同步 | GET_WIFI_INFO | - | `P2pLocalDevice` |
| `createGroup()` | `CreateGroup()` | 同步 | MANAGE_WIFI_HOTSPOT | `config: P2pConfig` | `boolean` |
| `removeGroup()` | `RemoveGroup()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `p2pConnect()` | `P2pConnect()` | 同步 | MANAGE_WIFI_HOTSPOT | `config: WifiP2pConfig` | `boolean` |
| `p2pCancelConnect()` / `p2pDisconnect()` | `P2pCancelConnect()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `startDiscoverDevices()` | `StartDiscoverDevices()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `stopDiscoverDevices()` | `StopDiscoverDevices()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `deletePersistentGroup()` | `DeletePersistentGroup()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `getP2pGroups()` | `GetP2pGroups()` | 同步 | GET_WIFI_INFO | - | `Array<P2pGroupInfo>` |
| `setDeviceName()` | `SetDeviceName()` | 同步 | MANAGE_WIFI_HOTSPOT | `name: string` | `boolean` |

---

## 事件订阅 API

| JS API | C++ 函数 | 参数 | 说明 |
|---------|----------|------|------|
| `on()` | `On()` | `event: string, callback: AsyncCallback<any>` | 订阅 WiFi 事件 |
| `off()` | `Off()` | `event: string` | 取消订阅 WiFi 事件 |

### 支持的事件类型
| 事件类型 | 参数类型 | 触发时机 | 所需权限 |
|-----------|---------|---------|----------|
| `wifiStateChange` | 状态变化 | WiFi 启用/禁用 | GET_WIFI_INFO |
| `wifiConnectionChange` | 连接状态变化 | 已连接/断开 | GET_WIFI_INFO |
| `scanStateChange` | 扫描状态变化 | 开始/结束/完成 | GET_WIFI_INFO |
| `rssiChange` | 信号强度变化 | RSSI 变化 | GET_WIFI_INFO |
| `wifiDeviceConfigChange` | 设备配置变化 | 配置添加/删除/更新 | GET_WIFI_INFO |
| `streamChange` | 流向变化 | 数据上传/下载 | MANAGE_WIFI_CONNECTION |
| `hotspotStateChange` | 热点状态变化 | AP 启用/禁用 | MANAGE_WIFI_HOTSPOT |
| `hotspotStaJoin` | 站点加入 | 新设备连接热点 | MANAGE_WIFI_HOTSPOT |
| `hotspotStaLeave` | 站点离开 | 设备断开热点 | MANAGE_WIFI_HOTSPOT |
| `p2pStateChange` | P2P 状态变化 | P2P 功能状态改变 | GET_WIFI_INFO |
| `p2pConnectionChange` | P2P 连接变化 | P2P 连接建立/断开 | GET_WIFI_INFO |
| `p2pDeviceChange` | P2P 设备变化 | 发现/失去设备 | GET_WIFI_INFO, LOCATION |
| `p2pPersistentGroupChange` | P2P 组变化 | 持久化组创建/删除 | GET_WIFI_INFO |
| `p2pPeerDeviceChange` | P2P 对端设备变化 | P2P 对端信息更新 | GET_WIFI_INFO, LOCATION, GET_WIFI_INFO_INTERNAL |
| `p2pDiscoveryChange` | P2P 发现状态变化 | 发现开始/结束 | GET_WIFI_INFO |

---

## 常量/枚举定义

### WiFi 状态枚举

**证据**: `wifi/frameworks/js/napi/src/wifi_napi_entry.cpp:24-287`

| 枚举名 | 值 | 说明 |
|---------|-----|------|
| `SuppState` | DISCONNECTED, INTERFACE_DISABLED, INACTIVE, SCANNING, AUTHENTICATING, ASSOCIATING, ASSOCIATED, FOUR_WAY_HANDSHAKE, GROUP_HANDSHAKE, COMPLETED, UNINITIALIZED, INVALID | Supplicant 状态 |
| `SecurityType` | WIFI_SEC_TYPE_INVALID, OPEN, WEP, PSK, SAE, EAP, EAP_SUITE_B, OWE, WAPI_CERT, WAPI_PSK | 安全类型 |
| `IpType` | STATIC, DHCP, UNKNOWN | IP 类型 |
| `ConnState` | SCANNING, CONNECTING, AUTHENTICATING, OBTAINING_IPADDR, CONNECTED, DISCONNECTING, DISCONNECTED, UNKNOWN | 连接状态 |
| `WifiChannelWidth` | WIDTH_20MHZ, WIDTH_40MHZ, WIDTH_80MHZ, WIDTH_160MHZ, WIDTH_80MHZ_PLUS, WIDTH_INVALID | 信道宽度 |
| `WifiStandard` | WIFI_STANDARD_11A/B/G/N/AC/AX/AD/UNDEFINED | WiFi 标准 |
| `WifiBandType` | WIFI_BAND_NONE, 2G, 5G, 6G, 60G | 频段类型 |
| `WifiDetailState` | UNKNOWN, INACTIVE, ACTIVATED, ACTIVATING, DEACTIVATING, SEMI_ACTIVATING, SEMI_ACTIVE | WiFi 详细状态 |
| `WifiCategory` | DEFAULT, WIFI6, WIFI6_PLUS, WIFI7, WIFI7_PLUS | WiFi 类别 |
| `WifiLinkType` | DEFAULT_LINK, WIFI7_SINGLE_LINK, WIFI7_MLSR, WIFI7_EMLSR, WIFI7_STR, WIFI7_LEGACY | WiFi7 链路类型 |
| `EapMethod` | EAP_NONE, PEAP, TLS, TTLS, PWD, SIM, AKA, AKA_PRIME, UNAUTH_TLS | EAP 方法 |
| `WifiStandard` | WIFI_STANDARD_11A/B/G/N/AC/AX/AD | WiFi 标准 |
| `ProxyMethod` | METHOD_NONE, METHOD_AUTO, METHOD_MANUAL | 代理方法 |

### P2P 相关枚举

| 枚举名 | 值 | 说明 |
|---------|-----|------|
| `P2pConnectState` | DISCONNECTED, CONNECTED | P2P 连接状态 |
| `P2pDeviceStatus` | CONNECTED, INVITED, FAILED, AVAILABLE, UNAVAILABLE | P2P 设备状态 |
| `GroupOwnerBand` | GO_BAND_AUTO, GO_BAND_2GHZ, GO_BAND_5GHZ | 组所有者频段 |
| `DisconnectedReason` | DISC_REASON_DEFAULT, WRONG_PWD, CONNECTION_FULL, CONNECTION_REJECTED | 断开原因 |

---

## 数据结构

### WifiScanInfo
**定义**: `kits/c/wifi_scan_info.h`
```typescript
interface WifiScanInfo {
    ssid: string;           // SSID
    bssid: string;          // BSSID
    securityType: number;   // 安全类型（WifiSecurityType 枚举）
    rssi: number;           // 信号强度（dBm）
    band: number;           // 频段（WifiBandType 枚举）
    frequency: number;       // 频率（MHz）
    timestamp: number;       // 扫描时间戳
    capabilities: number;    // 能力标志
    channelWidth: number;   // 信道宽度（WifiChannelWidth 枚举）
}
```

### WifiLinkedInfo
**定义**: `kits/c/wifi_linked_info.h`
```typescript
interface WifiLinkedInfo {
    ssid: string;              // SSID
    bssid: string;             // BSSID
    networkId: number;         // 网络 ID
    isLinked: boolean;          // 是否已连接
    rssi: number;              // 信号强度（dBm）
    band: number;               // 频段
    frequency: number;           // 频率（MHz）
    linkSpeed: number;          // 链路速度（Mbps）
    ipAddress: string;         // IP 地址
    macAddress: string;         // MAC 地址
    ipType: number;           // IP 类型（IpType 枚举）
    subnetMask: string;         // 子网掩码
    gateway: string;           // 网关地址
    dnsServers: string[];     // DNS 服务器列表
}
```

### WifiDeviceConfig
**定义**: `kits/c/wifi_device_config.h`
```typescript
interface WifiDeviceConfig {
    ssid: string;              // SSID
    bssid: string;             // BSSID（可选）
    preSharedKey: string;      // 预共享密钥（WPA/WPA2）
    isHiddenSsid: boolean;      // 是否为隐藏 SSID
    securityType: number;       // 安全类型（WifiSecurityType 枚举）
    keyMgmt: number;          // 密钥管理
    priority: number;           // 优先级
    networkId: number;          // 网络 ID
    passphrase: string;        // 密码短语
    wepTxKeyIndex: number;     // WEP TX 密钥索引
    creatorUid: number;        // 创建者 UID
}
```

### HotspotConfig
**定义**: `kits/c/wifi_hotspot_config.h`
```typescript
interface HotspotConfig {
    ssid: string;              // SSID
    securityType: number;       // 安全类型
    band: number;               // 频段
    channel: number;            // 信道
    maxConn: number;            // 最大连接数
    passphrase: string;        // 密码短语
    preSharedKey: string;      // 预共享密钥
    isHiddenSsid: boolean;      // 是否为隐藏 SSID
}
```

---

## 扩展模块 API

**证据**: `wifi/frameworks/js/napi/src/wifi_ext_napi_entry.cpp:21-77`

| JS API | C++ 函数 | 同步/异步 | 权限 | 参数 | 返回值 | 说明 |
|---------|----------|----------|------|------|--------|
| `enableHotspot()` | `EnableHotspot()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `disableHotspot()` | `DisableHotspot()` | 同步 | MANAGE_WIFI_HOTSPOT | - | `boolean` |
| `getSupportedPowerModel()` | `GetSupportedPowerModel()` | 同步 | GET_WIFI_INFO, MANAGE_WIFI_HOTSPOT | - | `Array<number>` |
| `getPowerModel()` | `GetPowerModel()` | 同步 | GET_WIFI_INFO, MANAGE_WIFI_HOTSPOT | - | `PowerModel` |
| `setPowerModel()` | `SetPowerModel()` | 同步 | MANAGE_WIFI_HOTSPOT | `model: PowerModel` | `boolean` |

### PowerModel 枚举
**证据**: `wifi/frameworks/js/napi/src/wifi_ext_napi_entry.cpp:21-29`

| 枚举值 | 说明 |
|---------|------|
| `SLEEPING` | 睡眠模式 |
| `GENERAL` | 通用模式 |
| `THROUGH_WALL` | 穿墙模式 |

---

## 错误码

### 常见错误码
| 错误码 | 名称 | 说明 |
|---------|------|------|
| 0 | SUCCESS | 操作成功 |
| -1 | ERROR_WIFI_UNKNOWN | 未知错误 |
| -2 | ERROR_WIFI_NOT_SUPPORTED | 功能不支持 |
| -3 | ERROR_WIFI_INVALID_ARGS | 参数无效 |
| -4 | ERROR_WIFI_BUSY | WiFi 繁忙 |
| -5 | ERROR_WIFI_NOT_STARTED | WiFi 未启动 |
| -6 | ERROR_WIFI_OPERATING | 正在操作中 |

**详细错误码列表请参考**: `kits/c/wifi_error_code.h`

---

## 调用链示例

### 连接 WiFi 网络

```javascript
import wifi from '@ohos.wifi';

// 方式一：使用配置 ID 连接
const networkId = await wifi.addDeviceConfig(config);
const connected = await wifi.connectToNetwork(networkId);

// 方式二：使用配置直接连接
await wifi.connectToDevice(config);

// 获取连接信息
const linkedInfo = await wifi.getLinkedInfo();
console.log(`SSID: ${linkedInfo.ssid}`);
console.log(`IP: ${linkedInfo.ipAddress}`);
```

### 扫描 WiFi 网络

```javascript
import wifi from '@ohos.wifi';

// 发起扫描
await wifi.scan();

// 获取扫描结果
const scanResults = await wifi.getScanInfos();
scanResults.forEach((ap, index) => {
    console.log(`AP ${index}:`);
    console.log(`  SSID: ${ap.ssid}`);
    console.log(`  BSSID: ${ap.bssid}`);
    console.log(`  RSSI: ${ap.rssi} dBm`);
    console.log(`  安全类型: ${ap.securityType}`);
});
```

### 订阅事件

```javascript
import wifi from '@ohos.wifi';

// 订阅 WiFi 状态变化事件
wifi.on('wifiStateChange', (data) => {
    console.log(`WiFi 状态: ${data.state}`);
});

// 订阅连接状态变化事件
wifi.on('wifiConnectionChange', (data) => {
    if (data.state === 1) {
        console.log(`已连接到: ${data.linkInfo.ssid}`);
    } else {
        console.log('已断开连接');
    }
});

// 取消订阅
wifi.off('wifiConnectionChange');
```

---

## 权限说明

### 所需权限

| 权限名 | 用途 | API 示例 |
|---------|------|---------|
| `ohos.permission.GET_WIFI_INFO` | 读取 WiFi 状态和基本信息 | `isWifiActive()`, `getScanInfos()`, `getLinkedInfo()` |
| `ohos.permission.SET_WIFI_INFO` | 修改 WiFi 状态 | `enableWifi()`, `disableWifi()` |
| `ohos.permission.MANAGE_WIFI_CONNECTION` | 管理 WiFi 连接 | `connectToNetwork()`, `disconnect()` |
| `ohos.permission.SET_WIFI_CONFIG` | 修改 WiFi 配置 | `addDeviceConfig()`, `updateNetwork()` |
| `ohos.permission.GET_WIFI_CONFIG` | 读取 WiFi 配置 | `getDeviceConfigs()` |
| `ohos.permission.MANAGE_WIFI_HOTSPOT` | 管理热点 | `enableHotspot()`, `setHotspotConfig()` |
| `ohos.permission.GET_WIFI_LOCAL_MAC` | 获取本地 MAC 地址 | `getDeviceMacAddress()` |
| `ohos.permission.LOCATION` | 基于位置的扫描 | `scan()` |
| `ohos.permission.GET_WIFI_PEERS_MAC` | 获取对端 MAC 地址 | `getLinkedInfo()` |

### 权限验证流程

**证据**: `wifi_permission_helper.cpp:28-51`

1. **同进程检查**: 调用者与服务在同一进程时自动授予权限
2. **令牌获取**: 获取调用者的 AccessTokenID
3. **权限验证**: 使用 AccessTokenKit 验证权限
4. **结果返回**: 返回 `PERMISSION_GRANTED` 或 `PERMISSION_DENIED`

---

## 参数校验

### 类型检查
- **SSID**: 字符串类型，长度限制
- **密码**: 字符串类型，最小长度验证
- **网络 ID**: 数字类型，范围检查
- **枚举值**: 必须是有效的枚举值
- **布尔值**: 必须是布尔类型

### 范围检查
- **RSSI**: 有效范围（-100 到 0 dBm）
- **信道**: 1-14 或有效信道列表
- **优先级**: 有效优先级范围

---

## 异步处理

### Promise 模式
```javascript
// Promise 模式 API
const result = await wifi.getScanInfos();
// Promise 会在异步操作完成后 resolve 或 reject
```

**实现细节**:
- 使用 `napi_create_promise()` 创建 Promise 对象
- 使用 `napi_resolve_deferred()` 处理异步结果
- 在工作线程中执行实际操作
- 通过 Event Handler 回调回到 JS 线程

### Callback 模式
```javascript
// Callback 模式 API
wifi.getScanInfos((err, result) => {
    if (err) {
        console.error('扫描失败:', err);
        return;
    }
    console.log('扫描结果:', result);
});
```

**实现细节**:
- 使用 `napi_create_threadsafe_function()` 创建线程安全的函数
- 通过 `napi_create_async_work()` 创建异步工作上下文
- 使用 `napi_async_complete()` 完成异步操作

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构与模块职责
- [02_Architecture.md](02_Architecture.md) - 架构说明
- [04_Inner_API.md](04_Inner_API.md) - 内部 API

---

**下一步**: 阅读 [05_GN_Targets.md](05_GN_Targets.md) 了解 GN 构建系统和编译产物。
