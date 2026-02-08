# 编译产物清单

## 1. 产物总览

NetManager Base 编译产生以下类型的产物：

| 类型 | 数量 | 安装位置 |
|------|------|----------|
| 共享库 (.so) | 20+ | `system/lib/` 或子目录 |
| 静态库 (.a) | 10+ | 仅编译时使用 |
| BPF 程序 | 1 | `system/etc/bpf/` |
| 配置文件 | 20+ | `system/etc/` 或子目录 |
| ABC 字节码 | 2+ | `system/framework/` |
| SA 配置文件 | 8 | `system/profile/` |

---

## 2. 共享库产物

### 2.1 对外接口库 (Public APIs)

| 产物名称 | 路径 | innerapi_tags | 说明 |
|----------|------|---------------|------|
| `libnet_conn_manager_if.z.so` | `system/lib/` | platformsdk, sasdk | 连接管理客户端接口 |
| `libnet_security_config_if.z.so` | `system/lib/` | platformsdk, sasdk | 网络安全配置接口 |
| `libsocket_permission.z.so` | `system/lib/` | platformsdk | 套接字权限接口 |
| `libnet_policy_manager_if.z.so` | `system/lib/` | platformsdk | 策略管理客户端接口 |
| `libnet_stats_manager_if.z.so` | `system/lib/` | platformsdk | 统计管理客户端接口 |
| `libnet_native_manager_if.z.so` | `system/lib/` | platformsdk | 原生服务客户端接口 |
| `libnet_manager_common.z.so` | `system/lib/` | platformsdk | 通用工具库 |
| `libnet_data_share.z.so` | `system/lib/` | platformsdk | 数据共享库 |
| `libnet_bundle_utils.z.so` | `system/lib/` | platformsdk | Bundle 工具库 |
| `libnapi_utils.z.so` | `system/lib/` | platformsdk | N-API 工具库 |
| `libnetsys_controller.z.so` | `system/lib/` | platformsdk_indirect | 网络系统控制器 |
| `libfwmark_client.z.so` | `system/lib/` | platformsdk_indirect | Fwmark 客户端 |
| `libnetsys_client.z.so` | `system/lib/` | platformsdk_indirect | NetSys C 客户端 |
| `libnetsys_bpf_utils.z.so` | `system/lib/` | platformsdk_indirect | BPF 工具库 |

**证据**: `bundle.json:153-340`

### 2.2 内部实现库

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `libnet_conn_manager.z.so` | `system/lib/` | 连接管理服务 (SA 1151) |
| `libnet_policy_manager.z.so` | `system/lib/` | 策略管理服务 (SA 1152) |
| `libnet_stats_manager.z.so` | `system/lib/` | 统计管理服务 (SA 1153) |
| `libnetsys_native_manager.z.so` | `system/lib/` | Native 服务 (SA 1158) |
| `libnet_service_common.z.so` | `system/lib/` | 服务公共代码 |
| `libnet_access_policy_dialog.z.so` | `system/lib/` | 访问策略对话框 |

### 2.3 JS/NAPI 模块

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `libconnection.z.so` | `system/lib/module/net/` | @ohos.net.connection |
| `libconnection_if.z.so` | `system/lib/` (内部) | connection 内部实现 |
| `libpolicy.z.so` | `system/lib/module/net/` | @ohos.net.policy |
| `libstatistics.z.so` | `system/lib/module/net/` | @ohos.net.statistics |
| `libnetwork.z.so` | `system/lib/module/` | @ohos.net.network |

### 2.4 C API (NDK)

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `libnet_connection.so` | `system/lib/ndk/` | C 接口共享库 |

### 2.5 ANI/Rust

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `libconnection_ani.dylib.so` | `system/lib/` | ETS Connection ANI |
| `libstatistics_ani.dylib.so` | `system/lib/` | ETS Statistics ANI |
| `libani_rs.a` | 仅编译使用 | Rust ANI 静态库 |
| `libani_sys.a` | 仅编译使用 | ANI 系统静态库 |

### 2.6 Cangjie FFI

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `libcj_net_connection_ffi.z.so` | `system/lib/` | Cangjie 连接 FFI |

---

## 3. BPF 产物

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `netsys.o` | `system/etc/bpf/` | eBPF 字节码 |

**源文件**: `services/netmanagernative/bpf/netsys.c`

---

## 4. 配置文件产物

### 4.1 SA 配置文件

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `1151.json` | `system/profile/` | NetConnService (SA 1151) |
| `1152.json` | `system/profile/` | NetPolicyService (SA 1152) |
| `1153.json` | `system/profile/` | NetStatsService (SA 1153) |
| `1154.json` | `system/profile/` | TetheringService (SA 1154) |
| `1155.json` | `system/profile/` | VpnService (SA 1155) |
| `1156.json` | `system/profile/` | DnsResolverService (SA 1156) |
| `1157.json` | `system/profile/` | EthernetService (SA 1157) |
| `1158.json` | `system/profile/` | NetsysNativeService (SA 1158) |

**源文件**: `sa_profile/*.json`

### 4.2 启动配置

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `netmanager_base.cfg` | `system/etc/init/` | netmanager 服务启动脚本 |
| `netsysnative.cfg` | `system/etc/init/` | netsysnative 服务启动脚本 |

### 4.3 信任配置

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `netmanager_trust.json` | `system/profile/` | NetManager 信任配置 |
| `netsysnative_trust.json` | `system/profile/` | NetsysNative 信任配置 |

### 4.4 网络配置

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `resolv.conf` | `system/etc/` | DNS 配置文件 |
| `netdetectionurl.conf` | `system/etc/` | 网络探测 URL 配置 |
| `detectionconfig.conf` | `system/etc/` | 探测配置 |
| `hosts` | `system/etc/` | Hosts 文件链接 |
| `initHosts` | `system/etc/` | 初始 Hosts |
| `xtables.lock` | `system/etc/` | iptables 锁文件 |
| `wearable_distributed_net_forward.json` | `system/etc/` | 可穿戴分布式网络配置 |

### 4.5 参数配置

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `netmanager_base.para` | `system/param/` | 系统参数 |
| `netmanager_base.para.dac` | `system/param/` | 系统参数 DAC |

---

## 5. 资源文件

### 5.1 通知资源

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `string.json` (多语言) | `system/etc/netmanager_base/resources/*/element/` | 多语言字符串 |
| `network_ic.png` | `system/etc/netmanager_base/resources/rawfile/` | 通知图标 |

### 5.2 ETS ABC

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `connection.abc` | `system/framework/` | Connection 模块字节码 |
| `statistics.abc` | `system/framework/` | Statistics 模块字节码 |

---

## 6. 产物依赖关系

### 6.1 运行时依赖图

```
应用层 (JS/TS)
    │
    ├── libconnection.z.so (@ohos.net.connection)
    │       ├── libconnection_if.z.so
    │       ├── libnet_conn_manager_if.z.so
    │       ├── libnapi_utils.z.so
    │       └── libnet_manager_common.z.so
    │
    ├── libpolicy.z.so (@ohos.net.policy)
    │       ├── libnet_policy_manager_if.z.so
    │       ├── libnapi_utils.z.so
    │       └── libnet_manager_common.z.so
    │
    └── libstatistics.z.so (@ohos.net.statistics)
            ├── libnet_stats_manager_if.z.so
            ├── libnapi_utils.z.so
            └── libnet_manager_common.z.so

服务层 (IPC)
    │
    ├── libnet_conn_manager.z.so (SA 1151)
    │       ├── libnet_service_common.z.so
    │       ├── libnetsys_controller.z.so
    │       └── libfwmark_client.z.so
    │
    ├── libnet_policy_manager.z.so (SA 1152)
    │       ├── libnet_service_common.z.so
    │       └── libnetsys_controller.z.so
    │
    ├── libnet_stats_manager.z.so (SA 1153)
    │       ├── libnet_service_common.z.so
    │       └── libnetsys_controller.z.so
    │
    └── libnetsys_native_manager.z.so (SA 1158)
            ├── libnetsys_bpf_utils.z.so
            ├── libfwmark_client.z.so
            └── netsys.o (BPF)

控制器层
    └── libnetsys_controller.z.so
            └── libnet_native_manager_if.z.so
```

### 6.2 进程加载关系

```
netmanager 进程启动时加载:
    - libnet_conn_manager.z.so
    - libnet_policy_manager.z.so
    - libnet_stats_manager.z.so
    - libnet_service_common.z.so
    - libnetsys_controller.z.so

netsysnative 进程启动时加载:
    - libnetsys_native_manager.z.so
    - libnetsys_bpf_utils.z.so
    - libfwmark_client.z.so
```

---

## 7. 产物映射表

### 7.1 Target → 产物映射

| Target | 产物类型 | 产物名称 | 安装路径 |
|--------|----------|----------|----------|
| `net_conn_manager` | ohos_shared_library | `libnet_conn_manager.z.so` | `system/lib/` |
| `net_policy_manager` | ohos_shared_library | `libnet_policy_manager.z.so` | `system/lib/` |
| `net_stats_manager` | ohos_shared_library | `libnet_stats_manager.z.so` | `system/lib/` |
| `netsys_native_manager` | ohos_shared_library | `libnetsys_native_manager.z.so` | `system/lib/` |
| `netsys_controller` | ohos_shared_library | `libnetsys_controller.z.so` | `system/lib/` |
| `net_conn_manager_if` | ohos_shared_library | `libnet_conn_manager_if.z.so` | `system/lib/` |
| `net_policy_manager_if` | ohos_shared_library | `libnet_policy_manager_if.z.so` | `system/lib/` |
| `net_stats_manager_if` | ohos_shared_library | `libnet_stats_manager_if.z.so` | `system/lib/` |
| `net_native_manager_if` | ohos_shared_library | `libnet_native_manager_if.z.so` | `system/lib/` |
| `connection` | ohos_shared_library | `libconnection.z.so` | `system/lib/module/net/` |
| `policy` | ohos_shared_library | `libpolicy.z.so` | `system/lib/module/net/` |
| `statistics` | ohos_shared_library | `libstatistics.z.so` | `system/lib/module/net/` |
| `network` | ohos_shared_library | `libnetwork.z.so` | `system/lib/module/` |
| `net_connection` | ohos_shared_library | `libnet_connection.so` | `system/lib/ndk/` |
| `netsys` | ohos_bpf | `netsys.o` | `system/etc/bpf/` |
| `net_manager_profile` | ohos_sa_profile | `1151.json` ~ `1158.json` | `system/profile/` |

---

## 8. 验证产物

### 8.1 检查产物安装

```bash
# 检查共享库
ls -la out/*/system/lib/libnet_*.z.so
ls -la out/*/system/lib/module/net/*.z.so

# 检查配置文件
ls -la out/*/system/profile/115*.json
ls -la out/*/system/etc/init/netmanager*.cfg

# 检查 BPF
ls -la out/*/system/etc/bpf/netsys.o
```

### 8.2 检查依赖关系

```bash
# 查看库依赖
readelf -d out/*/system/lib/libnet_conn_manager_if.z.so | grep NEEDED

# 查看符号
nm -D out/*/system/lib/libnet_conn_manager_if.z.so | grep NetConnClient
```

---

*生成时间: 2025-02-06*
