# 02 - OHOS 适配详情 (核心文档)

## 2.1 适配策略概述

### 无 Patch 文件的独特适配

与传统第三方库的 Patch 方式不同，PulseAudio 在 OpenHarmony 中采用**文件替换式适配**策略：

```
传统方式：
  上游源码 + Patch 文件 → 修改后的源码

PulseAudio OHOS 方式：
  上游源码 (大部分不变) 
  + ohos_* 替代文件 (关键文件替换)
  + HAVE_NO_OHOS 条件编译 (功能裁剪)
  + 新增 OHOS 专用文件 (扩展功能)
```

### 适配文件清单

| 类型 | 文件路径 | 说明 |
|-----|---------|------|
| **替代文件** | `src/daemon/ohos_pa_main.c` | 主入口替换，提供 `ohos_pa_main()` |
| **替代文件** | `src/daemon/ohos_daemon-conf.c` | 守护进程配置默认值调整 |
| **替代文件** | `src/pulsecore/ohos_socket-server.c` | Socket 服务器适配 init 机制 |
| **新增文件** | `include/log/audio_log.h` | HiLog 日志系统集成 |
| **修改文件** | `src/pulsecore/native-common.h` | 新增 OHOS 协议命令 |

---

## 2.2 详细适配分析

### 2.2.1 主入口适配 (ohos_pa_main.c)

**原始文件**：`src/daemon/main.c`
**适配文件**：`src/daemon/ohos_pa_main.c`

#### 关键差异

| 方面 | 原生实现 | OHOS 适配 |
|-----|---------|----------|
| 入口函数 | `main()` | `ohos_pa_main()` |
| 模块加载 | 动态加载 (.so 文件) | 静态链接 (条件编译禁用) |
| Socket 关闭 | 关闭所有文件描述符 | 保留 passed_fds |
| 权限设置 | umask(0077) | 移除 (注释显示 HAVE_NO_OHOS) |
| 优先级提升 | pa_raise_priority() | 条件编译禁用 |

#### HAVE_NO_OHOS 宏的使用

```c
// ohos_pa_main.c 中的条件编译示例

// 1. 禁用动态模块加载
#ifdef HAVE_NO_OHOS
    LTDL_SET_PRELOADED_SYMBOLS();
    pa_ltdl_init();
    ltdl_init = true;
    ...
#endif

// 2. 禁用文件描述符关闭
#ifdef HAVE_NO_OHOS
    pa_close_allv(passed_fds);
    pa_xfree(passed_fds);
#endif

// 3. 禁用权限掩码设置
#ifdef HAVE_NO_OHOS
    umask(0077);
#endif

// 4. 禁用优先级提升
#ifdef HAVE_NO_OHOS
    if (conf->high_priority)
        pa_raise_priority(conf->nice_level);
#endif

// 5. 禁用模块清理
#ifdef HAVE_NO_OHOS
    if (ltdl_init)
        pa_ltdl_done();
#endif
```

#### 权限管理增强

OHOS 版本在启动完成后添加了权限设置：

```c
// ohos_pa_main.c (约第 1342-1348 行)
change_permission("/data/data/.pulse_dir", 0755);
change_permission("/data/data/.pulse_dir/runtime", 0755);
change_permission("/data/data/.pulse_dir/state", 0755);
change_permission("/data/data/.pulse_dir/state/cookie", 0664);
change_permission("/data/data/.pulse_dir/runtime/cli", 0660);
change_permission("/data/data/.pulse_dir/runtime/native", 0666);
```

**OH 价值**：确保音频服务在沙箱环境中的正确权限，使客户端能够正常连接。

---

### 2.2.2 Socket 服务器适配 (ohos_socket-server.c)

**原始文件**：`src/pulsecore/socket-server.c`
**适配文件**：`src/pulsecore/ohos_socket-server.c`

#### 核心适配：Init Socket 机制

OpenHarmony 使用 init 进程预创建 socket，服务通过 `GetControlSocket()` 获取已创建的 socket。

```c
// ohos_socket-server.c
#define OHOS_SOCKET_PATH "/dev/unix/socket/native"
#define OHOS_SOCKET_NAME "native"

// 在 pa_socket_server_new_unix() 函数中
if (!strcmp(filename, OHOS_SOCKET_PATH)) {
    // OHOS 模式：使用 init 预创建的 socket
    fd = GetControlSocket(OHOS_SOCKET_NAME);
    if (fd < 0) {
        pa_log_error("GetControlSocket: fd < 0: %d", fd);
        goto fail;
    }
    
    pa_make_socket_low_delay(fd);
    if (listen(fd, 5) < 0) {
        pa_log_error("listen(): %s", pa_cstrerror(errno));
        goto fail;
    }
} else {
    // 传统模式：自主创建 socket
    if ((fd = pa_socket_cloexec(PF_UNIX, SOCK_STREAM, 0)) < 0) {
        ...
    }
    // ... bind, chmod, listen
}
```

#### 适配对比

| 步骤 | 原生实现 | OHOS 适配 |
|-----|---------|----------|
| Socket 创建 | `socket()` 系统调用 | `GetControlSocket()` 从 init 获取 |
| 路径绑定 | `bind()` 绑定到文件路径 | 跳过 (已在 init 中完成) |
| 权限设置 | `chmod(filename, 0777)` | 跳过 (init 已设置) |
| 监听 | `listen(fd, 5)` | 相同 |

#### 依赖关系

```
ohos_socket-server.c
    ├── <init_socket.h>     (init 提供的头文件)
    ├── GetControlSocket()  (libbegetutil 提供)
    └── init 进程           (预创建 socket)
```

**OH 价值**：与 OpenHarmony 的 init 机制集成，支持系统服务的统一 socket 管理。

---

### 2.2.3 日志系统适配 (audio_log.h)

**新增文件**：`include/log/audio_log.h`

#### 功能概述

将 PulseAudio 的日志系统替换为 OpenHarmony 的 HiLog 日志系统。

#### 关键定义

```c
// 日志域定义
#define LOG_DOMAIN 0xD002B88
#define LOG_TAG "PulseAudio"

// 基础宏
#define DECORATOR_HILOG(op, fmt, args...) \
    do { \
        op(LOG_CORE, fmt, ##args); \
    } while (0)

// 日志级别宏
#define AUDIO_DEBUG_LOG(fmt, ...) DECORATOR_HILOG(HILOG_DEBUG, fmt, ##__VA_ARGS__)
#define AUDIO_ERR_LOG(fmt, ...)   DECORATOR_HILOG(HILOG_ERROR, fmt, ##__VA_ARGS__)
#define AUDIO_WARNING_LOG(fmt, ...) DECORATOR_HILOG(HILOG_WARN, fmt, ##__VA_ARGS__)
#define AUDIO_INFO_LOG(fmt, ...)  DECORATOR_HILOG(HILOG_INFO, fmt, ##__VA_ARGS__)
#define AUDIO_FATAL_LOG(fmt, ...) DECORATOR_HILOG(HILOG_FATAL, fmt, ##__VA_ARGS__)
```

#### HiTrace 性能跟踪集成

```c
#ifdef FEATURE_HITRACE_METER
#include "hitrace_meter_c.h"
#define HITRACE_AUDIO_TAG (1ULL << 35)
#endif

// 跟踪函数
inline void CallStart(const char *traceName) {
#ifdef FEATURE_HITRACE_METER
    HiTraceStartTrace(HITRACE_AUDIO_TAG, traceName);
#endif
}

inline void CallEnd() {
#ifdef FEATURE_HITRACE_METER
    HiTraceFinishTrace(HITRACE_AUDIO_TAG);
#endif
}
```

#### 错误码定义

```c
#define AUDIO_OK                 0
#define AUDIO_INVALID_PARAM     (-1)
#define AUDIO_INIT_FAIL         (-2)
#define AUDIO_ERR               (-3)
#define AUDIO_PERMISSION_DENIED (-4)
```

#### 辅助宏

```c
// 条件检查并记录错误
#define CHECK_AND_RETURN_RET_LOG(cond, ret, fmt, ...) \
    do { \
        if (!(cond)) { \
            AUDIO_ERR_LOG(fmt, ##__VA_ARGS__); \
            return ret; \
        } \
    } while (0)

#define CHECK_AND_RETURN_LOG(cond, fmt, ...) ...
#define CHECK_AND_BREAK_LOG(cond, fmt, ...) ...
```

**OH 价值**：统一日志输出到 HiLog 系统，支持故障诊断和性能分析。

---

### 2.2.4 协议扩展 (native-common.h)

**修改文件**：`src/pulsecore/native-common.h`

#### 新增协议命令

```c
enum {
    // ... 原有命令 ...
    
    /* SERVER->CLIENT */
    PA_COMMAND_UNDERFLOW_OHOS,  // <-- OHOS 新增
    
    PA_COMMAND_MAX
};
```

#### 命令处理

在 `src/pulse/pdispatch.c` 中注册处理函数：

```c
// pdispatch.c 中
[PA_COMMAND_UNDERFLOW_OHOS] = pa_command_overflow_or_underflow,
```

#### 用途说明

`PA_COMMAND_UNDERFLOW_OHOS` 命令用于在音频下溢 (underflow) 发生时通知客户端，这是 OHOS 音频框架的特定需求。

**OH 价值**：支持 OHOS 音频框架的流状态监控。

---

## 2.3 配置适配 (ohos_daemon-conf.c)

**原始文件**：`src/daemon/daemon-conf.c`
**适配文件**：`src/daemon/ohos_daemon-conf.c`

### 主要变更

该文件主要提供与原生实现相同的配置解析功能，但可能包含 OHOS 特定的默认值调整。

### 关键配置项

```c
static const pa_daemon_conf default_conf = {
    .daemonize = false,
    .fail = true,
    .high_priority = true,
    .nice_level = -11,
    .realtime_scheduling = true,
    .realtime_priority = 5,
    .exit_idle_time = -1,  // 禁用空闲退出
    // ...
};
```

---

## 2.4 Sonic 变速库

### 概述
Sonic 是一个音频变速库，与 PulseAudio 一起提供音频变速播放功能。

### BUILD.gn 配置

```python
ohos_shared_library("sonic") {
    branch_protector_ret = "pac_ret"  # 返回地址保护
    sources = [ "./sonic.c" ]
    innerapi_tags = [ "platformsdk" ]
}
```

### 使用场景
音频框架使用 Sonic 实现：
- 音频播放速度调整
- 音频时间拉伸/压缩
- 不变调的变速播放

---

## 2.5 升级建议

### 升级上游版本时的检查清单

1. **协议兼容性**
   - [ ] 确认 `PA_COMMAND_UNDERFLOW_OHOS` 命令 ID 未冲突
   - [ ] 检查新增命令是否影响 OHOS 命令

2. **替代文件同步**
   - [ ] 对比 `main.c` 和 `ohos_pa_main.c`，同步关键变更
   - [ ] 对比 `daemon-conf.c` 和 `ohos_daemon-conf.c`
   - [ ] 对比 `socket-server.c` 和 `ohos_socket-server.c`

3. **HAVE_NO_OHOS 宏**
   - [ ] 确认新增代码是否需要条件编译
   - [ ] 检查现有条件编译是否仍适用

4. **Init 集成**
   - [ ] 确认 Socket 机制未被破坏
   - [ ] 验证 `GetControlSocket` 调用仍有效

5. **HiLog 集成**
   - [ ] 确认日志宏定义未被破坏
   - [ ] 验证日志输出正常

### 可以推向上游的变更

以下变更可能适合提交给上游 PulseAudio：
- 更灵活的主入口命名（允许自定义入口函数）
- 可配置的 socket 获取机制（支持预创建 socket）
- 模块化的日志系统接口

### OHOS 特有变更 (不建议推送)

以下变更是 OHOS 特有，不应推向上游：
- `PA_COMMAND_UNDERFLOW_OHOS` 协议命令
- `GetControlSocket` 调用（OHOS init 特有）
- HiLog 日志宏定义

---

## 2.6 回归风险

| 风险项 | 风险等级 | 说明 |
|-------|---------|------|
| Socket 机制变更 | 高 | 依赖 init 进程，升级时需确保兼容 |
| 协议命令冲突 | 中 | 新增命令 ID 可能与上游冲突 |
| HAVE_NO_OHOS 覆盖不全 | 中 | 新增功能可能需要添加到条件编译 |
| HiLog API 变更 | 低 | OHOS 内部 API，相对稳定 |
