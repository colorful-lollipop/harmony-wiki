# Audio Framework - 目录结构与代码地图

> 本文档提供 OpenHarmony 音频框架的目录结构说明和代码导航指南。

---

## 1. 顶层目录结构

```
foundation/multimedia/audio_framework/
├── frameworks/          # 框架层 - API 实现
├── interfaces/          # 接口层 - API 定义
├── services/            # 服务层 - 系统服务
├── sa_profile/          # SA 配置文件
├── plugins/             # 插件配置
├── test/                # 测试代码 (本文档不覆盖)
├── wiki/                # 本文档
├── config.gni           # Feature 开关配置
├── bundle.json          # 组件配置
└── README.md            # 项目说明
```

---

## 2. Frameworks 目录详解

### 2.1 JS/N-API 层

```
frameworks/js/napi/
├── common/              # N-API 公共代码
│   ├── napi_audio_entry.cpp      # N-API 入口点
│   ├── napi_param_utils.cpp      # 参数工具
│   ├── napi_audio_error.cpp      # 错误处理
│   └── napi_async_work.cpp       # 异步工作
├── audiorenderer/       # 音频渲染器 N-API
│   ├── napi_audio_renderer.cpp   # 主类实现
│   ├── napi_toneplayer.cpp       # Tone 播放器
│   └── callback/                 # 回调实现
├── audiocapturer/       # 音频采集器 N-API
│   ├── napi_audio_capturer.cpp   # 主类实现
│   └── callback/                 # 回调实现
└── audiomanager/        # 音频管理器 N-API
    ├── napi_audio_manager.cpp
    ├── napi_audio_volume_manager.cpp
    ├── napi_audio_routing_manager.cpp
    ├── napi_audio_stream_manager.cpp
    ├── napi_audio_effect_manager.cpp
    ├── napi_audio_interrupt_manager.cpp
    ├── napi_audio_spatialization_manager.cpp
    ├── napi_audio_session_manager.cpp
    ├── napi_audio_collaborative_manager.cpp
    └── callback/                   # 回调实现
```

**关键文件导航**:
| 功能 | 文件路径 |
|------|----------|
| N-API 注册入口 | `common/napi_audio_entry.cpp:36-58` |
| 音频渲染器 | `audiorenderer/napi_audio_renderer.cpp` |
| 音频采集器 | `audiocapturer/napi_audio_capturer.cpp` |
| 音量管理 | `audiomanager/napi_audio_volume_manager.cpp` |

### 2.2 Native 框架层

```
frameworks/native/
├── ohaudio/             # OHAudio C API
│   ├── OHAudioRenderer.cpp
│   ├── OHAudioCapturer.cpp
│   └── OHAudioManager.cpp
├── audiostream/         # 音频流实现
│   ├── audio_stream.cpp
│   └── fast_audio_stream.cpp
├── audiorenderer/       # 渲染器 Native 实现
│   └── audio_renderer.cpp
├── audiocapturer/       # 采集器 Native 实现
│   └── audio_capturer.cpp
├── audiomanager/        # 管理器 Native 实现
│   ├── audio_system_manager.cpp
│   ├── audio_stream_manager.cpp
│   └── audio_routing_manager.cpp
├── hdiadapter_new/      # HDI 驱动适配器
│   ├── sink/            # 音频输出适配
│   ├── source/          # 音频输入适配
│   ├── manager/         # 适配器管理
│   └── adapter/         # 设备管理
├── audioeffect/         # 音效处理
│   ├── audio_effect_chain.cpp
│   └── audio_effect_chain_manager.cpp
├── opensles/            # OpenSL ES 兼容层
└── toneplayer/          # Tone 播放器
```

**关键文件导航**:
| 功能 | 文件路径 |
|------|----------|
| C API 渲染器 | `ohaudio/OHAudioRenderer.cpp` |
| C API 采集器 | `ohaudio/OHAudioCapturer.cpp` |
| HDI 适配器管理 | `hdiadapter_new/manager/hdi_adapter_manager.cpp` |
| 音频效果链 | `audioeffect/audio_effect_chain_manager.cpp` |

### 2.3 Cangjie FFI 层

```
frameworks/cj/
├── include/             # Cangjie 头文件
└── src/                 # FFI 实现
    ├── multimedia_audio_ffi.cpp
    └── multimedia_audio_manager_impl.cpp
```

---

## 3. Services 目录详解

### 3.1 Audio Service

```
services/audio_service/
├── server/              # 服务端实现
│   ├── src/
│   │   ├── audio_server.cpp           # 服务入口
│   │   ├── audio_process_in_server.cpp # 音频处理
│   │   └── audio_server_proxy.cpp     # 代理实现
│   └── include/
│       └── audio_server.h
├── client/              # 客户端实现
│   ├── src/
│   │   └── audio_service_client.cpp
│   └── include/
│       └── pulseaudio_ipc_interface_code.h  # IPC 接口定义
├── common/              # 公共代码
│   └── va_shared_buffer.h  # 共享内存
└── idl/                 # IDL 接口定义
```

**关键文件导航**:
| 功能 | 文件路径 |
|------|----------|
| 服务入口 | `server/src/audio_server.cpp` |
| IPC 接口码 | `client/include/pulseaudio_ipc_interface_code.h` |
| 共享内存 | `common/include/va_shared_buffer.h` |

### 3.2 Audio Policy Service

```
services/audio_policy/
├── server/              # 策略服务端
│   ├── service/service_main/
│   │   ├── src/audio_policy_server.cpp
│   │   └── include/audio_policy_server.h
│   ├── domain/          # 领域模块
│   │   ├── device/      # 设备管理
│   │   ├── volume/      # 音量管理
│   │   ├── session/     # 会话管理
│   │   ├── router/      # 路由管理
│   │   └── interrupt/   # 中断管理
│   ├── infra/           # 基础设施
│   │   ├── config/      # 配置管理
│   │   │   └── parser/  # 配置解析器
│   │   └── ipc_proxy/   # IPC 代理
│   └── common/          # 公共代码
├── client/              # 策略客户端
└── common/              # 公共定义
    └── audio_policy_ipc_interface_code.h  # IPC 接口定义
```

**关键文件导航**:
| 功能 | 文件路径 |
|------|----------|
| 策略服务入口 | `server/service/service_main/src/audio_policy_server.cpp` |
| 设备管理 | `server/domain/device/` |
| 配置解析 | `server/infra/config/parser/` |
| IPC 接口码 | `common/audio_policy_ipc_interface_code.h` |

### 3.3 Audio Engine

```
services/audio_engine/
├── node/                # 音频处理节点
│   ├── src/
│   │   ├── hpae_mixer_node.cpp
│   │   ├── hpae_gain_node.cpp
│   │   └── hpae_render_effect_node.cpp
│   └── include/
├── buffer/              # 缓冲区管理
├── dfx/                 # DFX 工具
└── plugin/              # 插件
    └── resample/        # 重采样插件
```

### 3.4 Audio Suite

```
services/audio_suite/
├── client/              # 客户端
│   ├── node/            # 处理节点
│   └── manager/         # 管理器
└── server/              # 服务端
```

---

## 4. Interfaces 目录详解

### 4.1 Inner API (内部 API)

```
interfaces/inner_api/native/
├── audiorenderer/       # 渲染器接口
│   └── include/audio_renderer.h
├── audiocapturer/       # 采集器接口
│   └── include/audio_capturer.h
├── audiomanager/        # 管理器接口
│   └── include/
│       ├── audio_system_manager.h
│       ├── audio_stream_manager.h
│       └── audio_routing_manager.h
├── audiocommon/         # 公共定义
│   └── include/
│       ├── audio_info.h
│       └── audio_stream_types.h
├── audiosuite/          # Audio Suite 接口
└── toneplayer/          # TonePlayer 接口
```

### 4.2 Kits API (外部 C API)

```
interfaces/kits/c/
├── audio_manager/       # 管理器 C API
│   └── native_audio_manager.h
├── audio_renderer/      # 渲染器 C API
│   └── native_audiorenderer.h
├── audio_capturer/      # 采集器 C API
│   └── native_audiocapturer.h
└── common/              # 公共 C API
```

---

## 5. 代码导航图

### 5.1 音频播放流程

```
JS/ArkTS App
    ↓ N-API
frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp
    ↓
frameworks/native/audiorenderer/audio_renderer.cpp
    ↓ IPC
services/audio_policy/ (CreateRendererClient)
    ↓
services/audio_service/ (音频流管理)
    ↓ HDI
frameworks/native/hdiadapter_new/sink/
    ↓
硬件驱动
```

### 5.2 音频录制流程

```
JS/ArkTS App
    ↓ N-API
frameworks/js/napi/audiocapturer/napi_audio_capturer.cpp
    ↓
frameworks/native/audiocapturer/audio_capturer.cpp
    ↓ IPC
services/audio_policy/ (CreateCapturerClient, 权限检查点)
    ↓
services/audio_service/
    ↓ HDI
frameworks/native/hdiadapter_new/source/
    ↓
硬件驱动 (麦克风)
```

### 5.3 音量控制流程

```
JS/ArkTS App
    ↓ N-API
frameworks/js/napi/audiomanager/napi_audio_volume_manager.cpp
    ↓
frameworks/native/audiomanager/audio_system_manager.cpp
    ↓ IPC
services/audio_policy/server/domain/volume/
    ↓
持久化配置
```

---

## 6. 关键符号索引

### 6.1 核心类

| 类名 | 文件路径 | 说明 |
|------|----------|------|
| NapiAudioRenderer | `frameworks/js/napi/audiorenderer/napi_audio_renderer.h` | N-API 渲染器 |
| NapiAudioCapturer | `frameworks/js/napi/audiocapturer/napi_audio_capturer.h` | N-API 采集器 |
| AudioRenderer | `interfaces/inner_api/native/audiorenderer/include/audio_renderer.h` | Native 渲染器 |
| AudioCapturer | `interfaces/inner_api/native/audiocapturer/include/audio_capturer.h` | Native 采集器 |
| AudioSystemManager | `interfaces/inner_api/native/audiomanager/include/audio_system_manager.h` | 系统管理器 |
| AudioServer | `services/audio_service/server/include/audio_server.h` | 音频服务 |
| AudioPolicyServer | `services/audio_policy/server/service/service_main/include/audio_policy_server.h` | 策略服务 |

### 6.2 核心函数

| 函数名 | 文件路径 | 说明 |
|--------|----------|------|
| `Init` | `frameworks/js/napi/common/napi_audio_entry.cpp:36` | N-API 初始化 |
| `Create` | `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp:251` | 创建渲染器 |
| `Write` | `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp` | 写入音频数据 |
| `Read` | `frameworks/js/napi/audiocapturer/napi_audio_capturer.cpp` | 读取音频数据 |
| `OnRemoteRequest` | `services/audio_service/server/src/audio_server.cpp` | IPC 请求处理 |

---

## 7. 配置文件位置

| 配置类型 | 文件路径 | 说明 |
|---------|----------|------|
| Feature 开关 | `config.gni` | 编译时特性配置 |
| 组件配置 | `bundle.json` | 组件元数据 |
| SA 配置 | `sa_profile/audio_policy.json` | 策略服务配置 (SAID: 3009) |
| SA 配置 | `sa_profile/pulseaudio.json` | 音频服务配置 (SAID: 3001) |
| 音效配置 | `services/audio_policy/server/infra/config/file/audio_effect_config.xml` | 音效参数 |
| 音量配置 | `services/audio_policy/server/infra/config/file/audio_volume_config.xml` | 音量表 |
| 路由策略 | `services/audio_policy/server/infra/config/file/audio_strategy_router.xml` | 路由规则 |

---

## 8. 快速查找指南

### 按功能查找

| 功能需求 | 查找路径 |
|---------|----------|
| 添加 N-API 方法 | `frameworks/js/napi/*/napi_audio_*.cpp` |
| 修改 IPC 接口 | `services/*/common/include/*_ipc_interface_code.h` |
| 修改音频策略 | `services/audio_policy/server/domain/` |
| 添加音效 | `frameworks/native/audioeffect/` |
| 修改驱动适配 | `frameworks/native/hdiadapter_new/` |
| 修改配置文件 | `services/audio_policy/server/infra/config/file/` |

### 按问题查找

| 问题类型 | 查找路径 |
|---------|----------|
| N-API 崩溃 | `frameworks/js/napi/common/napi_audio_error.cpp` |
| IPC 失败 | `services/*/client/` |
| 策略异常 | `services/audio_policy/server/domain/` |
| 音效异常 | `frameworks/native/audioeffect/` |
| 设备问题 | `frameworks/native/hdiadapter_new/` |

---

*最后更新: 2026-02-07*
