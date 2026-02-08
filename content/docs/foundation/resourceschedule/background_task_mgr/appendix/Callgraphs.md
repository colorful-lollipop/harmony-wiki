# 附录 - 关键调用链

本文档记录后台任务管理模块的关键调用链，帮助开发者追踪代码执行路径。

---

## 1. 服务启动调用链

### 1.1 System Ability启动流程

```
系统启动
    ↓
resource_schedule_service进程启动
    ↓
SA框架 → MakeAndRegisterAbility(1903)
    ↓
BackgroundTaskMgrService构造函数
    ↓
SystemAbility构造函数
    ↓
OnStart() [services/core/src/background_task_mgr_service.cpp:85-118]
    ├── Create EventRunner("backgroundTaskMgr")
    ├── BgTransientTaskMgr::GetInstance()->Init(runner)
    │   ├── Create TimerManager
    │   ├── Create Watchdog
    │   └── Create DecisionMaker
    ├── BgContinuousTaskMgr::GetInstance()->Init(runner)
    │   ├── RegisterNotificationSubscriber()
    │   ├── RegisterSysCommEventListener()
    │   ├── RegisterAppStateObserver()
    │   └── HandlePersistenceData()
    ├── BgEfficiencyResourcesMgr::GetInstance()->Init(runner)
    │   └── Create ResourcesSubscriberMgr
    └── AddSystemAbilityListener()
        ↓
等待依赖SA就绪...
        ↓
OnAddSystemAbility(APP_MGR_SERVICE_ID)
    ↓
SetReady(TRANSIENT_SERVICE_READY)
    ↓
OnAddSystemAbility(BUNDLE_MGR_SERVICE_ID)
    ↓
SetReady(CONTINUOUS_SERVICE_READY)
    ↓
OnAddSystemAbility(OTHER_DEPS)
    ↓
SetReady(EFFICIENCY_RESOURCES_SERVICE_READY)
    ↓
All Ready → Publish(1903)
    ↓
STATE_RUNNING
```

---

## 2. 短时任务调用链

### 2.1 申请短时任务

```
JS调用
    ↓
requestSuspendDelay(reason, callback)
    ↓
N-API层 [interfaces/kits/napi/src/request_suspend_delay.cpp:90]
    ├── ParseParameters() 参数解析
    │   ├── napi_get_cb_info() 获取参数
    │   ├── GetStringValue() 解析reason
    │   └── GetCallback() 解析callback
    └── CreateExpiredCallback() 创建回调
    ↓
BackgroundTaskManager::RequestSuspendDelay() [frameworks/src/background_task_manager.cpp:63]
    ├── GetBackgroundTaskManagerProxy() 获取代理
    │   ├── samgr::GetSystemAbility(1903)
    │   └── iface_cast<IBackgroundTaskMgr>()
    └── proxy_->RequestSuspendDelay() IPC调用
    ↓
BackgroundTaskMgrStub::OnRemoteRequest() [IDL生成]
    └── 分发到 RequestSuspendDelay
    ↓
BackgroundTaskMgrService::RequestSuspendDelay() [services/core/src/background_task_mgr_service.cpp]
    ├── CheckHapCalling() 权限检查
    │   ├── GetCallingTokenID()
    │   ├── GetTokenTypeFlag() 检查TOKEN_HAP
    │   └── VerifyAccessToken(BGMODE_PERMISSION)
    └── BgTransientTaskMgr::RequestSuspendDelay() [services/transient_task/src/bg_transient_task_mgr.cpp:191]
        ├── IsCallingInfoLegal()
        │   ├── GetCallingUid/Pid()
        │   ├── VerifyCallingInfo() UID/PID >= 0
        │   ├── GetBundleNamesForUid() 获取Bundle名
        │   └── 检查callback有效性
        ├── DecisionMaker::Decide() [services/transient_task/src/decision_maker.cpp:45]
        │   ├── IsExceedMaxBgTaskDurationPerDay() 检查配额
        │   ├── IsDevice cooperative() 检查设备状态
        │   └── GetDecision() 决策
        ├── 生成RequestId
        ├── TimerManager::StartTimer() [services/transient_task/src/timer_manager.cpp]
        │   └── handler_->PostTask(delayTask, delayTime)
        ├── 保存到expiredCallbackMap_
        └── 返回DelaySuspendInfo
    ↓
返回给JS层
```

### 2.2 短时任务超时回调

```
Timer超时
    ↓
TimerManager::OnTimeout(requestId) [services/transient_task/src/timer_manager.cpp]
    ↓
BgTransientTaskMgr::HandleRequestExpired(requestId) [services/transient_task/src/bg_transient_task_mgr.cpp:357]
    ├── 查找expiredCallbackMap_
    ├── callback->OnExpired() 调用回调
    │   ↓
    │   ExpiredCallbackImpl::OnExpired() [IDL生成]
    │       ↓
    │   IPC回调到客户端
    │       ↓
    │   JS callback执行
    │
    └── ForceCancelSuspendDelay(requestId) 强制取消
        ├── 从keyInfoMap_移除
        └── 清理资源
```

---

## 3. 长时任务调用链

### 3.1 启动长时任务

```
JS调用
    ↓
startBackgroundRunning(context, bgMode, wantAgent)
    ↓
N-API层 [interfaces/kits/napi/src/bg_continuous_task_napi_module.cpp:542]
    ├── ParseContinuousTaskParam() 参数解析
    │   ├── GetContext() 获取context
    │   ├── GetBackgroundMode() 获取mode
    │   └── GetWantAgent() 获取wantAgent
    └── StartBackgroundRunningThrow()
    ↓
BackgroundTaskManager::RequestStartBackgroundRunning() [frameworks/src/background_task_manager.cpp:79]
    └── proxy_->StartBackgroundRunning() IPC调用
    ↓
BackgroundTaskMgrStub::OnRemoteRequest()
    ↓
BackgroundTaskMgrService::StartBackgroundRunning() [services/core/src/background_task_mgr_service.cpp]
    ├── CheckHapCalling() 权限检查
    ├── CheckAtomicService() 检查非原子服务
    └── BgContinuousTaskMgr::StartBackgroundRunning() [services/continuous_task/src/bg_continuous_task_mgr.cpp:486]
        ├── CheckIsSysReadyAndPermission()
        ├── Create ContinuousTaskRecord
        ├── InitRecordParam() 初始化记录
        │   ├── 设置UID/PID/Bundle名
        │   └── 设置bgModeId
        ├── CheckBgmodeType() 检查后台模式
        │   ├── GetBackgroundModeInfo() 从缓存获取
        │   └── 验证模式是否配置
        ├── CheckNotificationText() 检查通知文本
        ├── SendContinuousTaskNotification() [services/continuous_task/src/bg_continuous_task_mgr.cpp:687]
        │   ├── GetNotificationPrompt() 获取提示文本
        │   ├── CreateNotificationRequest() 创建通知
        │   └── NotificationHelper::PublishNotification()
        ├── RegisterAppStateObserver() 注册应用监听
        └── 保存到continuousTaskInfosMap_
    ↓
返回结果给JS
```

### 3.2 应用停止时清理长时任务

```
AppManager通知应用停止
    ↓
AppStateObserver::OnProcessDied() [services/common/src/app_state_observer.cpp]
    ↓
BgContinuousTaskMgr::OnAppStopped(uid) [services/continuous_task/src/bg_continuous_task_mgr.cpp:1127]
    ├── 查找continuousTaskInfosMap_
    ├── StopBackgroundRunningInner()
    │   ├── CancelNotification() 取消通知
    │   ├── RemoveContinuousTaskRecord() 移除记录
    │   └── NotifySubscribers(TASK_CANCEL) 通知订阅者
    └── 清理缓存数据
```

---

## 4. 能效资源调用链

### 4.1 申请能效资源

```
JS调用
    ↓
applyEfficiencyResources(request)
    ↓
N-API层 [interfaces/kits/napi/src/efficiency_resources_operation.cpp:165]
    ├── ParseParameters() 参数解析
    │   ├── GetNamedInt32Value("resourceTypes")
    │   ├── GetNamedBoolValue("isApply")
    │   └── GetNamedInt32Value("timeOut")
    └── CheckValidInfo() 验证参数
    ↓
BackgroundTaskManager::ApplyEfficiencyResources() [frameworks/src/background_task_manager.cpp:192]
    └── proxy_->ApplyEfficiencyResources() IPC调用
    ↓
BackgroundTaskMgrService::ApplyEfficiencyResources()
    └── BgEfficiencyResourcesMgr::ApplyEfficiencyResources() [services/efficiency_resources/src/bg_efficiency_resources_mgr.cpp:160]
        ├── IsCallingInfoLegal() 验证调用者
        ├── CheckResourceInfo() 检查资源信息
        │   └── resourceNumber有效性检查
        ├── CheckIfCanApplyCpuLevel() 检查CPU级别 [services/efficiency_resources/src/bg_efficiency_resources_mgr.cpp:292]
        │   ├── CheckBundleName() 检查Bundle在白名单
        │   └── CheckCpuLevel() 检查级别是否允许
        ├── GetBundleNamesForUid() 获取Bundle名
        ├── SendResourceApplyTask() 发送申请任务
        │   └── handler_->PostTask()
        └── ApplyEfficiencyResourcesInner()
            ├── Create ResourceCallbackInfo
            ├── UpdateResourcesEndtime()
            │   └── 计算超时时间
            ├── 保存到appResourceApplyMap_或procResourceApplyMap_
            └── ReportHisysEvent() 上报事件
```

### 4.2 应用死亡时释放资源

```
AppManager通知进程死亡
    ↓
AppStateObserver::OnProcessDied()
    ↓
BgEfficiencyResourcesMgr::RemoveProcessRecord(uid, pid, bundleName) [services/efficiency_resources/src/bg_efficiency_resources_mgr.cpp:247]
    ├── 查找procResourceApplyMap_
    ├── ResetEfficiencyResourcesInner()
    │   ├── 查找资源记录
    │   ├── 通知订阅者 OnEfficiencyResourcesReset
    │   └── 从map移除
    └── ReportHisysEvent() 上报释放事件
```

---

## 5. 订阅调用链

### 5.1 订阅后台任务事件

```
JS调用 subscribeContinuousTaskState
    ↓
N-API层 [interfaces/kits/napi/src/js_backgroundtask_subscriber.cpp]
    ↓
BackgroundTaskManager::SubscribeBackgroundTask() [frameworks/src/background_task_manager.cpp:148]
    ├── Create BackgroundTaskSubscriberImpl
    └── proxy_->SubscribeBackgroundTask() IPC
    ↓
BackgroundTaskMgrService::SubscribeBackgroundTask()
    ├── CheckCallingToken() / CheckHapCalling() 权限检查
    ├── CheckAtomicService() 检查非原子服务
    └── BgContinuousTaskMgr::AddSubscriber() [services/continuous_task/src/bg_continuous_task_mgr.cpp:157]
        └── AddSubscriberInner()
            ├── 创建SubscriberInfo
            ├── 添加到bgTaskSubscribers_列表
            └── susriberDeathRecipient_ 注册死亡监听
```

### 5.2 事件通知流程

```
长时任务状态变更
    ↓
BgContinuousTaskMgr::OnContinuousTaskChanged() [services/continuous_task/src/bg_continuous_task_mgr.cpp:1997]
    ├── Create ContinuousTaskCallbackInfo
    └── NotifySubscribers(changeEventType, callbackInfo)
        ├── NotifySubscribersTaskStart/Update/Cancel/Suspend/Active
        └── 遍历bgTaskSubscribers_
            └── CanNotifyHap() 检查是否有权限通知
                ↓
            IBackgroundTaskSubscriber::OnContinuousTaskStart() IPC回调
                ↓
            BackgroundTaskSubscriberProxy 代理
                ↓
            客户端BackgroundTaskSubscriberImpl 接收
                ↓
            JS层回调执行
```

---

## 6. Dump调用链

```
hidumper -s 1903 -a '-a'
    ↓
BackgroundTaskMgrService::Dump() [services/core/src/background_task_mgr_service.cpp:118]
    ├── AllowDump() 检查DUMP权限
    │   └── VerifyAccessToken("ohos.permission.DUMP")
    ├── DumpUsage() 输出帮助信息
    └── 根据参数分发:
        ├── DumpAllTransientTasks() [services/transient_task/src/bg_transient_task_mgr.cpp:618]
        │   └── 遍历keyInfoMap_输出
        ├── BgContinuousTaskMgr::ShellDump() [services/continuous_task/src/bg_continuous_task_mgr.cpp:1337]
        │   └── DumpAllTaskInfo() 输出continuousTaskInfosMap_
        └── BgEfficiencyResourcesMgr::ShellDump() [services/efficiency_resources/src/bg_efficiency_resources_mgr.cpp:178]
            └── DumpAllApplicationInfo() 输出appResourceApplyMap_
```

---

## 7. IPC接口定义调用链

### 7.1 IDL接口到实现映射

```
IBackgroundTaskMgr.idl 定义
    ↓
IDL编译器生成
    ├── BackgroundTaskMgrStub.h/cpp  (服务端存根)
    └── BackgroundTaskMgrProxy.h/cpp (客户端代理)
    ↓
服务端调用链:
BackgroundTaskMgrStub::OnRemoteRequest(code, data, reply, option)
    ├── 解析code确定方法
    └── 调用对应的虚函数
        ↓
    BackgroundTaskMgrService::具体方法() 实现
    
客户端调用链:
BackgroundTaskMgrProxy::具体方法(params)
    ├── data.WriteInterfaceToken()
    ├── data.Write参数序列化
    ├── Remote()->SendRequest() IPC发送
    └── reply.Read结果反序列化
```

---

## 8. 死亡监听调用链

### 8.1 客户端服务死亡监听

```
BackgroundTaskManager获取代理
    ↓
AddDeathRecipient(BgTaskMgrDeathRecipient)
    ↓
服务进程死亡
    ↓
BgTaskMgrDeathRecipient::OnRemoteDied() [frameworks/src/background_task_manager.cpp:366]
    └── backgroundTaskManager_.ResetBackgroundTaskManagerProxy()
        ├── mutex_锁定
        ├── proxy_.clear()
        └── recipient_.clear()
    ↓
下次调用自动重建代理
```

### 8.2 服务端客户端死亡监听

```
BgContinuousTaskMgr::AddSubscriber()
    ↓
sptr<RemoteDeathRecipient> susriberDeathRecipient_
    ↓
客户端进程死亡
    ↓
RemoteDeathRecipient::OnRemoteDied() [services/continuous_task/include/remote_death_recipient.h]
    └── callback_(object) 执行回调
        ↓
    BgContinuousTaskMgr::OnRemoteSubscriberDied(object) [services/continuous_task/src/bg_continuous_task_mgr.cpp:120]
        └── RemoveSubscriberInner() 清理订阅者
```

---

## 相关文档

- [架构说明](../02_Architecture.md) - 组件关系
- [N-API接口](../03_NAPI_Reference.md) - JS接口
- [内部API](../04_Inner_API.md) - C++接口
