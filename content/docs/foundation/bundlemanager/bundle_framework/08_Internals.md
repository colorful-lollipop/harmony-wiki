# 内部实现细节

## 概述

本文档深入解析 Bundle Framework 的内部实现细节，包括核心类职责、API 契约和资源生命周期管理。

**目标受众**：需要深入理解实现机制的开发者、安全研究员

**前置知识**：建议先阅读 [02_Architecture](02_Architecture.md) 和 [03_CodeMap](03_CodeMap.md)

---

## 1. 核心类职责

### 1.1 BundleMgrService

**文件**: `services/bundlemgr/src/bundle_mgr_service.cpp`

**职责**:
- SA 401 的主服务实现
- 协调所有 Bundle 管理操作
- 管理服务生命周期（OnStart/OnStop）

**初始化流程**:
```cpp
// bundle_mgr_service.cpp:122-150
bool BundleMgrService::Init() {
    if (ready_) return false;  // 防止重复初始化

    CreateBmsServiceDir();      // 创建服务目录
    InitBmsParam();             // 初始化参数
    InitPreInstallExceptionMgr(); // 预安装异常管理
    InitBundleMgrHost();        // IPC Host
    InitBundleInstaller();      // 安装器
    InitBundleDataMgr();        // 数据管理
    InitBundleUserMgr();        // 用户管理
    InitVerifyManager();        // 验证管理
    InitExtendResourceManager(); // 扩展资源
    InitBundleEventHandler();   // 事件处理
    InitFreeInstall();          // 自由安装
    InitDefaultApp();           // 默认应用
    InitAppControl();           // 应用控制
    InitBundleMgrExt();         // 扩展管理
    InitQuickFixManager();      // 快速修复
    InitOverlayManager();       // 叠加管理
    InitBundleResourceMgr();    // 资源管理
    // ...
}
```

**关键成员**:
| 成员 | 类型 | 职责 |
|------|------|------|
| `dataMgr_` | `sptr<BundleDataMgr>` | Bundle 数据管理 |
| `installer_` | `sptr<BundleInstaller>` | 安装器编排 |
| `handler_` | `EventHandler` | 事件处理 |
| `ready_` | `bool` | 服务就绪状态 |

### 1.2 BundleDataMgr

**文件**: `services/bundlemgr/src/bundle_data_mgr.cpp`

**职责**:
- Bundle 信息的中央存储与查询
- RDB 持久化管理
- 多用户数据管理

**关键方法**:
| 方法 | 职责 |
|------|------|
| `AddInnerBundleInfo()` | 添加 Bundle 信息 |
| `RemoveBundleInfo()` | 删除 Bundle 信息 |
| `GetBundleInfo()` | 查询 Bundle 信息 |
| `UpdateBundleInfo()` | 更新 Bundle 信息 |

### 1.3 BundleInstaller

**文件**: `services/bundlemgr/src/bundle_installer.cpp`

**职责**:
- 安装/卸载操作的编排
- 协调安装检查、解析、验证、安装
- 安装状态回调管理

**关键方法**:
| 方法 | 职责 |
|------|------|
| `Install()` | 安装入口 |
| `Uninstall()` | 卸载入口 |
| `Recover()` | 恢复安装 |

### 1.4 BaseBundleInstaller

**文件**: `services/bundlemgr/src/base_bundle_installer.cpp`

**职责**:
- 基础安装逻辑实现
- 实际的安装流程控制
- 与 InstalldService 交互

### 1.5 BundleInstallChecker

**文件**: `services/bundlemgr/src/bundle_install_checker.cpp`

**职责**:
- 安装前校验
- 权限检查
- 签名验证触发

### 1.6 InnerBundleInfo

**文件**: `services/bundlemgr/src/inner_bundle_info.cpp`

**职责**:
- Bundle 内部数据表示
- 包含所有 Bundle 相关数据
- 序列化和反序列化

---

## 2. 内部 API 契约

### 2.1 稳定接口（Inner API）

以下接口供其他子系统使用，稳定性较高：

| 接口 | 文件 | 说明 |
|------|------|------|
| `IBundleMgr` | `bundle_mgr_interface.h` | 主管理器接口 |
| `IBundleInstaller` | `bundle_installer_interface.h` | 安装器接口 |
| `IBundleStatusCallback` | `bundle_status_callback_interface.h` | 状态回调 |

**使用示例**:
```cpp
// 获取 IBundleMgr
auto bundleMgr = iface_cast<IBundleMgr>(SystemAbilityHelper::GetSystemAbility(BUNDLE_MGR_SERVICE_SYS_ABILITY_ID));
if (bundleMgr == nullptr) {
    return ERR_BUNDLEMANAGER_SERVICE_NOT_READY;
}

// 调用安装
InstallParam param;
param.userId = 100;
auto result = bundleMgr->Install(hapPath, param, nullptr);
```

### 2.2 内部实现（不应直接使用）

以下实现细节可能在版本间变化，不建议直接依赖：

| 类/模块 | 文件 | 说明 |
|---------|------|------|
| `BundleDataMgr` | `bundle_data_mgr.cpp` | 数据管理内部实现 |
| `BaseBundleInstaller` | `base_bundle_installer.cpp` | 安装逻辑实现 |
| `InstalldClient` | `installd_client.cpp` | IPC 客户端实现 |

### 2.3 私有实现

以下类为私有实现，外部不应直接访问：

| 类 | 文件 | 说明 |
|------|------|------|
| `BundleMgrService` | `bundle_mgr_service.cpp` | 服务主类 |
| `BundleMgrHostImpl` | `bundle_mgr_host_impl.cpp` | IPC Host 实现 |

---

## 3. 资源生命周期

### 3.1 Bundle 生命周期

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  安装中   │ → │   安装完成 │ → │   启用中  │ → │   更新中  │ → │   卸载中  │
│Installing│    │ Installed│    │ Enabled  │    │ Updating │    │Uninstalling│
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### 3.2 服务组件生命周期

| 组件 | 创建时机 | 销毁时机 | Owner |
|------|----------|----------|-------|
| `BundleDataMgr` | Init() | ~BundleMgrService() | BundleMgrService |
| `BundleInstaller` | Init() | ~BundleMgrService() | BundleMgrService |
| `BundleUserMgr` | Init() | ~BundleMgrService() | BundleMgrService |
| `RdbStore` | Init() | Service Stop | BundleDataMgr |

### 3.3 内存管理

**BundleInfo 存储**:
- 热数据：`BundleDataMgr` 内存缓存
- 冷数据：RDB 持久化存储

**缓存策略**:
```cpp
// bundle_data_mgr.cpp
class BundleDataMgr {
private:
    // 内存缓存
    std::map<std::string, InnerBundleInfo> bundleInfos_;
    
    // 持久化
    std::unique_ptr<IBundleDataStorage> dataStorage_;
    
    // 缓存清理
    void CleanBundleCache(const std::string& bundleName);
};
```

---

## 4. Owner 关系

### 4.1 Bundle 数据 Owner

```
InnerBundleInfo
  │
  ├── 内存 Owner: BundleDataMgr
  │     │
  │     └── 热数据缓存
  │
  ├── 持久化 Owner: RDB
  │     │
  │     └── 冷数据存储
  │
  └── 快照 Owner: AbilityManagerService
        │
        └── 应用运行时的 AbilityInfo 快照
```

### 4.2 文件 Owner

| 文件类型 | Owner | 路径 |
|----------|-------|------|
| HAP 文件 | 应用开发者 | `/data/app/` |
| 安装数据 | InstalldService | `/data/bms/install/` |
| 缓存文件 | BundleMgrService | `/data/bms/cache/` |

### 4.3 资源 Owner

| 资源类型 | Owner | 说明 |
|----------|-------|------|
| Bundle 记录 | BundleDataMgr | 数据库记录 |
| 权限配置 | AccessToken | 权限表 |
| 签名证书 | AppVerify | 证书管理 |

---

## 5. 线程模型

### 5.1 主线程

- SA 401 主线程处理 IPC 请求
- `BundleMgrHostImpl` 在主线程执行

### 5.2 事件处理线程

- `BundleEventHandler` 处理异步事件
- 安装完成回调、状态变更通知

### 5.3 工作线程

- 文件操作通过 InstalldService 在特权进程执行
- 签名验证可能在独立线程

### 5.4 线程安全

**锁使用**:
```cpp
// bundle_data_mgr.cpp
class BundleDataMgr {
private:
    mutable std::shared_mutex bundleInfoMutex_;  // 保护 bundleInfos_
    
public:
    ErrCode GetBundleInfo(...) {
        std::shared_lock<std::shared_mutex> lock(bundleInfoMutex_);  // 读锁
        // 查询操作
    }
    
    ErrCode AddInnerBundleInfo(...) {
        std::unique_lock<std::shared_mutex> lock(bundleInfoMutex_);  // 写锁
        // 修改操作
    }
};
```

---

## 6. 错误处理

### 6.1 错误码定义

**文件**: `interfaces/kits/native/inner_api/appexecfwk_errors.h`

| 错误码范围 | 模块 |
|------------|------|
| 0x0001-0x00FF | 通用错误 |
| 0x0100-0x01FF | 安装错误 |
| 0x0200-0x02FF | 数据库错误 |
| 0x0300-0x03FF | 签名错误 |

### 6.2 异常处理

```cpp
// 使用 SCOPE_GUARD 进行资源清理
#include "scope_guard.h"

ErrCode BundleMgrService::Install(...) {
    auto guard = MAKE_SCOPE_GUARD([&]() {
        // 失败时清理
        installdClient_->RemoveDir(installPath);
    });
    
    // 安装逻辑
    // ...
    
    guard.Dismiss();  // 成功，保留资源
    return ERR_OK;
}
```

---

## 7. 扩展点

### 7.1 Feature 开关

通过 `appexecfwk.gni` 控制功能开关：

| 开关 | 头文件 | 条件编译 |
|------|--------|----------|
| `BUNDLE_FRAMEWORK_FREE_INSTALL` | `free_install/bundle_connect_ability_mgr.h` | `#ifdef BUNDLE_FRAMEWORK_FREE_INSTALL` |
| `BUNDLE_FRAMEWORK_OVERLAY_INSTALLATION` | `bundle_overlay_manager.h` | `#ifdef BUNDLE_FRAMEWORK_OVERLAY_INSTALLATION` |
| `BUNDLE_FRAMEWORK_QUICK_FIX` | `quick_fix_mgr.h` | `#ifdef BUNDLE_FRAMEWORK_QUICK_FIX` |

### 7.2 插件机制

| 插件类型 | 接口 | 文件 |
|----------|------|------|
| 安装扩展 | `IInstallExt` | `plugin/` |
| 数据存储 | `IBundleDataStorage` | `rdb/` |
| 状态回调 | `IBundleStatusCallback` | `ipc/` |

---

## 8. 延伸阅读

- [02_Architecture](02_Architecture.md) - 架构说明
- [03_CodeMap](03_CodeMap.md) - 代码地图
- [07_Build](07_Build.md) - 构建与产物
- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析
