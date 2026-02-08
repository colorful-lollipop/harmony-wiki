# User File Service - 内部 API

## 概述

本文档描述供 OpenHarmony 系统内部组件使用的 API，主要包括内部件间接口 (Inner API) 和服务层接口。

**注意**：内部 API 可能随版本变更，不保证稳定性。

---

## 目录

### 1. 文件访问内部 API

| 接口 | 头文件 | 稳定性 |
|------|--------|--------|
| `IFileAccessExtBase` | `file_access_ext_base.h` | 稳定 |
| `FileAccessHelper` | `file_access_helper.h` | 稳定 |
| `FileAccessExtAbility` | `file_access_ext_ability.h` | 半稳定 |
| `FileAccessServiceBase` | `file_access_service_base_stub.h` | 稳定 |

### 2. 云盘管理内部 API

| 接口 | 头文件 | 稳定性 |
|------|--------|--------|
| `CloudDiskManagerKit` | `cloud_disk_manager_kit.h` | 半稳定 |
| `CloudDiskSyncFolderManager` | `cloud_disk_sync_folder_manager.h` | 半稳定 |

---

## IFileAccessExtBase 接口

**头文件**：`interfaces/inner_api/file_access/include/file_access_ext_base.h`

**职责**：文件访问扩展能力的基础接口

### 方法列表

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Access(path)` | string | int32_t | 检查访问权限 |
| `CreateFile(uri, name)` | string, string | int32_t | 创建文件 |
| `Mkdir(parentUri, name)` | string, string | int32_t | 创建目录 |
| `Delete(uri)` | string | int32_t | 删除 |
| `Open(uri, flags)` | string, int32_t | int32_t | 打开文件 |
| `Close(fd)` | int32_t | int32_t | 关闭文件 |
| `Read(fd, buffer)` | int32_t, buffer | int32_t | 读取文件 |
| `Write(fd, buffer)` | int32_t, buffer | int32_t | 写入文件 |
| `Move(srcUri, dstUri)` | string, string | int32_t | 移动 |
| `Rename(uri, newName)` | string, string | int32_t | 重命名 |
| `ListFile(uri)` | string | vector&lt;FileInfo&gt; | 列出文件 |
| `GetFileInfo(uri)` | string | FileInfo | 获取文件信息 |
| `ScanFile(uri, options)` | string, int32_t | vector&lt;FileInfo&gt; | 扫描文件 |

### 使用示例

```cpp
#include "file_access_ext_base.h"

sptr<IFileAccessExtBase> extProxy;
// extProxy 已通过 IPC 获取

int32_t result = extProxy->Access("/storage/emulated/0/Documents");
if (result == 0) {
    // 访问成功
}
```

---

## FileAccessHelper 接口

**头文件**：`interfaces/inner_api/file_access/include/file_access_helper.h`

**职责**：文件访问的 C++ 辅助类

### 关键方法

| 方法 | 说明 |
|------|------|
| `GetRoot()` | 获取根路径列表 |
| `ListFile(uri)` | 列出目录内容 |
| `CreateFile(uri, name)` | 创建文件 |
| `Mkdir(uri, name)` | 创建目录 |
| `Delete(uri)` | 删除 |
| `Move(src, dst)` | 移动 |
| `Rename(uri, name)` | 重命名 |
| `Open(uri, flags)` | 打开 |
| `Access(uri)` | 检查存在 |
| `GetFileInfo(uri)` | 获取信息 |

### 构造函数

```cpp
// 从上下文构造
FileAccessHelper(AbsContext *context);

// 从 Want 构造
FileAccessHelper(AbsContext *context, const Want &want);
```

---

## FileAccessExtAbility 接口

**头文件**：`interfaces/inner_api/file_access/include/file_access_ext_ability.h`

**职责**：文件访问扩展 Ability 基类

### 生命周期

```cpp
// 生命周期方法
void OnStart() override;
void OnStop() override;
void OnReady(const Want &want) override;
void OnCommand(const Want &want, int startId) override;
```

### 关键方法

| 方法 | 说明 |
|------|------|
| `GetFileAccessExtAbilityInfo()` | 获取扩展信息 |
| `ConnectServiceExtensionAbility()` | 连接服务扩展 |
| `DisconnectServiceExtensionAbility()` | 断开连接 |

### 派生类

| 类名 | 用途 |
|------|------|
| `MediaLibraryExtAbility` | 媒体库扩展 |
| `ExternalFileExtAbility` | 外置存储扩展 |

---

## FileAccessServiceBaseStub 接口

**头文件**：`interfaces/inner_api/file_access/include/file_access_service_base_stub.h`

**职责**：FileAccessService 的 IPC 存根接口

### IPC 接口代码

| Code | 方法 | 说明 |
|------|------|------|
| 1 | `ListFile` | 列出文件 |
| 2 | `CreateFile` | 创建文件 |
| 3 | `Delete` | 删除 |
| 4 | `Open` | 打开 |
| 5 | `Move` | 移动 |
| 6 | `Rename` | 重命名 |
| 7 | `Access` | 检查存在 |
| 8 | `GetFileInfo` | 获取信息 |
| 9 | `GetFileInfoFromUri` | 从 URI 获取 |
| 10 | `GetRoots` | 获取根路径 |

### 使用示例

```cpp
#include "file_access_service_base_stub.h"

// 获取 SA 代理
auto saMgr = SystemAbilityManagerClient::GetSystemAbilityManager();
auto remote = saMgr->GetSystemAbility(5010);
sptr<FileAccessServiceBaseStub> proxy = iface_cast<FileAccessServiceBaseStub>(remote);

// 调用 IPC 方法
int32_t result = proxy->ListFile(uri, fileInfos);
```

---

## 云盘管理 API

### CloudDiskManagerKit

**头文件**：`interfaces/inner_api/cloud_disk_kit_inner/include/cloud_disk_manager_kit.h`

**职责**：云盘管理能力套件

#### 主要方法

| 方法 | 说明 |
|------|------|
| `GetCloudDiskSyncFolder()` | 获取同步文件夹 |
| `RegisterNotify()` | 注册变化通知 |
| `UnregisterNotify()` | 取消注册 |

### CloudDiskSyncFolderManager

**头文件**：`interfaces/inner_api/cloud_disk_kit_inner/include/cloud_disk_sync_folder_manager.h`

**职责**：云同步文件夹管理

#### 主要方法

| 方法 | 说明 |
|------|------|
| `Register(syncFolder)` | 注册同步文件夹 |
| `Unregister(path)` | 注销同步文件夹 |
| `Active(path)` | 激活同步 |
| `Deactive(path)` | 去激活 |
| `GetSyncFolders()` | 获取同步文件夹列表 |
| `UpdateDisplayName(path, name)` | 更新显示名称 |

---

## 观察者 API

### IFileAccessObserver 接口

**头文件**：`interfaces/inner_api/file_access/include/file_access_observer_common.h`

**职责**：文件变化观察者回调

```cpp
class IFileAccessObserver : public IRemoteBroker {
public:
    virtual int32_t OnChange(const NotifyType &type, const Uri &uri) = 0;
};
```

### NotifyType 枚举

| 枚举值 | 说明 |
|--------|------|
| `BUF_MODIFIED` | 内容修改 |
| `CLOSE` | 文件关闭 |
| `MOVE_SELF` | 自身移动 |
| `MOVE` | 移动到 |
| `DELETE` | 删除 |
| `CREATE` | 创建 |

### 注册/注销观察者

```cpp
// FileAccessServiceBaseStub 方法
int32_t RegisterNotify(const Uri &uri, bool notifyForDescendants,
                       const sptr<IFileAccessObserver> &observer);
int32_t UnregisterNotify(const Uri &uri,
                         const sptr<IFileAccessObserver> &observer);
```

---

## URI 工具

### UriExt 类

**头文件**：`interfaces/inner_api/file_access/include/uri_ext.h`

**职责**：URI 解析和构造

```cpp
class UriExt {
public:
    explicit UriExt(const std::string &uri);
    std::string GetPath() const;
    std::string GetName() const;
    std::string GetScheme() const;
    int GetUserId() const;
    int GetBundleId() const;
};
```

---

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                      内部 API 依赖关系                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   interfaces/inner_api/file_access/                            │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  file_access_ext_base.h (基础接口)                      │   │
│   └─────────────────────────┬───────────────────────────────┘   │
│                             │                                   │
│             ┌───────────────┴───────────────┐                   │
│             ▼                               ▼                   │
│   ┌─────────────────────┐       ┌─────────────────────┐         │
│   │ file_access_helper.h│       │ file_access_ext_    │         │
│   │ (辅助类)             │       │ ability.h (扩展)     │         │
│   └─────────────────────┘       └─────────────────────┘         │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ file_access_service_base_stub.h (SA 存根)              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                             │                                   │
│                             ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ services/native/file_access_service/                    │   │
│   │ file_access_service.cpp (服务实现)                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 稳定性等级说明

| 等级 | 说明 | 使用建议 |
|------|------|----------|
| **稳定** | API 已冻结，变更需评审 | 可直接使用 |
| **半稳定** | 核心稳定，部分细节可能变更 | 推荐使用，注意版本兼容性 |
| **不稳定** | 实验性接口，随时可能变更 | 谨慎使用 |

---

## 版本兼容性

| API 版本 | 兼容性 | 说明 |
|----------|--------|------|
| 3.0 | ✅ | 初始版本 |
| 3.1 | ✅ | 向前兼容 |
| 4.0 | ⏳ | 待验证 |
