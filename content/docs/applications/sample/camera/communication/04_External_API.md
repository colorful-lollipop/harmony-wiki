# 对外 API

> 说明项目的对外 API，包括 N-API、导出符号、权限/参数/错误码

---

## 目的

本文档说明 `@ohos/camera_sample_communication` 项目的对外 API，帮助开发者了解如何使用本项目的功能。

## 适用范围

本文档适用于：
- 需要使用本项目 API 的开发者
- 集成本项目的工程师
- 需要理解 API 行为的开发人员

## 关键结论

- **本项目不提供 N-API / JavaScript API**：本项目是纯 C 语言示例程序，不包含 JS 绑定
- **对外接口**：三个独立的可执行程序（hostapd、wpa_supplicant、wpa_cli）
- **控制接口**：wpa_cli 使用 wpa_ctrl 接口（UDP socket）与 wpa_supplicant 通信
- **无权限管理**：示例代码不包含权限检查机制

## 相关跳转

- [内部 API](05_Internal_API.md) - 内部模块接口
- [架构说明](03_Architecture.md) - 组件交互和数据流

---

## N-API / JavaScript API

### 不适用声明 ⚠️

本项目**不提供 N-API 或 JavaScript API**。

**证据**:
- 代码中未发现 `napi_` 函数调用（如 `napi_define_properties`, `napi_create_function`）
- 代码中未发现 `NAPI_MODULE` 或 `napi_module_register` 宏
- 项目中无 TypeScript/JavaScript 类型定义文件（.d.ts）

如需在 OpenHarmony JS 应用中使用 WiFi 功能，请参考 OpenHarmony 官方 WiFi API 文档。

---

## 命令行接口

本项目提供三个独立的可执行程序，通过命令行接口交互。

### hostapd

**可执行文件**: `hostapd`

**用途**: 启动 WiFi 接入点（AP 模式）

**命令行参数**:
```bash
hostapd [配置文件路径]
```

**示例**:
```bash
hostapd /etc/hostapd.conf
```

**配置文件**: `hostapd.conf`

**证据**: `hostapd/config/hostapd.conf:1-8`

---

### wpa_supplicant

**可执行文件**: `wpa_supplicant`

**用途**: 启动 WiFi 客户端（STA 模式）

**命令行参数**:
```bash
wpa_supplicant -B -i <接口> -c <配置文件>
```

**参数说明**:
| 参数 | 说明 |
|------|------|
| `-B` | 后台运行 |
| `-i <接口>` | 指定网络接口（如 wlan0） |
| `-c <配置文件>` | 指定配置文件 |

**示例**:
```bash
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
```

**配置文件**: `wpa_supplicant.conf`

**证据**: `wpa_supplicant/config/wpa_supplicant.conf:1-5`

---

### wpa_cli

**可执行文件**: `wpa_cli`

**用途**: 控制 wpa_supplicant，执行网络扫描、配置、连接等操作

**命令行参数**: 无（示例代码中未处理命令行参数）

**内部功能**:
- 连接测试（PING）
- WiFi 网络扫描
- 网络配置和连接
- WiFi 事件监听

**证据**: `wpa_cli/src/wpa_cli_sample.c:245-255`

---

## wpa_ctrl 控制接口

wpa_cli 通过 wpa_ctrl 接口与 wpa_supplicant 通信。这是本项目的主要对外接口。

### 接口概述

| 特性 | 值 |
|------|-----|
| 传输协议 | UDP Socket |
| 接口类型 | 控制接口 + 监控接口 |
| 通信模式 | 请求/响应 + 事件推送 |
| 协议格式 | 文本协议 |

**证据**:
- 配置文件: `hostapd.conf:3`, `wpa_supplicant.conf:2` (`ctrl_interface=udp`)

### 接口初始化

**函数**: `InitControlInterface()`

**路径**: `wpa_cli/src/wpa_cli_sample.c:230-243`

```c
int InitControlInterface()
{
    g_ctrlConn = wpa_ctrl_open(WPA_IFACE_NAME); // 创建控制接口
    g_monitorConn = wpa_ctrl_open(WPA_IFACE_NAME); // 创建监控接口
    if (!g_ctrlConn || !g_monitorConn) {
        SAMPLE_ERROR("open wpa control interface failed.");
        return -1;
    }
    if (wpa_ctrl_attach(g_monitorConn) == 0) { // 启动监控
        pthread_create(&g_wpaThreadId, NULL, MonitorTask, NULL);
        return 0;
    }
    return -1;
}
```

**返回值**:
- `0` - 成功
- `-1` - 失败

---

### 控制命令

wpa_ctrl 支持的控制命令通过 `SendCtrlCommand()` 函数发送。

**函数**: `SendCtrlCommand()`

**路径**: `wpa_cli/src/wpa_cli_sample.c:127-138`

```c
static int SendCtrlCommand(const char *cmd, char *reply, size_t *replyLen)
{
    size_t len = *replyLen - 1;
    wpa_ctrl_request(g_ctrlConn, cmd, strlen(cmd), reply, &len, 0);
    if (len != 0 && !StrMatch(reply, WPA_CTRL_REQUEST_FAIL)) {
        *replyLen = len;
        return 0;
    }
    SAMPLE_ERROR("send ctrl request [%s] failed.", cmd);
    return -1;
}
```

**参数**:
- `cmd` - 控制命令（字符串）
- `reply` - 接收响应的缓冲区
- `replyLen` - 响应缓冲区长度（输入/输出）

**返回值**:
- `0` - 成功
- `-1` - 失败

#### 常用命令

| 命令 | 说明 | 使用示例 |
|------|------|---------|
| `PING` | 测试连接 | `SendCtrlCommand("PING", reply, &len)` |
| `SCAN` | 扫描 WiFi 网络 | `SendCtrlCommand("SCAN", reply, &len)` |
| `SCAN_RESULTS` | 获取扫描结果 | `SendCtrlCommand("SCAN_RESULTS", reply, &len)` |
| `ADD_NETWORK` | 添加网络配置 | `SendCtrlCommand("ADD_NETWORK", reply, &len)` |
| `SET_NETWORK <id> ssid "<ssid>"` | 设置 SSID | `SendCtrlCommand("SET_NETWORK 0 ssid \"example\"", reply, &len)` |
| `SET_NETWORK <id> psk "<password>"` | 设置密码 | `SendCtrlCommand("SET_NETWORK 0 psk \"password\"", reply, &len)` |
| `ENABLE_NETWORK <id>` | 启用网络 | `SendCtrlCommand("ENABLE_NETWORK 0", reply, &len)` |
| `REMOVE_NETWORK <id>` | 删除网络 | `SendCtrlCommand("REMOVE_NETWORK 0", reply, &len)` |
| `DISCONNECT` | 断开连接 | `SendCtrlCommand("DISCONNECT", reply, &len)` |
| `RECONNECT` | 重新连接 | `SendCtrlCommand("RECONNECT", reply, &len)` |

**证据**: `wpa_cli/src/wpa_cli_sample.c:140-185`

---

### 事件监听

wpa_ctrl 支持推送 WiFi 事件（如连接成功、扫描完成等）。

**事件接收**: `CliRecvPending()`

**路径**: `wpa_cli/src/wpa_cli_sample.c:91-105`

```c
static void CliRecvPending(void)
{
    while (wpa_ctrl_pending(g_monitorConn)) {
        char buf[4096];
        size_t len = sizeof(buf) - 1;
        if (wpa_ctrl_recv(g_monitorConn, buf, &len) == 0) {
            buf[len] = '\0';
            SAMPLE_INFO("event received %s", buf);
            WifiEventHandler(buf, len);
        } else {
            SAMPLE_INFO("could not read pending message.");
            break;
        }
    }
}
```

**事件处理**: `WifiEventHandler()`

**路径**: `wpa_cli/src/wpa_cli_sample.c:61-89`

#### WiFi 事件列表

| 事件 | 宏定义 | 说明 | 触发条件 |
|------|--------|------|---------|
| 连接成功 | `WPA_EVENT_CONNECTED` | WiFi 连接建立成功 | 认证成功，获得 IP |
| 扫描完成 | `WPA_EVENT_SCAN_RESULTS` | WiFi 网络扫描完成 | 扫描操作结束 |
| 认证失败 | `WPA_EVENT_TEMP_DISABLED` | 密码错误等认证失败 | 认证失败 |
| 断开连接 | `WPA_EVENT_DISCONNECTED` | WiFi 连接断开 | 主动断开或连接丢失 |

**证据**: `wpa_cli/src/wpa_cli_sample.c:72, 76, 81, 85`

---

### wpa_ctrl API 函数

| 函数 | 用途 | 路径 |
|------|------|------|
| `wpa_ctrl_open()` | 打开 wpa_ctrl 连接 | `wpa_cli_sample.c:232-233` |
| `wpa_ctrl_attach()` | 附加监控（启用事件推送） | `wpa_cli_sample.c:238` |
| `wpa_ctrl_request()` | 发送控制命令 | `wpa_cli_sample.c:130` |
| `wpa_ctrl_recv()` | 接收事件消息 | `wpa_cli_sample.c:96` |
| `wpa_ctrl_pending()` | 检查待处理消息 | `wpa_cli_sample.c:93` |
| `wpa_ctrl_get_fd()` | 获取文件描述符（用于 select） | `wpa_cli_sample.c:113` |

**注意**: 这些函数来自第三方 wpa_supplicant 库的头文件 `common/wpa_ctrl.h`。

---

## 权限管理

### 不适用声明 ⚠️

本项目**不包含权限管理机制**。

**证据**:
- 代码中未发现 `permission`、`access_token`、`uid/gid` 检查逻辑
- wpa_ctrl 接口无访问控制

### 安全建议

在生产环境中使用时，建议：
1. 使用 Unix 权限控制 wpa_ctrl socket 文件访问
2. 在应用层实现权限检查逻辑
3. 使用 IPC/ServiceAbility 机制替代直接 socket 访问

---

## 错误码

### wpa_ctrl 响应

| 响应 | 含义 | 说明 |
|------|------|------|
| `OK` | 命令执行成功 | 操作完成 |
| `FAIL` | 命令执行失败 | 操作失败 |
| `PONG` | 连接成功 | PING 命令的响应 |

**证据**: `wpa_cli/src/wpa_cli_sample.c:25-26`

```
#define WPA_CTRL_REQUEST_OK "OK"
#define WPA_CTRL_REQUEST_FAIL "FAIL"
```

### 函数返回值

| 函数 | 返回值 | 含义 |
|------|--------|------|
| `InitControlInterface()` | `0` | 成功 |
| `InitControlInterface()` | `-1` | 失败 |
| `SendCtrlCommand()` | `0` | 命令发送成功 |
| `SendCtrlCommand()` | `-1` | 命令发送失败 |

**证据**:
- `wpa_cli/src/wpa_cli_sample.c:230-243`
- `wpa_cli/src/wpa_cli_sample.c:127-138`

### 错误处理

本项目采用简单的错误处理策略：
- 打印错误日志
- 返回错误码
- 程序继续或退出（视严重程度）

**示例**:
```c
if (ret != 0) {
    SAMPLE_ERROR("send ctrl request [%s] failed.", cmd);
    return -1;
}
```

**证据**: `wpa_cli/src/wpa_cli_sample.c:136`

---

## 使用示例

### 示例 1：连接 WiFi 网络

```c
// 初始化控制接口
if (InitControlInterface() != 0) {
    SAMPLE_ERROR("control interface init failed.");
    return -1;
}

// 添加网络配置
char networkId[20] = {0};
size_t len = sizeof(networkId);
SendCtrlCommand("ADD_NETWORK", networkId, &len);

// 设置 SSID 和密码
char reply[100] = {0};
char cmd[200] = {0};
sprintf_s(cmd, sizeof(cmd), "SET_NETWORK %.*s ssid \"MyWiFi\"", len, networkId);
SendCtrlCommand(cmd, reply, &len);

sprintf_s(cmd, sizeof(cmd), "SET_NETWORK %.*s psk \"password123\"", len, networkId);
SendCtrlCommand(cmd, reply, &len);

// 启用网络并连接
sprintf_s(cmd, sizeof(cmd), "ENABLE_NETWORK %.*s", len, networkId);
SendCtrlCommand(cmd, reply, &len);
SendCtrlCommand("RECONNECT", reply, &len);

// 等待连接成功（通过事件监听）
```

### 示例 2：扫描 WiFi 网络

```c
// 发起扫描
char reply[100] = {0};
size_t len = sizeof(reply);
SendCtrlCommand("SCAN", reply, &len);

// 等待扫描完成（通过事件监听）
while (1) {
    sleep(1);
    if (g_scanAvailable == 1) {
        SAMPLE_INFO("scan result received.");
        break;
    }
}

// 获取扫描结果
char scanResult[4096] = {0};
size_t scanLen = sizeof(scanResult);
SendCtrlCommand("SCAN_RESULTS", scanResult, &scanLen);

// 打印扫描结果
printf("Scan results:\n%s\n", scanResult);
```

---

## API 稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| hostapd 命令行 | 稳定 | 标准 wpa_supplicant 接口 |
| wpa_supplicant 命令行 | 稳定 | 标准 wpa_supplicant 接口 |
| wpa_ctrl 控制接口 | 稳定 | 标准 wpa_supplicant 接口 |
| 示例代码函数 | 不稳定 | 仅用于演示，不建议直接使用 |

---

## 扩展建议

1. **封装高级 API**: 将 wpa_ctrl 接口封装为更高级的 API
2. **添加回调机制**: 使用回调函数处理 WiFi 事件
3. **错误处理增强**: 添加更详细的错误码和错误消息
4. **异步操作**: 支持异步命令发送和回调
5. **权限管理**: 添加访问控制机制
