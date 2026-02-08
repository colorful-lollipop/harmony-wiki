# 附录：关键调用链

## 目的

本文档记录 Resource Schedule Service 的关键调用链，帮助理解代码执行流程。

---

## 调用链 1: 应用启动事件处理

```
【应用启动】
AbilityRuntime::StartAbility()
  └─► AppStateObserver::OnAbilityRequestDone()
       └─► AppStateObserver::ReportAbilityState()
            └─► ResSchedClient::GetInstance().ReportData()
                 ├─► TryConnect()              // 建立 IPC 连接
                 ├─► CheckSystemAbility(1901)  // 查找服务
                 └─► rss_->ReportData()        // IPC 调用
                      └─► 【SA 1901 进程】
                           ResSchedService::ReportData()
                           ├─► CheckReportDataParcel()  // 权限检查
                           ├─► StringToJsonObj()        // 解析 payload
                           ├─► ResSchedMgr::GetInstance().ReportData()
                           │    └─► PluginMgr::GetInstance().DeliverResource()
                           │         ├─► 查找订阅该类型的插件
                           │         └─► plugin->OnDispatchResource()
                           │              └─► 【各插件处理】
                           │                   cgroup_sched_plugin::OnDispatchResource()
                           │                   ├─► 更新进程分组
                           │                   └─► ProcessGroup::SetProcessGroup()
                           │                        └─► 写入 /dev/cpuctl/...
                           │
                           └─► ResSchedIpcThread::GetInstance().SetQos()
```

---

## 调用链 2: JS 获取系统负载

```
【JS 调用】
systemload.getLevel()
  └─► NAPI: Systemload::GetSystemloadLevel()
       ├─► napi_create_promise()       // 创建 Promise
       ├─► napi_create_async_work()    // 创建异步任务
       ├─► napi_queue_async_work()     // 加入队列
       │
       └─► 【工作线程】
            Systemload::Execute()
            └─► ResSchedClient::GetInstance().GetSystemloadLevel()
                 ├─► TryConnect()              // 检查连接
                 └─► rss_->GetSystemloadLevel() // IPC 调用
                      └─► 【SA 1901 进程】
                           ResSchedService::GetSystemloadLevel()
                           └─► NotifierMgr::GetInstance().GetSystemloadLevel()
                                └─► 返回当前系统负载等级
       
       【主线程回调】
       Systemload::Complete()
       ├─► napi_resolve_deferred()     // 解析 Promise
       └─► napi_delete_async_work()    // 清理
```

---

## 调用链 3: 系统负载变化通知

```
【系统负载变化】
SocPerfPlugin::OnDispatchResource()  // 或 Thermal 事件
  └─► 检测到负载变化
       └─► NotifierMgr::GetInstance().OnDeviceLevelChanged()
            ├─► 更新内部负载状态
            └─► 遍历所有注册的 Notifier
                 └─► SystemloadListener::OnSystemloadLevel() (IPC callback)
                      └─► 【客户端进程】
                           ResSchedClient::SystemloadLevelListener::OnSystemloadLevel()
                           ├─► 遍历所有 JS 回调
                           └─► SystemloadListener::OnSystemloadLevel()
                                └─► napi_call_threadsafe_function()
                                     └─► 【JS 主线程】
                                          SystemloadListener::ThreadSafeCallBack()
                                          └─► JS 回调函数
```

---

## 调用链 4: 插件加载流程

```
【服务启动】
ResSchedServiceAbility::OnStart()
  ├─► ResSchedMgr::GetInstance().Init()
  │    └─► PluginMgr::GetInstance().Init()
  │         ├─► 读取 res_sched_plugin_switch.xml
  │         ├─► 读取 res_sched_config.xml
  │         └─► LoadPlugin(pluginName)
  │              ├─► dlopen(pluginPath)           // 加载动态库
  │              ├─► dlsym(handle, "CreatePlugin") // 获取创建函数
  │              ├─► createPlugin()               // 创建插件实例
  │              └─► plugin->OnPluginInit()       // 初始化插件
  │                   └─► PluginMgr::SubscribeResource()  // 订阅事件
  │
  └─► Publish(service_)  // 向 SA 管理器发布服务
```

---

## 调用链 5: 杀进程流程

```
【内存管理服务调用】
MemoryManager::KillProcess(pid)
  └─► ResSchedClient::GetInstance().KillProcess()
       ├─► TryConnect()
       └─► rss_->KillProcess(payload, ret)
            └─► 【SA 1901 进程】
                 ResSchedService::KillProcess()
                 ├─► 权限检查 (UID 白名单)
                 ├─► Payload 大小检查
                 └─► ResSchedMgr::GetInstance().KillProcessByClient()
                      ├─► 查找 KillReasonListener
                      └─► 验证杀进程原因
                           └─► 【SA 1918 进程】
                                ResSchedExeClient::SendRequestSync()
                                └─► ResSchedExeService::KillProcess()
                                     └─► kill(pid, SIGKILL)
```

---

## 调用链 6: 设置进程优先级 (N-API)

```
【JS 调用】
backgroundProcessManager.setProcessPriority(pid, priority)
  └─► NAPI: SetProcessPriority()
       ├─► 参数校验 (数量/类型)
       ├─► napi_get_value_int32()       // 提取参数
       ├─► OH_BackgroundProcessManager_SetProcessPriority()
       │    └─► ResSchedClient::ReportSyncEvent()
       │         ├─► TryConnect()
       │         └─► rss_->ReportSyncEvent()
       │              └─► 【SA 1901 进程】
       │                   ResSchedService::ReportSyncEvent()
       │                   ├─► 权限检查
       │                   └─► PluginMgr::DeliverResource()
       │                        └─► cgroup_sched_plugin::OnDispatchResource()
       │                             └─► CgroupAction::SetThreadGroupSched()
       │                                  └─► 写入 /dev/cpuctl/...
       │
       └─► HandleErrorCode()            // 处理错误码
```

---

## 代码证据

### ReportData 调用链入口

```cpp
// 文件: ressched/services/resschedservice/src/res_sched_service.cpp:326

ErrCode ResSchedService::ReportData(uint32_t resType, int64_t value, 
    const std::string& payload) {
    int32_t clientPid = IPCSkeleton::GetCallingPid();
    RESSCHED_LOGD("ResSchedService receive data from ipc resType: %{public}u, "
                  "value: %{public}lld, pid: %{public}d",
                  resType, (long long)value, clientPid);
    
    nlohmann::json reportDataPayload = StringToJsonObj(payload);
    reportDataPayload["callingUid"] = std::to_string(callingUid);
    reportDataPayload["clientPid"] = std::to_string(clientPid);
    
    ResSchedMgr::GetInstance().ReportData(resType, value, reportDataPayload);
    return ERR_OK;
}
```

### PluginMgr 事件分发

```cpp
// 文件: ressched/services/resschedmgr/resschedfwk/src/plugin_mgr.cpp

void PluginMgr::DeliverResource(const std::shared_ptr<ResData>& data) {
    std::lock_guard<std::mutex> lock(mutex_);
    
    // 获取订阅该资源类型的插件列表
    auto pluginList = resTypePluginMap_[data->resType];
    
    for (const auto& libName : pluginList) {
        auto plugin = pluginLibMap_[libName];
        if (plugin) {
            auto startTime = GetNowMicroTime();
            
            plugin->OnDispatchResource(data);
            
            // 检查处理时间
            auto duration = GetNowMicroTime() - startTime;
            if (duration > ERROR_TIME) {  // 10ms
                RESSCHED_LOGE("Plugin %s process time exceed 10ms", libName.c_str());
            }
        }
    }
}
```

---

## 相关链接

- [架构设计](01_Architecture.md) - 组件图
- [内部 API](04_Inner_API.md) - 接口说明
- [N-API 参考](03_NAPI_Reference.md) - JS 接口
