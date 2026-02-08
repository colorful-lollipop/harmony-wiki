# 项目概览

> 本文档描述 `@ohos/storage_service` 存储管理服务的定位、能力边界和运行环境。

## 组件定位

| 属性 | 值 |
|------|-----|
| 组件名称 | `storage_service` |
| 子系统 | `filemanagement` |
| 包名 | `@ohos/storage_service` |
| 版本 | 3.1 |

**一句话描述**：提供外置存储卡挂载管理、文件加解密、磁盘和卷的查询与管理、用户目录管理和空间统计等功能，为系统和应用提供基础的存储查询、管理能力。

**证据来源**：`bundle.json:3-4`

## 核心能力

### SystemCapabilities

| 能力名称 | 说明 |
|---------|------|
| `SystemCapability.FileManagement.StorageService.SpatialStatistics` | 空间统计能力 |
| `SystemCapability.FileManagement.StorageService.Volume` | 卷管理能力 |
| `SystemCapability.FileManagement.StorageService.Encryption` | 加密能力 |

**证据来源**：`bundle.json:15-19`

### Feature 列表

| Feature | 说明 | 默认启用 |
|---------|------|---------|
| `storage_service_fstools` | 文件系统工具 | ✓ |
| `storage_service_graphic` | 图形相关存储服务 | ✓ |
| `storage_service_user_file_sharing` | 用户文件共享 | ✓ |
| `storage_service_user_crypto_manager` | 用户加密管理 | ✓ |
| `storage_service_external_storage_manager` | 外置存储管理 | ✓ |
| `storage_service_storage_statistics_manager` | 存储统计管理 | ✓ |
| `storage_service_crypto_test` | 加密测试 | - |
| `storage_service_external_storage_qos_trans` | 外置存储 QoS | ✓ |
| `storage_service_media_fuse` | Media FUSE | ✓ |
| `storage_service_cloud_fuse` | Cloud FUSE | ✓ |
| `storage_service_enable_fscrypt_data_reliability_option` | fscrypt 数据可靠性 | ✓ |

**证据来源**：`bundle.json:20-32`

## 运行环境

### 适配系统类型

- `small` - 小型设备
- `standard` - 标准设备

### 资源占用

| 资源 | 大小 |
|------|------|
| ROM | 4096 KB |
| RAM | 10240 KB |

**证据来源**：`bundle.json:33-38`

## 外部依赖

| 依赖组件 | 说明 | 用途 |
|---------|------|------|
| `ability_base` | 能力基础库 | Want/Zuri 处理 |
| `ability_runtime` | 能力运行时 | 扩展能力 |
| `access_token` | 访问控制 | 权限校验 |
| `bundle_framework` | 包框架 | 应用信息查询 |
| `crypto_framework` | 加密框架 | 密钥管理 |
| `dfs_service` | 分布式文件服务 | 分布式存储 |
| `hilog` | 日志系统 | 日志输出 |
| `ipc` | IPC 通信 | 跨进程通信 |
| `napi` | N-API 框架 | JS 接口绑定 |
| `os_account` | 账户管理 | 多用户支持 |
| `safwk` | 系统能力框架 | SA 注册 |
| `samgr` | 服务管理 | 服务发现 |
| `media_library` | 媒体库 | 媒体文件统计 |

**证据来源**：`bundle.json:39-88`

## 约束与限制

### 接口约束

> storage_daemon 所有接口仅支持 storage_manager 服务进行调用。

**证据来源**：`README_zh.md:38`

### 权限要求

| 权限名称 | 用途 | 敏感级别 |
|---------|------|---------|
| `ohos.permission.STORAGE_MANAGER` | 存储管理 | system_basic |
| `ohos.permission.STORAGE_MANAGER_CRYPT` | 加密操作 | system_basic |
| `ohos.permission.MOUNT_UNMOUNT_MANAGER` | 挂载/卸载 | system_basic |
| `ohos.permission.MOUNT_FORMAT_MANAGER` | 格式化操作 | system_basic |

**证据来源**：`services/storage_manager/ipc/src/storage_manager_provider.cpp:71-74`

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [appexecfwk_standard](http://gitee.com/openharmony/appexecfwk_standard) | 包管理 |
| [multimedia_medialibrary_standard](https://gitee.com/openharmony/multimedia_medialibrary_standard) | 媒体库服务 |
| [filemanagement_user_file_service](https://gitee.com/openharmony/filemanagement_user_file_service) | 公共文件访问框架 |
| [filemanagement_dfs_service](https://gitee.com/openharmony/filemanagement_dfs_service) | 分布式文件服务 |
| [filemanagement_app_file_service](https://gitee.com/openharmony/filemanagement_app_file_service) | 应用文件服务 |
