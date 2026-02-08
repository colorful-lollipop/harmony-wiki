# 对外API - N-API

## 目的

本文档提供EDM组件所有N-API接口的完整清单，包括JS API名称、参数、返回值、权限和错误码。

## 适用范围

- 目标读者：MDM应用开发者
- 覆盖内容：18个N-API模块、约200+个JS方法
- 不包含：详细实现细节

## 关键结论

- EDM提供18个N-API模块，对应17个Manager
- 所有策略操作需要管理员权限
- 大部分API使用异步回调模式

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览
- [05_Internal_API.md](05_Internal_API.md) - 内部接口

---

## N-API模块清单

| 序号 | 模块名 | JS命名空间 | 文件 | 主要功能 |
|------|---------|------------|------|----------|
| 1 | adminManager | @ohos.enterprise.adminManager | admin_manager_addon.cpp | 管理员管理 |
| 2 | accountManager | @ohos.enterprise.accountManager | account_manager_addon.cpp | 账户管理 |
| 3 | applicationManager | @ohos.enterprise.applicationManager | application_manager_addon.cpp | 应用管理 |
| 4 | bluetoothManager | @ohos.enterprise.bluetoothManager | bluetooth_manager_addon.cpp | 蓝牙管理 |
| 5 | browser | @ohos.enterprise.browser | browser_addon.cpp | 浏览器策略 |
| 6 | bundleManager | @ohos.enterprise.bundleManager | bundle_manager_addon.cpp | 应用包管理 |
| 7 | commonManager | @ohos.enterprise.commonManager | common_manager_addon.cpp | 通用接口 |
| 8 | dateTimeManager | @ohos.enterprise.dateTimeManager | datetime_manager_addon.cpp | 日期时间管理 |
| 9 | deviceControl | @ohos.enterprise.deviceControl | device_control_addon.cpp | 设备控制 |
| 10 | deviceInfo | @ohos.enterprise.deviceInfo | device_info_addon.cpp | 设备信息 |
| 11 | deviceSettings | @ohos.enterprise.deviceSettings | device_settings_addon.cpp | 设备设置 |
| 12 | locationManager | @ohos.enterprise.locationManager | location_manager_addon.cpp | 位置管理 |
| 13 | networkManager | @ohos.enterprise.networkManager | network_manager_addon.cpp | 网络管理 |
| 14 | restrictions | @ohos.enterprise.restrictions | restrictions_addon.cpp | 限制策略 |
| 15 | securityManager | @ohos.enterprise.securityManager | security_manager_addon.cpp | 安全管理 |
| 16 | systemManager | @ohos.enterprise.systemManager | system_manager_addon.cpp | 系统管理 |
| 17 | telephonyManager | @ohos.enterprise.telephonyManager | telephony_manager_addon.cpp | 电话管理 |
| 18 | usbManager | @ohos.enterprise.usbManager | usb_manager_addon.cpp | USB管理 |
| 19 | wifiManager | @ohos.enterprise.wifiManager | wifi_manager_addon.cpp | WiFi管理 |

证据：`interfaces/kits/*/src/*_addon.cpp`中的`napi_module`定义

---

## adminManager模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| enableAdmin | AdminManager::EnableAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 启用管理员 |
| disableAdmin | AdminManager::DisableAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 禁用管理员 |
| enableDeviceAdmin | AdminManager::EnableDeviceAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 启用设备管理员 |
| disableDeviceAdmin | AdminManager::DisableDeviceAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 禁用设备管理员 |
| disableSuperAdmin | AdminManager::DisableSuperAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 禁用超级管理员 |
| getEnabledAdmin | AdminManager::GetEnabledAdmin | - | ✓ | 获取已启用管理员 |
| getEnterpriseInfo | AdminManager::GetEnterpriseInfo | GET_ENTERPRISE_INFO | ✓ | 获取企业信息 |
| setEnterpriseInfo | AdminManager::SetEnterpriseInfo | SET_ENTERPRISE_INFO | ✓ | 设置企业信息 |
| isSuperAdmin | AdminManager::IsSuperAdmin | - | ✓ | 检查是否超级管理员 |
| isAdminEnabled | AdminManager::IsAdminEnabled | - | ✓ | 检查管理员是否启用 |
| subscribeManagedEvent | AdminManager::SubscribeManagedEvent | ENTERPRISE_SUBSCRIBE_MANAGED_EVENT | ✓ | 订阅管理事件 |
| unsubscribeManagedEvent | AdminManager::UnsubscribeManagedEvent | ENTERPRISE_SUBSCRIBE_MANAGED_EVENT | ✓ | 取消订阅 |
| authorizeAdmin | AdminManager::AuthorizeAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 授权管理员 |
| getSuperAdmin | AdminManager::GetSuperAdmin | - | ✓ | 获取超级管理员 |
| setDelegatedPolicies | AdminManager::SetDelegatedPolicies | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 设置委托策略 |
| getDelegatedPolicies | AdminManager::GetDelegatedPolicies | - | ✓ | 获取委托策略 |
| getDelegatedBundleNames | AdminManager::GetDelegatedBundleNames | - | ✓ | 获取委托Bundle名称 |
| replaceSuperAdmin | AdminManager::ReplaceSuperAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 替换超级管理员 |
| getAdmins | AdminManager::GetAdmins | - | ✓ | 获取所有管理员 |
| setAdminRunningMode | AdminManager::SetAdminRunningMode | MANAGE_ENTERPRISE_DEVICE_ADMIN | ✓ | 设置管理员运行模式 |
| isByodAdmin | AdminManager::IsByodAdmin | - | ✓ | 检查是否BYOD管理员 |

证据：`interfaces/kits/admin_manager/src/admin_manager_addon.cpp:42-1370`

### 管理员事件常量

```javascript
// 在commonManager模块中定义
MANAGED_EVENT_BUNDLE_ADDED       // 应用添加
MANAGED_EVENT_BUNDLE_REMOVED     // 应用移除
MANAGED_EVENT_APP_START          // 应用启动
MANAGED_EVENT_APP_STOP           // 应用停止
MANAGED_EVENT_SYSTEM_UPDATE       // 系统更新
MANAGED_EVENT_ACCOUNT_ADDED       // 账号添加
MANAGED_EVENT_ACCOUNT_SWITCHED    // 账号切换
MANAGED_EVENT_ACCOUNT_REMOVED      // 账号移除
MANAGED_EVENT_STARTUP_GUIDE_COMPLETED  // 启动向导完成
MANAGED_EVENT_BOOT_COMPLETED      // 开机完成
```

证据：`interfaces/kits/admin_manager/src/admin_manager_addon.cpp`的事件常量导出

---

## deviceSettings模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| setScreenOffTime | DeviceSettingsAddon::SetScreenOffTime | ENTERPRISE_SET_SCREENOFF_TIME | ✓ | 设置屏幕超时 |
| getScreenOffTime | DeviceSettingsAddon::GetScreenOffTime | ENTERPRISE_GET_SETTINGS | ✓ | 获取屏幕超时 |
| setPowerPolicy | DeviceSettingsAddon::SetPowerPolicy | MANAGE_ENTERPRISE_SETTINGS | ✓ | 设置电源策略 |
| getPowerPolicy | DeviceSettingsAddon::GetPowerPolicy | ENTERPRISE_GET_SETTINGS | ✓ | 获取电源策略 |
| installUserCertificate | DeviceSettingsAddon::InstallUserCertificate | MANAGE_CERTIFICATE | ✓ | 安装用户证书 |
| uninstallUserCertificate | DeviceSettingsAddon::UninstallUserCertificate | MANAGE_CERTIFICATE | ✓ | 卸载用户证书 |
| setValue | DeviceSettingsAddon::SetValue | MANAGE_ENTERPRISE_SETTINGS | ✓ | 设置配置值 |
| getValue | DeviceSettingsAddon::GetValue | ENTERPRISE_GET_SETTINGS | ✓ | 获取配置值 |
| setValueForAccount | DeviceSettingsAddon::SetValueForAccount | MANAGE_ENTERPRISE_SETTINGS | ✓ | 为指定用户设置 |
| getValueForAccount | DeviceSettingsAddon::GetValueForAccount | ENTERPRISE_GET_SETTINGS | ✓ | 为指定用户获取 |
| setHomeWallpaper | DeviceSettingsAddon::SetHomeWallPaper | MANAGE_ENTERPRISE_SETTINGS | ✓ | 设置主屏壁纸 |
| setUnlockWallpaper | DeviceSettingsAddon::SetUnlockWallPaper | MANAGE_ENTERPRISE_SETTINGS | ✓ | 设置锁屏壁纸 |

### 常量导出

```javascript
// PowerScene枚举
PowerScene.TIME_OUT           // 超时关闭

// PowerPolicyAction枚举  
PowerPolicyAction.NONE          // 无操作
PowerPolicyAction.AUTO_SUSPEND  // 自动休眠
PowerPolicyAction.FORCE_SUSPEND // 强制休眠
PowerPolicyAction.HIBERNATE     // 休眠
PowerPolicyAction.SHUTDOWN       // 关机

// SettingsItem枚举
SettingsItem.DEVICE_NAME     // 设备名称
// ... 更多配置项
```

证据：`interfaces/kits/device_settings/src/device_settings_addon.cpp:26-62`

---

## deviceControl模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| operateDevice | DeviceControlAddon::OperateDevice | ENTERPRISE_OPERATE_DEVICE | ✓ | 操作设备（重启/关机） |

证据：`interfaces/kits/device_control/src/device_control_addon.cpp:45-86`

### OperateDevice操作类型

```javascript
// OperateDevice支持的操作
OperateType.SHUTDOWN      // 关机
OperateType.REBOOT        // 重启
OperateType.LOCK_SCREEN   // 锁屏
OperateType.SET_TIME      // 设置时间
```

---

## bundleManager模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| install | BundleManagerAddon::Install | ENTERPRISE_INSTALL_BUNDLE | ✓ | 安装应用 |
| uninstall | BundleManagerAddon::Uninstall | ENTERPRISE_INSTALL_BUNDLE | ✓ | 卸载应用 |

证据：`interfaces/kits/bundle_manager/src/bundle_manager_addon.cpp:87-131`

---

## applicationManager模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| getManagedInstalledBundles | ApplicationManagerAddon::GetManagedInstalledBundles | ENTERPRISE_GET_SETTINGS | ✓ | 获取已安装应用 |
| setKeepAliveApps | ApplicationManagerAddon::SetKeepAliveApps | MANAGE_ENTERPRISE_SETTINGS | ✓ | 设置常驻应用 |
| manageUserNonStopApps | ApplicationManagerAddon::ManageUserNonStopApps | MANAGE_ENTERPRISE_SETTINGS | ✓ | 管理用户不可停止应用 |
| clearUpApplicationData | ApplicationManagerAddon::ClearUpApplicationData | MANAGE_ENTERPRISE_SETTINGS | ✓ | 清除应用数据 |
| setKioskFeature | ApplicationManagerAddon::SetKioskFeature | MANAGE_ENTERPRISE_SETTINGS | ✓ | 设置Kiosk特性 |
| isAppKioskAllowed | ApplicationManagerAddon::IsAppKioskAllowed | ENTERPRISE_GET_SETTINGS | ✓ | 检查应用是否允许Kiosk |

证据：`interfaces/kits/application_manager/src/application_manager_addon.cpp:73-125`

---

## wifiManager模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| isWifiActive | WifiManagerAddon::IsWifiActive | ENTERPRISE_GET_NETWORK_INFO | ✓ | 检查WiFi是否激活 |
| getWifiSignalLevel | WifiManagerAddon::GetWifiSignalLevel | ENTERPRISE_GET_NETWORK_INFO | ✓ | 获取WiFi信号强度 |
| setWifiProfile | WifiManagerAddon::SetWifiProfile | ENTERPRISE_MANAGE_WIFI | ✓ | 设置WiFi配置 |
| getWifiProfile | WifiManagerAddon::GetWifiProfile | ENTERPRISE_GET_NETWORK_INFO | ✓ | 获取WiFi配置 |
| isWifiDisabled | WifiManagerAddon::IsWifiDisabled | ENTERPRISE_MANAGE_WIFI | ✓ | 检查WiFi是否禁用 |

证据：`interfaces/kits/wifi_manager/src/wifi_manager_addon.cpp:52-357`

---

## bluetoothManager模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| isBluetoothActive | BluetoothManagerAddon::IsBluetoothActive | ENTERPRISE_GET_NETWORK_INFO | ✓ | 检查蓝牙是否激活 |
| getBluetoothState | BluetoothManagerAddon::GetBluetoothState | ENTERPRISE_GET_NETWORK_INFO | ✓ | 获取蓝牙状态 |

证据：`interfaces/kits/bluetooth_manager/src/bluetooth_manager_addon.cpp:52-357`

---

## networkManager模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| getNetworkInterfaces | NetworkManagerAddon::GetNetworkInterfaces | ENTERPRISE_GET_NETWORK_INFO | ✓ | 获取网络接口 |
| getIpAddress | NetworkManagerAddon::GetIpAddress | ENTERPRISE_GET_NETWORK_INFO | ✓ | 获取IP地址 |
| getMac | NetworkManagerAddon::GetMac | ENTERPRISE_GET_NETWORK_INFO | ✓ | 获取MAC地址 |
| setGlobalProxy | NetworkManagerAddon::SetGlobalProxy | MANAGE_ENTERPRISE_MANAGE_NETWORK | ✓ | 设置全局代理 |
| getGlobalProxy | NetworkManagerAddon::GetGlobalProxy | ENTERPRISE_GET_NETWORK_INFO | ✓ | 获取全局代理 |

证据：`interfaces/kits/network_manager/src/network_manager_addon.cpp:52-1951`

---

## restrictions模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| isRunningAllowed | RestrictionsAddon::IsRunningAllowed | ENTERPRISE_RESTRICT_POLICY | ✓ | 检查应用是否允许运行 |
| isInstallationAllowed | RestrictionsAddon::IsInstallationAllowed | ENTERPRISE_RESTRICT_POLICY | ✓ | 检查应用是否允许安装 |

证据：`interfaces/kits/restrictions/src/restrictions_addon.cpp:144-897`

---

## securityManager模块

### 功能清单

| JS方法 | C++入口 | 权限 | 异步 | 说明 |
|---------|----------|--------|------|------|
| setPasswordPolicy | SecurityManagerAddon::SetPasswordPolicy | MANAGE_ENTERPRISE_MANAGE_SECURITY | ✓ | 设置密码策略 |
| getPasswordPolicy | SecurityManagerAddon::GetPasswordPolicy | ENTERPRISE_MANAGE_SECURITY | ✓ | 获取密码策略 |
| setWatermarkImage | SecurityManagerAddon::SetWatermarkImage | MANAGE_ENTERPRISE_MANAGE_SECURITY | ✓ | 设置水印图片 |

证据：`interfaces/kits/security_manager/src/security_manager_addon.cpp:65-988`

---

## 调用链示例：启用管理员

### JS调用链

```javascript
// MDM应用中
import adminManager from '@ohos.enterprise.adminManager'

async function enableAdmin() {
  const want = {
    bundleName: 'com.example.admin',
    abilityName: 'AdminAbility'
  }
  const entInfo = {
    name: 'Example Corp',
    description: 'Enterprise Device Management'
  }
  const adminType = adminManager.AdminType.NORMAL // 0: 普通，1: 超级，2: BYOD
  
  try {
    const result = await adminManager.enableAdmin(
      want,
      entInfo,
      adminType,
      0, // userId
      (err, data) => { // callback
        if (err.code === 0) {
          console.log('Admin enabled successfully')
        } else {
          console.error('Failed to enable admin:', err.code)
        }
      }
    )
  } catch (error) {
    console.error('Exception:', error)
  }
}
```

### Native执行流程

```
JS层: enableAdmin()
     ↓ 参数解析和校验
NAPI层: NativeEnableAdmin()
     ↓ 创建AsyncCallbackInfo
工作线程: 执行异步操作
     ↓ 加载EDM服务
Proxy层: EnterpriseDeviceMgrProxy::EnableAdmin()
     ↓ IPC调用
IPC层: SendRequest(ADD_DEVICE_ADMIN)
     ↓ 跨进程通信
服务层: EnterpriseDeviceMgrAbility::HandleDevicePolicy()
     ↓ 权限检查
权限检查: PermissionChecker::CheckCallerPermission()
     ↓ 验证EDM权限
服务层: AdminManager::EnableAdmin()
     ↓ 验证管理员类型
策略管理: PolicyManager::SetAdminValue()
     ↓ 保存到RDB
连接管理: ConnectionManager::ConnectAbility()
     ↓ 连接管理员Extension
     ↓ 返回成功
NAPI层: NativeVoidCallbackComplete()
     ↓ 调用JS回调
JS层: callback(null, result)
```

证据链：
1. JS调用：`interfaces/kits/admin_manager/src/admin_manager_addon.cpp:42`
2. NAPI绑定：`interfaces/kits/admin_manager/src/admin_manager_addon.cpp:84` (HandleAsyncWork)
3. Proxy调用：`interfaces/inner_api/common/src/enterprise_device_mgr_proxy.cpp` (EnableAdmin)
4. 服务端处理：`services/edm/src/enterprise_device_mgr_ability.cpp:2060` (HandleDevicePolicy)
5. 管理员实现：`services/edm/src/admin_manager.cpp` (EnableAdmin)

---

## 错误码

### 错误码定义（edm_errors.h）

| 错误码 | 值 | 说明 |
|---------|------|------|
| ERR_OK | 0 | 成功 |
| ERR_EDM_PERMISSION_ERROR | 201 | EDM权限错误 |
| ERR_EDM_SYS_CODE_DENIED | 202 | 系统API调用被拒绝 |
| ERR_EDM_INVALID_ADMIN | 203 | 无效的管理员 |
| ERR_EDM_PERMISSION_DENIED | 204 | 权限被拒绝 |
| ERR_EDM_ADMIN_INACTIVE | 205 | 管理员未激活 |
| ERR_EDM_ENABLE_ADMIN_FAILED | 206 | 激活管理员失败 |
| ERR_EDM_ADMIN_EDM_PERMISSION_DENIED | 207 | 管理员EDM权限被拒绝 |
| ERR_PARAM_ERROR | 4001 | 参数错误 |
| ERR_NULL_VALUE | 4002 | 值为空 |
| ERR_INVALID_VALUE | 4003 | 值无效 |
| ERR_COMPONENT_INVALID | 4011 | 组件无效 |

证据：`common/native/include/edm_errors.h`

---

## 权限定义

### API 9权限

| 权限名 | 说明 | 使用模块 |
|---------|------|----------|
| ohos.permission.ENTERPRISE_SET_DATETIME | 设置系统时间 | adminManager |
| ohos.permission.ENTERPRISE_SUBSCRIBE_MANAGED_EVENT | 订阅管理事件 | adminManager |
| ohos.permission.SET_ENTERPRISE_INFO | 设置企业信息 | adminManager |
| ohos.permission.MANAGE_ENTERPRISE_DEVICE_ADMIN | 管理设备管理员 | adminManager |

### API 10权限

| 权限名 | 说明 | 使用模块 |
|---------|------|----------|
| ohos.permission.ENTERPRISE_SET_SCREENOFF_TIME | 设置屏幕超时 | deviceSettings |
| ohos.permission.ENTERPRISE_RESTRICT_POLICY | 限制策略 | restrictions |
| ohos.permission.ENTERPRISE_GET_SETTINGS | 获取设置 | deviceSettings |
| ohos.permission.ENTERPRISE_RESET_DEVICE | 恢复出厂 | deviceControl |
| ohos.permission.ENTERPRISE_GET_DEVICE_INFO | 获取设备信息 | deviceInfo |
| ohos.permission.ENTERPRISE_GET_NETWORK_INFO | 获取网络信息 | networkManager |
| ohos.permission.ENTERPRISE_SET_NETWORK | 设置网络 | networkManager |
| ohos.permission.ENTERPRISE_MANAGE_NETWORK | 管理网络 | networkManager |
| ohos.permission.ENTERPRISE_INSTALL_BUNDLE | 安装应用 | bundleManager |
| ohos.permission.ENTERPRISE_MANAGE_CERTIFICATE | 管理证书 | deviceSettings |

### API 11权限

| 权限名 | 说明 | 使用模块 |
|---------|------|----------|
| ohos.permission.ENTERPRISE_LOCK_DEVICE | 锁定设备 | systemManager |
| ohos.permission.ENTERPRISE_REBOOT | 重启设备 | systemManager |
| ohos.permission.ENTERPRISE_MANAGE_SETTINGS | 管理设置 | deviceSettings |
| ohos.permission.ENTERPRISE_MANAGE_SECURITY | 管理安全 | securityManager |
| ohos.permission.ENTERPRISE_MANAGE_BLUETOOTH | 管理蓝牙 | bluetoothManager |
| ohos.permission.ENTERPRISE_MANAGE_LOCATION | 管理位置 | locationManager |
| ohos.permission.ENTERPRISE_MANAGE_WIFI | 管理WiFi | wifiManager |

证据：`common/native/include/edm_constants.h` + 各模块对应的权限使用

---

## 参数校验

### 通用校验规则

| 参数类型 | 校验项 | 证据 |
|---------|----------|--------|
| ElementName | bundleName、abilityName非空 | admin_manager_addon.cpp:57-66 |
| EnterpriseInfo | name、description长度限制 | admin_manager_addon.cpp:62-64 |
| AdminType | 必须为0/1/2 | admin_manager_addon.cpp:65-66 |
| UserId | 有效范围检查 | admin_manager_addon.cpp:71-78 |
| Callback | 函数类型校验 | 公共工具类 |
| PolicyValue | JSON格式、值范围检查 | 各插件实现 |

---

## 异步回调模式

### 回调类型

```javascript
// 大部分API支持Promise和Callback两种方式

// 1. Promise方式
adminManager.enableAdmin(want, entInfo, type, userId)
  .then((data) => {
    console.log('Success:', data)
  })
  .catch((error) => {
    console.error('Error:', error)
  })

// 2. Callback方式
adminManager.enableAdmin(want, entInfo, type, userId, (err, data) => {
  if (err.code === 0) {
    console.log('Success:', data)
  }
})
```

证据：所有`*_addon.cpp`文件中的`HandleAsyncWork`调用

---

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 架构和数据流
- [08_Security_Review.md](08_Security_Review.md) - 安全注意事项
