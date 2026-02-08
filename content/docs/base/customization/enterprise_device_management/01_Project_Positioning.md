# 项目定位与边界

## 目的

本文档描述EDM组件的项目定位、能力边界、核心功能和运行环境。

## 适用范围

- 目标读者：架构师、产品经理、开发者
- 覆盖内容：组件定位、能力范围、运行边界、关键概念
- 不包含：详细实现、API参考

## 关键结论

- EDM是企业设备管理的系统级组件，不属于普通应用层
- 提供受限但强大的设备管控能力，需要特殊权限
- 通过插件机制支持126+个策略，但核心框架固定
- 与系统服务深度集成，不直接面向终端用户

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Directory_Structure.md](02_Directory_Structure.md) - 代码组织
- [08_Security_Review.md](08_Security_Review.md) - 安全注意事项

---

## 项目定位

### 系统级组件

EDM是OpenHarmony的子系统`customization`下的组件`enterprise_device_management`，属于系统基础能力。

**证据**：
- `bundle.json:39-40` - `subsystem: "customization"`
- `bundle.json:41` - `syscap: ["SystemCapability.Customization.EnterpriseDeviceManager"]`

### 能力边界

EDM提供的能力范围：

| 类别 | 包含能力 | 不包含能力 |
|------|----------|----------|
| **设备控制** | 重启、关机、恢复出厂、锁屏 | 用户自定义硬件控制 |
| **应用管理** | 安装/卸载限制、应用黑名单、Kiosk | 应用内部逻辑、UI交互 |
| **网络管理** | WiFi、蓝牙、VPN策略、防火墙 | 应用网络权限管理 |
| **安全管理** | 密码策略、证书管理、设备加密 | 应用沙箱权限 |
| **系统设置** | 时间、壁纸、输入法、浏览器策略 | 用户偏好设置 |
| **信息查询** | 设备信息、应用列表、网络状态 | - |

### 信任边界

```
┌─────────────────────────────────────────────────────┐
│           不可信环境                              │
│  (普通三方应用、用户交互)                       │
└────────────────────┬────────────────────────────┘
                     │ 受限访问
                     ▼
┌─────────────────────────────────────────────────────┐
│           EDM管控环境                                │
│  (EDM管理应用、策略系统)                       │
│                                                     │
│  ┌────────────┐  ┌──────────────┐            │
│  │MDM应用     │  │EDM服务       │            │
│  │(特权应用)   │  │(SA 1601)     │            │
│  └──────┬─────┘  └──────┬───────┘            │
│         │                   │                          │
│         ▼                   ▼                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │       系统服务（受保护）                     │   │
│  │  (BundleManager、WiFiManager、...)              │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### 核心能力

| 能力 | 实现位置 | 限制条件 |
|------|----------|----------|
| 设备管理员激活 | `services/edm/src/enterprise_device_mgr_ability.cpp:1585` | 需要EDM权限、校验激活条件 |
| 策略设置 | `services/edm/src/plugin_manager.cpp` | 需要管理员权限、校验策略类型 |
| 策略查询 | `services/edm/src/query_policy/` | 需要管理员权限 |
| 管理事件订阅 | `interfaces/kits/admin_manager/src/admin_manager_addon.cpp` | 需要ENTERPRISE_SUBSCRIBE_MANAGED_EVENT权限 |

---

## 核心能力

### 1. 设备管理

| 功能 | 接口 | 插件 | 说明 |
|------|--------|------|------|
| 设备信息查询 | `@ohos.enterprise.deviceInfo` | `get_device_info_plugin` | 序列号、版本、型号等 |
| 设备重启 | `@ohos.enterprise.deviceControl` | `reboot_plugin` | 重启设备 |
| 设备关机 | `@ohos.enterprise.deviceControl` | `shutdown_plugin` | 关机设备 |
| 恢复出厂 | `@ohos.enterprise.deviceControl` | `reset_factory_plugin` | 恢复出厂设置 |
| 锁屏 | `@ohos.enterprise.systemManager` | `lock_screen_plugin` | 锁定设备屏幕 |

### 2. 应用管理

| 功能 | 接口 | 插件 | 说明 |
|------|--------|------|------|
| 应用安装 | `@ohos.enterprise.applicationManager` | `install_plugin` | 安装应用 |
| 应用卸载 | `@ohos.enterprise.applicationManager` | `uninstall_plugin` | 卸载应用 |
| 应用黑名单 | `@ohos.enterprise.restrictions` | `disallowed_running_bundles_plugin` | 禁止指定应用 |
| 应用白名单 | `@ohos.enterprise.restrictions` | `allowed_running_bundles_plugin` | 只允许指定应用 |
| Kiosk模式 | `@ohos.enterprise.applicationManager` | `kiosk_feature_plugin` | 单应用模式 |
| 应用冻结 | `@ohos.enterprise.applicationManager` | `manage_freeze_exempted_apps_plugin` | 冻结/解冻应用 |

### 3. 网络管理

| 功能 | 接口 | 插件 | 说明 |
|------|--------|------|------|
| WiFi管理 | `@ohos.enterprise.wifiManager` | `set_wifi_disabled_plugin`、`disallowed_wifi_list_plugin` | WiFi开关、SSID列表 |
| 蓝牙管理 | `@ohos.enterprise.bluetoothManager` | `disable_bluetooth_plugin`、`allowed_bluetooth_devices_plugin` | 蓝牙开关、设备列表 |
| 移动数据 | `@ohos.enterprise.telephonyManager` | `turnonoff_mobile_data_plugin` | 移动数据开关 |
| VPN策略 | `@ohos.enterprise.networkManager` | `disallow_vpn_plugin` | VPN禁用 |
| 防火墙 | `@ohos.enterprise.networkManager` | `firewall_rule_plugin` | 防火墙规则 |
| 全局代理 | `@ohos.enterprise.networkManager` | `global_proxy_plugin` | HTTP代理配置 |

### 4. 安全管理

| 功能 | 接口 | 插件 | 说明 |
|------|--------|------|------|
| 密码策略 | `@ohos.enterprise.securityManager` | `password_policy_plugin` | 密码复杂度要求 |
| 证书管理 | `@ohos.enterprise.deviceSettings` | `install_user_certificate_plugin` | 用户证书安装/卸载 |
| 指纹策略 | `@ohos.enterprise.restrictions` | `fingerprint_auth_plugin` | 指纹认证策略 |
| 相机禁用 | `@ohos.enterprise.restrictions` | `disable_camera_plugin` | 禁用相机 |
| USB控制 | `@ohos.enterprise.usbManager` | `disable_usb_plugin`、`usb_read_only_plugin` | USB开关、只读模式 |
| 水印 | `@ohos.enterprise.securityManager` | `set_watermark_image_plugin` | 屏幕水印 |

### 5. 系统设置

| 功能 | 接口 | 插件 | 说明 |
|------|--------|------|------|
| 日期时间 | `@ohos.enterprise.dateTimeManager` | `set_datetime_plugin` | 系统时间设置和禁用 |
| 壁纸设置 | `@ohos.enterprise.deviceSettings` | `set_wall_paper_plugin` | 锁屏/主屏壁纸 |
| 输入法 | `@ohos.enterprise.deviceSettings` | `set_default_input_method_plugin` | 默认输入法 |
| 浏览器策略 | `@ohos.enterprise.browser` | `set_browser_policies_plugin` | 浏览器主页、书签等 |
| 屏幕超时 | `@ohos.enterprise.deviceSettings` | `set_screen_off_time_plugin` | 自动锁屏时间 |

---

## 运行环境

### 系统要求

**证据**：`bundle.json:48-50`

| 项目 | 要求 |
|------|------|
| 适配系统 | OpenHarmony标准系统（Standard） |
| 最小版本 | API 9+ |
| 架构 | ARM64、X86_64 |
| 运行内存 | ~6MB（RAM占用） |
| 存储空间 | ~2MB（ROM占用） |

### 系统能力依赖

根据`common/config/common.gni`的26个组件开关：

| 系统能力 | EDM功能 | 开关宏 |
|----------|---------|--------|
| AbilityRuntime | Ability控制 | `ABILITY_RUNTIME_EDM_ENABLE` |
| BundleFramework | 应用管理 | `BUNDLE_FRAMEWORK_EDM_ENABLE` |
| IPC | 通信基础 | (始终启用) |
| Safwk | 系统服务框架 | (始终启用) |
| WiFi | WiFi管理 | `WIFI_EDM_ENABLE` |
| Bluetooth | 蓝牙管理 | `BLUETOOTH_EDM_ENABLE` |
| Location | 位置服务 | `LOCATION_EDM_ENABLE` |
| Storage | 存储服务 | `STORAGE_SERVICE_EDM_ENABLE` |
| UserIAM | 用户认证 | `USERIAM_EDM_ENABLE` |
| PowerManager | 电源管理 | `POWER_MANAGER_EDM_ENABLE` |
| ... | ... | ... |

---

## 关键概念

### 1. 设备管理员 (Device Administrator)

拥有特殊权限的应用，被企业管理后可以：
- 设置设备策略
- 查询设备信息
- 订阅系统事件
- 启用其他管理员

**权限级别**：
- Super Admin - 最高权限，可激活普通管理员
- Normal Admin - 中等权限，由Super Admin授权
- BYOD Admin - BYOD场景下的管理员

### 2. 策略 (Policy)

企业对设备的约束规则，包括：
- 布尔策略（禁用/启用功能）
- 枚举策略（设置特定值）
- 列表策略（允许/禁止列表）
- 对象策略（复杂配置对象）

**策略存储**：
- 持久化到RDB数据库
- 按管理员和用户ID索引
- 支持多管理员策略合并

### 3. 插件 (Plugin)

策略的具体实现模块，特点：
- 动态加载（dlopen）
- 实现`IPlugin`接口
- 一个funcCode对应一个或多个插件
- 支持策略设置和查询

**插件类型**：
- BASIC插件 - 基础功能
- EXTENSION插件 - 可扩展基础插件

### 4. 管理事件 (Managed Event)

系统事件通知机制：
- 应用级事件（安装、卸载、启动、停止）
- 系统级事件（更新、开机）
- 账号级事件（添加、移除、切换）

### 5. 委托策略 (Delegated Policy)

超级管理员可以将部分策略管理权限授权给普通管理员：
- 减轻超级管理员负担
- 支持分权管理
- 可设置可委托的策略列表

---

## 非目标

### 不直接提供的能力

- 用户界面（UI）- MDM应用自己提供
- 网络协议实现 - 依赖系统能力
- 硬件驱动 - 直接使用系统驱动
- 加密/解密服务 - 使用系统安全框架

### 限制

- 无法管理用户设备的应用（用户安装的普通应用）
- 无法访问用户数据（需用户授权）
- 策略不跨设备同步（需MDM服务端）

---

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 详细架构说明
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - 完整API清单
- [08_Security_Review.md](08_Security_Review.md) - 安全边界和风险
