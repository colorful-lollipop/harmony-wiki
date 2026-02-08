# HiLog 内部 API 文档

> 生成时间: 2026-02-06
> 相关证据: `interfaces/native/kits/include/`, `frameworks/libhilog/`

---

## 目的

本文档描述 HiLog 内部 API、模块接口、依赖方向、稳定性标注。

## 适用范围

适用于内部子系统开发者，通过 `libhilog` 和相关静态库使用 HiLog。

---

## 对外 API（应用层）

### C API（log_c.h）

#### 日志打印宏

| 宏 | 实际函数 | LogType | 用途 |
|------|---------|---------|------|
| `HILOG_DEBUG(type, ...)` | `HiLogPrint()` | type | 打印 DEBUG 级别日志 |
| `HILOG_INFO(type, ...)` | `HiLogPrint()` | type | 打印 INFO 级别日志 |
| `HILOG_WARN(type, ...)` | `HiLogPrint()` | type | 打印 WARN 级别日志 |
| `HILOG_ERROR(type, ...)` | `HiLogPrint()` | type | 打印 ERROR 级别日志 |
| `HILOG_FATAL(type, ...)` | `HiLogPrint()` | type | 打印 FATAL 级别日志 |

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:115-124`

#### 日志打印函数

```c
int HiLogPrint(LogType type, LogLevel level, unsigned int domain,
               const char *tag, const char *fmt, ...)
    __attribute__((__format__(os_log, 5, 6)));
```

**参数**:
- `type`: 日志类型（LOG_APP/LOG_CORE/LOG_INIT/LOG_KMSG）
- `level`: 日志级别（LOG_DEBUG/INFO/WARN/ERROR/FATAL）
- `domain`: 日志域（应用: 0x0-0xFFFF，系统: 0xD000000-0xD0FFFFF）
- `tag`: 日志标签字符串
- `fmt`: 格式化字符串（printf 风格）
- `...`: 可变参数

**返回值**: 成功返回写入字节数，失败返回 -1

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:107`

#### 日志查询函数

```c
bool HiLogIsLoggable(unsigned int domain, const char *tag, LogLevel level);
```

**说明**: 检查指定 domain/tag/level 的日志是否可打印。

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:161`

#### 日志级别设置函数

```c
void HiLogSetAppMinLogLevel(LogLevel level);
```

**说明**: 设置当前应用进程的最低日志级别。

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:197`

```c
void HiLogSetAppLogLevel(LogLevel level, PreferStrategy prefer);
```

**说明**: 设置当前应用进程的日志级别，支持偏好策略。

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:206`

#### 回调设置函数

```c
typedef void (*LogCallback)(const LogType type, const LogLevel level,
                         const unsigned int domain, const char *tag, const char *msg);

void LOG_SetCallback(LogCallback callback);
```

**说明**: 设置用户定义的日志处理函数，接收当前进程的所有日志。

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:174-189`

---

### C++ API（log_cpp.h）

#### HiLog 类

```cpp
namespace OHOS {
namespace HiviewDFX {
class HiLog {
public:
    static void Debug(const HiLogLabel &label, const char *fmt, ...);
    static void Info(const HiLogLabel &label, const char *fmt, ...);
    static void Warn(const HiLogLabel &label, const char *fmt, ...);
    static void Error(const HiLogLabel &label, const char *fmt, ...);
    static void Fatal(const HiLogLabel &label, const char *fmt, ...);
};

} // namespace HiviewDFX
} // namespace OHOS
```

#### HiLogLabel 结构

```cpp
struct HiLogLabel {
    unsigned int type;    // LogType
    unsigned int domain;  // domain ID
    const char *tag;      // tag string
};
```

**证据**: `interfaces/native/innerkits/include/hilog/log_cpp.h:28-34`

---

## 内部 API（子系统层）

### libhilog 基础库

| 头文件 | 职责 | 稳定性 |
|--------|------|--------|
| `hilog_base/log_base.h` | 基础类型定义 | ⭐ 稳定 |
| `hilog_base/hilog_base.c` | 基础实现 | ⭐ 稳定 |
| `hilog/hilog_types.h` | HilogMsg 等传输格式 | ⭐ 稳定 |
| `hilog/hilog_cmd.h` | IOCTL 命令结构 | ⭐ 稳定 |
| `hilog/hilog_common.h` | 错误码、常量 | ⭐ 稳定 |

**稳定性标注**:
- ⭐ 稳定：接口版本化，向后兼容
- 🔧 可变：可能随版本调整

**证据**: `interfaces/native/innerkits/include/hilog/` 目录结构

### libhilog 接口层

| 组件 | 接口 | 稳定性 |
|------|------|--------|
| socket/ | Socket 客户端/服务端 | 🔧 可变 |
| ioctl/ | LogIoctl 控制包装 | 🔧 可变 |
| param/ | 参数配置读取 | 🔧 可变 |
| utils/ | 工具函数 | 🔧 可变 |

---

## 依赖方向

### 纵向依赖（高层 → 低层）

```
应用层
    ↓
对内 API (log_c.h, log_cpp.h)
    ↓
libhilog (对内实现)
    ├─→ socket/
    ├─→ ioctl/
    ├─→ param/
    └─→ utils/
    ↓
Unix Domain Socket
    ↓
hilogd 服务
```

### 横向依赖（同层级）

```
libhilog
    ├─ socket/
    │   ├─ socket.h (基类) ⭐
    │   ├─ socket_client.h (基类) ⭐
    │   ├─ socket_server.h (基类) ⭐
    │   ├─ dgram_socket_client.h
    │   ├─ dgram_socket_server.h
    │   ├─ seq_packet_socket_client.h
    │   └─ seq_packet_socket_server.h
    └─ hilog_input_socket_client.h
```

### hilogd 内部依赖

```
hilogd
    ├─ log_buffer (核心) ⭐
    ├─ log_collector (输入)
    ├─ service_controller (命令处理) ⭐
    ├─ log_persister (输出)
    ├─ flow_control (策略) ⭐
    ├─ log_stats (监控)
    └─ log_domains (domain 管理)
```

**稳定性说明**:
- log_buffer: 核心缓冲区管理，接口稳定
- service_controller: IOCTL 命令处理，接口稳定

---

## 模块接口稳定性

### 稳定接口（可依赖）

| 接口 | 定义位置 | 稳定性理由 |
|------|---------|------------|
| `HiLogPrint()` | `interfaces/native/innerkits/include/hilog/log_c.h:107` | C API，版本化 |
| `HiLog::*()` | `interfaces/native/innerkits/include/hilog/log_cpp.h` | C++ 静态方法，版本化 |
| `HilogMsg` | `frameworks/libhilog/include/hilog_base.h` | 传输格式，版本控制 |
| `HilogBuffer` | `services/hilogd/include/log_buffer.h` | 缓冲区核心类 |
| `ServiceController` | `services/hilogd/include/service_controller.h` | 命令处理核心类 |

### 可变接口（可能变化）

| 接口 | 定义位置 | 可变性理由 |
|------|---------|------------|
| `HilogInputSocketClient` | `frameworks/libhilog/socket/include/hilog_input_socket_client.h` | 客户端实现细节 |
| `LogIoctl` | `frameworks/libhilog/ioctl/include/log_ioctl.h` | 控制包装方式 |
| `FlowControl` | `services/hilogd/include/flow_control.h` | 流控策略可能调整 |
| `LogPersister` | `services/hilogd/include/log_persister.h` | 压缩/落盘策略可能调整 |

---

## 数据类型定义

### LogType（日志类型）

```cpp
typedef enum {
    LOG_TYPE_MIN = 0,
    LOG_APP = 0,           // 应用日志
    LOG_INIT = 1,         // 启动日志
    LOG_CORE = 3,          // 核心服务日志
    LOG_KMSG = 4,          // 内核日志
    LOG_ONLY_PRERELEASE = 5, // 预发布版本日志
    LOG_TYPE_MAX
} LogType;
```

**domain 范围**:
- LOG_APP: 0x0 - 0xFFFF
- 其他: 0xD000000 - 0xD0FFFFF

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:43-58`

### LogLevel（日志级别）

```cpp
typedef enum {
    LOG_LEVEL_MIN = 0,
    LOG_DEBUG = 3,    // 调试信息
    LOG_INFO = 4,     // 普通信息
    LOG_WARN = 5,     // 警告信息
    LOG_ERROR = 6,    // 错误信息
    LOG_FATAL = 7,    // 致命信息
    LOG_LEVEL_MAX,
} LogLevel;
```

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:61-76`

### ErrorCode（错误码）

```cpp
typedef enum {
    SUCCESS_CONTINUE = 1,
    ERR_LOG_LEVEL_INVALID = -2,
    ERR_LOG_TYPE_INVALID = -3,
    ERR_INVALID_RQST_CMD = -4,
    ERR_INVALID_DOMAIN_STR = -5,
    ERR_QUERY_TYPE_INVALID = -8,
    ERR_LOG_PERSIST_FILE_SIZE_INVALID = -11,
    ERR_LOG_PERSIST_FILE_NAME_INVALID = -12,
    ERR_LOG_PERSIST_COMPRESS_BUFFER_EXP = -13,
    ERR_LOG_PERSIST_DIR_OPEN_FAIL = -14,
    ERR_LOG_PERSIST_COMPRESS_INIT_FAIL = -15,
    ERR_LOG_PERSIST_FILE_OPEN_FAIL = -16,
    ERR_LOG_PERSIST_JOBID_FAIL = -18,
    ERR_DOMAIN_INVALID = -19,
    ERR_MSG_LEN_INVALID = -21,
    ERR_LOG_PERSIST_FILE_PATH_INVALID = -25,
    ERR_LOG_PERSIST_JOBID_INVALID = -28,
    ERR_BUFF_SIZE_INVALID = -30,
    ERR_COMMAND_INVALID = -31,
    ERR_LOG_PERSIST_TASK_EXISTED = -32,
    ERR_LOG_FILE_NUM_INVALID = -34,
    ERR_NOT_NUMBER_STR = -35,
    ERR_TOO_MANY_ARGUMENTS = -36,
    ERR_DUPLICATE_OPTION = -37,
    ERR_INVALID_ARGUMENT = -38,
    ERR_TOO_MANY_DOMAINS = -39,
    ERR_INVALID_SIZE_STR = -40,
    ERR_TOO_MANY_PIDS = -41,
    ERR_TOO_MANY_TAGS = -42,
    ERR_TAG_STR_TOO_LONG = -43,
    ERR_REGEX_STR_TOO_LONG = -44,
    ERR_FILE_NAME_TOO_LONG = -45,
    ERR_SOCKET_CLIENT_INIT_FAIL = -46,
    ERR_SOCKET_WRITE_MSG_HEADER_FAIL = -47,
    ERR_SOCKET_WRITE_CMD_FAIL = -48,
    ERR_SOCKET_RECEIVE_RSP = -49,
    ERR_PERSIST_TASK_EMPTY = -50,
    ERR_JOBID_NOT_EXSIST = -60,
    ERR_TOO_MANY_JOBS = -61,
    ERR_STATS_NOT_ENABLE = -62,
    ERR_NO_RUNNING_TASK = -63,
    ERR_NO_PID_PERMISSION = -64,  // PID 过滤权限错误
} ErrorCode;
```

**证据**: `frameworks/libhilog/include/hilog_common.h:68-111`

---

## 常量定义

### 缓冲区常量

| 常量 | 值 | 说明 |
|------|------|------|
| `MIN_BUFFER_SIZE` | 64KB | 最小缓冲区大小 |
| `MAX_BUFFER_SIZE` | 16MB | 最大缓冲区大小 |
| `MIN_LOG_FILE_SIZE` | 64KB | 最小落盘文件大小 |
| `MAX_LOG_FILE_SIZE` | 512MB | 最大落盘文件大小 |
| `MIN_LOG_FILE_NUM` | 2 | 最小落盘文件数 |
| `MAX_LOG_FILE_NUM` | 1000 | 最大落盘文件数 |
| `MAX_JOBS` | 10 | 最大并发落盘任务数 |
| `JOB_ID_MIN` | 10 | 最小任务 ID |
| `JOB_ID_MAX` | UINT_MAX | 最大任务 ID |
| `WAITING_DATA_MS` | 5000ms | Persister 等待超时 |

**证据**: `frameworks/libhilog/include/hilog_common.h:35-49`

### Domain 常量

| 常量 | 值 | 说明 |
|------|------|------|
| `DOMAIN_APP_MIN` | 0x0 | 应用 domain 最小值 |
| `DOMAIN_APP_MAX` | 0xFFFF | 应用 domain 最大值 |
| `DOMAIN_OS_MIN` | 0xD000000 | 系统 domain 最小值 |
| `DOMAIN_OS_MAX` | 0xD0FFFFF | 系统 domain 最大值 |

**证据**: `frameworks/libhilog/include/hilog_common.h:42-47`

---

## 相关跳转链接

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [GN Targets](06_GN_Targets.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| C API 头文件 | interfaces/native/innerkits/include/hilog/log_c.h | 全文 |
| C++ API 头文件 | interfaces/native/innerkits/include/hilog/log_cpp.h | 全文 |
| 基础库头文件 | interfaces/native/innerkits/include/hilog_base/log_base.h | 全文 |
| HilogMsg 结构 | frameworks/libhilog/include/hilog_base.h | 全文 |
| 错误码定义 | frameworks/libhilog/include/hilog_common.h:68-111 | ErrorCode enum |
| 缓冲区常量 | frameworks/libhilog/include/hilog_common.h:35-49 | 全文 |
| Domain 常量 | frameworks/libhilog/include/hilog_common.h:42-47 | 全文 |
