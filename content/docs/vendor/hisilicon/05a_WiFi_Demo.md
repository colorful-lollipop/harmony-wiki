# WiFi Demo 详解

本文档详细说明 easy_wifi_demo 的实现原理和使用方法。

---

## 5.a.1 Demo 概述

**位置**：`hispark_pegasus/demo/easy_wifi_demo/`

**目的**：简化 OpenHarmony WiFi API 的使用，提供更易用的接口封装。

**特点**：
- STA 模式：连接 WiFi 热点
- AP 模式：创建 WiFi 热点
- 事件回调机制
- DHCP 客户端/服务端支持

---

## 5.a.2 文件结构

```
easy_wifi_demo/
├── demo/                          # Demo 入口
│   ├── BUILD.gn                  # 构建配置 [证据]
│   ├── wifi_connect_demo.c       # STA 模式示例
│   └── wifi_hotspot_demo.c       # AP 模式示例
├── src/                          # 核心实现
│   ├── BUILD.gn                  # 构建配置
│   ├── wifi_starter.c            # WiFi 启动器 [证据]
│   ├── wifi_starter.h            # 头文件
│   ├── wifi_connecter.c          # 连接器
│   ├── wifi_connecter.h          # 头文件
│   └── wifi_connecter.c
└── README.md                      # 说明文档
```

---

## 5.a.3 简化 API

### STA 模式

```c
// 连接到热点
int ConnectToHotspot(WifiDeviceConfig* apConfig);

// 断开连接
void DisconnectWithHotspot(int netId);
```

### AP 模式

```c
// 启动热点
int StartHotspot(const HotspotConfig* config);

// 停止热点
void StopHotspot(void);
```

---

## 5.a.4 核心实现

### 5.a.4.1 热点状态回调

**文件**：`wifi_starter.c:31-39` [证据]

```c
static void OnHotspotStateChanged(int state)
{
    printf("OnHotspotStateChanged: %d.\r\n", state);
    if (state == WIFI_HOTSPOT_ACTIVE) {
        g_hotspotStarted = 1;
    } else {
        g_hotspotStarted = 0;
    }
}
```

### 5.a.4.2 站点事件回调

**文件**：`wifi_starter.c:43-67` [证据]

```c
static void PrintStationInfo(StationInfo* info)
{
    if (!info) return;
    static char macAddress[32] = {0};
    unsigned char* mac = info->macAddress;
    // MAC 地址格式化
    if (snprintf_s(macAddress, sizeof(macAddress), "%02X:%02X:%02X:%02X:%02X:%02X",
        mac[ZERO], mac[ONE], mac[TWO], mac[THREE], mac[FOUR], mac[FIVE]) == TRUE) {
    printf("OK")
    }
    printf(" PrintStationInfo: mac=%s, reason=%d.\r\n", macAddress, info->disconnectedReason);
}

static void OnHotspotStaJoin(StationInfo* info)
{
    g_joinedStations++;
    PrintStationInfo(info);
    printf("+OnHotspotStaJoin: active stations = %d.\r\n", g_joinedStations);
}

static void OnHotspotStaLeave(StationInfo* info)
{
    g_joinedStations--;
    PrintStationInfo(info);
    printf("-OnHotspotStaLeave: active stations = %d.\r\n", g_joinedStations);
}
```

### 5.a.4.3 事件监听器注册

**文件**：`wifi_starter.c:69-73` [证据]

```c
WifiEvent g_defaultWifiEventListener = {
    .OnHotspotStaJoin = OnHotspotStaJoin,
    .OnHotspotStaLeave = OnHotspotStaLeave,
    .OnHotspotStateChanged = OnHotspotStateChanged,
};
```

### 5.a.4.4 启动热点

**文件**：`wifi_starter.c:77-112` [证据]

```c
int StartHotspot(const HotspotConfig* config)
{
    WifiErrorCode errCode = WIFI_SUCCESS;

    // 1. 注册事件监听
    errCode = RegisterWifiEvent(&g_defaultWifiEventListener);
    printf("RegisterWifiEvent: %d\r\n", errCode);

    // 2. 设置热点配置
    errCode = SetHotspotConfig(config);
    printf("SetHotspotConfig: %d\r\n", errCode);

    // 3. 启用热点
    g_hotspotStarted = 0;
    errCode = EnableHotspot();
    printf("EnableHotspot: %d\r\n", errCode);

    // 4. 等待热点启动
    while (!g_hotspotStarted) {
        osDelay(TEN);
    }
    printf("g_hotspotStarted = %d.\r\n", g_hotspotStarted);

    // 5. 配置网络接口
    g_iface = netifapi_netif_find("ap0");
    if (g_iface) {
        ip4_addr_t ipaddr;
        ip4_addr_t gateway;
        ip4_addr_t netmask;

        IP4_ADDR(&ipaddr,  192, 168, 1, 1);     // 硬编码 IP
        IP4_ADDR(&gateway, 192, 168, 1, 1);     // 网关
        IP4_ADDR(&netmask, 255, 255, 255, 0);   // 子网掩码

        err_t ret = netifapi_netif_set_addr(g_iface, &ipaddr, &netmask, &gateway);
        printf("netifapi_netif_set_addr: %d\r\n", ret);

        // 6. 启动 DHCP 服务端
        ret = netifapi_dhcps_start(g_iface, 0, 0);
        printf("netifapi_dhcp_start: %d\r\n", ret);
    }
    return errCode;
}
```

### 5.a.4.5 停止热点

**文件**：`wifi_starter.c:114-126` [证据]

```c
void StopHotspot(void)
{
    if (g_iface) {
        err_t ret = netifapi_dhcps_stop(g_iface);
        printf("netifapi_dhcps_stop: %d\r\n", ret);
    }

    WifiErrorCode errCode = UnRegisterWifiEvent(&g_defaultWifiEventListener);
    printf("UnRegisterWifiEvent: %d\r\n", errCode);

    errCode = DisableHotspot();
    printf("EnableHotspot: %d\r\n", errCode);
}
```

---

## 5.a.5 使用示例

### STA 模式

```c
WifiDeviceConfig apConfig = {0};
// 配置热点参数
strcpy(apConfig.ssid, "MyWiFi");
strcpy(apConfig.preSharedKey, "password");
apConfig.securityType = WIFI_SECURITY_TYPE_PSK;

// 连接
int netId;
int ret = ConnectToHotspot(&apConfig);
if (ret == WIFI_SUCCESS) {
    printf("Connected to hotspot\r\n");
}
```

### AP 模式

```c
HotspotConfig config = {0};
// 配置热点参数
strcpy(config.ssid, "MyHotspot");
strcpy(config.preSharedKey, "12345678");
config.band = HOTSPOT_BAND_2G;
config.channelNum = 6;

// 启动热点
int ret = StartHotspot(&config);
if (ret == WIFI_SUCCESS) {
    printf("Hotspot started\r\n");
}

// 停止热点
StopHotspot();
```

---

## 5.a.6 相关文档

- [Demo 示例](./05_Demos.md)
- [配置体系](../04_Configuration.md)
- [安全指南](../07_Security.md)
