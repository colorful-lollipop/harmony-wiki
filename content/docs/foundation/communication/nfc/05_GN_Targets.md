# GN 构建目标与编译系统

## 文档信息

| 项目 | 内容 |
|------|------|
| **目的** | 描述 NFC 组件的 GN 构建配置、目标和依赖关系 |
| **适用范围** | 构建工程师、系统集成 |
| **相关文档** | [构建产物](07_Build_Artifacts.md)、[目录结构](02_Directory_Structure.md) |

---

## 1. 构建配置总览

### 1.1 全局配置 (nfc.gni)

**位置**: `/foundation/communication/nfc/nfc.gni`

```gn
NFC_DIR = "//foundation/communication/nfc"

# Feature flags
declare_args() {
  nfc_use_vendor_nci_native = false
  nfc_service_feature_vendor_applications_enabled = false
  nfc_sim_feature = false
  nfc_service_feature_ndef_wifi_enabled = false
  nfc_service_feature_ndef_bt_enabled = false
  nfc_vibrator_disabled = false
  nfc_handle_screen_lock = false
}
```

### 1.2 Feature 开关

| Feature | 变量名 | 默认值 | 说明 |
|---------|--------|--------|------|
| 厂商 NCI | `nfc_use_vendor_nci_native` | false | 使用厂商 NCI 实现 |
| 厂商应用 | `nfc_service_feature_vendor_applications_enabled` | false | 启用厂商应用扩展 |
| SIM 卡 | `nfc_sim_feature` | false | SIM 卡 NFC 功能 |
| WiFi NDEF | `nfc_service_feature_ndef_wifi_enabled` | false | WiFi 配对功能 |
| 蓝牙 NDEF | `nfc_service_feature_ndef_bt_enabled` | false | 蓝牙配对功能 |
| 禁用振动 | `nfc_vibrator_disabled` | false | 禁用标签发现振动 |
| 屏幕锁 | `nfc_handle_screen_lock` | false | 处理屏幕锁定事件 |

---

## 2. 生产目标清单

### 2.1 核心服务目标

| 目标 | 类型 | 输出 | 位置 |
|------|------|------|------|
| `nfc_service` | ohos_shared_library | libnfc_service.z.so | services/BUILD.gn:173 |
| `nci_native_default` | ohos_shared_library | libnci_native_default.z.so | services/src/nci_adapter/nci_native_default/ |
| `nfc_notification` | ohos_shared_library | libnfc_notification.z.so | services/src/notification/ |
| `nfc_prebuilt_config` | ohos_prebuilt_etc | resources/ | services/BUILD.gn:68 |
| `nfc_service.rc` | ohos_prebuilt_etc | nfc_service.cfg | services/etc/init/ |

### 2.2 Inner API 库

| 目标 | 类型 | 输出 | 位置 |
|------|------|------|------|
| `nfc_inner_kits_common` | ohos_shared_library | libnfc_inner_kits_common.z.so | interfaces/inner_api/common/ |
| `nfc_inner_kits_controller` | ohos_shared_library | libnfc_inner_kits_controller.z.so | interfaces/inner_api/controller/ |
| `nfc_inner_kits_tags` | ohos_shared_library | libnfc_inner_kits_tags.z.so | interfaces/inner_api/tags/ |
| `nfc_inner_kits_card_emulation` | ohos_shared_library | libnfc_inner_kits_card_emulation.z.so | interfaces/inner_api/cardEmulation/ |

### 2.3 IDL 接口目标

| 目标 | 类型 | 生成来源 | 位置 |
|------|------|----------|------|
| `nfc_controller_interface` | idl_gen_interface | INfcController.idl | interfaces/inner_api/controller/ |
| `nfc_tag_interface` | idl_gen_interface | ITagSession.idl | interfaces/inner_api/tags/ |
| `nfc_hce_interface` | idl_gen_interface | IHceSession.idl | interfaces/inner_api/cardEmulation/ |

### 2.4 JS N-API 目标

| 目标 | 类型 | 输出 | 安装路径 |
|------|------|------|----------|
| `controller` | ohos_shared_library | libcontroller.z.so | system/lib/module/nfc/ |
| `tag` | ohos_shared_library | libtag.z.so | system/lib/module/nfc/ |
| `cardemulation` | ohos_shared_library | libcardemulation.z.so | system/lib/module/nfc/ |

### 2.5 ETS Taihe 目标

| 目标 | 类型 | 输出 | 安装路径 |
|------|------|------|----------|
| `nfc_fwk_taihe_controller` | taihe_shared_library | ANI 库 | system/framework/ |
| `nfc_fwk_taihe_tag` | taihe_shared_library | ANI 库 | system/framework/ |
| `nfc_fwk_taihe_cardEmulation` | taihe_shared_library | ANI 库 | system/framework/ |

### 2.6 CJ FFI 目标

| 目标 | 类型 | 输出 |
|------|------|------|
| `cj_nfc_controller_ffi` | ohos_shared_library | libcj_nfc_controller_ffi.z.so |
| `cj_nfc_cardemulation_ffi` | ohos_shared_library | libcj_nfc_cardemulation_ffi.z.so |

### 2.7 SA 配置

| 目标 | 类型 | 输出 |
|------|------|------|
| `nfc_profile` | ohos_sa_profile | 1140.json |

---

## 3. 依赖关系图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              nfc_service                                    │
│                         (libnfc_service.z.so)                               │
└─────────────────────────────────────────────────────────────────────────────┘
        │                 │                │                │
        ▼                 ▼                ▼                ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ nfc_prebuilt │  │ libnfc_hce_  │  │libnfc_ctrl_  │  │ libnfc_tag_  │
│   _config    │  │ interface_stub│  │interface_stub│  │ interface_stub│
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
                                         │                  │
                                         ▼                  ▼
                              ┌─────────────────┐  ┌─────────────────┐
                              │ nfc_inner_kits_ │  │ nfc_inner_kits_ │
                              │    controller   │  │      tags       │
                              └─────────────────┘  └─────────────────┘
                                       │                    │
                                       └──────────┬─────────┘
                                                  ▼
                                        ┌─────────────────┐
                                        │ nfc_inner_kits_ │
                                        │     common      │
                                        └─────────────────┘
                                                  │
                    ┌─────────────────────────────┼─────────────────────────────┐
                    ▼                             ▼                             ▼
           ┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
           │ nfc_notification│          │nci_native_default│          │      etc        │
           │ (notifications) │          │  (NCI adapter)  │          │ (init config)   │
           └─────────────────┘          └─────────────────┘          └─────────────────┘
```

### 3.1 Framework 层依赖

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         JS NAPI Layer (frameworks/js/napi)                  │
├─────────────────┬─────────────────┬─────────────────────────────────────────┤
│    controller   │      tag        │           cardemulation                 │
│  libcontroller  │    libtag       │        libcardemulation                 │
│     .z.so       │    .z.so        │           .z.so                         │
└────────┬────────┴────────┬────────┴─────────────────┬───────────────────────┘
         │                 │                          │
         ▼                 ▼                          ▼
┌─────────────────┐ ┌─────────────────┐  ┌─────────────────────────────────────┐
│nfc_inner_kits_  │ │nfc_inner_kits_  │  │    nfc_inner_kits_card_emulation    │
│  controller     │ │     tags        │  │           + common                  │
└─────────────────┘ └─────────────────┘  └─────────────────────────────────────┘
```

---

## 4. 关键 BUILD.gn 文件

### 4.1 服务层 (services/BUILD.gn)

```gn
# 公共配置
config("nfc_config") {
  visibility = [ ":*" ]
  defines = [ "DEBUG" ]
  
  # Feature defines
  if (nfc_use_vendor_nci_native) {
    defines += [ "USE_VENDOR_NCI_NATIVE" ]
  }
  if (nfc_service_feature_vendor_applications_enabled) {
    defines += [ "VENDOR_APPLICATIONS_ENABLED" ]
  }
  # ... more features
  
  include_dirs = [
    "include",
    "src/ipc/controller",
    "src/ipc/tags",
    "src/ipc/card_emulation",
    # ...
  ]
}

# 服务源码列表
nfc_service_source = [
  "src/nfc_service.cpp",
  "src/nfc_sa_manager.cpp",
  "src/nfc_polling_manager.cpp",
  # ...
]

# 外部依赖
nfc_service_external_deps = [
  "ability_base:want",
  "ability_runtime:ability_manager",
  "access_token:libaccesstoken_sdk",
  "ipc:ipc_core",
  # ...
]

# 主服务目标
ohos_shared_library("nfc_service") {
  sanitize = {
    cfi = true
    boundary_sanitize = true
    integer_overflow = true
    cfi_cross_dso = true
    ubsan = true
  }
  branch_protector_ret = "pac_ret"
  
  sources = nfc_service_source
  public_configs = [ ":nfc_config" ]
  
  deps = [
    ":nfc_prebuilt_config",
    "$NFC_DIR/interfaces/inner_api/cardEmulation:libnfc_hce_interface_stub",
    "$NFC_DIR/interfaces/inner_api/controller:libnfc_controller_interface_stub",
    # ...
  ]
  
  external_deps = nfc_service_external_deps
}
```

### 4.2 Inner API 示例 (interfaces/inner_api/controller/BUILD.gn)

```gn
import("$NFC_DIR/nfc.gni")

# IDL 接口生成
idl_gen_interface("nfc_controller_interface") {
  src_idls = [ "idl/INfcController.idl" ]
  # 生成 Proxy/Stub 代码
}

# 客户端库
ohos_shared_library("nfc_inner_kits_controller") {
  sources = [
    "nfc_controller.cpp",
    "nfc_sa_client.cpp",
    "nfc_controller_callback_stub.cpp",
    # ...
  ]
  
  deps = [
    ":nfc_controller_interface",
    "$NFC_DIR/interfaces/inner_api/common:nfc_inner_kits_common",
  ]
  
  external_deps = [
    "ipc:ipc_core",
    "samgr:samgr_proxy",
    # ...
  ]
}
```

### 4.3 JS N-API 示例 (frameworks/js/napi/controller/BUILD.gn)

```gn
ohos_shared_library("controller") {
  include_dirs = [
    "../common",
    "$NFC_DIR/interfaces/inner_api/common",
    # ...
  ]
  
  sources = [
    "nfc_napi_controller.cpp",
    "nfc_napi_controller_adapter.cpp",
    # ...
  ]
  
  deps = [
    "$NFC_DIR/interfaces/inner_api/controller:nfc_inner_kits_controller",
    "$NFC_DIR/interfaces/inner_api/common:nfc_inner_kits_common",
  ]
  
  relative_install_dir = "module/nfc"
  part_name = "nfc"
  subsystem_name = "communication"
}
```

---

## 5. 编译标志与定义

### 5.1 安全编译选项

所有共享库使用以下安全编译选项：

```gn
sanitize = {
  cfi = true                    # Control Flow Integrity
  boundary_sanitize = true      # 缓冲区边界检查
  integer_overflow = true       # 整数溢出检测
  cfi_cross_dso = true          # 跨 DSO CFI
  ubsan = true                  # 未定义行为检测
}
branch_protector_ret = "pac_ret"  # 返回地址保护
```

### 5.2 条件编译定义

| 定义 | 条件 | 影响 |
|------|------|------|
| `DEBUG` | 始终 | Debug 构建 |
| `USE_VENDOR_NCI_NATIVE` | `nfc_use_vendor_nci_native=true` | 使用厂商 NCI |
| `VENDOR_APPLICATIONS_ENABLED` | `nfc_service_feature_vendor_applications_enabled=true` | 厂商应用支持 |
| `NFC_SIM_FEATURE` | `nfc_sim_feature=true` | SIM 卡支持 |
| `NDEF_WIFI_ENABLED` | `nfc_service_feature_ndef_wifi_enabled=true` | WiFi NDEF |
| `NDEF_BT_ENABLED` | `nfc_service_feature_ndef_bt_enabled=true` | 蓝牙 NDEF |
| `NFC_VIBRATOR_DISABLED` | `nfc_vibrator_disabled=true` | 禁用振动 |
| `NFC_HANDLE_SCREEN_LOCK` | `nfc_handle_screen_lock=true` | 屏幕锁处理 |
| `DTFUZZ_TEST` | `is_asan \|\| use_clang_coverage` | 模糊测试构建 |

### 5.3 NCI 默认实现特定

```gn
cflags_cc = [ "-DNXP_EXTNS=TRUE" ]   # NXP 扩展启用
```

---

## 6. 依赖组件

### 6.1 核心依赖

| 依赖 | 用途 |
|------|------|
| `ability_base:want` | Want 对象处理 |
| `ability_runtime:ability_manager` | Ability 管理 |
| `access_token:libaccesstoken_sdk` | 权限校验 |
| `bundle_framework:appexecfwk_*` | 应用信息 |
| `ipc:ipc_core` | IPC 通信 |
| `safwk:system_ability_fwk` | SA 框架 |
| `samgr:samgr_proxy` | SA 管理 |

### 6.2 可选依赖

| 依赖 | Feature | 用途 |
|------|---------|------|
| `wifi:wifi_sdk` | NDEF_WIFI_ENABLED | WiFi 连接 |
| `bluetooth:btframework` | NDEF_BT_ENABLED | 蓝牙配对 |

---

## 7. 构建命令示例

### 7.1 完整构建

```bash
# 构建整个 NFC 组件
./build.sh --product {product_name} --build-target //foundation/communication/nfc/services:nfc_service

# 构建所有目标
./build.sh --product {product_name} --build-target //foundation/communication/nfc/...
```

### 7.2 构建特定目标

```bash
# 仅构建服务
./build.sh --product {product_name} --build-target //foundation/communication/nfc/services:nfc_service

# 构建 N-API
./build.sh --product {product_name} --build-target //foundation/communication/nfc/frameworks/js/napi/controller:controller
./build.sh --product {product_name} --build-target //foundation/communication/nfc/frameworks/js/napi/tag:tag
./build.sh --product {product_name} --build-target //foundation/communication/nfc/frameworks/js/napi/cardEmulation:cardemulation

# 构建 Inner API
./build.sh --product {product_name} --build-target //foundation/communication/nfc/interfaces/inner_api/common:nfc_inner_kits_common
./build.sh --product {product_name} --build-target //foundation/communication/nfc/interfaces/inner_api/controller:nfc_inner_kits_controller
./build.sh --product {product_name} --build-target //foundation/communication/nfc/interfaces/inner_api/tags:nfc_inner_kits_tags
```

### 7.3 带 Feature 构建

```bash
# 启用厂商 NCI
./build.sh --product {product_name} --gn-args "nfc_use_vendor_nci_native=true"

# 启用所有可选功能
./build.sh --product {product_name} --gn-args "
  nfc_use_vendor_nci_native=true
  nfc_service_feature_vendor_applications_enabled=true
  nfc_sim_feature=true
"
```

---

## 8. 产物输出路径

| 产物类型 | 输出路径 |
|----------|----------|
| 共享库 | `out/{product}/system/lib/` |
| N-API 模块 | `out/{product}/system/lib/module/nfc/` |
| ANI 库 | `out/{product}/system/framework/` |
| 配置文件 | `out/{product}/system/etc/init/` |
| SA 配置 | `out/{product}/system/profile/` |

