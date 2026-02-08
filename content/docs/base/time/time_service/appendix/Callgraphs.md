# 附录：调用链分析

> 关键功能的完整调用链

---

## setTime 调用链

```
JS: systemTime.setTime(time)
│
├─> NAPI: JSSystemTimeSetTime()
│   └─> ParseParametersBySetTime()      // 参数解析
│   └─> TimePaddingAsyncCallbackInfo()  // Promise/Callback 设置
│   └─> napi_create_async_work()        // 创建异步任务
│       └─> 执行器：TimeServiceClient::SetTime()
│           └─> GetProxy()
│               └─> SystemAbilityManager::GetSystemAbility(3702)
│           └─> ITimeService::SetTime()  // IPC 调用
│               └─> 服务端进程
│
服务端进程:
└─> TimeSystemAbility::SetTime()
    ├─> TimePermission::CheckSystemUidCallingPermission()  // 系统应用检查
    ├─> TimePermission::CheckCallingPermission(SET_TIME)   // 权限检查
    └─> SetTimeInner()
        └─> SetRealTime()
            ├─> IsValidTime()           // 时间有效性检查
            ├─> TimeUtils::GetWallTimeMs()  // 获取当前时间
            ├─> settimeofday()          // 系统调用：设置时间
            ├─> SetRtcTime()
            │   ├─> fopen(/dev/rtcX)    // 打开 RTC 设备
            │   ├─> gmtime_r()          // 转换时间格式
            │   └─> ioctl(RTC_SET_TIME) // 设置 RTC
            └─> TimeServiceNotify::PublishTimeChangeEvents()  // 发布事件
```

---

## createTimer 调用链

```
JS: systemTimer.createTimer(options)
│
├─> NAPI: CreateTimer()
│   └─> GetTimerOptions()               // 解析 options
│       └─> ParseTimerOptions()         // 字段校验
│   └─> NapiWork::AsyncEnqueue()        // 异步执行
│       └─> TimeServiceClient::CreateTimerV9()
│           └─> ITimeService::CreateTimer()  // IPC
│
服务端进程:
└─> TimeSystemAbility::CreateTimer()
    ├─> CheckSystemUidCallingPermission()  // 权限检查
    ├─> CreateTimer(timerInfo, callback, timerId)
    │   ├─> ParseTimerPara()             // 解析定时器参数
    │   │   └─> 计算 timerType (RTC/ELAPSED/WAKEUP)
    │   │   └─> 计算 flag (EXACT/IDLE/...)
    │   ├─> CheckTimerPara()             // 参数校验
    │   └─> TimerManager::CreateTimer()
    │       ├─> 生成 timerId
    │       ├─> CreateAppTimer()
    │       │   └─> new TimerInfo()      // 创建定时器对象
    │       └─> InsertTimerBundleNameMap()  // 保存映射
    └─> 返回 timerId
```

---

## 定时器触发回调链

```
内核层:
└─> timerfd 到期
    └─> epoll_wait 唤醒 (TimerHandler::WaitForAlarm)
        └─> TimerHandler::Callback()
            └─> TimerManager::OnTrigger()
                └─> 主循环处理

TimerManager::TimerLooper():
├─> 从 epoll 读取事件
├─> 获取到期定时器列表
├─> 遍历触发：
│   └─> timer->OnTrigger()
│       ├─> 执行回调函数
│       │   └─> ITimerCallback::NotifyTimer()  // IPC 回调
│       │       └─> 客户端进程
│       │           └─> JS 回调函数执行
│       └─> 如果是 repeat 定时器：
│           └─> 计算下次触发时间
│           └─> 重新设置 timerfd
└─> 继续 epoll_wait

客户端进程（JS 层）:
└─> ITimerInfoInstance::OnTrigger()
    └─> napi_send_event()      // 发送到 JS 线程
        └─> napi_call_function()  // 执行 JS 回调
```

---

## 定时器代理（后台冻结）调用链

```
系统框架层 (Resource Schedule):
└─> 应用进入后台
    └─> 调用 TimeServiceClient::ProxyTimer()
        └─> ITimeService::ProxyTimer()  // IPC
            └─> TimeSystemAbility::ProxyTimer()
                ├─> CheckProxyCallingPermission()  // Native/Shell 检查
                └─> TimerManager::ProxyTimer()
                    └─> TimerProxy::ProxyTimer()
                        ├─> 将定时器加入代理列表
                        ├─> 调整触发时间（延迟）
                        └─> UpdateTimerElapsed()  // 更新定时器状态

应用恢复前台:
└─> TimeServiceClient::ProxyTimer(uid, pids, false, needRetrigger)
    └─> TimerProxy::ProxyTimer(uid, pids, isProxy=false, ...)
        ├─> 从代理列表移除
        └─> 如果 needRetrigger：
            └─> 立即触发被延迟的定时器
```

---

## NTP 时间同步调用链

```
初始化时:
TimeSystemAbility::OnStart()
└─> NtpUpdateTime::Init()
    └─> 注册网络变化监听

网络变化时:
└─> EventManager::OnReceiveEvent()
    └─> NtpUpdateTime::SetSystemTime()
        └─> 创建线程：
            └─> NtpUpdateTime::RefreshNtpTime()
                ├─> NtpTrustedTime::ForceRefresh()
                │   └─> SNTPClient::RequestTime()
                │       ├─> getaddrinfo()      // 解析 NTP 服务器
                │       ├─> socket()            // 创建 UDP socket
                │       ├─> send()              // 发送 NTP 请求
                │       ├─> recv()              // 接收 NTP 响应
                │       └─> ParseNtpResponse()  // 解析响应
                └─> 如果时间有效：
                    └─> TimeServiceClient::SetTime()  // 设置系统时间
```

---

## 时区设置调用链

```
JS: systemTime.setTimezone(timezoneId)
│
├─> NAPI: JSSystemTimeSetTimeZone()
│   └─> TimeServiceClient::SetTimeZoneV9()
│       └─> ITimeService::SetTimeZone()  // IPC
│
服务端进程:
└─> TimeSystemAbility::SetTimeZone()
    ├─> CheckSystemUidCallingPermission()  // 系统应用检查
    ├─> CheckCallingPermission(SET_TIME_ZONE)  // 权限检查
    └─> SetTimeZoneInner()
        └─> TimeZoneInfo::SetTimezone()
            ├─> CheckTimeZoneId()       // 验证时区 ID 有效性
            ├─> settimeofday(nullptr, &tz)  // 系统调用：设置时区
            ├─> SaveTimeZoneToFile()     // 持久化到时区文件
            └─> TimeServiceNotify::PublishTimeZoneChangeEvents()  // 发布事件
```

---

## 服务启动调用链

```
系统启动:
└─> init 进程
    └─> 解析 timeservice.cfg
        └─> 启动 timeservice 进程
            └─> 加载 libtime_system_ability.z.so
                └─> REGISTER_SYSTEM_ABILITY_BY_ID(TimeSystemAbility, 3702)
                    └─> 注册到 SystemAbilityManager

TimeSystemAbility::OnStart():
├─> TimerManager::GetInstance()           // 初始化定时器管理器
├─> TimeTickNotify::GetInstance().Init()  // 初始化时间滴答通知
├─> TimeZoneInfo::GetInstance().Init()    // 初始化时区信息
├─> NtpUpdateTime::GetInstance().Init()   // 初始化 NTP 同步
├─> AddSystemAbilityListener()            // 监听依赖的 SA
│   ├─> COMMON_EVENT_SERVICE_ID
│   ├─> DEVICE_STANDBY_SERVICE_SYSTEM_ABILITY_ID
│   └─> ...
├─> InitDumpCmd()                         // 注册 HIDumper 命令（如启用）
└─> Publish(TimeSystemAbility::GetInstance())  // 发布服务
```

---

## 相关链接

- [架构说明](./../02_Architecture.md) - 组件关系
- [内部 API](./../04_Inner_API.md) - C++ 接口
- [N-API 参考](./../03_NAPI_Reference.md) - JS 接口
