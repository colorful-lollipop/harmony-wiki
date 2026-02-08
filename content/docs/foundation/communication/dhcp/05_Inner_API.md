# 内部 API 与接口稳定性

## 文档目的

本文档说明 DHCP 组件的内部 C++ API、接口稳定性、依赖方向和可替换点。

---

## 适用范围

- C++ SDK: `frameworks/native/src/`
- 内部接口: `interfaces/inner_api/`
- 组件版本: 3.1.0

---

## 内部 API 分类

### 稳定接口（对外 C++ 接口）

这些接口在 SDK 中实现，保持 ABI 稳定：

| 接口 | 位置 | 稳定性 | 说明 |
|------|------|--------|------|
| `DhcpClient` | `frameworks/native/src/dhcp_client.cpp` | 稳定 | C++ 客户端 SDK |
| `DhcpServer` | `frameworks/native/src/dhcp_server.cpp` | 稳定 | C++ 服务端 SDK |

### IPC 接口（IRemoteBroker）

跨进程通信接口，定义契约但实现可变：

| 接口 | 位置 | 稳定性 | 说明 |
|------|------|--------|------|
| `IDhcpClient` | `frameworks/native/interfaces/i_dhcp_client.h` | 契约稳定 | Client IPC 接口 |
| `IDhcpServer` | `frameworks/native/interfaces/i_dhcp_server.h` | 契约稳定 | Server IPC 接口 |
| `IDhcpClientCallback` | `frameworks/native/interfaces/i_dhcp_client_callback.h` | 契约稳定 | Client 回调接口 |
| `IDhcpServerCallback` | `frameworks/native/interfaces/i_dhcp_server_callback.h` | 契约稳定 | Server 回调接口 |

### 服务实现接口（不稳定）

内部服务实现，可能随时变更：

| 接口 | 位置 | 稳定性 | 说明 |
|------|------|--------|------|
| `DhcpClientServiceImpl` | `services/dhcp_client/include/` | 不稳定 | Client SA 实现 |
| `DhcpServerServiceImpl` | `services/dhcp_server/include/` | 不稳定 | Server SA 实现 |
| `DhcpClientStub` | `services/dhcp_client/include/` | 不稳定 | Client IPC Stub |
| `DhcpServerStub` | `services/dhcp_server/include/` | 不稳定 | Server IPC Stub |

### 工具类接口（内部）

跨模块使用的工具类：

| 接口 | 位置 | 稳定性 | 说明 |
|------|------|--------|------|
| `DhcpPermissionUtils` | `services/utils/include/` | 内部稳定 | 权限检查工具 |
| `DhcpSaManager` | `services/utils/include/` | 内部稳定 | SA 管理工具 |
| `DhcpArpChecker` | `services/utils/include/` | 内部稳定 | ARP 检查工具 |

---

## 接口依赖方向

### 依赖层次

```
┌─────────────────────────────────────────┐
│   应用层                                 │
│   DhcpClient / DhcpServer (C++ SDK)      │
└──────────────┬──────────────────────────┘
               │ 依赖（稳定）
               ▼
┌─────────────────────────────────────────┐
│   IPC 层（契约稳定）                      │
│   IDhcpClient / IDhcpServer              │
└──────────────┬──────────────────────────┘
               │ IPC 调用
               ▼
┌─────────────────────────────────────────┐
│   服务层（实现不稳定）                     │
│   DhcpClientServiceImpl                  │
│   DhcpServerServiceImpl                  │
└──────────────┬──────────────────────────┘
               │ 依赖
               ▼
┌─────────────────────────────────────────┐
│   工具层（内部稳定）                      │
│   DhcpPermissionUtils                   │
│   DhcpSaManager                         │
└─────────────────────────────────────────┘
```

### 依赖规则

1. **单向依赖**: 不存在循环依赖
2. **自顶向下**: 上层依赖下层，下层不依赖上层
3. **接口隔离**: 工具层不依赖具体服务实现

---

## 接口稳定性说明

### 稳定性标记标准

| 稳定性级别 | 定义 | 更新策略 | 版本兼容性 |
|-----------|------|----------|-----------|
| 稳定 | 对外暴露的 SDK 接口 | 仅在主版本变更时修改 | 主版本不兼容 |
| 契约稳定 | IPC 接口定义 | 保持契约，实现可变 | 契约向后兼容 |
| 内部稳定 | 跨模块工具接口 | 谨慎修改，需要协调 | 内部协调 |
| 不稳定 | 服务实现类 | 可随时变更 | 无兼容性要求 |

### 稳定性证据

#### 稳定接口证据
```cpp
// 证据: frameworks/native/src/dhcp_client.cpp
// 标注为对外接口，有 ABI 稳定性要求
class DhcpClient {
public:
    explicit DhcpClient(bool isSa = true);
    ~DhcpClient();

    // 公开 API，保持稳定
    int32_t RegisterDhcpClientCallBack(const std::string &ifname,
                                         const std::shared_ptr<DhcpClientCallBack> &callback);
    int32_t StartDhcpClient(const std::string &ifname, bool bIpv6 = false);
    // ...
};
```

#### 契约稳定接口证据
```cpp
// 证据: frameworks/native/interfaces/i_dhcp_client.h
// IRemoteBroker 接口，定义 IPC 契约
class IDhcpClient : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.Dhcp.IDhcpClient");

    // IPC 契约方法，不可修改
    virtual int32_t RegisterDhcpClientCallBack(const sptr<IRemoteObject> &callback) = 0;
    virtual int32_t StartDhcpClient(const std::string &ifname, bool bIpv6) = 0;
    virtual int32_t StopDhcpClient(const std::string &ifname, bool bIpv6, bool bIpv4) = 0;
    // ...
};
```

#### 不稳定接口证据
```cpp
// 证据: services/dhcp_client/include/dhcp_client_service_impl.h
// 服务实现类，属于内部细节
class DhcpClientServiceImpl : public SystemAbility, public DhcpClientStub {
public:
    // SA 生命周期方法，可变
    void OnStart() override;
    void OnStop() override;
    void OnDump() override;

private:
    // 私有实现细节，完全不稳定
    std::shared_ptr<DhcpClient> dhcpClient_;
    std::mutex clientMutex_;
    // ...
};
```

---

## 可替换点

### 模块替换点

| 替换点 | 当前实现 | 可替换方式 | 影响 |
|--------|----------|-----------|------|
| DHCP 协议栈 | 自研 | 可替换为标准 dhcp/dhcpd 库 | 需适配 IPC 接口 |
| 状态机 | dhcp_client_state_machine.cpp | 可替换为第三方实现 | 需保持状态转换逻辑 |
| 权限检查 | dhcp_permission_utils.cpp | 可替换为其他权限框架 | 需适配 TokenID |
| SA 管理 | dhcp_sa_manager.cpp | 可替换为其他 SA 框架 | 需适配 SystemAbility |

### 配置替换点

| 配置项 | 位置 | 替换方式 |
|--------|------|----------|
| 默认 DNS | dhcp.gni:21-22 | 修改 IPV4_DNS_PRI / IPV4_DNS_SEC |
| 租约时间 | dhcp_s_server.cpp | 配置文件或代码修改 |
| 超时时间 | dhcp_socket.cpp | 配置常量 |

---

## 接口变更影响分析

### 变更 IPC 接口

**影响范围**: 所有调用者（SDK、Proxy、Stub）

**影响等级**: 高（破坏 ABI 兼容性）

**变更步骤**:
1. 修改 `IDhcpClient` 或 `IDhcpServer` 接口
2. 同步更新 Proxy 实现（`dhcp_client_proxy.cpp`）
3. 同步更新 Stub 实现（`dhcp_client_stub.cpp`）
4. 更新命令码（`dhcp_manager_service_ipc_interface_code.h`）
5. 更新版本号

### 变更工具类接口

**影响范围**: 所有依赖该工具的模块

**影响等级**: 中（内部协调）

**变更步骤**:
1. 修改工具类接口
2. 同步更新所有调用者
3. 更新头文件

### 变更服务实现

**影响范围**: 服务内部

**影响等级**: 低（无外部影响）

**变更步骤**:
1. 直接修改实现代码
2. 运行单元测试

---

## 接口使用建议

### 对于模块集成者
- ✅ 优先使用稳定的 C API（`interfaces/kits/c/`）
- ✅ 如需 C++ 接口，使用 SDK 层接口（`DhcpClient`, `DhcpServer`）
- ❌ 不要直接调用服务实现类（`DhcpClientServiceImpl`）
- ❌ 不要依赖内部工具类（`dhcp_permission_utils`）

### 对于服务开发者
- ✅ 保持 IPC 接口契约稳定
- ✅ 实现类可以自由重构
- ✅ 工具类保持内部稳定
- ❌ 不要修改已公开的 SDK 接口

### 对于安全审计员
- ✅ 重点检查 IPC 接口的参数校验
- ✅ 关注权限检查的一致性
- ✅ 审核工具类的安全性
- ❌ 服务实现细节无需过度关注（因为可变）

---

## 接口兼容性检查清单

### 发布新版本时检查

- [ ] IPC 接口未破坏性修改（新增方法向后兼容）
- [ ] C API 函数签名未修改
- [ ] 错误码定义未删除或改变语义
- [ ] 工具类接口修改已通知依赖方
- [ ] 版本号已更新
- [ ] 单元测试已通过

### 接口变更记录

| 版本 | 变更类型 | 变更内容 | 影响模块 |
|------|----------|----------|----------|
| 3.1.0 | 初始版本 | 创建完整接口 | 所有 |

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [03_Architecture](03_Architecture.md) - 架构说明
- [04_C_API_Reference](04_C_API_Reference.md) - C API 参考
- [06_Build_System](06_Build_System.md) - 构建配置
