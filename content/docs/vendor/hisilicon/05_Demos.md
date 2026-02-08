# Demo 示例

本文档介绍 Hisilicon Vendor 仓库中提供的 Demo 示例程序。

---

## 5.1 Demo 总览

**Demo 位置**：`hispark_pegasus/demo/`

| Demo | 功能 | 系统 | 难度 |
|-----|------|------|------|
| [easy_wifi_demo](#52-easy_wifi_demo) | WiFi STA/AP 模式 | LiteOS-M | 初级 |
| [environment_demo](#53-environment_demo) | 环境传感器检测 | LiteOS-M | 中级 |
| [led_demo](#54-led_demo) | LED 控制 | LiteOS-M | 初级 |
| [mqtt_demo](#55-mqtt_demo) | MQTT 通信 | LiteOS-M | 中级 |
| [coap_demo](#56-coap_demo) | CoAP 通信 | LiteOS-M | 中级 |
| [mutex_demo](#57-mutex_demo) | 互斥锁使用 | LiteOS-M | 初级 |
| [semaphore_demo](#58-semaphore_demo) | 信号量使用 | LiteOS-M | 初级 |

---

## 5.2 easy_wifi_demo

### 5.2.1 功能说明

提供简化的 WiFi API 接口，支持 STA（站点）模式和 AP（热点）模式。

**位置**：`hispark_pegasus/demo/easy_wifi_demo/`

### 5.2.2 简化 API

**STA 模式**：

```c
// 连接热点
int ConnectToHotspot(WifiDeviceConfig* apConfig);

// 断开连接
void DisconnectWithHotspot(int netId);
```

**AP 模式**：

```c
// 启动热点
int StartHotspot(const HotspotConfig* config);

// 停止热点
void StopHotspot(void);
```

### 5.2.3 原始 WiFi API

**STA 模式 API**：

| API | 功能 |
|-----|------|
| `EnableWifi()` | 开启 STA [证据：easy_wifi_demo/README.md] |
| `DisableWifi()` | 关闭 STA |
| `IsWifiActive()` | 查询 STA 状态 |
| `Scan()` | 触发扫描 |
| `GetScanInfoList()` | 获取扫描结果 |
| `AddDeviceConfig()` | 添加热点配置 |
| `GetDeviceConfigs()` | 获取热点配置 |
| `RemoveDevice()` | 删除热点配置 |
| `ConnectTo()` | 连接到热点 |
| `Disconnect()` | 断开连接 |
| `GetLinkedInfo()` | 获取连接信息 |
| `RegisterWifiEvent()` | 注册事件监听 |
| `UnRegisterWifiEvent()` | 解除事件监听 |
| `GetDeviceMacAddress()` | 获取 MAC 地址 |

**AP 模式 API**：

| API | 功能 |
|-----|------|
| `EnableHotspot()` | 开启 AP |
| `DisableHotspot()` | 关闭 AP |
| `SetHotspotConfig()` | 设置热点配置 |
| `GetHotspotConfig()` | 获取热点配置 |
| `IsHotspotActive()` | 查询 AP 状态 |
| `GetStationList()` | 获取接入设备列表 |
| `GetSignalLevel()` | 获取信号强度 |
| `SetBand()` | 设置频段 |
| `GetBand()` | 获取频段 |

### 5.2.4 DHCP 接口

**DHCP 客户端**：

| API | 描述 |
|-----|------|
| `netifapi_netif_find()` | 查找网络接口 |
| `netifapi_dhcp_start()` | 启动 DHCP 客户端 |
| `netifapi_dhcp_stop()` | 停止 DHCP 客户端 |

**DHCP 服务端**：

| API | 描述 |
|-----|------|
| `netifapi_netif_set_addr()` | 设置接口地址 |
| `netifapi_dhcps_start()` | 启动 DHCP 服务端 |
| `netifapi_dhcps_stop()` | 停止 DHCP 服务端 |

### 5.2.5 关键代码

**wifi_starter.c** [证据：demo/easy_wifi_demo/src/wifi_starter.c]

```c
// 热点状态回调
static void OnHotspotStateChanged(int state)
{
    printf("OnHotspotStateChanged: %d.\r\n", state);
}

// 热点事件监听器
WifiEvent g_defaultWifiEventListener = {
    .OnHotspotStaJoin = OnHotspotStaJoin,
    .OnHotspotStaLeave = OnHotspotStaLeave,
    .OnHotspotStateChanged = OnHotspotStateChanged,
};

// 启动热点
int StartHotspot(const HotspotConfig* config)
{
    WifiErrorCode errCode = WIFI_SUCCESS;

    errCode = RegisterWifiEvent(&g_defaultWifiEventListener);
    errCode = SetHotspotConfig(config);
    errCode = EnableHotspot();
    // ...
}
```

---

## 5.3 environment_demo

### 5.3.1 功能说明

环境传感器检测 Demo，演示多种传感器的使用。

**位置**：`hispark_pegasus/demo/environment_demo/`

### 5.3.2 支持传感器

| 传感器 | 功能 | 头文件 |
|-------|------|-------|
| MQ-2 | 烟雾检测 | app_demo_mq2.h |
| GL5537-1 | 光照检测 | app_demo_gl5537_1.h |
| AHT20 | 温湿度 | app_demo_aht20.h |
| I2C OLED | 显示 | app_demo_i2c_oled.h |

### 5.3.3 关键文件

| 文件 | 功能 |
|-----|------|
| app_demo_environment.c | 主程序 |
| app_demo_environment.h | 头文件 |
| iot_adc.h | ADC 接口 |
| iot_gpio_ex.h | GPIO 扩展接口 |
| hal_iot_adc.c | ADC 硬件抽象层 |
| hal_iot_gpio_ex.c | GPIO 扩展硬件抽象层 |

---

## 5.4 led_demo

### 5.4.1 功能说明

LED 控制实验，演示 GPIO 控制LED。

---

## 5.5 mqtt_demo

### 5.5.1 功能说明

MQTT 通信实验，演示如何使用 MQTT 协议连接云端。

---

## 5.6 coap_demo

### 5.6.1 功能说明

CoAP 通信实验，演示如何使用 CoAP 协议进行轻量级通信。

---

## 5.7 mutex_demo

### 5.7.1 功能说明

互斥锁使用 Demo，演示多任务环境下的资源保护。

**位置**：`hispark_pegasus/demo/mutex_demo/`

**关键文件**：
- mutex.c

---

## 5.8 semaphore_demo

### 5.8.1 功能说明

信号量使用 Demo，演示任务同步机制。

**位置**：`hispark_pegasus/demo/semaphore_demo/`

**关键文件**：
- semp.c

---

## 5.9 Demo 构建与运行

### 5.9.1 编译方法

```bash
# 进入 OpenHarmony 源码根目录
# 修改 product 配置文件，将应用路径指向 Demo
# 例如修改 build/lite/product/wifiiot.json：
# 将 "//applications/sample/wifi-iot/app" 替换为 "easy_wifi:app"

# 执行编译
python build.py wifiiot
```

### 5.9.2 运行方法

1. 烧录固件到开发板
2. 串口连接查看日志
3. 观察 Demo 输出

---

## 5.10 相关文档

- [WiFi Demo 详解](./05a_WiFi_Demo.md)
- [传感器 Demo](./05b_Sensor_Demo.md)
- [构建指南](./06_Build.md)
