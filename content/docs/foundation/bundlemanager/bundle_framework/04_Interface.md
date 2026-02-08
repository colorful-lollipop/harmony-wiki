# 接口汇总

## 概述

本文档汇总 Bundle Framework 的所有对外接口，包括 N-API、IPC 和配置文件说明。

**目标受众**：需要快速查找接口的开发者

**详细文档**：
- JS API 详细说明 → [03_N-API_Reference](03_N-API_Reference.md)
- Inner API 详细说明 → [04_Inner_API](04_Inner_API.md)

---

## 1. N-API 接口清单

### 1.1 JS N-API 模块

| 模块 | 命名空间 | 文件位置 | 导出方法数 | 主要功能 |
|------|----------|----------|------------|----------|
| **bundleManager** | `bundle.bundleManager` | `bundle_manager/` | 98+ | **主包管理器 API（推荐）** |
| **bundlemgr** | `bundle` | `bundlemgr/` | 24 | 传统包管理器 API（兼容） |
| **installer** | `bundle.installer` | `installer/` | 13 | 安装器操作 |
| **launcherBundleManager** | `bundle.launcherBundleManager` | `launcher_bundle_manager/` | 8 | 启动器包管理 |
| **launchermgr** | `bundle.innerBundleManager` | `launchermgr/` | 5 | 内部启动器服务 |
| **appControl** | `bundle.appControl` | `app_control/` | 16 | 应用控制/处置规则 |
| **defaultApp** | `bundle.defaultAppManager` | `default_app/` | 10 | 默认应用管理 |
| **overlay** | `bundle.overlay` | `overlay/` | 6 | 叠加包管理 |
| **freeInstall** | `bundle.freeInstall` | `free_install/` | 6 | 自由安装/按需安装 |
| **bundleResource** | `bundle.bundleResourceManager` | `bundle_resource/` | 8 | 包资源管理 |
| **bundleMonitor** | `bundle.bundleMonitor` | `bundle_monitor/` | 2 | 包变更监控 |
| **shortcutManager** | `bundle.shortcutManager` | `shortcut_manager/` | 8 | 快捷方式管理 |
| **package** | `package` | `package/` | 1 | 包工具 |
| **zlib** | `zlib` | `zip/` | 15+ | ZIP 压缩工具 |

**证据来源**: `interfaces/kits/js/` 各模块的 `native_module.cpp`

### 1.2 Native NDK 接口

| 头文件 | 主要类型 | 说明 |
|--------|----------|------|
| `native_interface_bundle.h` | `OH_BundleMgr` | 主包管理器句柄 |
| `bundle_manager_common.h` | 错误码、枚举 | 公共类型和错误码 |

### 1.3 N-API 清单表

#### bundle.bundleManager 模块

| JS API | 参数/返回 | 同步/异步 | 权限要求 |
|--------|-----------|-----------|----------|
| `getBundleInfo(name, options)` | BundleInfo | Async | - |
| `getBundleInfoSync(name, options)` | BundleInfo | Sync | - |
| `getAllBundleInfo(options)` | BundleInfo[] | Async | `GET_BUNDLE_INFO_PRIVILEGED` |
| `getApplicationInfo(bundleName, options)` | ApplicationInfo | Async | - |
| `getApplicationInfoSync(bundleName, options)` | ApplicationInfo | Sync | - |
| `getAbilityInfo(want)` | AbilityInfo | Async | - |
| `getAbilityIcon(bundleName, abilityName)` | image.PixelMap | Async | - |
| `setApplicationEnabled(bundleName, enabled)` | void | Async | `CHANGE_BUNDLE_ENABLED_STATE` |
| `setAbilityEnabled(ability, enabled)` | void | Async | `CHANGE_BUNDLE_ENABLED_STATE` |
| `isApplicationEnabled(bundleName)` | boolean | Async | - |
| `isAbilityEnabled(ability)` | boolean | Async | - |
| `cleanBundleCacheFiles(bundleName)` | void | Async | `REMOVE_CACHE_FILES` |
| `getBundleArchiveInfo(hapFilePath)` | BundleInfo | Async | - |
| `getPermissionDef(permissionName)` | PermissionDef | Async | - |
| `getDynamicIcon(bundleName)` | DynamicIconInfo | Async | - |
| `enableDynamicIcon(bundleName)` | void | Async | `DYNAMIC_ICON` |
| `disableDynamicIcon(bundleName)` | void | Async | `DYNAMIC_ICON` |
| `getSandboxDataDir(bundleName, appIndex)` | string | Async | - |
| `createAppClone(bundleName)` | number | Async | `START_INSTALLED_APP` |
| `destroyAppClone(bundleName, appIndex)` | void | Async | - |
| `getAppCloneBundleInfo(bundleName, appIndex)` | BundleInfo | Async | - |

---

## 2. IPC 接口清单

### 2.1 系统能力接口

#### IBundleMgr (SA 401)

**定义**: `interfaces/inner_api/appexecfwk_core/include/bundlemgr/bundle_mgr_interface.h`

| 方法 | 功能 | 权限检查 |
|------|------|----------|
| `GetBundleInfo(bundleName, flags, bundleInfo, userId)` | 获取包信息 | 视 flags 而定 |
| `GetBundleInfos(flags, bundleInfos, userId)` | 获取所有包信息 | `GET_BUNDLE_INFO_PRIVILEGED` |
| `GetApplicationInfo(bundleName, flags, appInfo, userId)` | 获取应用信息 | 视 flags 而定 |
| `GetAbilityInfo(want, flags, abilityInfo, userId)` | 获取 Ability 信息 | - |
| `Install(bundlePath, installParam, resultCode)` | 安装 | `INSTALL_BUNDLE` |
| `Uninstall(bundleName, installParam, resultCode)` | 卸载 | `UNINSTALL_BUNDLE` |
| `SetApplicationEnabled(bundleName, enabled, userId)` | 设置启用状态 | `CHANGE_BUNDLE_ENABLED_STATE` |
| `CleanBundleCacheFiles(bundleName, userId)` | 清理缓存 | `REMOVE_CACHE_FILES` |

#### IBundleInstaller (SA 401)

**定义**: `interfaces/inner_api/appexecfwk_core/include/bundlemgr/bundle_installer_interface.h`

| 方法 | 功能 | 权限检查 |
|------|------|----------|
| `Install(bundlePath, installParam, resultCode)` | 安装 | `INSTALL_BUNDLE` |
| `Uninstall(bundleName, installParam, resultCode)` | 卸载 | `UNINSTALL_BUNDLE` |
| `Recover(bundleName, installParam, resultCode)` | 恢复 | `MANAGE_INTENT` |

#### IInstalld (SA 511)

**定义**: `services/bundlemgr/include/ipc/installd_interface.h`

| 方法 | 功能 | 调用方 |
|------|------|--------|
| `CreateBundleDir(bundlePath)` | 创建包目录 | SA 401 |
| `ExtractFiles(hapSourcePath, targetPath)` | 解压文件 | SA 401 |
| `MkDir(dirPath, dirOwner, bundleName)` | 创建目录 | SA 401 |
| `Rmdir(dirPath)` | 删除目录 | SA 401 |
| `CleanDir(dirPath)` | 清空目录 | SA 401 |
| `SetDirOwner(dirPath, uid, gid)` | 设置所有者 | SA 401 |
| `GetBundleStats(bundleDir, bundleStats)` | 获取统计 | SA 401 |
| `CleanBundleCacheFiles(bundleName)` | 清理缓存 | SA 401 |

### 2.2 IPC 调用方式

```cpp
// 获取 IBundleMgr
auto bundleMgr = iface_cast<IBundleMgr>(
    SystemAbilityHelper::GetSystemAbility(BUNDLE_MGR_SERVICE_SYS_ABILITY_ID));

// 调用 IPC 方法
InstallParam param;
param.userId = 100;
int32_t resultCode = 0;
bundleMgr->Install(hapPath, param, resultCode);
```

---

## 3. 配置文件说明

### 3.1 SA 配置文件

#### sa_profile/401.json

```json
{
    "process": "foundation",
    "systemability": [{
        "name": 401,
        "libpath": "libbms.z.so",
        "run-on-create": true,
        "depend": [3503],
        "extension": ["backup", "restore"]
    }]
}
```

| 字段 | 说明 |
|------|------|
| `process` | 运行进程（foundation） |
| `libpath` | 库路径 |
| `run-on-create` | 随系统启动 |
| `depend` | 依赖服务（Common Event） |

#### sa_profile/511.json

```json
{
    "process": "installs",
    "systemability": [{
        "name": 511,
        "libpath": "libinstalls.z.so",
        "run-on-create": false,
        "stop-on-demand": {
            "longtimeunused-unload": 180
        }
    }]
}
```

| 字段 | 说明 |
|------|------|
| `process` | 运行进程（installs） |
| `libpath` | 库路径 |
| `stop-on-demand` | 空闲 180 秒后卸载 |

### 3.2 系统配置

#### etc/bms.para

Bundle Manager 系统参数配置。

| 参数 | 说明 |
|------|------|
| `BMS_DATA_DIR` | 数据目录（/data/bms/） |
| `BMS_INSTALL_TMP_DIR` | 安装临时目录 |

### 3.3 Feature 配置

**文件**: `appexecfwk.gni`

| Feature 开关 | 默认值 | 启用模块 |
|--------------|--------|----------|
| `bundle_framework_free_install` | true | free_install/ |
| `bundle_framework_default_app` | true | default_app/ |
| `bundle_framework_quick_fix` | true | quick_fix/ |
| `bundle_framework_overlay_install` | true | overlay/ |
| `bundle_framework_sandbox_app` | true | sandbox_app/ |

---

## 4. 错误码定义

### 4.1 通用错误码

| 错误码 | 说明 |
|--------|------|
| `0` | 成功 |
| `1` | 参数错误 |
| `2` | 内存错误 |
| `3` | 未知错误 |

### 4.2 安装错误码

| 错误码 | 说明 |
|--------|------|
| `0x0101` | 安装成功 |
| `0x0102` | 安装包无效 |
| `0x0103` | 签名验证失败 |
| `0x0104` | 权限不足 |
| `0x0105` | 路径无效 |
| `0x0106` | 包名重复 |
| `0x0107` | 文件解析失败 |
| `0x0108` | 内部错误 |

### 4.3 数据库错误码

| 错误码 | 说明 |
|--------|------|
| `0x0201` | 数据库未初始化 |
| `0x0202` | 数据库操作失败 |
| `0x0203` | 数据不存在 |
| `0x0204` | 数据已存在 |

**证据来源**: `interfaces/kits/native/inner_api/appexecfwk_errors.h`

---

## 5. 权限清单

### 5.1 Bundle 权限

| 权限名 | 说明 | 保护操作 |
|--------|------|----------|
| `ohos.permission.INSTALL_BUNDLE` | 安装应用 | Install |
| `ohos.permission.UNINSTALL_BUNDLE` | 卸载应用 | Uninstall |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 查询所有包信息 | GetAllBundleInfo |
| `ohos.permission.CHANGE_BUNDLE_ENABLED_STATE` | 更改启用状态 | SetApplicationEnabled |
| `ohos.permission.REMOVE_CACHE_FILES` | 清理缓存 | CleanBundleCacheFiles |
| `ohos.permission.MANAGE_INTENT` | 管理 Intent | Recover |

### 5.2 权限检查点

| 检查点 | 文件 | 说明 |
|--------|------|------|
| N-API 层 | `napi_bundle_manager.cpp` | JS 参数校验 |
| IPC Host 层 | `bundle_mgr_host_impl.cpp` | 调用者身份验证 |
| Service 层 | `bundle_permission_mgr.cpp` | 权限校验 |
| Installd 层 | `installd_permission_mgr.cpp` | UID 验证 |

---

## 6. 延伸阅读

- [03_N-API_Reference](03_N-API_Reference.md) - JS/N-API 详细接口
- [04_Inner_API](04_Inner_API.md) - Inner API 详细接口
- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析
- [07_Build](07_Build.md) - 构建配置