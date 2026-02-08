# 05 - API 差异分析

## 5.1 概述

PulseAudio 在 OpenHarmony 中的 API 差异主要来源于：
1. **新增 OHOS 专用协议命令**
2. **日志系统的替换**
3. **Socket 机制的改变**
4. **功能裁剪导致的 API 行为变化**

---

## 5.2 新增 API

### 5.2.1 协议命令扩展

#### PA_COMMAND_UNDERFLOW_OHOS

**定义位置**：`src/pulsecore/native-common.h`

```c
enum {
    // ... 原有命令 ...
    
    /* SERVER->CLIENT */
    PA_COMMAND_UNDERFLOW_OHOS,  // OHOS 新增
    
    PA_COMMAND_MAX
};
```

**功能**：音频流下溢事件通知

**使用场景**：
- 当播放缓冲区下溢时，服务器向客户端发送此命令
- 客户端可据此调整缓冲区策略或显示提示

**处理函数**：`src/pulse/pdispatch.c`

```c
[PA_COMMAND_UNDERFLOW_OHOS] = pa_command_overflow_or_underflow,
```

**影响**：
- 原生 PulseAudio 客户端不识别此命令
- OHOS 客户端应处理此命令以优化音频体验

### 5.2.2 HiLog 日志宏

**定义位置**：`include/log/audio_log.h`

#### 基础日志宏

| 宏 | 级别 | 说明 |
|---|------|------|
| `AUDIO_DEBUG_LOG(fmt, ...)` | DEBUG | 调试日志 |
| `AUDIO_INFO_LOG(fmt, ...)` | INFO | 信息日志 |
| `AUDIO_WARNING_LOG(fmt, ...)` | WARN | 警告日志 |
| `AUDIO_ERR_LOG(fmt, ...)` | ERROR | 错误日志 |
| `AUDIO_FATAL_LOG(fmt, ...)` | FATAL | 致命错误 |

#### 预发布日志宏

| 宏 | 级别 | 说明 |
|---|------|------|
| `AUDIO_PRERELEASE_LOGD(fmt, ...)` | DEBUG | 预发布调试 |
| `AUDIO_PRERELEASE_LOGI(fmt, ...)` | INFO | 预发布信息 |
| `AUDIO_PRERELEASE_LOGW(fmt, ...)` | WARN | 预发布警告 |
| `AUDIO_PRERELEASE_LOGE(fmt, ...)` | ERROR | 预发布错误 |
| `AUDIO_PRERELEASE_LOGF(fmt, ...)` | FATAL | 预发布致命 |

#### 辅助检查宏

```c
// 条件检查，失败时记录错误并返回
CHECK_AND_RETURN_RET_LOG(cond, ret, fmt, ...)

// 条件检查，失败时记录错误并返回 (void)
CHECK_AND_RETURN_LOG(cond, fmt, ...)

// 条件检查，失败时记录错误并跳出循环
CHECK_AND_BREAK_LOG(cond, fmt, ...)
```

#### 错误码定义

```c
#define AUDIO_OK                 0   // 成功
#define AUDIO_INVALID_PARAM     (-1) // 无效参数
#define AUDIO_INIT_FAIL         (-2) // 初始化失败
#define AUDIO_ERR               (-3) // 通用错误
#define AUDIO_PERMISSION_DENIED (-4) // 权限拒绝
```

#### 性能跟踪函数

```c
// 开始跟踪
void CallStart(const char *traceName);

// 结束跟踪
void CallEnd();
```

**使用示例**：

```c
#include "log/audio_log.h"

void ProcessAudio() {
    AUDIO_INFO_LOG("Processing audio started");
    CallStart("AudioProcessing");
    
    // ... 处理逻辑 ...
    
    CallEnd();
    AUDIO_INFO_LOG("Processing audio completed");
}
```

### 5.2.3 OHOS 主入口函数

**定义**：`src/daemon/ohos_pa_main.c`

```c
int ohos_pa_main(int argc, char *argv[]);
```

**说明**：
- 替代原生 `main()` 函数
- 提供 OHOS 特定的初始化逻辑
- 在 `ohos_pa_main.c` 中定义，而非 `main.c`

---

## 5.3 行为变更的 API

### 5.3.1 Socket 创建函数

#### pa_socket_server_new_unix

**原生行为**：
- 调用 `socket()` 创建新 socket
- 调用 `bind()` 绑定到文件路径
- 调用 `chmod()` 设置权限
- 调用 `listen()` 开始监听

**OHOS 行为**：
- 检查路径是否为 `/dev/unix/socket/native`
- 如果是，调用 `GetControlSocket("native")` 获取预创建 socket
- 跳过 `bind()` 和 `chmod()`
- 仅调用 `listen()`

**代码对比**：

```c
// 原生实现
fd = socket(PF_UNIX, SOCK_STREAM, 0);
bind(fd, ...);
chmod(filename, 0777);
listen(fd, 5);

// OHOS 实现
if (!strcmp(filename, "/dev/unix/socket/native")) {
    fd = GetControlSocket("native");  // 从 init 获取
} else {
    fd = socket(PF_UNIX, SOCK_STREAM, 0);
    bind(fd, ...);
    chmod(filename, 0777);
}
listen(fd, 5);
```

### 5.3.2 日志输出函数

#### pa_log_*, pa_log_level_meta

**原生行为**：
- 输出到 syslog
- 输出到 journald (systemd)
- 输出到 stderr

**OHOS 行为**：
- 通过 `audio_log.h` 重定向到 HiLog
- LOG_DOMAIN: 0xD002B88
- LOG_TAG: "PulseAudio"

**影响**：
- 日志查看命令变化：从 `journalctl -u pulseaudio` 变为 `hilog | grep PulseAudio`
- 日志格式变化：符合 HiLog 格式规范

### 5.3.3 模块加载相关函数

#### pa_ltdl_init, pa_ltdl_done

**原生行为**：
- 初始化 libltdl 动态加载库
- 支持运行时加载 .so 模块

**OHOS 行为**：
- 被 `HAVE_NO_OHOS` 宏包围，不执行
- 模块静态链接

**影响**：
- 不支持运行时加载新模块
- 所有模块必须在编译时确定
- 配置中的模块加载命令可能无效

### 5.3.4 进程优先级函数

#### pa_raise_priority

**原生行为**：
- 提升进程调度优先级
- 设置 nice 值
- 请求实时调度

**OHOS 行为**：
- 被 `HAVE_NO_OHOS` 宏包围，不执行
- 依赖系统调度策略

**影响**：
- 音频线程优先级由系统统一管理
- 可能需要通过其他机制保证实时性

---

## 5.4 废弃或禁用的功能

### 5.4.1 动态模块加载

**状态**：禁用

**原因**：
- 安全考虑
- 系统简化
- 静态链接更易管理

**影响**：
- `module-detect` 等功能不可用
- 所有模块必须在 `BUILD.gn` 中显式链接

### 5.4.2 某些系统调用

#### close_allv

**状态**：条件禁用

**代码**：
```c
#ifdef HAVE_NO_OHOS
    pa_close_allv(passed_fds);
#endif
```

**原因**：可能与 OHOS 文件描述符管理冲突

### 5.4.3 文件权限掩码

#### umask(0077)

**状态**：禁用

**代码**：
```c
#ifdef HAVE_NO_OHOS
    umask(0077);
#endif
```

**原因**：OHOS 使用不同的权限模型

**替代方案**：
- 显式调用 `change_permission()` 设置权限
- 在 `ohos_pa_main.c` 中完成权限设置

---

## 5.5 数据类型和常量差异

### 5.5.1 路径常量

| 常量 | 原生值 | OHOS 值 | 说明 |
|-----|-------|--------|------|
| 运行时目录 | `~/.pulse` | `/data/data/.pulse_dir/runtime` | 系统路径 |
| 状态目录 | `~/.pulse` | `/data/data/.pulse_dir/state` | 系统路径 |
| 配置目录 | `/etc/pulse` | `/system/etc/pulse` | 系统分区 |

### 5.5.2 Socket 路径

| 常量 | 值 |
|-----|-----|
| `OHOS_SOCKET_PATH` | `/dev/unix/socket/native` |
| `OHOS_SOCKET_NAME` | `native` |

---

## 5.6 兼容性说明

### 与原生 PulseAudio 客户端的兼容性

| 方面 | 兼容性 | 说明 |
|-----|--------|------|
| 基本协议 | 兼容 | PA_COMMAND_* 基本命令 |
| 新增命令 | 不兼容 | `PA_COMMAND_UNDERFLOW_OHOS` 不被原生识别 |
| Socket 连接 | 部分兼容 | 原生客户端需使用 OHOS socket 路径 |
| 认证 | 兼容 | Cookie 认证机制相同 |

### 版本兼容性建议

1. **客户端开发**：
   - 优先使用标准 PulseAudio API
   - 如需使用 OHOS 特性，添加版本检查
   - 处理 `PA_COMMAND_UNDERFLOW_OHOS` 命令

2. **服务端升级**：
   - 保持 `PA_COMMAND_UNDERFLOW_OHOS` 命令 ID 不变
   - 确保新增命令不会与上游冲突
   - 维护 HAVE_NO_OHOS 宏的行为一致性

---

## 5.7 迁移指南

### 从原生 PulseAudio 迁移

1. **日志系统迁移**
   ```c
   // 原生
   pa_log_info("Message");
   
   // OHOS
   AUDIO_INFO_LOG("Message");
   ```

2. **Socket 路径迁移**
   ```c
   // 原生
   const char *socket_path = getenv("PULSE_RUNTIME_PATH");
   
   // OHOS
   const char *socket_path = "/dev/unix/socket/native";
   ```

3. **错误处理迁移**
   ```c
   // 添加 OHOS 错误码检查
   if (result == AUDIO_PERMISSION_DENIED) {
       // 处理权限错误
   }
   ```
