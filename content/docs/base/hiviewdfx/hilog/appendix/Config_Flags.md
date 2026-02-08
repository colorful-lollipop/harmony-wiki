# HiLog 配置标志参考

> 生成时间: 2026-02-06
> 相关证据: `hilog.gni`, `services/hilogd/etc/hilogd.cfg`, `frameworks/libhilog/include/hilog_common.h`

---

## 目的

本文档列出 HiLog 模块的所有关键宏、feature flags、编译选项，供开发者参考。

## 适用范围

涵盖全局 GN 配置、常量定义、条件编译选项。

---

## 全局 GN 配置（hilog.gni）

### Feature Flags

| Flag | 默认值 | 类型 | 说明 |
|------|---------|------|------|
| `hilog_native_feature_ohcore` | false | boolean | 控制 ohcore 原生功能，减小 ROM/RAM |
| `hilog_feature_support_usr_symlink` | false | boolean | 启用 `/usr/bin/hilog` 符号链接 |

**证据**: `hilog.gni:24-26`

### 平台列表

```gni
platforms = [
  "ohos",
  "windows",
  "mac",
  "linux",
  "android",
  "ios",
]
```

**证据**: `hilog.gni:14-21`

---

## 服务配置（hilogd.cfg）

### Socket 配置

| Socket | 类型 | 路径 | 权限 | UID | GID | 选项 |
|--------|------|------------|--------|------|---------|----------|
| hilogInput | SOCK_DGRAM | `/dev/unix/socket/hilogInput` | 0222 | logd | log | SOCKET_OPTION_PASSCRED |
| hilogOutput | SOCK_SEQPACKET | `/dev/unix/socket/hilogOutput` | 0666 | logd | log | SOCKET_OPTION_PASSCRED |
| hilogControl | SOCK_SEQPACKET | `/dev/unix/socket/hilogControl` | 0660 | logd | log | - |

**证据**: `services/hilogd/etc/hilogd.cfg`（内容）

### 服务属性

| 属性 | 值 | 说明 |
|------|------|------|
| name | "hilogd" | 服务名称 |
| uid | "logd" | 运行用户 UID (1036) |
| gid | "log" | 运行组 GID (1036) |
| caps | ["SYSLOG"] | 能力列表（读取内核日志） |

**证据**: `services/hilogd/etc/hilogd.cfg`（JSON 内容）

---

## 缓冲区常量（hilog_common.h）

### 大小限制

| 常量 | 值 | 说明 |
|--------|------|------|
| `MIN_BUFFER_SIZE` | 64 KB | 最小缓冲区大小 |
| `MAX_BUFFER_SIZE` | 16 MB | 最大缓冲区大小 |
| `MIN_BUFFER_SIZE_BYTES` | 64 * 1024 | 最小缓冲区（字节） |
| `MAX_BUFFER_SIZE_BYTES` | 16 * 1024 * 1024 | 最大缓冲区（字节） |

**证据**: `frameworks/libhilog/include/hilog_common.h:35-36`

### 日志文件大小限制

| 常量 | 值 | 说明 |
|--------|------|------|
| `MIN_LOG_FILE_SIZE` | 64 KB | 最小落盘文件大小 |
| `MAX_LOG_FILE_SIZE` | 512 MB | 最大落盘文件大小 |
| `MIN_LOG_FILE_SIZE_BYTES` | 64 * 1024 | 最小文件大小（字节） |
| `MAX_LOG_FILE_SIZE_BYTES` | 512 * 1024 * 1024 | 最大文件大小（字节） |

**证据**: `frameworks/libhilog/include/hilog_common.h:38-41`

### 日志文件数量限制

| 常量 | 值 | 说明 |
|--------|------|------|
| `MIN_LOG_FILE_NUM` | 2 | 最小落盘文件数 |
| `MAX_LOG_FILE_NUM` | 1000 | 最大落盘文件数 |
| `WAITING_DATA_MS` | 5000 ms | Persister 等待超时 |

**证据**: `frameworks/libhilog/include/hilog_common.h:40-41`

### 落盘任务配置

| 常量 | 值 | 说明 |
|--------|------|------|
| `MAX_JOBS` | 10 | 最大并发落盘任务数 |
| `JOB_ID_MIN` | 10 | 最小任务 ID |
| `JOB_ID_MAX` | UINT_MAX | 最大任务 ID |

**证据**: `frameworks/libhilog/include/hilog_common.h:34-47`

---

## Domain 范围定义

### 应用 Domain

| 常量 | 值 | 说明 |
|--------|------|------|
| `DOMAIN_APP_MIN` | 0x0 | 应用 domain 最小值 |
| `DOMAIN_APP_MAX` | 0xFFFF | 应用 domain 最大值 |

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:31-32`

### 系统 Domain

| 常量 | 值 | 说明 |
|--------|------|------|
| `DOMAIN_OS_MIN` | 0xD000000 | 系统 domain 最小值 |
| `DOMAIN_OS_MAX` | 0xD0FFFFF | 系统 domain 最大值 |

**证据**: `frameworks/libhilog/include/hilog_common.h:42-47`

---

## 日志类型和级别（log_c.h）

### LogType 枚举

| 值 | 名称 | 说明 |
|------|------|------|
| 0 | LOG_TYPE_MIN | 最小值 |
| 1 | LOG_INIT | 启动阶段重要日志 |
| 3 | LOG_CORE | 核心服务日志 |
| 4 | LOG_KMSG | 内核消息日志 |
| 5 | LOG_ONLY_PRERELEASE | 预发布版本日志（与 CORE 共享 buffer） |
| 5 | LOG_TYPE_MAX | 最大值 |

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:43-58`

### LogLevel 枚举

| 值 | 名称 | 说明 |
|------|------|------|
| 3 | LOG_DEBUG | 调试信息 |
| 4 | LOG_INFO | 普通信息 |
| 5 | LOG_WARN | 警告信息 |
| 6 | LOG_ERROR | 错误信息 |
| 7 | LOG_FATAL | 致命信息 |
| 0 | LOG_LEVEL_MIN | 最小值 |
| 7 | LOG_LEVEL_MAX | 最大值 |

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:61-76`

### PreferStrategy 枚举

| 值 | 名称 | 说明 |
|------|------|------|
| 0 | UNSET_LOGLEVEL | 取消之前的级别设置 |
| 1 | PREFER_CLOSE_LOG | 采用更严格的级别（max(新级别, 系统级别)） |
| 2 | PREFER_OPEN_LOG | 采用更宽松的级别（min(新级别, 系统级别)） |

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:79-94`

---

## 条件编译选项

### 平台标识

| Define | 条件 | 说明 |
|--------|--------|------|
| `__OHOS__` | `platform == "ohos"` | OpenHarmony 平台 |
| `__WINDOWS__` | `is_mingw` | Windows 平台 |
| `__MAC__` | `is_mac` | macOS 平台 |
| `__LINUX__` | `is_linux` | Linux 平台 |
| `ANDROID_PLATFORM` | `target_os == "android"` | Android 平台 |
| `IOS_PLATFORM` | `target_os == "ios"` | iOS 平台 |

**证据**: `frameworks/libhilog/BUILD.gn:91-93`, `interfaces/native/innerkits/BUILD.gn:73-74`

### 功能开关

| Define | 条件 | 说明 |
|--------|--------|------|
| `__RECV_MSG_WITH_UCRED_` | Non-desktop platforms | 在 socket 消息中包含用户凭证 |
| `HILOG_USE_MUSL` | `use_musl` | 使用 musl libc |
| `HILOG_PROHIBIT_ALLOCATION` | libhilog_base | 禁止动态内存分配 |

**证据**:
- `services/hilogd/BUILD.gn:48`
- `interfaces/native/innerkits/BUILD.gn:181-184`

---

## 错误码定义（ErrorCode 枚举）

### 成功码

| 值 | 名称 | 说明 |
|------|------|------|
| 1 | SUCCESS_CONTINUE | 操作成功继续 |

### 配置错误

| 值 | 名称 | 说明 |
|------|------|------|
| -2 | ERR_LOG_LEVEL_INVALID | 日志级别无效 |
| -3 | ERR_LOG_TYPE_INVALID | 日志类型无效 |
| -4 | ERR_INVALID_RQST_CMD | 无效的请求命令 |
| -5 | ERR_INVALID_DOMAIN_STR | 无效的 domain 字符串 |
| -8 | ERR_QUERY_TYPE_INVALID | 无效的查询类型 |
| -19 | ERR_DOMAIN_INVALID | domain 范围无效 |

### 落盘错误

| 值 | 名称 | 说明 |
|------|------|------|
| -11 | ERR_LOG_PERSIST_FILE_SIZE_INVALID | 落盘文件大小无效 |
| -12 | ERR_LOG_PERSIST_FILE_NAME_INVALID | 落盘文件名无效 |
| -13 | ERR_LOG_PERSIST_COMPRESS_BUFFER_EXP | 压缩缓冲区溢出 |
| -14 | ERR_LOG_PERSIST_DIR_OPEN_FAIL | 无法打开落盘目录 |
| -15 | ERR_LOG_PERSIST_COMPRESS_INIT_FAIL | 压缩初始化失败 |
| -16 | ERR_LOG_PERSIST_FILE_OPEN_FAIL | 无法打开落盘文件 |
| -18 | ERR_LOG_PERSIST_JOBID_FAIL | 任务 ID 失败 |
| -25 | ERR_LOG_PERSIST_FILE_PATH_INVALID | 落盘文件路径无效 |
| -28 | ERR_LOG_PERSIST_JOBID_INVALID | 任务 ID 无效 |
| -30 | ERR_BUFF_SIZE_INVALID | 缓冲区大小无效 |
| -31 | ERR_COMMAND_INVALID | 无效的命令 |
| -32 | ERR_LOG_PERSIST_TASK_EXISTED | 落盘任务已存在 |
| -34 | ERR_LOG_FILE_NUM_INVALID | 日志文件数量无效 |
| -50 | ERR_PERSIST_TASK_EMPTY | 落盘任务为空 |
| -60 | ERR_JOBID_NOT_EXSIST | 任务 ID 不存在 |
| -61 | ERR_TOO_MANY_JOBS | 落盘任务过多 |

### 其他错误

| 值 | 名称 | 说明 |
|------|------|------|
| -21 | ERR_MSG_LEN_INVALID | 消息长度无效 |
| -35 | ERR_NOT_NUMBER_STR | 数字字符串无效 |
| -36 | ERR_TOO_MANY_ARGUMENTS | 参数过多 |
| -37 | ERR_DUPLICATE_OPTION | 重复的选项 |
| -38 | ERR_INVALID_ARGUMENT | 无效的参数 |
| -39 | ERR_TOO_MANY_DOMAINS | 过多 domain |
| -40 | ERR_INVALID_SIZE_STR | 大小字符串无效 |
| -41 | ERR_TOO_MANY_PIDS | 过多 PID |
| -42 | ERR_TOO_MANY_TAGS | 过多 tag |
| -43 | ERR_TAG_STR_TOO_LONG | tag 字符串过长 |
| -44 | ERR_REGEX_STR_TOO_LONG | 正则字符串过长 |
| -45 | ERR_FILE_NAME_TOO_LONG | 文件名过长 |
| -46 | ERR_SOCKET_CLIENT_INIT_FAIL | Socket 客户端初始化失败 |
| -47 | ERR_SOCKET_WRITE_MSG_HEADER_FAIL | 写入消息头失败 |
| -48 | ERR_SOCKET_WRITE_CMD_FAIL | 写入命令失败 |
| -49 | ERR_SOCKET_RECEIVE_RSP | 接收响应失败 |
| -62 | ERR_STATS_NOT_ENABLE | 统计未启用 |
| -63 | ERR_NO_RUNNING_TASK | 无运行中的任务 |
| -64 | ERR_NO_PID_PERMISSION | 无 PID 过滤权限 |

**证据**: `frameworks/libhilog/include/hilog_common.h:68-111`

---

## 相关跳转链接

- [GN Targets](06_GN_Targets.md)
- [架构说明](03_Architecture.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| 全局配置 | hilog.gni:14-26 | platforms, feature flags |
| 服务配置 | services/hilogd/etc/hilogd.cfg | socket 配置, 服务属性 |
| 缓冲区常量 | frameworks/libhilog/include/hilog_common.h:35-49 | 大小限制, 文件限制 |
| Domain 范围 | interfaces/native/innerkits/include/hilog/log_c.h:31-32 | DOMAIN_APP/MIN/MAX, DOMAIN_OS/MIN/MAX |
| 日志类型 | interfaces/native/innerkits/include/hilog/log_c.h:43-58 | LogType enum |
| 日志级别 | interfaces/native/innerkits/include/hilog/log_c.h:61-76 | LogLevel enum |
| 错误码 | frameworks/libhilog/include/hilog_common.h:68-111 | ErrorCode enum |
