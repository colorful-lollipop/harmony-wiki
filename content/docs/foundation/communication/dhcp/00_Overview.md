# DHCP 组件概览

## 文档目的

本文档提供 OpenHarmony DHCP 组件（@ohos/dhcp）的快速概览，帮助开发者在 5 分钟内理解项目核心概念和架构。

---

## 适用范围

- 组件版本: 3.1.0
- 适配系统: small, standard
- 子系统: communication

---

## 组件介绍

DHCP（Dynamic Host Configuration Protocol，动态主机配置协议）是 OpenHarmony 通信子系统的核心组件，提供：

1. **DHCP 客户端** - 从 DHCP 服务器获取 IP 地址、网关、DNS 等网络配置
2. **DHCP 服务端** - 响应 DHCP 请求，管理地址池和租约表
3. **IPv4/IPv6 双栈支持** - 同时支持 IPv4 和 IPv6 协议

---

## 核心能力

| 能力 | 说明 | 证据 |
|------|------|------|
| DHCP Client | 客户端获取 IP、网关、DNS | `services/dhcp_client/` |
| DHCP Server | 服务端分配 IP、管理租约 | `services/dhcp_server/` |
| IPC 服务化 | 通过 SystemAbility 提供跨进程服务 | `SA ID: 1126/1127` |
| 权限管控 | 验证 NETWORK_DHCP 权限 | `dhcp_permission_utils.cpp` |
| IPv6 支持 | 支持双栈地址获取 | `dhcp_ipv6_client.cpp` |

---

## 系统边界

### 属于子系统
- **communication** - 通信子系统

### 进程归属
- 运行在 `wifi_manager_service` 进程
- 作为 SystemAbility（SA）提供服务

### 依赖组件
- **基础能力**: ability_runtime, ipc, samgr, safwk
- **网络**: netmanager_base
- **安全**: access_token
- **工具**: c_utils, hilog, time_service
- **第三方**: openssl

### 对外接口
- **C API**: `interfaces/kits/c/dhcp_c_api.h` (17 个导出方法)
- **C++ SDK**: `frameworks/native/src/` (通过 IPC 调用服务)
- **JS API**: 在 `communication_wifi` 仓库实现（N-API 绑定）

---

## 快速架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层                                    │
│                  (链接 libdhcp_sdk.z.so)                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   SDK 层 (dhcp_sdk)                          │
│         C API → Proxy → IPC → SystemAbility                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────┴─────────────────────────────────┐
│                                                              │
│  ┌──────────────────┐                        ┌──────────────┐│
│  │  DHCP Client SA  │                        │ DHCP Server  ││
│  │      (1126)      │   (无直接依赖)          │    (1127)    ││
│  └────────┬─────────┘                        └──────┬───────┘│
│           │                                          │       │
│           └──────────────┬───────────────────────────┘       │
│                          │                                   │
│                          ▼                                   │
│                  ┌───────────────┐                          │
│                  │  dhcp_utils   │                          │
│                  │ (权限/SA/ARP)  │                          │
│                  └───────────────┘                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │   外部依赖组件          │
              │ (ipc, safwk, samgr...) │
              └───────────────────────┘
```

---

## 关键概念

### SystemAbility (SA)
DHCP 组件提供两个 SystemAbility：
- **1126**: DHCP Client SA
- **1127**: DHCP Server SA

配置文件位于 `services/sa_profile/`

### IPC 通信
- 使用 OpenHarmony IPC 框架（HDI Binder）
- 接口定义: `i_dhcp_client.h`, `i_dhcp_server.h`
- 命令码: `dhcp_manager_service_ipc_interface_code.h`

### 权限模型
所有公开方法都执行双重权限检查：
1. **原生进程检查**: 验证调用者为 TOKEN_NATIVE
2. **网络权限检查**: 验证持有 `ohos.permission.NETWORK_DHCP`

### C API 设计
- 提供同步 C 接口，不使用 Promise
- 回调机制: `ClientCallBack`, `ServerCallBack`
- 错误码: `DhcpErrorCode` (enum)

---

## 使用流程

### DHCP Client 流程
```
应用调用 C API
    → RegisterDhcpClientCallBack(注册回调)
    → StartDhcpClient(启动获取)
    → 等待 OnIpSuccessChanged 回调
    → 获取 DhcpResult（IP/DNS/网关）
```

### DHCP Server 流程
```
应用调用 C API
    → StartDhcpServer(启动服务)
    → SetDhcpRange(设置地址池)
    → 等待 OnServerLeasesChanged 回调
    → 管理租约表
```

---

## 文件路径速查

| 文件类型 | 路径 | 说明 |
|----------|------|------|
| C API 头文件 | `interfaces/kits/c/dhcp_c_api.h` | 对外接口定义 |
| SDK 实现 | `frameworks/native/src/` | SDK + Proxy + Stub |
| Client 服务 | `services/dhcp_client/src/` | Client SA 实现 |
| Server 服务 | `services/dhcp_server/src/` | Server SA 实现 |
| 工具库 | `services/utils/src/` | 权限/SA/ARP 工具 |
| SA 配置 | `services/sa_profile/1126.json` | Client SA 配置 |
| SA 配置 | `services/sa_profile/1127.json` | Server SA 配置 |

---

## 相关链接

- [项目定位](01_Project_Positioning.md) - 详细边界与职责
- [架构说明](03_Architecture.md) - 组件图与数据流
- [C API 参考](04_C_API_Reference.md) - 17 个方法完整文档
- [构建系统](06_Build_System.md) - GN Targets 与依赖
- [安全评审](08_Security_Review.md) - 安全风险清单
