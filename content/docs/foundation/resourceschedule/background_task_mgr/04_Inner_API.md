# 内部API说明

## 概述

本文档描述后台任务管理模块的内部C++接口，供系统开发者和服务间对接使用。

**主要接口头文件位置**: `interfaces/innerkits/include/`

**命名空间**: `OHOS::BackgroundTaskMgr`

---

## 1. IPC接口定义

### IBackgroundTaskMgr (IDL接口)

**文件**: `interfaces/innerkits/IBackgroundTaskMgr.idl`

**接口描述符**: `u"ohos.resourceschedule.IBackgroundTaskMgr"`

**方法分类**:

#### 1.1 短时任务接口

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `RequestSuspendDelay` | reason, callback | DelaySuspendInfo | 申请延迟挂起 |
| `CancelSuspendDelay` | requestId | ErrCode | 取消延迟挂起 |
| `GetRemainingDelayTime` | requestId, delayTime | ErrCode | 获取剩余时间 |
| `GetAllTransientTasks` | remainingQuota, list | ErrCode | 获取所有短时任务 |
| `GetTransientTaskApps` | list | ErrCode | 获取短时任务应用列表 |
| `PauseTransientTaskTimeForInner` | uid | ErrCode | 暂停任务计时（内部） |
| `StartTransientTaskTimeForInner` | uid | ErrCode | 恢复任务计时（内部） |

#### 1.2 长时任务接口

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `StartBackgroundRunning` | taskParam, notificationId, continuousTaskId | ErrCode | 启动长时任务 |
| `UpdateBackgroundRunning` | taskParam, notificationId, continuousTaskId | ErrCode | 更新长时任务 |
| `RequestBackgroundRunningForInner` | taskParam | ErrCode | 内部启动接口 |
| `RequestGetContinuousTasksByUidForInner` | uid, list | ErrCode | 按UID查询 |
| `StopBackgroundRunning` | abilityName, abilityToken, abilityId, continuousTaskId | ErrCode | 停止长时任务 |
| `GetAllContinuousTasks` | list | ErrCode | 获取所有任务 |
| `GetContinuousTaskApps` | list | ErrCode | 获取任务应用列表 |
| `StopContinuousTask` | uid, pid, taskType, key | ErrCode | 停止指定任务 |
| `SuspendContinuousTask` | uid, pid, reason, key | ErrCode | 挂起任务 |
| `ActiveContinuousTask` | uid, pid, key | ErrCode | 激活挂起任务 |

#### 1.3 能效资源接口

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ApplyEfficiencyResources` | resourceInfo | ErrCode | 申请资源 |
| `ResetAllEfficiencyResources` | - | ErrCode | 重置所有资源 |
| `GetEfficiencyResourcesInfos` | appList, procList | ErrCode | 获取资源信息 |
| `GetAllEfficiencyResources` | resourceInfoList | ErrCode | 获取所有资源 |

#### 1.4 订阅接口

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `SubscribeBackgroundTask` | subscriber, flag | ErrCode | 订阅任务事件 |
| `UnsubscribeBackgroundTask` | subscriber, flag | ErrCode | 取消订阅 |

#### 1.5 系统管理接口

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `SetBgTaskConfig` | configData, sourceType | ErrCode | 设置配置 |
| `AVSessionNotifyUpdateNotification` | uid, pid, isPublish | ErrCode | AVSession通知更新 |
| `SuspendContinuousAudioTask` | uid | ErrCode | 挂起音频任务 |
| `IsModeSupported` | taskParam | ErrCode | 检查模式支持 |
| `RequestAuthFromUser` | taskParam, callback, notificationId | ErrCode | 请求用户授权 |
| `CheckSpecialScenarioAuth` | appIndex, authResult | ErrCode | 检查特殊场景授权 |
| `CheckTaskAuthResult` | bundleName, userId, appIndex | ErrCode | 检查任务授权 |
| `EnableContinuousTaskRequest` | uid, isEnable | ErrCode | 启用/禁用任务请求 |
| `SetBackgroundTaskState` | taskParam | ErrCode | 设置任务状态 |
| `GetBackgroundTaskState` | taskParam, authResult | ErrCode | 获取任务状态 |
| `SetSpecialExemptedProcess` | bundleNameSet | ErrCode | 设置特殊豁免进程 |
| `Dump` | fd, args | int32_t | Dump调试信息 |

---

## 2. BackgroundTaskMgrHelper

**文件**: `interfaces/innerkits/include/background_task_mgr_helper.h`

**描述**: 静态工具类，封装内部API调用，简化客户端使用

### 2.1 长时任务API

```cpp
// 申请启动长时任务
static ErrCode RequestStartBackgroundRunning(ContinuousTaskParam &taskParam);

// 申请更新长时任务
static ErrCode RequestUpdateBackgroundRunning(ContinuousTaskParam &taskParam);

// 内部接口：为内部Ability启动/停止长时任务
static ErrCode RequestBackgroundRunningForInner(
    const ContinuousTaskParamForInner &taskParam);

// 内部接口：按UID获取长时任务
static ErrCode RequestGetContinuousTasksByUidForInner(
    int32_t uid,
    std::vector<std::shared_ptr<ContinuousTaskInfo>> &list);

// 获取所有长时任务
static ErrCode RequestGetAllContinuousTasks(
    std::vector<std::shared_ptr<ContinuousTaskInfo>> &list);

// 获取所有长时任务（含挂起）
static ErrCode RequestGetAllContinuousTasks(
    std::vector<std::shared_ptr<ContinuousTaskInfo>> &list, 
    bool includeSuspended);

// 停止长时任务
static ErrCode RequestStopBackgroundRunning(
    const std::string &abilityName,
    const sptr<IRemoteObject> &abilityToken,
    int32_t abilityId = -1,
    int32_t continuousTaskId = -1);
```

### 2.2 订阅API

```cpp
// 订阅后台任务事件
static ErrCode SubscribeBackgroundTask(const BackgroundTaskSubscriber &subscriber);

// 取消订阅
static ErrCode UnsubscribeBackgroundTask(const BackgroundTaskSubscriber &subscriber);
```

### 2.3 查询API

```cpp
// 获取短时任务应用列表
static ErrCode GetTransientTaskApps(
    std::vector<std::shared_ptr<TransientTaskAppInfo>> &list);

// 获取长时任务应用列表
static ErrCode GetContinuousTaskApps(
    std::vector<std::shared_ptr<ContinuousTaskCallbackInfo>> &list);

// 获取能效资源信息
static ErrCode GetEfficiencyResourcesInfos(
    std::vector<std::shared_ptr<ResourceCallbackInfo>> &appList,
    std::vector<std::shared_ptr<ResourceCallbackInfo>> &procList);
```

### 2.4 内部控制API

```cpp
// 暂停短时任务计时（内部）
static ErrCode PauseTransientTaskTimeForInner(int32_t uid);

// 恢复短时任务计时（内部）
static ErrCode StartTransientTaskTimeForInner(int32_t uid);

// 停止长时任务
static ErrCode StopContinuousTask(
    int32_t uid, 
    int32_t pid, 
    uint32_t taskType, 
    const std::string &key);

// 挂起长时任务
static ErrCode SuspendContinuousTask(
    int32_t uid, 
    int32_t pid, 
    int32_t reason, 
    const std::string &key);

// 激活长时任务
static ErrCode ActiveContinuousTask(
    int32_t uid, 
    int32_t pid, 
    const std::string &key);

// AVSession通知更新
static ErrCode AVSessionNotifyUpdateNotification(
    int32_t uid, 
    int32_t pid, 
    bool isPublish = false);

// 设置配置
static ErrCode SetBgTaskConfig(
    const std::string &configData, 
    int32_t sourceType);
```

### 2.5 管理API

```cpp
// 挂起音频任务
static ErrCode SuspendContinuousAudioTask(int32_t uid);

// 检查模式支持
static ErrCode IsModeSupported(ContinuousTaskParam &taskParam);

// 设置支持TASK_KEEPING的进程
static ErrCode SetSupportedTaskKeepingProcesses(
    const std::set<std::string> &processSet);

// 设置恶意应用配置
static ErrCode SetMaliciousAppConfig(
    const std::set<std::string> &maliciousAppSet);

// 检查特殊场景授权
static ErrCode CheckSpecialScenarioAuth(
    int32_t appIndex, 
    uint32_t &authResult);

// 检查任务授权结果
static ErrCode CheckTaskAuthResult(
    const std::string &bundleName, 
    int32_t userId, 
    int32_t appIndex);

// 启用/禁用长时任务请求
static ErrCode EnableContinuousTaskRequest(
    int32_t uid, 
    bool isEnable);

// 设置任务状态
static ErrCode SetBackgroundTaskState(
    std::shared_ptr<BackgroundTaskStateInfo> taskParam);

// 获取任务状态
static ErrCode GetBackgroundTaskState(
    std::shared_ptr<BackgroundTaskStateInfo> taskParam);

// 获取所有长时任务（系统级）
static ErrCode GetAllContinuousTasksBySystem(
    std::vector<std::shared_ptr<ContinuousTaskInfo>> &list);

// 设置特殊豁免进程
static ErrCode SetSpecialExemptedProcess(
    const std::set<std::string> &bundleNameSet);

// 获取所有长时任务应用（含挂起）
static ErrCode GetAllContinuousTaskApps(
    std::vector<std::shared_ptr<ContinuousTaskCallbackInfo>> &list);
```

---

## 3. 数据结构

### 3.1 ContinuousTaskParam

**文件**: `interfaces/innerkits/include/continuous_task_param.h`

```cpp
struct ContinuousTaskParam : public Parcelable {
    bool isNewApi_ {false};           // 是否使用新API
    uint32_t bgModeId_ {0};           // 后台模式ID
    std::shared_ptr<AbilityRuntime::WantAgent::WantAgent> wantAgent_ {nullptr}; // WantAgent
    std::string abilityName_ {""};    // Ability名称
    sptr<IRemoteObject> abilityToken_ {nullptr}; // Ability Token
    std::string appName_ {""};        // 应用名称
    bool isBatchApi_ {false};         // 是否批量API
    std::vector<uint32_t> bgModeIds_ {}; // 批量模式ID列表
    int32_t abilityId_ {-1};          // Ability ID
    std::vector<uint32_t> bgSubModeIds_ {}; // 子模式ID列表
    bool isCombinedTaskNotification_ {false}; // 是否合并通知
    int32_t combinedNotificationTaskId_ {-1}; // 合并通知任务ID
    int32_t updateTaskId_ {-1};       // 更新任务ID
    bool isByRequestObject_ {false};  // 是否通过请求对象
    int32_t appIndex_ {-1};           // 应用索引
    int32_t notificationId_ {-1};     // 输出：通知ID
    int32_t continuousTaskId_ {-1};   // 输出：长时任务ID
};
```

### 3.2 DelaySuspendInfo

**文件**: `interfaces/innerkits/include/delay_suspend_info.h`

```cpp
class DelaySuspendInfo : public Parcelable {
public:
    int32_t requestId_ {0};           // 请求ID
    int32_t actualDelayTime_ {0};     // 实际延迟时间（毫秒）
};
```

### 3.3 EfficiencyResourceInfo

**文件**: `interfaces/innerkits/include/efficiency_resource_info.h`

```cpp
class EfficiencyResourceInfo : public Parcelable {
public:
    uint32_t resourceNumber_ {0};     // 资源类型位掩码
    bool isApply_ {false};            // 申请/释放
    int32_t timeOut_ {0};             // 超时时间（毫秒）
    std::string reason_ {""};         // 原因
    bool isPersist_ {false};          // 是否持久化
    bool isProcess_ {false};          // 是否进程级
    int32_t cpuLevel_ {0};            // CPU级别
    
    uint32_t GetResourceNumber() const;
    bool IsApply() const;
    int32_t GetTimeOut() const;
    bool IsPersist() const;
    bool IsProcess() const;
    int32_t GetCpuLevel() const;
    void SetCpuLevel(int32_t level);
};
```

### 3.4 ContinuousTaskInfo

**文件**: `interfaces/innerkits/include/continuous_task_info.h`

```cpp
class ContinuousTaskInfo : public Parcelable {
public:
    std::string abilityName_;         // Ability名称
    bool isContinuousTask_ {false};   // 是否为长时任务
    uint32_t bgModeId_ {0};           // 后台模式ID
    std::shared_ptr<Want> want_;      // Want对象
    int32_t uid_ {0};                 // UID
    int32_t pid_ {0};                 // PID
    int32_t abilityId_ {-1};          // Ability ID
    std::vector<uint32_t> bgModeIds_; // 批量模式列表
    int32_t continuousTaskId_ {-1};   // 长时任务ID
};
```

### 3.5 BackgroundTaskSubscriber

**文件**: `interfaces/innerkits/include/background_task_subscriber.h`

```cpp
class BackgroundTaskSubscriber {
public:
    // 长时任务开始回调
    virtual void OnContinuousTaskStart(
        const std::shared_ptr<ContinuousTaskCallbackInfo> &continuousTaskCallbackInfo) = 0;
    
    // 长时任务取消回调
    virtual void OnContinuousTaskCancel(
        const std::shared_ptr<ContinuousTaskCallbackInfo> &continuousTaskCallbackInfo) = 0;
    
    // 长时任务更新回调
    virtual void OnContinuousTaskUpdate(
        const std::shared_ptr<ContinuousTaskCallbackInfo> &continuousTaskCallbackInfo) = 0;
    
    // 长时任务挂起回调
    virtual void OnContinuousTaskSuspend(
        const std::shared_ptr<ContinuousTaskCallbackInfo> &continuousTaskCallbackInfo) = 0;
    
    // 长时任务激活回调
    virtual void OnContinuousTaskActive(
        const std::shared_ptr<ContinuousTaskCallbackInfo> &continuousTaskCallbackInfo) = 0;
    
    // 短时任务开始回调
    virtual void OnTransientTaskStart(
        const std::shared_ptr<TransientTaskAppInfo> &transientTaskAppInfo) = 0;
    
    // 短时任务结束回调
    virtual void OnTransientTaskEnd(
        const std::shared_ptr<TransientTaskAppInfo> &transientTaskAppInfo) = 0;
    
    // 短时任务失败回调
    virtual void OnTransientTaskErr(
        const std::shared_ptr<TransientTaskAppInfo> &transientTaskAppInfo) = 0;
    
    // 应用短时任务开始
    virtual void OnAppTransientTaskStart(
        const std::shared_ptr<TransientTaskAppInfo> &transientTaskAppInfo) = 0;
    
    // 应用短时任务结束
    virtual void OnAppTransientTaskEnd(
        const std::shared_ptr<TransientTaskAppInfo> &transientTaskAppInfo) = 0;
    
    // 能效资源申请回调
    virtual void OnEfficiencyResourcesApply(
        const std::shared_ptr<ResourceCallbackInfo> &resourceInfo) = 0;
    
    // 能效资源重置回调
    virtual void OnEfficiencyResourcesReset(
        const std::shared_ptr<ResourceCallbackInfo> &resourceInfo) = 0;
};
```

---

## 4. 枚举定义

### 4.1 BackgroundMode

**文件**: `interfaces/innerkits/include/background_mode.h`

```cpp
enum class BackgroundMode : uint32_t {
    DATA_TRANSFER = 0,
    AUDIO_PLAYBACK = 1,
    AUDIO_RECORDING = 2,
    LOCATION = 3,
    BLUETOOTH_INTERACTION = 4,
    MULTI_DEVICE_CONNECTION = 5,
    WIFI_INTERACTION = 6,
    VOIP = 7,
    TASK_KEEPING = 8
};
```

### 4.2 ResourceType

**文件**: `interfaces/innerkits/include/resource_type.h`

```cpp
enum class ResourceType : uint32_t {
    CPU = 1,
    COMMON_EVENT = 2,
    TIMER = 4,
    WORK_SCHEDULER = 8,
    BLUETOOTH = 16,
    GPS = 32,
    AUDIO = 64,
    RUNNING_LOCK = 128,
    SENSOR = 256
};
```

### 4.3 ContinuousTaskCancelReason

**文件**: `interfaces/innerkits/include/continuous_task_cancel_reason.h`

```cpp
enum class ContinuousTaskCancelReason : uint32_t {
    USER_CANCEL = 0,
    SYSTEM_CANCEL = 1,
    USER_CANCEL_REMOVE_NOTIFICATION = 2,
    SYSTEM_CANCEL_DATA_TRANSFER_LOW_SPEED = 3,
    SYSTEM_CANCEL_AUDIO_PLAYBACK_NOT_USE_AVSESSION = 4,
    SYSTEM_CANCEL_AUDIO_PLAYBACK_NOT_RUNNING = 5,
    SYSTEM_CANCEL_AUDIO_RECORDING_NOT_RUNNING = 6,
    SYSTEM_CANCEL_NOT_USE_LOCATION = 7,
    SYSTEM_CANCEL_NOT_USE_BLUETOOTH = 8,
    SYSTEM_CANCEL_NOT_USE_MULTI_DEVICE = 9,
    SYSTEM_CANCEL_USE_ILLEGALLY = 10
};
```

### 4.4 EfficiencyResourcesCpuLevel

**文件**: `interfaces/innerkits/include/efficiency_resources_cpu_level.h`

```cpp
enum class EfficiencyResourcesCpuLevel : int32_t {
    DEFAULT = 0,
    SMALL_CPU = 1,
    MEDIUM_CPU = 2,
    LARGE_CPU = 3
};
```

---

## 5. 错误码定义

### 5.1 通用错误码

**文件**: `frameworks/common/include/bgtaskmgr_inner_errors.h`

```cpp
// 通用错误 (201, 202, 401)
ERR_BGTASK_PERMISSION_DENIED = 201;           // 权限拒绝
ERR_BGTASK_NOT_SYSTEM_APP = 202;              // 非系统应用
ERR_BGTASK_INVALID_PARAM = 401;               // 参数错误

// 基础错误 (980000xxx)
ERR_BGTASK_NO_MEMORY = 980000101;             // 内存不足
ERR_BGTASK_PARCELABLE_FAILED = 980000201;     // 序列化失败
ERR_BGTASK_TRANSACT_FAILED = 980000301;       // IPC事务失败
ERR_BGTASK_SYS_NOT_READY = 980000401;         // 系统未就绪
ERR_BGTASK_SERVICE_NOT_CONNECTED = 980000402; // 服务未连接
```

### 5.2 长时任务错误码

```cpp
// 对象操作 (9800005xx)
ERR_BGTASK_OBJECT_EXISTS = 980000501;         // 对象已存在
ERR_BGTASK_OBJECT_NOT_EXIST = 980000502;      // 对象不存在
ERR_BGTASK_KEEPING_TASK_VERIFY_ERR = 980000503; // TASK_KEEPING验证错误
ERR_BGTASK_INVALID_BGMODE = 980000504;        // 无效后台模式

// 通知相关 (9800006xx)
ERR_BGTASK_NOTIFICATION_VERIFY_FAILED = 980000601; // 通知验证失败
ERR_BGTASK_NOTIFICATION_ERR = 980000602;           // 通知错误
ERR_BGTASK_CHECK_TASK_PARAM = 980000603;           // 任务参数检查失败
```

### 5.3 短时任务错误码

```cpp
// 调用信息错误 (9900001xx)
ERR_BGTASK_INVALID_PID_OR_UID = 990000101;    // 无效PID或UID
ERR_BGTASK_INVALID_BUNDLE_NAME = 990000102;   // 无效Bundle名
ERR_BGTASK_INVALID_REQUEST_ID = 990000103;    // 无效请求ID

// 回调错误 (9900002xx)
ERR_BGTASK_INVALID_CALLBACK = 990000201;      // 回调无效
ERR_BGTASK_CALLBACK_EXISTS = 990000202;       // 回调已存在
ERR_BGTASK_CALLBACK_NOT_EXIST = 990000203;    // 回调不存在

// 配额错误 (9900003xx)
ERR_BGTASK_NOT_IN_PRESET_TIME = 990000301;    // 不在预设时间
ERR_BGTASK_EXCEEDS_THRESHOLD = 990000302;     // 超出配额
ERR_BGTASK_TIME_INSUFFICIENT = 990000303;     // 时间不足
```

### 5.4 能效资源错误码

```cpp
// 资源申请错误 (18700001xx)
ERR_BGTASK_RESOURCES_EXCEEDS_MAX = 1870000101;     // 超出最大限制
ERR_BGTASK_RESOURCES_INVALID_PID_OR_UID = 1870000102; // 无效PID/UID
ERR_BGTASK_EFFICIENCY_RESOURCES_CPU_LEVEL_INVALID = 1870000103; // CPU级别无效
```

---

## 6. 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `BackgroundTaskMgrHelper` | 稳定 | 内部API，向后兼容 |
| `IBackgroundTaskMgr` | 稳定 | IDL接口，版本兼容 |
| `BackgroundTaskSubscriber` | 稳定 | 回调接口 |
| `ContinuousTaskParam` | 稳定 | 参数结构 |
| `DelaySuspendInfo` | 稳定 | 短时任务信息 |
| `EfficiencyResourceInfo` | 稳定 | 资源信息 |

---

## 7. 使用示例

### 7.1 使用Helper申请长时任务

```cpp
#include "background_task_mgr_helper.h"
#include "continuous_task_param.h"

using namespace OHOS::BackgroundTaskMgr;

void StartBackgroundTask() {
    ContinuousTaskParam param;
    param.isNewApi_ = true;
    param.bgModeId_ = static_cast<uint32_t>(BackgroundMode::AUDIO_PLAYBACK);
    param.abilityName_ = "MainAbility";
    // ... 设置其他参数
    
    ErrCode result = BackgroundTaskMgrHelper::RequestStartBackgroundRunning(param);
    if (result == ERR_OK) {
        // 成功
    }
}
```

### 7.2 实现订阅者

```cpp
#include "background_task_subscriber.h"

class MyTaskSubscriber : public BackgroundTaskSubscriber {
public:
    void OnContinuousTaskStart(
        const std::shared_ptr<ContinuousTaskCallbackInfo> &info) override {
        // 处理任务开始事件
    }
    
    void OnContinuousTaskCancel(
        const std::shared_ptr<ContinuousTaskCallbackInfo> &info) override {
        // 处理任务取消事件
    }
    
    // 实现其他纯虚函数...
};

// 注册订阅者
MyTaskSubscriber subscriber;
BackgroundTaskMgrHelper::SubscribeBackgroundTask(subscriber);
```

---

## 8. 相关文档

- [N-API接口](03_NAPI_Reference.md) - JS/ArkTS接口
- [架构说明](02_Architecture.md) - 组件关系
- [GN构建](05_GN_Build.md) - 库文件说明
