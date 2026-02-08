# OH 构建适配说明

## 概述

OpenHarmony 使用 **GN (Generate Ninja)** 构建系统，与 lwIP 上游使用的 Makefile/CMake 不同。本章说明 lwIP 在 OH 中的构建配置和适配方式。

## 构建文件结构

```
lwip/
├── BUILD.gn          # 主构建配置：定义 liblwip 目标
└── lwip.gni          # 源文件列表：定义各种配置下的源文件集合
```

## BUILD.gn 详解

### 目标定义

```gn
ohos_shared_library("liblwip") {
  sources = [
    # 核心协议栈源文件列表
    "src/core/altcp_alloc.c",
    "src/core/altcp.c",
    ...
    "src/core/lowpower.c",      # OH: 低功耗功能
    "src/core/net_group.c",     # OH: 网络容器功能
  ]
  
  configs = [ ":libext2fs-defaults" ]  # 编译配置
  deps = [ ":libext2_com_err" ]        # 依赖项
  
  cflags = [ "-Wno-unused-parameter" ]  # 忽略未使用参数警告
  include_dirs = [ "//third_party/lwip/include" ]  # 头文件路径
  
  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "lwip"
  install_images = [ "system", "updater" ]  # 安装分区
}
```

### 关键配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `install_enable` | `true` | 编译结果安装到系统镜像 |
| `install_images` | `system`, `updater` | 安装到 system 和 updater 分区 |
| `subsystem_name` | `thirdparty` | 所属子系统 |
| `part_name` | `lwip` | 组件名 |

### 与上游差异

| 方面 | 上游 lwIP | OH lwIP |
|------|----------|---------|
| **构建系统** | Makefile/CMake | GN |
| **目标类型** | 静态库/可配置 | 共享库 (ohos_shared_library) |
| **安装位置** | 用户指定 | system/updater 分区 |
| **应用组件** | 默认编译全部 | 需单独引用 |

## lwip.gni 详解

### 文件作用

`lwip.gni` 定义了源文件集合，供其他模块按需引用 lwIP 功能：

```gn
import("//third_party/lwip/lwip.gni")

# 使用预定义的源文件列表
sources = LWIPNOAPPSFILES  # 不含应用的核心栈
sources += SNMPFILES       # 添加 SNMP
```

### 源文件集合定义

| 变量名 | 内容 | 说明 |
|--------|------|------|
| `LWIPDIR` | `//third_party/lwip/src` | 源代码根目录 |
| `LWIP_INCLUDE_DIRS` | `["$LWIPDIR/include"]` | 头文件路径 |
| `COREFILES` | 15 个文件 | 最小核心功能 |
| `CORE4FILES` | 9 个文件 | IPv4 支持 |
| `CORE6FILES` | 9 个文件 | IPv6 支持 |
| `APIFILES` | 9 个文件 | Socket/Netconn API |
| `NETIFFILES` | 4 个文件 | 网络接口层 |
| `PPPFILES` | 26 个文件 | PPP 协议支持 |
| `SIXLOWPAN` | 4 个文件 | 6LoWPAN 支持 |
| `SNMPFILES` | 18 个文件 | SNMP 代理 |
| `HTTPFILES` | 4 个文件 | HTTP 服务器/客户端 |
| `MQTTFILES` | 1 个文件 | MQTT 客户端 |
| `SNTPFILES` | 1 个文件 | SNTP 客户端 |
| `MDNSFILES` | 1 个文件 | mDNS 响应器 |
| `MBEDTLS_FILES` | 3 个文件 | mbedTLS 集成 |
| `LWIPAPPFILES` | 所有应用 | 完整应用集合 |
| `LWIPNOAPPSFILES` | 核心+网络接口+PPP+6LoWPAN | 不含应用 |

### 使用示例

**自定义模块使用 lwIP**:
```gn
import("//third_party/lwip/lwip.gni")

ohos_static_library("my_network_module") {
  sources = LWIPNOAPPSFILES
  sources -= PPPFILES  # 不需要 PPP
  sources += MQTTFILES  # 需要 MQTT
  
  include_dirs = LWIP_INCLUDE_DIRS
  
  deps = [
    "//third_party/mbedtls:mbedtls",  # 如需 TLS
  ]
}
```

## OH 特有源文件

### 添加到 BUILD.gn 的 OH 文件

```gn
sources = [
  # ... 标准 lwIP 源文件 ...
  
  # OH 特有功能
  "src/core/lowpower.c",      # 低功耗模式
  "src/core/net_group.c",     # 网络容器
]
```

### 条件编译控制

这些 OH 特有文件的编译通过 `lwipopts.h` 中的宏控制：

```c
// lwipopts.h (通常由使用方提供)

// 启用低功耗模式
#define LWIP_LOWPOWER 1

// 启用网络容器 (通常由 kernel 定义 LOSCFG_NET_CONTAINER)
#define LOSCFG_NET_CONTAINER 1

// 启用分布式网络
#define LWIP_ENABLE_DISTRIBUTED_NET 1
```

## 配置选项 (lwipopts.h)

### 常用配置宏

| 宏 | 默认值 | 说明 |
|-----|--------|------|
| `NO_SYS` | 0 | 0=使用 OS，1=裸机 |
| `LWIP_TIMERS` | 1 | 启用内部定时器 |
| `LWIP_TCP` | 1 | 启用 TCP |
| `LWIP_UDP` | 1 | 启用 UDP |
| `LWIP_IPV4` | 1 | 启用 IPv4 |
| `LWIP_IPV6` | 0 | 启用 IPv6 |
| `LWIP_NETCONN_SEM_PER_THREAD` | 0 | 每线程信号量 |

### OH 特有配置宏

| 宏 | 来源 | 说明 |
|-----|------|------|
| `LWIP_LOWPOWER` | `CONFIG_LWIP_LOWPOWER` | 低功耗模式 |
| `LOSCFG_NET_CONTAINER` | kernel 配置 | 网络容器 |
| `LWIP_ENABLE_DISTRIBUTED_NET` | 应用配置 | 分布式网络 |

### 配置继承关系

```
lwip/src/include/lwip/opt.h      (默认配置)
       ↓
    lwipopts.h                   (项目自定义配置)
       ↓
    kernel 配置                  (如 LOSCFG_NET_CONTAINER)
```

## 与其他模块集成

### 内核集成 (LiteOS)

```gn
# kernel/liteos_m/liteos.gni
THIRDPARTY_LWIP_DIR = "//third_party/lwip"
```

### 软总线集成 (DSoftBus)

```gn
# foundation/communication/dsoftbus/adapter/BUILD.gn
include_dirs = [
  "//third_party/lwip/src/include",
]

# 对于 hispark_pegasus 设备
if (defined(hispark_pegasus_sdk_path)) {
  include_dirs += [
    "$hispark_pegasus_sdk_path/third_party/lwip_sack/include",
  ]
}
```

### 第三方库引用

```gn
# 其他模块的 BUILD.gn
import("//third_party/lwip/lwip.gni")

ohos_executable("my_app") {
  sources = [ "main.c" ]
  
  include_dirs = LWIP_INCLUDE_DIRS
  
  deps = [
    "//third_party/lwip:liblwip",
  ]
}
```

## 构建调试

### 检查编译选项

```bash
# 查看编译命令
gn args out/default --list

# 查看特定目标的编译配置
gn desc out/default //third_party/lwip:liblwp
```

### 常见问题

1. **头文件找不到**
   - 确保 `include_dirs` 包含 `LWIP_INCLUDE_DIRS`
   - 检查 `lwipopts.h` 是否存在

2. **未定义引用**
   - 确保链接 `//third_party/lwip:liblwip`
   - 检查 `lwipopts.h` 中启用了相应功能

3. **OH 特有功能未生效**
   - 检查 `CONFIG_LWIP_LOWPOWER` 是否在 kernel 配置中启用
   - 检查 `LOSCFG_NET_CONTAINER` 是否定义
