# JS API 文档（N-API）

## 目的

本文档提供 Net Manager Exposed 的完整 JavaScript API 清单，包括每个接口的参数、返回值、同步/异步模式、权限要求和对应的 C++ 实现位置。

**重要**：所有接口均通过 N-API 暴露，使用 Promise 或 Callback 模式。

---

## API 清单总览

| 模块 | N-API 模块名 | SA ID | 主要用途 |
|------|--------------|-------|---------|
| Ethernet | `net.ethernet` | - | 以太网接口配置 |
| Sharing | `net.sharing` | - | 网络共享管理 |
| MDNS | `net.mdns` | 1161 | mDNS 服务发现 |
| VPN | `net.vpn` | - | VPN 连接管理 |
| NetFirewall | `net.netfirewall` | 8300 | 防火墙规则管理 |

---

## ohos.net.ethernet

### N-API 注册信息

- **注册文件**：`frameworks/js/napi/ethernet/ethernet_module.cpp`
- **注册函数**：`RegisterEthernetInterface`
- **模块定义**：ethernet_module.cpp:149-157
- **N-API 模块名**：`net.ethernet`

### 导出常量

| 常量名 | 值 | 类型 | 说明 |
|---------|-----|------|------|
| `STATIC` | 0 | IPSetMode | 静态 IP |
| `DHCP` | 1 | IPSetMode | 动态 IP（DHCP） |

**代码证据**：ethernet_module.cpp:82-86

### API 接口清单

#### 1. setIfaceConfig

设置网络接口配置信息

**签名**：
```typescript
// Callback 模式
function setIfaceConfig(iface: string, ic: InterfaceConfiguration, callback: AsyncCallback<void>): void;

// Promise 模式
function setIfaceConfig(iface: string, ic: InterfaceConfiguration): Promise<void>;
```

**参数**：
| 参数名 | 类型 | 必需 | 说明 |
|-------|------|------|------|
| iface | string | 是 | 网络接口名（如 `"eth0"`） |
| ic | InterfaceConfiguration | 是 | 接口配置对象 |
| callback | AsyncCallback<void> | 否（Promise 模式） | 结果回调 |

**InterfaceConfiguration 结构**：
```typescript
interface InterfaceConfiguration {
  mode: IPSetMode;          // STATIC 或 DHCP
  ipAddr?: string;           // IP 地址（静态模式）
  routeAddr?: string;         // 路由地址（网关）
  gateAddr?: string;         // 默认网关
  maskAddr?: string;         // 子网掩码
  dnsAddr0?: string;         // DNS 服务器 1
  dnsAddr1?: string;         // DNS 服务器 2
}
```

**权限**：
- `ohos.permission.CONNECTIVITY_INTERNAL`

**同步/异步**：异步（Promise/Callback）

**C++ 实现**：
- **N-API 入口**：`ethernet_module.cpp:61-65` (SetIfaceConfig)
- **Context 类**：`SetIfaceConfigContext`
- **执行函数**：`EthernetAsyncWork::ExecSetIfaceConfig`
- **回调函数**：`EthernetAsyncWork::SetIfaceConfigCallback`
- **SA 调用**：`EthernetService::SetIfaceConfig` @ ethernet_service.cpp:226

**错误码**：
- 参数无效（接口不存在、配置格式错误）
- 权限不足
- 网络操作失败

---

#### 2. getIfaceConfig

获取网络接口配置信息

**签名**：
```typescript
// Callback 模式
function getIfaceConfig(iface: string, callback: AsyncCallback<InterfaceConfiguration>): void;

// Promise 模式
function getIfaceConfig(iface: string): Promise<InterfaceConfiguration>;
```

**参数**：
| 参数名 | 类型 | 必需 | 说明 |
|-------|------|------|------|
| iface | string | 是 | 网络接口名 |
| callback | AsyncCallback<InterfaceConfiguration> | 否 | 结果回调 |

**返回值**：`InterfaceConfiguration` 对象

**权限**：
- `ohos.permission.GET_NETWORK_INFO`

**同步/异步**：异步

**C++ 实现**：
- **N-API 入口**：`ethernet_module.cpp:55-59` (GetIfaceConfig)
- **Context 类**：`GetIfaceConfigContext`
- **执行函数**：`EthernetAsyncWork::ExecGetIfaceConfig`
- **回调函数**：`EthernetAsyncWork::GetIfaceConfigCallback`
- **SA 调用**：`EthernetService::GetIfaceConfig` @ ethernet_service.cpp:241

---

#### 3. isIfaceActive

判断接口是否已激活

**签名**：
```typescript
// Callback 模式
function isIfaceActive(iface?: string, callback: AsyncCallback<number>): void;

// Promise 模式
function isIfaceActive(iface?: string): Promise<number>;
```

**参数**：
| 参数名 | 类型 | 必需 | 说明 |
|-------|------|------|------|
| iface | string | 否 | 接口名，不传则检查所有接口 |
| callback | AsyncCallback<number> | 否 | 结果回调 |

**返回值**：`number`（激活状态）

**权限**：
- `ohos.permission.GET_NETWORK_INFO`

**同步/异步**：异步

**C++ 实现**：
- **N-API 入口**：`ethernet_module.cpp:67-71` (IsIfaceActive)
- **Context 类**：`IsIfaceActiveContext`
- **执行函数**：`EthernetAsyncWork::ExecIsIfaceActive`
- **回调函数**：`EthernetAsyncWork::IsIfaceActiveCallback`
- **SA 调用**：`EthernetService::IsIfaceActive` @ ethernet_service.cpp:256

---

#### 4. getAllActiveIfaces

获取活动的网络接口列表

**签名**：
```typescript
// Callback 模式
function getAllActiveIfaces(callback: AsyncCallback<Array<string>>): void;

// Promise 模式
function getAllActiveIfaces(): Promise<Array<string>>;
```

**参数**：
| 参数名 | 类型 | 必需 | 说明 |
|-------|------|------|------|
| callback | AsyncCallback<Array<string>> | 否 | 结果回调 |

**返回值**：`Array<string>` - 激活的接口名列表

**权限**：
- `ohos.permission.GET_NETWORK_INFO`

**同步/异步**：异步

**C++ 实现**：
- **N-API 入口**：`ethernet_module.cpp:73-77` (GetAllActiveIfaces)
- **Context 类**：`GetAllActiveIfacesContext`
- **执行函数**：`EthernetAsyncWork::ExecGetAllActiveIfaces`
- **回调函数**：`EthernetAsyncWork::GetAllActiveIfacesCallback`
- **SA 调用**：`EthernetService::GetAllActiveIfaces` @ ethernet_service.cpp:270

---

#### 5. getEthernetDeviceInfos

获取以太网设备信息

**签名**：
```typescript
function getEthernetDeviceInfos(): Promise<Array<EthernetDeviceInfos>>;
```

**参数**：无

**返回值**：`Array<EthernetDeviceInfos>`

**权限**：
- `ohos.permission.GET_NETWORK_INFO`

**同步/异步**：异步（Promise）

**C++ 实现**：
- **N-API 入口**：`ethernet_module.cpp:99-103` (GetDeviceInformation)
- **Context 类**：`GetDeviceInformationContext`
- **执行函数**：`EthernetAsyncWork::ExecGetDeviceInformation`
- **回调函数**：`EthernetAsyncWork::GetDeviceInformationCallback`
- **SA 调用**：`EthernetService::GetDeviceInformation` @ ethernet_service.cpp:505

---

#### 6. getMacAddress

获取接口 MAC 地址

**签名**：
```typescript
function getMacAddress(iface: string): Promise<string>;
```

**参数**：
| 参数名 | 类型 | 必需 | 说明 |
|-------|------|------|------|
| iface | string | 是 | 接口名 |

**返回值**：`string` - MAC 地址（如 `"00:11:22:33:44:55"`）

**权限**：
- `ohos.permission.GET_ETHERNET_LOCAL_MAC`

**同步/异步**：异步（Promise）

**C++ 实现**：
- **N-API 入口**：`ethernet_module.cpp:49-53` (GetMacAddress)
- **Context 类**：`GetMacAddressContext`
- **执行函数**：`EthernetAsyncWork::ExecGetMacAddress`
- **回调函数**：`EthernetAsyncWork::GetMacAddressCallback`
- **SA 调用**：`EthernetService::GetMacAddress` @ ethernet_service.cpp:211

---

#### 7. on / off - 接口状态监听

**签名**：
```typescript
function on(type: 'interfaceStateChange', callback: Callback<InterfaceState>): void;
function off(type: 'interfaceStateChange', callback?: Callback<InterfaceState>): void;
```

**事件类型**：`'interfaceStateChange'`

**回调参数**：`InterfaceState` 对象

**权限**：
- `ohos.permission.GET_NETWORK_INFO`

**同步/异步**：同步（立即注册）

**C++ 实现**：
- **N-API 入口**：`ethernet_module.cpp:89-97` (On, Off)
- **Observer 类**：`InterfaceStateObserverWrapper` @ interface_state_observer_wrapper.cpp
- **事件常量**：`EVENT_STATS_CHANGE = "interfaceStateChange"`

---

## ohos.net.sharing

### N-API 注册信息

- **注册文件**：`frameworks/js/napi/sharing/src/netshare_module.cpp`
- **注册函数**：`InitNetShareModule`
- **模块定义**：netshare_module.cpp:206-214
- **N-API 模块名**：`net.sharing`

### 导出常量

#### SharingIfaceState

| 常量名 | 值 | 说明 |
|---------|-----|------|
| `SHARING_NIC_SERVING` | 0 | 正在共享 |
| `SHARING_NIC_CAN_SERVER` | 1 | 可共享 |
| `SHARING_NIC_ERROR` | 2 | 共享错误 |

**代码证据**：netshare_module.cpp:53-66

#### SharingIfaceType

| 常量名 | 值 | 说明 |
|---------|-----|------|
| `SHARING_WIFI` | 0 | WiFi 热点共享 |
| `SHARING_USB` | 1 | USB 共享 |
| `SHARING_BLUETOOTH` | 2 | 蓝牙共享 |

**代码证据**：netshare_module.cpp:61-64

### API 接口清单

#### 1. isSharingSupported

获取当前系统是否支持网络共享

**签名**：
```typescript
function isSharingSupported(callback: AsyncCallback<boolean>): void;
function isSharingSupported(): Promise<boolean>;
```

**权限**：无

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:67-72` (IsSharingSupported)
- **Context 类**：`IsSharingSupportedContext`
- **执行函数**：`NetShareAsyncWork::ExecIsSharingSupported`

---

#### 2. isSharing

获取当前共享状态

**签名**：
```typescript
function isSharing(callback: AsyncCallback<boolean>): void;
function isSharing(): Promise<boolean>;
```

**权限**：无

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:74-79` (IsSharing)
- **Context 类**：`NetShareIsSharingContext`
- **执行函数**：`NetShareAsyncWork::ExecIsSharing`

---

#### 3. startSharing

开启网络共享

**签名**：
```typescript
function startSharing(type: SharingIfaceType, callback: AsyncCallback<void>): void;
function startSharing(type: SharingIfaceType): Promise<void>;
```

**参数**：
| 参数名 | 类型 | 必需 | 说明 |
|-------|------|------|------|
| type | SharingIfaceType | 是 | 共享类型（WiFi/USB/蓝牙） |

**权限**：
- `ohos.permission.CONNECTIVITY_INTERNAL`（在 SA 端检查）

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:81-86` (StartSharing)
- **Context 类**：`NetShareStartSharingContext`
- **执行函数**：`NetShareAsyncWork::ExecStartSharing`
- **SA 调用**：`NetworkShareService::StartSharing` @ networkshare_service.cpp:172

---

#### 4. stopSharing

停止网络共享

**签名**：
```typescript
function stopSharing(type: SharingIfaceType, callback: AsyncCallback<void>): void;
function stopSharing(type: SharingIfaceType): Promise<void>;
```

**权限**：
- `ohos.permission.CONNECTIVITY_INTERNAL`

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:88-93` (StopSharing)
- **Context 类**：`StopSharingContext`
- **执行函数**：`NetShareAsyncWork::ExecStopSharing`

---

#### 5-7. getStats*Bytes

获取共享流量统计

**签名**：
```typescript
function getStatsRxBytes(callback: AsyncCallback<number>): void;
function getStatsRxBytes(): Promise<number>;

function getStatsTxBytes(callback: AsyncCallback<number>): void;
function getStatsTxBytes(): Promise<number>;

function getStatsTotalBytes(callback: AsyncCallback<number>): void;
function getStatsTotalBytes(): Promise<number>;
```

**返回值**：`number` - 字节数（单位：KB）

**权限**：无

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:116-135`
- **Context 类**：`GetStats*BytesContext`
- **执行函数**：`NetShareAsyncWork::ExecGetStats*Bytes`

---

#### 8. getSharingIfaces

获取指定状态的网卡名称列表

**签名**：
```typescript
function getSharingIfaces(state: SharingIfaceState, callback: AsyncCallback<Array<string>>): void;
function getSharingIfaces(state: SharingIfaceState): Promise<Array<string>>;
```

**权限**：无

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:95-100` (GetSharingIfaces)
- **Context 类**：`GetSharingIfacesContext`

---

#### 9. getSharingState

获取指定类型的共享状态

**签名**：
```typescript
function getSharingState(type: SharingIfaceType, callback: AsyncCallback<SharingIfaceState>): void;
function getSharingState(type: SharingIfaceType): Promise<SharingIfaceState>;
```

**权限**：无

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:102-107` (GetSharingState)
- **Context 类**：`GetSharingStateContext`

---

#### 10. getSharableRegexes

获取与指定类型匹配的网卡正则表达式列表

**签名**：
```typescript
function getSharableRegexes(type: SharingIfaceType, callback: AsyncCallback<Array<string>>): void;
function getSharableRegexes(type: SharingIfaceType): Promise<Array<string>>;
```

**权限**：无

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:109-114` (GetSharableRegexes)
- **Context 类**：`GetSharableRegexesContext`

---

#### 11. on / off - 状态监听

**签名**：
```typescript
// 共享状态改变
function on(type: 'sharingStateChange', callback: Callback<boolean>): void;
function off(type: 'sharingStateChange', callback?: Callback<boolean>): void;

// 接口共享状态改变
function on(type: 'interfaceSharingStateChange', callback: Callback<InterfaceSharingState>): void;
function off(type: 'interfaceSharingStateChange', callback?: Callback<InterfaceSharingState>): void;

// 上行网卡改变
function on(type: 'sharingUpstreamChange', callback: Callback<NetHandle>): void;
function off(type: 'sharingUpstreamChange', callback?: Callback<NetHandle>): void;
```

**权限**：无

**C++ 实现**：
- **N-API 入口**：`netshare_module.cpp:137-149` (On, Off)
- **Observer 类**：`NetShareObserverWrapper` @ netshare_observer_wrapper.cpp

---

## 其他模块（TODO 补充）

### ohos.net.mdns

**注册文件**：`frameworks/js/napi/mdns/src/mdns_module.cpp`
**SA ID**：1161

**待补充 API**：
- 服务注册
- 服务发现
- 本地服务管理

### ohos.net.vpn

**注册文件**：`frameworks/js/napi/vpn/src/vpn_module.cpp`

**待补充 API**：
- 连接/断开 VPN
- 配置 VPN
- 获取 VPN 状态

### ohos.net.netfirewall

**注册文件**：`frameworks/js/napi/netfirewall/src/netfirewall_module.cpp`
**SA ID**：8300

**待补充 API**：
- 添加/删除防火墙规则
- 获取规则列表
- 设置默认策略

---

## 错误码映射

所有 N-API 调用可能返回以下错误（需要从代码中完整提取）：

| 错误码 | 说明 | 对应 C++ 错误 |
|---------|------|--------------|
| TODO | TODO | TODO |

**代码证据**：需要进一步搜索错误码定义文件

---

## 参数校验规则

### 公共校验规则

1. **接口名校验**：必须是有效的网络接口名（如 `eth0`, `wlan0`）
2. **IP 地址校验**：符合 IPv4/IPv6 格式
3. **权限校验**：在 SA 端执行（`NetManagerPermission::CheckPermission`）
4. **空值检查**：必需参数不能为空

**代码证据**：services/ethernetmanager/src/ethernet_service.cpp 中的校验逻辑

---

## 安全注意事项

- 所有配置接口都需要权限验证
- MAC 地址获取需要特定权限（`GET_ETHERNET_LOCAL_MAC`）
- 网络共享可能暴露敏感网络拓扑
- 防火墙规则变更需要管理员权限

详细安全分析见 [安全风险评审](08_Security_Review.md)。

---

## 相关跳转

- [项目定位](01_Project_Positioning.md) - 权限模型
- [架构说明](03_Architecture.md) - 调用链与数据流
- [内部 API](05_Inner_API.md) - SA 接口定义
- [安全风险评审](08_Security_Review.md) - 攻击面分析
