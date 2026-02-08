# 代码地图

## 概述

本文档提供 Bundle Framework 的代码导航图，帮助开发者快速定位关键功能对应的源代码文件。

**目标受众**：新人开发者、需要理解特定功能的工程师

**使用方式**：根据功能描述，查找对应的文件路径和代码位置

---

## 1. 核心入口定位

### 1.1 系统能力入口

| 功能 | 文件 | 行号 | 说明 |
|------|------|------|------|
| **SA 401 注册** | `services/bundlemgr/src/bundle_mgr_service.cpp` | 45-80 | BundleMgrService 主服务初始化 |
| **SA 511 注册** | `services/bundlemgr/src/installd/installd_service.cpp` | 30-60 | InstalldService 特权服务初始化 |
| **IPC Host 注册** | `services/bundlemgr/src/bundle_mgr_host_impl.cpp` | 20-50 | IBundleMgr 接口注册 |

**代码证据**：
```cpp
// services/bundlemgr/src/bundle_mgr_service.cpp:45-60
REGISTER_SYSTEM_ABILITY_BY_ID(BundleMgrService, SUBSYS_BUNDLEMANAGER,
    SAMGR_BUNDLE_MANAGER_SA_ID, false);
// ...
```

### 1.2 N-API 注册入口

| 功能 | 文件 | 行号 | 说明 |
|------|------|------|------|
| bundleManager 模块注册 | `interfaces/kits/js/bundle_manager/src/napi_bundle_manager.cpp` | 50-100 | 主包管理器 API |
| installer 模块注册 | `interfaces/kits/js/installer/src/napi_bundle_installer.cpp` | 30-80 | 安装器 API |
| appControl 模块注册 | `interfaces/kits/js/app_control/src/napi_app_control.cpp` | 30-70 | 应用控制 API |

---

## 2. 功能→文件映射

### 2.1 安装流程

| 子功能 | 关键文件 | 行号 |
|--------|----------|------|
| 安装入口 | `bundle_mgr_host_impl.cpp` | `Install()` |
| 安装编排 | `bundle_installer.cpp` | `Install()` |
| 基础安装逻辑 | `base_bundle_installer.cpp` | `Install()` |
| 安装前检查 | `bundle_install_checker.cpp` | `CheckInner()` |
| HAP 解析 | `bundle_parser.cpp` | `ParseBundleInfo()` |
| 签名验证 | `bundle_verify_mgr.cpp` | `VerifySignature()` |
| 权限校验 | `bundle_permission_mgr.cpp` | `VerifyCallingPermission()` |
| 特权文件操作 | `installd_service.cpp` | `ExtractFiles()`, `MkDir()` |
| 数据持久化 | `bundle_data_mgr.cpp` | `AddInnerBundleInfo()` |

**完整调用链**：
```
NAPI_Install()
  ↓ (interfaces/kits/js/installer/src/napi_bundle_installer.cpp)
BundleMgrHostImpl::Install()
  ↓ (services/bundlemgr/src/bundle_mgr_host_impl.cpp)
BundleInstaller::Install()
  ↓ (services/bundlemgr/src/bundle_installer.cpp)
BaseBundleInstaller::Install()
  ↓ (services/bundlemgr/src/base_bundle_installer.cpp)
BundleInstallChecker::CheckInner()
  ↓ (services/bundlemgr/src/bundle_install_checker.cpp)
BundleVerifyMgr::VerifySignature()
  ↓ (services/bundlemgr/src/bundle_verify_mgr.cpp)
InstalldClient::ExtractFiles()
  ↓ (services/bundlemgr/src/installd_client.cpp)
InstalldService::ExtractFiles()
  ↓ (services/bundlemgr/src/installd/installd_service.cpp)
BundleDataMgr::AddInnerBundleInfo()
  ↓ (services/bundlemgr/src/bundle_data_mgr.cpp)
```

### 2.2 卸载流程

| 子功能 | 关键文件 | 行号 |
|--------|----------|------|
| 卸载入口 | `bundle_mgr_host_impl.cpp` | `Uninstall()` |
| 卸载编排 | `bundle_installer.cpp` | `Uninstall()` |
| 卸载前检查 | `bundle_install_checker.cpp` | `CheckUninstall()` |
| 用户数据清理 | `uninstall_data_mgr.cpp` | `ProcessUninstallData()` |
| 文件删除 | `installd_service.cpp` | `RemoveDir()` |
| 数据清理 | `bundle_data_mgr.cpp` | `RemoveBundleInfo()` |

### 2.3 查询流程

| 子功能 | 关键文件 | 行号 |
|--------|----------|------|
| 包信息查询 | `bundle_mgr_host_impl.cpp` | `GetBundleInfo()` |
| 数据查询 | `bundle_data_mgr.cpp` | `GetBundleInfo()` |
| 应用信息查询 | `bundle_mgr_host_impl.cpp` | `GetApplicationInfo()` |
| Ability 查询 | `bundle_mgr_host_impl.cpp` | `GetAbilityInfo()` |

### 2.4 权限管理

| 子功能 | 关键文件 | 行号 |
|--------|----------|------|
| 权限校验入口 | `bundle_permission_mgr.cpp` | `VerifyCallingPermission()` |
| AccessToken 集成 | `bundle_permission_mgr.cpp` | `VerifyAccessToken()` |
| 权限定义 | `interfaces/inner_api/appexecfwk_base/include/bundle_constants.h` | - |

### 2.5 数据持久化

| 子功能 | 关键文件 | 行号 |
|--------|----------|------|
| RDB 初始化 | `bundle_data_storage_rdb.cpp` | `InitRdbStore()` |
| BundleInfo 存储 | `rdb/bundle_info_storage.cpp` | `Insert()`, `Update()` |
| BundleState 存储 | `bundle_state_storage.cpp` | `SaveBundleState()` |

---

## 3. 关键数据结构

### 3.1 核心类

| 类名 | 文件 | 职责 |
|------|------|------|
| `BundleMgrService` | `bundle_mgr_service.cpp` | SA 401 主服务，协调所有操作 |
| `BundleDataMgr` | `bundle_data_mgr.cpp` | Bundle 信息中央存储与查询 |
| `BundleInstaller` | `bundle_installer.cpp` | 安装/卸载操作编排 |
| `BaseBundleInstaller` | `base_bundle_installer.cpp` | 基础安装逻辑实现 |
| `BundleInstallChecker` | `bundle_install_checker.cpp` | 安装前校验 |
| `BundleParser` | `bundle_parser.cpp` | HAP 包解析 |
| `InnerBundleInfo` | `inner_bundle_info.cpp` | Bundle 内部表示 |
| `BundlePermissionMgr` | `bundle_permission_mgr.cpp` | 权限管理 |
| `BundleVerifyMgr` | `bundle_verify_mgr.cpp` | 签名验证管理 |
| `InstalldClient` | `installd_client.cpp` | SA 511 IPC 客户端 |

### 3.2 关键结构体

| 结构体 | 文件 | 用途 |
|--------|------|------|
| `BundleInfo` | `ability_info.h` | 对外 Bundle 信息 |
| `InnerBundleInfo` | `inner_bundle_info.h` | 内部 Bundle 表示 |
| `ApplicationInfo` | `application_info.h` | 应用信息 |
| `AbilityInfo` | `ability_info.h` | Ability 信息 |
| `ModuleInfo` | `module_info.h` | 模块信息 |
| `InstallParam` | `install_param.h` | 安装参数 |
| `InstallResult` | `install_result.h` | 安装结果 |

---

## 4. 配置文件位置

### 4.1 SA 配置

| 文件 | 用途 |
|------|------|
| `sa_profile/401.json` | BundleMgrService SA 配置 |
| `sa_profile/511.json` | InstalldService SA 配置 |

### 4.2 构建配置

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 根构建配置 |
| `appexecfwk.gni` | Feature flags 定义 |
| `services/bundlemgr/BUILD.gn` | 服务层构建配置 |
| `interfaces/kits/js/BUILD.gn` | N-API 构建配置 |

### 4.3 系统配置

| 文件 | 用途 |
|------|------|
| `etc/bms.para` | Bundle Manager 系统参数 |
| `etc/bms.para.dac` | DAC 参数配置 |

---

## 5. 测试代码定位

**注意**：以下测试目录在代码导航时应当忽略。

| 目录 | 用途 |
|------|------|
| `test/` | 系统级测试 |
| `services/bundlemgr/test/` | 服务层单元测试 |
| `services/bundlemgr/src/*_test.cpp` | 单元测试文件 |

---

## 6. 常见问题定位

| 问题 | 排查文件 |
|------|----------|
| 安装失败 | `bundle_install_checker.cpp`, `bundle_verify_mgr.cpp` |
| 权限问题 | `bundle_permission_mgr.cpp` |
| 数据查询异常 | `bundle_data_mgr.cpp`, `rdb/` |
| IPC 通信问题 | `ipc/`, `bundle_mgr_host_impl.cpp` |
| 文件操作问题 | `installd_service.cpp` |
| 性能问题 | `bundle_data_mgr.cpp`, `bundle_util.cpp` |

---

## 7. 延伸阅读

- [01_Directory_Structure](01_Directory_Structure.md) - 目录结构概览
- [02_Architecture](02_Architecture.md) - 架构说明
- [04_Interface](04_Interface.md) - 接口文档
- [08_Internals](08_Internals.md) - 内部实现细节
