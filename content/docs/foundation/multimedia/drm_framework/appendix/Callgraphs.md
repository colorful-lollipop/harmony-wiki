# 关键调用链

> 本文档描述 DRM Framework 的关键调用链，从入口到核心逻辑。

## 1. 创建 MediaKeySystem

```
JS Application
    │
    ├─→ drm.createMediaKeySystem("com.example.drm")
    │       │
    │       └─→ native_module_ohos_drm.cpp:28
    │               MediaKeySystemNapi::Init()
    │
    └─→ media_key_system_napi.cpp:130
            MediaKeySystemNapi::CreateMediaKeySystemInstance()
                    │
                    ├─→ media_key_system_factory_impl.cpp:118
                    │       MediaKeySystemFactoryImpl::CreateMediaKeySystem()
                    │               │
                    │               ├─→ GetInstance() [单例]
                    │               │
                    │               └─→ IPCSkeleton::GetContextObject()
                    │                       │
                    │                       └─→ IMediaKeySystemFactoryService Proxy
                    │                               │
                    │                               └─→ IMediaKeySystemFactoryService::CreateMediaKeySystem()
                    │                                       │
    ────────────────────────────────────────────────────────────── IPC ──→ SA Process (3012)
                    │
                    └─→ mediakeysystemfactory_service.cpp:245
                            MediaKeySystemFactoryService::CreateMediaKeySystem()
                                    │
                                    ├─→ IsListenerObjectSet()
                                    │
                                    ├─→ GetMediaKeySystemForPlugin()
                                    │
                                    ├─→ IPCSkeleton::GetCallingPid()
                                    │       │
                                    │       └─→ mediaKeySystemForPid_[pid].insert()
                                    │
                                    └─→ new MediaKeySystemService()
                                            │
                                            └─→ IMediaKeySystemProxy
                                                    │
                                                    └─→ IMediaKeySystemFactory::Create()
                                                            │
    ────────────────────────────────────────────────────────────── HDI ──→ DRM Plugin
                                                                    │
                                                                    └─→ IMediaKeySystem Handle
```

## 2. 证书 Provision

```
JS Application
    │
    └─→ keySystem.generateKeySystemRequest()
            │
            └─→ media_key_system_napi.cpp:646
                    MediaKeySystemNapi::GenerateKeySystemRequest()
                            │
                            ├─→ GenerateKeySystemRequestNapi()
                            │       │
                            │       └─→ napi_create_async_work()
                            │               │
                            └─→ AsyncExecute()
                                    │
                                    └─→ mediaKeySystemImpl_->GenerateKeySystemRequest()
                                            │
                                            ├─→ serviceProxy_->GenerateKeySystemRequest()
    ────────────────────────────────────────────────────────────── IPC ──→ SA Process
                                                    │
                                                    └─→ MediaKeySystemService::GenerateKeySystemRequest()
                                                            │
                                                            ├─→ hdiKeySystem_->GenerateKeySystemRequest()
    ────────────────────────────────────────────────────────────── HDI ──→ DRM Plugin
                                                                    │
                                                                    └─→ License Request Data
                                                                            │
                                                                            └─→ 返回给应用发送至证书服务器
```

## 3. 生成密钥请求

```
JS Application
    │
    └─→ keySession.generateMediaKeyRequest(mimeType, initData, type)
            │
            └─→ key_session_napi.cpp:238
                    MediaKeySessionNapi::GenerateMediaKeyRequest()
                            │
                            ├─→ GenerateMediaKeyRequestNapi()
                            │       │
                            │       └─→ napi_create_async_work()
                            │               │
                            └─→ AsyncExecute()
                                    │
                                    └─→ keySessionImpl_->GenerateMediaKeyRequest()
                                            │
                                            ├─→ serviceProxy_->GenerateMediaKeyRequest()
    ────────────────────────────────────────────────────────────── IPC ──→ SA Process
                                                    │
                                                    └─→ MediaKeySessionService::GenerateMediaKeyRequest()
                                                            │
                                                            ├─→ hdiMediaKeySession_->GenerateMediaKeyRequest()
    ────────────────────────────────────────────────────────────── HDI ──→ DRM Plugin
                                                                    │
                                                                    └─→ MediaKeyRequest (data + defaultUrl)
                                                                            │
                                                                            └─→ 返回给应用发送至许可证服务器
```

## 4. 处理密钥响应

```
JS Application
    │
    └─→ keySession.processMediaKeyResponse(licenseResponse)
            │
            └─→ key_session_napi.cpp:287
                    MediaKeySessionNapi::ProcessMediaKeyResponse()
                            │
                            ├─→ ProcessMediaKeyResponseNapi()
                            │       │
                            │       └─→ napi_create_async_work()
                            │               │
                            └─→ AsyncExecute()
                                    │
                                    └─→ keySessionImpl_->ProcessMediaKeyResponse()
                                            │
                                            ├─→ serviceProxy_->ProcessMediaKeyResponse()
    ────────────────────────────────────────────────────────────── IPC ──→ SA Process
                                                    │
                                                    └─→ MediaKeySessionService::ProcessMediaKeyResponse()
                                                            │
                                                            ├─→ hdiMediaKeySession_->ProcessMediaKeyResponse()
    ────────────────────────────────────────────────────────────── HDI ──→ DRM Plugin
                                                                    │
                                                                    └─→ License ID
                                                                            │
                                                                            └─→ 返回给应用
```

## 5. 媒体解密

```
Media Player
    │
    └─→ decryptModule.decrypt(encryptedBuffer)
            │
            └─→ media_decrypt_module_service.cpp:39
                    MediaDecryptModuleService::DecryptMediaData()
                            │
                            ├─→ SetCryptInfo()
                            │
                            ├─→ SetDrmBufferInfo()
                            │
                            └─→ hdiMediaDecryptModule_->DecryptMediaData()
    ────────────────────────────────────────────────────────────── HDI ──→ DRM Plugin
                                    │
                                    └─→ Decrypted Buffer
                                            │
                                            └─→ 返回给播放器解码播放
```

## 6. 事件回调

```
DRM Plugin Event
        │
        ├─→ HDI Callback Interface
        │       │
        │       └─→ IMediaKeySessionCallback::SendEvent()
        │               │
    ────────────────────────────────────────────────────────────── IPC ──→ SA Process
                        │
                        └─→ MediaKeySessionServiceCallbackStub::SendEvent()
                                │
                                ├─→ callback_->SendEvent()
                                │       │
                                │       └─→ IPC Proxy
                                │               │
    ────────────────────────────────────────────────────────────── IPC ──→ App Process
                                        │
                                        └─→ MediaKeySessionServiceCallback::SendEvent()
                                                │
                                                ├─→ eventQueue.push()
                                                │
                                                └─→ eventQueueThread.ProcessEventMessage()
                                                        │
                                                        └─→ napi_send_event()
                                                                │
                                                                └→ JS Callback (主线程)
```

## 调用链统计

| 调用链 | IPC 次数 | HDI 次数 | 关键路径 |
|--------|----------|----------|----------|
| CreateMediaKeySystem | 2 | 1 | Factory → SA |
| GenerateKeySystemRequest | 1 | 1 | System → Plugin |
| GenerateMediaKeyRequest | 1 | 1 | Session → Plugin |
| ProcessMediaKeyResponse | 1 | 1 | Session → Plugin |
| DecryptMediaData | 0 | 1 | 仅 HDI |
| 事件回调 | 1 | 0 | SA → App |
