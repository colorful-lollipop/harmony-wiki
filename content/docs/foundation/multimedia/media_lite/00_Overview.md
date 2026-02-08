# 项目概览

## 目的

本文档介绍 media_lite 项目的整体定位、边界、核心能力、运行环境和关键概念，帮助新人快速理解项目。

## 适用范围

- 适用于 OpenHarmony Lite 系统（mini、small 类型）
- 涵盖播放器（Player）和录制器（Recorder）两大模块
- 包含 JSI 绑定、IPC 服务、权限鉴权等关键技术

## 关键结论

1. **项目定位**：media_lite 是 OpenHarmony 多媒体子系统的精简版，提供核心播放和录制能力
2. **两种运行模式**：支持 Binder 模式（通过 IPC 通信）和 Passthrough 模式（直接链接）
3. **唯一 JS 绑定**：仅提供 Player 模块的 JSI API，Recorder 只有 C++ 原生接口
4. **服务化架构**：使用 SAMGR 管理服务，Player 和 Recorder 服务独立运行
5. **权限控制**：通过 permission_lite 系统进行集中式权限检查

---

## 项目定位与边界

### 在多媒体子系统中的位置

media_lite 位于多媒体子系统的最上层，与以下组件协作：

- **camera_lite** - 提供视频输入源（录制时使用）
- **audio_lite** - 提供音频播放和捕获能力
- **media_utils_lite** - 提供公共工具和类型定义

### 核心能力

| 能力 | 说明 | 相关模块 |
|------|------|---------|
| **媒体播放** | 播放本地文件、URI 流，支持音频/视频 | Player |
| **媒体录制** | 录制音频、视频，支持多种编码器和格式 | Recorder |
| **JS API** | 提供音频播放器 JSI 接口（仅 Player） | AudioModule |
| **IPC 服务** | 提供服务端实现，支持多客户端访问 | PlayerServer, RecorderServer |
| **权限管理** | 集成权限检查，保护敏感操作 | permission_lite |

### 不在范围

- 视频编解码器实现（依赖外部 hardware_media_sdk）
- 音频编解码器实现（依赖外部 audio_lite）
- 视频渲染（依赖 surface_lite）
- 测试框架和工具

---

## 核心能力

### Player 能力

- **播放源类型**：
  - 本地文件 URI
  - 文件描述符（FD）
  - 媒体流（部分支持）
- **播放控制**：
  - Play（播放）、Pause（暂停）、Stop（停止）
  - Rewind（跳转）
  - SetVolume（设置音量）
  - SetPlaybackSpeed（设置播放速度）
- **播放状态**：
  - IsPlaying（是否播放中）
  - GetPlayerState（获取状态）
  - GetCurrentTime（当前时间）
  - GetDuration（总时长）
- **循环播放**：
  - EnableSingleLooping（启用单循环）
  - IsSingleLooping（是否循环）
- **视频支持**：
  - SetVideoSurface（设置视频渲染 Surface）
  - GetVideoWidth/Height（获取视频尺寸）

### Recorder 能力

- **视频源类型**：
  - Surface YUV/RGB（通过 Surface 输入视频数据）
  - Surface ES（通过 Surface 输入编码数据）
- **音频源类型**：
  - 麦克风捕获
- **编码器支持**：
  - 视频编码器：H.264/H.265
  - 音频编码器：AAC/AMR-NB
- **录制控制**：
  - Prepare（准备）、Start（开始）、Pause（暂停）、Resume（恢复）、Stop（停止）
  - Reset（重置）、Release（释放）
- **录制参数**：
  - SetVideoSize（视频尺寸）
  - SetVideoFrameRate（视频帧率）
  - SetVideoEncodingBitRate（视频比特率）
  - SetCaptureRate（捕获速率）
  - SetAudioSampleRate（音频采样率）
  - SetAudioChannels（音频通道数）
  - SetAudioEncodingBitRate（音频比特率）
- **输出控制**：
  - SetOutputFormat（输出格式：MPEG4/TS）
  - SetOutputPath/SetOutputFile（输出路径或 FD）
  - SetMaxDuration/SetMaxFileSize（最大时长/文件大小）
- **文件分割**：
  - SetFileSplitDuration（手动分割文件）

---

## 运行环境

### 系统类型

| 系统类型 | 说明 | 支持情况 |
|---------|------|---------|
| **LiteOS-M** | 极小型系统内核 | Player 仅支持静态链接（histreamer） |
| **LiteOS-A** | 小型系统内核 | 支持全部功能 |

### 编译器支持

- GCC
- Clang

### 构建系统

- GN (Generate Ninja)
- HB (Harmony Build)

### 关键依赖

- **系统依赖**：
  - samgr_lite（服务管理）
  - ipc_lite（进程间通信）
  - hilog_lite（日志）
  - surface_lite（图形 Surface）
  - permission_lite（权限管理）
- **媒体依赖**：
  - audio_lite（音频播放和捕获）
  - camera_lite（相机输入）
  - media_utils_lite（公共工具）
- **第三方依赖**：
  - bounds_checking_function（边界检查）

---

## 关键概念

### 两种运行模式

**1. Binder 模式** (`enable_media_passthrough_mode = false`)
- 客户端和服务端分离
- 通过 IPC（SAMGR）通信
- 适合多进程架构

**2. Passthrough 模式** (`enable_media_passthrough_mode = true`)
- 客户端直接链接实现库
- 减少 IPC 开销
- 适合单进程、性能敏感场景

### SAMGR 服务管理

- 服务注册：`SAMGR_GetInstance()->RegisterService()`
- 服务发现：`SAMGR_GetInstance()->GetDefaultFeatureApi()`
- 服务名称："PlayerServer"、"RecorderServer"

### JSI (JavaScript Interface)

- OpenHarmony 自定义 JS 绑定 API
- 非 Node.js N-API
- 核心方法：`JSI::SetModuleAPI()`, `JSI::DefineNamedProperty()`

### 权限模型

- 集中式权限管理（permission_lite）
- 权限字符串：`ohos.permission.MODIFY_AUDIO_SETTINGS`, `ohos.permission.READ_MEDIA`, `ohos.permission.MICROPHONE`, `ohos.permission.WRITE_MEDIA`
- 检查函数：`CheckSelfPermission()`

---

## 相关跳转

- [目录结构与模块职责](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [对外 JSI 接口](03_JSI_Interfaces.md)
- [内部 API](04_Inner_API.md)
- [GN Targets](05_GN_Targets.md)
- [编译产物](06_Build_Artifacts.md)
- [安全风险评审](07_Security_Audit.md)
- [常见问题](08_FAQ.md)
