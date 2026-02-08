# 安全风险评审

## 1. 威胁模型

### 1.1 攻击面概览

```
                    ┌─────────────────────────────────────┐
                    │           外部输入源                │
                    │  ┌─────────┬─────────┬─────────┐  │
                    │  │  JS API │   IPC   │  网络   │  │
                    │  └────┬────┴────┬────┴────┬────┘  │
                    └───────┼─────────┼─────────┼────────┘
                            │         │         │
                            ▼         ▼         ▼
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐│
│  │  N-API 层     │  │  IPC 接口层   │  │  网络数据层           ││
│  │  (参数解析)   │  │  (Stub/Proxy) │  │  (DNS/HTTP/Netlink)   ││
│  └───────┬───────┘  └───────┬───────┘  └───────────┬───────────┘│
└──────────┼─────────────────┼──────────────────────┼────────────┘
           │                 │                      │
           ▼                 ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                      敏感操作层                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │  权限检查   │  │  文件操作   │  │  内核通信               │ │
│  │  策略下发   │  │  (配置)     │  │  (iptables/BPF/Netlink) │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击向量分类

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| N-API JS 接口 | 中 | 外部应用通过 JS API 输入数据 |
| IPC 调用 | 中 | 跨进程通信接口 |
| 配置文件 | 低 | 本地配置文件解析 |
| 网络数据 | 高 | DNS 响应、HTTP 代理配置、PAC 脚本 |
| 内核接口 | 高 | Netlink、iptables、BPF |

---

## 2. 安全检查点清单

### 2.1 权限检查

| 位置 | 代码片段 | 说明 |
|------|----------|------|
| `utils/common_utils/src/netmanager_base_permission.cpp:33-47` | `AccessTokenKit::VerifyAccessToken` | 统一权限检查入口 |
| `utils/common_utils/src/netmanager_base_permission.cpp:74-86` | `IsSystemCaller()` | 系统调用者检查 |
| `services/netpolicymanager/src/stub/net_policy_service_stub.cpp:40-97` | 权限映射表 | IPC 接口权限映射 |
| `services/netstatsmanager/src/net_stats_service.cpp:763` | `CheckPermission(CONNECTIVITY_INTERNAL)` | 统计服务权限检查 |

**权限检查模式**:
```cpp
// 标准权限检查
int32_t ret = NetManagerPermission::CheckPermission(Permission::MANAGE_NET_STRATEGY);
if (ret != NETMANAGER_SUCCESS) {
    NETMANAGER_BASE_LOGE("Permission denied");
    return NETMANAGER_ERROR;
}
```

### 2.2 UID/GID 校验

| 位置 | 代码片段 | 说明 |
|------|----------|------|
| `services/netpolicymanager/src/stub/net_policy_service_stub.cpp:236` | `IPCSkeleton::GetCallingUid()` | 获取调用者 UID |
| `services/netconnmanager/src/net_conn_service.cpp:299` | UID 校验 | 检查操作权限 |
| `services/netmanagernative/src/netsys_native_service_stub.cpp:468` | UID 获取与校验 | Native 服务 UID 检查 |

**调用示例**:
```cpp
// 记录调用者身份
pid_t callingPid = IPCSkeleton::GetCallingPid();
uint32_t callingUid = IPCSkeleton::GetCallingUid();
NETMANAGER_BASE_LOGI("Calling pid: %{public}d, uid: %{public}d", callingPid, callingUid);
```

### 2.3 输入验证

#### 2.3.1 空指针检查

| 位置 | 代码片段 | 说明 |
|------|----------|------|
| `services/netsyscontroller/src/netsys_controller.cpp` | 多处 `if (netsysService_ == nullptr)` | 服务实例检查 |
| `services/netmanagernative/src/netsys_native_service.cpp` | 多处 nullptr 检查 | 回调/代理检查 |
| `services/netmanagernative/src/netsys_native_service_stub.cpp` | 边界检查 | 数组大小校验 |

#### 2.3.2 边界检查

| 位置 | 代码片段 | 限制值 |
|------|----------|--------|
| `services/netmanagernative/src/netsys_native_service_stub.cpp:1121` | `size < 0 \|\| size > MAX_VNIC_UID_ARRAY_SIZE` | MAX_VNIC_UID_ARRAY_SIZE |
| `services/netmanagernative/src/netsys_native_service_stub.cpp:1236` | `index > MAX_DNS_CONFIG_SIZE` | MAX_DNS_CONFIG_SIZE |
| `services/netmanagernative/src/netsys_native_service_stub.cpp:1831` | `count >= MAX_ROUTE_TABLE_SIZE` | MAX_ROUTE_TABLE_SIZE |
| `services/netmanagernative/src/netsys_native_service_stub.cpp:2272` | `sharingTypeIsOn.size() > MAX_SHARING_TYPE_SIZE` | MAX_SHARING_TYPE_SIZE |

---

## 3. 可被利用点分析

### 3.1 高危风险点

#### 3.1.1 DNS 解析 - 潜在 DNS 劫持风险

**位置**: `services/netmanagernative/src/netsys/dnsresolv/`

**问题描述**:
- DNS 响应来自外部网络，可能被污染
- 解析结果用于网络连接决策

**证据**:
```cpp
// services/netmanagernative/src/netsys/dnsresolv/dns_resolv_listen.cpp
// DNS 响应解析后直接使用，需确保 DNSSEC 或 DNS over TLS 已启用
```

**影响**: 中间人攻击、流量劫持

**修复建议**:
1. 启用 DNS over TLS (DoT) 或 DNS over HTTPS (DoH)
2. 验证 DNS 响应完整性
3. 限制 DNS 缓存时间

#### 3.1.2 PAC 代理脚本执行

**位置**: `services/netconnmanager/src/pac_functions.cpp`

**问题描述**:
- PAC (Proxy Auto-Configuration) 脚本包含 JavaScript 代码
- 来自外部网络的 PAC 脚本可能包含恶意代码

**证据**:
```cpp
// services/netconnmanager/src/pac_functions.cpp:94-973
// PAC 脚本解析和执行
```

**影响**: 代码执行、信息泄露

**修复建议**:
1. 沙箱化 PAC 脚本执行环境
2. 限制 PAC 脚本可访问的 API
3. 验证 PAC URL 的 HTTPS 证书

### 3.2 中危风险点

#### 3.2.1 文件路径遍历风险

**位置**: 多处配置文件操作

**问题描述**:
- 部分文件操作使用拼接路径
- 如果路径组件来自外部输入，可能存在遍历风险

**证据**:
```cpp
// services/netmanagernative/src/manager/interface_manager.cpp:97
open(realPath.c_str(), ...)

// services/netpolicymanager/src/net_access_policy_config.cpp:164
stat(realPath, &st)
```

**影响**: 越权文件访问

**修复建议**:
1. 使用 `realpath()` 规范化路径
2. 验证路径前缀在白名单中
3. 避免直接使用用户输入构造路径

#### 3.2.2 内存分配返回值检查

**位置**: `services/netconnmanager/src/pac_functions.cpp:94-973`

**问题描述**:
- 多处 malloc 调用，部分未检查返回值

**影响**: 空指针解引用导致崩溃

**修复建议**:
1. 所有 malloc 后添加 NULL 检查
2. 使用智能指针管理内存
3. 使用 `std::vector` 替代裸数组

#### 3.2.3 数组越界访问 (memcpy_s 返回值)

**位置**: `services/netmanagernative/bpf/src/bpf_netfirewall.cpp:507-700`

**问题描述**:
- 多处 `memcpy_s` 调用，返回值未检查
- 虽然使用安全函数，但仍需确认复制成功

**影响**: 数据截断、逻辑错误

**修复建议**:
1. 检查 `memcpy_s` 返回值
2. 使用封装的安全拷贝函数

### 3.3 低危风险点

#### 3.3.1 系统调用返回值处理

**位置**: `services/netmanagernative/src/netsys/clatd.cpp:301`

**问题描述**:
- read 系统调用返回值未充分校验

**影响**: 数据处理不完整

**修复建议**:
1. 检查 read 返回值是否为 -1 或 0
2. 处理部分读取情况

#### 3.3.2 fork() 后处理

**位置**: `utils/common_utils/src/netmanager_base_common_utils.cpp:704`

**问题描述**:
- fork() 后子进程安全处理可加强

**影响**: 子进程继承不必要的资源

**修复建议**:
1. 使用 `pthread_atfork()` 清理锁
2. 关闭不必要的文件描述符

---

## 4. 安全机制

### 4.1 已实施的安全措施

| 机制 | 实现 | 说明 |
|------|------|------|
| **CFI** | 所有共享库 | 控制流完整性保护 |
| **PAC-RET** | 所有共享库 | 分支保护 |
| **UBSan** | 所有共享库 | 未定义行为检测 |
| **边界检查** | `netmanager_base_config.gni:68` | `-D_FORTIFY_SOURCE=2` |
| **符号隐藏** | 所有共享库 | `-fvisibility=hidden` |
| **安全字符串函数** | 全仓库 | strcpy_s/strncpy_s/memcpy_s |
| **权限检查** | IPC 入口 | AccessTokenKit |
| **UID 记录** | IPC 入口 | 审计日志 |

### 4.2 IPC 安全

| 检查点 | 实现 | 说明 |
|--------|------|------|
| 接口描述符校验 | `ReadInterfaceToken()` | 防止接口混淆攻击 |
| 权限映射 | Stub 层 | 每个 IPC 命令对应权限 |
| UID 获取 | `IPCSkeleton::GetCallingUid()` | 验证调用者身份 |

---

## 5. 信任边界

### 5.1 信任级别

| 层级 | 信任级别 | 组件 |
|------|----------|------|
| 内核层 | 最高 | Kernel、BPF、eBPF |
| 系统服务层 | 高 | NetManagerNative (netsysnative 进程) |
| 管理服务层 | 中 | NetConn/Policy/Stats Service (netmanager 进程) |
| 框架接口层 | 中 | InnerKits、N-API |
| 应用层 | 低 | 第三方应用 |

### 5.2 数据流信任边界

```
不可信输入              信任边界               敏感操作
    │                      │                      │
    │  JS API 调用         │                      │
    ├──────────────────────┤                      │
    │                      │  N-API 参数解析      │
    │                      ├──────────────────────┤
    │                      │                      │
    │                      │  IPC 调用            │
    │                      ├──────────────────────┤
    │                      │                      │  权限检查
    │                      │                      ├──────────┐
    │                      │                      │          │
    │                      │                      │  网络配置 │
    │                      │                      ├──────────┤
    │                      │                      │          │
    │                      │                      │  内核操作 │
    │                      │                      ├──────────┘
```

---

## 6. 修复建议优先级

### 高优先级

1. **DNS 安全增强**
   - 启用 DNS over TLS/HTTPS
   - 实现 DNS 响应验证

2. **PAC 沙箱化**
   - 限制 PAC 脚本执行环境
   - 禁止访问敏感 API

### 中优先级

3. **路径遍历防护**
   - 规范化所有文件路径
   - 添加路径前缀验证

4. **内存安全检查**
   - 添加 malloc 返回值检查
   - 检查 memcpy_s 返回值

### 低优先级

5. **系统调用处理**
   - 完善 read/write 返回值处理
   - fork() 后资源清理

---

## 7. 安全检查范围与局限性

### 7.1 已检查范围

- 权限检查点: 全仓库
- UID/GID 校验: Stub 层全面覆盖
- 输入验证: N-API 和 IPC 层
- 文件操作: 配置文件相关代码
- 字符串处理: 安全函数使用情况
- 内存操作: 关键路径检查

### 7.2 局限性说明

1. **测试代码未覆盖**: 本评审忽略 `test/` 目录
2. **Fuzz 测试未分析**: fuzztest 目录未深入分析
3. **动态行为**: 静态分析无法覆盖运行时行为
4. **依赖库**: 外部依赖库的安全依赖其自身维护

---

## 8. 安全审计建议

### 8.1 定期审计项

| 审计项 | 频率 | 方法 |
|--------|------|------|
| 权限映射更新 | 每次功能迭代 | 代码审查 |
| 新增 IPC 接口 | 每次迭代 | 安全审查 |
| 外部输入处理 | 每月 | 静态扫描 |
| 依赖库漏洞 | 每周 | CVE 扫描 |

### 8.2 监控建议

1. **异常 UID 访问**: 监控非预期 UID 的 IPC 调用
2. **DNS 异常**: 监控 DNS 解析失败率
3. **策略变更**: 记录网络策略变更操作
4. **崩溃日志**: 收集并分析服务崩溃信息

---

*生成时间: 2025-02-06*
