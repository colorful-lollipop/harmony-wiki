# 配置参数

## 目的与适用范围

本文档汇总分布式屏幕的所有配置参数、宏定义和编译开关。

---

## GN构建配置

### need_same_account

**定义文件**: `distributedscreen.gni:20-26`

```gn
declare_args() {
  need_same_account = true
  if (!defined(global_parts_info) || !defined(
          global_parts_info.distributedhardware_distributed_hardware_adapter)) {
    need_same_account = false
  }
}
```

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | true |
| **说明** | 是否支持同账号检查 |
| **使用位置** | `screensourcetrans/BUILD.gn:67-69` |

**条件编译**:
```gn
if (need_same_account) {
  defines += [ "SUPPORT_SAME_ACCOUNT" ]
}
```

**代码影响**:
- 启用: 软总线连接时检查同账号
- 禁用: 跳过同账号检查

---

### build_variant

**类型**: 预定义变量

**使用位置**:
- `sourceservice/BUILD.gn:78-80`
- `sinkservice/BUILD.gn:74-76`

**条件编译**:
```gn
if (build_variant == "root") {
  defines += [ "DUMP_DSCREEN_FILE" ]        // Source端
  defines += [ "DUMP_DSCREENREGION_FILE" ]  // Sink端
}
```

| 变体 | DUMP功能 | 适用场景 |
|------|----------|----------|
| root | 启用 | 调试版本 |
| user | 禁用 | 发布版本 |

---

## C++宏定义

### 日志相关

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `HI_LOG_ENABLE` | 所有BUILD.gn | 启用HiLog日志 |
| `DH_LOG_TAG` | 各模块BUILD.gn | 日志标签 |
| `LOG_DOMAIN` | 各模块BUILD.gn | 日志Domain (0xD004140) |

**模块日志标签**:
| 模块 | 标签 |
|------|------|
| common | "dscreenutil" |
| source_sdk | "dscreensourcesdk" |
| sink_sdk | "dscreensinksdk" |
| source | "dscreensource" |
| sink | "dscreensink" |

### 功能开关

| 宏 | 定义条件 | 说明 |
|----|----------|------|
| `SUPPORT_SAME_ACCOUNT` | need_same_account=true | 同账号支持 |
| `DUMP_DSCREEN_FILE` | build_variant=root | Source端Dump功能 |
| `DUMP_DSCREENREGION_FILE` | build_variant=root | Sink端Dump功能 |

---

## 常量配置

### 路径常量

**文件**: `common/include/dscreen_constants.h`

| 常量 | 值 | 说明 |
|------|-----|------|
| `PKG_NAME` | "ohos.dhardware.dscreen" | 包名 |
| `DATA_SESSION_NAME` | "ohos.dhardware.dscreen.data" | 数据会话名 |
| `JPEG_SESSION_NAME` | "ohos.dhardware.dscreen.jpeg" | JPEG会话名 |
| `DUMP_FILE_PATH` | "/data/data/dscreen" | Dump路径 |
| `DSCREEN_PROCESS_NAME` | "dscreen" | 进程名 |
| `DSCREEN_VERSION` | "1.0" | 版本号 |

### SA ID常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `DISTRIBUTED_HARDWARE_SCREEN_SOURCE_SA_ID` | 4807 | Source端SA ID |
| `DISTRIBUTED_HARDWARE_SCREEN_SINK_SA_ID` | 4808 | Sink端SA ID |

### 限制常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `DSCREEN_MAX_SESSION_NAME_LEN` | 50 | 会话名最大长度 |
| `DSCREEN_MAX_DEVICE_ID_LEN` | 100 | 设备ID最大长度 |
| `DSCREEN_MAX_RECV_DATA_LEN` | 104857600 (100MB) | 最大接收数据 |
| `DSCREEN_MAX_VIDEO_DATA_WIDTH` | 2560 | 最大视频宽度 |
| `DSCREEN_MAX_VIDEO_DATA_HEIGHT` | 2772 | 最大视频高度 |
| `DUMP_FILE_MAX_SIZE` | 295MB | Dump文件大小限制 |
| `MAX_YUV420_BUFFER_SIZE` | ~6MB | YUV缓冲区大小 |
| `DATA_QUEUE_MAX_SIZE` | 1000 | 数据队列最大长度 |
| `DATA_BUFFER_MAX_SIZE` | 10MB | 数据缓冲区大小 |

### 超时常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `SCREEN_LOADSA_TIMEOUT_MS` | 10000 (10s) | SA加载超时 |
| `DECODE_WAIT_MILLISECONDS` | 5000 (5s) | 解码等待超时 |
| `WAIT_TIMEOUT_MS` | 5000 (5s) | 通用等待超时 |
| `WATCHDOG_INTERVAL_TIME_MS` | 20000 (20s) | 看门狗间隔 |
| `WATCHDOG_DELAY_TIME_MS` | 5000 (5s) | 看门狗延迟 |
| `SESSION_WAIT_SECONDS` | 5 | 会话等待秒数 |
| `DATA_WAIT_SECONDS` | 1 | 数据等待秒数 |
| `TASK_WAIT_SECONDS` | 1 | 任务等待秒数 |

### 视频参数默认值

| 常量 | 值 | 说明 |
|------|-----|------|
| `DEFAULT_DENSITY` | 2.0 | 屏幕密度 |
| `DEFAULT_FPS` | 60.0 | 默认帧率 |
| `DEFAULT_CODECTYPE` | VIDEO_CODEC_TYPE_VIDEO_H264 | 默认编码 |
| `DEFAULT_VIDEO_FORMAT` | VIDEO_DATA_FORMAT_NV12 | 默认格式 |
| `BIT_RATE` | 12000000 (12Mbps) | 默认码率 |
| `JPEG_QUALITY` | 80 | JPEG质量 |

---

## 枚举定义

### 屏幕状态

**文件**: `common/include/dscreen_constants.h:30-38`

```cpp
enum DScreenState {
    DISABLED,      // 禁用
    ENABLED,       // 已使能
    DISABLING,     // 禁用中
    ENABLING,      // 使能中
    CONNECTING,    // 连接中
    CONNECTED,     // 已连接
    DISCONNECTING, // 断开中
};
```

### 任务类型

```cpp
enum TaskType {
    TASK_ENABLE,    // 使能任务
    TASK_DISABLE,   // 禁用任务
    TASK_CONNECT,   // 连接任务
    TASK_DISCONNECT,// 断开任务
};
```

### 编码类型

```cpp
enum CodecType : uint8_t {
    VIDEO_CODEC_TYPE_VIDEO_H264 = 0,   // H264 (默认)
    VIDEO_CODEC_TYPE_VIDEO_H265 = 1,   // H265
    VIDEO_CODEC_TYPE_VIDEO_MPEG4 = 2,  // MPEG4
};
```

### 视频格式

```cpp
enum VideoFormat : uint8_t {
    VIDEO_DATA_FORMAT_YUVI420 = 0,   // YUV I420
    VIDEO_DATA_FORMAT_NV12 = 1,      // NV12 (默认)
    VIDEO_DATA_FORMAT_NV21 = 2,      // NV21
    VIDEO_DATA_FORMAT_RGBA8888 = 3,  // RGBA8888
};
```

### 数据类型

```cpp
enum DataType : uint8_t {
    VIDEO_FULL_SCREEN_DATA = 0,  // 全屏数据
    VIDEO_PART_SCREEN_DATA = 1,  // 局部数据
};
```

---

## 版本控制

### 版本号定义

| 常量 | 值 | 说明 |
|------|-----|------|
| `DSCREEN_VERSION` | "1.0" | 分布式屏幕版本 |
| `DSCREEN_MIN_VERSION` | 1 | 最小版本 |
| `AV_TRANS_SUPPORTED_VERSION` | 3 | AV传输支持版本 |

### 版本判断

**代码位置**: `services/screenservice/sourceservice/dscreenmgr/2.0/src/dscreen_manager.cpp`

```cpp
bool IsSupportAVTransEngine(int32_t version) {
    return version >= AV_TRANS_SUPPORTED_VERSION;
}
```

- version >= 3: 使用AVTrans引擎 (V2.0)
- version < 3: 使用传统传输 (V1.0)

---

## 系统参数

### 持久化参数

| 参数名 | 值 | 说明 |
|--------|-----|------|
| `persist.distributedhardware.dscreen.partial.refresh.enable` | 0/1 | 局部刷新开关 |

**代码位置**: `common/include/dscreen_constants.h:209`

```cpp
constexpr const char* PARTIAL_REFRESH_PARAM = "persist.distributedhardware.dscreen.partial.refresh.enable";
```

---

## JSON字段键名

### 屏幕信息键

| 常量 | 值 |
|------|-----|
| `KEY_VERSION` | "screenVersion" |
| `KEY_DISPLAY_ID` | "displayId" |
| `KEY_SCREEN_ID` | "screenId" |
| `KEY_DISPLAY_RECT` | "displayRect" |
| `KEY_SCREEN_RECT` | "screenRect" |
| `KEY_POINT_START_X` | "startX" |
| `KEY_POINT_START_Y` | "startY" |
| `KEY_WIDTH` | "width" |
| `KEY_HEIGHT` | "height" |
| `KEY_VIDEO_WIDTH` | "videoWidth" |
| `KEY_VIDEO_HEIGHT` | "videoHeight" |
| `KEY_COLOR_FORMAT` | "colorFormat" |
| `KEY_FPS` | "fps" |
| `KEY_CODECTYPE` | "codecType" |

### 窗口信息键

| 常量 | 值 |
|------|-----|
| `SOURCE_WIN_ID` | "sourceWinId" |
| `SOURCE_DEV_ID` | "sourceDevId" |
| `SINK_DEV_ID` | "sinkDevId" |
| `SOURCE_WIN_WIDTH` | "sourceWinWidth" |
| `SOURCE_WIN_HEIGHT` | "sourceWinHeight" |
| `SINK_SHOW_WIN_ID` | "sinkShowWinId" |

---

## 相关跳转

- [GN构建系统](../05_Build_System.md) - 构建配置详解
- [项目概览](../00_Overview.md) - 项目基本信息