# 关键调用链

## 文档目的

本文档说明 DHCP 组件的关键调用链，包括 API 入口到核心逻辑的完整路径。

---

## DHCP Client 调用链

### StartDhcpClient 完整调用链

```
应用
  ↓ 调用 C API
[interfaces/kits/c/dhcp_c_api.h] StartDhcpClient(config)
  ↓
[frameworks/native/c_adapter/src/dhcp_c_service.cpp:79]
  StartDhcpClient()
    │
    ├─→ [frameworks/native/src/dhcp_client.cpp] DhcpClient::StartDhcpClient()
    │     │
    │     └─→ [frameworks/native/src/dhcp_client_proxy.cpp] DhcpClientProxy::StartDhcpClient()
    │           │
    │           ├─→ 获取 SA (DhcpSaManager::GetDhcpService())
    │           │     └─→ [services/utils/src/dhcp_sa_manager.cpp:100]
    │           │           DhcpSaManager::GetDhcpService()
    │           │             └─→ samgr::GetSystemAbility(1126)
    │           │
    │           └─→ IPC 调用
    │                 └─→ proxy->SendRequest(CMD, data)
    │                       │
    │                       └─→ Binder 驱动传输
    │
    └─→ [services/dhcp_client/src/dhcp_client_stub.cpp:100]
          DhcpClientStub::OnRemoteRequest()
            │
            ├─→ 验证权限 (DhcpPermissionUtils::VerifyDhcpNetworkPermission())
            │     └─→ [services/utils/src/dhcp_permission_utils.cpp:60]
            │           AccessTokenKit::VerifyAccessToken()
            │
            └─→ [services/dhcp_client/src/dhcp_client_service_impl.cpp:100]
                  DhcpClientServiceImpl::StartDhcpClient()
                    │
                    ├─→ 获取内部 Client (dhcpClient_)
                    │     └─→ [services/dhcp_client/src/dhcp_client.cpp:200]
                    │           创建 DhcpClientImpl
                    │
                    ├─→ [services/dhcp_client/src/dhcp_client_state_machine.cpp:200]
                    │     启动状态机
                    │     │
                    │     ├─→ INIT 状态
                    │     ├─→ 发送 DHCP DISCOVER
                    │     │     └─→ [services/dhcp_client/src/dhcp_socket.cpp:200]
                    │     │           SendDhcpPacket()
                    │     │             └─→ sendto(socket, buf, len, 0, ...)
                    │     │
                    │     └─→ 等待 DHCP OFFER
                    │
                    └─→ 等待结果回调
```

### RegisterDhcpClientCallBack 调用链

```
应用
  ↓
[interfaces/kits/c/dhcp_c_api.h] RegisterDhcpClientCallBack(ifname, event)
  ↓
[frameworks/native/c_adapter/src/dhcp_c_service.cpp:39]
  RegisterDhcpClientCallBack()
    │
    ├─→ 创建回调对象
    │     └─→ [frameworks/native/src/dhcp_event.cpp:50]
    │           DhcpClientCallback
    │
    └─→ IPC 注册
          └─→ [services/dhcp_client/src/dhcp_client_stub.cpp:50]
                DhcpClientStub::RegisterDhcpClientCallBack()
                  │
                  └─→ [services/dhcp_client/src/dhcp_client_service_impl.cpp:50]
                        DhcpClientServiceImpl::RegisterDhcpClientCallBack()
                          │
                          └─→ 保存回调指针 (clientCallback_)
```

### 回调通知调用链

```
[services/dhcp_client/src/dhcp_socket.cpp:200]
  接收到 DHCP ACK
    │
    └─→ [services/dhcp_client/src/dhcp_client_state_machine.cpp:300]
          处理 DHCP ACK
            │
            ├─→ 解析 IP、网关、DNS
            │     └─→ [services/dhcp_client/src/dhcp_result.cpp:80]
            │           解析 DhcpResult
            │
            └─→ 触发回调
                  └─→ [frameworks/native/src/dhcp_event.cpp:100]
                        回调 → ClientCallBack::OnIpSuccessChanged()
                          │
                          └─→ 应用层回调函数
```

---

## DHCP Server 调用链

### StartDhcpServer 完整调用链

```
应用
  ↓
[interfaces/kits/c/dhcp_c_api.h] StartDhcpServer(ifname)
  ↓
[frameworks/native/c_adapter/src/dhcp_c_service.cpp:137]
  StartDhcpServer()
    │
    ├─→ [frameworks/native/src/dhcp_server.cpp] DhcpServer::StartDhcpServer()
    │     │
    │     └─→ [frameworks/native/src/dhcp_server_proxy.cpp]
    │           DhcpServerProxy::StartDhcpServer()
    │                 │
    │                 ├─→ 获取 SA (DhcpSaManager::GetDhcpService())
    │                 │     └─→ samgr::GetSystemAbility(1127)
    │                 │
    │                 └─→ IPC 调用
    │
    └─→ [services/dhcp_server/src/dhcp_server_stub.cpp:50]
          DhcpServerStub::OnRemoteRequest()
            │
            ├─→ 验证权限
            └─→ [services/dhcp_server/src/dhcp_server_service_impl.cpp:50]
                  DhcpServerServiceImpl::StartDhcpServer()
                    │
                    ├─→ [services/dhcp_server/src/dhcp_s_server.cpp:300]
                    │     DhcpSServer::StartDhcpServer()
                    │       │
                    │       ├─→ 初始化地址池
                    │       │     └─→ [services/dhcp_server/src/dhcp_address_pool.cpp:100]
                    │       │           DhcpAddressPool::Init()
                    │       │
                    │       ├─→ 创建 Socket
                    │       │     └─→ socket(AF_INET, SOCK_DGRAM, 0)
                    │       │
                    │       └─→ 绑定端口 67
                    │             └─→ bind(sockfd, ...)
                    │
                    └─→ 启动 DHCPd 线程
```

### DHCP 请求处理调用链

```
[services/dhcp_server/src/dhcp_s_server.cpp:500]
  接收到 DHCP DISCOVER
    │
    ├─→ 验证 MAC 地址
    │     └─→ [services/dhcp_server/src/dhcp_arp_checker.cpp:50]
    │           DhcpArpChecker::CheckMacConflict()
    │
    ├─→ 分配 IP
    │     └─→ [services/dhcp_server/src/dhcp_address_pool.cpp:200]
    │           DhcpAddressPool::AllocateIp()
    │             │
    │             └─→ 检查 IP 冲突
    │                   └─→ ARP 检查
    │
    ├─→ 创建租约
    │     └─→ [services/dhcp_server/src/dhcp_binding.cpp:100]
    │           DhcpBinding::Create()
    │
    ├─→ 发送 DHCP OFFER
    │     └─→ [services/dhcp_server/src/dhcp_socket.cpp:50]
    │           SendDhcpPacket()
    │
    └─→ 等待 DHCP REQUEST
```

### GetDhcpClientInfos 调用链

```
应用
  ↓
[interfaces/kits/c/dhcp_c_api.h] GetDhcpClientInfos(ifname, staNumber, staInfo, staSize)
  ↓
[frameworks/native/c_adapter/src/dhcp_c_service.cpp:279]
  GetDhcpClientInfos()
    │
    └─→ IPC 调用
          └─→ [services/dhcp_server/src/dhcp_server_service_impl.cpp:600]
                DhcpServerServiceImpl::GetDhcpClientInfos()
                  │
                  └─→ [services/dhcp_server/src/dhcp_s_server.cpp:600]
                        获取租约列表
                          │
                          └─→ [services/dhcp_server/src/dhcp_binding.cpp:200]
                                DhcpBinding::GetAllBindings()
```

---

## 权限检查调用链

### IPC 请求权限检查

```
[services/dhcp_client/src/dhcp_client_stub.cpp:100]
  OnRemoteRequest()
    │
    ├─→ 获取调用者 TokenID
    │     └─→ IPCSkeleton::GetCallingTokenID()
    │
    ├─→ 检查是否为原生进程
    │     └─→ [services/utils/src/dhcp_permission_utils.cpp:50]
    │           DhcpPermissionUtils::VerifyIsNativeProcess()
    │             │
    │             ├─→ AccessTokenKit::GetTokenTypeFlag(tokenId)
    │             └─→ return (type == TOKEN_NATIVE)
    │
    └─→ 检查网络权限
          └─→ [services/utils/src/dhcp_permission_utils.cpp:60]
                DhcpPermissionUtils::VerifyDhcpNetworkPermission()
                  │
                  ├─→ 获取权限名称 ("ohos.permission.NETWORK_DHCP")
                  ├─→ AccessTokenKit::VerifyAccessToken(tokenId, permission)
                  └─→ return (result == PERMISSION_GRANTED)
```

---

## SA 管理调用链

### 获取 SA 实例

```
[frameworks/native/src/dhcp_client_proxy.cpp:100]
  获取 Client SA
    │
    └─→ [services/utils/src/dhcp_sa_manager.cpp:100]
          DhcpSaManager::GetDhcpService()
            │
            ├─→ 检查缓存 (static sa_)
            │
            ├─→ 如未缓存，获取 SA
            │     └─→ samgr::GetSystemAbility(1126)
            │           │
            │           └─→ 返回 IDhcpClient Proxy
            │
            └─→ 返回 SA 实例
```

### SA 生命周期

```
系统启动
  ↓
[SA Manager] 读取 /system/profile/1126.json
  ↓
注册 SA (不启动, run-on-create=false)
  ↓
应用调用 API
  ↓
[SA Manager] 按需启动 SA
  ↓
[services/dhcp_client/src/dhcp_client_service_impl.cpp:50]
  DhcpClientServiceImpl::OnStart()
    │
    ├─→ 初始化 Client (dhcpClient_)
    ├─→ 注册死亡通知
    │     └─→ DhcpClientDeathRecipient
    └─→ 发布就绪状态 (Publish())
  ↓
SA 运行
  ↓
应用调用 StopDhcpClient / 进程退出
  ↓
[services/dhcp_client/src/dhcp_client_service_impl.cpp:100]
  DhcpClientServiceImpl::OnStop()
    │
    ├─→ 停止 Client
    ├─→ 清理资源
    └─→ 取消发布
```

---

## 工具类调用链

### ARP 检查

```
[services/dhcp_server/src/dhcp_address_pool.cpp:300]
  检查 IP 冲突
    │
    └─→ [services/utils/src/dhcp_arp_checker.cpp:50]
          DhcpArpChecker::CheckIpConflict(ip, ifname)
            │
            ├─→ 发送 ARP 请求
            │     └─→ socket(AF_PACKET, SOCK_RAW, ...)
            │
            ├─→ 接收 ARP 响应
            │     └─→ recvfrom(sockfd, ...)
            │
            └─→ 检查 MAC 地址
                  └─→ 比较 MAC
```

### 定时器

```
[services/dhcp_server/src/dhcp_s_server.cpp:700]
  设置租约超时
    │
    └─→ [services/utils/src/dhcp_system_timer.cpp:50]
          DhcpSystemTimer::StartTimer()
            │
            ├─→ 创建定时器线程
            │     └─→ [services/utils/src/dhcp_thread.cpp:80]
            │           CreateThread()
            │
            └─→ 注册回调
                  └─→ 定时到期触发回调
```

---

## 线程模型调用链

### Client 线程

```
主线程 (SA 线程)
  ├── IPC 请求处理
  │     └─→ OnRemoteRequest()
  │
  ├── SA 生命周期
  │     ├── OnStart()
  │     └── OnStop()
  │
  └─→ 回调注册/注销

工作线程池 (DhcpThread)
  ├── DHCP 包处理
  │     └─→ SendDhcpPacket()
  │
  └─→ 回调通知
        └─→ 回调函数执行

IPv6 处理线程
  └─→ DHCPv6 地址获取
        └─→ [services/dhcp_client/src/dhcp_ipv6_client.cpp:150]

定时器线程
  └─→ 租约超时、重试
        └─→ [services/utils/src/dhcp_system_timer.cpp:50]
```

### Server 线程

```
主线程 (SA 线程)
  ├── IPC 请求处理
  ├── SA 生命周期
  └─→ 地址池管理

DHCPd 工作线程
  ├── DHCP 包接收
  ├── DHCP 包发送
  └─→ 租约管理

ARP 检查线程
  └─→ IP 冲突检测
        └─→ [services/utils/src/dhcp_arp_checker.cpp:50]

定时器线程
  └─→ 租约过期清理
```

---

## 错误处理调用链

### 错误码转换

```
[services/dhcp_client/src/dhcp_client_service_impl.cpp:100]
  内部错误 (ErrCode)
    │
    └─→ [frameworks/native/c_adapter/src/dhcp_c_utils.cpp:20]
          ErrCodeMap[errCode]
            │
            └─→ 转换为 DhcpErrorCode
                  │
                  ├─→ DHCP_E_SUCCESS → DHCP_SUCCESS
                  ├─→ DHCP_E_FAILED → DHCP_FAILED
                  ├─→ DHCP_E_INVALID_PARAM → DHCP_INVALID_PARAM
                  ├─→ DHCP_E_NON_SYSTEMAPP → DHCP_NON_SYSTEMAPP
                  ├─→ DHCP_E_PERMISSION_DENIED → DHCP_PERMISSION_DENIED
                  ├─→ DHCP_E_INVALID_CONFIG → DHCP_INVALID_CONFIG
                  └─→ DHCP_E_UNKNOWN → DHCP_UNKNOWN_ERROR
```

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [03_Architecture](03_Architecture.md) - 架构说明
- [04_C_API_Reference](04_C_API_Reference.md) - API 文档
- [05_Inner_API](05_Inner_API.md) - 内部接口
