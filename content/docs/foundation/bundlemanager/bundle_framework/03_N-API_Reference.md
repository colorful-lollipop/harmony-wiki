# N-API 接口参考

## 概述

Bundle Framework 提供三类 N-API 接口：

| 类型 | 目录 | 说明 |
|------|------|------|
| **JS N-API** | `interfaces/kits/js/` | JavaScript/TypeScript API |
| **Native NDK** | `interfaces/kits/native/` | C/C++ 原生 API |
| **ANI** | `interfaces/kits/ani/` | ArkTS Native Interface |

---

## JS N-API 模块清单

### 模块注册模式

所有 JS N-API 模块遵循一致的注册模式：

```cpp
// 1. 模块定义
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,           // 初始化函数
    .nm_modname = "bundle.xxx",         // 模块名
    .nm_priv = ((void *)0),
    .reserved = {0}
};

// 2. 注册入口 (C++ 构造函数)
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&_module);
}

// 3. Init 函数 (导出方法和属性)
static napi_value Init(napi_env env, napi_value exports)
{
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("methodName", MethodName),
        DECLARE_NAPI_PROPERTY("PropertyName", propertyValue),
        // ...
    };
    NAPI_CALL(env, napi_define_properties(env, exports, sizeof(desc)/sizeof(desc[0]), desc));
    return exports;
}
```

**证据来源**: `interfaces/kits/js/bundlemgr/native_module.cpp:142-157`

---

## 核心模块详解

### 1. bundle.bundleManager (推荐)

**模块名**: `bundle.bundleManager`
**文件**: `interfaces/kits/js/bundle_manager/native_module.cpp`
**导出方法数**: 98+

#### 函数清单

| 方法名 | 同步/异步 | 功能 |
|--------|----------|------|
| `getBundleInfo(bundleName, options)` | Promise | 获取包信息 |
| `getBundleInfoSync(bundleName, options)` | Sync | 同步获取包信息 |
| `getAllBundleInfo(options)` | Promise | 获取所有包信息 |
| `getApplicationInfo(bundleName, options)` | Promise | 获取应用信息 |
| `getApplicationInfoSync(bundleName, options)` | Sync | 同步获取应用信息 |
| `getAbilityInfo(want)` | Promise | 获取 Ability 信息 |
| `getAbilityIcon(bundleName, abilityName)` | Promise | 获取 Ability 图标 |
| `setApplicationEnabled(bundleName, enabled)` | Promise | 设置应用启用状态 |
| `setAbilityEnabled(ability, enabled)` | Promise | 设置 Ability 启用状态 |
| `isApplicationEnabled(bundleName)` | Promise | 检查应用是否启用 |
| `isAbilityEnabled(ability)` | Promise | 检查 Ability 是否启用 |
| `cleanBundleCacheFiles(bundleName)` | Promise | 清理包缓存 |
| `getLaunchWantForBundle(bundleName)` | Promise | 获取启动 Want |
| `queryAbilityInfo(want)` | Promise | 按 Want 查询 Ability |
| `getBundleNameByUid(uid)` | Promise | 根据 UID 获取包名 |
| `getBundleArchiveInfo(hapFilePath)` | Promise | 获取 HAP 包信息 |
| `getPermissionDef(permissionName)` | Promise | 获取权限定义 |
| `getProfileByAbility(...)` | Promise | 获取 Ability 配置文件 |
| `getDynamicIcon(bundleName)` | Promise | 获取动态图标 |
| `enableDynamicIcon(bundleName)` | Promise | 启用动态图标 |
| `disableDynamicIcon(bundleName)` | Promise | 禁用动态图标 |
| `getSharedBundleInfo(...)` | Promise | 获取共享包信息 |
| `getAppProvisionInfo(bundleName)` | Promise | 获取 Provision 信息 |
| `verifyAbc(bundleName, moduleName)` | Promise | 验证 ABC 签名 |
| `deleteAbc(bundleName, moduleName)` | Promise | 删除 ABC 文件 |
| `getAllSharedBundleInfo(bundleName)` | Promise | 获取所有共享包信息 |
| `canOpenLink(link)` | Promise | 检查是否能打开链接 |
| `getSandboxDataDir(bundleName, appIndex)` | Promise | 获取沙箱数据目录 |
| `createAppClone(bundleName)` | Promise | 创建应用克隆 |
| `destroyAppClone(bundleName, appIndex)` | Promise | 销毁应用克隆 |
| `getAppCloneBundleInfo(bundleName, appIndex)` | Promise | 获取克隆包信息 |
| `getExtResource(bundleName, moduleName)` | Promise | 获取扩展资源 |
| `startShortcut(shortcutWant)` | Promise | 启动快捷方式 |
| `getShortcutInfo(bundleName)` | Promise | 获取快捷方式信息 |
| `addDesktopShortcutInfo(bundleName, shortcutInfo)` | Promise | 添加桌面快捷方式 |
| `deleteDesktopShortcutInfo(bundleName)` | Promise | 删除桌面快捷方式 |
| `getAllDesktopShortcutInfo()` | Promise | 获取所有桌面快捷方式 |
| `setShortcutVisibleForSelf(shortcutWant)` | Promise | 设置快捷方式可见性 |
| `getBundlePackInfo(bundleName)` | Promise | 获取包打包信息 |
| `getBundleCacheSize()` | Promise | 获取包缓存大小 |
| `cleanAllBundleCache()` | Promise | 清理所有包缓存 |
| `getSignatureInfo(bundleName)` | Promise | 获取签名信息 |
| `getBundleInstallStatus(bundleName)` | Promise | 获取安装状态 |
| `switchUninstallState(bundleName, enable)` | Promise | 切换卸载状态 |
| `getPluginBundlePathForSelf(bundleName)` | Promise | 获取插件路径 |
| `getPluginInfo(bundleName)` | Promise | 获取插件信息 |
| `getRecoverableApplicationInfo(bundleName)` | Promise | 获取可恢复应用信息 |
| `setAdditionalInfo(bundleName, additionalInfo)` | Promise | 设置附加信息 |
| `getLaunchWant()` | Promise | 获取启动 Want |
| `getBundleStats(bundleName, userId)` | Promise | 获取包统计信息 |
| `migrateData(bundleName, migrateType)` | Promise | 迁移数据 |
| `getAllPreinstalledApplicationInfo()` | Promise | 获取所有预装应用信息 |
| `isHapModuleRemovable(bundleName, moduleName)` | Promise | 检查模块是否可移除 |
| `setHapModuleUpgradeFlag(bundleName, moduleName, upgradeFlag)` | Promise | 设置模块升级标志 |

#### 枚举属性

| 枚举名 | 说明 |
|--------|------|
| `AbilityFlag` | Ability 标志 |
| `ExtensionAbilityFlag` | 扩展 Ability 标志 |
| `ExtensionAbilityType` | 扩展 Ability 类型 |
| `ApplicationFlag` | 应用标志 |
| `BundleFlag` | 包标志 |
| `PermissionGrantState` | 权限授予状态 |
| `AbilityType` | Ability 类型 |
| `DisplayOrientation` | 显示方向 |
| `LaunchType` | 启动类型 |
| `SupportWindowMode` | 支持的窗口模式 |
| `ModuleType` | 模块类型 |
| `BundleType` | 包类型 |
| `CompatiblePolicy` | 兼容策略 |
| `AppDistributionType` | 应用分发类型 |
| `MultiAppModeType` | 多应用模式类型 |
| `ApplicationInfoFlag` | 应用信息标志 |
| `BundleInstallStatus` | 包安装状态 |
| `ProfileType` | 配置文件类型 |

**证据来源**: `interfaces/kits/js/bundle_manager/native_module.cpp:27-221`

---

### 2. bundle.installer

**模块名**: `bundle.installer`
**文件**: `interfaces/kits/js/installer/native_module.cpp`

#### 函数清单

| 方法名 | 功能 |
|--------|------|
| `getBundleInstaller()` | 获取安装器实例 |
| `getBundleInstallerSync()` | 同步获取安装器实例 |

#### 类: BundleInstaller

| 方法 | 功能 |
|------|------|
| `install(hapFilePaths, installParam)` | 安装 HAP |
| `recover(bundleName)` | 恢复应用 |
| `uninstall(bundleName, uninstallParam)` | 卸载应用 |
| `updateBundleForSelf(hapFilePath)` | 更新自身 |
| `uninstallUpdates(bundleName)` | 卸载更新 |
| `addExtResource(bundleName, hapFilePath)` | 添加扩展资源 |
| `removeExtResource(bundleName, moduleName)` | 移除扩展资源 |
| `createAppClone(bundleName)` | 创建应用克隆 |
| `destroyAppClone(bundleName, appIndex)` | 销毁应用克隆 |
| `installPreexistingApp(bundleName)` | 安装预置应用 |
| `installPlugin(bundleName, hapFilePath)` | 安装插件 |
| `uninstallPlugin(bundleName)` | 卸载插件 |

**证据来源**: `interfaces/kits/js/installer/native_module.cpp:36-80`

---

### 3. bundle.launcherBundleManager

**模块名**: `bundle.launcherBundleManager`
**文件**: `interfaces/kits/js/launcher_bundle_manager/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `getLauncherAbilityInfo(want)` | 获取启动器 Ability 信息 |
| `getLauncherAbilityInfoSync(want)` | 同步获取启动器 Ability 信息 |
| `getAllLauncherAbilityInfo(userId)` | 获取所有启动器 Ability 信息 |
| `getShortcutInfo(bundleName)` | 获取快捷方式信息 |
| `getShortcutInfoSync(bundleName)` | 同步获取快捷方式信息 |
| `getShortcutInfoByAppIndex(bundleName, appIndex)` | 根据应用索引获取快捷方式 |
| `startShortcut(shortcutWant)` | 启动快捷方式 |
| `startShortcutWithReason(shortcutWant, reason)` | 带原因启动快捷方式 |

---

### 4. bundle.defaultAppManager

**模块名**: `bundle.defaultAppManager`
**文件**: `interfaces/kits/js/default_app/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `isDefaultApplication(type)` | 检查是否是默认应用 |
| `isDefaultApplicationSync(type)` | 同步检查默认应用 |
| `getDefaultApplication(type)` | 获取默认应用 |
| `getDefaultApplicationSync(type)` | 同步获取默认应用 |
| `setDefaultApplication(type, bundleName)` | 设置默认应用 |
| `setDefaultApplicationSync(type, bundleName)` | 同步设置默认应用 |
| `resetDefaultApplication(type)` | 重置默认应用 |
| `resetDefaultApplicationSync(type)` | 同步重置默认应用 |
| `setDefaultApplicationForAppClone(bundleName, appIndex, type)` | 为应用克隆设置默认应用 |

**枚举**: `ApplicationType` (BROWSER, IMAGE, AUDIO, VIDEO, PDF, WORD, EXCEL, PPT, EMAIL)

---

### 5. bundle.appControl

**模块名**: `bundle.appControl`
**文件**: `interfaces/kits/js/app_control/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `getDisposedStatus(bundleName)` | 获取处置状态 |
| `setDisposedStatus(bundleName, disposedStatus)` | 设置处置状态 |
| `deleteDisposedStatus(bundleName)` | 删除处置状态 |
| `getDisposedStatusSync(bundleName)` | 同步获取处置状态 |
| `setDisposedStatusSync(bundleName, disposedStatus)` | 同步设置处置状态 |
| `deleteDisposedStatusSync(bundleName)` | 同步删除处置状态 |
| `getDisposedRule(bundleName)` | 获取处置规则 |
| `getAllDisposedRules()` | 获取所有处置规则 |
| `getDisposedRulesByBundle(bundleName)` | 根据包名获取处置规则 |
| `setDisposedRule(bundleName, disposedRule)` | 设置处置规则 |
| `setDisposedRules(rules)` | 批量设置处置规则 |
| `getUninstallDisposedRule(bundleName)` | 获取卸载处置规则 |
| `setUninstallDisposedRule(bundleName, rule)` | 设置卸载处置规则 |
| `deleteUninstallDisposedRule(bundleName)` | 删除卸载处置规则 |

---

### 6. bundle.overlay

**模块名**: `bundle.overlay`
**文件**: `interfaces/kits/js/overlay/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `setOverlayEnabled(bundleName, moduleName, isEnabled)` | 设置叠加包启用状态 |
| `setOverlayEnabledByBundleName(bundleName, isEnabled)` | 根据包名设置叠加包启用 |
| `getOverlayModuleInfo(bundleName, moduleName)` | 获取叠加包模块信息 |
| `getTargetOverlayModuleInfos(bundleName, moduleName)` | 获取目标叠加包信息 |
| `getOverlayModuleInfoByBundleName(bundleName)` | 根据包名获取叠加包信息 |
| `getTargetOverlayModuleInfosByBundleName(bundleName)` | 根据包名获取目标叠加包信息 |

---

### 7. bundle.freeInstall

**模块名**: `bundle.freeInstall`
**文件**: `interfaces/kits/js/free_install/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `isHapModuleRemovable(bundleName, moduleName)` | 检查 HAP 模块是否可移除 |
| `setHapModuleUpgradeFlag(bundleName, moduleName, upgradeFlag)` | 设置模块升级标志 |
| `getBundlePackInfo(bundleName)` | 获取包打包信息 |
| `getDispatchInfo(want)` | 获取分发信息 |

**枚举**: `UpgradeFlag`, `BundlePackFlag`

---

### 8. bundle.bundleResourceManager

**模块名**: `bundle.bundleResourceManager`
**文件**: `interfaces/kits/js/bundle_resource/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `getBundleResourceInfo(bundleName, resourceFlag)` | 获取包资源信息 |
| `getLauncherAbilityResourceInfo(bundleName, resourceFlag)` | 获取启动器 Ability 资源信息 |
| `getAllBundleResourceInfo(resourceFlag)` | 获取所有包资源信息 |
| `getAllLauncherAbilityResourceInfo(resourceFlag)` | 获取所有启动器 Ability 资源信息 |
| `getLauncherAbilityResourceInfoList(want, resourceFlag)` | 获取启动器 Ability 资源信息列表 |
| `getExtensionAbilityResourceInfo(want, resourceFlag)` | 获取扩展 Ability 资源信息 |
| `getAllUninstalledBundleResourceInfo(resourceFlag)` | 获取所有卸载包的资源信息 |

**枚举**: `ResourceFlag`

---

### 9. bundle.bundleMonitor

**模块名**: `bundle.bundleMonitor`
**文件**: `interfaces/kits/js/bundle_monitor/bundle_monitor.cpp`

| 方法 | 功能 |
|------|------|
| `on(type, callback)` | 监听包变更事件 |
| `off(type, callback)` | 取消监听包变更事件 |

---

### 10. bundle.shortcutManager

**模块名**: `bundle.shortcutManager`
**文件**: `interfaces/kits/js/shortcut_manager/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `addDesktopShortcutInfo(bundleName, shortcutInfo)` | 添加桌面快捷方式 |
| `deleteDesktopShortcutInfo(bundleName)` | 删除桌面快捷方式 |
| `getAllDesktopShortcutInfo()` | 获取所有桌面快捷方式 |
| `setShortcutVisibleForSelf(shortcutWant)` | 设置快捷方式可见性 |
| `getAllShortcutInfoForSelf(bundleName)` | 获取自身所有快捷方式 |
| `addDynamicShortcutInfos(shortcutInfos)` | 添加动态快捷方式 |
| `deleteDynamicShortcutInfos(shortcutInfos)` | 删除动态快捷方式 |
| `setShortcutsEnabled(bundleName, enabled)` | 设置快捷方式启用状态 |

---

### 11. package

**模块名**: `package`
**文件**: `interfaces/kits/js/package/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `hasInstalled(bundleName)` | 检查包是否已安装 |

---

### 12. zlib

**模块名**: `zlib`
**文件**: `interfaces/kits/js/zip/napi/native_module.cpp`

| 方法 | 功能 |
|------|------|
| `zipFile(sourceFile, targetFile)` | 压缩单个文件 |
| `unzipFile(sourceFile, targetDir)` | 解压缩文件 |
| `compressFile(sourceFile, targetFile)` | 压缩文件 |
| `compressFiles(sourceFiles, targetFile)` | 批量压缩文件 |
| `decompressFile(sourceFile, targetDir)` | 解压缩文件 |
| `getOriginalSize(zipFile)` | 获取原始大小 |

**枚举**: FlushType, CompressLevel, CompressFlushMode, CompressMethod, CompressStrategy, ParallelStrategy, PathSeparatorStrategy, MemLevel, OffsetReferencePoint, ReturnStatus, ErrorCode

---

## 调用链示例

### JS → BundleMgrService 调用链

```
1. 应用 (JS)
   import bundleManager from '@bundle.bundleManager'
   bundleManager.getBundleInfo('com.example.app')
   ↓
2. N-API 胶水层 (interfaces/kits/js/bundle_manager/)
   bundle_manager.cpp → native_api 调用
   ↓
3. IPC Proxy (interfaces/inner_api/appexecfwk_core/)
   BundleMgrProxy::GetBundleInfo()
   ↓
4. IPC (Binder)
   ↓
5. BundleMgrHost (services/bundlemgr/src/)
   BundleMgrHostImpl::OnRemoteRequest()
   BundleMgrHostImpl::GetBundleInfo()
   ↓
6. BundlePermissionMgr::VerifyCallingPermissionForAll()
   ↓
7. BundleDataMgr::GetBundleInfo()
   ↓
8. RDB 查询
   ↓
9. 返回结果
```

---

## Native NDK 接口

### 主要头文件

| 头文件 | 说明 |
|--------|------|
| `native_interface_bundle.h` | 主要 Native 接口 |
| `bundle_mgr_proxy_native.h` | Native Bundle Manager 代理 |
| `ability_resource_info.h` | Ability 资源信息 API |
| `bundle_manager_common.h` | 公共类型和错误码 |

### API 版本

| 版本 | 新增 API |
|------|----------|
| API 9 | `OH_NativeBundle_GetCurrentApplicationInfo()`, `OH_NativeBundle_ApplicationInfo` |
| API 11 | `OH_NativeBundle_GetAppId()`, `OH_NativeBundle_GetAppIdentifier()` |
| API 13 | `OH_NativeBundle_ElementName`, `OH_NativeBundle_GetMainElementName()` |
| API 14 | `OH_NativeBundle_GetCompatibleDeviceType()` |
| API 20 | `OH_NativeBundle_IsDebugMode()`, `OH_NativeBundle_GetModuleMetadata()`, `OH_NativeBundle_Metadata` |
| API 21 | `OH_NativeBundle_GetAbilityResourceInfo()`, `BundleManager_ErrorCode` |

---

## 延伸阅读

- [架构说明](02_Architecture.md)
- [内部 API](04_Inner_API.md)
- [安全风险评审](07_Security_Review.md)
