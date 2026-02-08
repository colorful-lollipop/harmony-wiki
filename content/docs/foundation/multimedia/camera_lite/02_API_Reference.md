# 02 - API 参考 (C++)

> Camera_Lite 对外C++ API完整参考

---

## 概述

Camera_Lite 提供**纯C++ API**，没有JavaScript绑定层。应用开发者通过以下方式使用相机能力：

1. **CameraKit** - 单例入口类，获取相机列表和能力
2. **Camera** - 相机实例，执行预览/录像/拍照操作
3. **配置类** - CameraConfig, FrameConfig 用于参数设置
4. **回调类** - 异步事件通知机制

**所有API位置**: `interfaces/kits/*.h`

---

## CameraKit

### 类定义

```cpp
namespace OHOS {
namespace Media {
class CameraKit {
public:
    static CameraKit *GetInstance();
    ~CameraKit();
    
    uint8_t GetCameraModeNum();
    int32_t SetCameraMode(uint8_t modeIndex);
    std::list<std::string> GetCameraIds();
    const CameraAbility *GetCameraAbility(std::string cameraId);
    const CameraInfo *GetCameraInfo(std::string cameraId);
    void RegisterCameraDeviceCallback(CameraDeviceCallback &callback, EventHandler &handler);
    void UnregisterCameraDeviceCallback(CameraDeviceCallback &callback);
    void CreateCamera(const std::string &cameraId, CameraStateCallback &callback, EventHandler &handler);
    
private:
    CameraKit();
};
} // namespace Media
} // namespace OHOS
```

### 方法详解

#### GetInstance

```cpp
static CameraKit *GetInstance();
```

**功能**: 获取CameraKit单例实例

**权限要求**: `ohos.permission.CAMERA`

**返回值**:
| 值 | 说明 |
|----|------|
| CameraKit* | 成功，返回实例指针 |
| nullptr | 失败，可能原因：权限不足 |

**证据**: `frameworks/camera_kit.cpp:30-38`

```cpp
CameraKit *CameraKit::GetInstance()
{
    if (CheckSelfPermission("ohos.permission.CAMERA") != GRANTED) {
        MEDIA_WARNING_LOG("Process can not access camera.");
        return nullptr;
    }
    static CameraKit kit;
    return &kit;
}
```

---

#### GetCameraIds

```cpp
std::list<std::string> GetCameraIds();
```

**功能**: 获取当前可用相机ID列表

**返回值**: 相机ID字符串列表，如 `["0", "1"]`

**示例**:
```cpp
CameraKit *kit = CameraKit::GetInstance();
std::list<std::string> ids = kit->GetCameraIds();
for (const std::string &id : ids) {
    std::cout << "Camera ID: " << id << std::endl;
}
```

---

#### GetCameraAbility

```cpp
const CameraAbility *GetCameraAbility(std::string cameraId);
```

**功能**: 获取指定相机的能力对象

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| cameraId | std::string | 相机ID |

**返回值**: CameraAbility指针，包含分辨率、AE/AF模式等能力

**示例**:
```cpp
const CameraAbility *ability = kit->GetCameraAbility("0");
std::list<CameraPicSize> sizes = ability->GetSupportedSizes(CAM_FORMAT_JPEG);
```

---

#### CreateCamera

```cpp
void CreateCamera(const std::string &cameraId, 
                  CameraStateCallback &callback, 
                  EventHandler &handler);
```

**功能**: 创建相机实例

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| cameraId | std::string | 要创建的相机ID |
| callback | CameraStateCallback | 状态回调接口 |
| handler | EventHandler | 事件处理器，用于回调线程调度 |

**回调时机**:
- 成功: `callback.OnCreated(Camera &c)`
- 失败: `callback.OnCreateFailed(cameraId, errorCode)`

**注意**: 此方法是**异步**的，创建结果通过回调通知

---

## Camera

### 类定义

```cpp
namespace OHOS {
namespace Media {
class Camera {
public:
    virtual ~Camera() = default;
    virtual std::string GetCameraId();
    virtual const CameraConfig *GetCameraConfig() const;
    virtual FrameConfig *GetFrameConfig(int32_t type);
    virtual void Configure(CameraConfig &config);
    virtual int32_t TriggerLoopingCapture(FrameConfig &frameConfig);
    virtual void StopLoopingCapture(int32_t type);
    virtual int32_t TriggerSingleCapture(FrameConfig &frameConfig);
    virtual void Release();

protected:
    Camera() = default;
};
} // namespace Media
} // namespace OHOS
```

### 方法详解

#### GetCameraId

```cpp
virtual std::string GetCameraId();
```

**功能**: 获取相机ID

**返回值**: 相机ID字符串

---

#### Configure

```cpp
virtual void Configure(CameraConfig &config);
```

**功能**: 配置相机参数

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| config | CameraConfig | 配置对象，包含回调和处理器 |

**注意**: 必须在启动捕获前调用

**示例**:
```cpp
CameraConfig *config = CameraConfig::CreateCameraConfig();
config->SetFrameStateCallback(&frameCallback, &handler);
camera->Configure(*config);
```

---

#### TriggerLoopingCapture

```cpp
virtual int32_t TriggerLoopingCapture(FrameConfig &frameConfig);
```

**功能**: 启动循环帧捕获 (预览/录像/回调)

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| frameConfig | FrameConfig | 帧配置，包含类型和Surface |

**返回值**:
| 值 | 说明 |
|----|------|
| MEDIA_OK (0) | 成功 |
| MEDIA_ERR (-1) | 失败 |
| MEDIA_INVALID_PARAM | 无效参数 |

**支持的FrameConfig类型**:
- `FRAME_CONFIG_PREVIEW` - 预览
- `FRAME_CONFIG_RECORD` - 录像
- `FRAME_CONFIG_CALLBACK` - 回调

**注意**: `FRAME_CONFIG_CAPTURE` 不支持循环捕获

---

#### StopLoopingCapture

```cpp
virtual void StopLoopingCapture(int32_t type);
```

**功能**: 停止循环帧捕获

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| type | int32_t | 要停止的类型，-1表示停止所有 |

**示例**:
```cpp
camera->StopLoopingCapture(FRAME_CONFIG_PREVIEW);  // 停止预览
camera->StopLoopingCapture(-1);                      // 停止所有
```

---

#### TriggerSingleCapture

```cpp
virtual int32_t TriggerSingleCapture(FrameConfig &frameConfig);
```

**功能**: 触发单帧捕获 (拍照)

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| frameConfig | FrameConfig | 帧配置，类型必须为 FRAME_CONFIG_CAPTURE |

**返回值**:
| 值 | 说明 |
|----|------|
| MEDIA_OK (0) | 成功 |
| MEDIA_ERR (-1) | 失败 |

---

#### Release

```cpp
virtual void Release();
```

**功能**: 释放相机资源

**注意**: 释放后会触发 `OnReleased` 回调

---

## CameraConfig

### 类定义

```cpp
namespace OHOS {
namespace Media {
class CameraConfig {
public:
    virtual ~CameraConfig() {}
    static CameraConfig *CreateCameraConfig();
    virtual void SetFrameStateCallback(FrameStateCallback *callback, EventHandler *handler);
    virtual EventHandler *GetEventHandler() const;
    virtual FrameStateCallback *GetFrameStateCb() const;

protected:
    CameraConfig() {}
};
} // namespace Media
} // namespace OHOS
```

### 方法详解

#### CreateCameraConfig

```cpp
static CameraConfig *CreateCameraConfig();
```

**功能**: 创建配置对象

**返回值**: CameraConfig指针

---

#### SetFrameStateCallback

```cpp
virtual void SetFrameStateCallback(FrameStateCallback *callback, EventHandler *handler);
```

**功能**: 设置帧状态回调

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| callback | FrameStateCallback* | 帧状态回调接口 |
| handler | EventHandler* | 事件处理器 |

---

## FrameConfig

### 类定义

```cpp
namespace OHOS {
namespace Media {
constexpr int32_t FRAME_CONFIG_PREVIEW = 0;
constexpr int32_t FRAME_CONFIG_RECORD = 1;
constexpr int32_t FRAME_CONFIG_CAPTURE = 2;
constexpr int32_t FRAME_CONFIG_CALLBACK = 3;

class FrameConfig {
public:
    FrameConfig() = delete;
    explicit FrameConfig(int32_t type);
    ~FrameConfig() {}
    
    int32_t GetFrameConfigType();
    std::list<Surface *> GetSurfaces();
    void AddSurface(Surface &surface);
    void RemoveSurface(Surface &surface);
    
    template<typename T> void SetParameter(uint32_t key, const T value);
    template<typename T> void GetParameter(uint32_t key, T &value);
    
    void SetVendorParameter(uint8_t *value, uint32_t len);
    void GetVendorParameter(uint8_t *value, uint32_t len);
};
} // namespace Media
} // namespace OHOS
```

### 方法详解

#### 构造函数

```cpp
explicit FrameConfig(int32_t type);
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| type | int32_t | 配置类型 |

**类型值**:
| 常量 | 值 | 用途 |
|------|----|------|
| FRAME_CONFIG_PREVIEW | 0 | 预览 |
| FRAME_CONFIG_RECORD | 1 | 录像 |
| FRAME_CONFIG_CAPTURE | 2 | 拍照 |
| FRAME_CONFIG_CALLBACK | 3 | 回调 |

---

#### AddSurface / RemoveSurface

```cpp
void AddSurface(Surface &surface);
void RemoveSurface(Surface &surface);
```

**功能**: 添加/移除输出Surface

**注意**: Record模式最多支持2个Surface

**证据**: `frameworks/binder/src/camera_device_client.cpp:158`
```cpp
constexpr uint32_t maxSurfaceNum = 2; // 2 surfaces at most
```

---

#### SetParameter / GetParameter

```cpp
template<typename T> void SetParameter(uint32_t key, const T value);
template<typename T> void GetParameter(uint32_t key, T &value);
```

**功能**: 设置/获取参数

**支持的参数键** (定义于 `meta_data.h`):

| 键名 | 说明 | 类型 |
|------|------|------|
| CAM_FRAME_FPS | 帧率 | int32_t |
| CAM_AE_MODE | 自动曝光模式 | int32_t |
| CAM_AE_EXPO_TIME | 曝光时间 | int32_t |
| CAM_AE_COMPENSATION | 曝光补偿 | int32_t |
| CAM_AWB_MODE | 白平衡模式 | int32_t |
| CAM_IMAGE_CROP_RECT | 裁剪区域 | CameraRect |
| CAM_IMAGE_INVERT_MODE | 镜像/旋转模式 | int32_t |
| CAM_IMAGE_FORMAT | 图像格式 | int32_t |
| PARAM_KEY_IMAGE_ENCODE_QFACTOR | JPEG质量 | int32_t |

**示例**:
```cpp
FrameConfig fc(FRAME_CONFIG_PREVIEW);
fc.SetParameter(CAM_FRAME_FPS, 30);

CameraRect crop = {0, 0, 1920, 1080};
fc.SetParameter(CAM_IMAGE_CROP_RECT, crop);
```

---

## CameraAbility

### 类定义

```cpp
namespace OHOS {
namespace Media {
class CameraAbility {
public:
    CameraAbility();
    virtual ~CameraAbility();
    
    std::list<CameraPicSize> GetSupportedSizes(int format) const;
    std::list<int32_t> GetSupportedAfModes() const;
    std::list<int32_t> GetSupportedAeModes() const;
    
    template<typename T> int32_t SetParameterRange(uint32_t key, std::list<T> rangeList);
    template<typename T> std::list<T> GetParameterRange(uint32_t key) const;
};
} // namespace Media
} // namespace OHOS
```

### 方法详解

#### GetSupportedSizes

```cpp
std::list<CameraPicSize> GetSupportedSizes(int format) const;
```

**功能**: 获取支持的图像尺寸列表

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| format | int | 图像格式，如 CAM_FORMAT_JPEG |

**返回值**: CameraPicSize列表

---

#### GetSupportedAfModes

```cpp
std::list<int32_t> GetSupportedAfModes() const;
```

**功能**: 获取支持的自动对焦模式

**返回值**: AF模式列表

**AF模式值** (`meta_data.h`):
| 常量 | 值 |
|------|-----|
| AF_MODE_CONTINUOUS | 0 |
| AF_MODE_CLOSE | 1 |

---

## 回调接口

### CameraStateCallback

```cpp
class CameraStateCallback {
public:
    CameraStateCallback() = default;
    virtual ~CameraStateCallback() {}
    
    virtual void OnCreated(Camera &c) {}
    virtual void OnCreateFailed(const std::string cameraId, int32_t errorCode) {}
    virtual void OnReleased(Camera &c) {}
    virtual void OnConfigured(Camera &c) {}
    virtual void OnConfigureFailed(const std::string cameraId, int32_t errorCode) {}
};
```

**回调时机**:

| 回调 | 触发条件 |
|------|----------|
| OnCreated | 相机创建成功 |
| OnCreateFailed | 相机创建失败 |
| OnReleased | 相机关闭/释放 |
| OnConfigured | 相机配置成功 |
| OnConfigureFailed | 相机配置失败 |

---

### FrameStateCallback

```cpp
class FrameStateCallback {
public:
    FrameStateCallback() = default;
    ~FrameStateCallback() = default;
    
    virtual void OnFrameFinished(Camera &camera, 
                                  FrameConfig &frameConfig, 
                                  FrameResult &frameResult) {}
    virtual void OnFrameError(Camera &camera, 
                              FrameConfig &frameConfig, 
                              int32_t errorCode, 
                              FrameResult &frameResult) {}
};
```

**回调时机**:

| 回调 | 触发条件 |
|------|----------|
| OnFrameFinished | 帧捕获完成 |
| OnFrameError | 帧捕获出错 |

---

### CameraDeviceCallback

```cpp
class CameraDeviceCallback {
public:
    CameraDeviceCallback() = default;
    virtual ~CameraDeviceCallback() {}
    
    enum CameraDeviceState { CAMERA_DEVICE_STATE_AVAILABLE, CAMERA_DEVICE_STATE_UNAVAILABLE };
    virtual void OnCameraStatus(std::string &cameraId, CameraDeviceState cameraStatus) {}
};
```

**用途**: 监听相机设备热插拔状态

---

## 数据结构

### CameraPicSize

```cpp
typedef struct {
    uint32_t width;
    uint32_t height;
} CameraPicSize;
```

### CameraRect

```cpp
typedef struct {
    int32_t x;
    int32_t y;
    int32_t w;
    int32_t h;
} CameraRect;
```

### 枚举类型

**CameraType**:
```cpp
typedef enum {
    WIDE_ANGLE,     // 广角相机
    FISH_EYE,       // 鱼眼相机
    TRUE_DEPTH,     // 深度相机
    OTHER_TYPE      // 其他
} CameraType;
```

**FacingType**:
```cpp
typedef enum {
    CAMERA_FACING_FRONT,    // 前置
    CAMERA_FACING_BACK,     // 后置
    CAMERA_FACING_OTHERS    // 其他
} FacingType;
```

---

## 完整示例

### 预览示例

```cpp
#include "camera_kit.h"
#include "camera.h"
#include "camera_config.h"
#include "frame_config.h"
#include "camera_state_callback.h"
#include "frame_state_callback.h"

using namespace OHOS::Media;

class MyCameraCallback : public CameraStateCallback {
public:
    void OnCreated(Camera &c) override {
        // 创建CameraConfig
        CameraConfig *config = CameraConfig::CreateCameraConfig();
        config->SetFrameStateCallback(&frameCb_, &handler_);
        c.Configure(*config);
        
        // 创建FrameConfig
        FrameConfig fc(FRAME_CONFIG_PREVIEW);
        fc.AddSurface(*surface_);  // surface需要预先创建
        c.TriggerLoopingCapture(fc);
    }
    
    void OnCreateFailed(const std::string cameraId, int32_t errorCode) override {
        printf("Create camera failed: %d\n", errorCode);
    }
    
private:
    MyFrameCallback frameCb_;
    EventHandler handler_;
    Surface *surface_;
};

class MyFrameCallback : public FrameStateCallback {
public:
    void OnFrameFinished(Camera &camera, 
                        FrameConfig &frameConfig, 
                        FrameResult &frameResult) override {
        printf("Frame captured\n");
    }
};

int main() {
    // 1. 获取CameraKit
    CameraKit *kit = CameraKit::GetInstance();
    if (kit == nullptr) {
        printf("No camera permission\n");
        return -1;
    }
    
    // 2. 获取相机列表
    std::list<std::string> ids = kit->GetCameraIds();
    if (ids.empty()) {
        printf("No camera available\n");
        return -1;
    }
    
    // 3. 创建相机
    MyCameraCallback callback;
    EventHandler handler;
    kit->CreateCamera(ids.front(), callback, handler);
    
    // 等待回调...
    
    return 0;
}
```

---

## 错误码

| 常量 | 值 | 说明 |
|------|----|------|
| MEDIA_OK | 0 | 成功 |
| MEDIA_ERR | -1 | 通用错误 |
| MEDIA_INVALID_PARAM | -2 | 无效参数 |

---

## 下一步

- [内部接口](03_Inner_API.md) - 了解内部实现
- [构建系统](04_Build_System.md) - 如何编译
- [安全风险](05_Security.md) - 安全注意事项
