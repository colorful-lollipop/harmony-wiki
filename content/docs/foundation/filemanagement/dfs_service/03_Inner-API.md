# 内部 Inner API

## 文档信息

| 项目 | 内容 |
|------|------|
| 目标读者 | 系统开发者、服务开发者 |
| 目的 | 理解系统服务间的内部接口 |
| 约束 | 仅供系统服务使用，外部应用不应直接调用 |
| 代码证据 | `interfaces/inner_api/native/` |

## Inner API 概述

Inner API 是供 OpenHarmony 系统服务之间调用的内部接口，定义在 `interfaces/inner_api/native/` 目录下。这些接口通过框架层的 ServiceProxy 进行 IPC 调用，实现跨进程通信。

### Inner Kit 列表

| Kit 名称 | 头文件目录 | 命名空间 | 职责 |
|----------|------------|----------|------|
| CloudSync Kit | `cloudsync_kit_inner/` | `OHOS::FileManagement::CloudSync` | 云同步服务间调用 |
| CloudDiskService Kit | `clouddiskservice_kit_inner/` | `OHOS::FileManagement::CloudDiskService` | 云盘服务间调用 |
| CloudFile Kit | `cloud_file_kit_inner/` | `OHOS::FileManagement::CloudFile` | 云文件工具间调用 |
| CloudDaemon Kit | `cloud_daemon_kit_inner/` | `OHOS::FileManagement::CloudFile` | 云守护进程间调用 |

---

## CloudSync Kit Inner

### 头文件列表

| 头文件 | 职责 |
|--------|------|
| `cloud_sync_manager.h` | 云同步管理器接口 |
| `cloud_sync_callback.h` | 同步回调接口 |
| `cloud_sync_callback_info.h` | 回调信息结构 |
| `i_cloud_sync_callback.h` | 同步回调定义 |
| `cloud_sync_asset_manager.h` | 资源同步管理器 |
| `svc_death_recipient.h` | 服务死亡接收者 |
| `cloud_sync_constants.h` | 常量定义 |
| `cloud_sync_common.h` | 公共类型定义 |

### CloudSyncManager 接口

**头文件**：`interfaces/inner_api/native/cloudsync_kit_inner/cloud_sync_manager.h`

**命名空间**：`OHOS::FileManagement::CloudSync`

**说明**：云同步服务的客户端管理器接口，提供单例获取方式和所有同步相关操作。

#### 核心方法清单

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `GetInstance` | void | `CloudSyncManager&` | 获取单例（静态） |
| `RegisterCallback` | `const CallbackInfo &` | `int32_t` | 注册同步回调 |
| `RegisterFileSyncCallback` | `const CallbackInfo &` | `int32_t` | 注册文件同步回调 |
| `UnRegisterCallback` | `const CallbackInfo &` | `int32_t` | 注销同步回调 |
| `UnRegisterFileSyncCallback` | `const CallbackInfo &` | `int32_t` | 注销文件同步回调 |
| `StartSync` | `const std::string &bundleName = ""` | `int32_t` | 启动同步 |
| `StartFileSync` | `const std::string &bundleName = ""` | `int32_t` | 启动文件同步 |
| `StartSync` | `bool forceFlag, const std::shared_ptr<CloudSyncCallback>` | `int32_t` | 强制启动同步 |
| `TriggerSync` | `const std::string &bundleName, const int32_t &userId` | `int32_t` | 触发同步 |
| `StopSync` | `const std::string &bundleName = "", bool forceFlag = false` | `int32_t` | 停止同步 |
| `StopFileSync` | `const std::string &bundleName = "", bool forceFlag = false` | `int32_t` | 停止文件同步 |
| `ResetCursor` | `const std::string &bundleName = ""` | `int32_t` | 重置光标 |
| `ResetCursor` | `bool flag, const std::string &bundleName = ""` | `int32_t` | 带标志重置光标 |
| `ChangeAppSwitch` | `const std::string &accoutId, const std::string &bundleName, bool status` | `int32_t` | 切换应用云同步开关 |
| `NotifyDataChange` | `const std::string &accoutId, const std::string &bundleName` | `int32_t` | 通知数据变更 |
| `NotifyEventChange` | `int32_t userId, const std::string &eventId, const std::string &extraData` | `int32_t` | 通知事件变更 |
| `EnableCloud` | `const std::string &accoutId, const SwitchDataObj &` | `int32_t` | 启用云功能 |
| `DisableCloud` | `const std::string &accoutId` | `int32_t` | 禁用云功能 |
| `Clean` | `const std::string &accountId, const CleanOptions &` | `int32_t` | 清理数据 |
| `OptimizeStorage` | `const OptimizeSpaceOptions &, const std::shared_ptr<CloudOptimizeCallback>` | `int32_t` | 优化存储 |
| `StopOptimizeStorage` | void | `int32_t` | 停止优化存储 |
| `StartDownloadFile` | `const std::string &uri, const std::shared_ptr<CloudDownloadCallback>, int64_t &downloadId` | `int32_t` | 开始下载文件 |
| `StartFileCache` | `const std::vector<std::string> &, int64_t &, int32_t, const std::shared_ptr<CloudDownloadCallback>, int32_t timeout` | `int32_t` | 开始文件缓存 |
| `StopDownloadFile` | `int64_t downloadId, bool needClean` | `int32_t` | 停止下载文件 |
| `StopFileCache` | `int64_t downloadId, bool needClean, int32_t timeout` | `int32_t` | 停止文件缓存 |
| `DownloadThumb` | void | `int32_t` | 下载缩略图 |
| `GetSyncTime` | `int64_t &syncTime, const std::string &bundleName = ""` | `int32_t` | 获取同步时间 |
| `CleanCache` | `const std::string &uri` | `int32_t` | 清理缓存 |
| `CleanFileCache` | `const std::string &uri` | `int32_t` | 清理文件缓存 |
| `CleanGalleryDentryFile` | void | `void` | 清理图库目录文件 |
| `CleanGalleryDentryFile` | `const std::string path` | `void` | 清理指定路径图库目录文件 |
| `BatchCleanFile` | `const std::vector<CleanFileInfo> &, std::vector<std::string> &failCloudId` | `int32_t` | 批量清理文件 |
| `BatchDentryFileInsert` | `const std::vector<DentryFileInfo> &, std::vector<std::string> &failCloudId` | `int32_t` | 批量插入目录文件 |
| `StartDowngrade` | `const std::string &bundleName, const std::shared_ptr<DowngradeDlCallback>` | `int32_t` | 开始降级下载 |
| `StopDowngrade` | `const std::string &bundleName` | `int32_t` | 停止降级下载 |
| `GetCloudFileInfo` | `const std::string &bundleName, CloudFileInfo &` | `int32_t` | 获取云文件信息 |
| `GetHistoryVersionList` | `const std::string &uri, const int32_t versionNumLimit, std::vector<HistoryVersion> &` | `int32_t` | 获取历史版本列表 |
| `DownloadHistoryVersion` | `const std::string &uri, int64_t &downloadId, const uint64_t versionId, const std::shared_ptr<CloudDownloadCallback>, std::string &versionUri` | `int32_t` | 下载历史版本 |
| `ReplaceFileWithHistoryVersion` | `const std::string &uri, const std::string &versionUri` | `int32_t` | 用历史版本替换文件 |
| `IsFileConflict` | `const std::string &uri, bool &isConflict` | `int32_t` | 检查文件冲突 |
| `ClearFileConflict` | `const std::string &uri` | `int32_t` | 清除文件冲突 |
| `GetBundlesLocalFilePresentStatus` | `const std::vector<std::string> &bundleNames, std::vector<LocalFilePresentStatus> &` | `int32_t` | 获取本地文件存在状态 |
| `IsFinishPull` | `bool &finishFlag` | `int32_t` | 检查是否完成拉取 |
| `GetDentryFileOccupy` | `int64_t &occupyNum` | `int32_t` | 获取目录文件占用 |

### 框架层实现

**位置**：`frameworks/native/cloudsync_kit_inner/`

| 实现类 | 头文件 | 职责 |
|--------|--------|------|
| `CloudSyncManagerImpl` | `cloud_sync_manager_impl.h` | 管理器实现 |
| `ServiceProxy` | `service_proxy.h` | 服务代理 |
| `CloudSyncCallbackClient` | `cloud_sync_callback_client.h` | 回调客户端 |
| `CloudSyncAssetManagerImpl` | `cloud_sync_asset_manager_impl.h` | 资源管理实现 |

---

## CloudDiskService Kit Inner

### 头文件列表

| 头文件 | 职责 |
|--------|------|
| `cloud_disk_common.h` | 公共类型定义 |
| `cloud_disk_service_callback.h` | 服务回调接口 |
| `cloud_disk_service_manager.h` | 服务管理器接口 |
| `i_cloud_disk_service_callback.h` | 回调定义 |
| `svc_death_recipient.h` | 服务死亡接收者 |

### CloudDiskServiceManager 接口

**头文件**：`interfaces/inner_api/native/clouddiskservice_kit_inner/cloud_disk_service_manager.h`

**命名空间**：`OHOS::FileManagement::CloudDiskService`

**说明**：云盘服务的客户端管理器接口。

#### 方法清单

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `GetInstance` | void | `CloudDiskServiceManager&` | 获取单例（静态） |
| `RegisterSyncFolderChanges` | `const std::string &syncFolder, const std::shared_ptr<CloudDiskServiceCallback>` | `int32_t` | 注册同步文件夹变更 |
| `UnregisterSyncFolderChanges` | `const std::string &syncFolder` | `int32_t` | 注销同步文件夹变更 |
| `GetSyncFolderChanges` | `const std::string &syncFolder, uint64_t count, uint64_t startUsn, ChangesResult &` | `int32_t` | 获取同步文件夹变更 |
| `SetFileSyncStates` | `const std::string &syncFolder, const std::vector<FileSyncState> &, std::vector<FailedList> &` | `int32_t` | 设置文件同步状态 |
| `GetFileSyncStates` | `const std::string &syncFolder, const std::vector<std::string> &, std::vector<ResultList> &` | `int32_t` | 获取文件同步状态 |
| `RegisterSyncFolder` | `int32_t userId, const std::string &bundleName, const std::string &path` | `int32_t` | 注册同步文件夹 |
| `UnregisterSyncFolder` | `int32_t userId, const std::string &bundleName, const std::string &path` | `int32_t` | 注销同步文件夹 |
| `UnregisterForSa` | `const std::string &path` | `int32_t` | 注销 SA |

### 框架层实现

**位置**：`frameworks/native/clouddiskservice_kit_inner/`

| 实现类 | 头文件 | 职责 |
|--------|--------|------|
| `CloudDiskServiceManagerImpl` | `cloud_disk_service_manager_impl.h` | 管理器实现 |
| `ServiceProxy` | `service_proxy.h` | 服务代理 |

---

## CloudFile Kit Inner

### 头文件列表

| 头文件 | 职责 |
|--------|------|
| `cloud_file_kit.h` | 云文件工具主接口 |
| `data_sync_manager.h` | 数据同步管理器 |
| `cloud_sync_helper.h` | 同步辅助工具 |
| `cloud_database.h` | 云数据库接口 |
| `cloud_assets_downloader.h` | 云资源下载器 |
| `cloud_info.h` | 云信息 |
| `visibility.h` | 可见性声明 |

### CloudFileKit 接口

**头文件**：`interfaces/inner_api/native/cloud_file_kit_inner/cloud_file_kit.h`

**命名空间**：`OHOS::FileManagement::CloudFile`

**说明**：云文件工具的主接口类，提供云用户信息、空间信息、应用开关等查询功能。

#### 方法清单

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `GetInstance` | void | `static CloudFileKit*` | 获取单例（静态） |
| `RegisterCloudInstance` | `CloudFileKit *instance` | `static bool` | 注册云实例（静态） |
| `GetCloudUserInfo` | `const std::string &bundleName, const int32_t userId, CloudUserInfo &` | `int32_t` | 获取云用户信息 |
| `GetSpaceInfo` | `const int32_t userId, const std::string &bundleName` | `std::pair<uint64_t, uint64_t>` | 获取空间信息 |
| `GetAppSwitchStatus` | `const std::string &bundleName, const int32_t userId, bool &switchStatus` | `int32_t` | 获取应用开关状态 |
| `ResolveNotificationEvent` | `const int32_t userId, const std::string &extraData, std::string &appBundleName, std::string &prepareTraceId` | `int32_t` | 解析通知事件 |
| `GetAppConfigParams` | `const int32_t userId, const std::string &bundleName, std::map<std::string, std::string> &` | `int32_t` | 获取应用配置参数 |
| `CleanCloudUserInfo` | `const int32_t userId, const std::string &bundleName` | `int32_t` | 清理云用户信息 |
| `OnUploadAsset` | `const int32_t userId, const std::string &request, std::string &result` | `int32_t` | 上传资源 |
| `GetDataSyncManager` | void | `std::shared_ptr<DataSyncManager>` | 获取数据同步管理器 |
| `GetCloudDatabase` | `const int32_t userId, const std::string &bundleName` | `std::shared_ptr<CloudDatabase>` | 获取云数据库 |
| `GetCloudAssetsDownloader` | `const int32_t userId, const std::string &bundleName` | `std::shared_ptr<CloudAssetsDownloader>` | 获取云资源下载器 |
| `GetCloudSyncHelper` | `const int32_t userId, const std::string &bundleName` | `std::shared_ptr<CloudSyncHelper>` | 获取云同步辅助器 |
| `GetPrepareTraceId` | `const int32_t userId, const std::string &bundleName` | `std::string` | 获取准备追踪 ID |
| `Release` | `int32_t userId` | `void` | 释放资源 |
| `GenerateLocalIds` | `const std::string &bundleName, int count, std::vector<std::string> &ids` | `int32_t` | 生成本地 ID |

---

## CloudDaemon Kit Inner

### 头文件列表

| 头文件 | 职责 |
|--------|------|
| `cloud_daemon_manager.h` | 守护进程管理器 |
| `i_cloud_daemon.h` | 守护进程接口定义 |
| `svc_death_recipient.h` | 服务死亡接收者 |

### CloudDaemonManager 接口

**头文件**：`interfaces/inner_api/native/cloud_daemon_kit_inner/cloud_daemon_manager.h`

**命名空间**：`OHOS::FileManagement::CloudFile`

**说明**：云守护进程的客户端管理器接口。

#### 方法清单

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `GetInstance` | void | `CloudDaemonManager&` | 获取单例（静态） |
| `StartFuse` | `int32_t userId, int32_t devFd, const string &path` | `int32_t` | 启动 FUSE 挂载 |

### 框架层实现

**位置**：`frameworks/native/cloud_daemon_kit_inner/`

| 实现类 | 头文件 | 职责 |
|--------|--------|------|
| `CloudDaemonManagerImpl` | `cloud_daemon_manager_impl.h` | 管理器实现 |
| `CloudDaemonServiceProxy` | `cloud_daemon_service_proxy.h` | 服务代理 |

---

## 稳定性标注

### 稳定接口

以下接口为系统稳定接口，可安全调用：

| 接口 | 稳定性保证 |
|------|------------|
| `CloudSyncManager::GetInstance` | 系统级接口，稳定 |
| `CloudDiskServiceManager::GetInstance` | 系统级接口，稳定 |
| `CloudFileKit::GetInstance` | 系统级接口，稳定 |
| `CloudDaemonManager::GetInstance` | 系统级接口，稳定 |

### 不稳定接口

以下接口为内部接口，可能发生变化：

| 接口 | 变化风险 |
|------|----------|
| 具体回调实现类 | 中，可能因版本更新调整 |
| 批量操作接口 | 中，可能优化参数或返回值 |

---

## 依赖方向

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层（JS/NAPI）                        │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    frameworks/native/                        │
│  ┌───────────────────────────────────────────────────────┐│
│  │  CloudSyncManagerImpl → ServiceProxy → IPC 调用         ││
│  │  CloudDiskServiceManagerImpl → ServiceProxy → IPC     ││
│  │  CloudDaemonManagerImpl → ServiceProxy → IPC         ││
│  └───────────────────────────────────────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    services/（SA 服务）                       │
│  ┌───────────────────────────────────────────────────────┐│
│  │  CloudSyncService (SA 5204)                           ││
│  │  CloudDiskService (SA 5207)                           ││
│  │  CloudDaemon (SA 5205)                                ││
│  │  DistributedFileDaemon (SA 5201)                      ││
│  └───────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

## 相关跳转

- 架构设计：[01_Architecture.md](./01_Architecture.md)
- 对外接口：[02_N-API.md](./02_N-API.md)
- 构建配置：[04_Build.md](./04_Build.md)
