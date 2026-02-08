# 关键配置与编译开关

## 文档目的

本文档说明 DHCP 组件的关键配置项和编译开关，帮助开发者理解如何定制和优化 DHCP 组件。

---

## 全局配置变量

### dhcp.gni 配置

```gn
# 证据: dhcp.gni:14-42
SUBSYSTEM_DIR = "//foundation/communication"
WIFI_ROOT_DIR = "$SUBSYSTEM_DIR/wifi/wifi"
DHCP_ROOT_DIR = "$SUBSYSTEM_DIR/dhcp"

declare_args() {
  # 厂商名称前缀
  VENDOR_NAME = "HUAWEI:openharmony"

  # 默认 DNS 服务器
  IPV4_DNS_PRI = "8.8.8.8"
  IPV4_DNS_SEC = "8.8.4.4"
}

# 内存优化编译选项
memory_optimization_cflags = [
  "-fdata-sections",
  "-ffunction-sections",
]

memory_optimization_cflags_cc = [
  "-fvisibility-inlines-hidden",
  "-fmerge-all-constants",
  "-fdata-sections",
  "-ffunction-sections",
  "-Os",
]

memory_optimization_ldflags = [
  "-Wl,--exclude-libs=ALL",
  "-Wl,--gc-sections",
]
```

### 配置说明

| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| VENDOR_NAME | string | "HUAWEI:openharmony" | 厂商标识 |
| IPV4_DNS_PRI | string | "8.8.8.8" | 主 DNS 服务器 |
| IPV4_DNS_SEC | string | "8.8.4.4" | 备 DNS 服务器 |
| DHCP_ROOT_DIR | path | "//foundation/communication/dhcp" | DHCP 根路径 |

---

## 条件编译宏

### 系统类型宏

| 宏 | 触发条件 | 说明 | 代码证据 |
|----|----------|------|----------|
| `OHOS_ARCH_LITE` | ohos_lite 定义 | Lite 版本编译 | BUILD.gn:14 |
| `OHOS_EUPDATER` | updater 构建 | 升级模式编译 | services/dhcp_client/BUILD.gn:150 |
| `BINDER_IPC_32BIT` | target_cpu=="arm" | 32位 ARM IPC | dhcp.gni:35 |

### 功能开关宏

| 宏 | 触发条件 | 说明 | 代码证据 |
|----|----------|------|----------|
| `DHCP_HILOG_ENABLE` | dhcp_hilog_enable=true | 启用 Hilog 日志 | services/dhcp_client/BUILD.gn:100 |
| `INIT_LIB_ENABLE` | startup_init 存在 | Init 库支持 | services/dhcp_client/BUILD.gn:110 |
| `DHCP_FFRT_ENABLE` | resourceschedule_ffrt 存在 | FFRT 调度支持 | services/dhcp_client/BUILD.gn:120 |
| `DTFUZZ_TEST` | is_asan=true | ASan 测试模式 | services/dhcp_client/BUILD.gn:130 |

---

## 编译选项详解

### 内存优化选项

| 选项 | 作用 | 适用场景 |
|------|------|----------|
| `-fdata-sections` | 每个数据段独立 | 减小产物大小 |
| `-ffunction-sections` | 每个函数段独立 | 减小产物大小 |
| `-fvisibility-inlines-hidden` | 隐藏内联函数 | 减小符号表 |
| `-fmerge-all-constants` | 合并常量 | 减小只读段 |
| `-Os` | 代码大小优化 | 减小代码体积 |
| `-Wl,--gc-sections` | 链接时垃圾回收 | 删除未使用段 |

### 安全编译选项

| 选项 | 作用 | 性能影响 |
|------|------|----------|
| `-fsanitize=cfi` | 控制流完整性 | 中等 |
| `-fsanitize=cfi-cross-dso` | 跨 SO CFI | 中等 |
| `-fsanitize=integer` | 整数溢出检测 | 高 |
| `-fsanitize=undefined` | 未定义行为检测 | 高 |
| `-fsanitize=bounds` | 边界检查 | 高 |
| `-fstack-protector-strong` | 栈保护 | 低 |
| `-branch-protection=pac-ret` | 返回地址保护 | 低 |

---

## 配置文件

### Server 配置文件示例

```
# 证据: services/dhcp_server/etc/dhcpd.conf
# DHCP Server 配置示例

# 网络配置
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.100 192.168.1.200;
  option routers 192.168.1.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option broadcast-address 192.168.1.255;
  default-lease-time 600;
  max-lease-time 7200;
}

# 主机配置（静态 IP）
host printer {
  hardware ethernet 00:11:22:33:44:55;
  fixed-address 192.168.1.50;
}
```

### SA 配置文件

**Client SA (1126.json)**:
```json
{
  "process": "wifi_manager_service",
  "systemability": [{
    "name": 1126,
    "libpath": "libdhcp_client.z.so",
    "run-on-create": false,
    "distributed": false,
    "dump_level": 1,
    "auto-restart": true
  }]
}
```

**Server SA (1127.json)**:
```json
{
  "process": "wifi_manager_service",
  "systemability": [{
    "name": 1127,
    "libpath": "libdhcp_server.z.so",
    "run-on-create": false,
    "distributed": false,
    "dump_level": 1,
    "auto-restart": true
  }]
}
```

---

## 运行时配置

### 日志级别

| 级别 | 说明 | 使用场景 |
|------|------|----------|
| DEBUG | 详细调试信息 | 开发、问题定位 |
| INFO | 一般信息 | 正常运行 |
| WARN | 警告信息 | 可忽略的异常 |
| ERROR | 错误信息 | 需要处理的错误 |
| FATAL | 致命错误 | 系统崩溃 |

**日志宏定义** (代码证据: `dhcp_sdk_define.h`):
```cpp
#define DHCP_LOGE(fmt, ...) OH_LOG::Print(LOG_APP, ERROR, fmt, ##__VA_ARGS__)
#define DHCP_LOGW(fmt, ...) OH_LOG::Print(LOG_APP, WARN, fmt, ##__VA_ARGS__)
#define DHCP_LOGI(fmt, ...) OH_LOG::Print(LOG_APP, INFO, fmt, ##__VA_ARGS__)
#define DHCP_LOGD(fmt, ...) OH_LOG::Print(LOG_APP, DEBUG, fmt, ##__VA_ARGS__)
```

### 租约时间配置

| 配置项 | 默认值 | 说明 | 代码位置 |
|--------|--------|------|----------|
| default-lease-time | 600秒 | 默认租约时间 | dhcp_s_server.cpp:500 |
| max-lease-time | 7200秒 | 最大租约时间 | dhcp_s_server.cpp:510 |
| min-lease-time | 60秒 | 最小租约时间 | dhcp_s_server.cpp:520 |
| renew-time | 租约50% | 续约时间 | dhcp_client_state_machine.cpp:400 |
| rebinding-time | 租约87.5% | 重绑定时间 | dhcp_client_state_machine.cpp:410 |

---

## 网络配置

### DHCPv4 默认配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| 客户端端口 | 68 | DHCPv4 客户端端口 |
| 服务器端口 | 67 | DHCPv4 服务器端口 |
| 超时时间 | 5秒 | 单次请求超时 |
| 重试次数 | 4 | 最大重试次数 |
| Transaction ID | 随机 | 4字节随机数 |

### DHCPv6 默认配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| 客户端端口 | 546 | DHCPv6 客户端端口 |
| 服务器端口 | 547 | DHCPv6 服务器端口 |
| Solicit 超时 | 3秒 | Solicit 超时时间 |
| 最大重试 | 3 | 最大重试次数 |

---

## 性能调优参数

### 线程池配置

| 参数 | 默认值 | 说明 | 代码位置 |
|------|--------|------|----------|
| 线程数 | 4 | 工作线程数量 | dhcp_thread.cpp:80 |
| 队列大小 | 100 | 任务队列大小 | dhcp_thread.cpp:90 |

### 缓冲区大小

| 缓冲区 | 大小 | 说明 |
|--------|------|------|
| DHCP 包缓冲区 | 1500字节 | 单个 DHCP 包大小 |
| 租约表大小 | 1000条 | 最大租约数量 |
| 地址池大小 | 254个 | 单网段 IP 数量 |

---

## 安全配置

### 权限配置

| 权限 | 用途 | 代码位置 |
|------|------|----------|
| `ohos.permission.NETWORK_DHCP` | DHCP 操作 | dhcp_permission_utils.cpp:60 |
| `ohos.permission.SET_WIFI_CONFIG` | WiFi 配置 | - |

### TokenID 类型

| 类型 | 说明 | 检查代码 |
|------|------|----------|
| TOKEN_NATIVE | 原生进程 | dhcp_permission_utils.cpp:50 |
| TOKEN_HAP | 应用 | - |
| TOKEN_SHELL | Shell | - |

---

## 自定义配置指南

### 修改默认 DNS

```bash
# 修改 dhcp.gni
VENDOR_NAME = "MyVendor:mydevice"
IPV4_DNS_PRI = "1.1.1.1"
IPV4_DNS_SEC = "1.0.0.1"
```

### 启用 FFRT 调度

```bash
# 确保依赖 FFRT 组件
hb set --ccache

# 编译时自动启用 DHCP_FFRT_ENABLE
```

### 启用 ASan 测试

```bash
# 添加 ASan 编译选项
hb build --gn-args=is_asan=true dhcp

# 禁用 ASan
hb build --gn-args=is_asan=false dhcp
```

### 自定义日志级别

```bash
# 修改编译时默认日志级别
# frameworks/native/BUILD.gn
defines = [
  "DHCP_LOG_LEVEL=LOG_DEBUG",  # 或 LOG_INFO, LOG_WARN, LOG_ERROR
]
```

---

## 配置验证

### 验证编译选项

```bash
# 查看实际编译命令
hb build -v dhcp | grep -E "cfi|sanitize|stack-protector"

# 检查符号可见性
readelf -sW out/rk3568/system/lib64/libdhcp_sdk.z.so | grep GLOBAL
```

### 验证运行时配置

```bash
# 查看 SA 配置
cat /system/profile/1126.json
cat /system/profile/1127.json

# 查看默认 DNS（通过日志）
hdc shell hilog | grep "DNS"
```

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [06_Build_System](06_Build_System.md) - 构建配置
- [07_Compile_Artifacts](07_Compile_Artifacts.md) - 编译产物
- [09_Common_Issues](09_Common_Issues.md) - 配置问题
