# 内部实现细节 (Internals)

> OpenHarmony Camera Framework - 核心类设计与实现原理

---

## 核心类职责

### 1. CameraManager (单例)

**职责**: 相机功能的统一入口，负责设备枚举、会话/输出创建

**代码位置**: `interfaces/inner_api/native/camera/include/input/camera_manager.h:160`

**关键属性**:
```cpp
class CameraManager : public RefBase {
    static sptr<CameraManager> g_cameraManager;  // 单例实例
    sptr<ICameraService> serviceProxyPrivate_;    // IPC 代理
    std::vector<sptr<CameraDevice>> cameraDeviceList_;  // 设备缓存
    
    // 回调管理
    std::shared_ptr<CameraStatusListenerManager> cameraStatusListenerManager_;
    std::shared_ptr<TorchServiceListenerManager> torchServiceListenerManager_;
};
```

**线程安全**: 单例使用 `g_instanceMutex` 保护，实例创建后内部状态管理使用各自锁。

---

### 2. CaptureSession (会话管理)

**职责**: 管理相机会话的生命周期和参数配置

**代码位置**: `interfaces/inner_api/native/camera/include/session/capture_session.h:650`

**状态机**:
```
SESSION_INIT 
    ↓ beginConfig()
SESSION_CONFIG_INPROGRESS
    ↓ addInput/addOutput
    ↓ commitConfig()
SESSION_CONFIG_COMMITTED
    ↓ start()
SESSION_STARTED
    ↓ stop()
SESSION_CONFIG_COMMITTED
    ↓ release()
SESSION_RELEASED
```

**关键方法**:
```cpp
class CaptureSession : public RefBase {
    int32_t BeginConfig();
    int32_t AddInput(sptr<CaptureInput>& input);
    int32_t AddOutput(sptr<CaptureOutput>& output);
    int32_t CommitConfig();
    int32_t Start();
    int32_t Stop();
    int32_t Release();
    
    // 参数控制
    int32_t SetZoomRatio(float ratio);
    int32_t SetFocusMode(FocusMode mode);
    int32_t SetFlashMode(FlashMode mode);
};
```

**继承体系**:
```
CaptureSession
├── PhotoSession
├── VideoSession
├── ScanSession
├── SecureCameraSession
└── CaptureSessionForSys (系统扩展)
    ├── PhotoSessionForSys
    ├── VideoSessionForSys
    ├── NightSession
    ├── PortraitSession
    └── ... (20+ 模式)
```

---

### 3. HCameraService (系统服务)

**职责**: 系统级相机服务，处理 IPC 请求，管理设备生命周期

**代码位置**: `services/camera_service/include/hcamera_service.h:116`

**关键属性**:
```cpp
class HCameraService : public CameraServiceStub {
    // 会话管理
    sptr<HCameraSessionManager> sessionManager_;
    
    // 设备管理
    sptr<HCameraDeviceManager> deviceManager_;
    sptr<HCameraHostManager> hostManager_;
    
    // 状态
    std::atomic<CameraServiceStatus> serviceStatus_;
};
```

**SA 注册**:
```cpp
// services/camera_service/src/hcamera_service.cpp
REGISTER_SYSTEM_ABILITY_BY_ID(HCameraService, CAMERA_SERVICE_ID, true)
```

---

### 4. HStreamOperator (流操作器)

**职责**: 协调多个流的创建、配置和生命周期管理

**代码位置**: `services/camera_service/include/hstream_operator.h`

**关键属性**:
```cpp
class HStreamOperator : public RefBase {
    sptr<HCameraDevice> cameraDevice_;  // 关联设备
    StreamContainer streamContainer_;    // 流容器
    sptr<OHOS::HDI::Camera::V1_0::IStreamOperator> hdiStreamOperator_;  // HAL 接口
};
```

**流容器**:
```cpp
class StreamContainer {
    std::map<const StreamType, std::list<sptr<HStreamCommon>>> streams_;
    // StreamType: CAPTURE(1), REPEAT(2), METADATA(3), DEPTH(4)
};
```

---

### 5. HStreamCommon (流基类)

**职责**: 定义流的通用接口

**继承体系**:
```
HStreamCommon (abstract)
├── HStreamRepeat      - 连续流 (预览、视频)
├── HStreamCapture     - 拍照流
├── HStreamMetadata    - 元数据流
└── HStreamDepthData   - 深度数据流
```

**关键方法**:
```cpp
class HStreamCommon : public RefBase {
    virtual int32_t LinkInput(wptr<HStreamOperator> operator) = 0;
    virtual int32_t Start() = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t ReleaseStream() = 0;
};
```

---

## 资源生命周期

### 对象所有权模型

```
App (JS/Native)
    │ owns
    ▼
CaptureSession (Framework)
    │ owns
    ├── CameraInput ──► HCameraDevice (Service)
    ├── PhotoOutput ──► HStreamCapture (Service)
    ├── PreviewOutput ─► HStreamRepeat (Service)
    └── VideoOutput ──► HStreamRepeat (Service)
```

### 生命周期时序

#### 1. CameraManager 生命周期

```
进程启动
  ↓
CameraManager::GetInstance() (懒加载)
  ↓
创建单例实例
  ↓
获取 ICameraService Proxy
  ↓
... 使用 ...
  ↓
进程退出
  ↓
自动释放
```

#### 2. CaptureSession 生命周期

```
创建: CameraManager::CreateCaptureSession()
  ↓
BeginConfig() - 开始配置
  ↓
AddInput() / AddOutput() - 添加输入输出
  ↓
CommitConfig() - 提交到 HAL
  ↓
Start() / Stop() - 运行控制
  ↓
Release() - 释放资源
```

**资源释放顺序**:
```cpp
// 正确的释放顺序 (HCaptureSession::Release())
1. Stop all streams
2. Release streams (HStreamRepeat/HStreamCapture)
3. Close camera device (HCameraDevice::Close())
4. Clear session state
5. Notify client
```

#### 3. Buffer 生命周期

```
HAL 层分配 Buffer
  ↓
通过 Surface BufferQueue 传递
  ↓
App 消费 Buffer (预览显示/照片保存)
  ↓
释放 Buffer 回到 BufferQueue
  ↓
HAL 层重用 Buffer
```

---

## 内部 API 契约

### 稳定接口 (Stable)

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `CameraManager::GetInstance()` | 稳定 | 单例获取 |
| `CameraManager::GetSupportedCameras()` | 稳定 | 设备枚举 |
| `CaptureSession` 基础方法 | 稳定 | 生命周期管理 |
| `ICameraService` IDL | 稳定 | IPC 接口 |

### 内部接口 (Internal)

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `HCameraService` 实现细节 | 可能变更 | 服务内部实现 |
| `HStreamOperator` | 可能变更 | 流管理内部 |
| `StreamContainer` | 可能变更 | 容器实现 |

---

## 设计模式

### 1. 单例模式 (Singleton)

**应用**: `CameraManager`, `HCameraSessionManager`

```cpp
class CameraManager : public RefBase {
public:
    static sptr<CameraManager>& GetInstance() {
        std::lock_guard<std::mutex> lock(g_instanceMutex);
        if (g_cameraManager == nullptr) {
            g_cameraManager = new CameraManager();
        }
        return g_cameraManager;
    }
};
```

### 2. 工厂模式 (Factory)

**应用**: `CameraManager` 创建各类输出

```cpp
sptr<CaptureOutput> CameraManager::CreatePhotoOutput(Profile profile, sptr<Surface> surface) {
    // 创建 PhotoOutput 实例
    return new PhotoOutput(profile, surface);
}
```

### 3. 代理模式 (Proxy)

**应用**: IPC 通信

```cpp
// Client 端使用 Proxy
sptr<ICameraService> proxy = CameraManager::GetServiceProxy();
proxy->CreateCameraDevice(cameraId);  // 实际是 IPC 调用

// Server 端使用 Stub
class HCameraService : public CameraServiceStub {
    int32_t OnRemoteRequest(...) override;
};
```

### 4. 观察者模式 (Observer)

**应用**: 状态回调

```cpp
class CameraManagerCallback {
    virtual void OnCameraStatusChanged(const CameraStatusInfo& info) = 0;
};

class CameraManager {
    void RegisterCameraStatusCallback(shared_ptr<CameraManagerCallback> callback);
};
```

### 5. RAII (资源管理)

**应用**: Buffer、锁的自动释放

```cpp
// 智能指针自动管理生命周期
sptr<CameraInput> input = cameraManager->CreateCameraInput(device);
// 超出作用域自动 Release()
```

---

## 线程安全

### 锁策略

| 类 | 锁类型 | 保护范围 |
|----|--------|----------|
| `CameraManager` | `g_instanceMutex` | 单例创建 |
| `CaptureSession` | `mutex_` | 状态变更 |
| `HCaptureSession` | `mutex_` | 会话状态 |
| `HCameraService` | `mutex_` | 会话列表 |
| `StreamContainer` | 无 (外部保证) | 流列表访问 |

### 线程模型

```
Main Thread (IPC Handler)
    │
    ├── 处理 IPC 调用
    ├── 状态机转换
    └── 调用 Service 方法
    
Worker Thread (Task Manager)
    │
    ├── 耗时操作 (配置、启动)
    └── 异步回调
    
HAL Callback Thread
    │
    ├── Buffer 回调
    ├── 元数据回调
    └── 错误回调
```

---

## 性能优化点

### 1. 设备列表缓存

```cpp
// CameraManager 缓存设备列表，避免频繁 IPC
std::vector<sptr<CameraDevice>> cameraDeviceList_;
```

### 2. Buffer 池

```cpp
// HAL 层使用 BufferQueue 实现 Buffer 复用
// 减少内存分配和拷贝
```

### 3. 延迟处理 (Deferred Processing)

```cpp
// 照片后期处理在独立服务中进行
// 不影响相机主流程性能
```

---

## 调试技巧

### 日志标签

| 标签 | 说明 |
|------|------|
| `CAMERA_FWK` | 框架层日志 |
| `CAMERA_SVC` | 服务层日志 |
| `CAMERA_NAPI` | N-API 日志 |
| `CAMERA_HAL` | HAL 层日志 |

### 调试命令

```bash
# 查看相机服务状态
hidumper -s 3008

# 查看相机日志
hilog -T CAMERA_FWK -T CAMERA_SVC

# 查看 SA 状态
sa -l | grep 3008
```

---

## 下一步阅读

- [代码地图](./03_CodeMap.md) - 文件导航
- [构建与产物](./07_Build.md) - 编译系统
- [安全风险评估](./06_SecurityReview.md) - 安全分析
