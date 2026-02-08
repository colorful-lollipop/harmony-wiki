# 附录：关键调用链

## 播放流程调用链（Source → Sink）

```
应用层
    │
    ├── AudioRenderer::Write()
    │
    ▼
音频框架
    │
    ├── AudioRendererImpl::Write()
    │
    ▼
HDF 驱动层
    │
    ├── 虚拟扬声器设备写入
    │
    ▼
DAudioHdiHandler::WriteData()  [services/audiohdiproxy/src/daudio_hdi_handler.cpp]
    │
    ├── 通过回调转发
    │
    ▼
DSpeakerDev::OnWriteData()  [services/audiomanager/managersource/src/dspeaker_dev.cpp]
    │
    ├── 写入环形缓冲区
    │
    ▼
DSpeakerDev::EnqueueThread()  [services/audiomanager/managersource/src/dspeaker_dev.cpp]
    │
    ├── 从缓冲区读取数据
    │
    ▼
AVTransSenderTransport::FeedAudioData()  [services/audiotransport/senderengine/src/av_sender_engine_transport.cpp]
    │
    ├── 通过 AV 引擎发送
    │
    ▼
软总线 (dsoftbus)
    │
    ├── 网络传输
    │
    ▼
对端设备
    │
    ├── AVTransReceiverTransport::OnEngineDataAvailable()
    │
    ▼
DSpeakerClient::OnEngineTransDataAvailable()  [services/audioclient/spkclient/src/dspeaker_client.cpp]
    │
    ├── 写入抖动队列
    │
    ▼
DSpeakerClient::PlayThreadRunning()  [services/audioclient/spkclient/src/dspeaker_client.cpp]
    │
    ├── 从队列读取数据
    │
    ▼
AudioRenderer::Write()  [音频框架]
    │
    ▼
真实扬声器播放
```

## 录音流程调用链（Sink → Source）

```
真实麦克风采集
    │
    ▼
AudioCapturer::Read()  [音频框架]
    │
    ▼
DMicClient::OnReadData()  [services/audioclient/micclient/src/dmic_client.cpp]
    │
    ├── 读取音频数据
    │
    ▼
DMicClient::CaptureThreadRunning()  [services/audioclient/micclient/src/dmic_client.cpp]
    │
    ├── 发送到传输层
    │
    ▼
AVTransSenderTransport::FeedAudioData()  [编码发送]
    │
    ▼
软总线传输
    │
    ▼
AVTransReceiverTransport::OnEngineDataAvailable()  [解码接收]
    │
    ▼
DMicDev::OnMicDataReceived()  [services/audiomanager/managersource/src/dmic_dev.cpp]
    │
    ├── 写入数据
    │
    ▼
DAudioHdiHandler::ReadData()  [HDI 读取]
    │
    ▼
HDF 虚拟麦克风
    │
    ▼
音频框架读取
    │
    ▼
应用层读取
```

## 设备启用调用链

```
分布式硬件框架
    │
    ├── 调用 RegisterDistributedHardware
    │
    ▼
DAudioSourceHandler::RegisterDistributedHardware()  [interfaces/inner_kits/native_cpp/audio_source/src/daudio_source_handler.cpp]
    │
    ├── 获取 Source 服务
    │
    ▼
DAudioSourceProxy::RegisterDistributedHardware()  [interfaces/inner_kits/native_cpp/audio_source/src/daudio_source_proxy.cpp]
    │
    ├── IPC 调用
    │
    ▼
DAudioSourceStub::OnRemoteRequest()  [services/audiomanager/servicesource/src/daudio_source_stub.cpp]
    │
    ├── 验证权限
    │
    ▼
DAudioSourceService::RegisterDistributedHardware()  [services/audiomanager/servicesource/src/daudio_source_service.cpp]
    │
    ├── 转发到管理器
    │
    ▼
DAudioSourceManager::EnableDAudio()  [services/audiomanager/managersource/src/daudio_source_manager.cpp]
    │
    ├── 创建设备实例
    │
    ▼
DAudioSourceDev::AwakeAudioDev()  [services/audiomanager/managersource/src/daudio_source_dev.cpp]
    │
    ├── 创建 EventHandler
    │
    ▼
DAudioSourceDev::TaskEnableDAudio()  [异步任务]
    │
    ├── 注册 HDF 设备
    │
    ▼
DAudioHdiHandler::RegisterAudioDevice()  [services/audiohdiproxy/src/daudio_hdi_handler.cpp]
    │
    ├── 调用 HDF 接口
    │
    ▼
HDF 驱动注册虚拟设备
    │
    ▼
音频框架发现新设备
    │
    ▼
回调通知注册成功
```

## 控制指令调用链

```
Source 端
    │
    ├── 用户调节音量
    │
    ▼
DAudioSourceDev::HandleVolumeSet()  [services/audiomanager/managersource/src/daudio_source_dev.cpp]
    │
    ├── 发送控制事件
    │
    ▼
DaudioSourceCtrlTrans::SendAudioEvent()  [services/audiotransport/audioctrltransport/src/daudio_source_ctrl_trans.cpp]
    │
    ├── 通过软总线发送
    │
    ▼
软总线通道
    │
    ▼
对端设备
    │
    ├── DaudioSinkCtrlTrans::OnChannelEvent()
    │
    ▼
DAudioSinkDev::HandleVolumeSet()  [services/audiomanager/managersink/src/daudio_sink_dev.cpp]
    │
    ├── 通知本地音频框架
    │
    ▼
实际调节 Sink 端音量
```

## 错误处理调用链

```
错误发生
    │
    ├── 记录错误日志 (DHLOGE)
    │
    ▼
daudio_hisysevent.cpp 上报事件
    │
    ├── DAUDIO_INIT_FAIL / DAUDIO_REGISTER_FAIL
    │
    ▼
通知回调层
    │
    ├── OnNotifyRegResult(status=ERROR)
    │
    ▼
分布式硬件框架处理错误
    │
    ▼
清理资源
```

## IPC 调用详细流程

```
客户端
    │
    ├── DAudioSourceProxy::InitSource(params, callback)
    │
    ▼
序列化数据
    │
    ├── WriteString(params)
    ├── WriteRemoteObject(callback)
    │
    ▼
发送 IPC 请求 (code = INIT_SOURCE)
    │
    ▼
服务端
    │
    ├── DAudioSourceStub::OnRemoteRequest(code, data, reply)
    │
    ▼
反序列化数据
    │
    ├── ReadString(params)
    ├── ReadRemoteObject(callback)
    │
    ▼
调用业务实现
    │
    ├── DAudioSourceService::InitSource(params, callback)
    │
    ▼
返回结果
    │
    ├── reply.WriteInt32(result)
    │
    ▼
客户端接收结果
```

## 数据传输时序

```
时间轴:

Source 端:
T0: HDF Write ──┐
T1:             ├── DSpeakerDev 处理 ──┐
T2:                                    ├── 编码 ──┐
T3:                                             ├── 网络发送

网络传输: ────────────────────────────────────────────►

Sink 端:
T4:                                             ◄── 网络接收 ──┐
T5:                                    ◄── 解码 ──┤
T6:             ◄── DSpeakerClient 处理 ──┤
T7: ◄── AudioRenderer Write               │
T8: ◄── 扬声器播放                        │
```

---

*文档生成时间: 2025-02-06*
