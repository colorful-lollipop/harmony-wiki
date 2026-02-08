# 目录结构与模块职责

## 顶层目录概览

```
distributed_camera/
├── common/                         # [公共模块] 工具类、日志、事件
├── interfaces/                      # [对外接口] Native C++ SDK
│   └── inner_kits/native_cpp/
│       ├── camera_source/          # Source SDK
│       └── camera_sink/            # Sink SDK
├── sa_profile/                     # [SA配置] SystemAbility 配置
├── services/                       # [服务实现] 核心业务逻辑
│   └── cameraservice/
│       ├── sourceservice/          # Source SA (4803)
│       ├── sinkservice/            # Sink SA (4804)
│       ├── cameraoperator/         # 相机操作抽象
│       │   ├── client/             # 相机客户端
│       │   └── handler/            # 相机处理器
│       └── base/                   # 公共基础
├── channel/                        # [通道模块] 软总线连接
├── data_process/                   # [处理模块] 数据编解码/缩放
├── test/                           # 测试代码
├── figures/                        # 文档图片
└── wiki/                          # Wiki 文档
```

---

## 模块详解

### 1. 公共模块 (common/)

**职责**：提供全局可复用的工具类和常量定义

| 目录/文件 | 职责 |
|-----------|------|
| `include/constants/` | 错误码、常量定义 |
| `include/utils/` | 工具类头文件 |
| `src/utils/` | 工具类实现 |

**关键文件**：
- `distributed_camera_errno.h` — 错误码定义
- `distributed_camera_constants.h` — 常量 (DID_MAX_SIZE, PARAM_MAX_SIZE 等)
- `dh_log.h` — 日志宏定义
- `dcamera_hisysevent_adapter.h` — 事件上报适配器
- `data_buffer.h/cpp` — 数据缓冲区

**构建产物**：`libdistributed_camera_utils.so`

---

### 2. 对外接口模块 (interfaces/inner_kits/native_cpp/)

**职责**：提供 C++ SDK 接口，供其他系统服务调用

#### 2.1 camera_source/ (Source SDK)

**职责**：相机源端 SDK，主控端使用

| 文件 | 职责 |
|------|------|
| `include/idistributed_camera_source.h` | IPC 接口定义 |
| `include/dcamera_source_handler.h` | Handler 接口 |
| `include/dcamera_source_handler_ipc.h` | IPC Handler |
| `include/dcamera_source_load_callback.h` | SA 加载回调 |
| `include/callback/idcamera_source_callback.h` | 回调接口 |
| `src/dcamera_source_handler.cpp` | Handler 实现 |

**导出函数**：
```cpp
IDistributedHardwareSource* GetSourceHardwareHandler();
```

**构建产物**：`libdistributed_camera_source_sdk.so`

#### 2.2 camera_sink/ (Sink SDK)

**职责**：相机被控端 SDK，被控端使用

| 文件 | 职责 |
|------|------|
| `include/idistributed_camera_sink.h` | IPC 接口定义 |
| `include/dcamera_sink_handler.h` | Handler 接口 |
| `include/dcamera_sink_handler_ipc.h` | IPC Handler |
| `include/dcamera_sink_load_callback.h` | SA 加载回调 |
| `include/callback/idcamera_sink_callback.h` | 回调接口 |
| `src/dcamera_sink_handler.cpp` | Handler 实现 |

**导出函数**：
```cpp
IDistributedHardwareSink* GetSinkHardwareHandler();
```

**构建产物**：`libdistributed_camera_sink_sdk.so`

---

### 3. SA 配置模块 (sa_profile/)

**职责**：SystemAbility 配置和进程权限声明

| 文件 | 职责 |
|------|------|
| `dcamera.cfg` | 进程权限配置 (uid/gid/permission/SELinux) |
| `4803.json` | Source SA 配置 (ID: 4803) |
| `4804.json` | Sink SA 配置 (ID: 4804) |

**权限声明** (`dcamera.cfg`)：
```json
{
    "permission": [
        "ohos.permission.ACCESS_SERVICE_DM",
        "ohos.permission.DISTRIBUTED_DATASYNC",
        "ohos.permission.DISTRIBUTED_SOFTBUS_CENTER",
        "ohos.permission.CAMERA",
        "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE"
    ]
}
```

---

### 4. 服务实现模块 (services/cameraservice/)

#### 4.1 sourceservice/ (Source SA: 4803)

**职责**：主控端服务，管理设备发现、流传输

```
sourceservice/
├── include/distributedcamera/
│   ├── distributed_camera_source_service.h/cpp  # 服务主类
│   ├── distributed_camera_source_stub.h/cpp     # IPC Stub
│   └── dcamera_source_callback_proxy.h/cpp       # 回调代理
├── include/distributedcameramgr/
│   ├── dcamera_source_dev.h/cpp           # 设备管理
│   ├── dcameracontrol/
│   │   └── dcamera_source_controller.h/cpp    # 控制层
│   ├── dcameradata/
│   │   ├── dcamera_source_data_process.h/cpp   # 数据处理
│   │   ├── dcamera_stream_data_process.h/cpp  # 流处理
│   │   └── feeding_smoother/                  # 帧率平滑
│   ├── dcamerahdf/
│   │   └── dcamera_provider_callback_impl.cpp  # HDF 回调
│   └── dcamerastate/
│       ├── dcamera_source_state_machine.h/cpp  # 状态机
│       └── *State.cpp                          # 各状态实现
└── BUILD.gn
```

**构建产物**：`libdistributed_camera_source.so`

#### 4.2 sinkservice/ (Sink SA: 4804)

**职责**：被控端服务，管理本地相机访问、隐私授权

```
sinkservice/
├── include/distributedcamera/
│   ├── distributed_camera_sink_service.h/cpp  # 服务主类
│   ├── distributed_camera_sink_stub.h/cpp     # IPC Stub
│   └── dcamera_sink_callback_proxy.h/cpp       # 回调代理
├── include/distributedcameramgr/
│   ├── dcamera_sink_controller.h/cpp     # 控制层
│   ├── dcamera_sink_data_process.h/cpp   # 数据处理
│   ├── dcamera_sink_output.h/cpp         # 输出管理
│   ├── dcamera_sink_access_control.h/cpp # 访问控制
│   └── callback/
│       └── dcamera_sink_output_result_callback.cpp
└── BUILD.gn
```

**构建产物**：`libdistributed_camera_sink.so`

#### 4.3 cameraoperator/ (相机操作抽象)

**client/职责**：本地相机客户端封装

| 文件 | 职责 |
|------|------|
| `dcamera_client.h/cpp` | 客户端主类 |
| `dcamera_session_callback.h/cpp` | 会话回调 |
| `dcamera_preview_callback.h/cpp` | 预览回调 |
| `dcamera_video_callback.h/cpp` | 录像回调 |
| `dcamera_photo_callback.h/cpp` | 拍照回调 |

**构建产物**：`libdistributed_camera_client.so`

**handler/职责**：相机事件处理

| 文件 | 职责 |
|------|------|
| `dcamera_handler.h/cpp` | 事件处理器 |
| `dcamera_manager_callback.h/cpp` | 管理回调 |

**构建产物**：`libdistributed_camera_handler.so`

---

### 5. 通道模块 (channel/)

**职责**：通过软总线建立主被控端连接

```
channel/
├── include/
│   ├── dcamera_channel.h/cpp         # 通道接口
│   ├── dcamera_channel_source.h/cpp   # Source 通道
│   ├── dcamera_channel_sink.h/cpp    # Sink 通道
│   └── allconnect/                    # AllConnect 管理
└── src/
    ├── dcamera_softbus_adapter.cpp    # 软总线适配器
    ├── dcamera_softbus_session.cpp    # 会话管理
    ├── dcamera_softbus_latency.cpp   # 延迟控制
    └── distributed_camera_allconnect_manager.cpp  # AllConnect
```

**构建产物**：`libdistributed_camera_channel.so`

---

### 6. 数据处理模块 (data_process/)

**职责**：图像数据编解码、缩放、帧率控制

```
data_process/
├── include/
│   ├── pipeline/                     # 处理流水线
│   │   ├── abstract_data_process.h/cpp
│   │   ├── dcamera_pipeline_source.h/cpp
│   │   └── dcamera_pipeline_sink.h/cpp
│   ├── interfaces/                  # 接口定义
│   ├── pipeline_node/
│   │   ├── multimedia_codec/
│   │   │   ├── encoder/            # 编码器
│   │   │   └── decoder/            # 解码器
│   │   ├── fpscontroller/          # 帧率控制
│   │   └── scale_conversion/       # 缩放转换
│   └── utils/                       # 工具类
└── src/
```

**构建产物**：`libdistributed_camera_data_process.so`

---

## 模块依赖关系

```
                    ┌─────────────────────┐
                    │   common/utils      │
                    │  (libdcamera_utils) │
                    └──────────┬──────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
        ▼                      ▼                      ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│   channel     │    │  data_process │    │ cameraoperator│
│ (dcam_channel)│    │(dcam_dataproc)│    │  client/handler│
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                      │
        └────────────────────┼──────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  cameraservice  │
                    │ sourceservice/  │
                    │ sinkservice     │
                    └─────────────────┘
```

---

## 稳定性标注

| 层级 | 模块 | 稳定性 | 依据 |
|------|------|--------|------|
| **稳定** | common/ | 高 | 基础工具，接口稳定 |
| **稳定** | interfaces/inner_kits | 高 | SDK 接口，版本兼容 |
| **稳定** | services/cameraservice/base | 高 | 公共基础代码 |
| **中等** | services/cameraservice/sourceservice | 中 | 核心业务，接口变化需谨慎 |
| **中等** | services/cameraservice/sinkservice | 中 | 核心业务，接口变化需谨慎 |
| **实验** | services/data_process | 中 | 视频处理，可能优化调整 |
| **实验** | services/channel | 中 | 通信模块，协议可能演进 |
