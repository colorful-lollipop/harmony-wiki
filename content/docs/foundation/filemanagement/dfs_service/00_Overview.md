# 项目概览

## 文档信息

| 项目 | 内容 |
|------|------|
| 组件名 | `@ohos/dfs_service` |
| 版本 | 3.1 |
| 子系统 | filemanagement |
| 仓库路径 | `foundation/filemanagement/dfs_service` |

## 项目定位

分布式文件服务（DFS Service）是 OpenHarmony 分布式能力的核心组成部分，提供跨设备的文件访问与同步能力。

### 核心能力

| 能力 | 描述 | 支持情况 |
|------|------|----------|
| 跨设备文件访问 | 符合 POSIX 规范的文件操作接口 | ✅ 完整支持 |
| 分布式文件系统 | 基于 hmdfs 的堆叠式文件系统 | ✅ 完整支持 |
| 云端数据同步 | 云文件与本地文件的自动同步 | ✅ 完整支持 |
| 同步文件夹 | 用户指定的云端同步目录管理 | ✅ 完整支持 |
| 文件版本管理 | 云端文件版本控制与回溯 | ✅ 完整支持 |

### 系统能力声明

```json
// bundle.json 中的系统能力声明
"syscap": [
    "SystemCapability.FileManagement.DistributedFileService.CloudSync.Core",
    "SystemCapability.FileManagement.DistributedFileService.CloudSyncManager"
]
```

## 核心模块

本服务由以下三大核心模块组成：

### 1. distributed_file_daemon

**职责**：分布式文件管理常驻用户态服务。

**核心功能**：
- 设备组网接入管理
- 分布式软总线通信
- 设备间文件传输
- hmdfs 文件系统挂载
- 设备上下线监控

**代码位置**：`services/distributedfiledaemon/`

### 2. distributed_file_service

**职责**：分布式文件访问能力服务，对应用提供分布式扩展能力。

**核心功能**：
- 分布式文件操作接口
- 远程文件读写
- 文件复制与移动

**代码位置**：`services/`（已整合至 `distributedfiledaemon/` 目录，不再单独存在）

### 3. hmdfs（Harmony Distributed File System）

**职责**：分布式文件系统核心模块，面向移动分布式场景的高性能堆叠式文件系统。

**核心特性**：
- 基于内核实现
- 与本地文件系统（如 ext4、f2fs）堆叠
- 支持 64 位文件大小
- 统一的逻辑文件系统视图

**代码位置**：位于内核子系统（`kernel/`），非本仓库

## 功能特性

### 运行环境要求

| 要求项 | 说明 |
|--------|------|
| 操作系统 | OpenHarmony |
| 系统类型 | standard、small |
| ROM 占用 | 2048 KB |
| RAM 占用 | 4096 KB |

### 特性开关

可通过以下宏开关控制功能编译：

| 开关 | 功能 | 默认值 | 说明 |
|------|------|--------|------|
| `dfs_service_feature_enable_cloud_adapter` | 云适配器 | `false` | 云端存储适配层 |
| `dfs_service_feature_enable_cloud_disk` | 云盘功能 | `false` | 云盘同步文件夹 |
| `dfs_service_feature_enable_dist_file_daemon` | 分布式文件守护 | `true` | 分布式文件管理 |
| `dfs_service_feature_enable_distributed_ability` | 分布式能力 | `true` | 分布式扩展能力 |

**证据来源**：`distributedfile.gni:39-42`

```gni
dfs_service_feature_enable_cloud_adapter = false
dfs_service_feature_enable_cloud_disk = false
dfs_service_feature_enable_dist_file_daemon = true
dfs_service_feature_enable_distributed_ability = true
```

## 约束与限制

### 不支持的系统调用

| 系统调用 | 限制说明 | 原因 |
|----------|----------|------|
| `symlink` | 不支持 | 分布式场景下符号链接可能导致循环引用 |
| `mmap` | 仅支持读 | 避免远程写入冲突 |
| `rename` | 仅支持同目录操作 | 跨目录移动涉及分布式一致性保证 |

### 规格限制

| 规格项 | 限制值 | 说明 |
|--------|--------|------|
| 最大目录层级 | 与被堆叠文件系统一致 | ext4、f2fs 等 |
| 最大文件名长度 | 680 B 与被堆叠文件最小值 | f2fs、ext4 均为 255 B |
| 最大单文件大小 | 2^64 B 与被堆叠文件最小值 | ext4 最大 16 TB，f2fs 最大 3.94 TB |

## 相关跳转

- 架构设计：[01_Architecture.md](./01_Architecture.md)
- 对外接口：[02_N-API.md](./02_N-API.md)
- 构建配置：[04_Build.md](./04_Build.md)
