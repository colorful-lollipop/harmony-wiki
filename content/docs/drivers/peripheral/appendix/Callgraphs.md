# 关键调用链

本文档说明各模块的关键调用链（入口 → 核心逻辑）。

## Audio 模块调用链

### 1.1 音频播放调用链

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: AudioService                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ dlopen()
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Service Layer: libhdi_audio.z.so                                         │
│  - audio_manager_service.c                                                   │
│  - audio_manager_driver.c                                                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ 函数调用
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation: audio_adapter.c                                          │
│  - InitAllPorts()                                                            │
│  - CreateRender() / DestroyRender()                                          │
│  - RenderFrame()                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ ioctl() / write()
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: ALSA Driver                                                    │
│  - snd_pcm_writei()                                                          │
│  - DMA 缓冲区映射                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 音频渲染关键调用

```
AudioService::StartRender()
        │
        ├─▶ GetInputInterface()     → 获取音频接口
        │
        ├─▶ LoadAdapter()           → 加载音频适配器
        │
        ├─▶ InitAllPorts()          → 初始化端口
        │
        ├─▶ CreateRender()          → 创建渲染器
        │       │
        │       └─▶ AudioAdapter::CreateRender()
        │               │
        │               ├─▶ OpenPcmDevice()    → 打开 PCM 设备
        │               ├─▶ SetParameters()     → 设置采样参数
        │               └─▶ AllocBuffer()       → 分配缓冲区
        │
        └─▶ RenderFrame()            → 渲染音频帧
                │
                └─▶ AudioRender::RenderFrame()
                        │
                        ├─▶ CheckPcmState()     → 检查 PCM 状态
                        ├─▶ CopyToBuffer()      → 复制音频数据
                        └─▶ WriteToDevice()     → 写入设备
```

---

## Input 模块调用链

### 2.1 输入设备打开流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: InputService                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ dlopen()
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Service Layer: libhdi_input.z.so                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Stub: input_interface_driver.cpp                                         │
│  - InputInterfaceDriverInit()                                                │
│  - OnRemoteRequest()                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ 函数调用
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation: input_manager.c                                          │
│  - OpenInputDevice()                                                         │
│  - CloseInputDevice()                                                        │
│  - GetInputDeviceList()                                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ open() / close() / ioctl()
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: Input Driver (/dev/input/eventX)                              │
│  - evdev_open()                                                              │
│  - evdev_event()                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 输入事件上报流程

```
                    ┌─────────────────────┐
                    │ 硬件产生中断         │
                    └──────────┬──────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: Input Driver                                                   │
│  - input_event()    ← 产生输入事件                                           │
│  - evdev_event()    ← 发送到用户空间                                         │
└─────────────────────────────────────────────────────────────────────────────┘
                               │
                               │ read() / poll()
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation: input_hub.c / input_reporter.c                          │
│  - ReadEventData()    ← 读取事件数据                                         │
│  - ParseEvent()       ← 解析事件                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                               │
                               │ 回调通知
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Stub: input_reporter_stub.cpp                                           │
│  - ReportEvent()      → IPC 调用上报                                         │
└─────────────────────────────────────────────────────────────────────────────┘
                               │
                               │ IPC (Binder)
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Proxy: input_reporter_proxy.cpp                                         │
│  - ReportEventCb()    → 回调函数                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                               │
                               │ 回调调用
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: InputService                                                 │
│  - OnInputEvent()     ← 处理输入事件                                         │
│        │                                                                      │
│        ├─▶ 分发到 WindowManager                                              │
│        ├─▶ 分发到 Application                                                │
│        └─▶ 分发到 Accessibility                                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 完整调用序列

```c
// InputService 代码示例（基于 input/README_zh.md）

// 1. 获取接口
int32_t ret = GetInputInterface(&g_inputInterface);

// 2. 打开设备
ret = g_inputInterface->iInputManager->OpenInputDevice(DEV_INDEX);

// 3. 注册回调
g_callback.ReportEventPkgCallback = ReportEventPkgCallback;
ret = g_inputInterface->iInputReporter->RegisterReportCallback(DEV_INDEX, &g_callback);

// 4. 等待事件（回调自动触发）
// ...

// 5. 注销回调
ret = g_inputInterface->iInputReporter->UnregisterReportCallback(DEV_INDEX);

// 6. 关闭设备
ret = g_inputInterface->iInputManager->CloseInputDevice(DEV_INDEX);
```

---

## Sensor 模块调用链

### 3.1 传感器数据订阅流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: SensorService                                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ dlopen()
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Service Layer: libhdi_sensor.z.so                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation: sensor_hdi.c                                             │
│  - SensorDataCallback()    ← 硬件数据回调                                   │
│  - Enable() / Disable()     → 使能/去使能传感器                             │
│  - SetBatch()               → 设置采样参数                                   │
│  - Register() / Unregister() → 注册/注销数据回调                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ read() / poll() / interrupt
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: Sensor Driver (/sys/class/sensor/)                           │
│  - sysfs 属性操作                                                           │
│  - 中断处理                                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 传感器操作序列

```c
// SensorService 代码示例（基于 sensor/README_zh.md）

// 1. 创建接口实例
const struct SensorInterface *sensorDev = NewSensorInterfaceInstance();

// 2. 注册数据回调
ret = sensorDev->Register(user, SensorDataCallback);

// 3. 获取传感器列表
ret = sensorDev->GetAllSensors(&sensorInfo, &count);

// 4. 设置采样率
ret = sensorDev->SetBatch(SENSOR_TYPE_ACCELEROMETER, sensorInterval, 0);

// 5. 使能传感器
ret = sensorDev->Enable(SENSOR_TYPE_ACCELEROMETER);

// 6. 等待数据（回调自动触发）
// ...

// 7. 去使能传感器
ret = sensorDev->Disable(SENSOR_TYPE_ACCELEROMETER);

// 8. 取消订阅
ret = sensorDev->Unregister(user);

// 9. 释放接口实例
ret = FreeSensorInterfaceInstance();
```

---

## Display 模块调用链

### 4.1 显示图层创建流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: DisplayManagerService                                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ dlopen()
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Service Layer: libhdi_display.z.so                                      │
│  - display_device_host_driver.cpp                                           │
│  - display_device_stub.cpp                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Stub: DisplayDeviceStub::OnRemoteRequest()                              │
│  - CMD_GET_LAYER_INFO    → 获取图层信息                                     │
│  - CMD_SET_LAYER_ZORDER  → 设置图层 Z 序                                    │
│  - CMD_FLUSH             → 刷新显示                                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation                                                          │
│  - GetLayerInfo()                                                           │
│  - SetLayerZorder()                                                         │
│  - Flush()                                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ ioctl() / DRM ioctl
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: DRM Driver                                                    │
│  - drm_mode_setplane()                                                      │
│  - drm_mode_page_flip()                                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Camera 模块调用链

### 5.1 摄像头预览流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: CameraService                                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ dlopen()
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Service Layer: libhdi_camera.z.so                                      │
│  - icamera_interface.cpp                                                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Stub: ICameraInterface::OnRemoteRequest()                              │
│  - CAMERA_CMD_OPEN         → 打开摄像头                                     │
│  - CAMERA_CMD_START_PREVIEW → 开始预览                                      │
│  - CAMERA_CMD_STOP_PREVIEW  → 停止预览                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation: camera_hal.cpp                                         │
│  - OpenCamera()                                                             │
│  - StartPreview()                                                           │
│  - StopPreview()                                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ V4L2 ioctl
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: V4L2 Driver (/dev/videoX)                                     │
│  - VIDIOC_QUERYCAP                                                          │
│  - VIDIOC_S_FMT                                                             │
│  - VIDIOC_STREAMON                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## USB 模块调用链

### 6.1 USB 设备读写流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: USBService                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ dlopen()
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Service Layer: libhdi_usb.z.so                                         │
│  - usb_ddk_*/                                                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation: usb_*.c                                                 │
│  - UsbInit()           → 初始化 USB 栈                                      │
│  - UsbOpenDevice()     → 打开设备                                          │
│  - UsbControlTransfer() → 控制传输                                         │
│  - UsbBulkTransfer()   → 批量传输                                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ libusb / ioctl
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: USB Core / USB Driver                                         │
│  - usb_submit_urb()                                                         │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 认证模块调用链

### 7.1 指纹认证流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ System Service: UserAuthService                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼ dlopen()
┌─────────────────────────────────────────────────────────────────────────────┐
│ HDI Service Layer: libhdi_fingerprint_auth.z.so                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ HAL Implementation: fingerprint_auth_*.cpp                                  │
│  - FingerprintAuthInit()    → 初始化认证                                     │
│  - Enroll()                 → 录入指纹                                      │
│  - Authenticate()           → 比对认证                                      │
│  - Delete()                 → 删除指纹                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ ioctl() / HID
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ Linux Kernel: Fingerprint Driver (/dev/uinput)                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 线程模型与异步调用

### 8.1 同步调用（等待结果）

```
调用者线程:
    │
    ├─▶ HDI 接口调用
    │
    ├─▶ IPC 发送请求 (阻塞)
    │
    ├─▶ 等待响应
    │
    └─▶ 返回结果
```

### 8.2 异步回调（事件驱动）

```
硬件线程:
    │
    ├─▶ 产生中断/事件
    │
    └─▶ 触发 HAL 回调
            │
            ├─▶ 放入事件队列
            │
            └─▶ 通知 IPC 线程
                    │
                    ├─▶ IPC 回调调用
                    │
                    └─▶ 系统服务回调执行
```

---

## 调用链入口汇总

| 模块 | 接口获取 | 设备打开 | 核心操作 |
|------|----------|----------|----------|
| Audio | `GetInputInterface()` | `LoadAdapter()` | `RenderFrame()` |
| Input | `GetInputInterface()` | `OpenInputDevice()` | `RegisterReportCallback()` |
| Sensor | `NewSensorInterfaceInstance()` | `Enable()` | `Register()` |
| Display | `GetDisplayInterface()` | `GetLayer()` | `Flush()` |
| Camera | `GetCameraInterface()` | `OpenCamera()` | `StartPreview()` |
| USB | `GetUsbInterface()` | `UsbOpenDevice()` | `UsbControlTransfer()` |
| Fingerprint | `GetFingerprintInterface()` | `AuthenticateInit()` | `Authenticate()` |
