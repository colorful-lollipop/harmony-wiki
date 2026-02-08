# 内部 API

## 8.1 概述

内部 API（InnerAPI）面向 OpenHarmony 系统内部模块使用，不对第三方应用开放。内部 API 提供了更底层的操作能力和更灵活的编程接口，适用于系统应用和系统服务开发场景。

### 8.1.1 内部 API 分类

| 分类 | 说明 | 典型使用方 |
|------|------|------------|
| InnerKit | 对内 Native 接口 | 系统应用、系统服务 |
| InnerAPI | 内部编程接口 | 备份恢复相关模块 |

## 8.2 InnerKit Native 接口

### 8.2.1 FileShare Native

**头文件**：`interfaces/innerkits/native/file_share/include/file_share.h`

**库文件**：`libfileshare_native.so`

**主要功能**：

| 功能 | 函数 | 说明 |
|------|------|------|
| URI 授权 | `GrantUriPermission()` | 授予 URI 访问权限 |
| 权限持久化 | `PersistPermission()` | 持久化权限信息 |
| 权限撤销 | `RevokePermission()` | 撤销已授予的权限 |
| 权限激活 | `ActivatePermission()` | 激活权限 |
| 权限停用 | `DeactivatePermission()` | 停用权限 |
| 权限检查 | `CheckPersistentPermission()` | 检查持久化权限 |
| 路径权限检查 | `CheckPathPermission()` | 检查路径权限 |

### 8.2.2 FileURI Native

**头文件**：`interfaces/innerkits/native/file_uri/include/file_uri.h`

**库文件**：`libfileuri_native.so`

**主要功能**：

| 功能 | 函数 | 说明 |
|------|------|------|
| URI 构造 | `FileUri()` | 创建 URI 对象 |
| 路径转换 | `GetPathFromUri()` | 从 URI 获取路径 |
| 路径获取 | `GetUriFromPath()` | 从路径获取 URI |
| URI 标准化 | `Normalize()` | 标准化 URI |
| 远程判断 | `IsRemoteUri()` | 判断是否为远程 URI |

### 8.2.3 RemoteFileShare Native

**头文件**：`interfaces/innerkits/native/remote_file_share/include/remote_file_share.h`

**库文件**：`libremote_file_share_native.so`

**主要功能**：

| 功能 | 函数 | 说明 |
|------|------|------|
| 远程 URI 授权 | `GrantRemoteUriPermission()` | 授予远程 URI 权限 |
| 远程权限管理 | `ManageRemotePermission()` | 管理远程权限 |

### 8.2.4 SandboxHelper Native

**头文件**：`interfaces/common/include/sandbox_helper.h`

**库文件**：`libsandbox_helper_native.so`

**主要功能**：

| 功能 | 函数 | 说明 |
|------|------|------|
| URI 转换 | `UriToSandboxPath()` | 将 URI 转换为沙箱路径 |
| 路径转换 | `SandboxPathToUri()` | 将沙箱路径转换为 URI |
| 沙箱映射 | `GetSandboxPath()` | 获取文件在沙箱中的映射路径 |

## 8.3 Backup Kit Inner API

### 8.3.1 模块概述

**头文件目录**：`interfaces/inner_api/native/backup_kit_inner/`

**库文件**：`libbackup_kit_inner.so`

Backup Kit Inner API 提供备份恢复的底层编程接口，主要供备份服务（Backup SA）和备份扩展使用。

### 8.3.2 头文件列表

| 文件 | 说明 |
|------|------|
| `backup_kit_inner.h` | 主头文件，导出主要接口 |
| `impl/b_file_info.h` | 文件信息结构 |
| `impl/b_incremental_backup_session.h` | 增量备份会话 |
| `impl/b_incremental_data.h` | 增量数据 |
| `impl/b_incremental_restore_session.h` | 增量恢复会话 |
| `impl/b_session_backup.h` | 备份会话 |
| `impl/b_session_restore.h` | 恢复会话 |
| `impl/b_session_restore_async.h` | 异步恢复会话 |
| `impl/service_client.h` | 服务客户端 |
| `impl/service_reverse.h` | 服务反向调用 |

### 8.3.3 主要类

**BFileInfo 文件信息**：

```cpp
struct BFileInfo {
    std::string fileName;       // 文件名
    std::string filePath;       // 文件路径
    std::string relativePath;   // 相对路径
    uint64_t fileSize;          // 文件大小
    uint64_t lastModify;        // 最后修改时间
};
```

**SessionBackup 备份会话**：

```cpp
class SessionBackup {
public:
    // 初始化
    ErrCode Init();
    
    // 追加待备份应用
    ErrCode AppendBundles(const std::vector<BundleName>& bundleNames);
    
    // 发布文件
    ErrCode PublishFile(const BFileInfo& fileInfo, int fd);
    
    // 完成
    ErrCode Finish();
    
    // 释放
    ErrCode Release();
};
```

**SessionRestore 恢复会话**：

```cpp
class SessionRestore {
public:
    // 初始化
    ErrCode Init(const std::string& backupPath);
    
    // 追加待恢复应用
    ErrCode AppendBundles(const std::vector<BundleName>& bundleNames);
    
    // 开始恢复
    ErrCode Restore();
    
    // 释放
    ErrCode Release();
};
```

### 8.3.4 增量备份 API

**IncrementalBackupSession**：

```cpp
class IncrementalBackupSession {
public:
    // 初始化增量备份
    ErrCode Init();
    
    // 获取增量数据
    ErrCode GetIncrementalData(
        const std::vector<BIncrementalData>& bundleNames,
        std::vector<BIncrementalFileInfo>& result
    );
    
    // 发布增量文件
    ErrCode PublishIncrementalFile(
        const BFileInfo& fileInfo,
        int fd
    );
    
    // 完成增量备份
    ErrCode Finish();
};
```

### 8.3.5 服务客户端

**ServiceClient**：

```cpp
class ServiceClient {
public:
    // 获取单例
    static ServiceClient& GetInstance();
    
    // 连接服务
    ErrCode Connect();
    
    // 断开连接
    void Disconnect();
    
    // 获取服务代理
    sptr<IService> GetServiceProxy();
    
    // 检查服务状态
    bool IsServiceReady();
};
```

## 8.4 稳定性标注

### 8.4.1 接口稳定性分类

| 标注 | 说明 | 使用建议 |
|------|------|----------|
| **SysApi** | 系统 API，稳定 | 可直接使用 |
| **InnerApi** | 内部 API，可能变化 | 系统模块使用 |
| **TestApi** | 测试 API，仅测试使用 | 禁止生产使用 |

### 8.4.2 头文件包含层级

**稳定接口**（可直接使用）：

```cpp
// 位于 interfaces/kits/ 目录
#include <file_uri.h>
#include <file_share.h>
```

**内部接口**（系统模块使用）：

```cpp
// 位于 interfaces/innerkits/ 目录
#include <file_uri.h>
#include <file_share.h>
#include <sandbox_helper.h>
```

**内部内部接口**（谨慎使用）：

```cpp
// 位于 interfaces/inner_api/ 目录
#include <backup_kit_inner.h>
```

### 8.4.3 命名规范

| 前缀 | 稳定性 | 说明 |
|------|--------|------|
| `OH_` | Stable | NDK 标准接口 |
| 无前缀 | Stable | JS N-API 接口 |
| `B` | Inner | 内部实现类 |
| `I` | Interface | 接口类 |

## 8.5 依赖方向

### 8.5.1 模块依赖图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            依赖关系图                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────┐                                                         │
│   │ Backup Kit   │                                                         │
│   │ Inner API    │                                                         │
│   └──────┬───────┘                                                         │
│          │                                                                  │
│          ├──依赖─▶┌──────────────┐                                         │
│          │         │ backup_idl   │ (IPC 接口定义)                          │
│          │         └──────┬───────┘                                         │
│          │                │                                                  │
│          │                ├──依赖─▶┌──────────────┐                          │
│          │                │         │ backup_sa   │ (SA 实现)               │
│          │                │         └──────────────┘                          │
│          │                │                                                  │
│          │                └──依赖─▶┌──────────────┐                          │
│          │                          │ backup_utils │ (工具库)                │
│          │                          └──────────────┘                          │
│          │                                                                  │
│   ┌──────┴───────┐                                                         │
│   │ FileShare    │                                                         │
│   │ Native       │                                                         │
│   └──────┬───────┘                                                         │
│          │                                                                  │
│          ├──依赖─▶┌──────────────┐                                           │
│          │         │ fileuri     │                                           │
│          │         │ native      │                                           │
│          │         └──────────────┘                                           │
│          │                                                                  │
│          ├──依赖─▶┌──────────────┐                                           │
│          │         │ sandbox     │                                           │
│          │         │ helper      │                                           │
│          │         └──────────────┘                                           │
│          │                                                                  │
│          └──依赖─▶┌──────────────┐                                           │
│                    │ access      │                                           │
│                    │ token       │                                           │
│                    └──────────────┘                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 8.6 相关文档

| 文档 | 说明 |
|------|------|
| [JS N-API 接口](10_NAPI_JS.md) | 对外 JS 接口 |
| [NDK 接口](11_NAPI_NDK.md) | 对外 NDK 接口 |
| [备份服务 SA](02_Service_SA.md) | 服务层如何使用 InnerAPI |
| [工具库](03_Utils.md) | 工具库详情 |
| [GN 构建配置](04_GN_Build.md) | InnerAPI 构建配置 |
| [安全评审](05_Security_Review.md) | InnerAPI 安全风险 |
