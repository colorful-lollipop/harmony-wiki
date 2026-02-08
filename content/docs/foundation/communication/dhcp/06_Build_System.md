# GN 构建系统

## 文档目的

本文档详细说明 DHCP 组件的 GN 构建配置，包括 targets 列表、依赖关系和编译选项。

---

## 适用范围

- 构建工具: GN (Generate Ninja)
- 构建文件: BUILD.gn, .gni, bundle.json
- 组件版本: 3.1.0

---

## 根构建入口

### 1. Lite 版本构建入口

```gn
# 证据: BUILD.gn:14-26
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")
  import("//foundation/communication/dhcp/dhcp_lite.gni")

  lite_component("dhcp") {
    deps = [
      "$DHCP_ROOT_DIR/services/dhcp_client:dhcp_client",
      "$DHCP_ROOT_DIR/services/dhcp_server:dhcp_server",
    ]
  }
}
```

### 2. 组件配置入口

```json
// 证据: bundle.json:37-80
"component": {
  "name": "dhcp",
  "subsystem": "communication",
  "adapted_system_type": ["small", "standard"]
}
```

---

## 关键 Targets（按模块分组）

### Frameworks 模块

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `dhcp_sdk` | ohos_shared_library | libdhcp_sdk.z.so | 主 SDK 库 |
| `dhcp_client_proxy_impl` | ohos_source_set | - | Client 代理源集合 |
| `dhcp_server_proxy_impl` | ohos_source_set | - | Server 代理源集合 |

#### dhcp_sdk 详细配置

```gn
# 证据: frameworks/native/BUILD.gn:20-80
ohos_shared_library("dhcp_sdk") {
  # 源文件
  sources = [
    "c_adapter/src/dhcp_c_service.cpp",
    "c_adapter/src/dhcp_c_utils.cpp",
    "src/dhcp_client.cpp",
    "src/dhcp_client_callback_stub.cpp",
    "src/dhcp_event.cpp",
    "src/dhcp_server.cpp",
    "src/dhcp_server_callback_stub.cpp",
    # ...
  ]

  # 依赖
  deps = [
    ":dhcp_client_proxy_impl",
    ":dhcp_server_proxy_impl",
  ]

  # 外部依赖
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
  ]

  # 符号控制
  version_script = "libdhcp_sdk.map"
  install_enable = true
}
```

---

### Services/dhcp_client 模块

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `dhcp_client` | ohos_shared_library | libdhcp_client.z.so | Client SA 库 |
| `dhcp_client_static` | ohos_static_library | libdhcp_client_static.a | Client 静态库 |
| `dhcp_updater_client` | ohos_shared_library | libdhcp_updater_client.z.so | 升级专用 Client |

#### dhcp_client 详细配置

```gn
# 证据: services/dhcp_client/BUILD.gn:20-100
ohos_shared_library("dhcp_client") {
  sources = [
    "src/dhcp_client_callback_proxy.cpp",
    "src/dhcp_client_death_recipient.cpp",
    "src/dhcp_client_service_impl.cpp",
    "src/dhcp_client_state_machine.cpp",
    "src/dhcp_ipv6_client.cpp",
    "src/dhcp_socket.cpp",
    # 15 个源文件
  ]

  deps = [
    "$DHCP_ROOT_DIR/services/utils:dhcp_utils",
  ]

  external_deps = [
    "ability_runtime:wantagent_innerkits",
    "bundle_framework:appexecfwk_base",
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "netmanager_base:net_native_manager_if",
    "safwk:system_ability_fwk",
    "time_service:time_client",
  ]

  # SA 类型
  shlib_type = "sa"
  install_enable = true
}
```

---

### Services/dhcp_server 模块

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `dhcp_server` | ohos_shared_library | libdhcp_server.z.so | Server SA 库 |
| `dhcp_server_static` | ohos_static_library | libdhcp_server_static.a | Server 静态库 |

---

### Services/utils 模块

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `dhcp_utils` | ohos_shared_library | libdhcp_utils.z.so | 工具共享库 |

#### dhcp_utils 详细配置

```gn
# 证据: services/utils/BUILD.gn:20-70
ohos_shared_library("dhcp_utils") {
  sources = [
    "src/dhcp_arp_checker.cpp",
    "src/dhcp_common_utils.cpp",
    "src/dhcp_permission_utils.cpp",
    "src/dhcp_sa_manager.cpp",
    "src/dhcp_system_timer.cpp",
    "src/dhcp_thread.cpp",
  ]

  external_deps = [
    "ability_runtime:wantagent_innerkits",
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "bundle_framework:appexecfwk_base",
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "samgr:samgr_proxy",
  ]

  version_script = "libdhcp_util.map"
  innerapi_tags = ["platformsdk"]
}
```

---

### Services/sa_profile 模块

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `wifi_standard_sa_profile` | ohos_sa_profile | 1126.json, 1127.json | SA 配置文件 |

---

## 依赖关系图

```
┌─────────────────────────────────────────┐
│         bundle.json (组件根)             │
└──────────────┬──────────────────────────┘
               │
       ┌───────┴───────┬──────────────┐
       ▼               ▼              ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  dhcp_sdk   │ │dhcp_client  │ │dhcp_server  │
│(libdhcp_sdk)│ │(libdhcp_    │ │(libdhcp_    │
│             │ │client.z.so) │ │server.z.so) │
└──────┬──────┘ └──────┬──────┘ └──────┬──────┘
       │               │              │
       └───────┬───────┴──────────────┘
               ▼
       ┌─────────────┐
       │ dhcp_utils  │
       │(libdhcp_    │
       │ utils.z.so) │
       └──────┬──────┘
              │
              ▼
       ┌─────────────────┐
       │ 外部依赖组件      │
       │c_utils, hilog,   │
       │ipc, safwk, ...   │
       └─────────────────┘
```

---

## 编译选项与宏定义

### 全局变量（dhcp.gni）

```gn
# 证据: dhcp.gni:19-23
declare_args() {
  VENDOR_NAME = "HUAWEI:openharmony"
  IPV4_DNS_PRI = "8.8.8.8"
  IPV4_DNS_SEC = "8.8.4.4"
}
```

### 条件编译宏

| 宏 | 触发条件 | 说明 |
|----|----------|------|
| `OHOS_ARCH_LITE` | ohos_lite 定义 | Lite 版本 |
| `OHOS_EUPDATER` | updater 构建 | 升级模式 |
| `DHCP_HILOG_ENABLE` | dhcp_hilog_enable=true | Hilog 日志 |
| `DHCP_FFRT_ENABLE` | ffrt 存在 | FFRT 调度 |
| `DTFUZZ_TEST` | is_asan=true | ASan 测试 |

### 安全编译选项

```gn
# 证据: frameworks/native/BUILD.gn:90-100
sanitize = {
  cfi = true                    # 控制流完整性
  boundary_sanitize = true      # 边界检查
  cfi_cross_dso = true          # 跨 SO CFI
  integer_overflow = true       # 整数溢出
  ubsan = true                  # 未定义行为
}
```

---

## 构建产物映射

### Target → 产物

| Target | 产物类型 | 安装路径 | 产物名称 |
|--------|----------|----------|----------|
| dhcp_sdk | ohos_shared_library | /system/lib64/ | libdhcp_sdk.z.so |
| dhcp_client | ohos_shared_library | /system/lib64/ | libdhcp_client.z.so |
| dhcp_server | ohos_shared_library | /system/lib64/ | libdhcp_server.z.so |
| dhcp_utils | ohos_shared_library | /system/lib64/ | libdhcp_utils.z.so |
| wifi_standard_sa_profile | ohos_sa_profile | /system/profile/ | 1126.json, 1127.json |

### 静态库

| Target | 产物名称 | 用途 |
|--------|----------|------|
| dhcp_client_static | libdhcp_client_static.a | Client 静态链接 |
| dhcp_server_static | libdhcp_server_static.a | Server 静态链接 |

---

## 测试 Targets

### 单元测试

| Target | 类型 | 说明 |
|--------|------|------|
| dhcp_unittest | group | 单元测试分组 |
| dhcp_native_unittest | ohos_unittest | Native 层测试 |
| dhcp_client_unittest | ohos_unittest | Client 测试 |
| dhcp_server_unittest | ohos_unittest | Server 测试 |
| dhcp_util_unittest | ohos_unittest | Utils 测试 |

### Fuzz 测试（18 个）

1. AddressUtilsFuzzTest
2. ClientStubFuzzTest
3. CommonUtilFuzzTest
4. DhcpAddressPoolFuzzTest
5. DhcpArgumentFuzzTest
6. DhcpBindingFuzzTest
7. DhcpClientFuzzTest
8. DhcpClientCbkStubFuzzTest
9. DhcpClientFunFuzzTest
10. DhcpFunctionFuzzTest
11. DhcpServerFuzzTest
12. DhcpServerCbkStubFuzzTest
13. DhcpServerImplFuzzTest
14. ServerStubFuzzTest
15. DhcpArpCheckerFuzzTest
16. DhcpCommonUtilsFuzzTest
17. DhcpEventFuzzTest
18. DhcpFunction2FuzzTest

---

## 构建命令

### 构建 DHCP 组件

```bash
# 标准系统
hb build dhcp

# Lite 系统
hb build -f foundation/communication/dhcp
```

### 清理构建产物

```bash
hb clean
```

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [02_Directory_Structure](02_Directory_Structure.md) - 目录结构
- [07_Compile_Artifacts](07_Compile_Artifacts.md) - 编译产物详情
