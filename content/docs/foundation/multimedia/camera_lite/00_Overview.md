# 00 - 项目概览

> Camera_Lite 组件全景介绍

---

## 组件定位

Camera_Lite 是 OpenHarmony **媒体子系统** 的轻量级相机组件，专为 **mini** 和 **small** 系统类型设计。

### 核心职责

1. **相机能力抽象** - 提供统一的相机硬件访问接口
2. **多媒体支持** - 支持预览、录像、拍照、回调四种工作模式
3. **进程通信** - 支持 Binder IPC 和 Passthrough 两种模式
4. **资源管理** - 管理相机设备生命周期和内存资源

### 在系统中的位置

```
应用层 (Application)
    ↓
相机应用 (Camera App)
    ↓
CameraKit (本组件对外API) ← 你在这里
    ↓
CameraServer (相机服务)
    ↓
HAL层 (硬件抽象层)
    ↓
相机驱动 (Camera Driver)
```

**证据**: 
- `bundle.json` 第14行: `"subsystem": "multimedia"`
- `bundle.json` 第18-20行: `"adapted_system_type": ["mini", "small"]`

---

## 核心能力

### 功能特性

| 特性 | 说明 | 配置类型 |
|------|------|----------|
| **预览 (Preview)** | 实时取景显示 | FRAME_CONFIG_PREVIEW |
| **录像 (Record)** | H.264/H.265视频编码录制 | FRAME_CONFIG_RECORD |
| **拍照 (Capture)** | JPEG/HEVC单帧捕获 | FRAME_CONFIG_CAPTURE |
| **回调 (Callback)** | 原始YUV数据回调 | FRAME_CONFIG_CALLBACK |
| **多路流** | 支持预览+录像同时运行 | - |
| **参数配置** | AE/AF/白平衡/裁剪等 | FrameConfig参数 |

**证据**: `interfaces/kits/meta_data.h` 第38-86行定义了支持的参数键值

### 支持的格式

| 格式类型 | 具体格式 | 说明 |
|----------|----------|------|
| 图像 | YUV420, RAW12 | 原始数据 |
| 编码 | JPEG, H.264, H.265 | 压缩格式 |

**证据**: `interfaces/kits/meta_data.h` 第99-111行

---

## 运行环境

### 系统要求

| 项目 | 要求 |
|------|------|
| 系统类型 | mini / small |
| C++标准 | C++11及以上 |
| 最小权限 | ohos.permission.CAMERA |

**证据**: 
- `README_zh.md` 第38行: `C++11版本或以上`
- `frameworks/camera_kit.cpp` 第32行: `CheckSelfPermission("ohos.permission.CAMERA")`

### 硬件依赖

| 组件 | 用途 |
|------|------|
| 相机硬件 | 图像采集 |
| 编解码器 | H.264/H.265/JPEG编码 |
| 显示层 | 预览输出 |

**证据**: `services/impl/src/camera_device.cpp` 使用了 `HalCamera`, `CodecInterface`, `DisplayLayer`

### 软件依赖

**内部依赖**:
- `hilog_lite` - 日志输出
- `permission_lite` - 权限检查
- `surface_lite` - 图形Surface
- `media_utils_lite` - 媒体工具
- `ipc` / `samgr_lite` - IPC通信 (Binder模式)

**外部依赖**:
- `bounds_checking_function` - 安全函数库

**证据**: `bundle.json` 第24-31行

---

## 关键概念

### 1. 相机ID (Camera ID)

- 字符串类型，如 `"0"`, `"1"`
- 对应物理相机设备编号
- 通过 `CameraKit::GetCameraIds()` 获取

**证据**: `frameworks/camera_kit.cpp` 第40-43行

### 2. 能力 (CameraAbility)

描述相机支持的能力范围：
- 支持的图像尺寸列表
- 支持的AE模式
- 支持的AF模式

**证据**: `interfaces/kits/camera_ability.h`

### 3. 配置 (FrameConfig)

定义帧输出配置：
- 配置类型 (预览/录像/拍照/回调)
- Surface列表 (输出目标)
- 参数 (FPS、编码质量、裁剪区域等)

**证据**: `interfaces/kits/frame_config.h`

### 4. 运行模式

#### Binder模式 (默认)
- 客户端和服务端分离
- 通过IPC通信
- 适合多进程架构

#### Passthrough模式
- 客户端直接链接服务端代码
- 无IPC开销
- 适合单进程/高性能场景

**证据**: `frameworks/BUILD.gn` 第27-58行

---

## 目录结构

```
foundation/multimedia/camera_lite/
├── frameworks/                    # 框架层 (客户端)
│   ├── binder/                   # Binder IPC实现
│   │   ├── include/              # 头文件
│   │   └── src/                  # 实现
│   ├── passthrough/              # Passthrough模式实现
│   │   ├── include/
│   │   └── src/
│   ├── camera_kit.cpp            # CameraKit实现
│   ├── camera_manager.cpp        # CameraManager实现
│   ├── camera_impl.cpp           # Camera实现
│   ├── camera_config.cpp         # CameraConfig实现
│   ├── camera_ability.cpp        # CameraAbility实现
│   ├── camera_client.cpp         # 客户端基类
│   ├── camera_info_impl.cpp      # CameraInfo实现
│   ├── frame_config.cpp          # FrameConfig实现
│   ├── event_handler.cpp         # 事件处理
│   ├── camera_manager.h          # CameraManager声明
│   ├── camera_impl.h             # CameraImpl声明
│   ├── camera_ability_impl.h
│   ├── camera_info_impl.h
│   ├── camera_service_callback.h
│   └── BUILD.gn                  # 构建脚本
│
├── interfaces/                    # 接口层
│   └── kits/                     # 对外API
│       ├── camera_kit.h          # 主入口
│       ├── camera.h              # Camera接口
│       ├── camera_config.h       # 配置接口
│       ├── camera_ability.h      # 能力接口
│       ├── camera_info.h         # 信息接口
│       ├── frame_config.h        # 帧配置
│       ├── camera_state_callback.h   # 状态回调
│       ├── frame_state_callback.h    # 帧回调
│       ├── camera_device_callback.h  # 设备回调
│       ├── event_handler.h       # 事件处理器
│       └── meta_data.h           # 元数据定义
│
├── services/                      # 服务层 (服务端)
│   ├── impl/                     # 服务实现
│   │   ├── include/
│   │   │   ├── camera_service.h  # 相机服务
│   │   │   └── camera_device.h   # 相机设备
│   │   └── src/
│   │       ├── camera_service.cpp
│   │       └── camera_device.cpp
│   └── server/                   # 服务器
│       ├── include/
│       │   ├── camera_server.h   # IPC服务
│       │   └── camera_type.h     # 类型定义
│       └── src/
│           ├── camera_server.cpp
│           └── samgr_camera.cpp  # SA注册
│
├── test/                         # 测试代码 (忽略)
├── figures/                      # 文档图片
├── bundle.json                   # 组件配置
└── README.md / README_zh.md      # 项目说明
```

---

## 快速开始

### 1. 获取单例

```cpp
#include "camera_kit.h"

// 自动检查权限
OHOS::Media::CameraKit *cameraKit = OHOS::Media::CameraKit::GetInstance();
if (cameraKit == nullptr) {
    // 权限不足或初始化失败
    return;
}
```

### 2. 获取相机列表

```cpp
std::list<std::string> cameraIds = cameraKit->GetCameraIds();
for (auto &id : cameraIds) {
    // 处理每个相机ID
}
```

### 3. 获取相机能力

```cpp
const CameraAbility *ability = cameraKit->GetCameraAbility(cameraId);
std::list<CameraPicSize> sizes = ability->GetSupportedSizes(CAM_FORMAT_JPEG);
```

### 4. 创建相机

```cpp
class MyCameraStateCallback : public CameraStateCallback {
    void OnCreated(Camera &c) override { /* 创建成功 */ }
    void OnCreateFailed(const std::string cameraId, int32_t errorCode) override { /* 创建失败 */ }
    void OnReleased(Camera &c) override { /* 已释放 */ }
};

MyCameraStateCallback callback;
EventHandler handler;
cameraKit->CreateCamera(cameraId, callback, handler);
```

### 5. 配置并启动预览

```cpp
// 获取相机实例后，在OnCreated回调中
CameraConfig *config = CameraConfig::CreateCameraConfig();
config->SetFrameStateCallback(&frameCallback, &handler);
camera->Configure(*config);

// 创建FrameConfig
FrameConfig fc(FRAME_CONFIG_PREVIEW);
fc.AddSurface(*surface);  // surface需预先创建
camera->TriggerLoopingCapture(fc);
```

---

## 下一步

- 深入了解架构: [01_Architecture.md](01_Architecture.md)
- 查看完整API: [02_API_Reference.md](02_API_Reference.md)
- 了解构建方式: [04_Build_System.md](04_Build_System.md)
