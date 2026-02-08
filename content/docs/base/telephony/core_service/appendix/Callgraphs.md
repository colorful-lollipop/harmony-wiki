# 附录：关键调用链

## 目的

本文档提供 `telephony_core_service` 关键功能的调用链追踪。

---

## 1. SIM 状态获取调用链

```
JS: sim.getSimState(slotId)
│
├── frameworks/js/sim/src/napi_sim.cpp
│   └── GetSimState() [line ~2500]
│       └── NapiCreateAsyncWork<SimStateContext, NativeGetSimState, GetSimStateCallback>()
│           ├── NativeGetSimState (execute)
│           │   └── DelayedRefSingleton<CoreServiceClient>::GetInstance().GetSimState(slotId, callback)
│           │       └── frameworks/native/src/core_service_client.cpp
│           │           └── GetSimState() [line ~200]
│           │               └── GetProxy()->GetSimState(slotId, callback)
│           │                   └── IPC -> services/core/src/core_service_stub.cpp
│           │                       └── OnRemoteRequest() [line ~100]
│           │                           └── CoreService::GetSimState(slotId, callback)
│           │                               └── services/core/src/core_service.cpp [line ~300]
│           │                                   └── simManager_->GetSimState(slotId)
│           │                                       └── services/sim/src/sim_state_manager.cpp
│           │                                           └── 返回缓存状态或查询 RIL
│           └── GetSimStateCallback (complete)
│               └── napi_resolve_deferred / napi_call_function
│
└── 返回: SimState (UNKNOWN/NOT_PRESENT/READY/LOCKED)
```

---

## 2. 网络状态获取调用链

```
JS: radio.getNetworkState(slotId)
│
├── frameworks/js/network_search/src/napi_radio.cpp
│   └── GetNetworkState() [line ~1500]
│       └── NativeGetNetworkState (execute)
│           └── DelayedRefSingleton<CoreServiceClient>::GetInstance().GetNetworkState(slotId, networkState)
│               └── IPC -> CoreService::GetNetworkState()
│                   └── services/core/src/core_service.cpp [line ~400]
│                       └── networkSearchManager_->GetNetworkState(slotId, networkState)
│                           └── services/network_search/src/network_search_manager.cpp
│                               └── networkSearchState_->GetNetworkStatus()
│                                   └── 返回 NetworkState 对象
│       └── GetNetworkStateCallback (complete)
│           └── 构造 NetworkState JS 对象
│
└── 返回: NetworkState {regStatus, psRadioTech, csRadioTech, operatorName, ...}
```

---

## 3. 射频开关调用链

```
JS: radio.turnOnRadio(slotId)
│
├── frameworks/js/network_search/src/napi_radio.cpp
│   └── TurnOnRadio()
│       └── NativeSetRadioState (execute)
│           └── CoreServiceClient::SetRadioState(slotId, isOn, callback)
│               └── IPC -> CoreService::SetRadioState()
│                   └── services/core/src/core_service.cpp [line ~500]
│                       └── AsyncNetSearchExecute([=]() { ... })
│                           └── networkSearchManager_->SetRadioState()
│                               └── services/network_search/src/network_search_manager.cpp
│                                   └── telRilManager_->SetRadioState()
│                                       └── services/tel_ril/src/tel_ril_modem.cpp
│                                           └── HDI -> RIL Adapter
│       └── SetRadioStateCallback (complete)
│           └── 回调通知设置结果
│
└── 返回: void (异步回调)
```

---

## 4. IMEI 获取调用链

```
JS: radio.getIMEI(slotId)
│
├── frameworks/js/network_search/src/napi_radio.cpp
│   └── GetImei()
│       └── NativeGetImei (execute)
│           └── CoreServiceClient::GetImei(slotId, callback)
│               └── IPC -> CoreService::GetImei()
│                   └── services/core/src/core_service.cpp [line ~600]
│                       └── telRilManager_->GetImei()
│                           └── services/tel_ril/src/tel_ril_modem.cpp
│                               └── HDI -> RIL Adapter -> Modem
│       └── GetImeiCallback (complete)
│           └── 返回 IMEI 字符串
│
└── 返回: string (IMEI)
```

---

## 5. IMS 注册状态回调注册调用链

```
JS: radio.on('imsRegStateChange', slotId, imsType, callback)
│
├── frameworks/js/network_search/src/napi_radio.cpp
│   └── RegisterImsRegInfoCallback()
│       └── napi_ims_reg_info_callback_manager.cpp
│           └── ImsRegInfoCallbackManager::RegisterImsRegInfoCallback()
│               └── CoreServiceClient::RegisterImsRegInfoCallback(slotId, imsSrvType, callback)
│                   └── IPC -> CoreService::RegisterImsRegInfoCallback()
│                       └── services/core/src/core_service.cpp
│                           └── networkSearchManager_->RegisterImsRegInfoCallback()
│                               └── services/network_search/src/network_search_manager.cpp
│                                   └── imsRegInfoCallbackMap_[slotId][imsSrvType] = callback
│
当 IMS 状态变化时:
├── ImsCoreServiceClient -> 回调
│   └── NetworkSearchManager::UpdateImsRegInfo()
│       └── 查找并调用注册的 callback
│           └── JS 回调函数被触发
│
└── 返回: void (事件监听)
```

---

## 6. CoreService 启动调用链

```
System -> SA Manager
│
└── 启动 telephony 进程
    └── sa_profile/4010.json 配置加载
        └── libtel_core_service.z.so 加载
            └── CoreService 构造函数
                ├── SystemAbility(TELEPHONY_CORE_SERVICE_SYS_ABILITY_ID, true)
                └── DECLARE_DELAYED_SINGLETON(CoreService)
            └── OnStart() [services/core/src/core_service.cpp:55]
                ├── Publish() -> 注册到 SA Manager
                ├── ffrt_set_cpu_worker_max_num()
                └── Init()
                    ├── TelRilManager::OnInit()
                    ├── SimManager::OnInit()
                    ├── EsimManager::OnInit() [if enabled]
                    ├── NetworkSearchManager::OnInit()
                    └── ImsCoreServiceClient::Init()
                └── NotifyCoreServiceReady() -> 发送广播
```

---

## 7. RIL 消息处理调用链

```
Modem -> RIL Adapter (HDF)
│
└── HDI Callback
    └── services/tel_ril/src/tel_ril_callback.cpp
        └── TelRilCallback::Process*
            ├── TelRilCallback::ProcessSimStatusChanged()
            │   └── ObserverHandler::NotifyObserver()
            │       └── SimManager::OnSimStatusChanged()
            ├── TelRilCallback::ProcessNetworkStateChanged()
            │   └── NetworkSearchManager::OnNetworkStateChanged()
            └── TelRilCallback::ProcessSignalStrengthUpdated()
                └── NetworkSearchManager::OnSignalStrengthUpdated()
```

---

## 8. eSIM Profile 下载调用链

```
JS: esim.downloadProfile(slotId, config)
│
├── frameworks/js/esim/src/napi_esim.cpp
│   └── DownloadProfile()
│       └── NativeDownloadProfile (execute)
│           └── CoreServiceClient::DownloadProfile()
│               └── IPC -> EsimManager::DownloadProfile()
│                   └── services/sim/src/esim_manager.cpp
│                       └── esimController_->DownloadProfile()
│                           └── services/sim/src/esim_controller.cpp
│                               └── 调用 ASN.1 编码
│                               └── 发送 APDU 到 RIL
│       └── DownloadProfileCallback (complete)
│
└── 返回: DownloadProfileResult
```

---

## 相关链接

- [N-API 接口](../03_NAPI_API.md)
- [内部 API](../04_Inner_API.md)
- [架构设计](../02_Architecture.md)
