# 架构设计

## 文档信息

| 项目 | 内容 |
|------|------|
| 目标读者 | 系统开发者、架构师 |
| 目的 | 理解分布式文件服务的整体架构 |
| 前置知识 | OpenHarmony SA 框架、IPC 通信 |

## 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层（JS/ArkTS）                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ file.cloudSync │  │ cloudSyncManager │  │ ANI 接口     │           │
│  │  (N-API)      │  │  (N-API)      │  │  (新一代)     │           │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘           │
├─────────┼──────────────────┼──────────────────┼──────────────────┤
│         │                  │                  │                  │
│         ▼                  ▼                  ▼                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                   NDK C 接口层                               │  │
│  │               (interfaces/kits/ndk/)                        │  │
│  └────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ CloudDaemon  │  │ CloudSyncService │ │ DistributedFile │        │
│  │  (SA 5205)   │  │  (SA 5204)    │  │ Daemon (SA 5201)│         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘           │
│         │                  │                  │                  │
│         ▼                  ▼                  ▼                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                   框架层                                     │  │
│  │              (frameworks/native/)                           │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │              Inner API（系统服务间调用）               │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                   软总线层（SoftBus）                       │  │
│  │            设备发现、会话管理、数据传输                      │  │
│  └────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                   内核层（hmdfs）                           │  │
│  │              分布式文件系统（堆叠式）                        │  │
│  └────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## 系统服务（SA）列表

| SA ID | 服务名 | 进程名 | 启动方式 | 职责 |
|-------|--------|--------|----------|------|
| 5201 | DistributedFileDaemon | distributedfiledaemon | 设备上线触发 | 分布式文件、设备连接、挂载管理 |
| 5204 | CloudSyncService | cloudfileservice | WiFi/充电/屏幕事件触发 | 云端数据同步、周期任务 |
| 5205 | CloudDaemon | cloudfiledaemon | 随系统启动 | 云文件 FUSE 挂载 |
| 5207 | CloudDiskService | clouddiskservice | 用户解锁触发 | 同步文件夹管理 |

### SA 配置文件

| 配置文件 | 服务 | 关键配置 |
|----------|------|----------|
| `services/5201.json` | DistributedFileDaemon | 依赖 SA 4700 |
| `services/5204.json` | CloudSyncService | WiFi 状态、充电状态、屏幕状态触发 |
| `services/5205.json` | CloudDaemon | run-on-create=true |
| `services/5207.json` | CloudDiskService | 用户解锁事件触发 |

### 服务启动依赖关系

```
用户解锁 → CloudDiskService (5207)
    ↓
WiFi 连接/充电/屏幕关闭 → CloudSyncService (5204)
    ↓
设备上线 → DistributedFileDaemon (5201)
    ↓
系统启动 → CloudDaemon (5205)
```

## 目录结构与模块职责

### interfaces/ — 对外接口层

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `kits/ndk/clouddiskmanager/` | NDK C 接口 | `oh_cloud_disk_manager.h` |
| `kits/js/cloudfilesync/` | JS N-API（云同步） | `cloud_sync_napi.cpp` |
| `kits/js/cloudsyncmanager/` | JS N-API（同步管理） | `cloud_sync_manager_napi.cpp` |
| `kits/js/ani/` | ArkTS Native Interface | `cloud_sync_ani.h` |
| `inner_api/native/` | 内部 Native API | `cloud_sync_manager.h` |

### frameworks/ — 框架层

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `native/distributed_file_inner/` | 分布式文件核心框架 | `distributed_file_daemon_manager.h` |
| `native/cloudsync_kit_inner/` | 云同步客户端框架 | `cloud_sync_manager.h` |
| `native/cloud_file_kit_inner/` | 云文件工具框架 | `cloud_file_kit.h` |
| `native/clouddiskservice_kit_inner/` | 云盘服务客户端框架 | `cloud_disk_service_manager.h` |

### services/ — 服务层

| 目录 | 职责 | 关键头文件 |
|------|------|------------|
| `distributedfiledaemon/` | 分布式文件守护进程 | `daemon.h`、`i_daemon.h` |
| `cloudsyncservice/` | 云同步服务 | `cloud_sync_service.h` |
| `cloudfiledaemon/` | 云文件守护进程 | `cloud_daemon.h` |
| `clouddiskservice/` | 云盘服务 | `cloud_disk_service.h` |

## 线程模型

### 线程模型概述

本服务采用多线程并发模型，主要包括三类线程：

```
┌─────────────────────────────────────────────────────────────────────┐
│                         线程模型架构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────┐                                                │
│  │   SA 主线程     │  ← SA 消息循环、IPC 请求分发                     │
│  │  (主线程)       │                                                │
│  └────────┬────────┘                                                │
│           │ IPC 请求分发                                              │
│           ▼                                                          │
│  ┌─────────────────┐                                                │
│  │  IPC 线程池      │  ← 并发处理跨进程请求                           │
│  │ MAX: 32 线程    │    `IPCSkeleton::SetMaxWorkThreadNum(32)`       │
│  └────────┬────────┘                                                │
│           │                                                         │
│           ▼                                                         │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐│
│  │  FFRT 工作线程   │    │  FFRT 工作线程   │    │  FFRT 工作线程   ││
│  │  (QoS 优先级)    │    │  (QoS 优先级)    │    │  (QoS 优先级)    ││
│  └─────────────────┘    └─────────────────┘    └─────────────────┘│
│                                                                     │
│  典型场景：                                                         │
│  - 文件传输线程（高优先级）                                           │
│  - 数据同步线程（中优先级）                                           │
│  - 监控任务线程（低优先级）                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 线程类型详解

#### 1. SA 主线程

**职责**：
- SA 消息循环初始化
- IPC 请求分发
- 服务生命周期管理（OnStart/OnStop）

**代码证据**：

```cpp
// services/distributedfiledaemon/src/ipc/daemon.cpp:136-159
void Daemon::OnStart()
{
    LOGI("Begin to start service");
    // ...
    PublishSA();
    StartEventHandler();
    // ...
    state_ = ServiceRunningState::STATE_RUNNING;
    LOGI("Start service successfully");
}
```

#### 2. IPC 线程池

**配置参数**：
- 最大线程数：`MAX_IPC_THREAD_NUM = 32`
- 配置位置：`services/distributedfiledaemon/src/ipc/daemon.cpp:94,151`

**代码证据**：

```cpp
// services/distributedfiledaemon/src/ipc/daemon.cpp:94
constexpr int32_t MAX_IPC_THREAD_NUM = 32;

// services/distributedfiledaemon/src/ipc/daemon.cpp:151
IPCSkeleton::SetMaxWorkThreadNum(MAX_IPC_THREAD_NUM);
```

**职责**：
- 并发处理跨进程 IPC 请求
- 权限校验前置
- 请求路由分发

#### 3. FFRT 工作线程池

**FFRT（Fair Future Runtime）** 是 OpenHarmony 的轻量级异步任务框架。

**QoS 优先级**：

| QoS 级别 | 用途 | 示例场景 |
|----------|------|----------|
| `qos_user_interactive` | 用户交互 | 文件操作响应 |
| `qos_default` | 默认任务 | 常规同步 |
| `qos_utility` | utility 任务 | 数据处理 |
| `qos_background` | 后台任务 | 监控、日志 |

**代码证据**（FFRT 使用示例）：

```cpp
// services/cloudfiledaemon/src/fuse_manager/fuse_manager.cpp:690
ffrt::submit(task, {}, {}, ffrt::task_attr().qos(ffrt_qos_background));

// services/cloudfiledaemon/src/cloud_disk/io_message_listener.cpp:362
ffrt::submit([this] { ReadAndReportIoMessage(); }, {}, {}, ffrt::task_attr().qos(ffrt::qos_background));

// services/clouddiskservice/sync_folder/src/cloud_disk_service_logfile.cpp:660
handle_ = ffrt_timer_start(ffrt_qos_default, CHANGE_DATA_TIMEOUT_MS, nullptr, CallBack, true);
```

**线程安全原语**：

```cpp
// services/cloudfiledaemon/src/fuse_manager/fuse_manager.cpp:184-186
ffrt::shared_mutex sessionLock;      // 共享 mutex（读多写少）
ffrt::mutex readLock;                 // 独占 mutex
ffrt::condition_variable cond;        // 条件变量
```

### 各服务线程模型

| 服务 | 主线程 | IPC 线程 | FFRT 线程 | 特点 |
|------|--------|----------|-----------|------|
| DistributedFileDaemon (5201) | ✅ | ✅ (32) | ✅ | 设备连接、文件传输 |
| CloudSyncService (5204) | ✅ | ✅ | ✅ | 云端同步、周期任务 |
| CloudDaemon (5205) | ✅ | ✅ | ✅ | FUSE 挂载、文件服务 |
| CloudDiskService (5207) | ✅ | ✅ | ✅ | 同步文件夹管理 |

### 线程间通信

```
┌─────────────────────────────────────────────────────────────────────┐
│                       线程间通信机制                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   IPC 线程 ──────▶ MessageParcel ──────▶ SA 主线程                 │
│                        (序列化/反序列化)                              │
│                                                                     │
│   SA 主线程 ─────▶ FFRT submit ──────▶ 工作线程                     │
│                        (任务提交)                                     │
│                                                                     │
│   FFRT 线程 ─────▶ callback ──────▶ SA 主线程                       │
│                        (异步结果)                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 线程安全最佳实践

| 场景 | 使用的原语 | 示例代码位置 |
|------|-----------|--------------|
| 读多写少 | `ffrt::shared_mutex` | `fuse_manager.cpp:184` |
| 独占访问 | `ffrt::mutex` | `fuse_manager.cpp:185` |
| 条件等待 | `ffrt::condition_variable` | `fuse_manager.cpp:155` |
| 异步任务 | `ffrt::submit()` | `cloud_disk_service.cpp` |
| 定时任务 | `ffrt_timer_start()` | `cloud_disk_service_logfile.cpp:660` |

## IPC 通信机制

### IPC 框架

- 使用 OpenHarmony IPC 框架
- `MessageParcel` 进行序列化/反序列化
- `MessageOption` 定义调用方式（同步/异步）

### 关键 IPC 存根

| 服务 | 存根类 | 关键文件 |
|------|--------|----------|
| DistributedFileDaemon | DaemonStub | `daemon_stub.cpp` |
| CloudDaemon | CloudDaemonStub | `cloud_daemon_stub.cpp` |
| CloudSyncService | CloudSyncServiceStub | `cloud_sync_service_stub.cpp`（TODO 需确认） |

### 权限校验入口

```cpp
// utils/system/src/dfsu_access_token_helper.cpp
bool DfsuAccessTokenHelper::CheckCallerPermission(const std::string &permissionName)
```

## 数据流

### 文件读取数据流

```
应用（JS/NAPI）
    ↓
CloudDaemon（FUSE 挂载）
    ↓
本地文件 / 远端文件 / 云端文件
```

### 云同步数据流

```
应用（changeAppCloudSwitch）
    ↓
CloudSyncManager（N-API）
    ↓
CloudSyncService（SA）
    ↓
云端存储（Cloud Adapter）
```

## 相关跳转

- 对外接口：[02_N-API.md](./02_N-API.md)
- 内部接口：[03_Inner-API.md](./03_Inner-API.md)
- 构建配置：[04_Build.md](./04_Build.md)
- 安全评审：[06_Security.md](./06_Security.md)
