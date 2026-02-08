# 内部 API (Inner API)

## 1. 接口层概述

内部 API 位于 `interfaces/innerkits/` 目录，为系统应用和框架层提供 C++ 接口。

### 1.1 接口库清单

| 库名 | 输出文件 | 路径 | innerapi_tags |
|------|----------|------|---------------|
| net_conn_manager_if | `libnet_conn_manager_if.z.so` | `interfaces/innerkits/netconnclient/` | platformsdk, sasdk |
| net_policy_manager_if | `libnet_policy_manager_if.z.so` | `interfaces/innerkits/netpolicyclient/` | platformsdk |
| net_stats_manager_if | `libnet_stats_manager_if.z.so` | `interfaces/innerkits/netstatsclient/` | platformsdk |
| net_native_manager_if | `libnet_native_manager_if.z.so` | `interfaces/innerkits/netmanagernative/` | platformsdk |
| net_conn_parcel | `libnet_conn_parcel.a` | `interfaces/innerkits/netconnclient/` | 内部 |
| net_policy_parcel | `libnet_policy_parcel.a` | `interfaces/innerkits/netpolicyclient/` | 内部 |
| net_stats_parcel | `libnet_stats_parcel.a` | `interfaces/innerkits/netstatsclient/` | 内部 |
| net_native_parcel | `libnet_native_parcel.a` | `interfaces/innerkits/netmanagernative/` | 内部 |

**证据**: `bundle.json:153-340`

---

## 2. NetConnClient 接口

### 2.1 类定义

**文件**: `interfaces/innerkits/netconnclient/include/net_conn_client.h`

```cpp
namespace OHOS {
namespace NetManagerStandard {

class NetConnClient {
public:
    static NetConnClient &GetInstance();
    
    // 网络查询
    int32_t GetDefaultNet(NetHandle &netHandle);
    int32_t HasDefaultNet(bool &hasDefaultNet);
    int32_t GetAllNets(std::list<int32_t> &netIdList);
    int32_t GetConnectionProperties(const NetHandle &netHandle, NetLinkInfo &info);
    int32_t GetNetCapabilities(const NetHandle &netHandle, NetAllCapabilities &cap);
    
    // 代理配置
    int32_t SetGlobalHttpProxy(const HttpProxy &httpProxy);
    int32_t GetGlobalHttpProxy(HttpProxy &httpProxy);
    int32_t GetDefaultHttpProxy(int32_t bindNetId, HttpProxy &httpProxy);
    
    // 监听注册
    int32_t RegisterNetConnCallback(const sptr<INetConnCallback> &callback);
    int32_t RegisterNetConnCallback(const sptr<NetSpecifier> &netSpecifier,
                                     const sptr<INetConnCallback> &callback,
                                     const uint32_t &timeoutMS);
    int32_t UnregisterNetConnCallback(const sptr<INetConnCallback> &callback);
    
    // 供应商管理 (系统级)
    int32_t RegisterNetSupplier(NetBearType bearerType, const std::string &ident,
                                 const std::set<NetCap> &netCaps, uint32_t &supplierId);
    int32_t UnregisterNetSupplier(uint32_t supplierId);
    int32_t UpdateNetSupplierInfo(uint32_t supplierId, const sptr<NetSupplierInfo> &netSupplierInfo);
    int32_t UpdateNetLinkInfo(uint32_t supplierId, const sptr<NetLinkInfo> &netLinkInfo);
};

} // namespace NetManagerStandard
} // namespace OHOS
```

### 2.2 回调接口

**文件**: `interfaces/innerkits/netconnclient/include/proxy/i_net_conn_callback.h`

```cpp
class INetConnCallback : public IRemoteBroker {
public:
    virtual int32_t OnNetAvailable(const sptr<NetHandle> &netHandle) = 0;
    virtual int32_t OnNetCapabilitiesChange(const sptr<NetHandle> &netHandle,
                                            const sptr<NetAllCapabilities> &netAllCap) = 0;
    virtual int32_t OnNetConnectionPropertiesChange(const sptr<NetHandle> &netHandle,
                                                     const sptr<NetLinkInfo> &info) = 0;
    virtual int32_t OnNetLost(const sptr<NetHandle> &netHandle) = 0;
    virtual int32_t OnNetUnavailable() = 0;
};
```

### 2.3 IPC 接口定义

**文件**: `interfaces/innerkits/netconnclient/include/proxy/i_net_conn_service.h`

```cpp
class INetConnService : public IRemoteBroker {
public:
    virtual int32_t SystemReady() = 0;
    virtual int32_t RegisterNetSupplier(NetBearType bearerType, const std::string &ident,
                                         const std::set<NetCap> &netCaps, uint32_t &supplierId) = 0;
    virtual int32_t UnregisterNetSupplier(uint32_t supplierId) = 0;
    virtual int32_t RegisterNetSupplierCallback(uint32_t supplierId,
                                                 const sptr<INetSupplierCallback> &callback) = 0;
    virtual int32_t RegisterNetConnCallback(const sptr<INetConnCallback> callback) = 0;
    virtual int32_t RegisterNetConnCallback(const sptr<NetSpecifier> &netSpecifier,
                                             const sptr<INetConnCallback> callback,
                                             const uint32_t &timeoutMS) = 0;
    virtual int32_t UnregisterNetConnCallback(const sptr<INetConnCallback> &callback) = 0;
    virtual int32_t UpdateNetSupplierInfo(uint32_t supplierId,
                                           const sptr<NetSupplierInfo> &netSupplierInfo) = 0;
    virtual int32_t UpdateNetLinkInfo(uint32_t supplierId, const sptr<NetLinkInfo> &netLinkInfo) = 0;
    virtual int32_t GetDefaultNet(int32_t &netId) = 0;
    virtual int32_t HasDefaultNet(bool &flag) = 0;
    virtual int32_t GetConnectionProperties(int32_t netId, NetLinkInfo &info) = 0;
    virtual int32_t GetNetCapabilities(int32_t netId, NetAllCapabilities &netAllCap) = 0;
    virtual int32_t SetGlobalHttpProxy(const HttpProxy &httpProxy) = 0;
    virtual int32_t GetGlobalHttpProxy(HttpProxy &httpProxy) = 0;
    virtual int32_t GetDefaultHttpProxy(int32_t bindNetId, HttpProxy &httpProxy) = 0;
    virtual int32_t SetAirplaneMode(bool state) = 0;
};
```

### 2.4 IPC 命令码

**文件**: `interfaces/innerkits/netconnclient/include/proxy/conn_ipc_interface_code.h`

| 命令码 | 值 | 说明 |
|--------|-----|------|
| CMD_NM_GET_DEFAULT_NET | 0 | 获取默认网络 |
| CMD_NM_HAS_DEFAULT_NET | 1 | 检查是否有默认网络 |
| CMD_NM_GET_ALL_NETS | 2 | 获取所有网络 |
| CMD_NM_GET_CONNECTION_PROPERTIES | 3 | 获取连接属性 |
| CMD_NM_GET_NET_CAPABILITIES | 4 | 获取网络能力 |
| CMD_NM_REGISTER_NET_SUPPLIER | 5 | 注册网络供应商 |
| CMD_NM_UNREGISTER_NET_SUPPLIER | 6 | 注销网络供应商 |
| CMD_NM_REGISTER_NET_CONN_CALLBACK | 7 | 注册连接回调 |
| CMD_NM_UNREGISTER_NET_CONN_CALLBACK | 8 | 注销连接回调 |
| CMD_NM_UPDATE_NET_SUPPLIER_INFO | 9 | 更新供应商信息 |
| CMD_NM_UPDATE_NET_LINK_INFO | 10 | 更新链路信息 |
| CMD_NM_SET_AIRPLANE_MODE | 11 | 设置飞行模式 |
| CMD_NM_SET_GLOBAL_HTTP_PROXY | 12 | 设置全局代理 |
| CMD_NM_GET_GLOBAL_HTTP_PROXY | 13 | 获取全局代理 |
| CMD_NM_GET_DEFAULT_HTTP_PROXY | 14 | 获取默认代理 |
| ... | ... | ... |

---

## 3. NetPolicyClient 接口

### 3.1 类定义

**文件**: `interfaces/innerkits/netpolicyclient/include/net_policy_client.h`

```cpp
class NetPolicyClient {
public:
    static NetPolicyClient &GetInstance();
    
    // UID 策略
    int32_t SetPolicyByUid(uint32_t uid, uint32_t policy);
    int32_t GetPolicyByUid(uint32_t uid, uint32_t &policy);
    int32_t GetUidsByPolicy(uint32_t policy, std::vector<uint32_t> &uids);
    
    // 后台策略
    int32_t SetBackgroundPolicy(bool allow);
    int32_t GetBackgroundPolicy(bool &allow);
    int32_t GetBackgroundPolicyByUid(uint32_t uid, uint32_t &policy);
    
    // 配额策略
    int32_t SetNetQuotaPolicies(const std::vector<NetQuotaPolicy> &policies);
    int32_t GetNetQuotaPolicies(std::vector<NetQuotaPolicy> &policies);
    
    // 设备空闲白名单
    int32_t SetDeviceIdleTrustlist(const std::vector<uint32_t> &uids, bool isAdd);
    int32_t GetDeviceIdleTrustlist(std::vector<uint32_t> &uids);
    
    // 网络访问检查
    int32_t IsUidNetAllowed(uint32_t uid, bool isMetered, bool &isAllowed);
    
    // 策略重置
    int32_t ResetPolicies(const std::string &simId);
    int32_t RestoreAllPolicies(const std::string &simId);
    
    // 监听
    int32_t RegisterNetPolicyCallback(const sptr<INetPolicyCallback> &callback);
    int32_t UnregisterNetPolicyCallback(const sptr<INetPolicyCallback> &callback);
};
```

### 3.2 IPC 命令码

**文件**: `interfaces/innerkits/netpolicyclient/include/policy_ipc_interface_code.h`

| 命令码 | 说明 |
|--------|------|
| CMD_NPS_SET_POLICY_BY_UID | 设置 UID 策略 |
| CMD_NPS_GET_POLICY_BY_UID | 获取 UID 策略 |
| CMD_NPS_GET_UIDS_BY_POLICY | 根据策略获取 UID 列表 |
| CMD_NPS_SET_BACKGROUND_POLICY | 设置后台策略 |
| CMD_NPS_GET_BACKGROUND_POLICY | 获取后台策略 |
| CMD_NPS_SET_NET_QUOTA_POLICIES | 设置配额策略 |
| CMD_NPS_GET_NET_QUOTA_POLICIES | 获取配额策略 |
| CMD_NPS_IS_UID_NET_ALLOWED | 检查 UID 网络权限 |
| CMD_NPS_REGISTER_NET_POLICY_CALLBACK | 注册策略回调 |
| CMD_NPS_UNREGISTER_NET_POLICY_CALLBACK | 注销策略回调 |

---

## 4. NetStatsClient 接口

### 4.1 类定义

**文件**: `interfaces/innerkits/netstatsclient/include/net_stats_client.h`

```cpp
class NetStatsClient {
public:
    static NetStatsClient &GetInstance();
    
    // 接口流量统计
    int32_t GetIfaceRxBytes(uint64_t &stats, const std::string &interfaceName);
    int32_t GetIfaceTxBytes(uint64_t &stats, const std::string &interfaceName);
    int32_t GetIfaceRxPackets(uint64_t &stats, const std::string &interfaceName);
    int32_t GetIfaceTxPackets(uint64_t &stats, const std::string &interfaceName);
    
    // 应用流量统计
    int32_t GetUidRxBytes(uint64_t &stats, uint32_t uid);
    int32_t GetUidTxBytes(uint64_t &stats, uint32_t uid);
    int32_t GetUidRxPackets(uint64_t &stats, uint32_t uid);
    int32_t GetUidTxPackets(uint64_t &stats, uint32_t uid);
    
    // 设备总流量
    int32_t GetAllRxBytes(uint64_t &stats);
    int32_t GetAllTxBytes(uint64_t &stats);
    int32_t GetAllRxPackets(uint64_t &stats);
    int32_t GetAllTxPackets(uint64_t &stats);
    
    // 详细统计信息
    int32_t GetIfaceStats(NetStatsInfo &info, const std::string &interfaceName);
    int32_t GetIfaceUidStats(NetStatsInfo &info, const std::string &interfaceName, uint32_t uid);
    
    // 数据更新
    int32_t UpdateIfacesStats();
    int32_t UpdateStatsData();
};
```

---

## 5. NetsysNativeClient 接口

### 5.1 类定义

**文件**: `interfaces/innerkits/netmanagernative/include/netsys_native_service_proxy.h`

```cpp
class INetsysService : public IRemoteBroker {
public:
    // 网络管理
    virtual int32_t NetworkCreatePhysical(int32_t netId, int32_t permission) = 0;
    virtual int32_t NetworkCreateVirtual(int32_t netId, bool hasDns) = 0;
    virtual int32_t NetworkDestroy(int32_t netId) = 0;
    virtual int32_t NetworkAddInterface(int32_t netId, const std::string &iface) = 0;
    virtual int32_t NetworkRemoveInterface(int32_t netId, const std::string &iface) = 0;
    
    // 接口配置
    virtual int32_t InterfaceAddAddress(const std::string &iface, const std::string &addr,
                                         int32_t prefixLen) = 0;
    virtual int32_t InterfaceDelAddress(const std::string &iface, const std::string &addr,
                                         int32_t prefixLen) = 0;
    virtual int32_t InterfaceSetUp(const std::string &iface) = 0;
    virtual int32_t InterfaceSetDown(const std::string &iface) = 0;
    
    // 路由管理
    virtual int32_t NetworkAddRoute(int32_t netId, const std::string &interfaceName,
                                     const std::string &destination, const std::string &nextHop) = 0;
    virtual int32_t NetworkRemoveRoute(int32_t netId, const std::string &interfaceName,
                                        const std::string &destination, const std::string &nextHop) = 0;
    
    // DNS 管理
    virtual int32_t SetResolverConfig(uint16_t netId, uint16_t baseTimeoutMsec, uint8_t retryCount,
                                       const std::vector<std::string> &servers,
                                       const std::vector<std::string> &domains) = 0;
    virtual int32_t GetResolverConfig(uint16_t netId, std::vector<std::string> &servers,
                                       std::vector<std::string> &domains) = 0;
    virtual int32_t CreateNetworkCache(uint16_t netId) = 0;
    virtual int32_t DestroyNetworkCache(uint16_t netId, bool isVpn) = 0;
    
    // 防火墙
    virtual int32_t FirewallSetUidsAllowedListChain(uint32_t chain, const std::vector<uint32_t> &uids) = 0;
    virtual int32_t FirewallSetUidsDeniedListChain(uint32_t chain, const std::vector<uint32_t> &uids) = 0;
    virtual int32_t FirewallEnableChain(uint32_t chain, bool enable) = 0;
    virtual int32_t FirewallSetUidRule(uint32_t chain, uint32_t uid, uint32_t firewallRule) = 0;
    
    // 流量统计
    virtual int32_t GetUidStats(uint64_t &stats, uint32_t uid, uint32_t type) = 0;
    virtual int32_t GetIfaceStats(uint64_t &stats, const std::string &interfaceName, uint32_t type) = 0;
    
    // 带宽控制
    virtual int32_t BandwidthEnableDataSaver(bool enable) = 0;
    virtual int32_t BandwidthSetIfaceQuota(const std::string &ifName, int64_t bytes) = 0;
    virtual int32_t BandwidthRemoveIfaceQuota(const std::string &ifName) = 0;
    virtual int32_t BandwidthAddDeniedList(uint32_t uid) = 0;
    virtual int32_t BandwidthRemoveDeniedList(uint32_t uid) = 0;
};
```

---

## 6. 数据结构定义

### 6.1 NetHandle (网络句柄)

**文件**: `interfaces/innerkits/netconnclient/include/net_handle.h`

```cpp
class NetHandle : public Parcelable {
public:
    NetHandle();
    explicit NetHandle(int32_t netId);
    
    int32_t GetNetId() const;
    int32_t BindSocket(int32_t socketFd) const;
    
    // DNS 解析
    int32_t GetAddressesByName(const std::string &host, std::vector<INetAddr> &addrList) const;
    int32_t GetAddressByName(const std::string &host, INetAddr &addr) const;
    
    // Parcelable
    bool Marshalling(Parcel &parcel) const override;
    static sptr<NetHandle> Unmarshalling(Parcel &parcel);
    
private:
    int32_t netId_;
};
```

### 6.2 NetLinkInfo (网络链路信息)

**文件**: `interfaces/innerkits/netconnclient/include/net_link_info.h`

```cpp
struct NetLinkInfo : public Parcelable {
    std::string ifaceName_;
    std::string domain_;
    std::vector<INetAddr> netAddrList_;
    std::vector<INetAddr> dnsList_;
    std::vector<INetAddr> routeList_;
    std::vector<std::string> searchDomains_;
    HttpProxy httpProxy_;
    
    bool Marshalling(Parcel &parcel) const override;
    static bool Unmarshalling(Parcel &parcel, NetLinkInfo &info);
};
```

### 6.3 NetAllCapabilities (网络能力)

**文件**: `interfaces/innerkits/netconnclient/include/net_all_capabilities.h`

```cpp
struct NetAllCapabilities : public Parcelable {
    std::set<NetCap> netCaps_;
    std::set<NetBearType> bearerTypes_;
    uint32_t linkUpBandwidthKbps_;
    uint32_t linkDownBandwidthKbps_;
    std::string bearerPrivateIdentifier_;
    
    bool Marshalling(Parcel &parcel) const override;
    static bool Unmarshalling(Parcel &parcel, NetAllCapabilities &cap);
};
```

### 6.4 HttpProxy (HTTP 代理)

**文件**: `interfaces/innerkits/netconnclient/include/http_proxy.h`

```cpp
struct HttpProxy : public Parcelable {
    std::string host_;
    int32_t port_;
    std::vector<std::string> exclusionList_;
    std::string pacUrl_;
    
    bool Marshalling(Parcel &parcel) const override;
    static bool Unmarshalling(Parcel &parcel, HttpProxy &httpProxy);
    
    bool IsValid() const;
    bool IsPacProxy() const;
    std::string ToString() const;
};
```

### 6.5 NetQuotaPolicy (流量配额策略)

**文件**: `interfaces/innerkits/netpolicyclient/include/net_quota_policy.h`

```cpp
struct NetQuotaPolicy : public Parcelable {
    std::string netType_;
    std::string ident_;
    int64_t limitBytes_;
    int64_t warningBytes_;
    int64_t lastLimitSnooze_;
    int64_t lastWarningSnooze_;
    int64_t metered_;
    int64_t limitAction_;
    
    bool Marshalling(Parcel &parcel) const override;
    static bool Unmarshalling(Parcel &parcel, NetQuotaPolicy &policy);
};
```

### 6.6 NetStatsInfo (流量统计信息)

**文件**: `interfaces/innerkits/netstatsclient/include/net_stats_info.h`

```cpp
struct NetStatsInfo : public Parcelable {
    std::string iface_;
    uint32_t uid_;
    uint64_t rxBytes_;
    uint64_t txBytes_;
    uint64_t rxPackets_;
    uint64_t txPackets_;
    
    bool Marshalling(Parcel &parcel) const override;
    static bool Unmarshalling(Parcel &parcel, NetStatsInfo &info);
};
```

---

## 7. 枚举类型

### 7.1 NetCap (网络能力)

**文件**: `interfaces/innerkits/include/netmanager_base_common_defs.h`

```cpp
enum NetCap {
    NET_CAPABILITY_MMS = 0,
    NET_CAPABILITY_SUPL = 1,
    NET_CAPABILITY_DUN = 2,
    NET_CAPABILITY_FOTA = 3,
    NET_CAPABILITY_IMS = 4,
    NET_CAPABILITY_CBS = 5,
    NET_CAPABILITY_WIFI_P2P = 6,
    NET_CAPABILITY_IA = 7,
    NET_CAPABILITY_RCS = 8,
    NET_CAPABILITY_XCAP = 9,
    NET_CAPABILITY_EIMS = 10,
    NET_CAPABILITY_NOT_METERED = 11,
    NET_CAPABILITY_INTERNET = 12,
    NET_CAPABILITY_NOT_RESTRICTED = 13,
    NET_CAPABILITY_TRUSTED = 14,
    NET_CAPABILITY_NOT_VPN = 15,
    NET_CAPABILITY_VALIDATED = 16,
    NET_CAPABILITY_CAPTIVE_PORTAL = 17,
    NET_CAPABILITY_INTERNAL_DEFAULT = 18,
    NET_CAPABILITY_NOT_SUSPENDED = 19,
    NET_CAPABILITY_FOREGROUND = 20,
    NET_CAPABILITY_CHECKING_PRIVATE_DNS = 21,
    NET_CAPABILITY_END,
};
```

### 7.2 NetBearType (网络承载类型)

```cpp
enum NetBearType {
    BEARER_CELLULAR = 0,
    BEARER_WIFI = 1,
    BEARER_BLUETOOTH = 2,
    BEARER_ETHERNET = 3,
    BEARER_VPN = 4,
    BEARER_WIFI_AWARE = 5,
    BEARER_DEFAULT = 6,
};
```

### 7.3 NetUidPolicy (UID 网络策略)

```cpp
enum NetUidPolicy {
    NET_POLICY_NONE = 0,
    NET_POLICY_ALLOW_METERED_BACKGROUND = 1,
    NET_POLICY_TEMPORARY_ALLOW_METERED = 2 << 0,
    NET_POLICY_REJECT_METERED_BACKGROUND = 1 << 1,
    NET_POLICY_ALLOW_ALL = 1 << 2,
    NET_POLICY_REJECT_ALL = 1 << 3,
    NET_POLICY_BULK_DUMP = 1 << 4,
    NET_POLICY_IDLE_ALLOW = 1 << 5,
    NET_POLICY_IDLE_REJECT = 1 << 6,
    NET_POLICY_DEFAULT = 1 << 7,
};
```

---

## 8. 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `NetConnClient` | 稳定 | platformsdk + sasdk |
| `NetPolicyClient` | 稳定 | platformsdk |
| `NetStatsClient` | 稳定 | platformsdk |
| `INetsysService` | 内部 | platformsdk_indirect |
| `NetHandle` | 稳定 | 核心数据结构 |
| `NetLinkInfo` | 稳定 | 核心数据结构 |
| `NetAllCapabilities` | 稳定 | 核心数据结构 |

---

## 9. 使用示例

### 9.1 获取默认网络

```cpp
#include "net_conn_client.h"

using namespace OHOS::NetManagerStandard;

void GetDefaultNetwork() {
    NetHandle netHandle;
    int32_t ret = NetConnClient::GetInstance().GetDefaultNet(netHandle);
    if (ret == NETMANAGER_SUCCESS) {
        int32_t netId = netHandle.GetNetId();
        // 使用 netId
    }
}
```

### 9.2 注册网络监听

```cpp
class NetCallback : public INetConnCallback {
public:
    int32_t OnNetAvailable(const sptr<NetHandle> &netHandle) override {
        // 处理网络可用事件
        return 0;
    }
    
    int32_t OnNetLost(const sptr<NetHandle> &netHandle) override {
        // 处理网络丢失事件
        return 0;
    }
    
    // 实现其他回调...
};

void RegisterCallback() {
    sptr<INetConnCallback> callback = new NetCallback();
    int32_t ret = NetConnClient::GetInstance().RegisterNetConnCallback(callback);
}
```

### 9.3 设置网络策略

```cpp
#include "net_policy_client.h"

void SetPolicy() {
    uint32_t uid = 10001;  // 目标应用 UID
    uint32_t policy = NET_POLICY_REJECT_ALL;  // 拒绝所有网络
    
    int32_t ret = NetPolicyClient::GetInstance().SetPolicyByUid(uid, policy);
    if (ret == NETMANAGER_SUCCESS) {
        // 设置成功
    }
}
```

---

*生成时间: 2025-02-06*
