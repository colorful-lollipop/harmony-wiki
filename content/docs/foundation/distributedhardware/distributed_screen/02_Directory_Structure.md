# 目录结构

## 目的与适用范围

本文档详细描述分布式屏幕项目的目录结构和模块职责。

---

## 顶层目录结构

```
/foundation/distributedhardware/distributed_screen
├── common/                    # 公共基础模块
├── interfaces/                # SDK接口层
├── sa_profile/                # SA配置文件
├── screenhandler/             # 硬件处理器
├── services/                  # 核心服务实现
├── figures/                   # 文档图片
├── bundle.json                # 组件配置
├── distributedscreen.gni      # GN构建变量
└── wiki/                      # 本Wiki文档
```

### 目录统计

**证据**: 基于glob和find结果

- **C++源文件**: ~49个（不含测试）
- **头文件**: ~70个（不含测试）
- **BUILD.gn**: 12个（不含测试）
- **SA配置文件**: 2个

---

## 详细目录说明

### common/ - 公共基础模块

**路径**: `/foundation/distributedhardware/distributed_screen/common/`

```
common/
├── include/                   # 公共头文件
│   ├── dscreen_constants.h    # 常量定义 (constants)
│   ├── dscreen_errcode.h      # 错误码定义
│   ├── dscreen_ipc_interface_code.h  # IPC命令码
│   ├── dscreen_log.h          # 日志宏定义
│   ├── dscreen_util.h         # 工具函数
│   ├── dscreen_json_util.h    # JSON工具
│   ├── dscreen_hisysevent.h   # 系统事件
│   └── dscreen_hitrace.h      # 性能跟踪
├── src/                       # 公共实现
│   ├── dscreen_util.cpp
│   ├── dscreen_json_util.cpp
│   └── dscreen_hisysevent.cpp
└── BUILD.gn                   # 构建: distributed_screen_utils
```

**职责**: 提供全模块共享的基础工具，包括错误码、常量、日志、JSON处理、系统事件等。

---

### interfaces/innerkits/native_cpp/ - SDK接口层

**路径**: `/foundation/distributedhardware/distributed_screen/interfaces/innerkits/native_cpp/`

```
interfaces/innerkits/native_cpp/
├── screen_source/             # 主控端(Source)SDK
│   ├── include/
│   │   ├── idscreen_source.h           # 接口定义 (IRemoteBroker)
│   │   ├── idscreen_source_callback.h  # 回调接口
│   │   ├── dscreen_source_handler.h    # Handler类
│   │   ├── dscreen_source_proxy.h      # Proxy类
│   │   └── callback/
│   │       ├── dscreen_source_callback_stub.h
│   │       └── dscreen_source_load_callback.h
│   ├── src/
│   │   ├── dscreen_source_handler.cpp
│   │   ├── dscreen_source_proxy.cpp
│   │   └── callback/
│   └── BUILD.gn               # 构建: distributed_screen_source_sdk
│
└── screen_sink/               # 被控端(Sink)SDK
    ├── include/
    │   ├── idscreen_sink.h             # 接口定义
    │   ├── dscreen_sink_handler.h      # Handler类
    │   └── dscreen_sink_proxy.h        # Proxy类
    ├── src/
    │   ├── dscreen_sink_handler.cpp
    │   └── dscreen_sink_proxy.cpp
    └── BUILD.gn               # 构建: distributed_screen_sink_sdk
```

**职责**: 对外提供Native C++接口，封装IPC通信细节。应用通过SDK调用分布式屏幕能力。

**关键类**:
- `IDScreenSource` - Source端接口定义 (`interfaces/innerkits/native_cpp/screen_source/include/idscreen_source.h:25`)
- `IDScreenSink` - Sink端接口定义 (`interfaces/innerkits/native_cpp/screen_sink/include/idscreen_sink.h:23`)

---

### sa_profile/ - SA配置文件

**路径**: `/foundation/distributedhardware/distributed_screen/sa_profile/`

```
sa_profile/
├── 4807.json                  # Source端SA配置 (SA ID: 4807)
├── 4808.json                  # Sink端SA配置 (SA ID: 4808)
├── dscreen.cfg                # 进程启动配置
└── BUILD.gn                   # 构建: dscreen_sa_profile, dscreen.cfg
```

**SA配置详情**:

**Source端 (4807.json)**:
```json
{
    "process": "dscreen",
    "systemability": [{
        "name": 4807,
        "libpath": "libdistributed_screen_source.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1
    }]
}
```

**Sink端 (4808.json)**:
```json
{
    "process": "dscreen",
    "systemability": [{
        "name": 4808,
        "libpath": "libdistributed_screen_sink.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1
    }]
}
```

**职责**: 定义System Ability服务配置，包括SA ID、库路径、启动策略等。

---

### screenhandler/ - 硬件处理器

**路径**: `/foundation/distributedhardware/distributed_screen/screenhandler/`

```
screenhandler/
├── include/
│   └── dscreen_handler.h      # 硬件处理主类
├── src/
│   └── dscreen_handler.cpp
└── BUILD.gn                   # 构建: distributed_screen_handler
```

**职责**: 作为分布式硬件框架的插件入口，处理屏幕硬件的使能/去使能请求。

**关键类**: `DScreenHandler` - 继承自分布式硬件框架的插件基类

---

### services/ - 核心服务实现

**路径**: `/foundation/distributedhardware/distributed_screen/services/`

```
services/
├── common/                    # 服务共用模块
│   ├── databuffer/            # 数据缓冲区
│   │   └── include/data_buffer.h
│   ├── screen_channel/        # 屏幕通道接口
│   │   └── include/iscreen_channel.h
│   ├── imageJpeg/             # JPEG图像处理
│   │   └── include/jpeg_image_processor.h
│   ├── decision_center/       # 决策中心
│   │   └── include/screen_decision_center.h
│   └── utils/                 # 工具类
│       └── include/
│           ├── dscreen_fwkkit.h
│           ├── video_param.h
│           └── dscreen_maprelation.h
│
├── screenservice/             # 屏幕主服务
│   ├── sourceservice/         # 主控端服务
│   │   ├── dscreenservice/    # Source SA实现
│   │   │   ├── include/
│   │   │   │   ├── dscreen_source_service.h    # SA服务类
│   │   │   │   ├── dscreen_source_stub.h       # IPC Stub
│   │   │   │   └── callback/
│   │   │   └── src/
│   │   │       ├── dscreen_source_service.cpp
│   │   │       ├── dscreen_source_stub.cpp
│   │   │       └── callback/
│   │   └── dscreenmgr/        # 屏幕管理器
│   │       ├── 1.0/           # v1.0版本
│   │       │   └── include/dscreen_manager.h
│   │       ├── 2.0/           # v2.0版本 (AVTrans)
│   │       │   └── include/dscreen_manager.h
│   │       └── common/        # 共用代码
│   │
│   └── sinkservice/           # 被控端服务
│       ├── dscreenservice/    # Sink SA实现
│       │   ├── include/
│       │   │   ├── dscreen_sink_service.h
│       │   │   └── dscreen_sink_stub.h
│       │   └── src/
│       └── screenregionmgr/   # 屏幕区域管理
│           ├── 1.0/           # v1.0版本
│           │   └── include/screenregionmgr.h
│           └── 2.0/           # v2.0版本 (AVTrans)
│               └── include/screenregionmgr.h
│
├── screentransport/           # 屏幕传输组件
│   ├── screensourcetrans/     # Source传输
│   │   ├── include/screen_source_trans.h
│   │   └── BUILD.gn           # 构建: distributed_screen_sourcetrans
│   ├── screensinktrans/       # Sink传输
│   │   ├── include/screen_sink_trans.h
│   │   └── BUILD.gn           # 构建: distributed_screen_sinktrans
│   ├── screensourceprocessor/ # Source数据处理(编码)
│   │   ├── include/image_source_processor.h
│   │   └── encoder/
│   │       └── include/image_source_encoder.h
│   ├── screensinkprocessor/   # Sink数据处理(解码)
│   │   └── decoder/
│   │       └── include/image_sink_decoder.h
│   └── screendatachannel/     # 数据传输通道
│       └── include/screen_data_channel_impl.h
│
├── screenclient/              # 屏幕客户端
│   ├── include/
│   │   ├── screen_client.h
│   │   └── screen_client_common.h
│   └── BUILD.gn               # 构建: distributed_screen_client
│
└── softbusadapter/            # 软总线适配器
    ├── include/
    │   ├── softbus_adapter.h
    │   ├── isoftbus_listener.h
    │   └── softbus_permission_check.h
    └── src/
        ├── softbus_adapter.cpp
        └── softbus_permission_check.cpp
```

#### services/common/ - 服务共用模块

**职责**: 提供服务层共用的数据缓冲区、通道接口、图像处理、决策逻辑等。

| 子模块 | 关键类 | 职责 |
|--------|--------|------|
| databuffer | `DataBuffer` | 屏幕数据缓冲区管理 |
| screen_channel | `IScreenChannel` | 屏幕数据传输通道接口 |
| imageJpeg | `JpegImageProcessor` | JPEG图像编码处理 |
| decision_center | `ScreenDecisionCenter` | 传输决策逻辑 |
| utils | `VideoParam`, `DScreenMapRelation` | 视频参数和映射关系 |

#### services/screenservice/ - 屏幕主服务

**职责**: System Ability服务实现，是分布式屏幕的核心业务逻辑层。

**Source端关键类** (`services/screenservice/sourceservice/`):
| 类 | 文件 | 职责 |
|----|------|------|
| `DScreenSourceService` | `dscreenservice/include/dscreen_source_service.h:29` | SA服务实现 |
| `DScreenSourceStub` | `dscreenservice/include/dscreen_source_stub.h` | IPC Stub |
| `DScreenManager` | `dscreenmgr/2.0/include/dscreen_manager.h` | 屏幕管理器 |
| `DScreen` | `dscreenmgr/2.0/include/dscreen.h` | 单个屏幕实例 |

**Sink端关键类** (`services/screenservice/sinkservice/`):
| 类 | 文件 | 职责 |
|----|------|------|
| `DScreenSinkService` | `dscreenservice/include/dscreen_sink_service.h:28` | SA服务实现 |
| `DScreenSinkStub` | `dscreenservice/include/dscreen_sink_stub.h` | IPC Stub |
| `ScreenRegionManager` | `screenregionmgr/2.0/include/screenregionmgr.h` | 区域管理器 |
| `ScreenRegion` | `screenregionmgr/2.0/include/screenregion.h` | 单个区域实例 |

#### services/screentransport/ - 屏幕传输组件

**职责**: 负责屏幕数据的编解码和网络传输。

| 子模块 | 关键类 | 职责 |
|--------|--------|------|
| screensourcetrans | `ScreenSourceTrans` | Source端传输管理 |
| screensinktrans | `ScreenSinkTrans` | Sink端传输管理 |
| screensourceprocessor | `ImageSourceProcessor`, `ImageSourceEncoder` | 视频编码 (H264/H265/MPEG4) |
| screensinkprocessor | `ImageSinkProcessor`, `ImageSinkDecoder` | 视频解码 |
| screendatachannel | `ScreenDataChannelImpl` | 数据传输通道实现 |

#### services/screenclient/ - 屏幕客户端

**职责**: 在被控端提供屏幕图像显示功能，管理代理显示窗口。

**关键类**: `ScreenClient` - 窗口管理和Surface Buffer管理

#### services/softbusadapter/ - 软总线适配器

**职责**: 封装软总线传输接口，为屏幕图像、输入事件提供统一传输接口。

**关键类**:
- `SoftbusAdapter` - 软总线适配器主类
- `SoftBusPermissionCheck` - 权限检查 (同账号验证)

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                         模块依赖图                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────┐     ┌─────────────────┐                  │
│   │   Source SDK    │     │   Sink SDK      │                  │
│   └────────┬────────┘     └────────┬────────┘                  │
│            │ IPC                  │ IPC                        │
│            ▼                      ▼                             │
│   ┌─────────────────────────────────────────────────┐          │
│   │         Screen Service (sourceservice/sinkservice)│          │
│   └────────┬──────────────────────┬────────────────┘          │
│            │                      │                            │
│            ▼                      ▼                             │
│   ┌─────────────────┐     ┌─────────────────┐                  │
│   │  Source Trans   │     │   Sink Trans    │                  │
│   └────────┬────────┘     └────────┬────────┘                  │
│            │                      │                            │
│            └──────────┬───────────┘                            │
│                       ▼                                         │
│            ┌─────────────────┐                                 │
│            │ SoftBus Adapter │                                 │
│            └────────┬────────┘                                 │
│                     │ 依赖                                      │
│                     ▼                                           │
│            ┌─────────────────┐                                 │
│            │   Common Utils  │                                 │
│            └─────────────────┘                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**依赖方向说明**:
1. **SDK → Service**: 通过IPC调用
2. **Service → Transport**: 直接依赖，调用传输接口
3. **Transport → SoftBusAdapter**: 直接依赖，发送/接收数据
4. **所有模块 → Common**: 依赖公共工具库

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目基本信息
- [架构设计](01_Architecture.md) - 详细架构说明
- [构建系统](05_Build_System.md) - 构建配置详解
- [附录/关键调用链](appendix/Callgraphs.md) - 调用关系图