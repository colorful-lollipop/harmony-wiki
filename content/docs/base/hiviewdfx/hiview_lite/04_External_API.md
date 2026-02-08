# 对外 API

## 目的

本文档描述 HiView Lite 的对外 API，包括导出符号、接口、参数和错误码。

## 适用范围

本文档适用于：
- 需要调用 HiView Lite 的模块
- 需要理解接口定义的集成方

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目定位
- [内部 API](05_Internal_API.md) - 了解模块接口
- [常见问题](09_FAQ.md) - 了解使用问题

---

## 重要说明

**HiView Lite 是一个纯 C 语言模块，没有 N-API（JS API）**。

本项目不提供 JavaScript 绑定，所有 API 都是 C 语言接口，用于模块间通信和组件注册。

---

## 导出符号清单

### 服务注册接口

| 符号名 | 头文件 | 说明 |
|--------|---------|------|
| `HiviewRegisterInitFunc` | hiview_service.h:62 | 注册组件初始化函数 |
| `HiviewRegisterMsgHandle` | hiview_service.h:63 | 注册消息处理函数 |
| `HiviewSendMessage` | hiview_service.h:64 | 发送内部消息 |

### 配置接口

| 符号名 | 头文件 | 说明 |
|--------|---------|------|
| `g_hiviewConfig` | hiview_config.h:97 | 全局配置变量（extern） |
| `HiviewConfigInit` | hiview_config.h:99 | 配置初始化函数（CORE_INIT 自动调用） |

### 缓存接口

| 符号名 | 头文件 | 说明 |
|--------|---------|------|
| `InitHiviewStaticCache` | hiview_cache.h:56 | 使用静态内存初始化缓存 |
| `InitHiviewCache` | hiview_cache.h:65 | 使用动态内存初始化缓存 |
| `WriteToCache` | hiview_cache.h:74 | 写入数据到缓存 |
| `ReadFromCache` | hiview_cache.h:83 | 从缓存读取数据 |
| `PrereadFromCache` | hiview_cache.h:94 | 预读数据（不改变读游标） |
| `DiscardCacheData` | hiview_cache.h:101 | 丢弃所有缓存数据 |
| `DestroyCache` | hiview_cache.h:107 | 销毁缓存并释放内存 |

### 文件接口

| 符号名 | 头文件 | 说明 |
|--------|---------|------|
| `InitHiviewFile` | hiview_file.h:105 | 初始化文件对象 |
| `WriteFileHeader` | hiview_file.h:113 | 写入文件头 |
| `ReadFileHeader` | hiview_file.h:121 | 读取文件头 |
| `WriteToFile` | hiview_file.h:132 | 写入数据到文件 |
| `ReadFromFile` | hiview_file.h:143 | 从文件读取数据 |
| `GetFileUsedSize` | hiview_file.h:151 | 获取文件已用大小 |
| `GetFileFreeSize` | hiview_file.h:159 | 获取文件空闲大小 |
| `CloseHiviewFile` | hiview_file.h:167 | 关闭文件 |
| `ProcFile` | hiview_file.h:177 | 处理文件（复制/重命名） |
| `RegisterFileWatcher` | hiview_file.h:186 | 注册文件监视器 |
| `UnRegisterFileWatcher` | hiview_file.h:195 | 注销文件监视器 |

### 工具接口

| 符号名 | 头文件 | 说明 |
|--------|---------|------|
| `HIVIEW_GetCurrentTime` | hiview_util.h:76 | 获取当前时间（毫秒） |
| `HIVIEW_RtcGetCurrentTime` | hiview_util.h:77 | 获取 RTC 时间 |
| `HIVIEW_MemAlloc` | hiview_util.h:79 | 内存分配 |
| `HIVIEW_MemFree` | hiview_util.h:80 | 内存释放 |
| `HIVIEW_MutexInit` | hiview_util.h:81 | 初始化互斥锁 |
| `HIVIEW_MutexLock` | hiview_util.h:82 | 加锁（等待） |
| `HIVIEW_MutexLockOrWait` | hiview_util.h:83 | 加锁（带超时） |
| `HIVIEW_MutexUnlock` | hiview_util.h:84 | 解锁 |
| `HIVIEW_IntLock` | hiview_util.h:85 | 关中断 |
| `HIVIEW_IntRestore` | hiview_util.h:86 | 恢复中断 |
| `HIVIEW_GetTaskId` | hiview_util.h:87 | 获取任务 ID |
| `HIVIEW_UartPrint` | hiview_util.h:88 | UART 打印 |
| `HIVIEW_Sleep` | hiview_util.h:89 | 延时 |
| `HIVIEW_InitHook` | hiview_util.h:92 | 初始化 Hook 机制 |
| `HIVIEW_FileOpen` | hiview_util.h:93 | 打开文件 |
| `HIVIEW_FileClose` | hiview_util.h:94 | 关闭文件 |
| `HIVIEW_FileRead` | hiview_util.h:95 | 读取文件 |
| `HIVIEW_FileWrite` | hiview_util.h:96 | 写入文件 |
| `HIVIEW_FileSeek` | hiview_util.h:97 | 文件定位 |
| `HIVIEW_FileSize` | hiview_util.h:98 | 获取文件大小 |
| `HIVIEW_FileSync` | hiview_util.h:99 | 同步文件 |
| `HIVIEW_FileUnlink` | hiview_util.h:100 | 删除文件 |
| `HIVIEW_FileCopy` | hiview_util.h:101 | 复制文件 |
| `HIVIEW_FileMove` | hiview_util.h:102 | 移动文件 |

---

## 核心接口详解

### 1. HiviewRegisterInitFunc

**功能**：注册组件初始化函数

**函数原型**：
```c
void HiviewRegisterInitFunc(HiviewComponentType type, HiviewInitFunc func);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| type | HiviewComponentType | 组件类型（DUMP/LOG/LOG_LIMIT/EVENT） |
| func | HiviewInitFunc | 初始化函数指针 |

**返回值**：无

**使用示例**：
```c
// 在组件初始化时调用
static void MyLogInit(void) {
    // 初始化日志组件
    printf("Log component initialized\n");
}

HiviewRegisterInitFunc(HIVIEW_CMP_TYPE_LOG, MyLogInit);
```

**证据来源**：
- hiview_service.h:62 - 函数声明
- hiview_service.c:117-120 - 函数实现

---

### 2. HiviewRegisterMsgHandle

**功能**：注册消息处理函数

**函数原型**：
```c
void HiviewRegisterMsgHandle(HiviewInnerMessage type, HiviewMsgHandle func);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| type | HiviewInnerMessage | 消息类型（LOG_FLOW/TEXT_FILE/BIN_FILE/EVENT_FLOW/EVENT_BIN_FILE） |
| func | HiviewMsgHandle | 消息处理函数指针 |

**返回值**：无

**使用示例**：
```c
// 在组件初始化时调用
static void MyLogTextFileHandle(const Request *request) {
    // 处理日志文本文件消息
    printf("Log text file message received\n");
}

HiviewRegisterMsgHandle(HIVIEW_MSG_OUTPUT_LOG_TEXT_FILE, MyLogTextFileHandle);
```

**证据来源**：
- hiview_service.h:63 - 函数声明
- hiview_service.c:122-125 - 函数实现

---

### 3. HiviewSendMessage

**功能**：发送内部消息

**函数原型**：
```c
void HiviewSendMessage(const char *srvName, int16 msgId, uint16 msgValue);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| srvName | const char * | 服务名称（通常是 "hiview"） |
| msgId | int16 | 消息 ID（HiviewInnerMessage 枚举值） |
| msgValue | uint16 | 消息值（具体含义由 msgId 决定） |

**返回值**：无

**使用示例**：
```c
// 发送日志流消息
HiviewSendMessage("hiview", HIVIEW_MSG_OUTPUT_LOG_FLOW, 0);

// 发送日志文本文件消息
HiviewSendMessage("hiview", HIVIEW_MSG_OUTPUT_LOG_TEXT_FILE, LOG_LEVEL_DEBUG);
```

**证据来源**：
- hiview_service.h:64 - 函数声明
- hiview_service.c:127-139 - 函数实现

---

### 4. InitHiviewCache

**功能**：使用动态内存初始化缓存

**函数原型**：
```c
boolean InitHiviewCache(HiviewCache *cache, HiviewCacheType type, uint16 size);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| cache | HiviewCache * | 缓存对象指针 |
| type | HiviewCacheType | 缓存类型（LOG_CACHE/EVENT_CACHE 等） |
| size | uint16 | 缓存大小（字节） |

**返回值**：
- `TRUE` - 成功
- `FALSE` - 失败（内存分配失败）

**使用示例**：
```c
HiviewCache logCache;
if (InitHiviewCache(&logCache, LOG_CACHE, LOG_STATIC_CACHE_SIZE)) {
    // 缓存初始化成功
    WriteToCache(&logCache, data, len);
} else {
    // 缓存初始化失败
    printf("Failed to init cache\n");
}
```

**证据来源**：
- hiview_cache.h:65 - 函数声明
- hiview_cache.c:38-56 - 函数实现

---

### 5. WriteToFile

**功能**：写入数据到循环文件

**函数原型**：
```c
int32 WriteToFile(HiviewFile *fp, const uint8 *data, uint32 len);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| fp | HiviewFile * | 文件对象指针 |
| data | const uint8 * | 要写入的数据 |
| len | uint32 | 要写入的长度 |

**返回值**：
- 成功：返回写入的字节数
- 失败：返回 0

**注意**：
- 如果文件已满，会自动触发 `ProcFile(RENAM)` 和文件监视器回调
- 避免频繁调用，可能导致 watchdog 超时

**使用示例**：
```c
HiviewFile logFile;
// ... 初始化文件 ...

int32 written = WriteToFile(&logFile, data, len);
if (written != len) {
    printf("Failed to write %d bytes, only %d written\n", len, written);
}
```

**证据来源**：
- hiview_file.h:132 - 函数声明
- hiview_file.c:151-173 - 函数实现

---

### 6. RegisterFileWatcher

**功能**：注册文件监视器（文件满回调）

**函数原型**：
```c
void RegisterFileWatcher(HiviewFile *fp, FileProc func, const char *path);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| fp | HiviewFile * | 文件对象指针 |
| func | FileProc | 文件处理回调函数 |
| path | const char * | 目标文件路径（可为 NULL，使用默认路径） |

**返回值**：无

**回调函数原型**：
```c
typedef void (*FileProc)(const char *path, uint8 type, uint8 event);
```

| 参数 | 说明 |
|------|------|
| path | 文件路径 |
| type | 文件类型（HiviewFileType） |
| event | 事件类型（目前只有 HIVIEW_FILE_FULL） |

**使用示例**：
```c
static void MyFileWatcher(const char *path, uint8 type, uint8 event) {
    if (event == HIVIEW_FILE_FULL) {
        printf("File %s is full\n", path);
        // 可以进行文件上传、清理等操作
    }
}

HiviewFile logFile;
// ... 初始化文件 ...
RegisterFileWatcher(&logFile, MyFileWatcher, "/data/log/output.log");
```

**证据来源**：
- hiview_file.h:186 - 函数声明
- hiview_file.c:291-311 - 函数实现

---

## 错误码

### 返回值约定

| 返回值 | 含义 |
|--------|------|
| TRUE (1) | 成功 |
| FALSE (0) | 失败 |
| 正数 | 成功，返回处理的数据长度或数量 |
| 0 | 无数据或失败 |
| -1 | 失败（参数错误、内存不足等） |
| 负数 | 失败，具体错误码由模块定义 |

### 常见错误

| 错误场景 | 返回值 | 说明 |
|----------|--------|------|
| NULL 指针 | FALSE/-1 | 大多数函数检查 NULL 指针 |
| 内存分配失败 | FALSE | InitHiviewCache、HIVIEW_FileOpen 等 |
| 文件操作失败 | -1 | HIVIEW_FileOpen、HIVIEW_FileWrite 等 |
| 缓存空间不足 | -1 | WriteToCache |
| 文件已满 | 自动处理 | WriteToFile 会触发文件重命名 |

---

## 调用方式

### 通过 SAMGR Lite 调用

HiView Lite 作为 SAMGR Lite 服务，可以通过服务名称 "hiview" 进行调用。

**步骤**：
1. 通过 `SAMGR_GetInstance()->GetDefaultFeatureApi("hiview")` 获取服务 API
2. 调用 `QueryInterface` 获取 HiviewInterface
3. 调用 `Output` 接口发送消息

**示例**：
```c
// 获取服务 API
IUnknown *api = SAMGR_GetInstance()->GetDefaultFeatureApi("hiview");
if (api != NULL) {
    HiviewInterface *hiviewInterface = NULL;
    api->QueryInterface(api, 0, (void **)&hiviewInterface);
    if (hiviewInterface != NULL) {
        // 发送消息
        hiviewInterface->Output((IUnknown *)hiviewInterface,
                              HIVIEW_MSG_OUTPUT_LOG_TEXT_FILE,
                              LOG_LEVEL_INFO);
    }
}
```

**证据来源**：
- hiview_service.c:127-139 - `HiviewSendMessage` 实现

---

## 关键结论

1. **纯 C 接口** - 所有 API 都是 C 语言函数，没有 JS 绑定。
2. **组件注册** - 外部模块可以注册初始化函数和消息处理函数，具有良好的扩展性。
3. **消息驱动** - 通过 SAMGR Lite 进行消息传递，支持异步通信。
4. **循环缓冲区** - 提供高效的循环缓冲区 API。
5. **循环文件** - 提供循环文件 API，支持文件满自动处理。

---

*最后更新：2026-02-06*
