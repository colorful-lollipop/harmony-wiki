# 附录：配置标志

## 目的

本文档列出EDM组件的所有编译配置宏、Feature Flags和环境变量。

## 适用范围

- 目标读者：构建工程师、开发者
- 覆盖内容：GN构建参数、C++宏定义、系统参数

## 关键结论

- EDM使用约30个Feature Flags控制不同功能模块
- 支持ARM64和X86_64两种架构
- 根据系统组件存在动态启用相关功能

## 相关跳转

- [06_GN_Targets.md](06_GN_Targets.md) - GN目标和配置

---

## Feature Flags

### 主开关

| Flag | 默认值 | 文件位置 | 说明 |
|------|---------|----------|------|
| `enterprise_device_management_support_all` | true | `common/config/common.gni:17` | 全功能支持开关 |
| `enterprise_device_management_feature_coverage` | false | `common/config/common.gni:48` | 代码覆盖率编译 |
| `enterprise_device_management_feature_charging_type_setting` | false | `common/config/common.gni:18` | 充电类型设置特性 |
| `enterprise_device_management_feature_pc_only` | false | `common/config/common.gni:19` | PC专属特性 |

证据：`common/config/common.gni:16-21`

### 系统能力依赖开关

| Flag | 默认值 | 依赖系统 | 说明 |
|------|---------|----------|------|
| `ability_runtime_edm_enable` | false | ability_ability_runtime | Ability运行时支持 |
| `audio_framework_edm_enable` | false | multimedia_audio_framework | 音频框架 |
| `bluetooth_edm_enable` | false | communication_bluetooth | 蓝牙管理 |
| `bundle_framework_edm_enable` | false | bundlemanager_bundle_framework | Bundle框架 |
| `camera_framework_edm_enable` | false | multimedia_camera_framework | 相机框架 |
| `cellular_data_edm_enable` | false | telephony_cellular_data | 蜂窝数据 |
| `certificate_manager_edm_enable` | false | security_certificate_manager | 证书管理 |
| `common_event_service_edm_enable` | false | notification_common_event_service | 公共事件服务 |
| `drivers_interface_usb_edm_enable` | false | hdf_drivers_interface_usb | USB驱动接口 |
| `inputmethod_imf_enable` | false | inputmethod_imf | 输入法 |
| `location_edm_enable` | false | location_location | 位置服务 |
| `netmanager_base_edm_enable` | false | communication_netmanager_base | 网络管理基础 |
| `netmanager_ext_edm_enable` | false | communication_netmanager_ext | 网络管理扩展 |
| `notification_edm_enable` | false | notification_distributed_notification_service | 通知服务 |
| `os_account_edm_enable` | false | account_os_account | OS账户 |
| `pasteboard_edm_enable` | false | distributeddatamgr_pasteboard | 剪贴板 |
| `power_manager_edm_enable` | false | powermgr_power_manager | 电源管理 |
| `screenlock_mgr_edm_enable` | false | theme_screenlock_mgr | 锁屏管理 |
| `sms_mms_edm_enable` | false | telephony_sms_mms | 短信彩信 |
| `storage_service_edm_enable` | false | filemanagement_storage_service | 存储服务 |
| `telephony_core_edm_enable` | false | telephony_core_service | 电话核心服务 |
| `time_service_edm_enable` | false | time_time_service | 时间服务 |
| `update_service_edm_enable` | false | updater_update_service | 更新服务 |
| `usb_manager_edm_enable` | false | usb_usb_manager | USB管理 |
| `useriam_edm_enable` | false | useriam_user_auth_framework | 用户认证 |
| `wifi_edm_enable` | false | communication_wifi | WiFi |
| `log_service_plugin_edm_enable` | false | hmoshiviewdfx_log_service_plugin | 日志插件 |

证据：`common/config/common.gni:21-162`

---

## C++编译宏

### 架构宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `_ARM64_` | `target_cpu == "arm64"` | `services/edm/BUILD.gn:43-44` | ARM64架构 |
| `_X86_64_` | `target_cpu == "x86_64"` | `services/edm/BUILD.gn:48-50` | X86_64架构 |

证据：`services/edm/BUILD.gn:43-50`

### 功能宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `EDM_SUPPORT_ALL_ENABLE` | `enterprise_device_management_support_all` | `services/edm/BUILD.gn:81` | 全功能支持 |
| `FEATURE_CHARGING_TYPE_SETTING` | `enterprise_device_management_feature_charging_type_setting` | 充电类型设置 |
| `FEATURE_PC_ONLY` | `enterprise_device_management_feature_pc_only` | PC专属功能 |

### WiFi相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `WIFI_EDM_ENABLE` | `wifi_edm_enable` | `services/edm_plugin/BUILD.gn` | WiFi功能 |
| `BLUETOOTH_EDM_ENABLE` | `bluetooth_edm_enable` | `services/edm_plugin/BUILD.gn` | 蓝牙功能 |

### 位置相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `LOCATION_EDM_ENABLE` | `location_edm_enable` | `services/edm_plugin/BUILD.gn` | 位置服务功能 |

### 用户认证相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `USERIAM_EDM_ENABLE` | `useriam_edm_enable` | `services/edm_plugin/BUILD.gn` | 用户认证功能 |

### 电话相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `TELEPHONY_CORE_EDM_ENABLE` | `telephony_core_edm_enable` | `services/edm_plugin/BUILD.gn` | 电话核心功能 |
| `CELLULAR_DATA_EDM_ENABLE` | `cellular_data_edm_enable` | `services/edm_plugin/BUILD.gn` | 蜂窝数据 |
| `SMS_EDM_ENABLE` / `MMS_EDM_ENABLE` | `sms_mms_edm_enable` | `services/edm_plugin/BUILD.gn` | 短信彩信 |

### 系统服务相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `ABILITY_RUNTIME_EDM_ENABLE` | `ability_runtime_edm_enable` | `services/edm/BUILD.gn` | Ability运行时 |
| `STORAGE_SERVICE_EDM_ENABLE` | `storage_service_edm_enable` | `services/edm_plugin/BUILD.gn` | 存储服务 |
| `EXTERNAL_STORAGE_SERVICE_EDM_ENABLE` | `storage_service_edm_enable` | `services/edm_plugin/BUILD.gn` | 外部存储 |
| `NOTIFICATION_EDM_ENABLE` | `notification_edm_enable` | `services/edm_plugin/BUILD.gn` | 通知服务 |
| `BACKUP_AND_RESTORE_EDM_ENABLE` | `sms_mms_edm_enable` | `services/edm_plugin/BUILD.gn` | 备份恢复 |
| `PRIVATE_SPACE_EDM_ENABLE` | `sms_mms_edm_enable` | `services/edm_plugin/BUILD.gn` | 隐私空间 |
| `APN_EDM_ENABLE` | `cellular_data_edm_enable` | `services/edm_plugin/BUILD.gn` | APN设置 |
| `MOBILE_DATA_ENABLE` | `cellular_data_edm_enable` | `services/edm_plugin/BUILD.gn` | 移动数据 |
| `TELEPHONY_EDM_ENABLE` | `telephony_core_edm_enable` | `services/edm_plugin/BUILD.gn` | 电话功能 |
| `POWER_MANAGER_EDM_ENABLE` | `power_manager_edm_enable` | `services/edm_plugin/BUILD.gn` | 电源管理 |

### 安全相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `SECURITY_GUARDE_ENABLE` | `security_guard`依赖 | `services/edm_plugin/BUILD.gn` | 安全守护 |

### USB相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `USB_SERVICE_EDM_ENABLE` | `usb_manager_edm_enable` | `services/edm_plugin/BUILD.gn` | USB服务 |
| `USB_STORAGE_SERVICE_EDM_ENABLE` | `storage_service_edm_enable` | `services/edm_plugin/BUILD.gn` | USB存储 |

### 网络相关宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `NET_MANAGER_BASE_EDM_ENABLE` | `netmanager_base_edm_enable` | `services/edm_plugin/BUILD.gn` | 网络管理基础 |
| `NETMANAGER_EXT_EDM_ENABLE` | `netmanager_ext_edm_enable` | `services/edm_plugin/BUILD.gn` | 网络管理扩展 |
| `COMMON_EVENT_SERVICE_EDM_ENABLE` | `common_event_service_edm_enable` | `services/edm_plugin/BUILD.gn` | 公共事件 |

### 其他宏

| 宏 | 触发条件 | 文件 | 说明 |
|-----|---------|--------|------|
| `SAMBA_EDM_ENABLE` | `FEATURE_PC_ONLY && !SMS_EDM_ENABLE` | `services/edm_plugin/BUILD.gn` | Samba功能（PC版） |

---

## 系统参数

### EDM参数配置

**文件**: `etc/param/edm.para`

| 参数 | 默认值 | 说明 |
|------|---------|------|
| `persist.edm.edm_enable` | false | EDM功能总开关 |
| `persist.edm.location_policy` | none | 位置策略（disabled/allowed/disallowed） |
| `persist.edm.mic_disable` | false | 麦克风禁用 |
| `persist.edm.is_local_install_enable` | false | 本地安装启用 |
| `persist.edm.disallow_virtual_service` | none | 禁止虚拟服务 |

证据：`etc/param/edm.para:14-18`

### EDM服务配置

**文件**: `etc/init/edm.cfg`

| 配置项 | 说明 |
|---------|------|
| `service` | EDM服务启动配置 |
| `ondemand` | 按需启动机制 |
| 权限 | EDM服务所需系统权限 |

---

## SA配置

**文件**: `sa_profile/1601.json`

| 配置项 | 值 | 说明 |
|---------|-----|------|
| `name` | 1601 | SA ID |
| `libpath` | libedmservice.z.so | 服务库文件 |
| `process` | edm | 进程名 |
| `run-on-create` | false | 非即时启动 |
| `start-on-demand` | bindercall + param | 按需启动 |
| `distributed` | false | 非分布式服务 |

**启动条件**：
- bindercall: true（binder call时启动）
- persist.edm.edm_enable: true（参数为true时启动）

证据：`sa_profile/1601.json:4-19`

---

## 使用示例

### 启用全功能

```bash
# 编译时启用所有EDM功能
gn gen --args="enterprise_device_management_support_all=true"
```

### 启用特定功能

```bash
# 仅启用WiFi功能
gn gen --args="wifi_edm_enable=true"

# 启用多个功能
gn gen --args="wifi_edm_enable=true bluetooth_edm_enable=true location_edm_enable=true"
```

### 禁用功能

```bash
# 禁用WiFi功能
gn gen --args="wifi_edm_enable=false"
```

### PC专用编译

```bash
# PC平台编译
gn gen --args="enterprise_device_management_feature_pc_only=true target_cpu=x86_64"
```

---

## 相关跳转

- [06_GN_Targets.md](06_GN_Targets.md) - GN目标和构建配置
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物和安装
