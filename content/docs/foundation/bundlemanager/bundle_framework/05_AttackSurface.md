# 攻击面分析

## 概述

本文档系统性地识别和分析了 Bundle Framework 的所有外部输入入口、敏感操作和信任边界，为安全研究员提供攻击面全景视图。

**目标受众**：安全研究员、渗透测试人员、审计人员

**前置知识**：建议先阅读 [02_Architecture](02_Architecture.md) 理解系统架构

---

## 1. 外部输入清单

### 1.1 N-API 接口输入

**位置**: `interfaces/kits/js/`

N-API 是应用层调用 Bundle Framework 的主要入口，接收来自第三方应用的不可信输入。

| 接口 | 输入类型 | 风险等级 | 说明 |
|------|----------|----------|------|
| `install(hapPaths)` | 文件路径数组 | **高** | 直接操作文件系统 |
| `uninstall(bundleName)` | 字符串 | **中** | 包名解析 |
| `getBundleInfo(name)` | 字符串 | 低 | 信息查询 |
| `cleanBundleCache()` | 路径 | **中** | 缓存清理 |
| `getBundleStats()` | UID | 中 | 资源统计 |
| `setHapBundleInstaller()` | 布尔值 | **高** | 安装器设置 |

**代码证据**：
```cpp
// interfaces/kits/js/bundle_manager/src/napi_bundle_manager.cpp
napi_value NapiInstall(napi_env env, napi_callback_info info) {
    // 参数解析入口
    size_t argc = MAX_ARGS;
    napi_value argv[MAX_ARGS];
    napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
    // ...
}
```

### 1.2 IPC 通信输入

| 通道 | 风险等级 | 说明 |
|------|----------|------|
| BundleMgrService (SA 401) | **中** | 用户空间 IPC，Foundation 进程内 |
| InstalldService (SA 511) | **高** | 特权文件操作，独立进程 |

**IPC 接口定义**：
```cpp
// interfaces/inner_api/appexecfwk_core/include/bundlemgr/bundle_mgr_interface.h
class IBundleMgr : public IRemoteBroker {
public:
    // IPC 方法声明
    virtual ErrCode Install(const std::string& hapPath,
                            const InstallParam& installParam,
                            const sptr<IBundleStatusCallback>& statusCallback) = 0;
    // ...
};
```

**代码证据**：
```cpp
// services/bundlemgr/src/ipc/bundle_mgr_host_impl.cpp
ErrCode BundleMgrHostImpl::Install(const std::string& hapPath, ...) {
    // IPC 处理入口
    // BundleMgrService::Install() 实际调用
}
```

### 1.3 文件系统输入

| 操作 | 风险等级 | 说明 |
|------|----------|------|
| HAP 文件解压 | **高** | 路径遍历风险 |
| 目录创建 | **中** | 符号链接攻击 |
| 权限设置 | **中** | 所有权变更 |
| 配置文件读取 | 低 | JSON 解析 |

**代码证据**：
```cpp
// services/bundlemgr/src/installd/installd_service.cpp
ErrCode InstalldService::ExtractFiles(const std::string& srcPath,
                                       const std::string& destPath) {
    // HAP 解压实现
    // zip_file.cpp 中的解压逻辑
}
```

### 1.4 系统能力调用

| SA | 风险等级 | 说明 |
|-----|----------|------|
| SA 401 (BundleMgrService) | **中** | 包管理主服务 |
| SA 511 (InstalldService) | **高** | 特权安装服务 |

**SA 配置证据**：
```json
// sa_profile/401.json
{
    "process": "foundation",
    "systemability": [{
        "name": 401,
        "libpath": "libbms.z.so",
        "run-on-create": true
    }]
}

// sa_profile/511.json
{
    "process": "installs",
    "systemability": [{
        "name": 511,
        "libpath": "libinstalls.z.so",
        "run-on-create": false
    }]
}
```

---

## 2. 敏感操作清单

### 2.1 特权系统调用

以下操作通过 SA 511（InstalldService）执行，具有系统级权限：

| 操作 | 系统调用 | 风险说明 |
|------|----------|----------|
| 文件创建 | `open()`, `creat()` | 路径遍历 |
| 目录操作 | `mkdir()`, `rmdir()` | 符号链接攻击 |
| 权限修改 | `chmod()`, `chown()` | 权限提升 |
| 文件删除 | `unlink()` | 关键文件删除 |
| 符号链接 | `symlink()` | 链接劫持 |

**代码证据**：
```cpp
// services/bundlemgr/src/installd/installd_service.cpp
ErrCode InstalldService::CreateFile(const std::string& path) {
    // 直接调用系统调用
    int fd = open(path.c_str(), O_CREAT | O_WRONLY, S_IRUSR | S_IWUSR);
    // ...
}
```

### 2.2 跨进程通信

| 调用方 | 被调用方 | 操作 |
|--------|----------|------|
| 应用进程 | SA 401 | 安装/卸载/查询 |
| SA 401 | SA 511 | 文件操作 |
| SA 401 | Common Event | 事件发布 |

**IPC 调用链**：
```
应用 → Binder → BundleMgrHostImpl → BundleMgrService
                                              ↓
                                    InstalldClient → Binder
                                              ↓
                                        InstalldService
```

### 2.3 权限敏感操作

| 操作 | 所需权限 | 风险等级 |
|------|----------|----------|
| 安装应用 | `OHOS.permission.INSTALL_BUNDLE` | **高** |
| 卸载应用 | `OHOS.permission.UNINSTALL_BUNDLE` | **高** |
| 查询系统应用 | `GET_BUNDLE_INFO_PRIVILEGED` | **中** |
| 清理缓存 | `CLEAN_BUNDLE_CACHE` | **中** |

**代码证据**：
```cpp
// services/bundlemgr/src/bundle_permission_mgr.cpp
ErrCode BundlePermissionMgr::VerifyCallingPermission(
    const std::string& permission) {
    auto tokenId = IPCSkeleton::GetCallingTokenID();
    return AccessTokenKit::VerifyAccessToken(tokenId, permission);
}
```

---

## 3. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         不可信区域 (Untrusted Zone)                          │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │   第三方应用       │  │   开发者工具      │  │   ADB 命令        │          │
│  │   (不可信输入)     │  │   (HAP 文件)      │  │   (安装命令)      │          │
│  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘          │
└───────────┼─────────────────────┼─────────────────────┼────────────────────┘
            │                     │                     │
            ▼                     ▼                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         边界: 应用层 IPC                                      │
│            (BundleMgrHostImpl 权限校验 + AccessToken)                        │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │  验证点:                                                           │     │
│  │  - IPCSkeleton::GetCallingUid()                                   │     │
│  │  - IPCSkeleton::GetCallingTokenID()                               │     │
│  │  - AccessTokenKit::VerifyAccessToken()                            │     │
│  └────────────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         半可信区域 (Semi-Trusted Zone)                        │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │              BundleMgrService (SA 401)                                │   │
│  │  运行进程: foundation                                                │   │
│  │  - AccessToken 权限校验                                               │   │
│  │  - BundlePermissionMgr                                               │   │
│  │  - 代码签名验证 (appverify)                                           │   │
│  │  - 包格式解析 (bundle_parser)                                         │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
            │
            ▼ IPC (仅 foundation 进程内)
┌─────────────────────────────────────────────────────────────────────────────┐
│                         可信区域 (Trusted Zone)                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │              InstalldService (SA 511)                                 │   │
│  │  运行进程: installs (特权进程)                                        │   │
│  │  - UID 校验 (仅允许 foundation 进程调用)                               │   │
│  │  - 特权文件操作                                                       │   │
│  │  - SELinux 标签管理                                                   │   │
│  │  - 路径规范化验证                                                     │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.1 边界说明

| 边界 | 跨越类型 | 防护措施 |
|------|----------|----------|
| 应用 → SA 401 | 用户→系统 | Binder 身份验证 + AccessToken 权限校验 |
| SA 401 → SA 511 | 同主机进程间 | UID 验证 + SELinux 策略 |

### 3.2 信任假设

1. **Binder 机制可信**：假设 IPC 调用者身份验证可靠
2. **SELinux 策略有效**：假设 SELinux 标签配置正确
3. **签名验证可靠**：假设 appverify 模块签名验证正确

---

## 4. 数据流分析

### 4.1 安装流程数据流

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  应用     │ →  │ N-API    │ →  │BMService │ →  │Installd  │ →  │ 文件系统  │
│ (HAP文件) │    │ install()│    │ (401)    │    │ (511)    │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     ↓               ↓              ↓              ↓
  不可信          参数校验       权限校验        路径规范化
```

### 4.2 关键数据验证点

| 阶段 | 验证内容 | 代码位置 |
|------|----------|----------|
| N-API | 参数类型、数量 | `napi_bundle_manager.cpp` |
| IPC Host | 权限校验 | `bundle_mgr_host_impl.cpp` |
| Service | 签名验证 | `bundle_verify_mgr.cpp` |
| Installd | 路径规范化 | `installd_service.cpp` |

---

## 5. 检查范围说明

### 5.1 已覆盖

| 模块 | 状态 | 说明 |
|------|------|------|
| N-API 接口 | ✅ | 参数校验、边界检查 |
| IPC 通信 | ✅ | 权限校验、UID 验证 |
| 文件操作 | ✅ | 路径规范化、符号链接检测 |
| 权限系统 | ✅ | AccessToken 集成 |
| 签名验证 | ✅ | 代码签名集成 |

### 5.2 未覆盖（超出范围）

| 模块 | 说明 |
|------|------|
| SELinux | 需要单独的安全审计 |
| 文件系统加密 | 依赖存储子系统 |
| 网络安全 | 无网络通信组件 |

---

## 6. 相关文档

- [安全风险评估](06_SecurityReview.md) - 详细风险分析和修复建议
- [02_Architecture](02_Architecture.md) - 系统架构说明
- [03_N-API_Reference](03_N-API_Reference.md) - API 接口文档
