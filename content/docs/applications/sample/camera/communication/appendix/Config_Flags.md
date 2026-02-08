# 关键宏/Feature Flags

> 项目中使用的宏定义和配置选项

---

## 目的

本文档记录 `@ohos/camera_sample_communication` 项目中使用的关键宏和配置选项，帮助开发者理解和修改项目配置。

## 适用范围

本文档适用于：
- 需要了解配置选项的开发者
- 准备修改配置的工程师
- 需要添加新功能的开发人员

---

## 宏定义列表

### hostapd 宏定义

**文件**: `hostapd/src/hostapd_sample.c`

| 宏 | 定义位置 | 说明 |
|----|---------|------|
| 无自定义宏 | - | hostapd 模块无自定义宏定义 |

**说明**: hostapd 模块代码简单，无自定义宏定义。

---

### wpa_supplicant 宏定义

**文件**: `wpa_supplicant/src/wpa_sample.c`

| 宏 | 定义位置 | 说明 |
|----|---------|------|
| 无自定义宏 | - | wpa_supplicant 模块无自定义宏定义 |

**说明**: wpa_supplicant 模块代码简单，无自定义宏定义。

---

### wpa_cli 宏定义

**文件**: `wpa_cli/src/wpa_cli_sample.c`

| 宏 | 定义位置 | 值 | 说明 |
|----|---------|-----|------|
| `WPA_IFACE_NAME` | 22 | `"wlan0"` | WiFi 网络接口名称 |
| `WIFI_AUTH_FAILED_REASON_STR` | 23 | `"WRONG_KEY"` | 认证失败原因字符串 |
| `WIFI_AUTH_FAILED_REASON_CODE` | 24 | `"reason=2"` | 认证失败原因代码 |
| `WPA_CTRL_REQUEST_OK` | 25 | `"OK"` | 请求成功响应 |
| `WPA_CTRL_REQUEST_FAIL` | 26 | `"FAIL"` | 请求失败响应 |

**代码证据**:
```c
#define WPA_IFACE_NAME "wlan0"
#define WIFI_AUTH_FAILED_REASON_STR "WRONG_KEY"
#define WIFI_AUTH_FAILED_REASON_CODE "reason=2"
#define WPA_CTRL_REQUEST_OK "OK"
#define WPA_CTRL_REQUEST_FAIL "FAIL"
```

---

## 配置文件选项

### hostapd.conf 配置选项

**文件**: `hostapd/config/hostapd.conf`

| 选项 | 值 | 说明 | 可修改 |
|------|-----|------|--------|
| `interface` | `wlan0` | WiFi 网络接口 | 是 |
| `driver` | `hdf wifi` | WiFi 驱动类型 | 否 |
| `ctrl_interface` | `udp` | 控制接口类型 | 否 |
| `ssid` | `testap` | 接入点 SSID | 是 |
| `hw_mode` | `g` | 802.11 模式 | 是 |
| `channel` | `1` | 无线信道 | 是 |
| `ignore_broadcast_ssid` | `0` | 是否广播 SSID | 是 |

**代码证据**:
```
interface=wlan0
driver=hdf wifi
ctrl_interface=udp
ssid=testap
hw_mode=g
channel=1
ignore_broadcast_ssid=0
```

---

### wpa_supplicant.conf 配置选项

**文件**: `wpa_supplicant/config/wpa_supplicant.conf`

| 选项 | 值 | 说明 | 可修改 |
|------|-----|------|--------|
| `country` | `GB` | 国家代码 | 是 |
| `ctrl_interface` | `udp` | 控制接口类型 | 否 |
| `network` | `{}` | 网络配置块 | 是 |

**代码证据**:
```
country=GB
ctrl_interface=udp
network={
}
```

---

## GN 构建配置

### 全局变量

| 变量 | 文件 | 类型 | 值 | 说明 |
|------|------|------|-----|------|
| `sample_sources` | hostapd/BUILD.gn | 列表 | `["src/hostapd_sample.c"]` | hostapd 源文件 |
| `sample_sources` | wpa_supplicant/BUILD.gn | 列表 | `["src/wpa_sample.c"]` | wpa_supplicant 源文件 |
| `sample_sources` | wpa_cli/BUILD.gn | 列表 | `["src/wpa_cli_sample.c"]` | wpa_cli 源文件 |
| `sample_include_dirs` | wpa_cli/BUILD.gn | 列表 | 见下表 | wpa_cli 头文件路径 |

### wpa_cli 头文件路径

| 路径 | 说明 |
|------|------|
| `//third_party/wpa_supplicant/wpa_supplicant-2.9/src/` | wpa_supplicant 源码目录 |
| `//third_party/bounds_checking_function:libsec_shared/include/` | 安全函数库头文件 |

---

## Feature Flags

### 当前状态

本项目**不包含** Feature Flags（功能开关）。

所有功能都是无条件编译的。

### 潜在 Feature Flags

如需添加 Feature Flags，可以在 BUILD.gn 中添加 `defines` 或 `config`：

```gn
# 添加编译时宏
config("debug_config") {
    defines = [
        "ENABLE_DEBUG_LOG=1",
        "MAX_NETWORKS=10",
    ]
}

executable("wpa_cli_exe") {
    configs = [":debug_config"]
    # ...
}

# 添加功能开关
declare_args() {
    enable_wifi_debug = false  # 是否启用 WiFi 调试
}

config("feature_config") {
    if (enable_wifi_debug) {
        defines = [ "ENABLE_WIFI_DEBUG" ]
    }
}
```

---

## 编译时选项

### 当前配置

| 选项 | 值 | 说明 |
|------|-----|------|
| 优化级别 | 默认 | 未指定 |
| 调试信息 | 未启用 | 未添加调试符号 |
| 静态链接 | 否 | 使用动态链接 |

### 添加编译选项

可以在 BUILD.gn 中添加编译选项：

```gn
executable("my_exe") {
    cflags = [
        "-O2",              # 优化级别
        "-g",               # 生成调试信息
        "-Wall",            # 启用所有警告
    ]
    defines = [
        "ENABLE_FEATURE_X",  # 定义宏
    ]
}
```

---

## 运行时配置

### 接口名称配置

**宏**: `WPA_IFACE_NAME`

**默认值**: `"wlan0"`

**位置**: `wpa_cli/src/wpa_cli_sample.c:22`

**修改方法**: 修改宏定义或使用配置文件

```c
// 修改宏定义
#define WPA_IFACE_NAME "wlan1"

// 或从配置文件读取
char iface_name[32];
GetConfig("interface", iface_name, sizeof(iface_name));
wpa_ctrl_open(iface_name);
```

---

## 系统常量

### WiFi 事件常量

| 常量 | 来源 | 说明 |
|------|------|------|
| `WPA_EVENT_CONNECTED` | wpa_ctrl.h | 连接成功 |
| `WPA_EVENT_SCAN_RESULTS` | wpa_ctrl.h | 扫描完成 |
| `WPA_EVENT_TEMP_DISABLED` | wpa_ctrl.h | 认证失败 |
| `WPA_EVENT_DISCONNECTED` | wpa_ctrl.h | 断开连接 |

### wpa_ctrl 响应常量

| 常量 | 定义位置 | 值 | 说明 |
|------|---------|-----|------|
| `WPA_CTRL_REQUEST_OK` | wpa_cli_sample.c:25 | `"OK"` | 请求成功 |
| `WPA_CTRL_REQUEST_FAIL` | wpa_cli_sample.c:26 | `"FAIL"` | 请求失败 |

---

## 配置修改建议

### 修改接口名称

如需修改 WiFi 接口名称，可以：

1. **修改宏定义**:
   ```c
   #define WPA_IFACE_NAME "wlan1"
   ```

2. **使用配置文件**:
   ```c
   char iface_name[32] = "wlan0";  // 从配置文件读取
   wpa_ctrl_open(iface_name);
   ```

### 添加调试宏

如需添加调试功能，可以：

1. **添加宏定义**:
   ```c
   #ifdef ENABLE_DEBUG_LOG
   #define DEBUG_LOG(fmt, ...) printf("[DEBUG] " fmt "\n", ##__VA_ARGS__)
   #else
   #define DEBUG_LOG(fmt, ...)
   #endif
   ```

2. **在 BUILD.gn 中启用**:
   ```gn
   defines = [ "ENABLE_DEBUG_LOG" ]
   ```

### 添加功能开关

如需添加功能开关，可以：

1. **在 BUILD.gn 中添加**:
   ```gn
   declare_args() {
       enable_wifi_debug = false
       enable_wifi_scan_cache = false
   }
   ```

2. **在代码中使用**:
   ```c
   #ifdef ENABLE_WIFI_DEBUG
   printf("Debug info\n");
   #endif
   ```

---

## 配置验证

### 验证配置文件

```bash
# 验证 hostapd 配置
hostapd -dd /etc/hostapd.conf

# 验证 wpa_supplicant 配置
wpa_supplicant -dd -i wlan0 -c /etc/wpa_supplicant.conf
```

### 验证宏定义

```bash
# 查看预定义宏
gcc -E -dM hostapd/src/hostapd_sample.c | grep WPA_IFACE_NAME
```

---

## 常见问题

### 问题 1：修改宏后需要重新编译

**原因**: 宏定义在编译时展开，修改后需要重新编译。

**解决方案**:
```bash
# 清理构建产物
hb clean

# 重新编译
hb build -f //applications/sample/camera/communication/wpa_cli
```

### 问题 2：配置文件修改不生效

**原因**: 配置文件可能被复制到其他位置，或使用了旧配置。

**解决方案**:
```bash
# 确认配置文件位置
ls -l /etc/hostapd.conf

# 重启服务
killall hostapd
hostapd /etc/hostapd.conf
```

### 问题 3：接口名称错误

**原因**: `WPA_IFACE_NAME` 与实际接口名称不匹配。

**解决方案**:
```bash
# 查看实际接口名称
ifconfig -a

# 修改宏定义
#define WPA_IFACE_NAME "实际接口名"
```

---

## 扩展建议

### 添加新配置选项

如需添加新的配置选项，可以：

1. **在配置文件中添加**:
   ```
   my_option=value
   ```

2. **在代码中解析**:
   ```c
   char my_option[32];
   ParseConfig("my_option", my_option, sizeof(my_option));
   ```

3. **添加宏定义**:
   ```c
   #define DEFAULT_MY_OPTION "default_value"
   ```

### 添加编译选项

如需添加新的编译选项，可以在 BUILD.gn 中添加：

```gn
config("my_config") {
    cflags = [
        "-O2",
        "-Wall",
    ]
    defines = [
        "MY_MACRO=1",
    ]
}

executable("my_exe") {
    configs = [":my_config"]
}
```

---

## 参考文档

- [wpa_supplicant 配置文档](https://w1.fi/wpa_supplicant/)
- [hostapd 配置文档](https://w1.fi/hostapd/)
- [GN 构建系统文档](https://gn.googlesource.com/gn/)
- [GCC 预定义宏](https://gcc.gnu.org/onlinedocs/cpp/Predefined-Macros.html)
