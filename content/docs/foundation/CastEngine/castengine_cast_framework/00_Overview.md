# CastEngine 框架概览

> 文档版本: 1.0.0
> 最后更新: 2026-02-06

## 目的

本文档提供 CastEngine 框架的整体概览，包括项目定位、核心能力、运行环境和关键概念，帮助新人快速理解项目背景和范围。

## 适用范围

- OpenHarmony 投屏引擎框架（castengine_cast_framework）
- 版本: 3.1
- 子系统: castplus
- 组件: cast_engine

## 项目定位

CastEngine 是 OpenHarmony 系统的投屏引擎框架，提供音频和视频广播能力。它为南北向开发者提供统一的接口和标准化的体验。

### 核心定位

**南向开发者（系统服务提供者）**:
- 提供 C++ 内部 API 用于框架扩展
- 支持 DLNA、WiFi Display、Cast+Stream 三种协议适配
- 提供设备发现、连接管理、会话管理能力

**北向开发者（应用开发者）**:
- 提供 JavaScript/N-API 接口用于应用开发
- 简化的投屏流程控制 API
- 设备发现、会话创建、媒体播放能力

### 与外部仓库的关系

CastEngine 框架本身不实现具体的协议细节，而是依赖以下外部适配器：

| 仓库 | URL | 协议 |
|-------|------|-------|
| castengine_cast_plus_stream | [Gitee](https://gitee.com/openharmony-sig/castengine_cast_plus_stream) | Cast+Stream（自适应） |
| castengine_wifi_display | [Gitee](https://gitee.com/openharmony-sig/castengine_wifi_display) | WiFi Display |
| castengine_dlna | [Gitee](https://gitee.com/openharmony-sig/castengine_dlna) | DLNA |

**证据**: 见 `README.md` 文件 35-41 行，引用了这三个仓库。

## 核心能力

### 1. 设备发现

支持发现附近的投屏设备，支持多种设备类型：
- 华为系列设备（HW TV, HiCar, MateBook, Pad 等）
- 通用设备（屏幕播放器、音箱、Miracast 等）

**设备类型定义**: `interfaces/inner_api/include/cast_engine_common.h:33-48`

### 2. 投屏会话管理

创建和管理投屏会话，支持：
- 多设备会话
- 会话状态跟踪
- 设备添加/移除
- 属性配置

**接口定义**: `interfaces/inner_api/include/i_cast_session.h`

### 3. 镜像投屏（Mirror）

实时同步本机屏幕到远程设备：
- 屏幕捕获和编码
- 虚拟屏幕尺寸调整
- 投屏路由配置
- 屏幕截图功能

**N-API 入口**: `interfaces/kits/js/src/napi_mirror_player.cpp`

### 4. 流媒体播放（Stream）

将本地或远程媒体文件在远程设备播放：
- 支持播放控制（播放、暂停、停止、快进、快退）
- 支持音量、速度、循环模式控制
- 支持播放列表（下一首、上一首）
- 支持播放状态查询

**N-API 入口**: `interfaces/kits/js/src/napi_stream_player.cpp`

### 5. 远程控制

支持远程设备反向控制本机：
- 播放控制
- 音量调节
- 播放列表导航

**事件定义**: `interfaces/inner_api/include/oh_remote_control_event.h`

## 运行环境

### 系统要求

| 项目 | 要求 |
|-----|------|
| 操作系统 | OpenHarmony（Standard 系统） |
| 系统类型 | standard（从 `bundle.json:20`） |
| ROM | 5M（估算） |
| RAM | 50M（估算） |

**证据**: `bundle.json:22-23` - `"ram":"50M"`, `"rom":"5M"`

### 依赖组件

**框架依赖**（从 `bundle.json:27-70`）:

| 依赖 | 用途 |
|-----|------|
| hilog | 日志 |
| hisysevent | 系统事件 |
| hitrace | 性能跟踪 |
| media_foundation | 媒体基础框架 |
| access_token | 权限管理 |
| audio_framework | 音频框架 |
| av_codec | 编解码器 |
| ipc | 进程间通信 |
| init | 初始化 |
| input | 输入设备 |
| safwk, samgr | System Ability 框架 |
| c_utils | C 工具库 |
| eventhandler | 事件处理 |
| power_manager | 电源管理 |
| dsoftbus | 分布式软总线 |
| device_manager | 设备管理 |
| common_event_service | 公共事件 |
| bundle_framework | Bundle 框架 |
| ability_base, ability_runtime | Ability 框架 |
| ace_engine | ACE 引擎（UI） |
| napi | N-API 支持 |
| graphic_2d, graphic_surface | 图形和 Surface |
| window_manager | 窗口管理 |
| player_framework | 播放器框架 |
| image_framework | 图像框架 |
| wifi | WiFi 管理 |
| device_auth | 设备认证 |
| device_info_manager | 设备信息 |
| thermal_manager | 热管理 |
| screenlock_mgr | 锁屏管理 |
| state_registry | 状态注册 |
| core_service | 核心服务 |
| call_manager | 电话管理 |
| os_account | 账号管理 |
| sharing_framework | 分享框架 |
| jsoncpp | JSON 解析 |
| openssl | 加密 |

### 第三方依赖

从 `bundle.json:71-74`:
- `bounds_checking_function` - 边界检查函数
- `musl` - 标准 C 库

## 关键概念

### 1. 投屏会话（Cast Session）

一次完整的投屏交互过程，包含：
- 设备发现和连接
- 协议协商
- 数据传输（镜像或流）
- 会话状态管理
- 会话释放和清理

**实现**: `service/src/session/src/cast_session_impl.cpp`

### 2. System Ability (SA)

CastEngine 服务作为 System Ability 运行，SA ID 为 5526。

**配置**: `sa_profile/5526.json`
- 进程名: `cast_engine_service`
- 库: `libcast_engine_service.z.so`
- run-on-create: `false`（延迟启动）
- distributed: `false`（非分布式）

### 3. IPC 通信

客户端和服务端通过 IPC 机制通信：
- 客户端使用 Proxy
- 服务端使用 Stub
- 基于 OpenHarmony IPC 框架

**证据**: `service/src/cast_session_manager_service_stub.cpp`, `client/src/cast_session_manager_service_proxy.cpp`

### 4. 两种投屏模式

#### 镜像模式（Mirror）

- 本机屏幕实时编码
- 通过网络传输编码后的音视频流
- 远程设备解码并显示

#### 流模式（Stream）

- 媒体文件解析（本地或网络）
- 通过网络传输媒体数据
- 远程设备播放控制

**切换**: `CastSession::setCastMode(mode)` 方法支持模式切换

### 5. N-API（Native API）

OpenHarmony 的原生模块接口，允许 JavaScript 应用调用 C++ 实现。

**模块名**: `cast`
**注册入口**: `interfaces/kits/js/src/native_module.cpp:46-53`

### 6. GN 构建系统

OpenHarmony 使用的构建系统，基于 Ninja 生成构建规则。

**配置文件**:
- 根配置: `BUILD.gn`
- 变量定义: `cast_engine.gni`
- 组件定义: `bundle.json`

## 主要用例场景

### 场景 1: 手机投屏到电视

1. 应用调用 `CastSessionManager.startDiscovery()` 发现电视
2. 用户选择电视后调用 `CastSession.addDevice()`
3. 创建 `MirrorPlayer` 开始镜像投屏
4. 电视接收并显示手机屏幕

### 场景 2: 手机播放视频到音箱

1. 应用调用 `CastSessionManager.startDiscovery()` 发现音箱
2. 用户选择后创建 `CastSession`
3. 创建 `StreamPlayer` 并加载视频文件
4. 用户通过 `StreamPlayer.play()` 控制播放
5. 音箱播放视频并支持远程控制

### 场景 3: 多设备协同

1. 一个会话可添加多个设备
2. 支持镜像模式切换到流模式
3. 支持设备状态同步
4. 支持会话属性动态配置

## 系统架构层次

```
┌─────────────────────────────────────────────────────────────┐
│              应用层（JavaScript 应用）                 │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│         CastEngine N-API 层 (libcast.z.so)      │
│  • CastSessionManager                              │
│  • CastSession                                     │
│  • StreamPlayer                                     │
│  • MirrorPlayer                                     │
└──────────────────────┬─────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│   CastEngine 内部 C++ API 层                    │
│  (libcast_engine_client.z.so)                    │
└──────────────────────┬─────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│      CastEngine 服务层 (SA)                      │
│   (libcast_engine_service.z.so)                    │
│  • SessionManager                                   │
│  • DeviceManager                                  │
│  • 协议适配器（外部仓库）                       │
└──────────────────────┬─────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│     OpenHarmony 系统服务                          │
│  • SoftBus（分布式通信）                           │
│  • IPC 框架                                      │
│  • 媒体框架                                      │
│  • 图形/Surface                                  │
└─────────────────────────────────────────────────────┘
```

## 数据流向

### 镜像投屏数据流

```
本机设备                                     远程设备
┌──────────┐                              ┌──────────┐
│ 屏幕   │  [1] 屏幕捕获              │ 接收器   │
└─────┬────┘         ↓                      └─────┬────┘
      │         ┌────────────┐                   │
      │         │ 编码器     │                   │
      │         └─────┬──────┘                   │
      │               │ [2] 网络传输              │ [5] 解码并显示
      │               ↓                            │
      │         ┌─────────────┐                   │
      │         │ 远程设备   │                   │
      │         └─────────────┘                   │
      └─────────────────────────────────────────────────┘
```

### 流媒体播放数据流

```
本机设备                                     远程设备
┌──────────┐                              ┌──────────┐
│ 文件   │  [1] 解析              │ 接收器   │
└─────┬────┘         ↓                      └─────┬────┘
      │         ┌────────────┐                   │
      │         │ 播放器    │                   │
      │         └─────┬──────┘                   │
      │               │ [2] 网络传输              │ [3] 播放控制
      │               ↓                            │
      │         ┌─────────────┐                   │
      │         │ 远程设备   │                   │
      │         └─────────────┘                   │
      │                                     │
      └─────────────────────────────────────────────┘
```

## 关键文件位置

### 配置和元数据

| 文件 | 位置 | 说明 |
|-----|------|------|
| bundle.json | 根目录 | 组件定义和依赖 |
| BUILD.gn | 根目录 | 根构建配置 |
| cast_engine.gni | 根目录 | GN 变量定义 |
| sa_profile/5526.json | sa_profile/ | SA 配置 |
| hisysevent.yaml | 根目录 | HiSysEvent 配置 |

### N-API 实现

| 类 | 文件 | 说明 |
|----|------|------|
| CastSessionManager | interfaces/kits/js/src/napi_cast_session_manager.cpp:1-450 | 会话管理器 |
| CastSession | interfaces/kits/js/src/napi_cast_session.cpp:1-800+ | 投屏会话 |
| StreamPlayer | interfaces/kits/js/src/napi_stream_player.cpp:1-1700+ | 流播放器 |
| MirrorPlayer | interfaces/kits/js/src/napi_mirror_player.cpp:1-600+ | 镜像播放器 |

### 内部接口

| 接口 | 文件 | 说明 |
|-----|------|------|
| ICastSession | interfaces/inner_api/include/i_cast_session.h | 会话接口 |
| IStreamPlayer | interfaces/inner_api/include/i_stream_player.h | 流播放器接口 |
| IMirrorPlayer | interfaces/inner_api/include/i_mirror_player.h | 镜像播放器接口 |
| ICastSessionManager | interfaces/inner_api/include/cast_session_manager.h | 会话管理器接口 |

### 服务实现

| 组件 | 文件 | 说明 |
|-----|------|------|
| CastSessionManagerService | service/src/cast_session_manager_service.cpp | SA 主服务 |
| CastSessionImpl | service/src/session/src/cast_session_impl.cpp | 会话实现 |
| DiscoveryManager | service/src/device_manager/src/discovery_manager.cpp | 设备发现管理器 |
| ConnectionManager | service/src/device_manager/src/connection_manager.cpp | 连接管理器 |

## 相关文档

- [目录结构](01_Directory_Structure.md) - 详细的代码组织说明
- [架构设计](02_Architecture.md) - 深入的架构分析
- [N-API 文档](03_N-API.md) - 完整的 JavaScript API 参考
- [内部 API](04_Internal_API.md) - C++ 接口文档

---

**返回**: [SUMMARY.md](SUMMARY.md)
