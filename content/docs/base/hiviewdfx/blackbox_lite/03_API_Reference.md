# 03_API_Reference - 接口参考

> **重要说明**：本模块为 **纯 C 内核态模块**，不提供 N-API / JS 接口。以下为 **内部 C API** 参考。

## 3.1 公共接口（外部调用）

### 3.1.1 BBoxNotifyError - 故障通知

**功能**：通知 blackbox 模块发生故障，触发数据采集和可选的系统重启。

**头文件**：`interfaces/native/kits/blackbox.h:65`

**函数原型**：
```c
int BBoxNotifyError(const char event[EVENT_MAX_LEN],
    const char module[MODULE_MAX_LEN],
    const char errorDesc[ERROR_DESC_MAX_LEN],
    int needSysReset);
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| event | `const char[]` | 是 | 故障事件名称（预定义常量见下文） |
| module | `const char[]` | 是 | 模块名称，需与注册的 ModuleOps.module 匹配 |
| errorDesc | `const char[]` | 是 | 故障描述信息 |
| needSysReset | `int` | 是 | 是否重启：0-不重启，1-重启 |

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 参数错误或内存分配失败 |

**调用链**：
```
调用者 (异常中断上下文)
    │
    ▼
BBoxNotifyError()  ◄── blackbox_core.c:252
    │
    ├──► FormatErrorInfo()     ◄── blackbox_core.c:71
    │       格式化 ErrorInfo
    │
    ├──► 遍历 g_opsList        ◄── blackbox_core.c:277
    │       查找匹配的 ModuleOps
    │
    ├──► ops->Dump() (可选)    ◄── blackbox_core.c:292
    │       采集故障数据
    │
    ├──► ops->Reset() (可选)   ◄── blackbox_core.c:297
    │       系统复位前处理
    │
    └──► RebootSystem() (可选) ◄── blackbox_core.c:312
            系统重启
```

**代码示例**：
```c
// 在异常处理中调用
void OsExcHandleEntry(void)
{
    BBoxNotifyError("EVENT_PANIC", "KERNEL", "Unhandled exception", 1);
}
```

### 3.1.2 BBoxRegisterModuleOps - 模块注册

**功能**：向 blackbox 注册模块操作接口。

**头文件**：`interfaces/native/kits/blackbox.h:64`

**函数原型**：
```c
int BBoxRegisterModuleOps(struct ModuleOps *ops);
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ops | `struct ModuleOps*` | 是 | 模块操作接口指针 |

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 参数错误、内存分配失败或模块已注册 |

**实现细节**：`blackbox_core.c:206-250`

- 使用 `malloc` 分配 `struct BBoxOps`（避免栈溢出）
- 使用信号量 `g_opsListSem` 保护链表操作
- 检查模块名是否已注册，防止重复

### 3.1.3 BBoxDriverInit - 驱动初始化

**功能**：驱动初始化接口（当前未在代码中使用）。

**头文件**：`interfaces/native/kits/blackbox.h:69`

**函数原型**：
```c
int BBoxDriverInit(void);
```

**注意**：此函数在代码中声明但未实现调用，可能是预留接口。

## 3.2 适配器接口（平台需实现）

以下接口在 `blackbox_adapter.c` 中定义为 **WEAK 符号**，平台层必须提供实现。

### 3.2.1 SystemModuleDump - 系统 Dump

**功能**：系统异常时进行数据预处理，可存储异常栈等信息。

**头文件**：`blackbox_adapter.h:33`

**函数原型**：
```c
void SystemModuleDump(const char *logDir, struct ErrorInfo *info);
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| logDir | `const char*` | 是 | 日志存储目录 |
| info | `struct ErrorInfo*` | 是 | 故障信息 |

**默认实现**：`blackbox_adapter.c:25-30`
```c
WEAK void SystemModuleDump(const char *logDir, struct ErrorInfo *info)
{
    (void)logDir;
    (void)info;
    BBOX_PRINT_ERR("Please implement the interface according to the platform!\n");
}
```

### 3.2.2 SystemModuleReset - 系统复位

**功能**：系统复位前的清理操作。

**头文件**：`blackbox_adapter.h:34`

**函数原型**：
```c
void SystemModuleReset(struct ErrorInfo *info);
```

**默认实现**：`blackbox_adapter.c:32-36`

### 3.2.3 SystemModuleGetLastLogInfo - 获取日志信息

**功能**：重启后检查是否有待保存的日志。

**头文件**：`blackbox_adapter.h:35`

**函数原型**：
```c
int SystemModuleGetLastLogInfo(struct ErrorInfo *info);
```

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 0 | 无日志需要保存 |
| 非0 | 有日志需要保存 |

**默认实现**：`blackbox_adapter.c:38-43`

### 3.2.4 SystemModuleSaveLastLog - 保存日志

**功能**：将故障日志保存到文件系统。

**头文件**：`blackbox_adapter.h:36`

**函数原型**：
```c
int SystemModuleSaveLastLog(const char *logDir, struct ErrorInfo *info);
```

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| 非0 | 失败 |

**默认实现**：`blackbox_adapter.c:45-51`

### 3.2.5 FullWriteFile - 文件写操作

**功能**：向文件写入数据（覆写或追加模式）。

**头文件**：`blackbox_adapter.h:37`

**函数原型**：
```c
int FullWriteFile(const char *filePath, const char *buf,
    unsigned int bufSize, int isAppend);
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | `const char*` | 是 | 文件路径 |
| buf | `const char*` | 是 | 待写入数据 |
| bufSize | `unsigned int` | 是 | 数据大小 |
| isAppend | `int` | 是 | 0-覆写模式，1-追加模式 |

**默认实现**：`blackbox_adapter.c:53-62`

### 3.2.6 GetFaultLogPath - 获取日志路径

**功能**：返回故障日志存储路径。

**头文件**：`blackbox_adapter.h:38`

**函数原型**：
```c
char *GetFaultLogPath(void);
```

**返回值**：日志存储路径字符串

**默认实现**：`blackbox_adapter.c:64-68`

### 3.2.7 RebootSystem - 系统重启

**功能**：执行系统重启操作。

**头文件**：`blackbox_adapter.h:39`

**函数原型**：
```c
void RebootSystem(void);
```

**默认实现**：`blackbox_adapter.c:70-73`

## 3.3 事件上报接口

### 3.3.1 UploadEventByFile - 通过文件上报

**功能**：通过文件路径上报故障事件。

**头文件**：`blackbox_detector.h:21`

**函数原型**：
```c
int UploadEventByFile(const char *filePath);
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | `const char*` | 是 | 故障信息文件路径 |

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 参数错误 |

**实现**：`blackbox_detector.c:18-25`

**注意**：当前实现返回 0，实际事件上报由 `hiview_lite` 子系统实现。

### 3.3.2 UploadEventByStream - 通过流上报

**功能**：通过数据流上报故障事件。

**头文件**：`blackbox_detector.h:22`

**函数原型**：
```c
int UploadEventByStream(const char *buf, unsigned int bufSize);
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| buf | `const char*` | 是 | 事件数据 |
| bufSize | `unsigned int` | 是 | 数据大小 |

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 参数错误 |

**实现**：`blackbox_detector.c:27-34`

## 3.4 预定义常量

### 3.4.1 模块名

| 常量 | 值 | 说明 |
|------|-----|------|
| MODULE_SYSTEM | `"SYSTEM"` | 系统模块 |

**证据**：`blackbox.h:39`

### 3.4.2 事件类型

| 常量 | 值 | 说明 |
|------|-----|------|
| EVENT_SYSREBOOT | `"SYSREBOOT"` | 系统重启 |
| EVENT_LONGPRESS | `"LONGPRESS"` | 长按事件 |
| EVENT_COMBINATIONKEY | `"COMBINATIONKEY"` | 组合键事件 |
| EVENT_SUBSYSREBOOT | `"SUBSYSREBOOT"` | 子系统重启 |
| EVENT_POWEROFF | `"POWEROFF"` | 关机事件 |
| EVENT_PANIC | `"PANIC"` | 系统崩溃 |
| EVENT_SYS_WATCHDOG | `"SYSWATCHDOG"` | 系统看门狗 |
| EVENT_HUNGTASK | `"HUNGTASK"` | 任务挂起 |
| EVENT_BOOTFAIL | `"BOOTFAIL"` | 启动失败 |

**证据**：`blackbox.h:40-48`

### 3.4.3 缓冲区大小

| 常量 | 值 | 说明 |
|------|-----|------|
| ERROR_INFO_MAX_LEN | 768 | 错误信息最大长度 |
| PATH_MAX_LEN | 256 | 路径最大长度 |
| EVENT_MAX_LEN | 32 | 事件名最大长度 |
| MODULE_MAX_LEN | 32 | 模块名最大长度 |
| ERROR_DESC_MAX_LEN | 512 | 错误描述最大长度 |

**证据**：`blackbox.h:29, 35-38`

## 3.5 调用链总结

### 3.5.1 故障通知调用链

```
外部调用
    │
    ▼
BBoxNotifyError(event, module, errorDesc, needSysReset)
    │
    ├─ [1] 分配 ErrorInfo 内存
    │
    ├─ [2] 调用 FormatErrorInfo() 格式化信息
    │
    ├─ [3] 遍历 g_opsList 查找匹配的 ModuleOps
    │       (信号量保护)
    │
    ├─ [4] 若找到匹配的 ops:
    │   │
    │   ├─ [4.1] 若 ops->Dump 非空:
    │   │       调用 ops->Dump(logDir, info)
    │   │
    │   ├─ [4.2] 若 ops->Reset 非空:
    │   │       调用 ops->Reset(info)
    │   │
    │   └─ [4.3] 若 needSysReset == 1:
    │           调用 RebootSystem()
    │
    └─ [5] 若未找到匹配的 ops 或无 Dump/Reset:
            调用 SaveBasicErrorInfo() 保存基本信息
                │
                └──► FullWriteFile() (适配层)
                    └──► UploadEventByFile() (hiview)
```

### 3.5.2 日志保存调用链（SaveErrorLog 线程）

```
SaveErrorLog() ◄── 开机时创建
    │
    ├─ [1] 等待 LOG_ROOT_DIR 就绪
    │
    ├─ [2] 遍历 g_opsList
    │       (信号量保护)
    │
    ├─ [3] 对每个模块:
    │   │
    │   ├─ [3.1] 调用 ops->GetLastLogInfo(info)
    │   │       返回非0表示有待保存日志
    │   │
    │   └─ [3.2] 若有待保存日志:
    │           调用 ops->SaveLastLog(dirName, info)
    │               │
    │               └──► FullWriteFile() (适配层)
    │
    └─ [4] 调用 UploadEventByFile() 上报事件
            (hiview_lite)
```

---

## 参考文档

- [01_Overview](01_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构说明
- [04_Build_Configuration](04_Build_Configuration.md) - 构建配置
- [05_Security_Review](05_Security_Review.md) - 安全评审
