# 常见问题与问题定位

## 1. 构建问题

### 1.1 编译失败

#### 症状: undefined reference to `xxx`

**可能原因**:
- 依赖库未正确链接
- 头文件路径配置错误
- 符号未导出 (hidden visibility)

**定位路径**:
1. 检查 `BUILD.gn` 中的 `deps` 和 `external_deps`
2. 检查头文件 `include_dirs` 配置
3. 检查符号导出: `nm -D libxxx.z.so | grep 符号名`

**修复**:
```gn
# 确保依赖正确声明
deps = [
    "//path/to:dependency",
]

# 确保头文件路径正确
include_dirs = [
    "include",
    "//other/path/include",
]
```

#### 症状: 找不到 BPF 编译器

**可能原因**:
- `netmanager_base_enable_traffic_statistic` 为 true 但未安装 BPF 工具链

**定位路径**:
```bash
cat netmanager_base_config.gni | grep netmanager_base_enable_traffic_statistic
```

**修复**:
- 安装 BPF 工具链，或
- 关闭特性: `netmanager_base_enable_traffic_statistic = false`

---

## 2. 运行时问题

### 2.1 服务启动失败

#### 症状: netmanager 进程崩溃

**日志路径**:
```bash
# 查看系统日志
hilog | grep -i "netmanager\|net_conn\|net_policy\|net_stats"

# 查看 crash 日志
ls /data/log/faultlog/
```

**定位步骤**:
1. 检查 SA 配置文件是否正确安装
   ```bash
   ls /system/profile/115*.json
   cat /system/profile/1151.json
   ```

2. 检查库文件是否存在
   ```bash
   ls /system/lib/libnet_conn_manager.z.so
   ls /system/lib/libnet_policy_manager.z.so
   ls /system/lib/libnet_stats_manager.z.so
   ```

3. 检查启动配置
   ```bash
   cat /system/etc/init/netmanager_base.cfg
   ```

#### 症状: netsysnative 进程无法启动

**可能原因**:
- BPF 程序加载失败
- 权限配置错误

**定位**:
```bash
# 检查 BPF 程序
ls /system/etc/bpf/netsys.o

# 检查权限
cat /system/profile/netsysnative_trust.json
```

### 2.2 IPC 调用失败

#### 症状: GetDefaultNet 返回错误

**定位路径**:
1. 检查服务是否运行
   ```bash
   ps -ef | grep netmanager
   ```

2. 检查 SA ID 是否正确
   ```bash
   # NetConnService SA ID 应为 1151
   cat /system/profile/1151.json
   ```

3. 查看 IPC 错误日志
   ```bash
   hilog | grep -i "ipc\|remote"
   ```

#### 症状: 权限检查失败

**错误码**: `NETMANAGER_ERROR` 或 `PERMISSION_DENIED`

**定位路径**:
1. 检查应用权限声明 (`config.json` 或 `module.json5`)
   ```json
   {
     "module": {
       "requestPermissions": [
         {
           "name": "ohos.permission.GET_NETWORK_INFO"
         }
       ]
     }
   }
   ```

2. 检查权限检查点日志
   ```bash
   hilog | grep -i "permission\|access_token"
   ```

3. 确认用户授权状态
   ```bash
   # 查看权限授权状态
   aa dump -a
   ```

---

## 3. 网络连接问题

### 3.1 无法获取默认网络

**症状**: `getDefaultNet()` 返回空或错误

**定位步骤**:

1. 检查网络供应商是否注册
   ```bash
   # 查看服务日志
   hilog | grep -i "supplier\|register"
   ```

2. 检查网络链路信息是否更新
   ```bash
   hilog | grep -i "netlink\|linkinfo"
   ```

3. 检查网络评分逻辑
   ```bash
   hilog | grep -i "score\|bestnetwork"
   ```

**关键代码路径**:
- `services/netconnmanager/src/net_conn_service.cpp:GetDefaultNet()`
- `services/netconnmanager/src/net_conn_service.cpp:FindBestNetworkForRequest()`

### 3.2 网络状态回调不触发

**症状**: `on('netAvailable')` 不触发

**定位步骤**:

1. 检查回调是否注册成功
   ```bash
   hilog | grep -i "register.*callback\|remotedied"
   ```

2. 检查网络状态变更
   ```bash
   hilog | grep -i "netavailable\|onnetavailable"
   ```

3. 检查 DeathRecipient 是否触发
   ```bash
   hilog | grep -i "deathrecipient\|remotedied"
   ```

**常见问题**:
- 回调对象被垃圾回收
- 服务端进程重启
- 权限不足导致注册失败

### 3.3 DNS 解析失败

**症状**: `getAddressesByName()` 返回空

**定位步骤**:

1. 检查 DNS 配置
   ```bash
   cat /system/etc/resolv.conf
   ```

2. 查看 DNS 日志
   ```bash
   hilog | grep -i "dns\|resolver"
   ```

3. 检查 DNS 缓存
   ```bash
   hilog | grep -i "dnscache\|getaddrinfo"
   ```

**关键代码路径**:
- `services/netmanagernative/src/netsys/dnsresolv/`
- `services/netmanagernative/src/manager/dns_manager.cpp`

---

## 4. 网络策略问题

### 4.1 策略不生效

**症状**: 设置策略后应用仍可访问网络

**定位步骤**:

1. 检查策略是否保存到数据库
   ```bash
   hilog | grep -i "policy.*database\|setpolicybyuid"
   ```

2. 检查 iptables 规则是否下发
   ```bash
   # 查看 iptables 规则
   iptables -L -n -v
   iptables -L -n -v -t mangle
   ```

3. 检查策略同步
   ```bash
   hilog | grep -i "firewall\|uidrule"
   ```

**关键代码路径**:
- `services/netpolicymanager/src/net_policy_service.cpp:SetPolicyByUid()`
- `services/netmanagernative/src/manager/firewall_manager.cpp`

### 4.2 后台策略异常

**症状**: 应用后台仍可访问网络

**可能原因**:
- 后台策略未正确应用
- 应用在前台有活动组件
- 设备空闲白名单

**定位**:
```bash
hilog | grep -i "background\|idle"
```

---

## 5. 流量统计问题

### 5.1 流量数据为 0

**症状**: `getUidRxBytes()` 返回 0

**定位步骤**:

1. 检查 BPF 程序是否加载
   ```bash
   # 检查 BPF Map
   ls /sys/fs/bpf/
   ```

2. 检查统计服务是否运行
   ```bash
   ps -ef | grep net_stats
   hilog | grep -i "net_stats"
   ```

3. 检查数据库是否有数据
   ```bash
   hilog | grep -i "database\|sqlite"
   ```

**关键代码路径**:
- `services/netstatsmanager/src/net_stats_service.cpp`
- `services/netmanagernative/bpf/src/bpf_stats.cpp`

### 5.2 统计回调不触发

**定位**:
```bash
hilog | grep -i "stats.*callback\|onstatschange"
```

---

## 6. 调试技巧

### 6.1 日志级别设置

```bash
# 设置 NetManager 日志级别
hilog -b D  # Debug 级别
hilog -b I  # Info 级别

# 过滤 NetManager 日志
hilog | grep -E "NetConn|NetPolicy|NetStats|Netsys"
```

### 6.2 Dump 信息

```bash
# 查看服务 Dump 信息
netdump netmanager
# 或
dumper -s NetConnService
```

### 6.3 GDB 调试

```bash
# 附加到 netmanager 进程
gdbserver :5039 $(pidof netmanager)

# 在开发机连接
gdb out/path/to/libnet_conn_manager.z.so
(gdb) target remote device-ip:5039
```

### 6.4 代码位置速查

| 功能 | 文件路径 |
|------|----------|
| 服务启动 | `services/*/src/*_service.cpp:OnStart()` |
| IPC 入口 | `services/*/src/stub/*_service_stub.cpp:OnRemoteRequest()` |
| 网络选择 | `services/netconnmanager/src/net_conn_service.cpp:FindBestNetworkForRequest()` |
| 策略下发 | `services/netpolicymanager/src/net_policy_service.cpp:SetPolicyByUid()` |
| 流量查询 | `services/netstatsmanager/src/net_stats_service.cpp:GetUidRxBytes()` |
| DNS 解析 | `services/netmanagernative/src/manager/dns_manager.cpp` |
| 防火墙 | `services/netmanagernative/src/manager/firewall_manager.cpp` |
| N-API 入口 | `frameworks/js/napi/*/*_module/src/*_module.cpp` |

---

## 7. 常见错误码

| 错误码 | 值 | 说明 | 排查方向 |
|--------|-----|------|----------|
| `NETMANAGER_SUCCESS` | 0 | 成功 | - |
| `NETMANAGER_ERROR` | 1 | 通用错误 | 查看日志 |
| `NETMANAGER_ERR_PARAMETER_ERROR` | 401 | 参数错误 | 检查参数类型/范围 |
| `NETMANAGER_ERR_CAPABILITY_NOT_SUPPORTED` | 801 | 能力不支持 | 检查系统能力 |
| `NETMANAGER_ERR_INTERNAL_ERROR` | 2100001 | 内部错误 | 查看服务日志 |
| `NETMANAGER_ERR_OPERATION_FAILED` | 2100002 | 操作失败 | 检查权限/状态 |
| `NET_CONN_ERR_NO_NETWORK` | 2100003 | 无网络 | 检查网络连接 |
| `NET_CONN_ERR_NO_HTTP_PROXY` | 2100005 | 无代理 | 检查代理配置 |
| `NET_CONN_ERR_HTTP_PROXY_INVALID` | 2100006 | 代理无效 | 检查代理参数 |

---

## 8. 相关命令速查

### 8.1 服务管理

```bash
# 查看服务状态
service check netmanager
service check netsysnative

# 重启服务
service stop netmanager
service start netmanager
```

### 8.2 网络诊断

```bash
# 查看网络接口
ifconfig

# 查看路由表
ip route

# 查看 DNS 配置
cat /system/etc/resolv.conf

# 测试网络连通
ping 8.8.8.8
```

### 8.3 防火墙检查

```bash
# 查看 iptables 规则
iptables -L -n -v
iptables -t mangle -L -n -v

# 查看特定 UID 规则
iptables -L -n -v | grep uid
```

### 8.4 BPF 检查

```bash
# 查看 BPF 程序
ls /sys/fs/bpf/

# 查看 BPF Map
bpftool map list
bpftool map dump id <map-id>
```

---

*生成时间: 2025-02-06*
