# 目录结构与模块职责

## 目的

本文档详细说明EDM组件的代码组织、模块职责和依赖关系。

## 适用范围

- 目标读者：开发者、架构师
- 覆盖内容：完整目录树、每个模块职责、核心类和接口
- 不包含：测试代码

## 关键结论

- EDM采用清晰的分层架构：common → interfaces → services
- 插件系统是核心扩展机制，包含126+个策略插件
- 服务端采用Manager模式：AdminManager、PolicyManager、PluginManager

## 相关跳转

- [01_Project_Positioning.md](01_Project_Positioning.md) - 项目定位和边界
- [03_Architecture.md](03_Architecture.md) - 详细架构说明

---

## 完整目录结构

```
/base/customization/enterprise_device_management
├── common/                              # 公共代码
│   ├── config/                           # GN配置
│   │   ├── common.gni                    # Feature flags定义
│   │   └── BUILD.gn                     # 配置编译选项
│   ├── external/                          # 外部系统封装
│   │   ├── include/
│   │   │   ├── bundle_manager_wrapper.h
│   │   │   ├── app_manager_wrapper.h
│   │   │   ├── os_account_manager_wrapper.h
│   │   │   └── external_manager_factory.h
│   │   └── src/
│   │       └── edm_access_token_manager_impl.cpp
│   └── native/                            # 原生代码
│       ├── include/
│       │   ├── edm_errors.h                 # 错误码定义
│       │   ├── edm_constants.h             # 常量定义
│       │   ├── edm_log.h                    # 日志宏
│       │   ├── func_code.h                  # IPC功能码
│       │   ├── admin_type.h                 # 管理员类型
│       │   └── policy_struct.h              # 策略数据结构
│       └── src/
│           ├── edm_errors.cpp                # 错误处理
│           ├── usb_id_parser.cpp           # USB ID解析
│           ├── wifi_id_parser.cpp          # WiFi ID解析
│           ├── rdb_operations.cpp          # RDB操作
│           └── security_report.cpp         # 安全上报
│
├── framework/                            # 框架层
│   └── extension/
│       ├── include/
│       │   ├── enterprise_admin_extension.h   # Extension基类
│       │   └── ienterprise_admin.h          # Extension接口
│       └── src/
│           ├── enterprise_admin_extension.cpp   # Extension实现
│           └── enterprise_admin_stub.cpp      # IPC Stub
│
├── interfaces/                           # 接口层
│   ├── ets/ani/                          # ArkTS原生接口（新一代）
│   │   ├── BUILD.gn
│   │   └── enterprise_device_management/
│   │       ├── adminManager/
│   │       ├── networkManager/
│   │       ├── securityManager/
│   │       └── restrictions/
│   ├── inner_api/                         # 内部C++ API
│   │   ├── include/
│   │   │   ├── common/                # 通用接口
│   │   │   │   ├── ienterprise_device_mgr.h
│   │   │   │   ├── enterprise_device_mgr_proxy.h
│   │   │   │   ├── edm_ipc_interface_code.h
│   │   │   │   ├── edm_load_callback.h
│   │   │   │   ├── result_code.h
│   │   │   │   ├── managed_policy.h
│   │   │   │   └── ent_info.h
│   │   │   └── plugin_kits/            # 插件接口
│   │   │       ├── iplugin.h            # 插件基接口
│   │   │       ├── iplugin_manager.h    # 插件管理器接口
│   │   │       ├── ipolicy_manager.h   # 策略管理接口
│   │   │       └── utils/              # 序列化工具
│   │   ├── src/
│   │   │   ├── common/               # Proxy实现
│   │   │   │   ├── enterprise_device_mgr_proxy.cpp
│   │   │   │   ├── edm_load_manager.cpp
│   │   │   │   └── result_code.cpp
│   │   │   └── plugin_kits/         # 插件工具类
│   │   └── */include/src/             # 各Manager Proxy
│   └── kits/                             # JS/TS开发者API（NAPI）
│       ├── account_manager/
│       ├── admin_manager/
│       ├── application_manager/
│       ├── bluetooth_manager/
│       ├── browser/
│       ├── bundle_manager/
│       ├── common/
│       ├── common_kits/
│       ├── datetime_manager/
│       ├── device_control/
│       ├── device_info/
│       ├── device_settings/
│       ├── enterprise_admin_extension/
│       ├── enterprise_admin_extension_context/
│       ├── location_manager/
│       ├── network_manager/
│       ├── restrictions/
│       ├── security_manager/
│       ├── system_manager/
│       ├── telephony_manager/
│       ├── usb_manager/
│       └── wifi_manager/
│
├── services/                             # 服务实现
│   ├── edm/                                # EDM主服务
│   │   ├── include/
│   │   │   ├── enterprise_device_mgr_ability.h  # 主Ability类
│   │   │   ├── enterprise_device_mgr_stub.h     # IPC Stub
│   │   │   ├── admin_manager.h                # 管理员管理
│   │   │   ├── policy_manager.h               # 策略管理
│   │   │   ├── plugin_manager.h              # 插件管理
│   │   │   ├── permission_checker.h           # 权限检查
│   │   │   ├── database/
│   │   │   │   ├── edm_rdb_data_manager.h
│   │   │   │   ├── edm_rdb_open_callback.h
│   │   │   │   └── edm_rdb_filed_const.h
│   │   │   ├── admin/
│   │   │   │   ├── admin.h
│   │   │   │   ├── super_device_admin.h
│   │   │   │   ├── sub_super_device_admin.h
│   │   │   │   ├── device_admin.h
│   │   │   │   ├── byod_admin.h
│   │   │   │   └── virtual_device_admin.h
│   │   │   ├── connection/
│   │   │   │   ├── ienterprise_connection.h
│   │   │   │   ├── enterprise_admin_connection.h
│   │   │   │   ├── enterprise_admin_proxy.h
│   │   │   │   ├── enterprise_conn_manager.h
│   │   │   │   ├── enterprise_bundle_connection.h
│   │   │   │   ├── enterprise_account_connection.h
│   │   │   │   ├── enterprise_kiosk_connection.h
│   │   │   │   ├── enterprise_market_connection.h
│   │   │   │   ├── enterprise_update_connection.h
│   │   │   │   └── enterprise_collect_log_connection.h
│   │   │   ├── query_policy/              # 策略查询（70+个）
│   │   │   │   ├── ipolicy_query.h
│   │   │   │   ├── password_policy_query.h
│   │   │   │   ├── allowed_install_bundles_query.h
│   │   │   │   └── ... (70+个查询类)
│   │   │   ├── strategy/
│   │   │   │   ├── execute_strategy.h
│   │   │   │   ├── enhance_execute_strategy.h
│   │   │   │   └── single_execute_strategy.h
│   │   │   ├── observer/
│   │   │   │   ├── iadmin_observer.h
│   │   │   │   └── application_state_observer.h
│   │   │   ├── watermark/
│   │   │   │   ├── watermark_observer_manager.h
│   │   │   │   └── watermark_observer.h
│   │   │   └── ability/
│   │   │       ├── ability_controller.h
│   │   │       ├── ability_controller_factory.h
│   │   │       ├── ui_ability_controller.h
│   │   │       └── app_service_extension_controller.h
│   │   └── src/
│   │       ├── enterprise_device_mgr_ability.cpp
│   │       ├── enterprise_device_mgr_stub.cpp
│   │       ├── admin_manager.cpp
│   │       ├── policy_manager.cpp
│   │       ├── plugin_manager.cpp
│   │       ├── permission_checker.cpp
│   │       ├── database/
│   │       ├── connection/
│   │       ├── query_policy/
│   │       ├── strategy/
│   │       ├── observer/
│   │       ├── watermark/
│   │       └── ability/
│   │   └── system_service_start_handler.cpp
│   ├── edm_plugin/                          # 插件实现
│   │   ├── include/
│   │   │   ├── network/
│   │   │   │   ├── wifi_policy.h
│   │   │   │   └── bluetooth_policy.h
│   │   │   └── utils/
│   │   └── src/
│   │       ├── 126+个*_plugin.cpp文件         # 策略插件实现
│   │       │   ├── device_core/            # 设备核心（60+个）
│   │       │   │   ├── disable_camera_plugin.cpp
│   │       │   │   ├── disable_printer_plugin.cpp
│   │       │   │   ├── reboot_plugin.cpp
│   │       │   │   └── ...
│   │       │   ├── communication/         # 通信（30+个）
│   │       │   │   ├── set_wifi_disabled_plugin.cpp
│   │       │   │   ├── disable_bluetooth_plugin.cpp
│   │       │   │   └── ...
│   │       │   ├── sys_service/            # 系统服务（30+个）
│   │       │   │   ├── password_policy_plugin.cpp
│   │       │   │   ├── install_user_certificate_plugin.cpp
│   │       │   │   └── ...
│   │       │   └── need_extra/             # 额外功能（10+个）
│   │       │       ├── manage_keep_alive_apps_plugin.cpp
│   │       │       └── ...
│   └── idl/                                # IDL接口定义
│       ├── IEnterpriseDeviceMgrIdl.idl    # 主服务IDL
│       └── AdminType.idl                    # 管理员类型IDL
│
├── sa_profile/                            # SA配置
│   └── 1601.json                            # EDM服务SA 1601配置
│
├── etc/                                  # 进程配置
│   ├── init/
│   │   └── edm.cfg                       # EDM服务启动配置
│   └── param/
│       ├── edm.para                       # 系统参数配置
│       └── edm.para.dac                  # 参数权限配置
│
├── tools/                                 # 工具
│   └── edm/
│       ├── src/
│       │   └── edm_main.cpp              # EDM命令行工具
│       └── BUILD.gn
│
└── wiki/                                  # 文档
```

---

## 顶层目录职责

### 1. common/

| 子目录 | 职责 | 证据 |
|--------|------|------|
| config | GN构建配置、feature flags | `common/config/common.gni:16-162` |
| external | 外部系统服务封装（Bundle、AppManager等） | `common/external/include/*.h` |
| native | 原生代码、错误处理、工具类 | `common/native/include/*.h` |

### 2. framework/

| 子目录 | 职责 | 证据 |
|--------|------|------|
| extension | EnterpriseAdminExtension框架实现 | `framework/extension/include/*.h` |

### 3. interfaces/

| 子目录 | 职责 | 证据 |
|--------|------|------|
| ets/ani | ArkTS原生接口（新一代TS接口） | `interfaces/ets/ani/*.cpp` |
| inner_api | 内部C++ API（Proxy、插件接口） | `interfaces/inner_api/include/*.h` |
| kits | JS/TS开发者API（N-API绑定） | `interfaces/kits/*/_addon.cpp` |

### 4. services/

| 子目录 | 职责 | 证据 |
|--------|------|------|
| edm | EDM主服务实现（SA 1601） | `services/edm/include/*.h` |
| edm_plugin | 策略插件实现（126+个） | `services/edm_plugin/src/*_plugin.cpp` |
| idl | IPC接口定义（IDL） | `services/idl/*.idl` |

### 5. sa_profile/

| 文件 | 职责 | 证据 |
|------|------|------|
| 1601.json | EDM SystemAbility配置 | `sa_profile/1601.json:3-21` |

### 6. etc/

| 子目录 | 职责 | 证据 |
|--------|------|------|
| init | EDM进程启动配置 | `etc/init/edm.cfg` |
| param | 系统参数配置 | `etc/param/edm.para` |

### 7. tools/

| 目录 | 职责 | 证据 |
|------|------|------|
| edm | EDM命令行工具 | `tools/edm/src/edm_main.cpp` |

---

## services/edm主模块

### 核心Manager类

| 类 | 文件 | 职责 | 关键方法 |
|---|------|------|----------|
| EnterpriseDeviceMgrAbility | `enterprise_device_mgr_ability.h` | 主系统服务、SA入口 | `OnStart()`、`OnStop()`、`OnAddSystemAbility()` |
| AdminManager | `admin_manager.h` | 设备管理员生命周期管理 | `EnableAdmin()`、`DisableAdmin()`、`GetAdmins()` |
| PolicyManager | `policy_manager.h` | 策略持久化和查询 | `SetPolicy()`、`GetPolicy()`、`RemoveAdminPolicy()` |
| PluginManager | `plugin_manager.h` | 插件加载和执行 | `HandlePolicy()`、`GetPolicy()`、`LoadPlugin()` |
| PermissionChecker | `permission_checker.h` | 权限验证 | `CheckCallerPermission()`、`VerifyCallingPermission()` |
| EdmRdbDataManager | `database/edm_rdb_data_manager.h` | RDB数据库管理 | `Init()`、`Insert()`、`Query()` |
| EnterpriseConnManager | `connection/enterprise_conn_manager.h` | IPC连接管理 | `ConnectAbility()`、`DisConnect()` |

### 管理员类型

| Admin类 | 说明 | 权限级别 |
|---------|------|----------|
| SuperDeviceAdmin | 超级设备管理员 | 最高 |
| SubSuperDeviceAdmin | 子超级管理员 | 高 |
| DeviceAdmin | 普通设备管理员 | 中等 |
| ByodAdmin | BYOD管理员 | 中等 |
| VirtualDeviceAdmin | 虚拟管理员（委派策略用） | 中等 |

证据：`services/edm/include/admin/*.h`

### 查询策略

目录`query_policy/`包含70+个策略查询实现：
- `disable_camera_query`
- `password_policy_query`
- `allowed_install_bundles_query`
- `disallowed_running_bundles_query`
- 等等...

每个查询类实现`IPolicyQuery`接口，提供：
- `GetPolicy()` - 获取策略值
- `Merge()` - 合并多管理员策略

证据：`services/edm/include/query_policy/ipolicy_query.h`

---

## interfaces/kits管理器模块

### 17个N-API管理器

| 管理器 | 模块名 | 文件 | 功能范围 |
|--------|---------|------|----------|
| AdminManager | enterprise.adminManager | `admin_manager_addon.cpp` | 管理员激活/禁用、授权 |
| AccountManager | enterprise.accountManager | `account_manager_addon.cpp` | OS账户管理 |
| ApplicationManager | enterprise.applicationManager | `application_manager_addon.cpp` | 应用安装/卸载、Kiosk模式 |
| BluetoothManager | enterprise.bluetoothManager | `bluetooth_manager_addon.cpp` | 蓝牙策略 |
| Browser | enterprise.browser | `browser_addon.cpp` | 浏览器策略 |
| BundleManager | enterprise.bundleManager | `bundle_manager_addon.cpp` | 应用包管理 |
| DateTimeManager | enterprise.dateTimeManager | `datetime_manager_addon.cpp` | 日期时间设置 |
| DeviceControl | enterprise.deviceControl | `device_control_addon.cpp` | 设备控制（重启/关机） |
| DeviceInfo | enterprise.deviceInfo | `device_info_addon.cpp` | 设备信息查询 |
| DeviceSettings | enterprise.deviceSettings | `device_settings_addon.cpp` | 设备设置管理 |
| LocationManager | enterprise.locationManager | `location_manager_addon.cpp` | 位置策略 |
| NetworkManager | enterprise.networkManager | `network_manager_addon.cpp` | 网络配置 |
| Restrictions | enterprise.restrictions | `restrictions_addon.cpp` | 应用限制 |
| SecurityManager | enterprise.securityManager | `security_manager_addon.cpp` | 安全策略 |
| SystemManager | enterprise.systemManager | `system_manager_addon.cpp` | 系统管理 |
| TelephonyManager | enterprise.telephonyManager | `telephony_manager_addon.cpp` | 电话策略 |
| UsbManager | enterprise.usbManager | `usb_manager_addon.cpp` | USB策略 |
| WifiManager | enterprise.wifiManager | `wifi_manager_addon.cpp` | WiFi策略 |
| CommonManager | enterprise.common | `common_manager_addon.cpp` | 通用枚举和事件 |

证据：`interfaces/kits/*/src/*_addon.cpp`

---

## interfaces/inner_api插件接口

### 核心接口

| 接口 | 文件 | 职责 |
|------|------|------|
| IPlugin | `plugin_kits/include/iplugin.h` | 插件基接口 |
| IPluginManager | `plugin_kits/include/iplugin_manager.h` | 插件管理器接口 |
| IPolicyManager | `plugin_kits/include/ipolicy_manager.h` | 策略管理接口 |
| IPolicySerializer | `plugin_kits/include/ipolicy_serializer.h` | 策略序列化接口 |
| IPluginTemplate | `plugin_kits/include/iplugin_template.h` | 策略模板 |

### 序列化工具

位于`plugin_kits/include/utils/`：
- `string_serializer` - 字符串序列化
- `array_string_serializer` - 字符串数组序列化
- `bool_serializer` - 布尔序列化
- `int_serializer` / `uint_serializer` / `long_serializer` - 数字序列化
- `map_string_serializer` - Map序列化
- `cjson_serializer` - JSON序列化
- `func_code_utils` - 功能码工具

证据：`interfaces/inner_api/plugin_kits/include/utils/*.h`

---

## services/edm_plugin插件分类

### 插件库分组

| 库名 | 插件类型 | 插件数量 |
|------|----------|----------|
| device_core_plugin | 设备核心 | ~60 |
| communication_plugin | 通信管理 | ~30 |
| sys_service_plugin | 系统服务 | ~30 |
| need_extra_plugin | 额外功能 | ~6 |

证据：`services/edm_plugin/BUILD.gn`

### 典型插件实现

| 插件 | 文件 | 功能 |
|------|------|------|
| DisableCameraPlugin | `disable_camera_plugin.cpp` | 禁用相机 |
| PasswordPolicyPlugin | `password_policy_plugin.cpp` | 密码策略 |
| InstallUserCertificatePlugin | `install_user_certificate_plugin.cpp` | 安装证书 |
| SetWifiDisabledPlugin | `set_wifi_disabled_plugin.cpp` | WiFi开关 |
| DisableBluetoothPlugin | `disable_bluetooth_plugin.cpp` | 蓝牙开关 |
| RebootPlugin | `reboot_plugin.cpp` | 设备重启 |
| LockScreenPlugin | `lock_screen_plugin.cpp` | 锁屏 |
| ResetFactoryPlugin | `reset_factory_plugin.cpp` | 恢复出厂 |
| AllowedInstallBundlesPlugin | `allowed_install_bundles_query.cpp` | 应用白名单 |
| DisallowedRunningBundlesPlugin | `disallowed_running_bundles_plugin.cpp` | 应用黑名单 |

证据：`services/edm_plugin/src/*_plugin.cpp`

---

## 依赖方向

### 层级依赖

```
应用层 (MDM Apps)
    ↓ 调用
NAPI层 (interfaces/kits)
    ↓ Proxy
IPC层 (interfaces/inner_api)
    ↓ 跨进程
服务层 (services/edm)
    ↓ 内部调用
插件层 (services/edm_plugin)
    ↓ 调用
外部系统 (common/external)
```

### 模块间依赖

| 调用者 | 被调用者 | 依赖类型 |
|--------|----------|----------|
| AdminManager | PolicyManager, PermissionChecker | 组合依赖 |
| PolicyManager | PluginManager, EdmRdbDataManager | 组合依赖 |
| PluginManager | 各插件 (IPlugin) | 接口依赖（dlopen动态加载） |
| EnterpriseDeviceMgrAbility | 所有Manager、PermissionChecker | 组合依赖 |
| ConnectionManager | AbilityManager | IPC依赖 |
| 各Proxy | EnterpriseDeviceMgrProxy | IPC依赖 |

---

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 详细架构设计
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - N-API接口文档
- [05_Internal_API.md](05_Internal_API.md) - 内部接口说明
