# CUPS 构建适配详解

## 构建系统概述

CUPS 使用 GN (Generate Ninja) 构建系统适配 OpenHarmony，与上游的 Autotools 构建系统完全分离。

---

## 配置文件结构

```
third_party/cups/
├── BUILD.gn              # 主构建配置 (692 行)
├── cups.gni              # GN 导入配置
└── install.py           # 源文件生成脚本
```

### cups.gni - GN 配置参数

```gni
# 路径配置
CUPS_SERVICE_DATA_DIR = "/data/service/el1/public/print_service/cups"
cups_bin_dir = "bin"
cups_sbin_dir = "bin"
cups_serverbin_dir = "bin/cups"
init_service_cfg_path = "etc/init"

# 编译宏定义
cups_defines = [
  "CUPS_BINDIR = \"/system/$cups_bin_dir\"",
  "CUPS_SBINDIR = \"/system/$cups_sbin_dir\"",
  "CUPS_SERVERBIN = \"$CUPS_SERVICE_DATA_DIR/serverbin\"",
  "UNI_PRINT_DRIVER_BINDIR = \"/system/bin/uni_print_driver/ghostscript/bin\"",
  "CUPS_DATADIR = \"$CUPS_SERVICE_DATA_DIR/datadir\"",
  "CUPS_DOCROOT = \"$CUPS_SERVICE_DATA_DIR/doc\"",
  "CUPS_LOCALEDIR = \"$CUPS_SERVICE_DATA_DIR/locale\"",
  "CUPS_LOGDIR = \"$CUPS_SERVICE_DATA_DIR/log\"",
  "CUPS_SERVERROOT = \"$CUPS_SERVICE_DATA_DIR\"",
  "CUPS_CACHEDIR = \"$CUPS_SERVICE_DATA_DIR/cache\"",
  "CUPS_REQUESTS = \"$CUPS_SERVICE_DATA_DIR/spool\"",
  "CUPS_STATEDIR = \"$CUPS_SERVICE_DATA_DIR/run\"",
]

# 功能开关
declare_args() {
  cups_feature_pstops_filter = false
  cups_feature_virtual_printer = false
}

enable_cups_lpd_backend = true
enable_cups_socket_backend = true
```

---

## BUILD.gn 目标结构

### 核心库目标

| 目标 | 类型 | 说明 |
|-----|------|------|
| `cups` | ohos_shared_library | CUPS 核心共享库 |
| `cupsimage` | ohos_shared_library | 图像处理库 |
| `cupsppdc` | ohos_shared_library | PPD 编译器库 |
| `cupsmime` | ohos_shared_library | MIME 类型库 |

### 后端目标

| 目标 | 类型 | 说明 | 可选 |
|-----|------|------|-----|
| `backend` | ohos_shared_library | 通用后端库 | ❌ |
| `ipp` | ohos_executable | IPP 后端 | ❌ |
| `usb` | ohos_executable | USB 打印后端 | ❌ |
| `lpd` | ohos_executable | LPD 后端 | ✅ |
| `socket` | ohos_executable | Socket 后端 | ✅ |
| `virtual-printer` | ohos_executable | 虚拟打印机 | ✅ |

### 守护进程目标

| 目标 | 类型 | 说明 |
|-----|------|------|
| `cupsd` | ohos_executable | CUPS 调度守护进程 |
| `cups-deviced` | ohos_executable | 设备管理 |
| `cups-driverd` | ohos_executable | 驱动管理 |
| `cups-exec` | ohos_executable | 作业执行 |

### 工具目标

| 目标 | 类型 | 说明 |
|-----|------|------|
| `lp` | ohos_executable | 打印命令 |
| `lpadmin` | ohos_executable | 打印机管理 |
| `lpinfo` | ohos_executable | 设备信息 |
| `rastertopwg` | ohos_executable | 栅格转换 |
| `pstops` | ohos_executable | PS 过滤 |
| `ppdc` | ohos_executable | PPD 编译器 |

### 配置文件目标

| 目标 | 类型 | 说明 |
|-----|------|------|
| `mime.convs` | ohos_prebuilt_etc | MIME 转换配置 |
| `mime.types` | ohos_prebuilt_etc | MIME 类型配置 |
| `virtual_printer_ppd` | ohos_prebuilt_etc | 虚拟打印机 PPD |

---

## 编译配置详解

### cups_config

```gn
config("cups_config") {
  defines = cups_defines
  include_dirs = [
    "$config_dir",
    "$cups_code_dir",
    "$core_code_dir",
    get_label_info(":cups_action", "target_gen_dir") + "/cups-2.4.14",
  ]

  cflags = [
    "-Wno-unused-function",
    "-Wno-unused-value",
    "-Wno-implicit-function-declaration",
    "-Wno-int-conversion",
    "-D_FORTIFY_SOURCE=2",
    "-fstack-protector-all",
    "-fdata-sections",
    "-ffunction-sections",
    "-fno-asynchronous-unwind-tables",
    "-fno-unwind-tables",
    "-Os",
  ]
}
```

**编译器标志说明**：

| 标志 | 作用 |
|-----|------|
| `-D_FORTIFY_SOURCE=2` | 运行时缓冲区溢出检测 |
| `-fstack-protector-all` | 全栈保护 |
| `-fdata-sections` | 数据分段 |
| `-ffunction-sections` | 函数分段 |
| `-Os` | 优化代码大小 |

---

## 外部依赖

### 依赖列表

```gn
external_deps = [
  "openssl:libcrypto_shared",   # 加密库
  "openssl:libssl_shared",       # SSL/TLS
  "zlib:libz",                   # 压缩
  "c_utils:utils",               # C 工具
  "drivers_interface_usb:libusb_proxy_1.0",  # USB 代理
  "hilog:libhilog",             # 日志系统
  "libusb:libusb",              # USB 访问
  "usb_manager:usbsrv_client",  # USB 管理
  "ipc:ipc_core",              # 进程通信
  "ipc:ipc_single",            # 单例通信
]
```

### 各目标依赖配置

| 目标 | 外部依赖 |
|-----|---------|
| `cups` | openssl, zlib |
| `cupsimage` | cups |
| `backend` | openssl |
| `ipp` | openssl, backend |
| `usb` | openssl, libusb, hilog, usb_manager, ipc |
| `lpadmin` | - |
| `cupsd` | hilog, openssl |
| `cupsfilter` | openssl |

---

## Patch 应用机制

### cups_action 目标

```gn
action("cups_action") {
  script = "//third_party/cups/install.py"
  outputs = cups_generated_sources

  inputs = [
    "//third_party/cups/ohos-multi-file-print.patch",
    "//third_party/cups/ohos-usb-manager.patch",
    "//third_party/cups/ohos-usb-print.patch",
    "//third_party/cups/ohos-hilog-print.patch",
    "//third_party/cups/cups-log-datamasking.patch",
    "//third_party/cups/backport-CVE-*.patch",
    # ... 更多 patch
  ]

  # 排除 OH 特有文件（不参与 patch）
  inputs -= [
    "//third_party/cups/scheduler/hilog-helper.c",
    "//third_party/cups/backend/usb_manager.cxx",
    "//third_party/cups/backend/usb_ipp_manager.cpp",
    "//third_party/cups/backend/usb_monitor.cpp",
    "//third_party/cups/conf/mime.convs",
    "//third_party/cups/backend/virtual-printer.cpp",
    "//third_party/cups/backend/virtual-printer.ppd",
    "//third_party/cups/scheduler/datamasking.c",
  ]
}
```

### install.py 工作流程

```python
# install.py 伪代码

def main():
    # 1. 解压上游源码
    extract_source("cups-2.4.14-source.tar.gz")

    # 2. 应用 OH 特有 Patch
    for patch in ohos_patches:
        apply_patch(patch)

    # 3. 复制 OH 特有文件
    for oh_file in oh_特有文件:
        copy_to_build(oh_file)

    # 4. 生成构建所需的源文件列表
    generate_source_list()
```

---

## 安装目录结构

构建后安装到设备的目录结构：

```
/system/
├── bin/
│   ├── lp                    # 打印命令
│   ├── lpadmin              # 管理命令
│   ├── lpinfo               # 信息命令
│   ├── rastertopwg         # 栅格转换
│   └── cups/
│       ├── filter/
│       │   ├── imagetopdf
│       │   ├── imagetoraster
│       │   ├── pstops (可选)
│       │   └── cupsfilter
│       ├── backend/
│       │   ├── ipp
│       │   ├── usb
│       │   ├── lpd (可选)
│       │   ├── socket (可选)
│       │   └── virtual-printer (可选)
│       ├── daemon/
│       │   ├── cupsd
│       │   ├── cups-deviced
│       │   ├── cups-driverd
│       │   └── cups-exec
│       └── bin/ (symlink)
└── bin/cups/ (实际安装路径)
    └── serverbin/

/data/service/el1/public/print_service/cups/
├── serverbin/        # 后端程序
├── datadir/          # 数据文件
│   └── share/
│       ├── mime/
│       │   ├── mime.convs
│       │   └── mime.types
│       └── model/
│           └── virtual-printer.ppd
├── doc/             # 文档
├── locale/          # 本地化
├── log/             # 日志
├── cache/           # 缓存
├── spool/           # 打印队列
└── run/             # 运行时状态
```

---

## 功能开关配置

### 全局配置 (print.gni)

```gn
# base/print/print_fwk/print.gni
cups_enable = true  # 全局 CUPS 开关
```

### CUPS 内部配置 (cups.gni)

```gni
# 是否启用 PSTOPS 过滤器
cups_feature_pstops_filter = false

# 是否启用虚拟打印机
cups_feature_virtual_printer = false

# 是否启用 LPD 后端
enable_cups_lpd_backend = true

# 是否启用 Socket 后端
enable_cups_socket_backend = true
```

---

## 与上游构建差异

| 特性 | 上游 Autotools | OH GN |
|-----|---------------|-------|
| **构建系统** | ./configure + make | gn + ninja |
| **配置方式** | configure 参数 | declare_args + 条件编译 |
| **Patch 应用** | make patch | install.py 脚本 |
| **安装路径** | /usr/bin, /etc/cups | /system/bin, /data/service |
| **日志系统** | syslog | HiLog |
| **USB 支持** | usblp 内核模块 | OH USB 服务 |

---

## 相关文档

- [02_Patches.md](02_Patches.md) - Patch 详解
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用情况
- [06_Security.md](06_Security.md) - 安全分析
