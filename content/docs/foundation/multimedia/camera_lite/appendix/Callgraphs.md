# 附录 - 关键调用链

> Camera_Lite 核心调用链详细分析

---

## 调用链说明

本文档记录 Camera_Lite 组件的关键调用链，帮助理解代码执行流程。

**格式说明**:
```
函数名()
  @ 文件路径:行号
  → 调用函数1()
  → 调用函数2()
```

---

## 1. 组件初始化调用链

### 1.1 CameraKit 初始化

```
CameraKit::GetInstance()
  @ frameworks/camera_kit.cpp:30
  → CheckSelfPermission("ohos.permission.CAMERA")  [权限检查]
  → CameraKit() 构造函数
    → CameraManager::GetInstance()
      @ frameworks/camera_manager.cpp:174
      → CameraManagerImpl() 构造函数
        → CameraServiceClient::GetInstance()
          @ frameworks/binder/src/camera_service_client.cpp:31
          → CameraClient::GetInstance()
        → CameraServiceClient::InitCameraServiceClient(callback)
          → cameraClient_->InitCameraClient()
            → SAMGR_GetInstance()->GetFeatureApi()  [获取SAMGR接口]
            → CAMERA_SERVER_QUERY_SERVICE  [查询相机服务]
            → cameraClient_->GetIClientProxy()  [获取IPC代理]
            → CameraServiceClient::GetCameraIdList()  [获取相机列表]
              → IPC调用 → CameraServer::GetCameraIdList()
```

### 1.2 CameraServer 初始化

```
SAMGR_Bootstrap()  [系统启动]
  → CameraServer::InitCameraServer()
    @ services/server/src/camera_server.cpp:82
    → CameraService::GetInstance()
      @ services/impl/src/camera_service.cpp:46
      → CameraService() 构造函数
    → CameraService::Initialize()
      @ services/impl/src/camera_service.cpp:52
      → HalCameraInit()  [HAL层初始化]
        @ HAL层实现
```

---

## 2. 相机创建调用链

### 2.1 应用层创建相机

```
应用代码: cameraKit->CreateCamera(cameraId, callback, handler)
  ↓
CameraKit::CreateCamera(cameraId, callback, handler)
  @ frameworks/camera_kit.cpp:65
  → cameraManager_->CreateCamera(cameraId, callback, handler)
    @ frameworks/camera_manager.cpp:145
    → 查找 cameraMapCache_
    → cameraImpl->RegistCb(callback, handler)  [注册回调]
      @ frameworks/camera_impl.cpp:233
    → cameraServiceClient_->CreateCamera(cameraId)  [IPC调用]
      @ frameworks/binder/src/camera_service_client.cpp:300
      → IPC序列化 cameraId
      → proxy_->Invoke(CAMERA_SERVER_CREATE_CAMERA)
        @ IPC框架
        → CameraServer::CameraServerRequestHandle()
          @ services/server/src/camera_server.cpp:38
          → CameraServer::CreateCamera(req, reply)
            @ services/server/src/camera_server.cpp:150
            → ReadString(req, &sz)  [反序列化cameraId]
            → CameraService::CreateCamera(cameraId)
              @ services/impl/src/camera_service.cpp:158
              → HalCameraDeviceOpen(cameraId)  [打开硬件]
              → new CameraDevice(cameraId)  [创建设备实例]
              → device->Initialize()
                @ services/impl/src/camera_device.cpp:27
                → CodecInit()  [初始化编解码器]
              → deviceMap_.insert()  [缓存设备]
              → 返回 CAMERA_STATUS_CREATED
            → ReadRemoteObject(req, &sid)  [读取客户端回调]
            → OnCameraStatusChange(status, &sid)  [发送回调]
              @ services/server/src/camera_server.cpp:311
              → SendRequest(sid, ON_CAMERA_STATUS_CHANGE)
                → 客户端回调: CameraServiceClient::ServiceClientCallback()
                  @ frameworks/binder/src/camera_service_client.cpp:256
                  → 解析 status
                  → cameraServiceCb_->OnCameraStatusChange(cameraId, status)
                    → CameraManagerImpl::OnCameraStatusChange()
                      @ frameworks/camera_manager.cpp:49
                      → 处理 CAMERA_STATUS_CREATED
                      → cameraMapCache_[cameraId]->OnCreate(cameraId)
                        @ frameworks/camera_impl.cpp:187
                        → CameraDeviceClient::GetInstance()
                        → deviceClient_->SetCameraId(cameraId)
                        → deviceClient_->SetCameraCallback()  [设置设备回调]
                          @ frameworks/binder/src/camera_device_client.cpp:258
                        → handler_->Post(lambda)  [异步回调到应用]
                          → callback.OnCreated(*this)  [应用回调]
```

---

## 3. 预览启动调用链

### 3.1 配置相机

```
应用代码: camera->Configure(config)
  ↓
CameraImpl::Configure(config)
  @ frameworks/camera_impl.cpp:50
  → 检查 config_ != nullptr  [防止重复配置]
  → 检查 config.GetFrameStateCb() != nullptr
  → 检查 config.GetEventHandler() != nullptr
  → 检查 deviceClient_ != nullptr
  → deviceClient_->SetCameraConfig(config)  [IPC调用]
    @ frameworks/binder/src/camera_device_client.cpp:85
    → IPC序列化 cameraId
    → proxy_->Invoke(CAMERA_SERVER_SET_CAMERA_CONFIG)
      → CameraServer::SetCameraConfig(req, reply)
        @ services/server/src/camera_server.cpp:178
        → ReadString(req, &sz)  [cameraId]
        → CameraService::GetCameraDevice(cameraId)
        → device_->SetCameraConfig()
          @ services/impl/src/camera_device.cpp:38
          → 返回 MEDIA_OK
      → OnCameraConfigured(setStatus)  [发送配置完成回调]
        @ services/server/src/camera_server.cpp:357
        → SendRequest(sid, ON_CAMERA_CONFIGURED)
          → 客户端: CameraDeviceClient::DeviceClientCallback()
            @ frameworks/binder/src/camera_device_client.cpp:288
            → cameraImpl_->OnConfigured(ret, *cc)
              @ frameworks/camera_impl.cpp:71
              → config_ = &config  [保存配置]
              → handler_->Post(lambda)
                → stateCb_->OnConfigured(*this)  [应用回调]
```

### 3.2 启动预览

```
应用代码: camera->TriggerLoopingCapture(fc)
  ↓
CameraImpl::TriggerLoopingCapture(fc)
  @ frameworks/camera_impl.cpp:96
  → 检查 config_ != nullptr  [必须已配置]
  → 检查 type != FRAME_CONFIG_CAPTURE  [不支持拍照]
  → 检查是否已存在相同类型的FrameConfig
  → deviceClient_->TriggerLoopingCapture(fc)  [IPC调用]
    @ frameworks/binder/src/camera_device_client.cpp:154
    → SerilizeFrameConfig(io, fc, maxSurfaceNum)  [序列化配置]
      @ frameworks/binder/src/camera_device_client.cpp:108
      → WriteInt32(type)
      → fc.GetSurfaces()  [获取Surface列表]
      → SurfaceImpl::WriteIoIpcIo()  [序列化Surface]
      → WriteInt32(qfactor)
      → WriteInt32(streamFps)
      → WriteInt32(invertMode)
      → WriteInt32(crop.x/y/w/h)
      → WriteInt32(format)
      → fc.GetVendorParameter()  [私有参数]
    → proxy_->Invoke(CAMERA_SERVER_TRIGGER_LOOPING_CAPTURE)
      → CameraServer::TriggerLoopingCapture(req, reply)
        @ services/server/src/camera_server.cpp:264
        → ReadString(req, &sz)  [cameraId]
        → CameraService::GetCameraDevice(cameraId)
        → DeserializeFrameConfig(*req)  [反序列化]
          @ services/server/src/camera_server.cpp:199
          → new FrameConfig(type)
          → ReadUint32(surfaceNum)
          → SurfaceImpl::GenericSurfaceByIpcIo()  [重建Surface]
          → ReadInt32(qfactor/fps/invertMode/crop/format)
          → fc->SetParameter()  [设置参数]
        → device_->TriggerLoopingCapture(*fc, &streamId)  [设备层]
          @ services/impl/src/camera_device.cpp:52
          → 根据 fcType 选择 Assistant:
             - FRAME_CONFIG_PREVIEW → PreviewAssistant
             - FRAME_CONFIG_RECORD → RecordAssistant
             - FRAME_CONFIG_CALLBACK → CallbackAssistant
          → 检查 assistant->state_  [状态机检查]
          → assistant->SetFrameConfig(fc, &streamId)  [配置助手]
            @ services/impl/src/camera_device.cpp (各Assistant实现)
            
            // PreviewAssistant::SetFrameConfig 流程:
            → fc.GetSurfaces()  [获取Surface]
            → StreamAttrInitialize()  [初始化流属性]
              @ services/impl/src/camera_device.cpp:295
              → memset_s()  [清零结构体]
              → fc.GetParameter(CAM_IMAGE_FORMAT, format)
              → surface->GetWidth()/GetHeight()
              → fc.GetParameter(CAM_FRAME_FPS, fps)
            → HalCameraStreamCreate(cameraId_, &stream, streamId)  [创建HAL流]
              @ HAL层
            → HalCameraStreamSetInfo()  [设置流信息]
          → assistant->Start(*streamId)  [启动捕获]
            @ services/impl/src/camera_device.cpp:70
            
            // PreviewAssistant::Start 流程:
            → pthread_create(&threadId, NULL, YuvCopyProcess, this)  [创建线程]
              @ services/impl/src/camera_device.cpp:577
              → 注: YuvCopyProcess 当前为空实现
            → HalCameraStreamOn(cameraId_, streamId)  [启动HAL流]
              @ HAL层
            → state_ = LOOP_LOOPING  [更新状态]
        → OnTriggerLoopingCaptureFinished(loopingCaptureStatus, streamId)  [回调]
          → SendRequest(sid, ON_TRIGGER_LOOPING_CAPTURE_FINISHED)
            → 客户端回调处理
  → frameConfigs_.emplace_back(&fc)  [缓存FrameConfig]
  → 返回 MEDIA_OK
```

---

## 4. 帧数据处理调用链

### 4.1 预览数据流 (HAL → Display)

```
[硬件中断 - 新帧到达]
  ↓
HAL层回调
  → HalCamera驱动层处理
    → 将YUV数据写入显示Buffer
    → DisplayLayer::Flip()  [刷新显示]
      @ 显示HAL层
```

**注意**: Preview模式数据不经过CameraDevice用户态处理，直接由HAL送入显示。

### 4.2 录像数据流 (HAL → Codec → Surface)

```
[硬件中断 - 新帧到达]
  ↓
HalCamera驱动层
  → 将YUV数据送入视频编码器 (硬件VENC)
    → 编码完成中断
      → CodecCallback::OnVencBufferAvailble()  [编码回调]
        @ services/impl/src/camera_device.cpp:339
        → 遍历 vencSurfaces_ 列表
        → surface->RequestBuffer()  [请求输出Buffer]
          @ Surface模块
        → CopyCodecOutput(buf, &size, outBuf)  [拷贝编码数据]
          @ services/impl/src/camera_device.cpp:277
          → for 循环遍历 CodecBuffer
            → memcpy_s(dstBuf, *size, src, packSize)  [安全拷贝]
              @ services/impl/src/camera_device.cpp:285
        → surfaceBuf->SetInt32(KEY_IS_SYNC_FRAME, ...)  [设置元数据]
        → surfaceBuf->SetInt64(KEY_TIME_US, timestamp)
        → SurfaceSetSize(surfaceBuf, surface, size)  [设置数据大小]
          @ services/impl/src/camera_device.cpp:319
          → surfaceBuf->SetSize()
          → surface->FlushBuffer(surfaceBuf)  [提交Buffer]
        → CodecQueueOutput()  [归还编码Buffer]
```

### 4.3 回调数据流 (HAL → CallbackAssistant → Surface)

```
CallbackAssistant::Start()
  @ services/impl/src/camera_device.cpp:739
  → pthread_create(&threadId, NULL, StreamCopyProcess, this)  [创建拷贝线程]
    ↓
    StreamCopyProcess(void *arg)
      @ services/impl/src/camera_device.cpp:753
      → 循环 while (assistant->state_ == LOOP_LOOPING):
        → assistant->capSurface_->RequestBuffer()  [请求Buffer]
        → HalCameraDequeueBuf(assistant->cameraId_, assistant->streamId_, &streamBuffer)
          @ HAL层
          → 等待HAL层数据
          → 返回填充好的YUV数据指针
        → streamBuffer.virAddr = surfaceBuf->GetVirAddr()  [关联地址]
        → assistant->capSurface_->FlushBuffer(surfaceBuf)  [提交到Surface]
        → usleep(DELAY_TIME_ONE_FRAME)  [30ms延时，控制帧率]
      → 循环结束 (state_ 改变)
      → HalCameraQueueBuf()  [归还最后一个Buffer]
```

---

## 5. 拍照调用链

```
应用代码: camera->TriggerSingleCapture(fc)
  ↓
CameraImpl::TriggerSingleCapture(fc)
  @ frameworks/camera_impl.cpp:156
  → 检查 config_ != nullptr
  → 检查 fc.GetFrameConfigType() == FRAME_CONFIG_CAPTURE
  → deviceClient_->TriggerSingleCapture(fc)
    → IPC调用 → CameraServer::TriggerSingleCapture()
      @ services/server/src/camera_server.cpp:280
      → CameraService::GetCameraDevice(cameraId)
      → DeserializeFrameConfig()
      → device_->TriggerSingleCapture(*fc, &streamId)
        → device_->TriggerLoopingCapture(fc, streamId)  [复用循环捕获逻辑]
          @ services/impl/src/camera_device.cpp:33
          → 选择 CaptureAssistant
          → CaptureAssistant::SetFrameConfig()  [配置编码器]
            @ services/impl/src/camera_device.cpp:603
            → CameraCreateJpegEnc()  [创建JPEG编码器]
              @ services/impl/src/camera_device.cpp:235
              → CodecCreateByType(VIDEO_ENCODER, MEDIA_MIMETYPE_IMAGE_JPEG)
              → CodecSetParameter()  [设置编码参数]
              → SetVencSource()  [设置视频源]
          → CaptureAssistant::Start(streamId)  [启动拍照]
            @ services/impl/src/camera_device.cpp:642
            → HalCameraStreamOn()  [启动流]
            → CodecStart(vencHdl_)  [启动编码器]
            → new CodecBuffer[...]  [分配输出Buffer]
            → DO-WHILE 循环读取编码结果:
              → memset_s(outInfo)  [清零Buffer信息]
              → CodecDequeueOutput()  [获取编码数据]
                → 阻塞等待编码完成
              → capSurface_->RequestBuffer()  [请求输出Surface]
              → CopyCodecOutput()  [拷贝JPEG数据到Surface]
              → capSurface_->FlushBuffer()  [提交完成]
            → CodecStop()  [停止编码]
            → CodecDestroy()  [销毁编码器]
            → HalCameraStreamOff()  [停止流]
            → HalCameraStreamDestroy()  [销毁流]
            → delete outInfo  [释放资源]
      → OnTriggerSingleCaptureFinished()  [发送完成回调]
        → SendRequest(sid, ON_TRIGGER_SINGLE_CAPTURE_FINISHED)
          → 客户端: cameraImpl_->OnFrameFinished()
            @ frameworks/camera_impl.cpp:202
            → handler_->Post(lambda)
              → fsc->OnFrameFinished(*this, fc, frameResult)  [应用回调]
```

---

## 6. 相机关闭调用链

```
应用代码: camera->Release()
  ↓
CameraImpl::Release()
  @ frameworks/camera_impl.cpp:80
  → delete config_  [删除配置]
  → deviceClient_->Release()  [IPC调用]
    → CameraServer::CloseCamera()
      → CameraService::CloseCamera(cameraId)
        @ services/impl/src/camera_service.cpp:179
        → CameraService::GetCameraDevice(cameraId)
        → device->StopLoopingCapture(-1)  [停止所有流]
          @ services/impl/src/camera_device.cpp:07
          → 调用各Assistant的Stop():
            → PreviewAssistant::Stop()
              → pthread_join(threadId)  [等待线程结束]
              → HalCameraStreamOff()  [停止流]
              → HalCameraStreamDestroy()  [销毁流]
            → RecordAssistant::Stop()
              → ClearFrameConfig()  [清理编码器]
              → HalCameraStreamOff/Destroy()
            → CallbackAssistant::Stop()
              → pthread_join(threadId)
              → HalCameraStreamOff/Destroy()
        → deviceMap_.erase(cameraId)  [从缓存移除]
        → HalCameraDeviceClose(cameraId)  [关闭硬件]
      → OnCameraStatusChange(CAMERA_STATUS_CLOSE)  [发送关闭回调]
  → handler_->Post(lambda)
    → stateCb_->OnReleased(*this)  [应用回调]
```

---

## 7. IPC通信调用链

### 7.1 客户端发送请求

```
CameraServiceClient::GetCameraAbility(cameraId)
  @ frameworks/binder/src/camera_service_client.cpp:199
  → 检查 deviceAbilityMap_ 缓存
  → cameraIdForAbility = cameraId  [保存用于回调匹配]
  → IpcIoInit(&io, tmpData, DEFAULT_IPC_SIZE, 0)  [初始化IPC]
  → WriteString(&io, cameraId.c_str())  [序列化参数]
  → 准备 CallBackPara
    → para.funcId = CAMERA_SERVER_GET_CAMERA_ABILITY
    → para.data = this
  → proxy_->Invoke(proxy_, CAMERA_SERVER_GET_CAMERA_ABILITY, &io, &para, Callback)
    @ IPC框架
    → 发送IPC请求到服务端
    → 阻塞等待回调
  ← Callback() 被调用  [服务端返回]
    @ frameworks/binder/src/camera_service_client.cpp:65
    → 解析 reply
    → ReadUint32(reply, &supportProperties)
    → ReadUint32(reply, &listSize)
    → FOR 循环读取 CameraPicSize 列表
      → ReadRawData(reply, sizeof(CameraPicSize))
    → new CameraAbility  [创建能力对象]
    → ability->SetParameterRange(...)  [填充能力数据]
    → deviceAbilityMap_.insert(...)  [缓存结果]
  → 再次查找 deviceAbilityMap_
  → 返回 CameraAbility*
```

### 7.2 服务端处理请求

```
[IPC请求到达]
  ↓
CameraServer::CameraServerRequestHandle(funcId, origin, req, reply)
  @ services/server/src/camera_server.cpp:38
  → SWITCH funcId:
    CASE CAMERA_SERVER_GET_CAMERA_ABILITY:
      → CameraServer::GetCameraAbility(req, reply)
        → ReadString(req, &sz)  [cameraId]
        → CameraService::GetCameraAbility(cameraId)
          → 检查 deviceAbilityMap_ 缓存
          → HalCameraGetStreamCapNum()  [获取能力数量]
          → new StreamCap[num]  [分配数组]
          → HalCameraGetStreamCap()  [获取能力列表]
          → FOR 循环处理能力:
            → ability->SetParameterRange()  [设置能力范围]
          → delete[] streamCap  [释放数组]
          → deviceAbilityMap_.insert()  [缓存]
        → WriteUint32(reply, supportProperties)  [序列化响应]
        → WriteUint32(reply, listSize)
        → FOR 循环写入 CameraPicSize:
          → WriteRawData(reply, &supportSizeItem, sizeof(CameraPicSize))
        → WriteUint32(reply, afListSize)
        → FOR 循环写入 AF modes
        → WriteUint32(reply, aeListSize)
        → FOR 循环写入 AE modes
      → [IPC框架自动发送reply]
```

---

## 调用链速查表

| 功能 | 入口函数 | 关键中间函数 | 结束点 |
|------|----------|--------------|--------|
| 初始化 | CameraKit::GetInstance | CameraManagerImpl() → InitCameraServiceClient | CameraService::Initialize |
| 创建相机 | CameraKit::CreateCamera | CameraServiceClient::CreateCamera → CameraService::CreateCamera | CameraImpl::OnCreate |
| 配置相机 | Camera::Configure | CameraDeviceClient::SetCameraConfig | CameraImpl::OnConfigured |
| 启动预览 | Camera::TriggerLoopingCapture | PreviewAssistant::SetFrameConfig → Start | HalCameraStreamOn |
| 启动录像 | Camera::TriggerLoopingCapture | RecordAssistant::SetFrameConfig → Start | CodecStart |
| 拍照 | Camera::TriggerSingleCapture | CaptureAssistant::SetFrameConfig → Start | CodecDequeueOutput |
| 停止捕获 | Camera::StopLoopingCapture | Assistant::Stop | HalCameraStreamOff |
| 释放相机 | Camera::Release | CameraService::CloseCamera | CameraImpl::Release |
| 获取能力 | CameraKit::GetCameraAbility | CameraServiceClient::GetCameraAbility | CameraAbility |
| IPC请求 | XXXClient::XXX | proxy_->Invoke | XXXServer::XXX |

---

## 参考

- [架构说明](../01_Architecture.md) - 架构概述
- [内部接口](../03_Inner_API.md) - 接口定义
