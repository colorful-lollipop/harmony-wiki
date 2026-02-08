# 目录结构与代码地图 (Code Map)

> OpenHarmony Camera Framework - 代码导航指南

---

## 顶层目录结构

```
foundation/multimedia/camera_framework/
├── bundle.json                    # 组件定义和依赖
├── multimedia_camera_framework.gni # GN 构建配置
├── ohos.build                     # 旧版构建配置
├── README.md                      # 项目说明
├── sa_profile/                    # System Ability 配置
│   ├── 3008.json                  # camera_service SAID 配置
│   └── BUILD.gn
│
├── common/                        # 公共基础设施
│   ├── include/                   # 公共头文件
│   ├── src/                       # 公共实现
│   └── utils/                     # 工具类
│       ├── media_manager/         # 媒体管理代理
│       ├── moving_photo/          # 动态照片代理
│       └── photo_asset_proxy.h    # 照片资源代理
│
├── dynamic_libs/                  # 动态加载库 (9个)
│   ├── av_codec/                  # 音视频编解码适配
│   ├── camera_notification/       # 通知服务适配
│   ├── dfx/                       # DFX 报告
│   ├── image_effect/              # 图像效果适配
│   ├── image_framework/           # 图片框架适配
│   ├── media_library/             # 媒体库适配
│   ├── media_manager/             # 媒体管理适配
│   ├── moving_photo/              # 动态照片处理
│   ├── watermark_exif_metadata/   # 水印/EXIF 元数据
│   └── xcomponent_controller/     # XComponent 控制
│
├── frameworks/                    # 框架层实现
│   ├── cj/                        # Cangjie FFI 绑定
│   ├── js/
│   │   └── camera_napi/           # JavaScript N-API
│   │       ├── demo/              # 演示应用
│   │       ├── src/               # N-API 实现
│   │       └── include/           # N-API 头文件
│   ├── native/
│   │   ├── camera/
│   │   │   ├── base/              # 核心 Native 框架
│   │   │   ├── extension/         # 扩展框架 (系统 API)
│   │   │   └── test/              # 测试代码
│   │   └── ndk/                   # NDK C API
│   └── taihe/                     # Taihe 框架绑定
│
├── interfaces/                    # 接口定义
│   ├── inner_api/                 # 内部 Native API
│   │   └── native/camera/include/
│   │       ├── input/             # CameraManager, CameraInput
│   │       ├── output/            # PhotoOutput, PreviewOutput
│   │       ├── session/           # CaptureSession
│   │       └── utils/             # 工具类
│   └── kits/
│       ├── js/camera_napi/        # JS API 定义
│       │   ├── @ohos.multimedia.camera.d.ts
│       │   └── @ohos.multimedia.cameraPicker.d.ts
│       └── native/include/camera/ # NDK 头文件
│
├── mediastream/                   # 媒体流处理
│   ├── include/                   # 头文件
│   └── src/                       # 实现
│
├── moviefile/                     # 视频文件处理
│   ├── include/
│   └── src/
│
└── services/                      # 服务层
    ├── camera_service/            # 核心相机服务 (SAID 3008)
    │   ├── binder/                # IPC Binder 实现
    │   ├── include/               # 头文件
    │   ├── idls/                  # IDL 接口定义
    │   └── src/                   # 服务实现
    └── deferred_processing_service/ # 延迟处理服务
```

---

## 核心文件定位

### 服务层 (Service Layer)

| 文件 | 职责 | 关键类 |
|------|------|--------|
| `services/camera_service/src/hcamera_service.cpp` | 核心服务实现 | HCameraService |
| `services/camera_service/src/hcamera_device.cpp` | 设备管理 | HCameraDevice |
| `services/camera_service/src/hcapture_session.cpp` | 会话管理 | HCaptureSession |
| `services/camera_service/src/hstream_operator.cpp` | 流操作管理 | HStreamOperator |
| `services/camera_service/src/hstream_repeat.cpp` | 预览/视频流 | HStreamRepeat |
| `services/camera_service/src/hstream_capture.cpp` | 拍照流 | HStreamCapture |
| `services/camera_service/src/camera_util.cpp` | 工具函数 | CheckPermission |
| `services/camera_service/src/camera_privacy.cpp` | 隐私控制 | CameraPrivacy |

### 框架层 (Framework Layer)

| 文件 | 职责 | 关键类 |
|------|------|--------|
| `frameworks/native/camera/base/src/input/camera_manager.cpp` | 管理器实现 | CameraManager |
| `frameworks/native/camera/base/src/session/capture_session.cpp` | 会话实现 | CaptureSession |
| `frameworks/native/camera/base/src/output/photo_output.cpp` | 拍照输出 | PhotoOutput |
| `frameworks/native/camera/base/src/output/preview_output.cpp` | 预览输出 | PreviewOutput |
| `frameworks/native/camera/base/src/output/video_output.cpp` | 视频输出 | VideoOutput |

### N-API 层

| 文件 | 职责 | 关键类 |
|------|------|--------|
| `frameworks/js/camera_napi/src/native_module_ohos_camera.cpp` | 模块注册 | - |
| `frameworks/js/camera_napi/src/input/camera_manager_napi.cpp` | 管理器 N-API | CameraManagerNapi |
| `frameworks/js/camera_napi/src/session/camera_session_napi.cpp` | 会话 N-API | CameraSessionNapi |
| `frameworks/js/camera_napi/src/output/photo_output_napi.cpp` | 拍照 N-API | PhotoOutputNapi |
| `frameworks/js/camera_napi/src/output/preview_output_napi.cpp` | 预览 N-API | PreviewOutputNapi |

### IDL 接口

| 文件 | 职责 |
|------|------|
| `services/camera_service/idls/ICameraService.idl` | 主服务接口 |
| `services/camera_service/idls/ICameraDeviceService.idl` | 设备服务接口 |
| `services/camera_service/idls/ICaptureSession.idl` | 会话接口 |
| `services/camera_service/idls/IStreamCapture.idl` | 拍照流接口 |
| `services/camera_service/idls/IStreamRepeat.idl` | 预览流接口 |

---

## 功能到文件映射

### 相机设备管理

| 功能 | 入口文件 | 服务文件 |
|------|----------|----------|
| 获取相机列表 | `camera_manager.cpp` | `hcamera_service.cpp` |
| 打开相机 | `camera_input.cpp` | `hcamera_device.cpp` |
| 关闭相机 | `camera_input.cpp` | `hcamera_device.cpp` |
| 权限检查 | - | `camera_util.cpp:390` |

### 会话管理

| 功能 | 入口文件 | 服务文件 |
|------|----------|----------|
| 创建会话 | `camera_manager.cpp` | `hcamera_service.cpp` |
| 配置会话 | `capture_session.cpp` | `hcapture_session.cpp` |
| 启动/停止 | `capture_session.cpp` | `hcapture_session.cpp` |
| 释放会话 | `capture_session.cpp` | `hcapture_session.cpp` |

### 流管理

| 功能 | 入口文件 | 服务文件 |
|------|----------|----------|
| 创建预览输出 | `camera_manager.cpp` | `hstream_repeat.cpp` |
| 创建拍照输出 | `camera_manager.cpp` | `hstream_capture.cpp` |
| 创建视频输出 | `camera_manager.cpp` | `hstream_repeat.cpp` |
| 触发拍照 | `photo_output.cpp` | `hstream_capture.cpp` |

### 参数控制

| 功能 | 入口文件 | 服务文件 |
|------|----------|----------|
| 变焦 | `capture_session.cpp` | `hcapture_session.cpp` |
| 对焦 | `capture_session.cpp` | `hcapture_session.cpp` |
| 曝光 | `capture_session.cpp` | `hcapture_session.cpp` |
| 闪光灯 | `camera_input.cpp` | `hcamera_device.cpp` |

---

## 关键调用链

### 调用链 1: 打开相机

```
JS: cameraInput.open()
  ↓
native_module_ohos_camera.cpp → CameraInputNapi::Open()
  ↓
camera_input.cpp → CameraInput::Open()
  ↓ IPC
hcamera_device.cpp → HCameraDevice::Open()
  ↓
HCameraHostManager::OpenCameraDevice()
  ↓ HDI
Camera HAL → ICameraDevice::Open()
```

### 调用链 2: 配置会话

```
JS: session.beginConfig() / addInput() / addOutput() / commitConfig()
  ↓
camera_session_napi.cpp
  ↓
capture_session.cpp
  ↓ IPC
hcapture_session.cpp → HCaptureSession::CommitConfig()
  ↓
hstream_operator.cpp → HStreamOperator::LinkInputAndOutputs()
  ↓ HDI
IStreamOperator::CreateStreams() / CommitStreams()
```

### 调用链 3: 拍照

```
JS: photoOutput.capture()
  ↓
photo_output_napi.cpp
  ↓
photo_output.cpp → PhotoOutput::Capture()
  ↓ IPC
hstream_capture.cpp → HStreamCapture::Capture()
  ↓ HDI
IStreamOperator::Capture()
  ↓
[HAL 处理完成回调]
  ↓
hstream_capture.cpp → OnPhotoAvailable()
  ↓ IPC
photo_output.cpp → 回调到 JS
```

---

## 快速导航

### 按任务查找文件

| 任务 | 查找位置 |
|------|----------|
| **添加新 JS API** | `frameworks/js/camera_napi/src/` |
| **修改相机 HAL 接口** | `services/camera_service/idls/` |
| **修改权限检查** | `services/camera_service/src/camera_util.cpp:390` |
| **修改会话逻辑** | `services/camera_service/src/hcapture_session.cpp` |
| **修改流管理** | `services/camera_service/src/hstream_operator.cpp` |
| **添加动态库** | `dynamic_libs/BUILD.gn` |
| **修改构建配置** | `multimedia_camera_framework.gni` |

### 按问题查找文件

| 问题类型 | 检查文件 |
|----------|----------|
| 权限问题 | `camera_util.cpp`, `camera_privacy.cpp` |
| 会话问题 | `hcapture_session.cpp` |
| 预览问题 | `hstream_repeat.cpp` |
| 拍照问题 | `hstream_capture.cpp` |
| 设备问题 | `hcamera_device.cpp` |
| IPC 问题 | `*_stub.cpp`, `*_proxy.cpp` |
| N-API 崩溃 | `*_napi.cpp` |

---

## 代码统计

| 目录 | 文件数 | 主要语言 | 说明 |
|------|--------|----------|------|
| `services/camera_service/` | 150+ | C++ | 核心服务实现 |
| `frameworks/native/camera/` | 200+ | C++ | Native 框架 |
| `frameworks/js/camera_napi/` | 80+ | C++ | JS 绑定 |
| `interfaces/` | 100+ | C++/IDL/TS | 接口定义 |
| `dynamic_libs/` | 70+ | C++ | 动态库 |
| `test/` | 100+ | C++ | 测试代码 |

---

## 下一步阅读

- [对外接口文档](./04_Interface.md) - API 详细说明
- [构建与产物](./07_Build.md) - GN Targets 和编译产物
- [内部实现细节](./08_Internals.md) - 设计模式和资源生命周期
