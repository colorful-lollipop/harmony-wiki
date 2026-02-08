# Audio Framework - 关键调用链

## JS 创建 AudioRenderer

```mermaid
sequenceDiagram
    participant JS as JavaScript
    participant NAPI as frameworks/js/napi
    participant Native as frameworks/native
    participant IPC as IPC/Binder
    participant Service as AudioServer (SA 3001)
    participant HAL as hdiadapter_new

    JS->>NAPI: createAudioRenderer(options)
    NAPI->>Native: AudioRenderer::Create(options)
    Native->>IPC: IStandardAudioService::CreateAudioProcess()
    IPC->>Service: OnRemoteRequest(CREATE_AUDIOPROCESS)
    Service->>Service: CheckPlaybackPermission()
    Service->>Service: Create AudioProcess
    Service->>HAL: Allocate hardware resources
    HAL-->>Service: Hardware allocated
    Service-->>Native: AudioProcess proxy
    Native-->>NAPI: AudioRenderer instance
    NAPI-->>JS: AudioRenderer object
```

## JS 设置音量

```mermaid
sequenceDiagram
    participant JS as JavaScript
    participant NAPI as napi_audio_manager
    participant IPC as IPC/Binder
    participant Policy as AudioPolicyServer (SA 3009)

    JS->>NAPI: setVolume(volumeType, volume)
    NAPI->>NAPI: VerifyPermission(MODIFY_AUDIO_SETTINGS)
    NAPI->>IPC: SetSystemVolumeLevel()
    IPC->>Policy: OnRemoteRequest(SET_SYSTEM_VOLUMELEVEL)
    Policy->>Policy: Check permission
    Policy->>Policy: Update volume in memory
    Policy->>Policy: Persist to settings
    Policy->>IPC: Return result
    IPC-->>NAPI: SUCCESS
    NAPI-->>JS: void
```

## 设备插拔事件流

```mermaid
sequenceDiagram
    participant HAL as Hardware
    participant Service as AudioServer (SA 3001)
    participant Policy as AudioPolicyServer (SA 3009)
    participant Manager as AudioRoutingManager
    participant Client as Application

    HAL->>Service: Device state change interrupt
    Service->>Service: Handle PnP event
    Service->>Policy: NotifyDeviceInfo()
    Policy->>Policy: Update device status
    Policy->>Policy: Re-evaluate routing
    Policy->>Manager: Notify device change
    Manager->>Client: on('deviceChange', callback)
```

## 音频播放数据流

```mermaid
graph LR
    A[JS Write] --> B[NAPI write]
    B --> C[AudioRenderer]
    C --> D[AudioProcess]
    D --> E[AudioServer]
    E --> F[PlaybackEngine]
    F --> G[Pipeline]
    G --> H[HDI Adapter]
    H --> I[HAL/Sink]
```

## 音频采集数据流

```mermaid
graph LR
    A[HAL/Source] --> B[HDI Adapter]
    B --> C[AudioCapturerInServer]
    C --> D[AudioProcess]
    D --> E[AudioServer]
    E --> F[IPC]
    F --> G[NAPI Read]
    G --> H[JS Read]
```

## 权限验证调用链

```mermaid
sequenceDiagram
    participant App as Application
    participant NAPI as PermissionUtil
    participant Token as AccessTokenKit
    participant IPC as IPCSkeleton

    App->>NAPI: VerifyPermission(permName)
    NAPI->>IPCSkeleton: GetCallingTokenID()
    IPCSkeleton-->>NAPI: tokenId
    NAPI->>Token: VerifyAccessToken(tokenId, permName)
    Token-->>NAPI: PERMISSION_GRANTED/DENIED
    NAPI-->>App: true/false
```

## 焦点/中断管理

```mermaid
sequenceDiagram
    participant App1 as App A (播放)
    participant App2 as App B (通话)
    participant Manager as AudioInterruptManager
    participant Policy as AudioPolicyServer

    App1->>Manager: RequestInterrupt(info)
    Manager->>Policy: ActivateInterrupt(info)
    Policy->>Policy: Evaluate focus request
    Policy->>Policy: Grant focus to App1
    Policy-->>Manager: INTERRUPT_HINT_DENIED
    Manager-->>App1: AudioInterruptEvent(DENIED)

    App2->>Manager: RequestInterrupt(info)
    Manager->>Policy: ActivateInterrupt(info)
    Policy->>Policy: Phone call has priority
    Policy->>Policy: Interrupt App1
    Policy->>Manager: INTERRUPT_HINT_STOP
    Manager-->>App1: AudioInterruptEvent(STOP)
```

## 相关文档

- [N-API 接口](../03_NAPI.md)
- [架构设计](../02_Architecture.md)
- [安全机制](../05_Security.md)
