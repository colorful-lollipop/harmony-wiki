# 备份服务（Backup Service Ability）

## 3.1 服务概述

备份服务是应用文件服务的核心组件，以系统能力（System Ability，SA）方式运行在独立进程中。备份服务负责管理整个备份恢复流程，包括会话管理、扩展调度、数据传输、状态通知等核心功能。服务采用单会话模型，同一时刻只处理一个备份或恢复任务。

### 3.1.1 服务基本信息

| 属性 | 值 |
|------|------|
| **SA ID** | `FILEMANAGEMENT_BACKUP_SERVICE_SA_ID`（定义于系统头文件） |
| **进程** | 独立进程（`ccsuitsa`） |
| **运行模式** | 按需启动（`runOnCreate = false`） |
| **安装路径** | `system/lib/${arch}/libbackup_sa.so` |
| **配置文件** | `system/etc/init/backup.cfg`、`system/profile/5203.json` |

### 3.1.2 服务职责

备份服务承担以下核心职责：

**会话管理**：创建、维护、销毁备份或恢复会话。每个会话包含操作类型、目标应用列表、传输配置等状态信息。服务支持完整备份会话、增量备份会话、恢复会话等多种会话类型。

**扩展调度**：管理备份扩展的生命周期，包括扩展的启动、连接、回调处理、销毁等。服务负责协调多个扩展的并发执行，控制并发度以避免资源竞争。

**数据传输**：在应用扩展和目标存储之间传输备份数据。服务提供数据队列机制，支持流式传输以减少内存占用。数据传输过程中负责校验数据完整性。

**状态通知**：向调用方报告备份恢复进度和结果。通过回调机制，调用方可实时获取每个应用的开始、完成、失败等状态事件。

**异常处理**：处理各种异常情况，包括扩展崩溃、连接超时、数据传输错误等。服务支持自动重试和手动取消两种异常处理策略。

## 3.2 服务注册与生命周期

### 3.2.1 SA 注册

备份服务通过 `REGISTER_SYSTEM_ABILITY_BY_ID` 宏注册到系统能力框架：

**注册位置**：`services/backup_sa/src/module_ipc/service.cpp:75`

```cpp
REGISTER_SYSTEM_ABILITY_BY_ID(Service, FILEMANAGEMENT_BACKUP_SERVICE_SA_ID, false);
```

注册参数说明：
- `Service`：服务类名，继承自 `SystemAbility` 和 `ServiceStub`
- `FILEMANAGEMENT_BACKUP_SERVICE_SA_ID`：系统分配的服务 ID
- `false`：表示不随系统启动时立即创建服务实例，服务按需懒加载

### 3.2.2 服务类继承结构

**文件**：`services/backup_sa/include/module_ipc/service.h:79`

```cpp
class Service : public SystemAbility, public ServiceStub, protected NoCopyable
```

服务类继承关系：
- `SystemAbility`：提供 SA 基本能力，包括生命周期管理、发布订阅等
- `ServiceStub`：实现 IPC 接口，接收来自 Proxy 的调用
- `NoCopyable`：禁止拷贝构造和赋值，确保单例模式

### 3.2.3 OnStart 流程

**文件**：`services/backup_sa/src/module_ipc/service.cpp:212-253`

```cpp
void Service::OnStart()
{
    HITRACE_METER_NAME(HITRACE_TAG_FILEMANAGEMENT, __PRETTY_FUNCTION__);
    HILOGI("SA OnStart Begin.");
    
    // 1. 清理残留配置
    ClearDisposalOnSaStart();
    
    // 2. 发布服务到 SA 框架
    SystemAbility::Publish(sptr(this));
    
    // 3. 启动调度器定时器
    OnStartSched();
    
    // 4. 清理处置配置
    DeleteDisConfigFile();
    
    HILOGI("SA OnStart End, res = %{public}d", res);
}
```

OnStart 主要步骤：
1. 检查并清理上次异常退出遗留的配置
2. 将服务实例发布到系统能力管理器，使其他进程可以发现并连接
3. 启动调度器定时器，准备处理备份恢复任务
4. 清理临时配置文件

### 3.2.4 OnStop 流程

**文件**：`services/backup_sa/src/module_ipc/service.cpp:260-270`

```cpp
void Service::OnStop()
{
    HITRACE_METER_NAME(HITRACE_TAG_FILEMANAGEMENT, __PRETTY_FUNCTION__);
    HILOGI("SA OnStop Begin.");
    
    // 恢复内存参数
    int32_t oldMemoryParaSize = BConstants::DEFAULT_VFS_CACHE_PRESSURE;
    
    HILOGI("SA OnStop End.");
}
```

OnStop 负责清理服务资源，包括停止线程池、释放内存等。

### 3.2.5 Dump 能力

**文件**：`services/backup_sa/include/module_ipc/service.h:159`

```cpp
int Dump(int fd, const std::vector<std::u16string> &args) override;
```

服务支持通过 `dumpsys` 命令查询内部状态，用于调试和诊断。

## 3.3 IPC 接口定义

### 3.3.1 接口文件

备份服务的 IPC 接口通过 IDL（Interface Definition Language）定义，位于以下文件：

| 文件 | 说明 |
|------|------|
| `IService.idl` | 服务端接口，46 个 IPC 方法 |
| `IServiceReverse.idl` | 回调接口，30 个回调方法 |
| `IExtension.idl` | 扩展接口，17 个方法 |
| `ServiceType.idl` | 服务类型定义 |
| `ServiceReverseType.idl` | 回调类型定义 |

### 3.3.2 IService.idl 接口方法

| IPC Code | 方法名 | 功能描述 |
|----------|--------|----------|
| 39 | InitRestoreSession | 初始化恢复会话 |
| 1 | InitRestoreSessionWithErrMsg | 初始化恢复会话（带错误消息） |
| 2 | InitBackupSession | 初始化备份会话 |
| 3 | InitBackupSessionWithErrMsg | 初始化备份会话（带错误消息） |
| 4 | Start | 启动备份/恢复任务 |
| 5 | GetLocalCapabilities | 获取本地备份能力（返回 FD） |
| 6 | GetLocalCapabilitiesForBundleInfos | 获取带 Bundle 信息的备份能力 |
| 7 | PublishFile | 发布备份文件 |
| 8 | GetFileHandle | 获取文件句柄（单向调用） |
| 9-10 | AppendBundlesRestoreSessionData | 追加恢复任务 |
| 11-12 | AppendBundlesBackupSession | 追加备份任务 |
| 13 | Finish | 完成追加 |
| 14 | Release | 释放会话 |
| 15 | CancelForResult | 取消操作并返回结果 |
| 16 | GetAppLocalListAndDoIncrementalBackup | 获取增量备份本地列表 |
| 17 | GetIncrementalFileHandle | 获取增量文件句柄 |
| 18 | GetBackupInfo | 获取备份信息 |
| 19 | UpdateTimer | 更新超时定时器 |
| 20 | UpdateSendRate | 更新发送速率 |
| 21-23 | StartExtTimer/StartFwkTimer/StopExtTimer | 定时器控制 |
| 24 | GetLocalCapabilitiesIncremental | 获取增量备份能力 |
| 25-26 | InitIncrementalBackupSession | 初始化增量备份会话 |
| 27-28 | AppendBundlesIncrementalBackupSession | 追加增量备份任务 |
| 29 | PublishIncrementalFile | 发布增量文件 |
| 30 | PublishSAIncrementalFile | 发布 SA 增量文件 |
| 31-32 | AppIncrementalFileReady/AppIncrementalDone | 增量备份回调 |
| 33 | ReportAppProcessInfo | 报告处理信息 |
| 34 | RefreshDataSize | 刷新数据大小 |
| 35-36 | AppDone/AppFileReady | 应用完成回调 |
| 37 | ServiceResultReport | 服务结果报告 |
| 38 | GetBackupDataSize | 获取备份数据大小 |
| 40 | CleanBundleTempDir | 清理临时目录 |
| 41 | HandleExtDisconnect | 处理扩展断开 |
| 42 | GetExtOnRelease | 检查扩展是否在释放 |
| 43-44 | AppFileReadyWithoutFd/AppIncrementalFileReadyWithoutFd | 无 FD 文件就绪 |
| 45 | GetCompatibilityInfo | 获取兼容性信息 |
| 46 | StartCleanData | 启动数据清理 |

### 3.3.3 IServiceReverse.idl 回调接口

回调接口支持四种场景：

**完整备份场景（BackupOnXxx）**：
- `OnBackupStarted`：备份开始
- `OnBackupExtStarted`：扩展备份开始
- `OnBackupExtFinished`：扩展备份完成
- `OnBackupFileReady`：文件就绪
- `OnBackupFinished`：备份完成

**完整恢复场景（RestoreOnXxx）**：
- `OnRestoreStarted`：恢复开始
- `OnRestoreExtStarted`：扩展恢复开始
- `OnRestoreExtFinished`：扩展恢复完成
- `OnRestoreFileReady`：文件就绪
- `OnRestoreFinished`：恢复完成

**增量备份场景**和**增量恢复场景**：类似上述模式，支持增量数据流。

## 3.4 核心组件

### 3.4.1 会话管理器（SvcSessionManager）

**头文件**：`services/backup_sa/include/module_ipc/svc_session_manager.h`

会话管理器负责维护当前备份/恢复会话的状态。单会话模型意味着同一时刻只有一个会话处于活动状态。

```cpp
struct Impl {
    uint32_t clientToken;                      // 客户端令牌
    IServiceReverseType::Scenario scenario;    // 场景类型
    std::map<BundleName, BackupExtInfo> backupExtNameMap;  // 扩展映射
    sptr<IServiceReverse> clientProxy;         // 客户端回调代理
    bool isBackupStart;                        // 是否已开始备份
    int32_t userId;                            // 用户 ID
    // ... 其他状态
};
```

会话状态流转：
```
Idle -> Initializing -> Ready -> Running -> Finishing -> Idle
```

### 3.4.2 调度器（SchedScheduler）

**头文件**：`services/backup_sa/include/module_sched/sched_scheduler.h`

调度器负责管理备份任务的队列和执行顺序。调度策略确保：
- 按顺序处理已添加的应用
- 支持超时控制
试

### 3.4.- 支持失败重3 扩展连接管理（SvcBackupConnection）

**头文件**：`services/backup_sa/include/module_ipc/svc_backup_connection.h`

```cpp
class SvcBackupConnection : public AAFwk::AbilityConnectionStub {
    void OnAbilityConnectDone() override;      // 连接完成回调
    void OnAbilityDisconnectDone() override;  // 断开连接回调
    sptr<IExtension> GetBackupExtProxy();      // 获取扩展代理
};
```

扩展连接管理负责：
- 启动备份扩展进程
- 与扩展建立 IPC 连接
- 处理连接断开和异常

### 3.4.4 SA 扩展连接（SABackupConnection）

**头文件**：`services/backup_sa/include/module_ipc/sa_backup_connection.h`

```cpp
class SABackupConnection : public SystemAbilityExtensionPara {
    int32_t saId_ = BConstants::BACKUP_DEFAULT_SA_ID;
};
```

SA 扩展连接用于备份系统能力（System Ability）的数据，支持系统级备份场景。

## 3.5 关键流程

### 3.5.1 备份流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              完整备份流程                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. 初始化阶段                                                              │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                            │
│  │ JS 调用  │────▶│ 创建会话  │────▶│ 发布服务  │                            │
│  │InitBackup│     │ 配置参数  │     │ 等待连接  │                            │
│  └──────────┘     └──────────┘     └──────────┘                            │
│                                                                             │
│  2. 扩展启动阶段                                                            │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐           │
│  │ 启动扩展 │────▶│ IPC 连接 │────▶│ 扩展就绪 │────▶│ 发送配置 │           │
│  │ 进程     │     │ 建立     │     │ 回调     │     │ 参数     │           │
│  └──────────┘     └──────────┘     └──────────┘     └──────────┘           │
│                                                                             │
│  3. 数据传输阶段                                                            │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐           │
│  │ 获取文件 │────▶│ 数据打包 │────▶│ 队列缓存 │────▶│ IPC 发送 │           │
│  │ 信息     │     │ (tar)    │     │          │     │          │           │
│  └──────────┘     └──────────┘     └──────────┘     └──────────┘           │
│       ↑                              │              │                       │
│       └──────────────────────────────┴──────────────┘                       │
│              (循环处理所有文件)                                             │
│                                                                             │
│  4. 完成阶段                                                               │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                            │
│  │ 发送完成 │────▶│ 清理资源  │────▶│ 释放扩展  │                            │
│  │ 回调     │     │ 队列清空  │     │ 进程     │                            │
│  └──────────┘     └──────────┘     └──────────┘                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.5.2 恢复流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              完整恢复流程                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. 初始化阶段                                                              │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                            │
│  │ JS 调用  │────▶│ 创建会话  │────▶│ 解析 Manifest │                       │
│  │InitRestore│     │ 配置参数  │     │ 文件     │                          │
│  └──────────┘     └──────────┘     └──────────┘                            │
│                                    │                                         │
│                                    ▼                                         │
│                          ┌──────────┐                                       │
│                          │ 加载目标 │                                       │
│                          │ 应用扩展  │                                       │
│                          └──────────┘                                       │
│                                                                             │
│  2. 数据传输阶段                                                            │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐           │
│  │ 读取Manifest│   │ IPC 发送 │────▶│ 扩展接收 │────▶│ 解压还原 │           │
│  │ 文件     │     │ 数据     │     │          │     │          │           │
│  └──────────┘     └──────────┘     └──────────┘     └──────────┘           │
│       ↑                              │              │                       │
│       └──────────────────────────────┴──────────────┘                       │
│              (循环处理所有文件)                                             │
│                                                                             │
│  3. 安装阶段（可选）                                                        │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                            │
│  │ 安装应用 │────▶│ 恢复数据 │────▶│ 完成回调 │                            │
│  │ (BMS)   │     │          │     │          │                            │
│  └──────────┘     └──────────┘     └──────────┘                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 3.6 常量定义

**文件**：`utils/include/b_resources/b_constants.h`

| 常量 | 值 | 说明 |
|------|-----|------|
| `BACKUP_DEFAULT_SA_ID` | -1 | 默认 SA ID |
| `BACKUP_SA_RELOAD_MAX` | 2 | 最大 SA 重载次数 |
| `BACKUP_LOADSA_TIMEOUT_MS` | 5000 | SA 加载超时 |
| `DEFAULT_USER_ID` | 100 | 默认用户 ID |
| `EXTENSION_THREAD_POOL_COUNT` | 1 | 扩展线程池大小 |
| `EXT_CONNECT_MAX_COUNT` | 3 | 最大扩展连接次数 |
| `EXT_CONNECT_MAX_TIME` | 25000 | 最大连接等待时间 |
| `DEFAULT_TIMEOUT` | 15 * 60 * 1000 | 默认超时（15分钟） |

## 3.7 相关文档

| 文档 | 说明 |
|------|------|
| [系统架构](01_Architecture.md) | 组件图、数据流、线程模型 |
| [项目概览](00_Overview.md) | 项目定位、核心能力 |
| [JS N-API 接口](10_NAPI_JS.md) | Backup JS API |
| [内部 API](20_Inner_API.md) | backup_kit_inner 接口 |
| [工具库](03_Utils.md) | 工具模块详解 |
| [GN 构建配置](04_GN_Build.md) | SA 构建配置 |
| [安全评审](05_Security_Review.md) | SA 安全风险 |
