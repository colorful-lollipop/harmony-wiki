# 附录 2_关键调用链

## 1. 视频解码调用链

### 创建到启动

```
用户代码
    │
    ▼
OH_VideoDecoder_CreateByMime("video/avc")
    │
    ├── native_video_decoder.cpp:Create()
    │       │
    │       └── AVCodecClient::Create()
    │               │
    │               └── IPC: CodecServiceProxy::Init()
    │                       │
    │                       └── [IPC 调用] av_codec_service
    │                               │
    │                               └── CodecServiceStub::Init()
    │                                       │
    │                                       └── Engine: VideoDecoder::Init()
    │
    ▼
OH_VideoDecoder_Configure(codec, format)
    │
    ├── native_video_decoder.cpp:Configure()
    │       │
    │       └── IPC: CodecServiceProxy::Configure()
    │               │
    │               └── [IPC 调用]
    │                       │
    │                       └── CodecServiceStub::Configure()
    │
    ▼
OH_VideoDecoder_Prepare(codec)
    │
    ├── native_video_decoder.cpp:Prepare()
    │       │
    │       └── IPC: CodecServiceProxy::Prepare()
    │               │
    │               └── [IPC 调用]
    │                       │
    │                       └── CodecServiceStub::Prepare()
    │                               │
    │                               └── Engine: VideoDecoder::Prepare()
    │
    ▼
OH_VideoDecoder_Start(codec)
    │
    └── 触发回调: OH_AVCodecOnNeedInputBuffer()
```

### 数据处理循环

```
1. 获取输入 Buffer
   OH_VideoDecoder_GetInputBuffer(codec, index)
        │
        └── Fill input data (用户代码)
              │
              ▼
2. 提交输入
   OH_VideoDecoder_PushInputBuffer(codec, index)
        │
        ├── IPC: CodecServiceProxy::PushInputBuffer()
        │       │
        │       └── [IPC 调用]
        │               │
        │               └── CodecServiceStub::PushInputBuffer()
        │                       │
        │                       └── Engine: VideoDecoder::QueueInputBuffer()
        │
        └── 触发回调: OH_AVCodecOnOutputBufferAvailable()
              │
              ▼
3. 处理输出
   OH_VideoDecoder_RenderOutputBuffer(codec, index)
        │
        └── 渲染到 Surface 或释放
```

---

## 2. 解封装调用链

```
用户代码
    │
    ▼
OH_AVSource_CreateWithDataSource(url)
    │
    ├── native_avsource.cpp
    │       │
    │       └── IPC: SourceServiceProxy::Init()
    │               │
    │               └── [IPC 调用]
    │
    ▼
OH_AVDemuxer_CreateWithSource(source)
    │
    ├── native_avdemuxer.cpp
    │       │
    │       └── IPC: DemuxerServiceProxy::Init()
    │               │
    │               └── [IPC 调用]
    │
    ▼
OH_AVDemuxer_SelectTrackByID(demuxer, trackId)
    │
    └── IPC: DemuxerServiceProxy::SelectTrackByID()
            │
            └── [IPC 调用]
                    │
                    └── CodecServiceStub::SelectTrack()
                            │
                            └── MediaDemuxer::SelectTrack()
                                    │
                                    └── 解复用器插件
                                            │
                                            └── FFmpegDemuxer
```

---

## 3. 封装调用链

```
用户代码
    │
    ▼
OH_AVMuxer_Create(fd)
    │
    ├── native_avmuxer.cpp
    │       │
    │       └── IPC: MuxerServiceProxy::Init()
    │               │
    │               └── [IPC 调用]
    │
    ▼
OH_AVMuxer_AddTrack(muxer, trackDesc)
    │
    └── IPC: MuxerServiceProxy::AddTrack()
            │
            └── [IPC 调用]
                    │
                    └── CodecServiceStub::AddTrack()
                            │
                            └── MediaMuxer::AddTrack()
                                    │
                                    └── 封装器插件
                                            │
                                            └── FFmpegMuxer
    │
    ▼
OH_AVMuxer_Start(muxer)
    │
    └── IPC: MuxerServiceProxy::Start()
            │
            └── [IPC 调用]
                    │
                    └── CodecServiceStub::Start()
    │
    ▼
OH_AVMuxer_WriteSampleBuffer(muxer, trackId, buffer)
    │
    └── IPC: MuxerServiceProxy::WriteSample()
            │
            └── [IPC 调用]
                    │
                    └── CodecServiceStub::WriteSample()
                            │
                            └── MediaMuxer::WriteSample()
```

---

## 4. 错误回调路径

```
编解码引擎内部错误
        │
        ├── CodecServiceStub::OnError()
        │       │
        │       └── IPC: CodecListenerProxy::OnError()
        │               │
        │               └── [IPC 调用回调]
        │                       │
        │                       └── CodecListenerStub::OnRemoteRequest(ON_ERROR)
        │                               │
        │                               └── native_*codec.cpp:ErrorCallback()
        │                                       │
        │                                       └── 用户回调: OH_AVCodecOnError()
        │
        └── CodecServiceStub::OnOutputFormatChanged()
                │
                └── 用户回调: OH_AVCodecOnStreamChanged()
```

---

**相关文档**: [首页](README.md) | [架构设计](03_Architecture.md) | [对外 C-API](04_C_API.md)
