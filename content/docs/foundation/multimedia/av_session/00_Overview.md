# AVSession 概览

## 项目定位

AVSession 是 OpenHarmony 多媒体子系统的核心组件，为系统提供**统一的媒体控制能力**。

### 核心价值

| 面向对象 | 价值描述 |
|----------|----------|
| **用户** | 提供便捷的全局播控入口，将媒体信息充分展示给用户。支持分布式设备信息展示，操作远端媒体如同操作本地媒体。 |
| **开发者** | 提供精简的 JS 接口，帮助开发者快速构建媒体应用，能够方便地接入系统播控中心，使用分布式播控能力。 |

### 系统能力 (SystemCapability)

```json
// bundle.json 中定义
"syscap": [
    "SystemCapability.Multimedia.AVSession.AVCast",
    "SystemCapability.Multimedia.AVSession.Core",
    "SystemCapability.Multimedia.AVSession.ExtendedDisplayCast",
    "SystemCapability.Multimedia.AVSession.Manager",
    "SystemCapability.Multimedia.AVSession.AVInputCast = false",
    "SystemCapability.Multimedia.AVSession.AVMusicTemplate = false"
]
```

---

## 核心能力

### 1. 会话管理
- 创建/销毁媒体会话 (AVSession)
- 会话状态追踪 (激活/停用)
- 会话生命周期管理

### 2. 媒体控制
- 播放/暂停/停止/上一首/下一首
- 快进/快退/跳转 (Seek)
- 播放速度调节
- 循环模式设置

### 3. 元数据管理
- 标题、艺术家、专辑信息
- 媒体封面图片
- 播放列表管理
- 歌词信息

### 4. 分布式控制 (AVCast)
- 本地设备音频投送到远端设备播放
- 远端会话同步和控制
- 跨设备播放状态同步

### 5. 系统集成
- 媒体按键事件处理
- 桌面歌词展示
- 播控中心集成

---

## 运行环境

### 依赖组件

```json
// bundle.json 中定义的依赖
"deps": {
    "components": [
        "ability_base", "ability_runtime",
        "audio_framework", "bundle_framework",
        "ipc", "napi", "safwk", "samgr",
        "window_manager", "background_task_mgr",
        "bluetooth", "dsoftbus",
        // ... 更多依赖
    ]
}
```

### 运行时资源

| 资源类型 | 大小 |
|----------|------|
| ROM | 3000KB |
| RAM | 5120KB |

### 系统类型

```gn
// config.gni
adapted_system_type: ["standard"]
```

---

## 关键概念

### 会话 (AVSession)
音视频应用向控制中心申请会话，会话与应用绑定。应用通过会话向系统传递信息，系统通过会话向应用传递控制命令。

**会话类型**:
- `SESSION_TYPE_AUDIO` (0): 音频会话
- `SESSION_TYPE_VIDEO` (1): 视频会话
- `SESSION_TYPE_VOICE_CALL` (2): 语音通话会话
- `SESSION_TYPE_VIDEO_CALL` (3): 视频通话会话
- `SESSION_TYPE_PHOTO` (4): 相册会话

### 控制器 (AVSessionController)
系统播控中心向会话控制中心申请控制器，播控中心通过控制器可以控制指定的会话，进而控制应用行为。

### 播放状态 (AVPlaybackState)

| 状态 | 枚举值 | 说明 |
|------|--------|------|
| INITIAL | 0 | 初始状态 |
| PREPARE | 1 | 准备中 |
| PLAY | 2 | 播放中 |
| PAUSE | 3 | 已暂停 |
| FAST_FORWARD | 4 | 快进中 |
| REWIND | 5 | 快退中 |
| STOP | 6 | 已停止 |
| COMPLETED | 7 | 已完成 |
| RELEASED | 8 | 已释放 |
| ERROR | 9 | 错误 |
| IDLE | 10 | 空闲 |
| BUFFERING | 11 | 缓冲中 |

### 控制命令 (AVControlCommand)

| 命令 | 枚举值 | 说明 |
|------|--------|------|
| PLAY | 0 | 播放 |
| PAUSE | 1 | 暂停 |
| STOP | 2 | 停止 |
| PLAY_NEXT | 3 | 下一首 |
| PLAY_PREVIOUS | 4 | 上一首 |
| FAST_FORWARD | 5 | 快进 |
| REWIND | 6 | 快退 |
| SEEK | 7 | 跳转 |
| SET_SPEED | 8 | 设置速度 |
| SET_LOOP_MODE | 9 | 设置循环模式 |
| TOGGLE_FAVORITE | 10 | 收藏 |

---

## 目录结构

```
av_session/
├── frameworks/                      # 框架层
│   ├── common/                       # 公共框架代码
│   ├── js/napi/session/              # N-API 实现 (47个文件)
│   │   ├── include/                  # N-API 头文件 (25个)
│   │   └── src/                      # N-API 源文件 (22个)
│   ├── native/
│   │   ├── ohavsession/             # OH AVSession C API
│   │   └── session/                 # Native C++ 客户端
│   ├── cj/                          # Cangjie 语言绑定
│   └── taihe/                       # Taihe 跨语言框架
├── interfaces/                       # 接口层
│   └── inner_api/native/session/
│       └── include/                 # Native API 头文件 (34个核心文件)
├── services/                        # 服务层
│   ├── etc/                         # 服务配置
│   └── session/
│       ├── adapter/                 # 外部依赖适配
│       ├── ipc/                     # IPC 通信层
│       │   ├── base/               # IPC 接口定义
│       │   ├── proxy/              # 客户端代理
│       │   └── stub/               # 服务端存根
│       └── server/                  # 服务端核心
│           ├── migrate/            # 会话迁移
│           ├── remote/             # 分布式会话
│           └── softbus/            # 软总线通信
├── utils/                          # 公共工具库
├── avpicker/                       # 投屏选择器 UI
├── avinputcastpicker/              # 输入投屏选择器
├── avvolumepanel/                  # 音量面板 UI
├── sa_profile/                     # SA 配置
├── bundle.json                     # 组件配置
├── config.gni                      # GN 配置
└── hisysevent.yaml                 # 性能监控配置
```

---

## 模块职责

| 模块 | 职责 |
|------|------|
| **frameworks/js/napi/** | 提供 JS/TS 语言的 N-API 接口封装 |
| **frameworks/native/** | 提供 Native C++ 客户端实现 |
| **frameworks/cj/** | 提供 Cangjie 语言 FFI 绑定 |
| **frameworks/taihe/** | 提供 Taihe 跨语言框架支持 |
| **interfaces/** | 定义 Native API 接口契约 |
| **services/session/server/** | 实现 AVSession SA 服务 |
| **services/session/ipc/** | 实现 IPC 通信的 Proxy/Stub |
| **services/session/adapter/** | 适配外部系统依赖 |
| **utils/** | 提供日志、权限检查等公共能力 |
