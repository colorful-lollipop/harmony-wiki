# C API 参考文档

## 文档目的

本文档详细说明 DHCP 组件对外暴露的 C 语言 API，包括 17 个导出方法、参数、返回值、错误码和权限要求。

---

## 适用范围

- 头文件: `interfaces/kits/c/dhcp_c_api.h`
- 实现: `frameworks/native/c_adapter/src/dhcp_c_service.cpp`
- 组件版本: 3.1.0

---

## 重要说明

**本项目没有直接的 N-API 绑定**

- 使用 C API（本节内容）供原生 C/C++ 应用调用
- 通过 SystemAbility 框架提供跨进程服务
- JavaScript API 在 `communication_wifi` 仓库中通过 N-API 封装本服务

---

## API 清单表

| API | 类别 | 权限要求 | 异步方式 |
|-----|------|----------|----------|
| `RegisterDhcpClientCallBack` | Client | NETWORK_DHCP | 回调 |
| `RegisterDhcpClientReportCallBack` | Client | NETWORK_DHCP | 回调 |
| `StartDhcpClient` | Client | NETWORK_DHCP | 回调 |
| `DealWifiDhcpCache` | Client | NETWORK_DHCP | 同步 |
| `StopDhcpClient` | Client | NETWORK_DHCP | 同步 |
| `StopDhcpdClientSa` | Client | NETWORK_DHCP | 同步 |
| `RegisterDhcpServerCallBack` | Server | NETWORK_DHCP | 回调 |
| `StartDhcpServer` | Server | NETWORK_DHCP | 同步 |
| `StopDhcpServer` | Server | NETWORK_DHCP | 同步 |
| `SetDhcpRange` | Server | NETWORK_DHCP | 同步 |
| `SetDhcpName` | Server | NETWORK_DHCP | 同步 |
| `PutDhcpRange` | Server | NETWORK_DHCP | 同步 |
| `RemoveAllDhcpRange` | Server | NETWORK_DHCP | 同步 |
| `RemoveDhcpRange` | Server | NETWORK_DHCP | 同步 |
| `GetDhcpClientInfos` | Server | NETWORK_DHCP | 同步 |
| `UpdateLeasesTime` | Server | NETWORK_DHCP | 同步 |
| `StopDhcpdServerSa` | Server | NETWORK_DHCP | 同步 |

---

## 错误码定义

### C API 错误码

```c
// 证据: interfaces/kits/c/dhcp_error_code.h:10-20
typedef enum {
    DHCP_SUCCESS = 0,              /* 成功 */
    DHCP_FAILED  = -1,             /* 失败 */
    DHCP_INVALID_PARAM = -2,       /* 无效参数 */
    DHCP_NON_SYSTEMAPP = -3,       /* 非系统应用拒绝 */
    DHCP_PERMISSION_DENIED = -4,   /* 权限拒绝 */
    DHCP_INVALID_CONFIG = -5,      /* 无效配置 */
    DHCP_UNKNOWN_ERROR             /* 未知错误 */
} DhcpErrorCode;
```

### 错误码映射

| C API 错误码 | C++ 内部码 | 含义 |
|-------------|-----------|------|
| DHCP_SUCCESS | DHCP_E_SUCCESS | 操作成功 |
| DHCP_FAILED | DHCP_E_FAILED | 操作失败 |
| DHCP_INVALID_PARAM | DHCP_E_INVALID_PARAM | 参数无效 |
| DHCP_NON_SYSTEMAPP | DHCP_E_NON_SYSTEMAPP | 非系统应用 |
| DHCP_PERMISSION_DENIED | DHCP_E_PERMISSION_DENIED | 权限不足 |
| DHCP_INVALID_CONFIG | DHCP_E_INVALID_CONFIG | 配置无效 |
| DHCP_UNKNOWN_ERROR | DHCP_E_UNKNOWN | 未知错误 |

证据: `frameworks/native/c_adapter/src/dhcp_c_utils.cpp:20-40`

---

## DHCP Client API

### 1. RegisterDhcpClientCallBack

**函数签名**:
```c
// 证据: interfaces/kits/c/dhcp_c_api.h:30
DhcpErrorCode RegisterDhcpClientCallBack(const char *ifname,
                                         const ClientCallBack *event);
```

**参数**:
| 参数 | 类型 | 说明 | 校验 |
|------|------|------|------|
| ifname | const char* | 网络接口名称（如 "wlan0"） | CHECK_PTR_RETURN |
| event | const ClientCallBack* | 回调函数指针 | CHECK_PTR_RETURN |

**返回值**:
- `DHCP_SUCCESS` - 注册成功
- `DHCP_INVALID_PARAM` - 参数为空
- `DHCP_PERMISSION_DENIED` - 权限不足
- `DHCP_FAILED` - 注册失败

**权限要求**:
- 必须持有 `ohos.permission.NETWORK_DHCP`
- 必须为原生进程（TOKEN_NATIVE）

**实现位置**: `frameworks/native/c_adapter/src/dhcp_c_service.cpp:39`

**调用链**:
```
C API → DhcpClient::RegisterDhcpClientCallBack() → IPC → Client SA
```

**示例**:
```c
ClientCallBack cb;
cb.OnIpSuccessChanged = OnSuccess;
cb.OnIpFailChanged = OnFail;

DhcpErrorCode ret = RegisterDhcpClientCallBack("wlan0", &cb);
if (ret != DHCP_SUCCESS) {
    // 处理错误
}
```

---

### 2. StartDhcpClient

**函数签名**:
```c
// 证据: interfaces/kits/c/dhcp_c_api.h:45
DhcpErrorCode StartDhcpClient(const RouterConfig &config);
```

**参数**:
```c
// 证据: interfaces/kits/c/dhcp_result_event.h:30-50
typedef struct RouterConfig {
    char ifname[32];              /* 接口名 */
    char bssid[18];               /* BSSID */
    bool prohibitUseCacheIp;     /* 禁止使用缓存IP */
    bool bIpv6;                   /* 启用IPv6 */
    bool bSpecificNetwork;       /* 特定网络 */
    bool isStaticIpv4;           /* 静态IPv4 */
    bool bIpv4;                   /* 启用IPv4 */
} RouterConfig;
```

**返回值**:
- `DHCP_SUCCESS` - 启动成功
- `DHCP_INVALID_PARAM` - 参数无效
- `DHCP_PERMISSION_DENIED` - 权限不足
- `DHCP_FAILED` - 启动失败

**权限要求**:
- 必须持有 `ohos.permission.NETWORK_DHCP`

**实现位置**: `frameworks/native/c_adapter/src/dhcp_c_service.cpp:79`

**调用链**:
```
C API → DhcpClient::StartDhcpClient() → IPC → Client SA → State Machine
```

**异步回调**:
- 成功: `ClientCallBack::OnIpSuccessChanged()`
- 失败: `ClientCallBack::OnIpFailChanged()`

---

### 3. StopDhcpClient

**函数签名**:
```c
// 证据: interfaces/kits/c/dhcp_c_api.h:60
DhcpErrorCode StopDhcpClient(const char *ifname, bool bIpv6, bool bIpv4);
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| ifname | const char* | 网络接口名称 |
| bIpv6 | bool | 是否停止 IPv6 |
| bIpv4 | bool | 是否停止 IPv4 |

**返回值**:
- `DHCP_SUCCESS` - 停止成功
- `DHCP_INVALID_PARAM` - 参数无效
- `DHCP_FAILED` - 停止失败

**实现位置**: `frameworks/native/c_adapter/src/dhcp_c_service.cpp:104`

---

### 4. DealWifiDhcpCache

**函数签名**:
```c
// 证据: interfaces/kits/c/dhcp_c_api.h:75
DhcpErrorCode DealWifiDhcpCache(int32_t cmd,
                                const IpCacheInfo &ipCacheInfo);
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| cmd | int32_t | 命令类型（保存/删除缓存） |
| ipCacheInfo | const IpCacheInfo& | IP 缓存信息 |

**用途**: WiFi 场景下管理 IP 缓存，加快连接速度

**实现位置**: `frameworks/native/c_adapter/src/dhcp_c_service.cpp:93`

---

## DHCP Server API

### 5. StartDhcpServer

**函数签名**:
```c
// 证据: interfaces/kits/c/dhcp_c_api.h:90
DhcpErrorCode StartDhcpServer(const char *ifname);
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| ifname | const char* | 网络接口名称 |

**返回值**:
- `DHCP_SUCCESS` - 启动成功
- `DHCP_INVALID_PARAM` - 参数无效
- `DHCP_PERMISSION_DENIED` - 权限不足
- `DHCP_FAILED` - 启动失败

**权限要求**:
- 必须持有 `ohos.permission.NETWORK_DHCP`

**实现位置**: `frameworks/native/c_adapter/src/dhcp_c_service.cpp:137`

---

### 6. SetDhcpRange

**函数签名**:
```c
// 证据: interfaces/kits/c/dhcp_c_api.h:105
DhcpErrorCode SetDhcpRange(const char *ifname, const DhcpRange *range);
```

**参数**:
```c
// 证据: interfaces/kits/c/dhcp_result_event.h:60-70
typedef struct DhcpRange {
    char tagName[64];             /* 标签名称 */
    char strStartIp[128];         /* 起始IP */
    char strEndIp[128];           /* 结束IP */
    uint32_t subnetMask;         /* 子网掩码 */
} DhcpRange;
```

**返回值**:
- `DHCP_SUCCESS` - 设置成功
- `DHCP_INVALID_PARAM` - 参数无效
- `DHCP_PERMISSION_DENIED` - 权限不足
- `DHCP_FAILED` - 设置失败

**实现位置**: `frameworks/native/c_adapter/src/dhcp_c_service.cpp:168`

---

### 7. GetDhcpClientInfos

**函数签名**:
```c
// 证据: interfaces/kits/c/dhcp_c_api.h:120
DhcpErrorCode GetDhcpClientInfos(const char *ifname,
                                  int staNumber,
                                  DhcpStationInfo *staInfo,
                                  int *staSize);
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| ifname | const char* | 网络接口名称 |
| staNumber | int | 期望获取的客户端数量 |
| staInfo | DhcpStationInfo* | 客户端信息输出数组 |
| staSize | int* | 实际获取的数量 |

**返回值**:
- `DHCP_SUCCESS` - 查询成功
- `DHCP_INVALID_PARAM` - 参数无效
- `DHCP_FAILED` - 查询失败

**用途**: 查询当前连接的 DHCP 客户端信息（租约）

**实现位置**: `frameworks/native/c_adapter/src/dhcp_c_service.cpp:279`

---

## 回调机制

### ClientCallBack

```c
// 证据: interfaces/kits/c/dhcp_result_event.h:80-90
typedef struct {
    void (*OnIpSuccessChanged)(int status,
                               const char *ifname,
                               DhcpResult *result);
    void (*OnIpFailChanged)(int status,
                            const char *ifname,
                            const char *reason);
} ClientCallBack;
```

### DhcpResult 结构

```c
// 证据: interfaces/kits/c/dhcp_result_event.h:100-130
typedef struct {
    int iptype;                          /* 0-ipv4, 1-ipv6 */
    bool isOptSuc;                       /* 获取结果状态 */
    char strOptClientId[128];            /* 客户端 IP */
    char strOptServerId[128];            /* DHCP 服务器 IP */
    char strOptSubnet[128];              /* 子网掩码 */
    char strOptDns1[128];                /* DNS 服务器1 */
    char strOptDns2[128];                /* DNS 服务器2 */
    char strOptRouter1[128];             /* 路由器1 */
    char strOptRouter2[128];             /* 路由器2 */
    uint32_t uOptLeasetime;              /* 租约时间(秒) */
    DnsList dnsList;                     /* DNS 列表 */
    AddrList addrList;                    /* 地址列表 */
    Ipv6LifeTime ipv6LifeTime;            /* IPv6 生命周期 */
} DhcpResult;
```

---

## 参数校验

### 校验宏定义

```cpp
// 证据: frameworks/native/c_adapter/inc/dhcp_c_utils.h:10-20
#define CHECK_PTR_RETURN(ptr, retValue)             \
    if ((ptr) == nullptr) {                         \
        DHCP_LOGE("Error: "#ptr" is null!");       \
        return retValue;                            \
    }
```

### 校验点统计

| API | 校验点 | 校验内容 |
|-----|--------|----------|
| RegisterDhcpClientCallBack | 2 | ifname, event |
| StartDhcpClient | 1 | clientPtr |
| StopDhcpClient | 3 | ifname, clientPtr, callback |
| StartDhcpServer | 2 | ifname, serverPtr |
| SetDhcpRange | 3 | ifname, range, serverPtr |
| GetDhcpClientInfos | 4 | ifname, staInfo, staSize, serverPtr |

---

## 权限检查

### 检查机制

所有公开 API 都执行双重权限检查：

1. **原生进程检查**:
   ```cpp
   // 证据: services/utils/src/dhcp_permission_utils.cpp:50
   bool DhcpPermissionUtils::VerifyIsNativeProcess() {
       uint32_t tokenId = IPCSkeleton::GetCallingTokenID();
       ATokenTypeEnum callingType = AccessTokenKit::GetTokenTypeFlag(tokenId);
       return callingType == TOKEN_NATIVE;
   }
   ```

2. **网络权限检查**:
   ```cpp
   // 证据: services/utils/src/dhcp_permission_utils.cpp:60
   bool DhcpPermissionUtils::VerifyDhcpNetworkPermission(const std::string &permissionName) {
       int result = AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
       return result == PermissionState::PERMISSION_GRANTED;
   }
   ```

### 权限检查位置

| 服务 | 方法 | 证据 |
|------|------|------|
| DhcpClientServiceImpl | 所有公开方法 | `dhcp_client_service_impl.cpp:50-200` |
| DhcpServerServiceImpl | 所有公开方法 | `dhcp_server_service_impl.cpp:50-200` |

---

## 调用链总结

### Client API 完整调用链

```
应用
  ↓
dhcp_c_api.h (C API)
  ↓
dhcp_c_service.cpp (C 适配层)
  ↓
dhcp_client_proxy.cpp (IPC Proxy)
  ↓
Binder IPC 驱动
  ↓
dhcp_client_stub.cpp (IPC Stub)
  ↓
dhcp_client_service_impl.cpp (SA 实现)
  ↓
dhcp_client_state_machine.cpp (状态机)
  ↓
dhcp_socket.cpp (网络通信)
```

### Server API 完整调用链

```
应用
  ↓
dhcp_c_api.h (C API)
  ↓
dhcp_c_service.cpp (C 适配层)
  ↓
dhcp_server_proxy.cpp (IPC Proxy)
  ↓
Binder IPC 驱动
  ↓
dhcp_server_stub.cpp (IPC Stub)
  ↓
dhcp_server_service_impl.cpp (SA 实现)
  ↓
dhcp_s_server.cpp (Server 核心)
  ↓
dhcp_address_pool.cpp (地址池管理)
```

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [03_Architecture](03_Architecture.md) - 架构说明
- [05_Inner_API](05_Inner_API.md) - 内部 API
- [08_Security_Review](08_Security_Review.md) - 权限模型
