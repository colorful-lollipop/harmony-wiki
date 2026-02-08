# 目录结构与模块职责

> 说明项目的代码组织和各模块的职责

---

## 目的

本文档说明 `@ohos/camera_sample_communication` 项目的目录结构和各模块的职责，帮助读者快速理解代码组织方式。

## 适用范围

本文档适用于：
- 需要了解项目结构的开发者
- 准备修改或扩展功能的工程师
- 需要定位特定功能的开发人员

## 关键结论

- 项目结构简单，包含三个主模块：hostapd、wpa_cli、wpa_supplicant
- 每个模块独立构建，有自己的 BUILD.gn 文件
- 不包含测试目录
- 源代码文件少，每个模块只有一个 C 文件

## 相关跳转

- [项目定位与边界](01_Project_Position.md) - 项目职责和边界
- [架构说明](03_Architecture.md) - 组件交互和数据流

---

## 目录树

```
applications/sample/camera/communication/
├── hostapd/                          # hostapd 接入点模块
│   ├── BUILD.gn                      # GN 构建文件
│   ├── src/
│   │   └── hostapd_sample.c         # hostapd 示例代码
│   └── config/
│       └── hostapd.conf             # hostapd 配置文件
├── wpa_cli/                         # wpa_cli 控制客户端模块
│   ├── BUILD.gn                      # GN 构建文件
│   └── src/
│       └── wpa_cli_sample.c         # wpa_cli 示例代码
├── wpa_supplicant/                  # wpa_supplicant 客户端模块
│   ├── BUILD.gn                      # GN 构建文件
│   ├── src/
│   │   └── wpa_sample.c            # wpa_supplicant 示例代码
│   └── config/
│       └── wpa_supplicant.conf      # wpa_supplicant 配置文件
├── BUILD.gn                         # GN 构建入口
├── bundle.json                      # 组件配置
├── LICENSE                          # Apache 2.0 许可证
├── README.md                        # 项目说明
└── wiki/                            # Wiki 文档目录
    ├── _work/                        # 工作目录
    └── appendix/                     # 附录目录
```

---

## 模块职责

### 顶层文件

| 文件 | 职责 | 类型 |
|------|------|------|
| `BUILD.gn` | GN 构建系统入口文件 | 构建配置 |
| `bundle.json` | 组件配置（名称、依赖、子系统等） | 组件配置 |
| `LICENSE` | Apache 2.0 许可证文件 | 许可证 |
| `README.md` | 项目说明文档 | 文档 |

### hostapd 模块

**路径**: `hostapd/`

**职责**:
- 展示如何启动 hostapd（WiFi 接入点）
- 提供配置文件示例
- 动态加载 libwpa.so 的 ap_main 函数

**文件说明**:

| 文件 | 职责 | 行数 |
|------|------|------|
| `BUILD.gn` | GN 构建配置，定义可执行文件和配置复制 | 41 行 |
| `src/hostapd_sample.c` | hostapd 启动示例代码 | 71 行 |
| `config/hostapd.conf` | hostapd 配置文件（接口、SSID、信道等） | 8 行 |

**关键函数**:

| 函数 | 职责 | 位置 |
|------|------|------|
| `main()` | 程序入口，创建工作线程 | `hostapd_sample.c:56` |
| `ThreadMain()` | 工作线程，加载 libwpa.so 并调用 ap_main | `hostapd_sample.c:26` |

### wpa_supplicant 模块

**路径**: `wpa_supplicant/`

**职责**:
- 展示如何启动 wpa_supplicant（WiFi 客户端）
- 提供配置文件示例
- 动态加载 libwpa.so 的 wpa_main 函数

**文件说明**:

| 文件 | 职责 | 行数 |
|------|------|------|
| `BUILD.gn` | GN 构建配置，定义可执行文件和配置复制 | 41 行 |
| `src/wpa_sample.c` | wpa_supplicant 启动示例代码 | 71 行 |
| `config/wpa_supplicant.conf` | wpa_supplicant 配置文件（国家码、控制接口等） | 5 行 |

**关键函数**:

| 函数 | 职责 | 位置 |
|------|------|------|
| `main()` | 程序入口，创建工作线程 | `wpa_sample.c:56` |
| `ThreadMain()` | 工作线程，加载 libwpa.so 并调用 wpa_main | `wpa_sample.c:26` |

### wpa_cli 模块

**路径**: `wpa_cli/`

**职责**:
- 展示如何使用 wpa_ctrl 接口控制 wpa_supplicant
- 实现网络扫描、配置、连接功能
- 监听并处理 WiFi 事件

**文件说明**:

| 文件 | 职责 | 行数 |
|------|------|------|
| `BUILD.gn` | GN 构建配置，定义可执行文件和依赖 | 45 行 |
| `src/wpa_cli_sample.c` | wpa_cli 控制示例代码（功能最完整） | 256 行 |

**关键函数**:

| 函数 | 职责 | 位置 |
|------|------|------|
| `main()` | 程序入口，初始化控制接口并运行测试 | `wpa_cli_sample.c:245` |
| `InitControlInterface()` | 初始化 wpa_ctrl 控制和监控接口 | `wpa_cli_sample.c:230` |
| `MonitorTask()` | 监控任务线程，监听 WiFi 事件 | `wpa_cli_sample.c:107` |
| `CliRecvPending()` | 接收待处理的事件消息 | `wpa_cli_sample.c:91` |
| `WifiEventHandler()` | 处理 WiFi 事件（连接、扫描、断开等） | `wpa_cli_sample.c:61` |
| `SendCtrlCommand()` | 发送控制命令到 wpa_supplicant | `wpa_cli_sample.c:127` |
| `TestCliConnection()` | 测试与 wpa_supplicant 的连接 | `wpa_cli_sample.c:187` |
| `TestScan()` | 测试 WiFi 网络扫描功能 | `wpa_cli_sample.c:199` |
| `TestNetworkConfig()` | 测试网络配置和连接功能 | `wpa_cli_sample.c:140` |
| `StartTest()` | 执行所有测试用例 | `wpa_cli_sample.c:223` |

---

## 配置文件

### hostapd.conf

**路径**: `hostapd/config/hostapd.conf`

**用途**: hostapd 接入点配置

**配置项**:

| 配置 | 值 | 说明 |
|------|-----|------|
| `interface` | wlan0 | WiFi 网络接口 |
| `driver` | hdf wifi | 使用 OpenHarmony HDF WiFi 驱动 |
| `ctrl_interface` | udp | 控制接口类型（UDP） |
| `ssid` | testap | 接入点 SSID |
| `hw_mode` | g | 802.11g 模式 |
| `channel` | 1 | 无线信道 |
| `ignore_broadcast_ssid` | 0 | 广播 SSID |

### wpa_supplicant.conf

**路径**: `wpa_supplicant/config/wpa_supplicant.conf`

**用途**: wpa_supplicant 客户端配置

**配置项**:

| 配置 | 值 | 说明 |
|------|-----|------|
| `country` | GB | 国家代码（英国） |
| `ctrl_interface` | udp | 控制接口类型（UDP） |
| `network` | {} | 网络配置块（空） |

---

## 源代码统计

| 模块 | C 文件数量 | 总行数 | 主要功能 |
|------|-----------|--------|---------|
| hostapd | 1 | 71 | AP 启动示例 |
| wpa_supplicant | 1 | 71 | STA 启动示例 |
| wpa_cli | 1 | 256 | 控制接口完整示例 |
| **总计** | **3** | **398** | - |

---

## 不包含的内容

### 测试代码 ❌

本项目不包含测试代码：
- 无 `test/` 目录
- 无 `*_test.c` 文件
- 无单元测试、集成测试

### 头文件 ❌

本项目不包含本地头文件：
- 所有头文件均为系统库或第三方库头文件
- hostapd 和 wpa_supplicant 模块直接使用系统头文件
- wpa_cli 使用第三方 wpa_supplicant 头文件

### 文档代码 ❌

本项目不包含文档代码：
- 无示例代码注释块
- 无 Doxygen 风格注释
- 代码注释主要用于功能说明

---

## 模块依赖关系

```
                    ┌─────────────┐
                    │ libwpa.so  │
                    │ (第三方)    │
                    └──────┬──────┘
                           │ dlopen
             ┌─────────────┼─────────────┐
             │             │             │
      ┌──────▼─────┐ ┌────▼──────┐ ┌───▼──────┐
      │  hostapd   │ │wpa_supp    │ │ wpa_cli  │
      │  (AP)      │ │  (STA)     │ │ (Control) │
      └────────────┘ └────────────┘ └──────────┘
             │             │             │
             └─────────────┴─────────────┘
                           │
                    ┌──────▼──────┐
                    │   OpenHarmony│
                    │   HDF Driver │
                    └─────────────┘
```

### 依赖说明

| 模块 | 依赖 | 依赖类型 | 说明 |
|------|------|---------|------|
| hostapd | libwpa.so | 动态加载 | 通过 dlopen 加载 |
| wpa_supplicant | libwpa.so | 动态加载 | 通过 dlopen 加载 |
| wpa_cli | wpa_supplicant (target) | GN 依赖 | 链接 libwpa_client.so |
| wpa_cli | libsec_shared | GN 依赖 | 安全函数库 |

---

## 文件类型分布

| 类型 | 数量 | 说明 |
|------|------|------|
| C 源文件 (.c) | 3 | 主要代码 |
| GN 构建文件 (.gn) | 4 | 构建配置 |
| 配置文件 (.conf) | 2 | 运行时配置 |
| JSON 配置 | 1 | 组件配置 |
| Markdown 文档 (.md) | 2 | README 和 Wiki |

---

## 代码组织特点

1. **扁平化结构**: 每个模块只包含 src 和 config 子目录
2. **独立构建**: 每个模块有自己的 BUILD.gn，可以独立编译
3. **单一职责**: 每个模块专注于一个功能点
4. **示例导向**: 代码简洁，便于学习和理解

---

## 扩展建议

如果需要扩展项目功能，建议按以下方式组织：

```
hostapd/
├── BUILD.gn
├── src/
│   ├── hostapd_sample.c         # 主程序
│   ├── hostapd_config.c         # 配置管理（新增）
│   └── hostapd_event.c         # 事件处理（新增）
├── include/                     # 头文件目录（新增）
│   └── hostapd_api.h
└── config/
    └── hostapd.conf
```

保持模块边界清晰，每个文件单一职责。
