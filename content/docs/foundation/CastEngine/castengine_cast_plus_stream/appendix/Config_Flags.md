# Appendix: Config_Flags - 配置与宏定义

> 本文档汇总 Cast+ Stream 模块的关键宏定义、配置常量和 Feature Flags。

---

## 1. 版本与协议常量

### 1.1 Cast+ 版本

```cpp
// include/cast_session_impl_class.h:132
static constexpr double CAST_VERSION = 1.1;
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `CAST_VERSION` | 1.1 | Cast+ 协议版本 |

### 1.2 RTSP 版本

```cpp
// src/rtsp/src/rtsp_basetype.h
static const std::string RTSP_VERSION = "RTSP/1.0";
```

---

## 2. 超时与定时器

### 2.1 会话超时

```cpp
// include/cast_session_impl_class.h:133
static constexpr int TIMEOUT_CONNECT = 60 * 1000;  // 60 秒
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `TIMEOUT_CONNECT` | 60000 ms | 连接超时时间 |

### 2.2 RTSP 超时

```cpp
// src/rtsp/src/rtsp_basetype.h
static const int RTSP_TIMEOUT_MS = 5000;  // 5 秒
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `RTSP_TIMEOUT_MS` | 5000 ms | RTSP 响应超时 |

---

## 3. 缓冲区与大小限制

### 3.1 数据包大小

```cpp
// src/channel/src/tcp/tcp_connection.cpp
static const int MAX_PACKET_SIZE = 10 * 1024 * 1024;  // 10 MB
static const int ILLEGAL_LENGTH = 10 * 1024 * 1024;   // 10 MB
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `MAX_PACKET_SIZE` | 10 MB | 最大数据包大小 |
| `SOCKET_SEND_BUFFER_SIZE` | 512 KB | TCP 发送缓冲区 |
| `SOCKET_RECV_BUFFER_SIZE` | 10 MB | TCP 接收缓冲区 |

### 3.2 缓存大小

```cpp
// src/stream/src/local/include/local_data_source.h
static const int MAX_CACHE_NUM = 4;           // 最大缓存数
static const int CACHE_SIZE = 5 * 1024 * 1024; // 每个缓存 5 MB
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `MAX_CACHE_NUM` | 4 | 最大缓存块数 |
| `CACHE_SIZE` | 5 MB | 每块缓存大小 |
| **总缓存** | 20 MB | 最大总缓存 |

### 3.3 字符串长度

```cpp
// 多处定义
static const int MAX_DEVICE_ID_LEN = 64;
static const int MAX_SESSION_ID_LEN = 32;
```

---

## 4. 视频参数限制

### 4.1 视频属性范围

```cpp
// src/rtsp/include/rtsp_basetype.h
static const int VIDEO_FPS_MIN = 20;
static const int VIDEO_FPS_60 = 60;
static const int VIDEO_BITRATE_MIN = 500000;
static const int VIDEO_BITRATE_MAX = 20000000;
static const int VIDEO_GOP_MIN = 30;
static const int VIDEO_GOP_MAX = 600;
```

| 参数 | 最小值 | 最大值 | 说明 |
|------|--------|--------|------|
| FPS | 20 | - | 帧率 |
| Bitrate | 500 Kbps | 20 Mbps | 视频码率 |
| GOP | 30 | 600 | 关键帧间隔 |

### 4.2 分辨率限制

```cpp
// 实际限制由编解码器决定
// 常见分辨率支持:
// - 1920x1080 (1080p)
// - 1280x720 (720p)
// - 640x480 (480p)
```

---

## 5. 加密常量

### 5.1 AES 参数

```cpp
// src/utils/include/encrypt_decrypt.h
static const int AES_KEY_LEN_128 = 16;
static const unsigned int AES_IV_LEN = 16;
static const int AES_KEY_LEN = 16;
static const int AES_KEY_SIZE = 16;
static const int PC_ENCRYPT_LEN = 64;
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `AES_KEY_LEN_128` | 16 字节 | AES-128 密钥长度 |
| `AES_IV_LEN` | 16 字节 | IV 长度 |

### 5.2 加密算法代码

```cpp
// src/utils/include/encrypt_decrypt.h
static const int INVALID_CODE = -1;
static const int DEFAULT_CODE = 0;
static const int CTR_CODE = 1;
static const int GCM_CODE = 2;

static const std::string PC_ENCRYPT_ALG = "aes128ctr";
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `INVALID_CODE` | -1 | 无效算法 |
| `DEFAULT_CODE` | 0 | 默认算法 |
| `CTR_CODE` | 1 | AES-128-CTR |
| `GCM_CODE` | 2 | AES-128-GCM |

### 5.3 GCM 参数

```cpp
// src/utils/include/encrypt_decrypt.h
static const int AES_GCM_MAX_IVLEN = 12;
static const int AES_GCM_SIV_TAG_LEN = 16;
```

---

## 6. 网络常量

### 6.1 端口范围

```cpp
// include/cast_session_impl_class.h
static constexpr int INVALID_PORT = -1;
static constexpr int MAX_PORT = 65535;
static constexpr int MIN_PORT = 1024;
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `MIN_PORT` | 1024 | 最小端口号 |
| `MAX_PORT` | 65535 | 最大端口号 |
| `INVALID_PORT` | -1 | 无效端口标记 |

### 6.2 Socket 选项

```cpp
// src/channel/src/tcp/tcp_socket.cpp
static const int SOCKET_SEND_BUFFER_SIZE = 512 * 1024;  // 512 KB
static const int SOCKET_RECV_BUFFER_SIZE = 10 * 1024 * 1024;  // 10 MB
```

---

## 7. 枚举定义

### 7.1 会话状态

```cpp
// include/cast_session_common.h
enum class SessionState : uint8_t {
    DEFAULT,
    DISCONNECTED,
    CONNECTING,
    CONNECTED,
    PLAYING,
    PAUSED,
    DISCONNECTING,
    STREAM,
    AUTHING,
    SESSION_STATE_MAX,
};
```

### 7.2 消息 ID

```cpp
// include/cast_session_enums.h
enum MessageId : int {
    MSG_CONNECT,
    MSG_SETUP,
    MSG_SETUP_SUCCESS,
    MSG_SETUP_FAILED,
    MSG_SETUP_DONE,
    MSG_PLAY,
    MSG_PAUSE,
    MSG_PLAY_REQ,
    MSG_PAUSE_REQ,
    MSG_DISCONNECT,
    MSG_CONNECT_TIMEOUT,
    MSG_PROCESS_TRIGGER_REQ,
    MSG_UPDATE_VIDEO_SIZE,
    MSG_STREAM_RECV_ACTION_EVENT_FROM_PEERS,
    MSG_STREAM_SEND_ACTION_EVENT_TO_PEERS,
    MSG_PEER_RENDER_READY,
    MSG_ERROR,
    MSG_SET_CAST_MODE,
    MSG_READY_TO_PLAYING,
    MSG_AUTH,
    MSG_MIRROR_SEND_ACTION_EVENT_TO_PEERS,
    MSG_ID_MAX,
};
```

### 7.3 通道链接类型

```cpp
// src/channel/include/channel_info.h
enum class ChannelLinkType : uint8_t {
    SOFT_BUS = 0,
    TCP = 1,
    VTP = 2,
};
```

### 7.4 模块类型

```cpp
// src/channel/include/channel_info.h
enum class ModuleType : uint8_t {
    AUTH = 0,
    RTSP,
    VIDEO,
    AUDIO,
    RTCP,
    REMOTE_CONTROL,
    STREAM,
    UI_FILES,
    UI_BYTES,
    MODULE_TYPE_MAX,
};
```

### 7.5 端类型

```cpp
// 来自父级框架
enum class EndType : uint8_t {
    SOURCE = 0,
    SINK = 1,
};
```

### 7.6 RTSP 引擎状态

```cpp
// src/rtsp/src/rtsp_basetype.h
enum class RtspEngineState : uint8_t {
    STATE_STOPPED,
    STATE_STARTED,
    STATE_STOPPING,
    STATE_ESTABLISHED,
};
```

### 7.7 RTSP 动作类型

```cpp
// src/rtsp/include/rtsp_basetype.h
enum class RtspActionType : uint8_t {
    RTSP_ACTION_PLAY = 0,
    RTSP_ACTION_PAUSE,
    RTSP_ACTION_TEARDOWN,
    RTSP_ACTION_SET_PARAM,
    RTSP_ACTION_GET_PARAM,
};
```

### 7.8 投屏模式

```cpp
// 来自父级框架
enum class CastMode : uint8_t {
    MIRROR_CAST = 0,
    STREAM_CAST = 1,
};
```

---

## 8. 错误码

### 8.1 通用错误码

```cpp
// 来自父级框架
static const int ERR_NONE = 0;
static const int ERR_NULL_OBJECT = -1;
static const int ERR_INVALID_DATA = -2;
static const int ERR_UNKNOWN_TRANSACTION = -3;
static const int IPC_STUB_WRITE_PARCEL_ERR = -4;
static const int IPC_STUB_ERR = -5;
```

### 8.2 安全错误码

```cpp
// src/utils/include/encrypt_decrypt.h
enum ErrorCode : int {
    SEC_COMMON_ERR_BASE = 0x66010000,
    SEC_ERR_CREATECIPHER_FAIL = SEC_COMMON_ERR_BASE + 10,
    SEC_ERR_ENCRYPTUPDATE_FAIL,
    SEC_ERR_ENCRYPTFINAL_FAIL,
    SEC_ERR_GCMGETTAG_FAIL,
    SEC_ERR_INVALID_AAD,
    SEC_ERR_INVALID_IV,
    SEC_ERR_INVALID_KEY,
    SEC_ERR_INVALID_KEY_LEN,
    // ... 更多错误码
};
```

---

## 9. 日志域

### 9.1 HiLog 域定义

| 域 | 值 | 说明 |
|-----|-----|------|
| CastEngine 主域 | 0xD004601 | 主模块日志 |
| 会话域 | 0xD002B00 | 会话相关 |
| 通道域 | 0xD00ff00 | 通道相关 |
| RTSP 域 | 0xD002b2b | RTSP 协议 |
| 流媒体域 | 0xD003900 | 流媒体播放 |
| 镜像域 | 0xD0015c0 | 镜像播放 |

---

## 10. Feature Flags

### 10.1 编译时特性

```gn
# BUILD.gn 中可能的特性开关
# 当前代码中未定义明确的 feature flags
# 主要通过 deps 控制功能启用
```

### 10.2 运行时配置

```cpp
// 会话属性配置
struct CastSessionProperty {
    CastMode castMode;           // MIRROR_CAST / STREAM_CAST
    ProtocolType protocolType;   // 协议类型选择
    // 其他属性...
};

// RTSP 参数配置
class ParamInfo {
public:
    VideoProperty videoProperty;
    AudioProperty audioProperty;
    UibcProperty uibcProperty;
    bool isSupportVtp;           // 是否支持 VTP
    // 其他参数...
};
```

---

## 11. 相关文档

- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [06_GN_Targets.md](./06_GN_Targets.md) - GN 构建目标
