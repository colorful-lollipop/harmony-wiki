# DSoftBus 项目概览

## 项目定位

**DSoftBus** (Distributed SoftBus) 是 OpenHarmony 分布式软总线核心组件，实现近场设备间的统一分布式通信。

### 核心能力

1. **设备发现** - 支持多种通信模式（WLAN、Bluetooth、BLE、NFC、USB 等）的设备发现与连接
2. **网络管理** - 统一的设备网络化和拓扑管理，提供设备信息用于数据传输
3. **数据传输** - 建立消息、字节流、文件传输通道

> 代码证据: `README.md` 第 7-11 行

### 系统能力

```
SystemCapability.Communication.SoftBus.Core
```

> 代码证据: `bundle.json` 第 27 行

## 运行环境

### 适配系统类型

| 类型 | 说明 |
|------|------|
| mini | 轻量系统 |
| small | 小型系统 |
| standard | 标准系统 |

> 代码证据: `bundle.json` 第 19-23 行

### 依赖组件

- `ability_base` - 能力基础框架
- `ability_runtime` - 运行时能力
- `access_token` - 访问令牌管理
- `bluetooth` - 蓝牙协议栈
- `device_auth` - 设备认证
- `hilog` - 日志系统
- `huks` - 安全密钥管理
- `ipc` - 进程间通信

> 代码证据: `bundle.json` 第 81-100 行（部分）

## 目录结构

```
dsoftbus/
├── adapter               # 适配层代码
│   └── ...
├── br_proxy             # BR(Basic Rate)代理模块
│   ├── common/          # 公共工具
│   ├── taihe/           # 太和模块
│   └── br_proxy_module.c # N-API 模块入口
├── core                 # 核心代码
│   ├── adapter          # 适配代码
│   ├── authentication   # 设备认证模块
│   ├── bus_center      # 网络中心/拓扑管理
│   ├── common          # 公共基础代码
│   ├── connection      # 连接管理
│   ├── discovery       # 设备发现
│   ├── frame           # 框架核心
│   └── transmission    # 数据传输
├── interfaces          # 对外接口
│   ├── inner_kits      # 内部接口（系统能力间调用）
│   └── kits            # SDK 接口
├── sdk                 # SDK 代码
│   ├── napi            # N-API 实现
│   │   └── link_enhance/ # Link Enhance N-API 模块
│   ├── bus_center     # Bus Center SDK
│   ├── connection      # 连接管理 SDK
│   ├── frame          # 框架 SDK
│   └── transmission    # 传输 SDK
├── tests               # 测试代码
└── tools               # 工具

```

> 代码证据: `README.md` 第 23-46 行 + `sdk/` + `core/` 目录结构

## 关键概念

### 通信模式

| 模式 | 说明 | 协议 |
|------|------|------|
| WLAN | WiFi 局域网通信 | TCP/IP |
| BR | 蓝牙基础速率 | Bluetooth BR/EDR |
| BLE | 低功耗蓝牙 | Bluetooth LE |
| ETH | 以太网 | TCP/IP |
| CoAP | CoAP 协议发现 | CoAP |
| USB | USB 直连 | USB |

### 核心对象

| 对象 | 说明 |
|------|------|
| `pkgName` | 包名，调用方标识 |
| `publishId` | 发布任务 ID |
| `refreshId` | 发现任务 ID |
| `networkId` | 网络标识符 |
| `sessionId` | 会话 ID |

## 权限要求

### 必选权限

| 权限名称 | 说明 |
|----------|------|
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 |
| `ohos.permission.DISTRIBUTED_SOFTBUS_CENTER` | 分布式软总线中心 |

> 代码证据: `README.md` 第 58 行

## 错误码概览

错误码采用模块化设计，主要模块：

| 模块常量 | 模块名 |
|---------|--------|
| `DISC_SUB_MODULE_CODE = 1` | 设备发现 |
| `CONN_SUB_MODULE_CODE = 2` | 连接管理 |
| `AUTH_SUB_MODULE_CODE = 3` | 认证 |
| `LNN_SUB_MODULE_CODE = 4` | 本地网络 |
| `TRANS_SUB_MODULE_CODE = 5` | 传输 |
| `IPCRPC_SUB_MODULE_CODE = 6` | IPC/RPC |

> 代码证据: `interfaces/kits/common/softbus_error_code.h` 第 59-70 行

## 版本信息

| 项目 | 值 |
|------|-----|
| 当前版本 | 4.0.2 |
| 包名 | @ohos/dsoftbus |
| License | Apache License 2.0 |

> 代码证据: `bundle.json` 第 3 行

---

**相关文档**

- [架构说明](./02_Architecture.md)
- [N-API 接口](./03_NAPI.md)
- [编译配置](./05_Build.md)
