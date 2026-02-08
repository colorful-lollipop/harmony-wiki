# 关键调用链

## 目的

本文档描述 HiView Lite 的关键函数调用链，帮助理解执行流程。

## 适用范围

本文档适用于：
- 需要理解执行流程的开发者
- 需要调试的维护者
- 需要优化的性能分析师

## 相关跳转

- [架构说明](03_Architecture.md) - 了解组件关系
- [内部 API](05_Internal_API.md) - 了解模块接口

---

## 初始化调用链

### 完整初始化流程

```
系统启动
  │
  ├─> CORE_INIT 阶段（优先级 0）
  │     │
  │     └─> HiviewConfigInit() [hiview_config.c:29]
  │           │
  │           ├─> 初始化 g_hiviewConfig
  │           │     ├─> outputOption = OUTPUT_OPTION
  │           │     ├─> level = OUTPUT_LEVEL
  │           │     ├─> logSwitch = HILOG_LITE_SWITCH
  │           │     ├─> eventSwitch = HIEVENT_LITE_SWITCH
  │           │     ├─> dumpSwitch = DUMP_LITE_SWITCH
  │           │     ├─> logOutputModule = LOG_OUTPUT_MODULE
  │           │     └─> writeFailureCount = 0
  │           │
  │           └─> 完成
  │
  └─> SYS_SERVICE_INIT 阶段
        │
        └─> Init() [hiview_service.c:45]
              │
              ├─> SAMGR_GetInstance()->RegisterService(&g_hiviewService)
              │     └─> 服务注册到 SAMGR Lite
              │
              ├─> SAMGR_GetInstance()->RegisterDefaultFeatureApi(HIVIEW_SERVICE, GET_IUNKNOWN(g_hiviewService))
              │     └─> 注册 Default Feature API
              │
              ├─> GetTaskConfig() [hiview_service.c:88]
              │     └─> 返回 { LEVEL_LOW, HIVIEW_STACK_PRIO, HIVIEW_STACK_SIZE, 10, SINGLE_TASK }
              │
              └─> InitHiviewComponent() [hiview_service.c:107]
                    │
                    └─> 遍历 g_hiviewInitFuncList[]
                          │
                          ├─> [0] g_hiviewInitFuncList[0]()
                          │     └─> dump_init_func()
                          │
                          ├─> [1] g_hiviewInitFuncList[1]()
                          │     └─> log_init_func()
                          │
                          ├─> [2] g_hiviewInitFuncList[2]()
                          │     └─> log_limit_init_func()
                          │
                          └─> [3] g_hiviewInitFuncList[3]()
                                └─> event_init_func()
                    │
                    └─> 完成
              │
              └─> Initialize() [hiview_service.c:59]
                    │
                    ├─> 设置 identity
                    └─> g_hiviewConfig.hiviewInited = TRUE
```

**证据来源**：
- hiview_config.c:37 - `CORE_INIT_PRI(HiviewConfigInit, 0)`
- hiview_service.c:51 - `SYS_SERVICE_INIT(Init)`
- hiview_service.c:47-48 - 服务注册
- hiview_service.c:107-115 - `InitHiviewComponent`
- hiview_service.c:69 - `g_hiviewConfig.hiviewInited = TRUE`

---

## 消息传递调用链

### 发送消息流程

```
外部模块
  │
  └─> HiviewSendMessage("hiview", msgId, msgValue) [hiview_service.c:127]
        │
        ├─> SAMGR_GetInstance()->GetDefaultFeatureApi("hiview")
        │     │
        │     └─> 返回 IUnknown* api
        │
        ├─> api->QueryInterface(api, 0, (void **)&hiviewInterface)
        │     │
        │     └─> 返回 HiviewInterface* hiviewInterface
        │
        └─> hiviewInterface->Output((IUnknown *)hiviewInterface, msgId, msgValue) [hiview_service.c:95]
              │
              ├─> 构造 Request { msgId, msgValue, data=NULL, len=0 }
              │
              └─> SAMGR_SendRequest(&(service->identity), &request, NULL)
                    │
                    └─> MessageHandle(Service *service, Request *request) [hiview_service.c:75]
                          │
                          ├─> 检查 request->msgId < HIVIEW_MSG_MAX
                          │
                          ├─> 查找 g_hiviewMsgHandleList[request->msgId]
                          │     │
                          │     └─> 返回 HiviewMsgHandle func
                          │
                          └─> func(request)
                                │
                                └─> 执行注册的消息处理函数
```

**证据来源**：
- hiview_service.c:127-139 - `HiviewSendMessage` 实现
- hiview_service.c:95-105 - `Output` 实现
- hiview_service.c:75-86 - `MessageHandle` 实现

---

## 缓存写入调用链

### 写入缓存流程

```
写入方
  │
  └─> WriteToCache(cache, data, wLen) [hiview_cache.c:58]
        │
        ├─> 检查 cache != NULL
        ├─> 检查 data != NULL
        └─> 检查 cache->buffer != NULL
              │
              ├─> HIVIEW_IntLock() [hiview_util.c:118]
              │     └─> 返回 intSave
              │
              ├─> 检查 cache->size >= wLen + cache->usedSize
              │
              │     ├─> 否：HIVIEW_IntRestore(intSave)，返回 -1
              │
              │     └─> 是：继续
              │
              ├─> 检查 cache->wCursor + wLen <= cache->size
              │
              │     ├─> 否（需要回绕）：
              │     │     │
              │     │     ├─> firstLen = cache->size - cache->wCursor
              │     │     │
              │     │     ├─> memcpy_s(cache->buffer + wCursor, firstLen, data, firstLen)
              │     │     ├─> wCursor += firstLen
              │     │     ├─> usedSize += firstLen
              │     │     ├─> wCursor = 0
              │     │     ├─> secondLen = wLen - firstLen
              │     │     ├─> memcpy_s(cache->buffer + wCursor, secondLen, data + firstLen, secondLen)
              │     │     ├─> wCursor += secondLen
              │     │     └─> usedSize += secondLen
              │     │
              │     └─> HIVIEW_IntRestore(intSave)
              │
              │     └─> is（不需要回绕）：
              │           │
              │           └─> memcpy_s(cache->buffer + wCursor, wLen, data, wLen)
              │                 ├─> wCursor += wLen
              │                 ├─> usedSize += wLen
              │                 └─> HIVIEW_IntRestore(intSave)
              │
              └─> 返回 wLen
```

**证据来源**：
- hiview_cache.c:58-107 - `WriteToCache` 实现
- hiview_util.c:118-121 - `HIVIEW_IntLock` 实现

---

## 文件写入调用链（含文件满处理）

### 写入文件流程

```
写入方
  │
  └─> WriteToFile(fp, data, len) [hiview_file.c:151]
        │
        ├─> 检查 fp != NULL
        ├─> 检查 fp->fhandle >= 0
        └─> 检查 len != 0
              │
              ├─> 检查 h->wCursor + len <= h->size
              │
              │     ├─> 否（文件满）：
              │     │     │
              │     │     └─> ProcFile(fp, fp->outPath, HIVIEW_FILE_RENAME) [hiview_file.c:227]
              │     │           │
              │     │           ├─> HIVIEW_MutexLockOrWait(fp->mutex, OUT_PATH_WAIT_TIMEOUT) [hiview_util.c:102]
              │     │           │     └─> 返回 0（成功）或 -1（超时）
              │     │           │
              │     │           ├─> HIVIEW_FileClose(fp->fhandle) [hiview_util.c:185]
              │     │           │     │
              │     │           │     └─> 关闭文件句柄
              │     │           │
              │     │           ├─> HIVIEW_FileMove(fp->path, dest) [hiview_util.c:284]
              │     │           │     │
              │     │           │     └─> 移动文件到目标路径
              │     │           │
              │     │           ├─> InitHiviewFile(fp, type, size) [hiview_file.c:40]
              │     │           │     │
              │     │           │     ├─> HIVIEW_FileOpen(fp->path) [hiview_util.c:176]
              │     │           │     │     ├─> 打开文件
              │     │           │     │     └─> 返回文件句柄
              │     │           │     │
              │     │           │     ├─> ReadFileHeader(fp) 或 WriteFileHeader(fp)
              │     │           │     │     │
              │     │           │     │     └─> 初始化文件头
              │     │           │     │
              │     │           │     ├─> 调用 pFunc(fp->outPath, type, HIVIEW_FILE_FULL)
              │     │           │     │     │
              │     │           │     │     └─> 执行文件监视器回调
              │     │           │     │
              │     │           │     └─> HIVIEW_MutexUnlock(fp->mutex) [hiview_util.c:110]
              │     │           │           │
              │     │           │           └─> 解锁
              │     │           │
              │     │           └─> 返回 0
              │     │
              │     └─> HIVIEW_FileSeek(fp->fhandle, h->wCursor, HIVIEW_SEEK_SET) [hiview_util.c:209]
              │           │
              │           └─> 定位到写游标
              │
              ├─> HIVIEW_FileWrite(fp->fhandle, data, len) [hiview_util.c:201]
              │     │
              │     └─> 写入数据
              │
              ├─> h->wCursor += len
              ├─> 返回 len
              │
              └─> 是（文件未满）：
                    │
                    ├─> HIVIEW_FileSeek(fp->fhandle, h->wCursor, HIVIEW_SEEK_SET) [hiview_util.c:209]
                    │     └─> 定位到写游标
                    │
                    ├─> HIVIEW_FileWrite(fp->fhandle, data, len) [hiview_util.c:201]
                    │     └─> 写入数据
                    │
                    ├─> h->wCursor += len
                    └─> 返回 len
```

**证据来源**：
- hiview_file.c:151-173 - `WriteToFile` 实现
- hiview_file.c:159-163 - 文件满检测
- hiview_file.c:227-274 - `ProcFile` 实现

---

## Hook 机制调用链

### Hook 初始化流程

```
调用方
  │
  └─> HIVIEW_InitHook(&hooks) [hiview_util.c:148]
        │
        ├─> 检查 hooks != NULL
        │
        │     ├─> 是：使用自定义函数
        │     │     │
        │     │     ├─> hiview_open = hooks->open_fn ? hooks->open_fn : open
        │     │     ├─> hiview_close = hooks->close_fn ? hooks->close_fn : close
        │     │     ├─> hiview_read = hooks->read_fn ? hooks->read_fn : read
        │     │     ├─> hiview_write = hooks->write_fn ? hooks->write_fn : write
        │     │     ├─> hiview_lseek = hooks->lseek_fn ? hooks->lseek_fn : lseek
        │     │     ├─> hiview_fsync = hooks->fsync_fn ? hooks->fsync_fn : fsync
        │     │     ├─> hiview_unlink = hooks->unlink_fn ? hooks->unlink_fn : unlink
        │     │     ├─> hiview_rename = hooks->rename_fn
        │     │     ├─> hiview_get_time = hooks->hiview_get_time_fn ? hooks->hiview_get_time_fn : HIVIEW_GetCurrentTimeDef
        │     │     └─> hiview_uart_print = hooks->hiview_uart_print_fn ? hooks->hiview_uart_print_fn : HIVIEW_UartPrintDef
        │     │
        │     └─> 完成
        │
        └─> 否：重置为默认函数
              │
              ├─> hiview_open = open
              ├─> hiview_close = close
              ├─> hiview_read = read
              ├─> hiview_write = write
              ├─> hiview_lseek = lseek
              ├─> hiview_fsync = fsync
              ├─> hiview_unlink = unlink
              ├─> hiview_rename = NULL
              ├─> hiview_get_time = HIVIEW_GetCurrentTimeDef
              └─> hiview_uart_print = HIVIEW_UartPrintDef
```

**证据来源**：
- hiview_util.c:148-174 - `HIVIEW_InitHook` 实现
- hiview_util.h:52-74 - `HIVIEW_Hooks` 结构体

---

## 文件操作调用链

### 文件封装流程

```
调用方
  │
  └─> HIVIEW_FileOpen(path) [hiview_util.c:176]
        │
        └─> hiview_open(path, O_RDWR | O_CREAT, 0)
              │
              └─> 返回文件句柄或 -1

└─> HIVIEW_FileRead(handle, buf, len) [hiview_util.c:193]
        │
        └─> hiview_read(handle, (char *)buf, len)
              │
              └─> 返回读取字节数或 -1

└─> HIVIEW_FileWrite(handle, buf, len) [hiview_util.c:201]
        │
        └─> hiview_write(handle, (const char *)buf, len)
              │
              └─> 返回写入字节数或 -1

└─> HIVIEW_FileClose(handle) [hiview_util.c:185]
        │
        └─> hiview_close(handle)
              │
              └─> 返回 0 或 -1
```

**证据来源**：
- hiview_util.c:176-231 - 文件操作封装实现
- hiview_util.h:93-102 - 文件操作函数声明

---

## 关键结论

1. **初始化分阶段** - CORE_INIT 和 SYS_SERVICE_INIT 两个阶段清晰。
2. **消息驱动** - 通过 SAMGR Lite 进行消息传递，调用链清晰。
3. **缓存环形写入** - 处理回绕逻辑，保证数据连续性。
4. **文件满处理** - 自动触发文件重命名和回调。
5. **Hook 可替换** - 所有文件操作和系统函数都可被替换。

---

*最后更新：2026-02-06*
