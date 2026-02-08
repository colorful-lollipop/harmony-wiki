# 项目概览 (Overview)

> OpenHarmony Camera Framework - 相机能力提供方

---

## 一句话定义

**camera_framework** 是 OpenHarmony 的相机标准框架，为上层应用提供统一的相机功能接口（预览、拍照、录像），负责相机硬件的生命周期管理、参数配置、数据流处理和隐私权限控制。

---

## 能力边界

### 能做什么 ✅

| 功能类别 | 具体能力 |
|----------|----------|
| **设备管理** | 枚举可用相机设备、打开/关闭相机、切换摄像头 |
| **预览** | 实时预览、预览参数配置、Surface 渲染 |
| **拍照** | 单拍/连拍、动态照片、RAW 输出、深度数据 |
| **录像** | 视频录制、音频同步、元数据输出 |
| **参数控制** | 变焦、对焦、曝光、闪光灯、滤镜、防抖 |
| **特殊模式** | 人像、夜景、全景、慢动作、延时摄影、专业模式等 |
| **后期处理** | 延迟处理（Deferred Processing）、云增强 |
| **隐私安全** | 权限检查、隐私提示、应用隔离 |

### 不能做什么 ❌

| 功能 | 说明 | 替代方案 |
|------|------|----------|
| 直接访问传感器 | 框架封装了 HAL 层 | 通过 HDI 接口访问 |
| 跨设备相机共享 | 本框架仅支持本地相机 | 分布式相机服务 |
| 图像编解码 | 由 media_library/image_framework 提供 | 调用对应服务 |
| 视频压缩 | 由 av_codec 服务处理 | 调用 av_codec |

---

## 运行环境

### 系统要求

| 属性 | 要求 |
|------|------|
| **系统类型** | OpenHarmony standard (标准系统) |
| **系统能力** | SystemCapability.Multimedia.Camera.Core |
| **最低 API** | API 10+ |
| **权限要求** | ohos.permission.CAMERA |

### 依赖系统服务

| 服务 | 用途 | SAID |
|------|------|------|
| **camera_service** | 核心相机服务 | 3008 |
| **deferred_processing_service** | 延迟照片/视频处理 | 动态分配 |
| **access_token** | 权限检查 | 系统服务 |
| **app_manager** | 应用状态监听 | 系统服务 |
| **window_manager** | Surface/窗口管理 | 系统服务 |
| **hdi_camera_host** | 相机 HAL 接口 | HDI 服务 |

### 权限要求

| 权限 | 权限级别 | 用途 |
|------|----------|------|
| `ohos.permission.CAMERA` | normal | 基础相机访问 |
| `ohos.permission.MICROPHONE` | normal | 录音录像 |

---

## 快速开始

### JavaScript 最小示例

```typescript
import camera from '@ohos.multimedia.camera';
import { BusinessError } from '@ohos.base';

// 1. 获取 CameraManager
let cameraManager = camera.getCameraManager(context);

// 2. 获取可用相机
let cameras = cameraManager.getSupportedCameras();
if (cameras.length === 0) {
    console.error('No camera available');
    return;
}

// 3. 创建相机输入
let cameraInput = cameraManager.createCameraInput(cameras[0]);

// 4. 创建会话
let session = cameraManager.createCaptureSession();

// 5. 配置会话
session.beginConfig();
session.addInput(cameraInput);

// 创建预览输出（使用 XComponent 的 Surface）
let previewOutput = cameraManager.createPreviewOutput(profile, surfaceId);
session.addOutput(previewOutput);

session.commitConfig();

// 6. 启动预览
await cameraInput.open();
await session.start();
```

### Native C++ 最小示例

```cpp
#include "input/camera_manager.h"

using namespace OHOS::CameraStandard;

// 1. 获取 CameraManager 单例
sptr<CameraManager> camManagerObj = CameraManager::GetInstance();

// 2. 获取相机列表
std::vector<sptr<CameraDevice>> cameraObjList = camManagerObj->GetSupportedCameras();

// 3. 创建相机输入
sptr<CaptureInput> cameraInput = camManagerObj->CreateCameraInput(cameraObjList[0]);
cameraInput->Open();

// 4. 创建并配置会话
sptr<CaptureSession> captureSession = camManagerObj->CreateCaptureSession();
captureSession->BeginConfig();
captureSession->AddInput(cameraInput);

// 5. 创建预览输出
sptr<CaptureOutput> previewOutput = camManagerObj->CreatePreviewOutput(previewProfile, surface);
captureSession->AddOutput(previewOutput);

captureSession->CommitConfig();
captureSession->Start();
```

### NDK (C API) 最小示例

```c
#include "multimedia/camera_framework/camera_manager.h"

// 1. 获取 CameraManager
Camera_Manager* cameraManager = OH_Camera_GetCameraManager();

// 2. 获取相机列表
Camera_Device** cameras = NULL;
uint32_t size = 0;
OH_CameraManager_GetSupportedCameras(cameraManager, &cameras, &size);

// 3. 创建输入
Camera_Input* input = NULL;
OH_CameraManager_CreateCameraInput(cameraManager, cameras[0], &input);
OH_CameraInput_Open(input);

// 4. 创建会话
Camera_CaptureSession* session = NULL;
OH_CameraManager_CreateCaptureSession(cameraManager, &session);

// 5. 配置
OH_CaptureSession_BeginConfig(session);
OH_CaptureSession_AddInput(session, input);
Camera_PreviewOutput* previewOutput = NULL;
OH_CameraManager_CreatePreviewOutput(cameraManager, profile, surfaceId, &previewOutput);
OH_CaptureSession_AddOutput(session, previewOutput);
OH_CaptureSession_CommitConfig(session);

OH_CaptureSession_Start(session);
```

---

## 项目元信息

| 属性 | 内容 |
|------|------|
| **项目名称** | @ohos/camera_framework |
| **仓库路径** | foundation/multimedia/camera_framework |
| **子系统** | multimedia |
| **版本** | 3.1 |
| **License** | Apache License 2.0 |
| **系统能力** | SystemCapability.Multimedia.Camera.Core |
| **SAID** | 3008 (camera_service) |

---

## 相关资源

- **官方文档**: [OpenHarmony Camera 开发指南](https://docs.openharmony.cn/pages/v5.0/zh-cn/application-dev/media/camera/camera-preparation.md)
- **TypeScript 定义**: `interfaces/kits/js/camera_napi/@ohos.multimedia.camera.d.ts`
- **NDK 头文件**: `interfaces/kits/native/include/camera/`
- **示例代码**: `frameworks/js/camera_napi/demo/`
- **Gitee 仓库**: [multimedia_camera_framework](https://gitee.com/openharmony/multimedia_camera_framework)

---

## 下一步阅读

- [架构与数据流](./02_Architecture.md) - 了解组件关系和调用流程
- [攻击面分析](./05_AttackSurface.md) - 了解安全关键入口
- [对外接口文档](./04_Interface.md) - 完整的 API 清单

---

## 文档信息

| 项 | 内容 |
|----|------|
| **文档版本** | v1.0 |
| **更新日期** | 2025-02-07 |
| **作者** | OpenHarmony Wiki Agent |
