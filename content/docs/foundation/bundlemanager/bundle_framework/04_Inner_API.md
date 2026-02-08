# 内部 API 文档

## 概述

Inner API 位于 `interfaces/inner_api/` 目录，供 OpenHarmony 其他子系统使用，仅限 C++ 调用。

---

## 目录结构

```
interfaces/inner_api/
├── appexecfwk_base/           # 基础数据结构
│   ├── include/
│   │   ├── ability_info.h
│   │   ├── application_info.h
│   │   ├── bundle_info.h
│   │   ├── hap_module_info.h
│   │   ├── extension_ability_info.h
│   │   ├── bundle_constants.h
│   │   ├── appexecfwk_errors.h
│   │   └── ...
│   └── src/
├── appexecfwk_core/           # 核心 IPC 接口
│   ├── include/bundlemgr/
│   │   ├── bundle_mgr_interface.h
│   │   ├── bundle_mgr_proxy.h
│   │   ├── bundle_mgr_host.h
│   │   ├── bundle_installer_interface.h
│   │   ├── bundle_installer_proxy.h
│   │   ├── bundle_user_mgr_interface.h
│   │   ├── ...
│   └── ...
├── bundlemgr_extension/       # 扩展接口
│   └── ...
└── bundlemgr_graphics/        # 图形相关
    └── ...
```

---

## 核心 IPC 接口

### IBundleMgr

**定义**: `bundle_mgr_interface.h`
**实现**: `BundleMgrHost` (stub), `BundleMgrProxy` (proxy)

#### 主要方法

| 方法 | 功能 |
|------|------|
| `GetBundleInfo(bundleName, flags, bundleInfo, userId)` | 获取包信息 |
| `GetBundleInfos(flags, bundleInfos, userId)` | 获取所有包信息 |
| `GetApplicationInfo(bundleName, flags, appInfo, userId)` | 获取应用信息 |
| `GetAbilityInfo(want, flags, abilityInfo, userId)` | 获取 Ability 信息 |
| `QueryAbilityInfos(want, flags, abilityInfos, userId)` | 查询 Ability 列表 |
| `Install(bundlePath, installParam, installResult)` | 安装 |
| `Uninstall(bundleName, installParam, installResult)` | 卸载 |
| `SetApplicationEnabled(bundleName, enabled, userId)` | 设置应用启用状态 |
| `IsAbilityEnabled(abilityInfo, userId)` | 检查 Ability 是否启用 |

**证据来源**: `interfaces/inner_api/appexecfwk_core/include/bundlemgr/bundle_mgr_interface.h`

---

### IBundleInstaller

**定义**: `bundle_installer_interface.h`

#### 主要方法

| 方法 | 功能 |
|------|------|
| `Install(bundlePath, installParam, resultCode)` | 安装 |
| `Uninstall(bundleName, installParam, resultCode)` | 卸载 |
| `Recover(bundleName, installParam, resultCode)` | 恢复 |
| `InstallPreexistingApp(bundleName, userId)` | 安装预置应用 |

---

### IInstalld

**定义**: `services/bundlemgr/include/ipc/installd_interface.h:41-594`

#### 主要方法

| 方法 | 功能 |
|------|------|
| `CreateBundleDir(bundlePath)` | 创建包目录 |
| `ExtractFiles(hapSourcePath, targetPath)` | 解压文件 |
| `Rename(bundleDir, newBundleName)` | 重命名 |
| `MkDir(dirPath, dirOwner, bundleName)` | 创建目录 |
| `Rmdir(dirPath)` | 删除目录 |
| `CleanDir(dirPath)` | 清空目录 |
| `GetBundleStats(bundleDir, bundleStats)` | 获取包统计 |
| `SetDirOwner(dirPath, uid, gid)` | 设置目录所有者 |
| `GetBundleCacheSize(bundleName, cacheSize)` | 获取缓存大小 |
| `CleanBundleCacheFiles(bundleName)` | 清理缓存 |
| `LoadModule(path, hapName, module)` | 加载模块 |

**证据来源**: `services/bundlemgr/include/ipc/installd_interface.h`

---

## 核心数据结构

### AbilityInfo

```cpp
struct AbilityInfo {
    std::string name;                    // Ability 名称
    std::string bundleName;              // 包名
    std::string moduleName;              // 模块名
    AbilityType type;                    // Ability 类型
    AbilitySubType subType;              // 子类型
    std::string displayName;             // 显示名称
    std::string description;             // 描述
    std::string iconPath;               // 图标路径
    LaunchMode launchMode;               // 启动模式
    std::vector<std::string> permissions; // 所需权限
    bool visible;                        // 是否可见
    // ...
};
```

### ApplicationInfo

```cpp
struct ApplicationInfo {
    std::string bundleName;              // 包名
    std::string versionName;              // 版本名
    int versionCode;                     // 版本号
    std::string signatureKey;             // 签名密钥
    std::vector<std::string> permissions; // 权限列表
    std::string entryDir;                // 入口目录
    std::string resourcePath;             // 资源路径
    // ...
};
```

### BundleInfo

```cpp
struct BundleInfo {
    std::string name;                     // 包名
    std::string vendor;                   // 厂商
    int versionCode;                      // 版本号
    std::string versionName;              // 版本名
    std::string hwSignature;              // 华为签名
    ApplicationInfo appInfo;              // 应用信息
    std::map<std::string, AbilityInfo> abilityInfos;  // Ability 列表
    std::vector<HapModuleInfo> hapModules; // HAP 模块列表
    // ...
};
```

---

## 权限校验

### BundlePermissionMgr

**位置**: `services/bundlemgr/include/bundle_permission_mgr.h`

#### 关键方法

| 方法 | 功能 |
|------|------|
| `VerifyCallingPermissionForAll(permission)` | 验证调用方权限 |
| `VerifyCallingPermissionsForAll(permissions)` | 批量验证权限 |
| `IsSystemApp()` | 检查是否是系统应用 |
| `VerifySystemApp()` | 验证系统应用 |
| `CheckPermission(bundleName, permission)` | 检查权限 |
| `GrantPermission(bundleName, permission)` | 授予权限 |
| `GetReqPermissions(bundleName, reqPermissions)` | 获取请求权限 |

**证据来源**: `services/bundlemgr/src/bundle_permission_mgr.cpp:311`

---

## 依赖方向

```
                    ┌─────────────────────┐
                    │  其他子系统 (调用方)   │
                    └──────────┬──────────┘
                               │ 调用 Inner API
                               ▼
┌─────────────────────────────────────────────────────────┐
│              interfaces/inner_api/                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │            IBundleMgr (接口定义)                   │   │
│  └─────────────────────────────────────────────────┘   │
│                              │                           │
│                    ┌─────────┴─────────┐               │
│                    ▼                   ▼               │
│  ┌──────────────────────┐  ┌──────────────────────┐   │
│  │ BundleMgrProxy       │  │   BundleMgrHost      │   │
│  │ (客户端 IPC Proxy)    │  │   (服务端 Stub)       │   │
│  └──────────────────────┘  └──────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼ IPC
┌─────────────────────────────────────────────────────────┐
│           services/bundlemgr/src/                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │          BundleMgrHostImpl                       │   │
│  │          (IBundleMgr 接口实现)                    │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │          BundleDataMgr                          │   │
│  │          (数据管理)                               │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 稳定性标注

### 稳定接口

以下接口为 **稳定接口**，可供其他子系统使用：

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `IBundleMgr::GetBundleInfo` | 稳定 | 核心查询接口 |
| `IBundleMgr::GetBundleInfos` | 稳定 | 批量查询接口 |
| `IBundleMgr::GetApplicationInfo` | 稳定 | 应用信息查询 |
| `IBundleMgr::QueryAbilityInfos` | 稳定 | Ability 查询 |
| `IBundleInstaller::Install` | 稳定 | 安装接口 |
| `IBundleInstaller::Uninstall` | 稳定 | 卸载接口 |

### 内部实现

以下目录/接口为 **内部实现**，不推荐直接调用：

| 目录/文件 | 说明 |
|-----------|------|
| `services/bundlemgr/src/common/` | 服务层公共工具 |
| `services/bundlemgr/src/data/` | 数据处理 |
| `rdb/` | RDB 封装 |

---

## 错误码定义

**文件**: `interfaces/inner_api/appexecfwk_base/include/appexecfwk_errors.h`

| 错误码 | 说明 |
|--------|------|
| `ERR_OK` | 成功 |
| `ERR_APPEXECFWK_INSTALL_FAILED` | 安装失败 |
| `ERR_APPEXECFWK_UNINSTALL_FAILED` | 卸载失败 |
| `ERR_APPEXECFWK_BUNDLE_NOT_FOUND` | 包不存在 |
| `ERR_APPEXECFWK_PERMISSION_DENIED` | 权限拒绝 |
| `ERR_APPEXECFWK_INVALID_PARAMETER` | 无效参数 |

---

## 延伸阅读

- [N-API 参考](03_N-API_Reference.md)
- [架构说明](02_Architecture.md)
- [安全风险评审](07_Security_Review.md)
