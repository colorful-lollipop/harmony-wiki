# 构建与产物

## 概述

本文档描述 Bundle Framework 的构建系统配置、编译产物清单和安装路径。

**目标受众**：构建工程师、系统集成工程师

---

## 1. 构建系统

### 1.1 GN 构建配置

Bundle Framework 使用 GN (Generate Ninja) 作为构建系统。

**根构建文件**: `BUILD.gn`

**主聚合目标**: `bms_target`

```gn
group("bms_target") {
  deps = [
    "common:common_target",
    "interfaces/inner_api/appexecfwk_base:appexecfwk_base",
    "interfaces/inner_api/appexecfwk_core:appexecfwk_core",
    "interfaces/inner_api/appexecfwk_core:appexecfwk_core_headers",
    "interfaces/inner_api/appexecfwk_core:bundlemgr_mini",
    "interfaces/inner_api/bundlemgr_extension:bundlemgr_extension",
    "interfaces/kits/cj:cj_bundle_manager_ffi",
    "interfaces/kits/js:napi_packages",
    "interfaces/kits/native/app_detail_ability:app_detail_ability",
    "interfaces/kits/native/bundle:bundle_ndk",
    "sa_profile:appexecfwk_sa_profile",
    "services/bundlemgr:bms_target",
  ]
  if (bundle_framework_graphics) {
    deps += [ "interfaces/inner_api/bundlemgr_graphics:bundlemgr_graphics" ]
  }
}
```

**证据来源**: `BUILD.gn:17-35`

---

## 2. GN Targets

### 2.1 服务层 Targets

**文件**: `services/bundlemgr/BUILD.gn`

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `bms_target` | source_set | libbms.z.so | 主服务 |
| `bms_host` | shared_library | libbms_host.z.so | IPC Host |
| `bms_proxy` | shared_library | libbms_proxy.z.so | IPC Proxy |
| `bms_mini` | source_set | libbms_mini.z.a | 轻量版 |

**关键配置**:

```gn
config("bundlemgr_common_config") {
  include_dirs = [
    "include",
    "include/aot",
    "include/bundle_permission",
    # ... 更多 include
  ]

  defines = [
    "APP_LOG_TAG = \"BMS\"",
    "LOG_DOMAIN = 0xD001120",
  ]
}
```

### 2.2 JS N-API Targets

**文件**: `interfaces/kits/js/BUILD.gn`

```gn
group("napi_packages") {
  deps = []
  if (support_jsapi) {
    deps += [
      "app_control:appcontrol",
      "bundle_manager:bundle_manager_common",
      "bundle_manager:bundlemanager",
      "bundle_monitor:bundlemonitor",
      "bundle_resource:bundle_res_common",
      "bundle_resource:bundleresourcemanager",
      "bundlemgr:bundle",
      "default_app:defaultappmanager",
      "free_install:freeinstall",
      "installer:installer",
      "launcher_bundle_manager:launcherbundlemanager",
      "launchermgr:innerbundlemanager",
      "overlay:overlay",
      "package:package",
      "shortcut_manager:shortcutmanager",
      "zip:tools_zip",
    ]
  }
}
```

---

## 3. Feature 开关

**文件**: `appexecfwk.gni`

| 开关 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `bundle_framework_free_install` | true | bool | 自由安装 |
| `bundle_framework_default_app` | true | bool | 默认应用管理 |
| `bundle_framework_launcher` | true | bool | 启动器服务 |
| `bundle_framework_sandbox_app` | true | bool | 沙箱应用 |
| `bundle_framework_quick_fix` | true | bool | 快速修复 |
| `bundle_framework_app_control` | true | bool | 应用控制 |
| `bundle_framework_overlay_install` | true | bool | 叠加安装 |
| `bundle_framework_bundle_resource` | true | bool | 包资源 |
| `bundle_framework_graphics` | true | bool | 图形相关 |
| `bundle_framework_power_mgr_enable` | true | bool | 电源管理 |
| `distributed_bundle_framework` | true | bool | 分布式包管理 |

**证据来源**: `appexecfwk.gni:28-42`

### 条件依赖

```gn
# 如果没有 ability_runtime，则禁用 free_install
if (!defined(global_parts_info.ability_ability_runtime)) {
  ability_runtime_enable = false
  bundle_framework_free_install = false
}

# 如果没有 account_os_account，禁用 free_install
if (!defined(global_parts_info.account_os_account)) {
  account_enable = false
  bundle_framework_free_install = false
}
```

---

## 4. 构建命令

### 4.1 完整构建

```bash
# 构建所有 bundle framework targets
./build.sh --product-name <product> --build-target bundle_framework

# 示例
./build.sh --product-name rk3568 --build-target bundle_framework
./build.sh --product-name ohos-sdk --build-target bundle_framework
```

### 4.2 单独构建服务层

```bash
./build.sh --product-name rk3568 --build-target services/bundlemgr:bms_target
```

### 4.3 带 Feature 构建

```bash
./build.sh --product-name rk3568 --build-target bundle_framework \
  --gn-args bundle_framework_free_install=true
```

---

## 5. 编译产物

### 5.1 Foundation 进程产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `libbms.z.so` | `out/<product>/foundation/bundlemanager/bundle_framework/services/bundlemgr/` | BundleMgrService (SA 401) |
| `libbms_host.z.so` | `out/<product>/foundation/bundlemanager/bundle_framework/services/bundlemgr/` | IPC Host |
| `libbms_proxy.z.so` | `out/<product>/foundation/bundlemanager/bundle_framework/services/bundlemgr/` | IPC Proxy |

### 5.2 特权进程产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `libinstalls.z.so` | `out/<product>/foundation/bundlemanager/bundle_framework/services/bundlemgr/` | InstalldService (SA 511) |

### 5.3 N-API 产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `libbundle_ndk.z.so` | `out/<product>/foundation/bundlemanager/bundle_framework/interfaces/kits/native/` | Native NDK 库 |
| `@bundle.bundlemanager` | SDK | JS N-API 包 |

---

## 6. 头文件产物

### 6.1 Inner API 头文件

| 头文件 | 位置 | 说明 |
|--------|------|------|
| `ability_info.h` | `interfaces/inner_api/appexecfwk_base/include/` | Ability 信息 |
| `application_info.h` | `interfaces/inner_api/appexecfwk_base/include/` | 应用信息 |
| `bundle_info.h` | `interfaces/inner_api/appexecfwk_base/include/` | 包信息 |
| `bundle_mgr_interface.h` | `interfaces/inner_api/appexecfwk_core/include/` | IPC 接口 |
| `bundle_installer_interface.h` | `interfaces/inner_api/appexecfwk_core/include/` | 安装器接口 |

### 6.2 Native 头文件

| 头文件 | 位置 | 说明 |
|--------|------|------|
| `native_interface_bundle.h` | `interfaces/kits/native/bundle/include/` | 主要 Native 接口 |
| `bundle_manager_common.h` | `interfaces/kits/native/bundle/include/` | 公共类型和错误码 |

---

## 7. SDK 产物

### 7.1 JS API SDK

```javascript
// 导入方式
import bundleManager from '@bundle.bundleManager';
import installer from '@bundle.installer';
```

### 7.2 NDK SDK

```c
// 包含头文件
#include <native_interface_bundle.h>
#include <bundle_manager_common.h>
```

---

## 8. 安装路径

### 8.1 系统路径

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `libbms.z.so` | `/system/lib/` | BundleMgrService |
| `libinstalls.z.so` | `/system/lib/` | InstalldService |
| `libbundle_ndk.z.so` | `/system/lib/` | Native NDK |

### 8.2 数据路径

| 路径 | 说明 |
|------|------|
| `/data/bms/` | Bundle Manager 数据目录 |
| `/data/bms/app/` | 应用数据 |
| `/data/bms/install/` | 安装临时文件 |
| `/data/bms/backup/` | 备份数据 |

---

## 9. 加载关系

```
应用进程
  │
  ├── load libbundle_ndk.z.so (Native 应用)
  │
  └── load @bundle.bundlemanager (JS 应用)
         │
         └── IPC 调用
                │
                ▼
         Foundation 进程
                │
                ├── load libbms.z.so
                │      │
                │      ├── IPC Host (libbms_host.z.so)
                │      │
                │      └── IPC Proxy (libbms_proxy.z.so)
                │
                └── IPC 调用 InstalldService
                       │
                       └── load libinstalls.z.so
```

---

## 10. 常见问题

### 10.1 编译失败

**检查清单**:
1. hb 工具是否正确安装
2. product name 是否有效
3. 依赖的子系统是否已构建

### 10.2 Feature 开关

```bash
# 启用某功能
--gn-args bundle_framework_free_install=true

# 禁用某功能
--gn-args bundle_framework_free_install=false
```

---

## 11. 延伸阅读

- [03_CodeMap](03_CodeMap.md) - 代码地图
- [08_Internals](08_Internals.md) - 内部实现细节
- [08_FAQ](08_FAQ.md) - 构建相关问题
