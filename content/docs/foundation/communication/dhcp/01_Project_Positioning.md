# 项目定位与核心能力

## 文档目的

本文档定义 DHCP 组件的项目定位、系统边界、核心能力、运行环境与关键概念。

---

## 适用范围

- 组件: @ohos/dhcp
- 版本: 3.1.0
- 子系统: communication
- 适配系统: small, standard

---

## 项目定位

### 角色定位
DHCP 组件是 OpenHarmony 通信子系统的**网络配置基础服务**，提供：
1. DHCP 客户端功能 - 终端设备自动获取网络配置
2. DHCP 服务端功能 - 网络设备分配 IP 地址
3. IPC 服务化 - 向系统其他模块提供 DHCP 能力

### 不包含的功能
- ❌ WiFi 连接管理（由 `communication_wifi` 负责）
- ❌ 网络策略路由（由 `netmanager_base` 负责）
- ❌ DNS 解析（由系统 DNS 服务负责）
- ❌ IP 地址冲突检测（仅 ARP 检查）

---

## 系统边界

### 输入边界

| 输入来源 | 证据 | 数据类型 | 安全处理 |
|---------|------|----------|----------|
| IPC 请求 | `dhcp_client_stub.cpp:50` | 结构体序列化 | TokenID + 权限检查 |
| C API 调用 | `dhcp_c_service.cpp:79` | C 函数参数 | 参数校验宏 |
| DHCP 包 | `dhcp_socket.cpp:200` | 网络字节流 | 边界检查（部分实现） |
| 配置文件 | `dhcp_config.cpp:259` | JSON/文本文件 | 路径验证（待加强） |

### 输出边界

| 输出目标 | 证据 | 数据类型 | 安全措施 |
|---------|------|----------|----------|
| IPC 响应 | `dhcp_server_stub.cpp:100` | 结构体序列化 | 序列化安全 |
| 回调通知 | `dhcp_event.cpp:50` | 回调函数 | 虚表检查 |
| 日志输出 | `dhcp_client_service_impl.cpp:512` | 文本 | 敏感信息脱敏（部分） |
| 内核参数 | `dhcp_common_utils.cpp:345` | /proc/sys 写入 | 路径验证 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│              不可信区域 (外部网络、恶意应用)                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              输入验证层 (权限检查、参数校验)                    │
│            dhcp_permission_utils + 参数宏                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              业务逻辑层 (DHCP 协议处理、状态机)               │
│         dhcp_client_service_impl + dhcp_server_service      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              系统调用层 (内核、网络栈)                         │
│                   /proc + Socket API                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心能力

### 1. DHCP 客户端能力

| 能力 | 说明 | 证据 |
|------|------|------|
| IPv4 地址获取 | 从 DHCPv4 服务器获取 IP、子网掩码、网关 | `dhcp_client_state_machine.cpp:400` |
| IPv6 地址获取 | 支持 SLAAC + DHCPv6 | `dhcp_ipv6_client.cpp:150` |
| DNS 配置 | 获取主备 DNS 服务器 | `dhcp_result.cpp:80` |
| 缓存机制 | WiFi 场景缓存 IP 加快连接 | `dhcp_client_service_impl.cpp:300` |
| 异步回调 | IP 获取成功/失败通知 | `dhcp_event.cpp:50` |

### 2. DHCP 服务端能力

| 能力 | 说明 | 证据 |
|------|------|------|
| 地址池管理 | 配置可分配 IP 范围 | `dhcp_address_pool.cpp:100` |
| 租约管理 | 记录客户端租约信息 | `dhcp_s_server.cpp:500` |
| ARP 检查 | 检测 IP 冲突 | `dhcp_arp_checker.cpp:50` |
| DHCPv4 服务 | 标准 DHCPv4 协议实现 | `dhcp_s_server.cpp:300` |
| 租约导出 | 查询已分配租约列表 | `dhcp_server_service_impl.cpp:600` |

### 3. 基础服务能力

| 能力 | 说明 | 证据 |
|------|------|------|
| IPC 服务化 | SystemAbility 提供跨进程调用 | `dhcp_sa_manager.cpp:100` |
| 权限检查 | 验证调用者身份与权限 | `dhcp_permission_utils.cpp:50` |
| 线程池 | 异步任务处理 | `dhcp_thread.cpp:80` |
| 定时器 | 租约超时、重试机制 | `dhcp_system_timer.cpp:50` |

---

## 运行环境

### 系统要求

| 项 | 标准版 | Lite 版 |
|---|--------|---------|
| 最小内存 | 32MB | 512KB |
| 系统版本 | OpenHarmony 4.0+ | OpenHarmony 3.2+ |
| 必需组件 | safwk, ipc, samgr | liteipc |
| 依赖库 | c_utils, hilog, access_token | c_utils |

### 进程模型
- **服务进程**: `wifi_manager_service`
- **Client SA**: SA ID 1126, 按需启动 (run-on-create=false)
- **Server SA**: SA ID 1127, 按需启动 (run-on-create=false)
- **自动重启**: 失败后自动重启 (auto-restart=true)

### 线程模型
- **主线程**: SA 生命周期管理
- **工作线程池**: DHCP 包处理、异步回调
- **定时器线程**: 租约超时、重试任务

### 网络要求
- **接口类型**: WiFi, Ethernet（通过 `ifname` 指定）
- **协议栈**: IPv4/IPv6 双栈
- **Socket**: UDP 67/68 (DHCPv4), UDP 546/547 (DHCPv6)

---

## 关键概念

### DHCP 协议
- **RFC 2131**: DHCPv4 协议标准
- **RFC 3315**: DHCPv6 协议标准
- **Transaction ID**: 请求唯一标识（4 字节随机数）

### 系统能力 (SystemAbility)
- **SA Manager**: 系统能力管理器，负责 SA 注册与发现
- **SA Profile**: JSON 配置文件，定义 SA 属性
- **SA 生命周期**: OnStart → OnStop → OnDump

### 权令模型
- **TokenID**: OpenHarmony 访问令牌，标识调用者身份
- **TOKEN_NATIVE**: 原生进程 Token
- **AccessTokenKit**: 权限验证接口

### 回调机制
- **ClientCallBack**: 客户端 IP 获取结果回调
- **ServerCallBack**: 服务端租约变更回调
- **DhcpClientReport**: 客户端状态上报回调

### 状态机
- **DhcpClientStateMachine**: 客户端状态机（INIT → DISCOVER → OFFER → REQUEST → BOUND）
- **状态转换**: `dhcp_client_state_machine.cpp:200`

---

## 依赖关系

### 上游依赖（被依赖）
- `communication_wifi` - WiFi 服务调用 DHCP Client
- 其他子系统服务 - 可能调用 DHCP Server

### 下游依赖（依赖其他组件）
- **safwk**: SystemAbility 框架
- **ipc**: IPC 通信
- **samgr**: SA Manager
- **access_token**: 权限验证
- **hilog**: 日志输出
- **netmanager_base**: 网络管理接口
- **c_utils**: 工具库
- **openssl**: 加密相关

### 无循环依赖
- ✅ dhcp_sdk → dhcp_client / dhcp_server
- ✅ dhcp_client / dhcp_server → dhcp_utils
- ✅ dhcp_utils → 基础组件

---

## 性能指标（设计目标）

| 指标 | 目标值 | 说明 |
|------|--------|------|
| IP 获取延迟 | < 3 秒 | 典型 WiFi 场景 |
| 租约管理容量 | 1000+ | 单 Server |
| 并发连接数 | 32+ | 多网卡场景 |
| 内存占用 | < 5MB | 标准版 |

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [03_Architecture](03_Architecture.md) - 系统架构详情
- [06_Build_System](06_Build_System.md) - 依赖与构建
- [08_Security_Review](08_Security_Review.md) - 安全边界分析
