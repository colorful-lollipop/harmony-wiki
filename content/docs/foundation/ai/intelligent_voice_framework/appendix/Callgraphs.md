# 关键调用链

> **目的**: 记录 Intelligent Voice Framework 的关键调用路径  
> **适用范围**: 调试、问题定位、性能分析  
> **最后更新**: 2026-02-06

---

## 1. 引擎创建调用链

### 1.1 EnrollIntelligentVoiceEngine 创建

```
JS 应用
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ intelligentVoice.createEnrollIntelligentVoiceEngine()            │
│ 路径: interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts          │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ EnrollIntellVoiceEngineNapi::CreateEnrollIntelligentVoiceEngine │
│ 路径: frameworks/js/napi/enroll_intell_voice_engine_napi.cpp     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ CreateEnrollIntelligentVoiceEngineWrapper()                      │
│ 异步上下文创建                                                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ IntellVoiceManager::CreateIntellVoiceEngine()                   │
│ 路径: frameworks/native/intell_voice_manager.cpp                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ IPC (Binder)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ IIntellVoiceService::CreateIntellVoiceEngine() [IPC Stub]       │
│ 路径: services/intell_voice_service/server/sa/intell_voice_      │
│       service_stub.cpp                                           │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ IntellVoiceService::CreateIntellVoiceEngine()                    │
│ 路径: services/intell_voice_service/server/sa/intell_voice_      │
│       service.cpp                                                │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ IntellVoiceEngineManager::CreateEngine()                         │
│ 路径: services/intell_voice_engine/server/manager/               │
│       intell_voice_engine_manager.cpp                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ EngineFactory::CreateIntellVoiceEngine()                         │
│ 路径: services/intell_voice_engine/server/base/engine_factory.cpp│
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ EnrollEngine::EnrollEngine()                                     │
│ 路径: services/intell_voice_engine/server/enroll/enroll_engine.cpp│
```

### 1.2 WakeupIntelligentVoiceEngine 创建

```
JS 应用
    │
    ▼
intelligentVoice.createWakeupIntelligentVoiceEngine()
    │
    ▼
WakeupIntellVoiceEngineNapi::CreateWakeupIntelligentVoiceEngine()
    │
    ▼
IntellVoiceManager::CreateIntellVoiceEngine(WAKEUP_ENGINE_TYPE)
    │
    │ IPC
    ▼
IntellVoiceService::CreateIntellVoiceEngine()
    │
    ▼
IntellVoiceEngineManager::CreateEngine()
    │
    ▼
EngineFactory::CreateIntellVoiceEngine()
    │
    ▼
WakeupEngine::WakeupEngine()
```

---

## 2. 引擎初始化调用链

```
JS: engine.init(config)
    │
    ▼
EnrollIntellVoiceEngineNapi::Init()
    │
    ▼
EnrollIntellVoiceEngine::Init(config)
    │
    │ IPC
    ▼
IIntellVoiceEngine::Init() [Proxy]
    │
    ▼
IIntellVoiceEngineStub::OnRemoteRequest(ENGINE_INIT)
    │
    ▼
EnrollEngine::Init()
    │
    ▼
HDI Adapter
    │
    ▼
DSP Driver (libintell_voice_engine_proxy)
```

---

## 3. 语音注册调用链

```
JS: engine.enrollForResult(isLast)
    │
    ▼
EnrollIntellVoiceEngineNapi::EnrollForResult()
    │
    ▼
EnrollIntellVoiceEngine::EnrollForResult()
    │
    │ IPC
    ▼
IIntellVoiceEngine::Start() [Proxy]
    │
    ▼
EnrollEngine::Start()
    │
    ▼
AudioSource::StartCapture()  ← 麦克风权限检查
    │
    ▼
HDI: Start()
    │
    ▼
DSP Driver: 音频采集
    │
    ▼
DSP Event Callback
    │
    ▼
EnrollEngine::OnCallback()
    │
    ▼
IPC: IIntellVoiceEngineCallback::OnCallback()
    │
    ▼
IPC Proxy → Stub
    │
    ▼
IntellVoiceManager::OnCallback()
    │
    ▼
N-API Callback
    │
    ▼
JS: EnrollCallbackInfo 返回
```

**关键点**:
- 麦克风权限在 `AudioSource::StartCapture()` 检查
- 隐私使用记录通过 `PrivacyKit` 追踪

---

## 4. 语音唤醒调用链

```
DSP Hardware (低功耗监听)
    │
    ▼
DSP Driver Event
    │
    ▼
TriggerDetector::OnTriggerEvent()
    │
    ▼
TriggerConnector::OnDetected()
    │
    ▼
TriggerManager::OnWakeupDetected()
    │
    ▼
IntellVoiceService::OnWakeupEvent()
    │
    ▼
WakeupEngine::OnWakeupEvent()
    │
    ▼
IPC: WakeupEventCallback
    │
    ▼
WakeupIntellVoiceEngineNapi::OnWakeupEvent()
    │
    ▼
JS: engine.on('wakeupIntelligentVoiceEvent', callback)
```

---

## 5. 权限验证调用链

```
JS API 调用
    │
    ▼
N-API 入口函数
    │
    ▼
IntellVoiceCommonNapi::CheckIsSystemApp()
    │
    │ IPCSkeleton::GetCallingFullTokenID()
    ▼
TokenIdKit::IsSystemAppByFullTokenID()
    │
    ├─→ ✅ 系统应用 → 继续执行
    └─→ ❌ 非系统应用 → 返回错误 202
```

```
JS API 调用 (需要权限)
    │
    ▼
IntellVoiceUtil::VerifyClientPermission()
    │
    │ IPCSkeleton::GetCallingTokenID()
    ▼
AccessTokenKit::VerifyAccessToken()
    │
    ├─→ ✅ 权限已授予 → 继续执行
    └─→ ❌ 权限被拒绝 → 返回错误 201
```

---

## 6. 隐私权限记录调用链

```
引擎启动 (StartCapture)
    │
    ▼
IntellVoiceUtil::RecordPermissionPrivacy()
    │
    ├─→ PrivacyKit::StartUsingPermission()
    │
    └─→ PrivacyKit::AddPermissionUsedRecord()

引擎停止 (StopCapture)
    │
    ▼
IntellVoiceUtil::RecordPermissionPrivacy()
    │
    └─→ PrivacyKit::StopUsingPermission()
```

---

## 7. 服务生命周期调用链

### 7.1 SA 启动

```
系统启动
    │
    ▼
SA Framework (SA 312 未就绪)
    │
    │ 按需请求 (首次 API 调用)
    ▼
SAMgr::StartSystemAbility(312)
    │
    ▼
intell_voice_service 进程启动
    │
    ▼
intell_voice_service.cpp:main()
    │
    ▼
IntellVoiceService::OnStart()
    │
    ├─→ Init()
    ├─→ RegisterPermissionCallback()
    └─→ SystemEventObserver::Subscribe()
    │
    ▼
SA 就绪 (312)
```

### 7.2 SA 死亡与恢复

```
SA 进程崩溃/被杀
    │
    ▼
SA Framework 检测
    │
    ▼
auto-restart: true → 自动重启
    │
    ▼
intell_voice_service 进程重启
    │
    ▼
重新初始化
```

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [架构设计](./02_Architecture.md) | 组件关系 |
| [N-API 参考](./04_NAPI_Reference.md) | API 调用 |
| [安全评审](./08_Security_Review.md) | 安全机制 |
