# 目录结构与模块职责

## 目的

本文档介绍 media_lite 项目的目录结构、各模块职责、依赖关系和关键文件。

## 适用范围

- 不包含测试目录
- 仅列出业务相关代码和配置
- 适用于了解项目组织的新人和开发者

---

## 完整目录树（不含 test 目录）

```
/foundation/multimedia/media_lite/
├── LICENSE
├── OAT.xml
├── README.md
├── README_zh.md
├── bundle.json                 # 组件定义
├── config.gni                 # 全局配置
├── figures/                   # 架构图
├── frameworks/                 # 框架层 - 客户端代理实现
│   ├── recorder_lite/
│   │   ├── BUILD.gn
│   │   ├── binder/                 # IPC 模式实现
│   │   │   ├── recorder_client.h
│   │   │   └── recorder_client.cpp
│   │   ├── passthrough/           # 透传模式实现
│   │   │   ├── recorder_client.h
│   │   │   └── recorder_client.cpp
│   │   └── recorder.cpp           # 框架入口
│   └── player_lite/
│       ├── BUILD.gn
│       ├── binder/                 # IPC 模式实现
│       │   ├── player_type.h
│       │   ├── player.cpp
│       │   ├── player_client.h
│       │   └── player_client.cpp
│       └── passthrough/           # 透传模式实现
│           ├── liteplayer/
│           │   ├── player.cpp
│           │   ├── player_client.h
│           │   └── player_client.cpp
│           └── histreamer/
│               └── player.cpp
├── interfaces/                  # 接口层
│   ├── kits/                    # 对外接口（JSI API）
│   │   ├── player_lite/
│   │   │   ├── player.h              # C++ 原生接口
│   │   │   └── js/
│   │   │       └── builtin/
│   │   │           ├── BUILD.gn
│   │   │           ├── include/
│   │   │           │   ├── audio_module.h
│   │   │           │   └── audio_player.h
│   │   │           └── src/
│   │   │               ├── audio_module.cpp    # JSI 模块注册
│   │   │               └── audio_player.cpp    # AudioPlayer 实现
│   └── innerkits/               # 内部接口
└── services/                   # 服务端实现
    ├── media_main.cpp              # 媒体服务主入口
    ├── recorder_lite/
    │   ├── BUILD.gn
    │   ├── server/
    │   │   ├── include/
    │   │   │   ├── recorder_common.h    # IPC 枚举定义
    │   │   │   └── recorder_service.h    # 服务端接口
    │   │   └── src/
    │   │       ├── recorder_samgr.cpp  # SAMGR 服务注册
    │   │       └── recorder_service.cpp  # IPC 请求处理
    │   └── impl/                   # 录音器实现
    │       ├── include/             # 实现头文件
    │       └── src/                # 实现源文件
    └── player_lite/
        ├── BUILD.gn
        ├── server/                  # 服务端
        │   ├── include/
        │   │   └── player_server.h     # 服务端接口
        │   └── src/
        │       ├── samgr_player_server.cpp  # SAMGR 服务注册
        │       └── player_server.cpp      # IPC 请求处理
        ├── impl/                    # 播放器实现
        │   ├── include/             # 实现头文件
        │   └── src/                # 实现源文件
        └── factory/                # 播放器工厂
            ├── include/
            └── src/
```

---

## 模块职责

### frameworks/ 模块

#### frameworks/recorder_lite/

**职责**：
- 提供录制器的客户端框架封装
- 支持两种运行模式（Binder/Passthrough）
- 封装 IPC 调用或直接链接实现
- 提供统一的 Recorder C++ 接口

**关键文件**：
- `recorder.cpp` - 录制器框架入口，实现 `OHOS::Media::Recorder` 接口
- `binder/recorder_client.cpp` - IPC 模式客户端实现
- `passthrough/recorder_client.cpp` - 透传模式客户端实现

**依赖**：
- `permission_lite` - 权限检查
- `surface_lite` - 视频 Surface
- `media_utils_lite` - 公共工具
- `samgr` / `ipc_lite` - IPC 通信（Binder 模式）
- `recorder_impl` - 录音器实现（Passthrough 模式）

---

#### frameworks/player_lite/

**职责**：
- 提供播放器的客户端框架封装
- 支持两种运行模式（Binder/Passthrough）
- 封装 IPC 调用或直接链接实现
- 提供统一的 Player C++ 接口

**关键文件**：
- `binder/player.cpp` - 播放器框架入口，实现 `OHOS::Media::Player` 接口
- `binder/player_client.cpp` - IPC 模式客户端实现
- `passthrough/liteplayer/player.cpp` - Passthrough 模式客户端实现
- `passthrough/histreamer/player.cpp` - LiteOS-M 内核的 histreamer 播放器

**依赖**：
- `permission_lite` - 权限检查
- `surface_lite` - 视频 Surface
- `media_utils_lite` - 公共工具
- `samgr` / `ipc_lite` - IPC 通信（Binder 模式）
- `player_impl` - 播放器实现（Passthrough 模式）

---

### interfaces/ 模块

#### interfaces/kits/

**职责**：
- 定义对外 C++ 原生接口
- 提供 JSI 绑定层，暴露功能到 JS 层

**关键文件**：
- `player_lite/player.h` - Player 接口定义（行 37-424）
- `recorder_lite/recorder.h` - Recorder 接口定义（行 37-632）

**接口要点**：
- 使用枚举定义状态、模式等类型
- 使用 `std::shared_ptr` 管理回调对象
- 返回值使用 int32_t，0 表示成功，非 0 表示失败

---

#### interfaces/kits/player_lite/js/builtin/

**职责**：
- 实现 AudioModule JSI 绑定
- 暴露音频播放器功能到 JS 层
- 管理 AudioPlayer 单例和事件监听器

**关键文件**：
- `include/audio_module.h` - AudioModule 类定义
- `include/audio_player.h` - AudioPlayer 类定义
- `src/audio_module.cpp` - JSI 模块注册和方法导出（行 15-364）
- `src/audio_player.cpp` - AudioPlayer 实现（行 14-613）

**导出功能**：
- 方法：play, pause, stop, getPlayState
- 属性：src, currentTime, duration, autoplay, loop, volume, muted
- 事件：onplay, onpause, onstop, onloadeddata, onended, onerror, ontimeupdate

---

### services/ 模块

#### services/

**职责**：
- 媒体服务主入口
- 初始化各子服务
- 处理信号和终止

**关键文件**：
- `media_main.cpp` - 媒体服务主程序（行 15-68）

**初始化流程**：
1. `OHOS_SystemInit()` - 初始化 SAMGR
2. `CameraServer::GetInstance()->InitCameraServer()` - 初始化相机服务（如果启用）
3. `PlayerServer::GetInstance()->PlayerServerInit()` - 初始化播放器服务（非 Passthrough）
4. `AudioCapturerServer::GetInstance()->AudioCapturerServerInit()` - 初始化音频捕获服务
5. 信号等待循环 - 处理 SIGABRT, SIGINT, SIGTERM

---

#### services/recorder_lite/

**职责**：
- 录音器服务端实现
- 处理 IPC 请求和分发
- 管理录音器实例和回调

**关键文件**：
- `server/include/recorder_service.h` - RecorderService 类定义
- `server/src/recorder_samgr.cpp` - SAMGR 服务注册（RECORDER_SERVICE_NAME = "RecorderServer"）
- `server/src/recorder_service.cpp` - IPC 请求分发处理
- `impl/` - 录音器实现（包含 video_source, audio_source, recorder_sink, data_source, recorder_impl）

**服务入口点**：
```cpp
// recorder_samgr.cpp:65-70
static RecorderService recorderSvc = {
    .GetName = GetName,
    .Initialize = Initialize,
    .MessageHandle = MessageHandle,
    .GetTaskConfig = GetTaskConfig,
    SERVER_IPROXY_IMPL_BEGIN,
    .Invoke = Invoke,  // IPC 调用入口
    IPROXY_END,
};
```

---

#### services/player_lite/

**职责**：
- 播放器服务端实现
- 处理 IPC 请求和分发
- 管理播放器实例和回调

**关键文件**：
- `server/include/player_server.h` - PlayerServer 类定义
- `server/src/samgr_player_server.cpp` - SAMGR 服务注册（SERVICE_NAME = "PlayerServer"）
- `server/src/player_server.cpp` - IPC 请求处理
- `impl/` - 播放器实现（包含 source, decoder, sink, player, buffersource）
- `factory/` - 播放器工厂

**服务入口点**：
```cpp
// samgr_player_server.cpp:23-25
static PlayerService g_example = {
    INHERIT_SERVICE,
    INHERIT_IUNKNOWNENTRY(DefaultFeatureApi),
    Identity identity,
};
```

---

## 模块依赖关系

### 依赖方向

```
应用层 (JS/C++)
    ↓
AudioModule (JSI)
    ↓
Player/Recorder C++ 接口
    ↓
    (Binder 模式)
Frameworks 层
    ↓
    IPC (SAMGR)
    ↓
Services 层
    ↓
    实现层 (recorder_impl/player_impl)
```

### 关键依赖

| 模块 | 依赖的外部组件 | 用途 |
|------|---------------|------|
| **frameworks/recorder_lite** | permission_lite, surface_lite, samgr/ipc_lite, recorder_impl | 权限、Surface、IPC、实现 |
| **frameworks/player_lite** | permission_lite, surface_lite, samgr/ipc_lite, player_impl, media_utils_lite | 权限、Surface、IPC、实现、工具 |
| **services/recorder_lite** | ipc_single, surface_lite, recorder_impl, audio_capturer_impl, media_common, samgr, pms_client | IPC、Surface、实现、音频、工具、服务管理、权限 |
| **services/player_lite** | surface_lite, player_impl, media_common, samgr | Surface、实现、工具、服务管理 |
| **interfaces/kits/player_lite/js/builtin** | player_lite, media_utils_lite | 播放器框架、工具 |

---

## 相关跳转

- [项目概览](00_Overview.md)
- [架构说明](02_Architecture.md)
- [对外 JSI 接口](03_JSI_Interfaces.md)
- [内部 API](04_Inner_API.md)
- [GN Targets](05_GN_Targets.md)
