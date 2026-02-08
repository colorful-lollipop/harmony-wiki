# GN 构建系统

## 1. 构建配置概览

### 1.1 全局配置文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `netmanager_base_config.gni` | `//foundation/communication/netmanager_base/` | 全局构建配置 |

### 1.2 全局变量定义

```gni
# 代码根目录
NETMANAGER_BASE_ROOT = "//foundation/communication/netmanager_base"
SUBSYSTEM_DIR = "//foundation/communication"

# 子模块路径
NETCONNMANAGER_SOURCE_DIR = "$NETMANAGER_BASE_ROOT/services/netconnmanager"
NETPOLICYMANAGER_SOURCE_DIR = "$NETMANAGER_BASE_ROOT/services/netpolicymanager"
NETSTATSMANAGER_SOURCE_DIR = "$NETMANAGER_BASE_ROOT/services/netstatsmanager"
NETMANAGERNATIVE_ROOT = "$NETMANAGER_BASE_ROOT/services/netmanagernative"
NETSYSCONTROLLER_ROOT_DIR = "$NETMANAGER_BASE_ROOT/services/netsyscontroller"
INNERKITS_ROOT = "$NETMANAGER_BASE_ROOT/interfaces/innerkits"
```

### 1.3 特性开关

**文件**: `netmanager_base_config.gni:46-64`

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `enable_netmgr_debug` | true | 调试日志 |
| `netmanager_base_enable_feature_net_firewall` | false | 网络防火墙 |
| `netmanager_base_enable_feature_wearable_distributed_net` | false | 可穿戴分布式网络 |
| `netmanager_base_enable_feature_sysvpn` | false | 系统 VPN |
| `netmanager_base_enable_netsys_access_policy_diag_listen` | false | 访问策略诊断监听 |
| `netmanager_base_feature_support_powermanager` | false | 电源管理支持 |
| `netmanager_base_enable_feature_hosts` | false | Hosts 管理 |
| `netmanager_base_enable_public_dns_server` | false | 公共 DNS 服务器 |
| `netmanager_base_support_ebpf_memory_miniaturization` | false | eBPF 内存精简 |
| `netmanager_base_enable_traffic_statistic` | false | 流量统计 |
| `netmanager_base_enable_pac_proxy` | false | PAC 代理 |
| `netmanager_base_enable_set_app_frozened` | false | 应用冻结 |
| `netmanager_base_feature_enterprise_route_custom` | false | 企业路由定制 |
| `netmanager_base_extended_features` | true | 扩展功能集 |

---

## 2. 构建目标清单

### 2.1 按模块分组

#### 2.1.1 工具模块 (utils/)

| Target | 类型 | 源文件 | 对外接口 |
|--------|------|--------|----------|
| `net_manager_common` | ohos_shared_library | base64_utils.cpp, event_report.cpp | **对外** (platformsdk) |
| `net_data_share` | ohos_shared_library | net_datashare_utils.cpp | **对外** (platformsdk) |
| `net_bundle_utils` | ohos_shared_library | net_bundle_impl.cpp | **对外** (platformsdk) |
| `napi_utils` | ohos_shared_library | base_context.cpp, event_manager.cpp | **对外** (platformsdk) |

**BUILD.gn**: `utils/BUILD.gn`

```gn
ohos_shared_library("net_manager_common") {
  sources = [
    "common_utils/src/base64_utils.cpp",
    "common_utils/src/event_report.cpp",
    "common_utils/src/netmanager_base_common_utils.cpp",
    "common_utils/src/netmanager_base_permission.cpp",
  ]
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "access_token:libaccesstoken_sdk",
  ]
  innerapi_tags = [ "platformsdk" ]
}
```

#### 2.1.2 连接管理服务 (services/netconnmanager/)

| Target | 类型 | 说明 |
|--------|------|------|
| `net_conn_manager` | ohos_shared_library | 连接管理服务 |
| `net_conn_manager_static` | ohos_static_library | 静态库（测试用） |

**关键源文件**:
- `src/net_conn_service.cpp`
- `src/net_supplier.cpp`
- `src/network.cpp`
- `src/net_monitor.cpp`

**关键依赖**:
```gn
deps = [
  "$NETCONNMANAGER_COMMON_DIR:net_service_common",
  "$NETSYSCONTROLLER_ROOT_DIR:netsys_controller",
  "$NETMANAGERNATIVE_ROOT/fwmarkclient:fwmark_client",
]
```

#### 2.1.3 策略管理服务 (services/netpolicymanager/)

| Target | 类型 | 说明 |
|--------|------|------|
| `net_policy_manager` | ohos_shared_library | 策略管理服务 |
| `net_policy_manager_static` | ohos_static_library | 静态库 |
| `net_access_policy_dialog` | ohos_shared_library | 访问策略对话框 |

#### 2.1.4 统计管理服务 (services/netstatsmanager/)

| Target | 类型 | 说明 |
|--------|------|------|
| `net_stats_manager` | ohos_shared_library | 统计管理服务 |
| `net_stats_manager_static` | ohos_static_library | 静态库 |

#### 2.1.5 Native 服务 (services/netmanagernative/)

| Target | 类型 | 说明 |
|--------|------|------|
| `netsys_native_manager` | ohos_shared_library | Native 服务主库 |
| `netsys_native_manager_static` | ohos_static_library | 静态库 |
| `netsys_client` | ohos_shared_library | C 接口客户端 |
| `netsys` | ohos_bpf | eBPF 程序 |
| `netsys_bpf_utils` | ohos_shared_library | BPF 工具库 |
| `fwmark_client` | ohos_shared_library | Fwmark 客户端 |

#### 2.1.6 网络系统控制器 (services/netsyscontroller/)

| Target | 类型 | 说明 |
|--------|------|------|
| `netsys_controller` | ohos_shared_library | 控制器库 |
| `netsys_controller_static` | ohos_static_library | 静态库 |

#### 2.1.7 内部接口 (interfaces/innerkits/)

| Target | 类型 | 路径 | 说明 |
|--------|------|------|------|
| `net_conn_manager_if` | ohos_shared_library | netconnclient/ | 连接管理客户端 |
| `net_conn_parcel` | ohos_static_library | netconnclient/ | 数据结构序列化 |
| `socket_permission` | ohos_shared_library | netconnclient/ | 套接字权限 |
| `net_security_config_if` | ohos_shared_library | netconnclient/ | 网络安全配置 |
| `net_policy_manager_if` | ohos_shared_library | netpolicyclient/ | 策略管理客户端 |
| `net_policy_parcel` | ohos_static_library | netpolicyclient/ | 策略数据结构 |
| `net_stats_manager_if` | ohos_shared_library | netstatsclient/ | 统计管理客户端 |
| `net_stats_parcel` | ohos_static_library | netstatsclient/ | 统计数据结构 |
| `net_native_manager_if` | ohos_shared_library | netmanagernative/ | Native 服务客户端 |
| `net_native_parcel` | ohos_static_library | netmanagernative/ | Native 数据结构 |

#### 2.1.8 C API (interfaces/kits/c/)

| Target | 类型 | 路径 | 安装位置 |
|--------|------|------|----------|
| `net_connection` | ohos_shared_library | netconnclient/ | `system/lib/ndk/` |

#### 2.1.9 JS/NAPI (frameworks/js/napi/)

| Target | 类型 | 路径 | 安装位置 |
|--------|------|------|----------|
| `connection` | ohos_shared_library | connection/ | `system/lib/module/net/` |
| `connection_if` | ohos_shared_library | connection/ | 内部 |
| `policy` | ohos_shared_library | netpolicy/ | `system/lib/module/net/` |
| `statistics` | ohos_shared_library | netstats/ | `system/lib/module/net/` |
| `network` | ohos_shared_library | network/ | `system/lib/module/` |

#### 2.1.10 ETS/ANI (frameworks/ets/ani/)

| Target | 类型 | 路径 | 说明 |
|--------|------|------|------|
| `ani_package` | group | ./ | 组合目标 |
| `connection_ani` | ohos_rust_shared_library | connection/ | Rust ANI |
| `connection` | generate_static_abc | connection/ | ETS 编译 |
| `connection_etc` | ohos_prebuilt_etc | connection/ | ABC 安装 |
| `statistics_ani` | ohos_rust_shared_library | statistics/ | Rust ANI |
| `statistics` | generate_static_abc | statistics/ | ETS 编译 |
| `statistics_etc` | ohos_prebuilt_etc | statistics/ | ABC 安装 |

#### 2.1.11 SA 配置 (sa_profile/)

| Target | 类型 | 源文件 |
|--------|------|--------|
| `net_manager_profile` | ohos_sa_profile | 1151.json ~ 1158.json |

#### 2.1.12 配置文件 (services/etc/init/)

| Target | 类型 | 源文件 | 安装位置 |
|--------|------|--------|----------|
| `netmanager_trust` | ohos_prebuilt_etc | netmanager_trust.json | `system/profile/` |
| `netsysnative_trust` | ohos_prebuilt_etc | netsysnative_trust.json | `system/profile/` |
| `netmanager_base.rc` | ohos_prebuilt_etc | netmanager_base.cfg | `system/etc/init/` |
| `netsysnative.rc` | ohos_prebuilt_etc | netsysnative.cfg | `system/etc/init/` |
| `resolv.conf` | ohos_prebuilt_etc | resolv.conf | `system/etc/` |
| `netdetectionurl.conf` | ohos_prebuilt_etc | netdetectionurl.conf | `system/etc/` |
| `netmanager_base.para` | ohos_prebuilt_etc | netmanager_base.para | `system/param/` |

---

## 3. 关键构建配置详解

### 3.1 安全加固配置

所有主要共享库启用以下安全特性：

```gn
ohos_shared_library("xxx") {
  sanitize = {
    cfi = true              # 控制流完整性
    cfi_cross_dso = true
    boundary_sanitize = true
    all_ubsan = true        # 未定义行为检测
    debug = false
  }
  
  branch_protector_ret = "pac_ret"  # 分支保护
  
  cflags = [
    "-D_FORTIFY_SOURCE=2",  # 源码强化
    "-O2",
    "-fvisibility=hidden",  # 符号隐藏
  ]
  
  ldflags = [
    "-Wl,--gc-sections",    # 移除未使用代码
  ]
}
```

### 3.2 N-API 模块配置示例

**文件**: `frameworks/js/napi/connection/BUILD.gn`

```gn
ohos_shared_library("connection") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    boundary_sanitize = true
    all_ubsan = true
    debug = false
  }
  
  branch_protector_ret = "pac_ret"
  
  include_dirs = [
    "async_context/include",
    "async_work/include",
    "connection_exec/include",
    "connection_helper/include",
    "connection_module/include",
    "observer/include",
    "options/include",
  ]
  
  sources = [ "connection_module/src/connection_module.cpp" ]
  
  deps = [
    "$INNERKITS_ROOT/netconnclient:net_conn_manager_if",
    ":connection_if",  # 内部接口
    "$NETMANAGER_BASE_ROOT/utils:net_manager_common",
    "$NETMANAGER_BASE_ROOT/utils/napi_utils:napi_utils",
  ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "napi:ace_napi",
  ]
  
  relative_install_dir = "module/net"
  part_name = "netmanager_base"
  subsystem_name = "communication"
}

ohos_shared_library("connection_if") {
  # 内部实现库，供 connection 依赖
  # 包含所有异步工作、上下文、执行函数
  sources = [
    "async_context/src/*.cpp",
    "async_work/src/connection_async_work.cpp",
    "connection_exec/src/connection_exec.cpp",
    # ...
  ]
  
  # 不设置 relative_install_dir，不单独安装
  part_name = "netmanager_base"
  subsystem_name = "communication"
}
```

### 3.3 服务模块配置示例

**文件**: `services/netconnmanager/BUILD.gn`

```gn
ohos_shared_library("net_conn_manager") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    boundary_sanitize = true
    ubsan = true
    debug = false
  }
  
  branch_protector_ret = "pac_ret"
  
  include_dirs = [
    "include",
    "include/stub",
  ]
  
  sources = [
    "src/net_conn_service.cpp",
    "src/net_supplier.cpp",
    "src/network.cpp",
    "src/net_activate.cpp",
    "src/net_monitor.cpp",
    # ...
  ]
  
  deps = [
    "//foundation/communication/netmanager_base/services/common:net_service_common",
    "//foundation/communication/netmanager_base/services/netsyscontroller:netsys_controller",
    "//foundation/communication/netmanager_base/services/netmanagernative/fwmarkclient:fwmark_client",
    "//foundation/communication/netmanager_base/interfaces/innerkits/netconnclient:net_conn_manager_if",
  ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "ffrt:libffrt",
    # ...
  ]
  
  part_name = "netmanager_base"
  subsystem_name = "communication"
  installed = true
}
```

---

## 4. 依赖关系图

```
应用层
  ├── connection (JS N-API)
  │       └── 依赖: connection_if, net_conn_manager_if, napi_utils
  │
  ├── policy (JS N-API)
  │       └── 依赖: net_policy_manager_if, napi_utils
  │
  └── statistics (JS N-API)
          └── 依赖: net_stats_manager_if, napi_utils

接口层
  ├── net_conn_manager_if
  │       └── 依赖: net_conn_parcel, net_manager_common
  │
  ├── net_policy_manager_if
  │       └── 依赖: net_policy_parcel, net_manager_common
  │
  ├── net_stats_manager_if
  │       └── 依赖: net_stats_parcel, net_manager_common
  │
  └── net_native_manager_if
          └── 依赖: net_native_parcel, net_manager_common

服务层
  ├── net_conn_manager
  │       └── 依赖: net_service_common, netsys_controller, fwmark_client
  │
  ├── net_policy_manager
  │       └── 依赖: net_service_common, netsys_controller
  │
  ├── net_stats_manager
  │       └── 依赖: net_service_common, netsys_controller
  │
  └── netsys_native_manager
          └── 依赖: netsys_bpf_utils, fwmark_client, net_conn_manager_if

控制器层
  └── netsys_controller
          └── 依赖: net_native_manager_if

工具层
  ├── net_manager_common
  ├── net_data_share
  ├── net_bundle_utils
  └── napi_utils
      └── 依赖: net_manager_common
```

---

## 5. BUILD.gn 文件索引

| 目录 | BUILD.gn 文件 | 主要 Targets |
|------|--------------|--------------|
| `utils/` | BUILD.gn | net_manager_common, net_data_share, net_bundle_utils |
| `utils/napi_utils/` | BUILD.gn | napi_utils |
| `services/common/` | BUILD.gn | net_service_common |
| `services/netconnmanager/` | BUILD.gn | net_conn_manager |
| `services/netpolicymanager/` | BUILD.gn | net_policy_manager |
| `services/netstatsmanager/` | BUILD.gn | net_stats_manager |
| `services/netmanagernative/` | BUILD.gn | netsys_native_manager, netsys_client |
| `services/netmanagernative/bpf/` | BUILD.gn | netsys (BPF), netsys_bpf_utils |
| `services/netmanagernative/fwmarkclient/` | BUILD.gn | fwmark_client |
| `services/netsyscontroller/` | BUILD.gn | netsys_controller |
| `interfaces/innerkits/netconnclient/` | BUILD.gn | net_conn_manager_if, net_conn_parcel |
| `interfaces/innerkits/netpolicyclient/` | BUILD.gn | net_policy_manager_if, net_policy_parcel |
| `interfaces/innerkits/netstatsclient/` | BUILD.gn | net_stats_manager_if, net_stats_parcel |
| `interfaces/innerkits/netmanagernative/` | BUILD.gn | net_native_manager_if, net_native_parcel |
| `interfaces/kits/c/netconnclient/` | BUILD.gn | net_connection |
| `frameworks/js/napi/connection/` | BUILD.gn | connection, connection_if |
| `frameworks/js/napi/network/` | BUILD.gn | network |
| `frameworks/js/napi/netpolicy/` | BUILD.gn | policy |
| `frameworks/js/napi/netstats/` | BUILD.gn | statistics |
| `frameworks/ets/ani/connection/` | BUILD.gn | connection_ani, connection |
| `frameworks/ets/ani/statistics/` | BUILD.gn | statistics_ani, statistics |
| `frameworks/cj/connection/` | BUILD.gn | cj_net_connection_ffi |
| `common/ani_rs/` | BUILD.gn | ani_rs |
| `common/ani_sys/` | BUILD.gn | ani_sys |
| `sa_profile/` | BUILD.gn | net_manager_profile |
| `services/etc/init/` | BUILD.gn | 各种配置文件 |

---

## 6. 构建命令示例

### 6.1 构建单个目标

```bash
# 构建连接管理服务
hb build //foundation/communication/netmanager_base/services/netconnmanager:net_conn_manager

# 构建 N-API 模块
hb build //foundation/communication/netmanager_base/frameworks/js/napi/connection:connection

# 构建接口库
hb build //foundation/communication/netmanager_base/interfaces/innerkits/netconnclient:net_conn_manager_if
```

### 6.2 构建整个模块

```bash
# 构建 netmanager_base 所有目标
hb build //foundation/communication/netmanager_base/...
```

---

*生成时间: 2025-02-06*
