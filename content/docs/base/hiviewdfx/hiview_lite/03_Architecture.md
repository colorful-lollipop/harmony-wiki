# 架构说明

## 目的

本文档描述 HiView Lite 的架构，包括组件图、数据流、线程模型和关键时序。

## 适用范围

本文档适用于：
- 需要理解系统架构的开发者
- 需要分析性能的维护者
- 需要扩展功能的集成方

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目定位
- [目录结构](02_Directory_Structure.md) - 了解模块职责
- [内部 API](05_Internal_API.md) - 了解模块接口

---

## 组件图

### 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (外部模块)                   │
│           hilog_lite, hievent_lite, 等                │
└───────────────────────┬─────────────────────────────────┘
                        │
                        │ 注册初始化/消息处理函数
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                   服务层 (hiview_service)              │
│  ┌─────────────────────────────────────────────────┐    │
│  │  HiviewService (SAMGR Lite Service)       │    │
│  │  - Initialize                              │    │
│  │  - MessageHandle                           │    │
│  │  - Output                                 │    │
│  └─────────────────┬───────────────────────┬───┘    │
│                    │                       │         │
│  ┌─────────────────┘                       │         │
│  │                                         │         │
│  │ g_hiviewInitFuncList[]                │         │
│  │ [0]: dump_init_func                  │         │
│  │ [1]: log_init_func                   │         │
│  │ [2]: log_limit_init_func             │         │
│  │ [3]: event_init_func                 │         │
│  │                                         │         │
│  │ g_hiviewMsgHandleList[]                 │         │
│  │ [0]: log_flow_handle                  │         │
│  │ [1]: log_text_file_handle            │         │
│  │ [2]: log_bin_file_handle             │         │
│  │ [3]: event_flow_handle              │         │
│  │ [4]: event_bin_file_handle          │         │
│  └─────────────────┬─────────────────────┘         │
└────────────────────┼───────────────────────────────────┘
                   │
                   │ 依赖
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                   配置层 (hiview_config)              │
│  ┌─────────────────────────────────────────────────┐    │
│  │  HiviewConfig (全局配置)                  │    │
│  │  - outputOption                          │    │
│  │  - hiviewInited                          │    │
│  │  - level                                 │    │
│  │  - logSwitch / eventSwitch / dumpSwitch    │    │
│  │  - logOutputModule                       │    │
│  └─────────────────────────────────────────────────┘    │
└────────────────────┬───────────────────────────────────┘
                   │
                   │ 依赖
         ┌─────────┴─────────┐
         │                   │
         ▼                   ▼
┌──────────────────┐  ┌──────────────────┐
│  缓存层        │  │  文件层         │
│(hiview_cache)  │  │ (hiview_file)   │
│                │  │                 │
│  循环缓冲区      │  │  循环文件        │
│  - 缓存日志      │  │  - 文件头        │
│  - 缓存事件      │  │  - 文件满检测    │
│  - 缓存 Dump     │  │  - 文件重命名    │
└────────┬───────┘  └────────┬────────┘
         │                   │
         └─────────┬─────────┘
                   │
                   │ 依赖
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                  工具层 (hiview_util)                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │  内存管理 (malloc/free)                     │    │
│  │  互斥锁 (osMutexNew/Acquire/Release)      │    │
│  │  中断锁 (LOS_IntLock/Restore)           │    │
│  │  文件系统封装 (open/close/read/write)     │    │
│  │  Hook 机制 (可覆盖)                       │    │
│  └─────────────────────────────────────────────────┘    │
└────────────────────┬───────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    系统层                             │
│  ┌─────────────────────────────────────────────────┐    │
│  │  SAMGR Lite (服务管理器)                   │    │
│  │  LiteOS-M (操作系统内核)                    │    │
│  │  POSIX (文件系统接口)                      │    │
│  │  CMSIS-OS (RTOS 接口)                     │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 组件职责

| 层级 | 组件 | 职责 |
|------|------|------|
| 应用层 | 外部模块 | 注册初始化函数，调用消息接口 |
| 服务层 | hiview_service | 管理服务生命周期，协调各组件 |
| 配置层 | hiview_config | 管理全局配置，控制功能开关 |
| 缓存层 | hiview_cache | 提供循环缓冲区，减少文件 I/O |
| 文件层 | hiview_file | 管理循环文件，持久化数据 |
| 工具层 | hiview_util | 封装底层接口，提供 Hook 机制 |
| 系统层 | SAMGR/LiteOS-M | 提供基础服务 |

---

## 数据流

### 1. 初始化流程

```mermaid
sequenceDiagram
    participant CORE_INIT as CORE_INIT
    participant HiviewConfig as HiviewConfigInit
    participant SYS_SERVICE as SYS_SERVICE_INIT(Init)
    participant SAMGR as SAMGR Lite
    participant HiviewService as HiviewService
    participant InitComp as InitHiviewComponent
    participant External as 外部模块(hilog/hievent)

    CORE_INIT->>HiviewConfig: 1. CORE_INIT 宏触发
    HiviewConfig->>HiviewConfig: 2. 调用 HiviewConfigInit
    HiviewConfig->>HiviewConfig: 3. 初始化 g_hiviewConfig
    HiviewConfig-->>CORE_INIT: 4. 完成

    Note over HiviewService: 系统服务初始化阶段

    SYS_SERVICE->>HiviewService: 5. SYS_SERVICE_INIT 宏触发
    HiviewService->>SAMGR: 6. RegisterService
    SAMGR-->>HiviewService: 7. 注册成功
    HiviewService->>SAMGR: 8. RegisterDefaultFeatureApi
    SAMGR-->>HiviewService: 9. 注册成功
    HiviewService->>InitComp: 10. InitHiviewComponent
    InitComp->>External: 11. 调用注册的初始化函数
    External-->>InitComp: 12. 初始化完成
    InitComp-->>HiviewService: 13. 所有组件初始化完成
    HiviewService->>HiviewConfig: 14. 设置 hiviewInited = TRUE
    HiviewService-->>SYS_SERVICE: 15. 服务启动完成
```

**说明**：
1. **CORE_INIT 阶段**：`CORE_INIT_PRI(HiviewConfigInit, 0)` 触发配置初始化，此时内存管理和文件系统未完全启动。
2. **SYS_SERVICE_INIT 阶段**：`SYS_SERVICE_INIT(Init)` 触发服务初始化，注册服务到 SAMGR Lite，并调用所有注册的组件初始化函数。

**证据来源**：
- hiview_config.c:37 - `CORE_INIT_PRI(HiviewConfigInit, 0)`
- hiview_service.c:51 - `SYS_SERVICE_INIT(Init)`
- hiview_service.c:47-48 - 注册服务和 Feature API
- hiview_service.c:107-115 - `InitHiviewComponent`

### 2. 组件注册流程

```mermaid
sequenceDiagram
    participant External as 外部模块(hilog/hievent)
    participant RegInit as HiviewRegisterInitFunc
    participant InitList as g_hiviewInitFuncList[]
    participant RegMsg as HiviewRegisterMsgHandle
    participant MsgList as g_hiviewMsgHandleList[]
    participant InitComp as InitHiviewComponent
    participant MsgHandle as MessageHandle

    External->>RegInit: 1. 调用 HiviewRegisterInitFunc
    RegInit->>InitList: 2. 保存初始化函数到数组
    RegInit-->>External: 3. 注册成功

    External->>RegMsg: 4. 调用 HiviewRegisterMsgHandle
    RegMsg->>MsgList: 5. 保存消息处理函数到数组
    RegMsg-->>External: 6. 注册成功

    Note over InitComp: 服务启动时

    InitComp->>InitList: 7. 遍历初始化函数数组
    loop 对每个初始化函数
        InitComp->>External: 8. 调用初始化函数
        External-->>InitComp: 9. 初始化完成
    end

    Note over MsgHandle: 收到消息时

    MsgHandle->>MsgList: 10. 根据 msgId 查找处理函数
    MsgList->>External: 11. 调用消息处理函数
    External-->>MsgHandle: 12. 处理完成
```

**说明**：
1. 外部模块（如 hilog_lite、hievent_lite）调用 `HiviewRegisterInitFunc` 注册初始化函数。
2. 外部模块调用 `HiviewRegisterMsgHandle` 注册消息处理函数。
3. 服务启动时，`InitHiviewComponent` 遍历并调用所有注册的初始化函数。
4. 收到消息时，`MessageHandle` 根据 msgId 查找并调用对应的处理函数。

**证据来源**：
- hiview_service.h:59-60 - 函数指针类型定义
- hiview_service.c:117-120 - `HiviewRegisterInitFunc` 实现
- hiview_service.c:122-125 - `HiviewRegisterMsgHandle` 实现
- hiview_service.c:107-115 - `InitHiviewComponent` 实现
- hiview_service.c:75-86 - `MessageHandle` 实现

### 3. 消息传递流程

```mermaid
sequenceDiagram
    participant Sender as 发送方
    participant SendMsg as HiviewSendMessage
    participant SAMGR as SAMGR Lite
    participant GetAPI as GetDefaultFeatureApi
    participant HiviewInterface as HiviewInterface
    participant Output as Output(IUnknown)
    participant SAMGR_Send as SAMGR_SendRequest
    participant Identity as Service Identity
    participant MsgHandle as MessageHandle
    participant MsgList as g_hiviewMsgHandleList[]
    participant Handler as 消息处理函数

    Sender->>SendMsg: 1. 调用 HiviewSendMessage
    SendMsg->>SAMGR: 2. GetDefaultFeatureApi(srvName)
    SAMGR-->>SendMsg: 3. 返回 Default Feature API
    SendMsg->>HiviewInterface: 4. QueryInterface(0, &hiviewInterface)
    HiviewInterface-->>SendMsg: 5. 返回 HiviewInterface
    SendMsg->>Output: 6. 调用 Output(msgId, msgValue)
    Output->>SAMGR_Send: 7. SAMGR_SendRequest(&identity, &request, NULL)
    SAMGR_Send->>MsgHandle: 8. MessageHandle 收到请求
    MsgHandle->>MsgList: 9. 根据 msgId 查找处理函数
    MsgList->>Handler: 10. 调用处理函数(request)
    Handler-->>MsgHandle: 11. 处理完成
    MsgHandle-->>SAMGR_Send: 12. 返回结果
    SendMsg-->>Sender: 13. 发送完成
```

**说明**：
1. 发送方调用 `HiviewSendMessage(srvName, msgId, msgValue)`。
2. 通过 SAMGR 获取服务的 Default Feature API。
3. 调用 Output 接口发送消息。
4. SAMGR 将请求发送到服务的 `MessageHandle`。
5. `MessageHandle` 根据 msgId 查找并调用对应的处理函数。

**证据来源**：
- hiview_service.c:127-139 - `HiviewSendMessage` 实现
- hiview_service.c:95-105 - `Output` 实现
- hiview_service.c:75-86 - `MessageHandle` 实现

### 4. 缓存写入流程

```mermaid
sequenceDiagram
    participant Writer as 写入方
    participant WriteCache as WriteToCache
    participant IntLock as HIVIEW_IntLock
    participant Cache as HiviewCache
    participant IntRestore as HIVIEW_IntRestore

    Writer->>WriteCache: 1. 调用 WriteToCache(data, len)
    WriteCache->>Cache: 2. 检查 buffer、data、size
    WriteCache->>IntLock: 3. HIVIEW_IntLock()
    IntLock-->>WriteCache: 4. 返回中断状态

    alt 缓存空间充足
        WriteCache->>Cache: 5. 检查: size >= len + usedSize
        alt 不需要回绕
            WriteCache->>Cache: 6. memcpy_s(buffer + wCursor, len, data, len)
            WriteCache->>Cache: 7. wCursor += len
            WriteCache->>Cache: 8. usedSize += len
        else 需要回绕
            WriteCache->>Cache: 9. firstLen = size - wCursor
            WriteCache->>Cache: 10. memcpy_s(buffer + wCursor, firstLen, data, firstLen)
            WriteCache->>Cache: 11. wCursor = 0
            WriteCache->>Cache: 12. secondLen = len - firstLen
            WriteCache->>Cache: 13. memcpy_s(buffer + wCursor, secondLen, data + firstLen, secondLen)
            WriteCache->>Cache: 14. wCursor += secondLen
            WriteCache->>Cache: 15. usedSize += len
        end
    else 缓存空间不足
        WriteCache->>IntRestore: 16. 恢复中断
        WriteCache-->>Writer: 17. 返回 -1
    end

    WriteCache->>IntRestore: 18. HIVIEW_IntRestore(intSave)
    WriteCache-->>Writer: 19. 返回写入长度
```

**说明**：
1. 写入前检查参数有效性。
2. 关中断，保证写入操作的原子性。
3. 检查缓存空间是否充足。
4. 如果需要回绕（wCursor + len > size），分两次写入。
5. 使用 `memcpy_s` 安全复制数据。
6. 更新游标和已用大小。
7. 恢复中断。

**证据来源**：
- hiview_cache.c:58-107 - `WriteToCache` 实现
- hiview_cache.c:66 - `HIVIEW_IntLock()`
- hiview_cache.c:104 - `HIVIEW_IntRestore(intSave)`

### 5. 文件写入流程（含文件满处理）

```mermaid
sequenceDiagram
    participant Writer as 写入方
    participant WriteFile as WriteToFile
    participant HiviewFile as HiviewFile
    participant FileSeek as HIVIEW_FileSeek
    participant FileWrite as HIVIEW_FileWrite
    participant ProcFile as ProcFile
    participant Mutex as HIVIEW_MutexLock
    participant FileMove as HIVIEW_FileMove
    participant InitFile as InitHiviewFile
    participant Callback as File Watcher

    Writer->>WriteFile: 1. 调用 WriteToFile(data, len)
    WriteFile->>HiviewFile: 2. 检查 handle、len

    alt 文件未满 (wCursor + len <= size)
        WriteFile->>FileSeek: 3. HIVIEW_FileSeek(handle, wCursor, SEEK_SET)
        FileSeek-->>WriteFile: 4. 定位成功
        WriteFile->>FileWrite: 5. HIVIEW_FileWrite(handle, data, len)
        FileWrite-->>WriteFile: 6. 写入成功
        WriteFile->>HiviewFile: 7. wCursor += len
        WriteFile-->>Writer: 8. 返回写入长度
    else 文件已满 (wCursor + len > size)
        WriteFile->>ProcFile: 9. ProcFile(fp, outPath, RENAME)
        ProcFile->>Mutex: 10. HIVIEW_MutexLockOrWait(mutex, timeout)
        Mutex-->>ProcFile: 11. 加锁成功
        ProcFile->>FileMove: 12. HIVIEW_FileMove(path, outPath)
        FileMove-->>ProcFile: 13. 移动成功
        ProcFile->>InitFile: 14. InitHiviewFile(fp, type, size)
        InitFile-->>ProcFile: 15. 初始化新文件成功
        ProcFile->>Callback: 16. 调用 pFunc(outPath, type, FILE_FULL)
        Callback-->>ProcFile: 17. 回调完成
        ProcFile->>Mutex: 18. HIVIEW_MutexUnlock(mutex)
        WriteFile->>FileSeek: 19. HIVIEW_FileSeek(handle, wCursor, SEEK_SET)
        FileSeek-->>WriteFile: 20. 定位成功
        WriteFile->>FileWrite: 21. HIVIEW_FileWrite(handle, data, len)
        FileWrite-->>WriteFile: 22. 写入成功
        WriteFile->>HiviewFile: 23. wCursor += len
        WriteFile-->>Writer: 24. 返回写入长度
    end
```

**说明**：
1. 写入前检查文件句柄和长度。
2. 检测文件是否已满（wCursor + len > size）。
3. 如果文件已满：
   - 调用 `ProcFile` 进行文件重命名。
   - 使用互斥锁保护文件操作。
   - 移动当前文件到输出路径。
   - 初始化新文件。
   - 调用注册的文件监视器回调。
4. 定位到写游标位置。
5. 写入数据。
6. 更新写游标。

**证据来源**：
- hiview_file.c:151-173 - `WriteToFile` 实现
- hiview_file.c:159-163 - 文件满检测
- hiview_file.c:227-274 - `ProcFile` 实现

---

## 线程模型

### 服务线程

HiView Lite 服务运行在独立的线程中，由 SAMGR Lite 管理。

**线程配置**（来自 `GetTaskConfig`）：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 线程优先级 | HIVIEW_STACK_PRIO | 默认 24（可配置） |
| 栈大小 | HIVIEW_STACK_SIZE | 默认 4096 字节（可配置） |
| 队列深度 | 10 | 消息队列深度 |
| 任务类型 | SINGLE_TASK | 单任务 |

**证据来源**：
- hiview_service.c:88-93 - `GetTaskConfig` 实现
- BUILD.gn:23-24 - 栈大小和优先级配置

### 线程安全

#### 中断锁

**使用场景**：缓存写入操作

**实现**：
- `HIVIEW_IntLock()` - 关中断
- `HIVIEW_IntRestore(intSave)` - 恢复中断

**证据来源**：
- hiview_cache.c:66 - `uint32 intSave = HIVIEW_IntLock();`
- hiview_cache.c:104 - `HIVIEW_IntRestore(intSave);`

#### 互斥锁

**使用场景**：文件处理操作（复制/重命名）

**实现**：
- `HIVIEW_MutexInit()` - 创建互斥锁
- `HIVIEW_MutexLock()` - 加锁（等待）
- `HIVIEW_MutexLockOrWait(mutex, timeout)` - 加锁（带超时）
- `HIVIEW_MutexUnlock()` - 解锁

**证据来源**：
- hiview_file.c:233 - `HIVIEW_MutexLockOrWait(fp->mutex, OUT_PATH_WAIT_TIMEOUT)`
- hiview_file.c:272 - `HIVIEW_MutexUnlock(fp->mutex)`

---

## 关键时序

### 启动时序

```mermaid
timeline
    title HiView Lite 启动时序
    section CORE_INIT 阶段
        系统启动 : 触发 CORE_INIT 优先级 0
        HiviewConfigInit : 初始化 g_hiviewConfig
                     : 设置 hiviewInited = FALSE
                     : 设置 logOutputModule = LOG_OUTPUT_MODULE
                     : 设置 writeFailureCount = 0
        配置完成 : 内存/文件系统未完全启动
    section SYS_SERVICE_INIT 阶段
        系统启动 : 触发 SYS_SERVICE_INIT
        Init : 注册服务到 SAMGR Lite
             : 注册 Default Feature API
             : 调用 InitHiviewComponent
        InitHiviewComponent : 遍历 g_hiviewInitFuncList[]
                          : 调用所有注册的初始化函数
                          : dump_init_func()
                          : log_init_func()
                          : log_limit_init_func()
                          : event_init_func()
        Initialize : 设置服务 Identity
                 : 设置 g_hiviewConfig.hiviewInited = TRUE
        服务启动完成 : 所有组件已初始化
    section 运行时
        收到消息 : MessageHandle 被调用
                 : 根据 msgId 查找处理函数
                 : 调用 g_hiviewMsgHandleList[msgId](request)
        文件写入 : WriteToFile 检测文件满
                 : 触发 ProcFile(RENAM)
                 : 调用 File Watcher 回调
```

**证据来源**：
- hiview_config.c:37 - `CORE_INIT_PRI(HiviewConfigInit, 0)`
- hiview_service.c:51 - `SYS_SERVICE_INIT(Init)`
- hiview_service.c:107-115 - `InitHiviewComponent`
- hiview_service.c:69 - `g_hiviewConfig.hiviewInited = TRUE`
- hiview_file.c:159-163 - 文件满检测

---

## 关键结论

1. **分层架构** - 清晰的分层设计（服务层、配置层、缓存层、文件层、工具层、系统层）。
2. **消息驱动** - 通过 SAMGR Lite 进行消息传递，支持异步通信。
3. **组件注册** - 外部模块可以注册初始化函数和消息处理函数，具有良好的扩展性。
4. **线程安全** - 使用中断锁和互斥锁保证并发安全。
5. **循环缓冲区** - 提供高效的循环缓冲区实现，减少文件 I/O。
6. **循环文件** - 提供循环文件实现，支持文件满检测和自动处理。

---

*最后更新：2026-02-06*
