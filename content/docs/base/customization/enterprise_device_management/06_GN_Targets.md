# GN Targets与编译配置

## 目的

本文档详细说明EDM组件的GN构建系统、所有编译目标、依赖关系和编译选项。

## 适用范围

- 目标读者：构建工程师、开发者
- 覆盖内容：主要targets、依赖关系、产物、编译选项
- 不包含：所有targets的详细列表

## 关键结论

- EDM使用分层GN结构，约30个targets
- 主要产物类型：shared_library（服务、N-API）、ohos_executable（工具）
- 支持30+个feature flags控制不同功能模块
- 启用安全编译选项（sanitize、CFI等）

## 相关跳转

- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物详情
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 所有配置标志

---

## 主要BUILD.gn文件

### 核心服务层

| BUILD.gn | 主要targets | 说明 |
|-----------|------------|------|
| `services/edm/BUILD.gn` | edmservice | EDM主服务（SA 1601） |
| `services/edm_plugin/BUILD.gn` | device_core_plugin, communication_plugin, sys_service_plugin, need_extra_plugin | 策略插件库 |
| `services/idl/BUILD.gn` | enterprise_device_mgr_idl | IDL接口生成 |

### 接口层

| BUILD.gn | 主要targets | 说明 |
|-----------|------------|------|
| `interfaces/inner_api/BUILD.gn` | edmservice_kits, plugin_kits | 内部C++ API |
| `interfaces/kits/*/BUILD.gn` | 18个Manager模块 | N-API接口库 |
| `interfaces/ets/ani/BUILD.gn` | ani_edm_package | ArkTS原生接口 |
| `framework/extension/BUILD.gn` | enterprise_admin_extension | Extension框架 |

### 公共代码层

| BUILD.gn | 主要targets | 说明 |
|-----------|------------|------|
| `common/native/BUILD.gn` | edm_commom | 原生工具库 |
| `common/external/BUILD.gn` | edm_external_adapters | 外部适配器 |
| `common/config/BUILD.gn` | coverage_flags | 编译选项配置 |

### 工具层

| BUILD.gn | 主要targets | 说明 |
|-----------|------------|------|
| `tools/edm/BUILD.gn` | edm | EDM命令行工具 |

### 配置层

| BUILD.gn | 主要targets | 说明 |
|-----------|------------|------|
| `sa_profile/BUILD.gn` | edm_sa_profile | SA配置（1601.json） |
| `etc/init/BUILD.gn` | edm.cfg | 服务启动配置 |
| `etc/param/BUILD.gn` | edm.para, edm.para.dac | 系统参数配置 |

---

## 核心Targets详解

### 1. edmservice (主服务)

**位置**: `services/edm/BUILD.gn:33-78`

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | `ohos_shared_library` | 共享库，SystemAbility类型 |
| 名称 | `libedmservice.z.so` | 输出库名 |
| 源文件 | `enterprise_device_mgr_stub.cpp` + 根据feature开关添加多个源 | 主要实现文件 |
| 公共配置 | `:edm_config`, `../idl:enterprise_device_mgr_idl_gen_config` | 配置和IDL生成 |
| 外部依赖 | ability_runtime, bundle_framework, ipc, safwk, samgr等20+个组件 | 系统依赖 |
| 宏定义 | `_ARM64_`或`_X86_64_`, `EDM_SUPPORT_ALL_ENABLE` | 架构和功能宏 |
| 安装 | `true` (通过sa_profile安装) | 安装到系统 |

**关键源文件**（条件编译）：
- `ability/ability_controller.cpp` - Ability控制
- `admin/admin.cpp`, `admin/super_device_admin.cpp`等 - 管理员类型
- `admin_manager.cpp` - 管理员管理
- `policy_manager.cpp` - 策略管理
- `plugin_manager.cpp` - 插件管理
- `permission_checker.cpp` - 权限检查
- `database/edm_rdb_data_manager.cpp` - RDB管理
- `connection/*.cpp` - IPC连接管理
- `query_policy/*.cpp` - 70+个策略查询
- `strategy/*.cpp` - 执行策略
- `observer/*.cpp` - 观察者
- `watermark/*.cpp` - 水印

证据：`services/edm/BUILD.gn:80-178`

### 2. device_core_plugin (设备核心插件)

**位置**: `services/edm_plugin/BUILD.gn:60-125`

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | `ohos_shared_library` | 共享库 |
| 名称 | `edm_plugin/libdevice_core_plugin.so` | 插件库 |
| 插件数量 | ~60个 | 设备核心策略 |
| 源文件 | device_core/*.cpp | 各种设备策略插件 |

**典型插件示例**：
- `disable_camera_plugin.cpp` - 禁用相机
- `reset_factory_plugin.cpp` - 恢复出厂
- `lock_screen_plugin.cpp` - 锁屏
- `set_wall_paper_plugin.cpp` - 设置壁纸

证据：`services/edm_plugin/BUILD.gn:80-118`

### 3. communication_plugin (通信插件)

**位置**: `services/edm_plugin/BUILD.gn:150-190`

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | `ohos_shared_library` | 共享库 |
| 名称 | `edm_plugin/libcommunication_plugin.so` | 通信管理插件库 |
| 插件数量 | ~30个 | 网络和通信策略 |
| 源文件 | communication/*.cpp | 网络策略插件 |

**典型插件**：
- `set_wifi_disabled_plugin.cpp` - WiFi开关
- `disable_bluetooth_plugin.cpp` - 蓝牙开关
- `disallow_vpn_plugin.cpp` - VPN禁用
- `global_proxy_plugin.cpp` - 全局代理

证据：`services/edm_plugin/BUILD.gn:158-188`

### 4. sys_service_plugin (系统服务插件)

**位置**: `services/edm_plugin/BUILD.gn:240-286`

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | `ohos_shared_library` | 共享库 |
| 名称 | `edm_plugin/libsys_service_plugin.so` | 系统服务插件库 |
| 插件数量 | ~30个 | 系统服务策略 |
| 源文件 | sys_service/*.cpp | 系统服务策略插件 |

**典型插件**：
- `password_policy_plugin.cpp` - 密码策略
- `install_user_certificate_plugin.cpp` - 证书安装
- `manage_keep_alive_apps_plugin.cpp` - 常驻应用管理
- `set_watermark_image_plugin.cpp` - 水印

证据：`services/edm_plugin/BUILD.gn:248-276`

### 5. need_extra_plugin (额外功能插件)

**位置**: `services/edm_plugin/BUILD.gn:295-320`

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | `ohos_shared_library` | 共享库 |
| 名称 | `edm_plugin/libneed_extra_plugin.so` | 额外功能插件库 |
| 插件数量 | ~6个 | 额外策略 |
| 源文件 | need_extra/*.cpp | 额外功能插件 |

**典型插件**：
- `manage_freeze_exempted_apps_plugin.cpp` - 应用冻结豁免
- `is_app_kiosk_allowed_plugin.cpp` - Kiosk允许检查
- `manage_user_non_stop_apps_plugin.cpp` - 用户不可停止应用

证据：`services/edm_plugin/BUILD.gn:303-320`

### 6. N-API Manager Targets

每个Manager都是独立的`ohos_shared_library` target：

| Target | 输出库 | 模块 | 说明 |
|--------|---------|------|------|
| adminmanager | `module/enterprise/libadminmanager.so` | adminManager | 管理员管理 |
| accountmanager | `module/enterprise/libaccountmanager.so` | accountManager | 账户管理 |
| applicationmanager | `module/enterprise/libapplicationmanager.so` | applicationManager | 应用管理 |
| devicemanager | `module/enterprise/libdevicesettings.so` | deviceSettings | 设备设置 |
| devicecontrol | `module/enterprise/libdevicecontrol.so` | deviceControl | 设备控制 |
| bundlemanger | `module/enterprise/libbundlemanger.so` | bundleManager | Bundle管理 |
| networkmanager | `module/enterprise/libnetworkmanager.so` | networkManager | 网络管理 |
| securitymanager | `module/enterprise/libsecuritymanager.so` | securityManager | 安全管理 |
| systemmanager | `module/enterprise/libsystemmanager.so` | systemManager | 系统管理 |
| wifimanager | `module/enterprise/libwifimanager.so` | wifiManager | WiFi管理 |
| bluetoothmanager | `module/enterprise/libbluetoothmanager.so` | bluetoothManager | 蓝牙管理 |
| telephonymanager | `module/enterprise/libtelephonymanager.so` | telephonyManager | 电话管理 |
| usbmanager | `module/enterprise/libusbmanager.so` | usbManager | USB管理 |
| locationmanager | `module/enterprise/liblocationmanager.so` | locationManager | 位置管理 |
| restrictions | `module/enterprise/librestrictions.so` | restrictions | 限制策略 |

证据：`interfaces/kits/*/BUILD.gn`

### 7. 公共库Targets

| Target | 类型 | 输出库 | 说明 |
|--------|------|---------|------|
| edm_commom | `ohos_shared_library` | `libedm_commom.so` | 原生工具 |
| edmservice_kits | `ohos_shared_library` | `libedmservice_kits.so` | 内部API Proxy |
| plugin_kits | `ohos_shared_library` | `libplugin_kits.so` | 插件接口 |
| edm_external_adapters | `ohos_shared_library` | `libedm_external_adapters.so` | 外部适配器 |

证据：`common/native/BUILD.gn`, `interfaces/inner_api/BUILD.gn`

---

## 依赖关系图

### 整体依赖层次

```
N-API模块 (17个)
    ↓ depends
内部API层 (edmservice_kits, plugin_kits)
    ↓ depends
EDM主服务 (edmservice)
    ↓ depends
插件库 (4个plugin库)
    ↓ depends
公共层 (edm_commom, external_adapters)
    ↓ depends
系统服务 (ability_runtime, bundle_framework等)
```

### 关键依赖说明

| 调用者 | 依赖 | 类型 |
|--------|------|------|
| 所有N-API Manager | edmservice_kits | 通过Proxy调用IPC |
| edmservice | edmservice_kits | 服务端实现 |
| edmservice | plugin_kits | 插件接口 |
| edmservice | edm_commom | 工具类和错误处理 |
| 所有Manager | edm_external_adapters | BundleManager、AppManager等 |
| edmservice | 所有系统服务 | ability_runtime, bundle_framework, ipc等 |

---

## 编译宏和Feature Flags

### 主要编译宏

| 宏 | 定义位置 | 条件 | 说明 |
|-----|---------|------|------|
| `_ARM64_` | `services/edm/BUILD.gn:44` | `target_cpu == "arm64"` | ARM64架构 |
| `_X86_64_` | `services/edm/BUILD.gn:49` | `target_cpu == "x86_64"` | X86_64架构 |
| `EDM_SUPPORT_ALL_ENABLE` | `services/edm/BUILD.gn:81` | `enterprise_device_management_support_all` | 全功能支持 |
| `FEATURE_CHARGING_TYPE_SETTING` | `common/config/common.gni:19` | 充电类型设置特性 |

### Feature Flags

来自`common/config/common.gni:16-162`的30+个组件开关：

| Flag | 默认值 | 依赖 | 功能 |
|------|---------|------|------|
| `wifi_edm_enable` | false | `global_parts_info.communication_wifi` | WiFi管理 |
| `bluetooth_edm_enable` | false | `global_parts_info.communication_bluetooth` | 蓝牙管理 |
| `location_edm_enable` | false | `global_parts_info.location_location` | 位置服务 |
| `os_account_edm_enable` | false | `global_parts_info.account_os_account` | OS账户 |
| `power_manager_edm_enable` | false | `global_parts_info.powermgr_power_manager` | 电源管理 |
| `camera_framework_edm_enable` | false | `global_parts_info.multimedia_camera_framework` | 相机 |
| `useriam_edm_enable` | false | `global_parts_info.useriam_user_auth_framework` | 用户认证 |
| `storage_service_edm_enable` | false | `global_parts_info.filemanagement_storage_service` | 存储服务 |
| `netmanager_base_edm_enable` | false | `global_parts_info.communication_netmanager_base` | 网络管理基础 |
| `telephony_core_edm_enable` | false | `global_parts_info.telephony_core_service` | 电话核心 |

证据：`common/config/common.gni:16-162`

---

## 安全编译选项

### Sanitize配置

所有主要targets启用了以下安全选项：

```gn
sanitize = {
  boundary_sanitize = true       # 边界检查
  cfi = true                      # 控制流完整性
  cfi_cross_dso = true           # 跨DSO CFI
  debug = false                    # 非调试模式
  integer_overflow = true       # 整数溢出检查
  ubsan = true                   # 未定义行为检查
}

branch_protector_ret = "pac_ret"  # ARM PAC分支保护
```

证据：`services/edm/BUILD.gn:133-139`（推断）

### Coverage配置

```gn
// common/config/BUILD.gn
declare_args() {
  enterprise_device_management_feature_coverage = false
}

config("coverage_flags") {
  if (enterprise_device_management_feature_coverage) {
    cflags = [ "--coverage" ]
    cflags_cc = [ "--coverage" ]
    ldflags = [ "--coverage" ]
  }
}
```

---

## bundle.json构建组

### group_type定义

来自`bundle.json:114-157`：

**base_group**: 无

**fwk_group** (框架层）：
- `enterprise_admin_extension` - Extension框架
- `enterprise_admin_extension_module` - Extension模块

**service_group** (服务层）：
- `edmservice` - EDM主服务
- `device_core_plugin`, `communication_plugin`, `sys_service_plugin`, `need_extra_plugin` - 4个插件库
- `edm_sa_profile` - SA配置
- `edm.cfg`, `edm.para`, `edm.para.dac` - 配置文件

### inner_kits定义

来自`bundle.json:160-191`：

| 库名 | 头文件base | 头文件列表 |
|------|----------|----------|
| edmservice_kits | `interfaces/inner_api` | ienterprise_device_mgr.h, restrictions_proxy等 |
| plugin_kits | `interfaces/inner_api/plugin_kits` | iplugin.h, ipolicy_manager.h, 序列化工具 |

---

## 相关跳转

- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 产物清单和安装路径
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 完整配置标志列表
