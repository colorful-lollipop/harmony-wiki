# 安全风险评估 (Security Risk Assessment)

**目的**: 深入分析 netmanager_ext 各模块的具体安全风险，提供可被利用点分析、触发路径、影响评估和修复建议。

**受众**: 安全研究员、安全开发工程师

**相关文档**: [05_AttackSurface.md](05_AttackSurface.md) - 攻击面总览

---

## 风险评估摘要

| 模块 | 风险等级 | 主要风险类型 | 关键风险点 |
|------|----------|--------------|-----------|
| **VPN** | 🔴 高 | 配置注入、权限提升 | 配置文件解析、路由劫持 |
| **NetFirewall** | 🔴 高 | 规则绕过、DoS | 规则验证、资源耗尽 |
| **Sharing** | 🟡 中 | 流量暴露、资源耗尽 | 共享配置、时长控制 |
| **Ethernet** | 🟡 中 | 配置注入、DoS | IP 格式验证、网关验证 |
| **MDNS** | 🟢 低 | 信息泄露、欺骗 | 服务伪造 |

---

## 1. VPN 模块风险分析

### 1.1 风险 V1: VPN 配置文件注入 (高危)

**风险描述**: VPN 配置包含大量敏感参数（服务器地址、证书路径、预共享密钥等），攻击者可能通过构造恶意配置实现路径遍历或命令注入。

**代码位置**:
- `frameworks/js/napi/vpn/src/vpn_config_utils.cpp:34-258`
- `frameworks/js/napi/vpnext/src/vpn_config_utils_ext.cpp:34-258`

**问题代码**:
```cpp
// vpn_config_utils.cpp:250-258
GetStringFromJsOptionItem(env, config, CONFIG_OPENVPN_CFG_FILE_PATH, openvpnConfig->ovpnConfigFilePath_);
GetStringFromJsOptionItem(env, config, CONFIG_OPENVPN_CA_CERT_FILE_PATH, openvpnConfig->ovpnCaCertFilePath_);
GetStringFromJsOptionItem(env, config, CONFIG_OPENVPN_USER_CERT_FILE_PATH, openvpnConfig->ovpnUserCertFilePath_);
GetStringFromJsOptionItem(env, config, CONFIG_OPENVPN_PRIVATE_KEY_FILE_PATH, ...);
```

**触发路径**:
```
JS: vpn.setUp({
       vpnType: 0,
       ovpnConfigFilePath: "../../../etc/passwd",  // 恶意路径
       ...
   })
  ↓
NAPI: vpn_config_utils.cpp:255 解析路径
  ↓
SA: networkvpn_service.cpp:299 保存配置
  ↓
文件系统: 可能读取/覆盖敏感文件
```

**影响评估**:
- 🔴 文件读取：可能读取系统敏感配置文件
- 🔴 配置污染：VPN 进程可能加载恶意配置
- 🟡 权限提升：取决于 VPN 进程权限

**修复建议**:
```cpp
// 添加路径规范化检查
bool IsValidVpnConfigPath(const std::string& path) {
    // 1. 转换为绝对路径
    char resolved[PATH_MAX];
    if (realpath(path.c_str(), resolved) == nullptr) {
        return false;
    }
    
    // 2. 检查是否在允许的目录
    const std::string basePath = "/data/app/el1/base/";
    return strncmp(resolved, basePath.c_str(), basePath.length()) == 0;
}
```

---

### 1.2 风险 V2: 路由劫持 (高危)

**风险描述**: VPN 配置中的路由参数可控制全局流量走向，恶意应用可能通过 VPN 劫持所有网络流量。

**代码位置**:
- `frameworks/js/napi/vpn/src/vpn_config_utils.cpp:112-115`
- `services/vpnmanager/src/networkvpn_service.cpp:299`

**问题代码**:
```cpp
// vpn_config_utils.cpp:112-115
uint32_t routesLength = NapiUtils::GetArrayLength(env, routes);
for (uint32_t idx = 0; idx < routesLength; idx++) {
    if (!ParseRoute(env, NapiUtils::GetArrayElement(env, routes, idx), routeInfo)) {
```

**触发路径**:
```
JS: vpn.setUp({
       routes: [{ address: "0.0.0.0", prefix: 0 }]  // 捕获所有流量
   })
  ↓
NAPI: 解析路由配置
  ↓
SA: 应用路由规则到系统路由表
  ↓
所有流量通过 VPN 隧道
```

**影响评估**:
- 🔴 流量劫持：所有网络流量可被 VPN 服务器截获
- 🔴 隐私泄露：敏感数据被传输到恶意服务器
- 🔴 DNS 劫持：可重定向 DNS 查询

**修复建议**:
```cpp
// 添加路由范围限制
bool IsValidRouteConfig(const std::vector<RouteInfo>& routes) {
    for (const auto& route : routes) {
        // 限制不可使用 0.0.0.0/0 (全流量捕获)
        if (route.address_ == "0.0.0.0" && route.prefix_ == 0) {
            // 需要额外权限 MANAGE_VPN_FULL_TUNNEL
            if (!HasFullTunnelPermission()) {
                return false;
            }
        }
    }
    return true;
}
```

---

### 1.3 风险 V3: 套接字保护绕过 (中危)

**风险描述**: `Protect` API 允许 VPN 应用保护特定套接字不走 VPN 隧道，但传入的 fd 可能未经验证。

**代码位置**:
- `frameworks/js/napi/vpn/src/context/protect_context.cpp:53`
- `services/vpnmanager/src/networkvpn_service.cpp:674`

**问题代码**:
```cpp
// protect_context.cpp:53
socketFd_ = NapiUtils::GetInt32FromValue(GetEnv(), params[0]);
```

**影响评估**:
- 🟡 流量绕过：恶意应用可能绕过 VPN 监控
- 🟡 检测规避：VPN 可见流量不完整

---

## 2. NetFirewall 模块风险分析

### 2.1 风险 F1: 防火墙规则注入 (高危)

**风险描述**: 防火墙规则直接控制网络访问，恶意规则可能开放危险端口或阻断关键服务。

**代码位置**:
- `services/netfirewallmanager/src/netfirewall_stub.cpp:62`
- `services/netfirewallmanager/src/netfirewall_rule_manager.cpp`

**问题代码**:
```cpp
// netfirewall_stub.cpp:62
if (!strPermission.empty() && !NetManagerPermission::CheckPermission(strPermission)) {
    NETMGR_EXT_LOG_E("CheckPermission failed");
    return NETMANAGER_EXT_ERR_PERMISSION_DENIED;
}
```

**注意**: 权限检查存在，但规则内容验证不足。

---

### 2.2 风险 F2: 规则资源耗尽 (中危)

**风险描述**: 添加大量防火墙规则可能导致内核 nftables 表溢出，造成拒绝服务。

**触发路径**:
```
循环调用 addFirewallRule() 数千次
  ↓
内核 nftables 规则表满
  ↓
无法添加新规则 / 系统变慢
```

**修复建议**:
```cpp
// 添加规则数量限制
constexpr int32_t MAX_FIREWALL_RULES = 1000;

int32_t AddFirewallRule(const NetFirewallRule& rule) {
    if (GetCurrentRuleCount() >= MAX_FIREWALL_RULES) {
        return ERR_RULE_LIMIT_EXCEEDED;
    }
    // ...
}
```

---

## 3. NetworkSharing 模块风险分析

### 3.1 风险 S1: 未授权共享启动 (中危)

**风险描述**: 网络共享可能暴露设备到外部网络，如果权限检查不严格，恶意应用可能私自开启热点。

**代码位置**:
- `services/networksharemanager/src/networkshare_service.cpp:172`

**证据**:
```cpp
// networkshare_service.cpp:172
if (!NetManagerPermission::CheckPermission(Permission::CONNECTIVITY_INTERNAL)) {
    return NETMANAGER_EXT_ERR_PERMISSION_DENIED;
}
```

**评估**: 权限检查存在，但 `CONNECTIVITY_INTERNAL` 权限可能授予过多应用。

---

### 3.2 风险 S2: 共享持久化攻击 (中危)

**风险描述**: 如果共享状态在系统重启后自动恢复，恶意配置可能持久化。

**检查点**: 需确认配置是否保存到持久化存储。

---

## 4. Ethernet 模块风险分析

### 4.1 风险 E1: IP 配置注入 (中危)

**风险描述**: 网卡配置参数直接传递到内核，缺乏格式验证可能导致无效配置或网络中断。

**代码位置**:
- `frameworks/js/napi/ethernet/context/set_iface_config_context.cpp:57-79`

**问题代码**:
```cpp
// set_iface_config_context.cpp:57-79
iface_ = NapiUtils::GetStringFromValueUtf8(GetEnv(), params[ARG_NUM_0]);
config_->mode_ = static_cast<IPSetMode>(NapiUtils::GetInt32Property(GetEnv(), params[ARG_NUM_1], "mode"));
ipAddresses = NapiUtils::GetStringPropertyUtf8(GetEnv(), params[ARG_NUM_1], "ipAddr");
```

**缺少验证**:
- IP 地址格式 (IPv4/IPv6)
- 子网掩码有效性
- 网关是否可达
- DNS 服务器格式

**修复建议**:
```cpp
// 添加 IP 格式验证
bool IsValidIpv4Address(const std::string& ip) {
    struct sockaddr_in sa;
    return inet_pton(AF_INET, ip.c_str(), &(sa.sin_addr)) == 1;
}

bool IsValidIpv6Address(const std::string& ip) {
    struct sockaddr_in6 sa6;
    return inet_pton(AF_INET6, ip.c_str(), &(sa6.sin6_addr)) == 1;
}
```

---

### 4.2 风险 E2: 网卡名称遍历 (低危)

**风险描述**: `iface` 参数直接用于系统调用，可能用于探测系统网卡信息。

---

## 5. MDNS 模块风险分析

### 5.1 风险 M1: 服务信息泄露 (低危)

**风险描述**: mDNS 服务注册可能泄露设备信息和运行的服务。

**代码位置**:
- `services/mdnsmanager/src/mdns_service.cpp:47`

**评估**: 这是 mDNS 协议的正常行为，但需确保敏感信息不在服务名中。

---

## 6. 通用风险

### 6.1 风险 C1: 参数长度限制缺失

**多个位置** 的字符串参数缺乏长度限制，可能导致：
- 缓冲区溢出（如果后续使用不安全函数）
- 内存分配过大导致 OOM
- 日志注入

**受影响参数**:
- VPN 配置中的各类字符串字段
- MDNS 服务类型和名称
- Ethernet 配置参数

**修复建议**:
```cpp
constexpr size_t MAX_STRING_LENGTH = 4096;

std::string GetStringWithLimit(napi_env env, napi_value value) {
    std::string str = NapiUtils::GetStringFromValueUtf8(env, value);
    if (str.length() > MAX_STRING_LENGTH) {
        throw std::invalid_argument("String too long");
    }
    return str;
}
```

---

### 6.2 风险 C2: IPC 超时缺失

**风险描述**: 同步 IPC 调用可能阻塞，导致拒绝服务。

**受影响接口**: 所有同步 IPC 方法

---

## 7. 权限模型评估

### 7.1 权限粒度分析

| 权限 | 用途 | 风险 |
|------|------|------|
| `CONNECTIVITY_INTERNAL` | 通用网络操作 | 🔴 过于宽泛，多个模块共用 |
| `MANAGE_VPN` | VPN 管理 | 🟡 合理，但应区分系统/用户 VPN |
| `MANAGE_NET_FIREWALL` | 防火墙管理 | 🟢 特定功能 |
| `GET_ETHERNET_LOCAL_MAC` | 获取 MAC | 🟢 最小权限 |

**建议**:
- 拆分 `CONNECTIVITY_INTERNAL` 为更细粒度的权限
- 区分 VPN 创建和系统级 VPN 配置权限

---

## 8. 可利用性评分 (CVSS-like)

| 风险 ID | 模块 | 利用难度 | 影响 | 评分 |
|---------|------|----------|------|------|
| V1 | VPN | 中 | 高 | **7.5** (高危) |
| V2 | VPN | 低 | 高 | **8.0** (高危) |
| V3 | VPN | 高 | 中 | **5.5** (中危) |
| F1 | Firewall | 中 | 高 | **7.0** (高危) |
| F2 | Firewall | 低 | 中 | **5.0** (中危) |
| S1 | Sharing | 中 | 中 | **5.5** (中危) |
| E1 | Ethernet | 低 | 中 | **5.5** (中危) |
| E2 | Ethernet | 低 | 低 | **3.0** (低危) |
| M1 | MDNS | 低 | 低 | **2.5** (低危) |

---

## 9. 修复优先级建议

### P0 (立即修复)
1. **V1**: VPN 配置文件路径遍历防护
2. **V2**: VPN 路由配置全流量捕获限制
3. **F1**: 防火墙规则内容验证

### P1 (短期修复)
4. **E1**: Ethernet IP 格式验证
5. **C1**: 全局字符串长度限制
6. **F2**: 防火墙规则数量限制

### P2 (中期改进)
7. **权限粒度细化**
8. **IPC 超时机制**
9. **V3**: 套接字保护 fd 验证

---

## 10. 安全开发建议

### 输入验证清单
- [ ] 所有字符串参数长度限制 (< 4096)
- [ ] IP 地址格式验证 (inet_pton)
- [ ] 文件路径规范化 (realpath)
- [ ] 枚举值范围校验
- [ ] 数组/列表大小限制

### 权限检查清单
- [ ] 敏感操作前权限检查
- [ ] 权限检查在数据解析之后
- [ ] IPC Stub 端二次验证

### 资源限制清单
- [ ] 回调注册数量限制
- [ ] 规则/配置数量限制
- [ ] IPC 调用超时设置

---

*文档版本: v2.0 | 更新时间: 2025-02-07*

---

## 攻击面分析

### 1. N-API 接口攻击面

#### 风险 1：参数注入（接口配置）

**位置**：
- `ethernet_module.cpp:61-65` (SetIfaceConfig)
- `services/ethernetmanager/src/ethernet_service.cpp:226` (SetIfaceConfig 实现)

**可利用路径**：
1. JavaScript 应用调用 `setIfaceConfig(eth0, maliciousConfig)`
2. N-API 解析 `InterfaceConfiguration` 对象
3. 未验证的 IP/网关/DNS 被传递到 SA
4. SA 直接应用到网络接口

**触发条件**：
- 参数校验不充分
- 未检查 IP 地址格式合法性
- 未验证网关可达性

**影响**：
- 拒绝服务（无效网络配置）
- 中间人攻击（恶意 DNS 服务器）
- 网络劫持（错误网关）

**证据**：
- ethernet_service.cpp:226 - 权限检查后直接应用配置

**修复建议**：
```cpp
// 在 SetIfaceConfig 中添加
bool IsValidIpAddress(const std::string& ip) {
    // 检查 IPv4/IPv6 格式
    // 检查是否为私有地址范围
    // 检查非保留地址
}

bool IsValidGateway(const std::string& gw) {
    // 检查网关可达性
    // ARP 探测
}
```

---

#### 风险 2：参数注入（网络共享）

**位置**：
- `netshare_module.cpp:81-86` (StartSharing)
- `services/networksharemanager/src/networkshare_service.cpp:172` (StartSharing 实现)

**可利用路径**：
1. 恶意应用调用 `startSharing(Wifi)`
2. 无权限检查（部分场景）
3. 启动 WiFi 热点，共享所有流量
4. 攻击者连接并窃听流量

**触发条件**：
- 权限验证不严格
- 未验证调用者身份
- 未限制共享时长

**影响**：
- 流量窃听
- 流量劫持
- 资源耗尽（持续共享）

**证据**：
- networkshare_service.cpp:172 - 权限检查 `CONNECTIVITY_INTERNAL`
- 但部分调用可能绕过

**修复建议**：
```cpp
// 添加调用者身份验证
bool IsTrustedCaller(uid_t uid) {
    // 检查 UID 是否为系统应用
    // 检查签名完整性
}

// 添加共享时长限制
void LimitSharingDuration() {
    // 超时自动停止
}
```

---

### 2. IPC 攻击面

#### 风险 3：IPC 消息伪造

**位置**：
- `frameworks/native/ethernetclient/src/proxy/ethernet_proxy.cpp`
- `services/ethernetmanager/src/ethernet_stub.cpp`

**可利用路径**：
1. 攻击者获取 Root 权限
2. 伪造 IPC 消息调用 SA 方法
3. 绕过 N-API 层的权限检查
4. 直接操作网络配置

**触发条件**：
- Root 设备被攻破
- SELinux 策略配置不当

**影响**：
- 完全控制网络配置
- 绕过应用权限模型
- 持久化恶意配置

**证据**：
- 所有 IPC Proxy/Stub 都有明确的接口定义
- 但未在 Stub 中重新验证权限

**修复建议**：
```cpp
// 在 Stub 端重新验证权限
bool EthernetStub::SetIfaceConfig(Config config) {
    if (!NetManagerPermission::CheckPermission(...)) {
        NETMGR_EXT_LOG_E("IPC request without permission");
        return ERR_PERMISSION_DENIED;
    }
    return service_->SetIfaceConfig(config);
}
```

---

#### 风险 4：拒绝服务（IPC 拥塞）

**位置**：
- 所有 SA 服务的 IPC Stub
- 同步调用可能导致阻塞

**可利用路径**：
1. 恶意应用发起大量 IPC 调用
2. SA 服务端处理队列耗尽
3. 正常请求被阻塞
4. 服务无响应

**触发条件**：
- IPC 调用无超时限制
- 同步调用耗时操作
- 无速率限制

**影响**：
- 拒绝服务
- 系统无响应
- 资源耗尽

**修复建议**：
```cpp
// 添加 IPC 调用超时
const int IPC_TIMEOUT_MS = 5000;

// 添加速率限制
RateLimiter limiter(10); // 每秒最多 10 次
```

---

### 3. 权限提升攻击面

#### 风险 5：权限检查绕过

**位置**：
- `services/ethernetmanager/src/ethernet_service.cpp:211-527`
- `services/networksharemanager/src/networkshare_service.cpp:172-560`

**可利用路径**：
1. 应用通过 N-API 调用敏感接口
2. SA 端检查权限（如 `CONNECTIVITY_INTERNAL`）
3. 但未检查调用者 UID
4. 应用通过其他方式获取 UID 令牌

**触发条件**：
- 仅检查权限字符串，未验证令牌真实性
- 权限授予后未验证调用者上下文

**影响**：
- 权限提升
- 越权访问网络配置
- 绕过系统权限模型

**证据**：
- ethernet_service.cpp:226 - `NetManagerPermission::CheckPermission(Permission::CONNECTIVITY_INTERNAL)`
- 未验证 UID 或令牌来源

**修复建议**：
```cpp
// 验证权限令牌
bool ValidateAccessToken(const std::string& permission) {
    AccessTokenID tokenId = GetCallingAccessTokenID();
    return AccessTokenKit::VerifyPermission(tokenId, permission);
}
```

---

### 4. 网络配置攻击面

#### 风险 6：DNS 劫持

**位置**：
- `ethernet_service.cpp:226` (SetIfaceConfig)
- InterfaceConfiguration 中的 `dnsAddr0`, `dnsAddr1`

**可利用路径**：
1. 恶意应用设置恶意 DNS 服务器
2. 所有 DNS 查询被劫持
3. 钓鱼攻击、流量劫持

**触发条件**：
- 未验证 DNS 服务器 IP 合法性
- 未限制 DNS 为已知可信服务器

**影响**：
- DNS 劫持
- 流量重定向
- 中间人攻击

**修复建议**：
```cpp
// 白名单验证
bool IsTrustedDNS(const std::string& dns) {
    std::vector<std::string> trustedDNS = {
        "8.8.8.8", "1.1.1.1",
        // 系统 DNS
    };
    return std::find(trustedDNS.begin(), trustedDNS.end(), dns) != trustedDNS.end();
}
```

---

#### 风险 7：MAC 地址泄露

**位置**：
- `ethernet_service.cpp:211` (GetMacAddress)

**可利用路径**：
1. 恶意应用调用 `getMacAddress`
2. 获取设备唯一标识
3. 追踪用户设备

**触发条件**：
- 权限 `GET_ETHERNET_LOCAL_MAC` 可能被授予给过多应用

**影响**：
- 设备指纹识别
- 用户追踪
- 隐私泄露

**证据**：
- ethernet_service.cpp:211 - 权限检查 `GET_ETHERNET_LOCAL_MAC`

**修复建议**：
```cpp
// 限制 MAC 地址访问
bool ShouldAllowMacAccess(uid_t uid) {
    // 仅系统应用和特定网络工具可访问
    return uid == SYSTEM_UID || uid == NETWORK_TOOL_UID;
}
```

---

### 5. VPN 攻击面

#### 风险 8：VPN 流量劫持

**位置**：
- `services/vpnmanager/src/networkvpn_service.cpp:674` (SetUp)
- VPN 隧道配置

**可利用路径**：
1. 恶意 VPN 应用建立隧道
2. 所有网络流量通过 VPN
3. VPN 服务端窃听/篡改流量

**触发条件**：
- VPN 配置未验证服务器证书
- 未限制 VPN 服务器 IP 范围

**影响**：
- 流量完全劫持
- 数据泄露
- 中间人攻击

**修复建议**：
```cpp
// 验证 VPN 服务器
bool IsAuthorizedVPNServer(const std::string& serverIP) {
    // 白名单验证
    // 证书验证
}
```

---

### 6. 防火墙攻击面

#### 风险 9：防火墙规则注入

**位置**：
- `services/netfirewallmanager/src/netfirewall_service.cpp`
- `services/netfirewallmanager/src/netfirewall_rule_manager.cpp`

**可利用路径**：
1. 恶意应用添加防火墙规则
2. 阻断合法网络流量
3. 或允许恶意流量

**触发条件**：
- 规则语法未严格验证
- 未限制规则数量

**影响**：
- 拒绝服务
- 网络隔离
- 绕过安全防护

**修复建议**：
```cpp
// 限制规则数量
const int MAX_RULES = 100;

// 验证规则语法
bool IsValidRule(const FirewallRule& rule) {
    // 检查 IP/端口范围
    // 防止规则冲突
}
```

---

#### 风险 10：iptables 指令注入

**位置**：
- `services/netfirewallmanager/src/netfirewall_rule_native_helper.cpp`

**可利用路径**：
1. 规则包含未转义的特殊字符
2. 直接拼接到 iptables 命令
3. 命令注入攻击

**触发条件**：
- 使用 `system()` 或 `popen()` 直接执行命令
- 未转义用户输入

**影响**：
- 命令执行
- Root 权限获取
- 系统完全沦陷

**修复建议**：
```cpp
// 使用 nftables API 替代命令行
// 或严格转义输入
std::string EscapeShellArg(const std::string& arg) {
    // Shell 转义逻辑
}
```

---

## 输入校验现状

### 已实现的校验

| 校验类型 | 位置 | 覆盖范围 |
|---------|------|---------|
| 权限检查 | 所有 SA 服务 | ✓ 基本覆盖 |
| 类型检查 | N-API 层 | ✓ napi_get_type_info() |
| 非空检查 | 部分 API | ✗ 不全面 |

### 缺失的校验

| 校验类型 | 说明 | 影响 |
|---------|------|------|
| IP 地址格式 | 未验证 IPv4/IPv6 格式 | 参数注入 |
| IP 地址范围 | 未检查是否为合法网络段 | 网络劫持 |
| DNS 服务器白名单 | 允许任意 DNS | DNS 劫持 |
| 参数长度限制 | 可能超长字符串 | 缓冲区溢出 |
| 特殊字符转义 | 直接使用 Shell 命令 | 命令注入 |

---

## 权限模型分析

### 权限清单

| 权限 | 风险等级 | 暴露接口 | 建议 |
|-------|---------|---------|------|
| `GET_NETWORK_INFO` | 中 | 所有配置查询 | 限制为只读 |
| `GET_ETHERNET_LOCAL_MAC` | 高 | getMacAddress | 限制为系统应用 |
| `CONNECTIVITY_INTERNAL` | 高 | 所有配置修改 | 严格验证 |
| `MANAGE_VPN` | 高 | VPN 连接/断开 | 证书验证 |
| `MANAGE_NET_FIREWALL` | 高 | 防火墙规则 | 限制规则数量 |
| `MANAGE_ENTERPRISE_WIFI_CONNECTION` | 中 | EAP 配置 | 验证配置合法性 |

### 权限检查实现

**当前实现**：
```cpp
// ethernet_service.cpp:226
if (!NetManagerPermission::CheckPermission(Permission::CONNECTIVITY_INTERNAL)) {
    NETMGR_EXT_LOG_E("EthernetService SetIfaceConfig no js permission");
    return NETMANAGER_EXT_ERR_PERMISSION_DENIED;
}
```

**问题**：
- 仅检查权限字符串，未验证令牌
- 未检查调用者 UID
- 无操作审计日志

**建议改进**：
```cpp
// 完整权限验证
napi_status CheckPermissionWithContext(napi_env env, const std::string& permission) {
    napi_value global = nullptr;
    napi_get_global(env, &global);
    napi_value callerInfo = GetCallerInfo(env, global);

    // 验证 AccessToken
    // 记录审计日志
    return NAPI_OK;
}
```

---

## 数据隐私风险

### 敏感数据处理

| 数据类型 | 处理位置 | 隐私风险 |
|---------|---------|---------|
| MAC 地址 | getMacAddress | 设备指纹 |
| IP 地址 | SetIfaceConfig | 网络位置 |
| 流量统计 | getStats*Bytes | 用户行为分析 |
| 网络拓扑 | startSharing | 网络结构暴露 |

**建议**：
- 添加数据访问审计日志
- 限制敏感数据读取频率
- 实施数据最小化原则

---

## 加密与传输安全

### IPC 通信安全

| 组件 | 传输方式 | 安全措施 |
|-------|---------|---------|
| N-API → SA | HDI/HBinder | 需确认加密方式 |
| SA → Kernel | 系统调用 | SELinux 保护 |

**待确认**：
- IPC 消息是否加密？
- 是否有消息完整性校验？
- 是否有重放攻击防护？

---

## 审计与日志

### 当前日志实现

- 使用 `NETMGR_EXT_LOGE` 等宏
- 日志通过 hilog 输出

**缺失**：
- 操作审计日志（谁在何时做了什么）
- 敏感操作记录
- 异常访问告警

**建议**：
```cpp
// 添加审计日志
void AuditLog(const std::string& operation, uid_t uid, bool success) {
    NETMGR_EXT_LOGI("AUDIT: op=%s, uid=%d, result=%s",
                   operation.c_str(), uid, success ? "ok" : "failed");
}
```

---

## 安全修复优先级

### 高优先级（P0）

1. **命令注入**（风险 10）
   - 直接威胁系统安全
   - 必须立即修复

2. **权限提升**（风险 5）
   - 可能绕过系统权限模型
   - 必须修复

3. **DNS 劫持**（风险 6）
   - 影响所有网络流量
   - 高优先级

### 中优先级（P1）

4. **参数注入**（风险 1, 2）
   - 可能导致配置错误
   - 需要修复

5. **拒绝服务**（风险 4）
   - 影响可用性
   - 需要缓解措施

### 低优先级（P2）

6. **MAC 地址泄露**（风险 7）
   - 隐私问题
   - 可以规划修复

---

## 检查范围与局限性

### 已检查范围

✅ N-API 接口参数校验
✅ IPC 权限检查实现
✅ 网络配置安全性
✅ VPN 管理安全性
✅ 防火墙规则安全性
✅ 数据隐私保护

### 未检查范围

❌ SELinux 策略配置
❌ 文件系统权限（如 `/etc/network/`）
❌ 硬件驱动安全性（如 `/dev/net/*`）
❌ 加密算法实现（如 VPN）
❌ 第三方依赖库安全（如 OpenSSL）

**原因**：
- 这些内容不在本仓库范围
- 需要跨仓库协同审计

---

## 相关跳转

- [项目定位](01_Project_Positioning.md) - 权限模型
- [JS API 文档](04_JS_API.md) - 接口安全要求
- [内部 API](05_Inner_API.md) - IPC 安全
- [架构说明](03_Architecture.md) - 信任边界
