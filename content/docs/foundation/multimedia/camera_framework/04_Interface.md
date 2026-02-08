# 对外接口文档 (Interface)

> OpenHarmony Camera Framework - API 接口说明

---

## 接口总览

Camera Framework 提供 **4 层 API**:

| 层级 | 技术 | 目标用户 | 文件位置 |
|------|------|----------|----------|
| **JavaScript API** | TypeScript/N-API | 应用开发者 | `@ohos.multimedia.camera` |
| **NDK API** | C/C++ | Native 应用 | `camera_manager.h` |
| **Inner Native API** | C++ | 系统开发者 | `interfaces/inner_api/` |
| **IPC API** | IDL/HDI | 服务间通信 | `services/*/idls/` |

---

## 1. JavaScript API (N-API)

### 模块导入

```typescript
import camera from '@ohos.multimedia.camera';
```

### CameraManager API

| 方法名 | 参数 | 返回值 | 说明 | 权限 |
|--------|------|--------|------|------|
| `getCameraManager` | `context: Context` | `CameraManager` | 获取管理器 | 无 |
| `getSupportedCameras` | - | `CameraDevice[]` | 获取相机列表 | CAMERA |
| `getSupportedSceneModes` | `device: CameraDevice` | `SceneMode[]` | 获取支持模式 | CAMERA |
| `createCameraInput` | `device: CameraDevice` | `CameraInput` | 创建输入 | CAMERA |
| `createCaptureSession` | - | `CaptureSession` | 创建会话 | CAMERA |
| `createSession` | `mode: SceneMode` | `CaptureSession` | 创建模式会话 | CAMERA |
| `createPreviewOutput` | `profile: Profile, surfaceId: string` | `PreviewOutput` | 创建预览 | CAMERA |
| `createPhotoOutput` | `profile: Profile, surfaceId: string` | `PhotoOutput` | 创建拍照 | CAMERA |
| `createVideoOutput` | `profile: VideoProfile, surfaceId: string` | `VideoOutput` | 创建录像 | CAMERA+MIC |
| `createMetadataOutput` | `profile: MetadataProfile` | `MetadataOutput` | 创建元数据 | CAMERA |
| `isCameraMuted` | - | `boolean` | 是否静音 | 无 |
| `muteCamera` | `mute: boolean` | `void` | 静音/取消 | CAMERA |
| `isTorchSupported` | - | `boolean` | 是否支持闪光灯 | 无 |
| `setTorchMode` | `mode: TorchMode` | `void` | 设置闪光灯 | CAMERA |

### CaptureSession API

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `beginConfig` | - | 开始配置 |
| `addInput` | `input: CameraInput` | 添加输入 |
| `addOutput` | `output: CaptureOutput` | 添加输出 |
| `removeInput` | `input: CameraInput` | 移除输入 |
| `removeOutput` | `output: CaptureOutput` | 移除输出 |
| `commitConfig` | - | 提交配置 |
| `start` | - | 启动会话 |
| `stop` | - | 停止会话 |
| `release` | - | 释放会话 |

### CaptureSession 参数控制 API

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `setZoomRatio` | `ratio: number` | 设置变焦比例 |
| `getZoomRatio` | - | 获取变焦比例 |
| `setFocusMode` | `mode: FocusMode` | 设置对焦模式 |
| `getFocusMode` | - | 获取对焦模式 |
| `setFocusPoint` | `point: Point` | 设置对焦点 |
| `setExposureMode` | `mode: ExposureMode` | 设置曝光模式 |
| `setExposureBias` | `bias: number` | 设置曝光补偿 |
| `setFlashMode` | `mode: FlashMode` | 设置闪光灯模式 |
| `setVideoStabilizationMode` | `mode: VideoStabilizationMode` | 设置防抖模式 |
| `setFilter` | `filter: FilterType` | 设置滤镜 |
| `setBeauty` | `type: BeautyType, value: number` | 设置美颜 |

### PhotoOutput API

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `capture` | `settings?: PhotoCaptureSetting` | 拍照 |
| `confirmCapture` | - | 确认拍照 |
| `release` | - | 释放 |
| `enableMovingPhoto` | `enable: boolean` | 启用动态照片 |
| `enableMirror` | `enable: boolean` | 启用镜像 |
| `isMovingPhotoSupported` | - | 是否支持动态照片 |

### PreviewOutput API

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `start` | - | 开始预览 |
| `stop` | - | 停止预览 |
| `release` | - | 释放 |
| `setFrameRate` | `min: number, max: number` | 设置帧率 |
| `getActiveFrameRate` | - | 获取当前帧率 |

### VideoOutput API

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `start` | - | 开始录像 |
| `stop` | - | 停止录像 |
| `release` | - | 释放 |
| `setFrameRate` | `min: number, max: number` | 设置帧率 |
| `enableMirror` | `enable: boolean` | 启用镜像 |

### CameraInput API

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `open` | - | 打开相机 |
| `close` | - | 关闭相机 |
| `release` | - | 释放 |

### 事件监听

所有主要类都支持事件监听:

```typescript
// 注册事件
photoOutput.on('photoAvailable', (photo: Photo) => {
    // 处理照片
});

// 注册一次性事件
previewOutput.once('frameStart', (info: FrameInfo) => {
    // 处理帧开始
});

// 取消监听
photoOutput.off('photoAvailable', callback);
```

### 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `CAMERA_OK` | 0 | 成功 |
| `CAMERA_NO_PERMISSION` | 7400101 | 权限不足 |
| `CAMERA_INVALID_ARG` | 7400102 | 参数错误 |
| `CAMERA_SERVICE_ERROR` | 7400201 | 服务错误 |
| `CAMERA_DEVICE_ERROR` | 7400202 | 设备错误 |
| `CAMERA_DEVICE_OCCUPIED` | 7400203 | 设备被占用 |
| `CAMERA_CONFIG_ERROR` | 7400204 | 配置错误 |

---

## 2. NDK API (C/C++)

### 头文件包含

```c
#include "multimedia/camera_framework/camera_manager.h"
```

### CameraManager API

| 函数 | 参数 | 说明 |
|------|------|------|
| `OH_Camera_GetCameraManager` | - | 获取管理器 |
| `OH_CameraManager_GetSupportedCameras` | `manager, **cameras, *size` | 获取相机列表 |
| `OH_CameraManager_GetSupportedCameraOutputCapability` | `manager, device, **capability` | 获取能力 |
| `OH_CameraManager_CreateCameraInput` | `manager, device, **input` | 创建输入 |
| `OH_CameraManager_CreateCaptureSession` | `manager, **session` | 创建会话 |
| `OH_CameraManager_CreatePreviewOutput` | `manager, profile, surfaceId, **output` | 创建预览 |
| `OH_CameraManager_CreatePhotoOutput` | `manager, profile, surfaceId, **output` | 创建拍照 |
| `OH_CameraManager_CreateVideoOutput` | `manager, profile, surfaceId, **output` | 创建录像 |

### CameraInput API

| 函数 | 参数 | 说明 |
|------|------|------|
| `OH_CameraInput_Open` | `input` | 打开相机 |
| `OH_CameraInput_Close` | `input` | 关闭相机 |
| `OH_CameraInput_Release` | `input` | 释放 |

### CaptureSession API

| 函数 | 参数 | 说明 |
|------|------|------|
| `OH_CaptureSession_BeginConfig` | `session` | 开始配置 |
| `OH_CaptureSession_AddInput` | `session, input` | 添加输入 |
| `OH_CaptureSession_AddOutput` | `session, output` | 添加输出 |
| `OH_CaptureSession_CommitConfig` | `session` | 提交配置 |
| `OH_CaptureSession_Start` | `session` | 启动 |
| `OH_CaptureSession_Stop` | `session` | 停止 |
| `OH_CaptureSession_Release` | `session` | 释放 |

### 回调设置

```c
// 设置相机状态回调
CameraManager_Callbacks callbacks = {
    .onCameraStatus = OnCameraStatusCallback,
    .onFlashlightStatus = OnFlashlightStatusCallback
};
OH_CameraManager_RegisterCallback(manager, &callbacks);

// 设置拍照回调
PhotoOutput_Callbacks photoCallbacks = {
    .onPhotoAvailable = OnPhotoAvailable
};
OH_PhotoOutput_RegisterCallback(photoOutput, &photoCallbacks);
```

---

## 3. IPC 接口 (IDL)

### 主要服务

| 服务 | SAID | IDL 文件 | 说明 |
|------|------|----------|------|
| CameraService | 3008 | `ICameraService.idl` | 主服务 |
| DeferredProcessing | 动态 | `IDeferredPhotoProcessingSession.idl` | 延迟处理 |

### ICameraService 方法 (关键)

| IPC Code | 方法名 | 参数 | 返回 |
|----------|--------|------|------|
| 0 | `CreateCameraDevice` | `cameraId: string` | `ICameraDeviceService` |
| 4 | `GetSupportedCameras` | - | `CameraDevice[]` |
| 5 | `CreateCaptureSession` | `modeName: string` | `ICaptureSession` |
| 6 | `CreatePhotoOutput` | `streamType, surfaceId` | `IStreamCapture` |
| 7 | `CreatePreviewOutput` | `profile, surfaceId` | `IStreamRepeat` |
| 9 | `CreateVideoOutput` | `profile, surfaceId` | `IStreamRepeat` |
| 12 | `MuteCamera` | `muteMode: boolean` | - |
| 21 | `CreateDeferredPhotoProcessingSession` | - | `IDeferredPhotoProcessingSession` |

### ICaptureSession 方法 (关键)

| IPC Code | 方法名 | 说明 |
|----------|--------|------|
| 0 | `BeginConfig` | 开始配置 |
| 1 | `AddInput` | 添加输入 |
| 3 | `AddOutput` | 添加输出 |
| 6 | `CommitConfig` | 提交配置 |
| 7 | `Start` | 启动 |
| 8 | `Stop` | 停止 |
| 9 | `Release` | 释放 |

---

## 4. 配置文件

### 权限配置 (module.json5)

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.CAMERA",
        "reason": "$string:camera_permission_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.MICROPHONE",
        "reason": "$string:mic_permission_reason"
      }
    ]
  }
}
```

### Feature 配置 (multimedia_camera_framework.gni)

```gn
declare_args() {
  camera_framework_feature_moving_photo = true
  camera_framework_feature_deferred = true
  camera_framework_feature_movie_file = true
  camera_framework_feature_media_stream = true
  camera_framework_feature_camera_rotate_plugin = false
}
```

---

## API 对比表

| 功能 | JS API | NDK API | Inner API |
|------|--------|---------|-----------|
| 获取管理器 | `camera.getCameraManager()` | `OH_Camera_GetCameraManager()` | `CameraManager::GetInstance()` |
| 获取相机 | `getSupportedCameras()` | `OH_CameraManager_GetSupportedCameras()` | `GetSupportedCameras()` |
| 创建输入 | `createCameraInput()` | `OH_CameraManager_CreateCameraInput()` | `CreateCameraInput()` |
| 创建会话 | `createCaptureSession()` | `OH_CameraManager_CreateCaptureSession()` | `CreateCaptureSession()` |
| 开始配置 | `beginConfig()` | `OH_CaptureSession_BeginConfig()` | `BeginConfig()` |
| 提交配置 | `commitConfig()` | `OH_CaptureSession_CommitConfig()` | `CommitConfig()` |
| 启动 | `start()` | `OH_CaptureSession_Start()` | `Start()` |
| 拍照 | `capture()` | `OH_PhotoOutput_Capture()` | `Capture()` |

---

## 下一步阅读

- [构建与产物](./07_Build.md) - GN Targets 和编译产物
- [架构与数据流](./02_Architecture.md) - API 调用流程
