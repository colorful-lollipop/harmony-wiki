# 调用链分析（附录）

## 目的

本文档提供 Net Manager Ext 关键 API 的详细调用链，帮助开发者理解从 JavaScript 到内核的完整数据流。

---

## 调用链图例

```
JavaScript App
    ↓ (1) N-API 入口
N-API Module
    ↓ (2) 参数解析与校验
Async Work Context
    ↓ (3) 异步工作队列
IPC Proxy
    ↓ (4) HDI/HBinder 传输
SA Stub
    ↓ (5) 权限验证
SA Service
    ↓ (6) 业务逻辑
netmanager_base / netstack
    ↓ (7) 系统调用
Kernel / Drivers
```

---

## 以太网配置调用链

### 1. JavaScript 调用

```typescript
import ethernet from '@ohos.net.ethernet';

const config = {
  mode: ethernet.STATIC,
  ipAddr: '192.168.1.100',
  routeAddr: '192.168.1.1',
  gateAddr: '192.168.1.1',
  maskAddr: '255.255.255.0',
  dnsAddr0: '8.8.8.8',
};

try {
  await ethernet.setIfaceConfig('eth0', config);
  console.log('Configuration set successfully');
} catch (error) {
  console.error('Failed:', error.code, error.message);
}
```

### 2. N-API 层处理

**文件**：`frameworks/js/napi/ethernet/ethernet_module.cpp:61-65`

```cpp
napi_value SetIfaceConfig(napi_env env, napi_callback_info info)
{
    return ModuleTemplate::Interface<SetIfaceConfigContext>(
        env, info, SET_IFACE, nullptr,
        EthernetAsyncWork::ExecSetIfaceConfig,
        EthernetAsyncWork::SetIfaceConfigCallback);
}
```

**Context 解析**：
- `SetIfaceConfigContext::ParseParams()` - 提取接口名和配置对象
- 验证参数类型（string, object）
- 转换为 `InterfaceConfiguration` 结构

### 3. 异步工作执行

**文件**：`frameworks/js/napi/ethernet/src/set_iface_config_context.cpp`

```cpp
void SetIfaceConfigContext::Exec(napi_env env, void* data)
{
    // 1. 获取 IPC Proxy 实例
    auto proxy = GetEthernetProxy();

    // 2. 转换参数
    EthernetConfigParcel config;
    config.iface = iface_;
    config.ipAddr = ic_.ipAddr;
    config.maskAddr = ic_.maskAddr;
    // ...

    // 3. IPC 调用
    int32_t ret = proxy->SetIfaceConfig(config);

    // 4. 设置结果
    result_ = ret;
}
```

### 4. IPC 传输

**Proxy 文件**：`frameworks/native/ethernetclient/src/proxy/ethernet_proxy.cpp`

**Stub 文件**：`services/ethernetmanager/src/ethernet_stub.cpp`

```
N-API 进程                SA 进程
    │                       │
    ├────── HDI/HBinder ─────►
    │                       │
Proxy                   Stub
    │                       │
    ▼                       ▼
EthernetProxy          EthernetStub
```

### 5. SA 权限验证

**文件**：`services/ethernetmanager/src/ethernet_service.cpp:226`

```cpp
int32_t EthernetService::SetIfaceConfig(const EthernetConfig& config)
{
    // 1. 权限检查
    if (!NetManagerPermission::CheckPermission(Permission::CONNECTIVITY_INTERNAL)) {
        NETMGR_EXT_LOG_E("EthernetService SetIfaceConfig no js permission");
        return NETMANAGER_EXT_ERR_PERMISSION_DENIED;
    }

    // 2. 调用者信息
    auto callerInfo = GetCallingInfo();
    NETMGR_EXT_LOGI("Caller: uid=%d, pid=%d", callerInfo.uid, callerInfo.pid);

    // 3. 业务逻辑
    return ApplyIfaceConfig(config);
}
```

### 6. 网络配置应用

**文件**：`services/ethernetmanager/src/ethernet_service.cpp`

```cpp
int32_t EthernetService::ApplyIfaceConfig(const EthernetConfig& config)
{
    // 1. 验证配置
    if (!ValidateConfig(config)) {
        return NETMANAGER_EXT_ERR_INVALID_PARAM;
    }

    // 2. 停止 DHCP（如果是静态）
    if (config.mode == STATIC) {
        dhcpController_->Stop(config.iface);
    }

    // 3. 应用 IP 配置
    ipManager_->SetAddress(config.iface, config.ipAddr, config.maskAddr);

    // 4. 应用网关
    ipManager_->SetRoute(config.iface, config.gateAddr);

    // 5. 配置 DNS
    dnsManager_->SetServers({config.dnsAddr0, config.dnsAddr1});

    // 6. 持久化配置
    PersistConfig(config);

    return NETMANAGER_EXT_SUCCESS;
}
```

### 7. 内核调用

**文件**：`netmanager_base`（外部依赖）

```
┌─────────────────────────────┐
│  netmanager_base           │
│  ┌──────────────────┐   │
│  │ IP Manager     │   │
│  │ DHCP Client    │   │
│  │ DNS Manager    │   │
│  └──────────────────┘   │
└──────────┬─────────────────┘
           │ syscalls
           ↓
┌─────────────────────────────┐
│  Linux Kernel            │
│  ┌──────────────────┐   │
│  │  netdev       │   │
│  │  ip route     │   │
│  │  dnsmasq      │   │
│  └──────────────────┘   │
└─────────────────────────────┘
```

---

## 网络共享启动调用链

### 1. JavaScript 调用

```typescript
import sharing from '@ohos.net.sharing';

try {
  await sharing.startSharing(sharing.SHARING_WIFI);
  console.log('Sharing started');
} catch (error) {
  console.error('Failed:', error.code);
}
```

### 2. N-API 层处理

**文件**：`frameworks/js/napi/sharing/src/netshare_module.cpp:81-86`

```cpp
napi_value StartSharing(napi_env env, napi_callback_info info)
{
    return ModuleTemplate::Interface<NetShareStartSharingContext>(
        env, info, FUNCTION_START_SHARING, nullptr,
        NetShareAsyncWork::ExecStartSharing,
        NetShareAsyncWork::StartSharingCallback);
}
```

### 3. IPC 传输

**Proxy**：`frameworks/native/netshareclient/src/proxy/sharing_proxy.cpp`
**Stub**：`services/networksharemanager/src/networkshare_stub.cpp`

### 4. SA 权限验证

**文件**：`services/networksharemanager/src/networkshare_service.cpp:172`

```cpp
int32_t NetworkShareService::StartSharing(int32_t type)
{
    if (!NetManagerPermission::CheckPermission(Permission::CONNECTIVITY_INTERNAL)) {
        NETMGR_EXT_LOG_E("StartSharing no permission");
        return NETMANAGER_EXT_ERR_PERMISSION_DENIED;
    }
    // 业务逻辑...
}
```

### 5. 共享启动流程

**文件**：`services/networksharemanager/src/networkshare_main_statemachine.cpp`

```
启动
  ↓
检查上游网络
  ↓
停止 DHCP
  ↓
配置 WiFi 热点
  ├─ 设置 SSID
  ├─ 设置密码
  ├─ 配置 IP (10.42.0.1)
  └─ 启动 hostapd
  ↓
配置 NAT
  ├─ 添加转发规则
  └─ 配置 masquerade
  ↓
启动 DHCP 服务
  └─ 为连接设备分配 IP
  ↓
共享状态 = SHARING_NIC_SERVING
```

### 6. 事件回调

**Observer**：`frameworks/js/napi/sharing/src/netshare_observer_wrapper.cpp`

```cpp
void NetShareObserverWrapper::OnEvent(const SharingEvent& event)
{
    napi_value jsEvent = CreateJsEvent(event);
    napi_value callback = GetCallback(event.type);
    napi_call_threadsafe_function(env_, callback, nullptr, jsEvent, nullptr, nullptr);
}
```

**事件传播**：
```
SA 事件 → IPC Callback → N-API Observer → JavaScript
```

---

## VPN 连接调用链

### 1. JavaScript 调用

```typescript
import vpn from '@ohos.net.vpn';

const config = {
  vpnId: 1,
  name: 'MyVPN',
  username: 'user',
  password: 'pass',
  server: 'vpn.example.com',
};

try {
  await vpn.setUp(config);
  console.log('VPN connected');
} catch (error) {
  console.error('Failed:', error);
}
```

### 2. N-API 层处理

**文件**：`frameworks/js/napi/vpn/src/vpn_module.cpp`

### 3. IPC 传输

**Proxy**：`frameworks/native/netvpnclient/src/proxy/vpn_proxy.cpp`
**Stub**：`services/vpnmanager/src/networkvpn_stub.cpp`

### 4. SA 权限验证

**文件**：`services/vpnmanager/src/networkvpn_service.cpp:674`

```cpp
int32_t NetworkVpnService::SetUp(const VpnConfig& config)
{
    if (!NetManagerPermission::CheckPermission(Permission::MANAGE_VPN)) {
        NETMGR_EXT_LOG_E("check vpn permission failed");
        return NETMANAGER_EXT_ERR_PERMISSION_DENIED;
    }
    // 业务逻辑...
}
```

### 5. VPN 建立流程

```
验证配置
  ↓
检查权限
  ↓
调用 VPN 类型特定控制器
  ├─ L2TP (l2tp_vpn_ctl.cpp)
  ├─ IPsec (ipsec_vpn_ctl.cpp)
  └─ 虚拟 VPN (virtual_vpn_ctl.cpp)
  ↓
创建 tun 设备
  ↓
配置路由
  ↓
添加 DNS
  ↓
VPN 状态 = CONNECTED
```

---

## MDNS 服务发现调用链

### 1. JavaScript 调用

```typescript
import mdns from '@ohos.net.mdns';

const service = {
  name: 'MyService',
  type: '_http._tcp.local',
  port: 8080,
};

try {
  await mdns.addLocalService(service);
  console.log('Service registered');
} catch (error) {
  console.error('Failed:', error);
}
```

### 2. N-API 层处理

**文件**：`frameworks/js/napi/mdns/src/mdns_module.cpp`

### 3. IPC 传输

**Proxy**：`frameworks/native/mdnsclient/src/proxy/mdns_proxy.cpp`
**Stub**：`services/mdnsmanager/src/mdns_stub.cpp`

### 4. SA 处理

**文件**：`services/mdnsmanager/src/mdns_service.cpp`

```
注册服务
  ↓
创建 DNS 记录 (SRV, PTR, A, AAAA)
  ↓
绑定 mDNS socket (224.0.0.1:5353)
  ↓
定期广播
  ↓
响应查询
```

### 5. 网络层

```
mDNS 协议
  ┌────────────────────┐
  │  Multicast DNS    │
  │  224.0.0.1:5353 │
  └────────┬───────────┘
           │ UDP
           ↓
┌────────────────────┐
│  LAN 设备       │
│  ├─ 查询服务     │
│  └─ 发布服务     │
└────────────────────┘
```

---

## 防火墙规则添加调用链

### 1. JavaScript 调用

```typescript
import firewall from '@ohos.net.netfirewall';

const rule = {
  uid: 1000,
  direction: 0, // OUTGOING
  action: 1,    // ALLOW
};

try {
  await firewall.addFirewallRule(rule);
  console.log('Rule added');
} catch (error) {
  console.error('Failed:', error);
}
```

### 2. N-API 层处理

**文件**：`frameworks/js/napi/netfirewall/src/netfirewall_module.cpp`

### 3. IPC 传输

**Proxy**：`frameworks/native/netfirewallclient/src/netfirewall_proxy.cpp`
**Stub**：`services/netfirewallmanager/src/netfirewall_stub.cpp`

### 4. SA 权限验证

**文件**：`services/netfirewallmanager/src/netfirewall_service.cpp`

```cpp
int32_t NetFirewallService::AddFirewallRule(const FirewallRule& rule)
{
    if (!NetManagerPermission::CheckPermission(PERMISSION_MANAGE_NET_FIREWALL)) {
        NETMGR_EXT_LOG_E("Permission denied");
        return NETMANAGER_EXT_ERR_PERMISSION_DENIED;
    }
    // 业务逻辑...
}
```

### 5. 规则应用流程

**文件**：`services/netfirewallmanager/src/netfirewall_rule_manager.cpp`

```
验证规则
  ↓
检查限制
  ↓
转换为 iptables 命令
  ↓
执行系统调用
  ↓
应用到内核 netfilter
  ┌────────────────────┐
  │  iptables/nftables │
  └─────────┬─────────┘
            │
            ↓
┌────────────────────┐
│  Linux Kernel    │
│  ┌────────────┐ │
│  │ netfilter   │ │
│  └────────────┘ │
└────────────────────┘
```

---

## 关键调用链总结

| 功能 | N-API 入口 | IPC Proxy | SA 服务 | 内核调用 |
|------|-----------|----------|---------|---------|
| 以太网配置 | SetIfaceConfig | EthernetProxy | EthernetService | netdev, ip route |
| 网络共享 | StartSharing | SharingProxy | NetworkShareService | hostapd, iptables |
| VPN 连接 | SetUp | VpnProxy | NetworkVpnService | tun, ip route |
| MDNS 注册 | AddLocalService | MdnsProxy | MdnsService | multicast socket |
| 防火墙规则 | AddFirewallRule | FirewallProxy | NetFirewallService | iptables, nftables |

---

## 相关跳转

- [架构说明](03_Architecture.md) - 整体架构图
- [JS API 文档](04_JS_API.md) - 接口定义
- [内部 API](05_Inner_API.md) - IPC 接口
- [安全风险评审](08_Security_Review.md) - 调用链安全
