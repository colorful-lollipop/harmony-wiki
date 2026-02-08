# DSoftBus 编译配置

## 构建系统

DSoftBus 使用 **GN (Generate Ninja)** 作为构建系统。

## 根构建入口

**文件**: `BUILD.gn`

```gn
import("dsoftbus.gni")

lite_component("dsoftbus") {
  features = [
    "core:softbus_server",
    "sdk:softbus_client",
    "tests:softbus_test",
  ]
}
```

> 代码证据: `BUILD.gn`

## 主要 Targets

### 1. softbus_server (核心服务)

**类型**: `group`

**路径**: `core/BUILD.gn`

```gn
group("softbus_server") {
  deps = [
    "frame:softbus_server",
    "../sdk/napi/link_enhance:linkenhance",
    "../sdk/taihe:linkEnhance_taihe",
  ]
}
```

**产物**: 软总线服务模块

### 2. softbus_client (SDK 库)

**类型**: `ohos_static_library` / `ohos_shared_library`

**路径**: `sdk/BUILD.gn:75-135`

```gn
target(build_type, "softbus_client") {
  sources = common_client_src
  include_dirs = common_client_inc
  deps = common_client_deps
  external_deps = common_client_ext_deps + libsoftbus_stream_ext_deps
  public_configs = [ ":dsoftbus_sdk_interface" ]
  defines += TRANS_SDK_DEFINES
}
```

**产物**: `libsoftbus_client.a` 或 `libsoftbus_client.so`

### 3. linkenhance (N-API 模块)

**类型**: `ohos_shared_library`

**路径**: `sdk/napi/link_enhance/BUILD.gn`

```gn
ohos_shared_library("linkenhance") {
  sources = [...]
  include_dirs = [...]
  deps = [
    "$dsoftbus_root_path/core/common:softbus_utils",
    ...
  ]
}
```

**产物**: `liblinkenhance.so`

### 4. br_proxy 模块

**类型**: 静态库

**路径**: `br_proxy/BUILD.gn`

```gn
static_library("br_proxy") {
  sources = [...]
  include_dirs = [...]
  deps = [...]
}
```

**产物**: `libbr_proxy.a`

## SDK 接口配置

**文件**: `sdk/BUILD.gn:59-73`

```gn
config("dsoftbus_sdk_interface") {
  include_dirs = [
    "$dsoftbus_dfx_path/interface/include",
    "$dsoftbus_root_path/interfaces/kits",
    "$dsoftbus_root_path/interfaces/inner_kits/lnn",
    "$dsoftbus_root_path/interfaces/kits/bus_center",
    "$dsoftbus_root_path/interfaces/kits/common",
    "$dsoftbus_root_path/interfaces/kits/discovery",
    "$dsoftbus_root_path/interfaces/kits/transport",
    "$dsoftbus_root_path/sdk/transmission/session/cpp/include",
    "$dsoftbus_root_path/interfaces/inner_kits/transport",
    "$dsoftbus_root_path/core/transmission/common/include",
    "$dsoftbus_root_path/br_proxy",
  ]
}
```

## Feature 开关

### 系统 Feature

| Feature | 说明 | 默认值 |
|---------|------|--------|
| `dsoftbus_feature_conn_ble` | BLE 连接 | 开启 |
| `dsoftbus_feature_conn_br` | BR 连接 | 开启 |
| `dsoftbus_feature_conn_tcp_comm` | TCP 通信 | 开启 |
| `dsoftbus_feature_disc_ble` | BLE 发现 | 开启 |
| `dsoftbus_feature_disc_coap` | CoAP 发现 | 开启 |
| `dsoftbus_feature_lnn_wifi` | WiFi LNN | 开启 |
| `dsoftbus_feature_lnn_ble` | BLE LNN | 开启 |
| `dsoftbus_feature_trans_udp` | UDP 传输 | 开启 |
| `dsoftbus_feature_trans_qos` | QoS 传输 | 开启 |

> 代码证据: `bundle.json` 第 29-77 行

### 构建配置

| 配置项 | 说明 |
|-------|------|
| `dsoftbus_feature_build_shared_sdk` | 构建共享库 |
| `dsoftbus_access_token_feature` | 访问令牌特性 |
| `dsoftbus_feature_conn_ble_direct` | BLE 直连 |

## 编译产物

### 产物清单

| 产物 | 路径 | 说明 |
|-----|------|------|
| `libsoftbus_client.so` | `out/` | 共享库 SDK |
| `libsoftbus_client.a` | `out/` | 静态库 SDK |
| `liblinkenhance.so` | `out/` | N-API 模块 |
| `libbr_proxy.a` | `out/` | BR 代理静态库 |

### ROM/RAM 限制

| 限制 | 大小 |
|-----|------|
| ROM | 3000 KB |
| RAM | 40 MB |

> 代码证据: `bundle.json` 第 79-80 行

## 依赖关系

```
softbus_server
├── softbus_adapter (adapter/)
├── softbus_utils (core/common/)
├── wifi_direct (core/connection/wifi_direct_cpp/)
└── softbus_client (sdk/libsoftbus_client)

softbus_client
├── softbus_utils
├── softbus_adapter
├── softbus_dfx
├── ipc
└── access_token (可选)
```

## 外部依赖

| 组件 | 用途 |
|-----|------|
| `ability_base` | 能力基础 |
| `ability_runtime` | 运行时能力 |
| `access_token` | 访问令牌 |
| `bluetooth` | 蓝牙协议栈 |
| `device_auth` | 设备认证 |
| `hilog` | 日志 |
| `huks` | 密钥管理 |
| `ipc` | 进程通信 |
| `ffrt` | 函数运行时 |

> 代码证据: `bundle.json` 第 81-100 行

## 编译命令

```bash
# 标准系统
./build.sh --product-name {product} --build-type {debug|release}

# 查看产物
ls out/{product}/libs/
```

---

**相关文档**

- [项目概览](./01_Overview.md)
- [架构说明](./02_Architecture.md)
- [安全风险评审](./06_Security.md)
