# 目录结构与代码地图

> **文档版本**: 1.0  
> **最后更新**: 2026-02-07  
> **代码版本**: v3.1

## 3.1 顶层目录结构

```
hiview_lite/
├── BUILD.gn                    # GN 构建配置
├── bundle.json                 # 组件元数据
├── LICENSE                     # Apache 2.0 许可证
├── README.md                   # 英文项目说明
├── README_zh.md                # 中文项目说明
├── hiview_service.c/h          # SAMGR 服务实现
├── hiview_config.c/h           # 配置管理
├── hiview_cache.c/h            # 缓存机制
├── hiview_file.c/h             # 文件操作
├── hiview_util.c/h             # 工具函数
└── hiview_def.h                # 公共定义
```

### 目录职责说明

| 目录/文件 | 职责 | 代码行数 |
|-----------|------|----------|
| `BUILD.gn` | GN 构建配置、Feature 开关、编译目标 | 81 行 |
| `bundle.json` | 组件元数据、依赖声明、构建配置 | 50 行 |
| `hiview_service.c/h` | SAMGR 服务实现、消息处理、组件注册 | 213 行 (合计) |
| `hiview_config.c/h` | 配置管理、文件路径、Feature 宏定义 | 147 行 (合计) |
| `hiview_cache.c/h` | 循环缓冲区、缓存读写 | 339 行 (合计) |
| `hiview_file.c/h` | 文件读写、循环文件、文件头管理 | 545 行 (合计) |
| `hiview_util.c/h` | 内存管理、互斥锁、Hook 机制 | 429 行 (合计) |
| `hiview_def.h` | 常量定义、数据结构定义 | 61 行 |

**证据来源**：`NOTES.md:21-39`

---

## 3.2 代码地图

### 核心功能定位

| 功能 | 关键文件 | 关键函数/结构 |
|------|----------|---------------|
| **SAMGR 服务注册** | `hiview_service.c/h` | `Init()`, `GetName()`, `MessageHandle()` |
| **组件初始化调度** | `hiview_service.c/h` | `InitHiviewComponent()`, `HiviewRegisterInitFunc()` |
| **消息路由** | `hiview_service.c/h` | `MessageHandle()`, `HiviewSendMessage()`, `Output()` |
| **配置初始化** | `hiview_config.c/h` | `HiviewConfigInit()`, `g_hiviewConfig` |
| **文件路径配置** | `hiview_config.h` | `HIVIEW_FILE_DIR`, `HIVIEW_FILE_OUT_PATH_*` |
| **缓存写入** | `hiview_cache.c/h` | `WriteToCache()`, `HiviewCache` |
| **缓存读取** | `hiview_cache.c/h` | `ReadFromCache()`, `PrereadFromCache()` |
| **文件初始化** | `hiview_file.c/h` | `InitHiviewFile()`, `WriteFileHeader()`, `ReadFileHeader()` |
| **文件写入** | `hiview_file.c/h` | `WriteToFile()`, `HiviewFile` |
| **文件读取** | `hiview_file.c/h` | `ReadFromFile()`, `ProcFile()` |
| **内存管理** | `hiview_util.c/h` | `HIVIEW_MemAlloc()`, `HIVIEW_MemFree()` |
| **互斥锁** | `hiview_util.c/h` | `HIVIEW_MutexInit()`, `HIVIEW_MutexLock()`, `HIVIEW_MutexUnlock()` |
| **中断锁** | `hiview_util.c/h` | `HIVIEW_IntLock()`, `HIVIEW_IntRestore()` |
| **Hook 机制** | `hiview_util.c/h` | `HIVIEW_InitHook()`, `HIVIEW_Hooks` |

---

## 3.3 关键入口定位

### 服务初始化入口

**功能**：SAMGR 服务注册入口

| 项目 | 值 | 证据位置 |
|------|-----|----------|
| **文件** | `hiview_service.c` | `hiview_service.c:45` |
| **函数** | `Init()` | `hiview_service.c:45-50` |
| **初始化宏** | `SYS_SERVICE_INIT(Init)` | `hiview_service.c:51` |
| **服务名** | `"hiview"` | `hiview_service.c:56`, `hiview_def.h:27` |

**关键代码**：`hiview_service.c:45-51`

```c
static void Init(void)
{
    SAMGR_GetInstance()->RegisterService((Service *)&g_hiviewService);
    SAMGR_GetInstance()->RegisterDefaultFeatureApi(HIVIEW_SERVICE, GET_IUNKNOWN(g_hiviewService));
    InitHiviewComponent();
}
SYS_SERVICE_INIT(Init);
```

### 配置初始化入口

**功能**：DFX 子系统配置初始化

| 项目 | 值 | 证据位置 |
|------|-----|----------|
| **文件** | `hiview_config.c` | `hiview_config.c:29` |
| **函数** | `HiviewConfigInit()` | `hiview_config.c:29-34` |
| **初始化宏** | `CORE_INIT_PRI(HiviewConfigInit, 0)` | `hiview_config.c:37` |
| **优先级** | 0 (最高) | `hiview_config.c:37` |

**关键代码**：`hiview_config.c:29-37`

```c
void HiviewConfigInit(void)
{
    g_hiviewConfig.hiviewInited = FALSE;
    g_hiviewConfig.logOutputModule = 0xFFFFFFFFFFFFFFFF;
}
CORE_INIT_PRI(HiviewConfigInit, 0)
```

---

## 3.4 组件注册机制

### 注册函数

| 函数 | 功能 | 证据位置 |
|------|------|----------|
| `HiviewRegisterInitFunc()` | 注册组件初始化函数 | `hiview_service.c:117-120` |
| `HiviewRegisterMsgHandle()` | 注册消息处理函数 | `hiview_service.c:122-125` |

### 注册回调数组

| 数组 | 类型 | 功能 | 证据位置 |
|------|------|------|----------|
| `g_hiviewInitFuncList[]` | `HiviewInitFunc[HIVIEW_CMP_TYPE_MAX]` | 组件初始化函数列表 | `hiview_service.c:41` |
| `g_hiviewMsgHandleList[]` | `HiviewMsgHandle[HIVIEW_MSG_MAX]` | 消息处理函数列表 | `hiview_service.c:42` |

**关键代码**：`hiview_service.c:41-42`

```c
static HiviewInitFunc g_hiviewInitFuncList[HIVIEW_CMP_TYPE_MAX] = { NULL };
static HiviewMsgHandle g_hiviewMsgHandleList[HIVIEW_MSG_MAX] = { NULL };
```

### 注册示例

```c
// hilog_lite 或其他组件的注册代码
HiviewRegisterInitFunc(HIVIEW_CMP_TYPE_LOG, LogComponentInit);
HiviewRegisterMsgHandle(HIVIEW_MSG_OUTPUT_LOG_TEXT_FILE, LogTextFileHandler);
```

---

## 3.5 消息处理路径

### 消息处理入口

| 项目 | 值 | 证据位置 |
|------|-----|----------|
| **文件** | `hiview_service.c` | `hiview_service.c:75` |
| **函数** | `MessageHandle()` | `hiview_service.c:75-86` |
| **参数** | `Service *service, Request *request` | `hiview_service.c:75` |

**关键代码**：`hiview_service.c:75-86`

```c
static BOOL MessageHandle(Service *service, Request *request)
{
    (void)service;
    if ((request == NULL) || (request->msgId >= HIVIEW_MSG_MAX)) {
        return TRUE;  // 空消息或无效 msgId，直接返回成功
    }
    if (g_hiviewMsgHandleList[request->msgId] != NULL) {
        (*(g_hiviewMsgHandleList[request->msgId]))(request);  // 调用注册的处理器
    }
    return TRUE;
}
```

### 消息分发流程

```
MessageHandle(request)
    ↓
检查 request 和 msgId 有效性
    ↓
从 g_hiviewMsgHandleList[msgId] 获取处理函数
    ↓
调用处理函数 (if not NULL)
```

---

## 3.6 文件操作关键路径

### 文件初始化

| 项目 | 值 | 证据位置 |
|------|-----|----------|
| **文件** | `hiview_file.c` | `hiview_file.c:40` |
| **函数** | `InitHiviewFile()` | `hiview_file.c:40-96` |
| **参数** | `HiviewFile *fp, HiviewFileType type, uint32 size` | `hiview_file.c:40` |

### 文件写入

| 项目 | 值 | 证据位置 |
|------|-----|----------|
| **文件** | `hiview_file.c` | `hiview_file.c:151` |
| **函数** | `WriteToFile()` | `hiview_file.c:151-173` |
| **参数** | `HiviewFile *fp, const uint8 *data, uint32 len` | `hiview_file.c:151` |

### 文件读取

| 项目 | 值 | 证据位置 |
|------|-----|----------|
| **文件** | `hiview_file.c` | `hiview_file.c:175` |
| **函数** | `ReadFromFile()` | `hiview_file.c:175-197` |
| **参数** | `HiviewFile *fp, uint8 *data, uint32 readLen` | `hiview_file.c:175` |

---

## 3.7 配置路径速查

### 文件路径配置宏

| 宏定义 | 默认值 | 说明 | 证据位置 |
|--------|--------|------|----------|
| `HIVIEW_FILE_DIR` | `""` | 基础目录 | `hiview_config.h:31` |
| `HIVIEW_FILE_OUT_PATH_LOG` | `debug.log` | 日志输出 | `hiview_config.h:34` |
| `HIVIEW_FILE_OUT_PATH_UE_EVENT` | `ue.event` | UE 事件 | `hiview_config.h:35` |
| `HIVIEW_FILE_OUT_PATH_FAULT_EVENT` | `fault.event` | 故障事件 | `hiview_config.h:36` |
| `HIVIEW_FILE_OUT_PATH_STAT_EVENT` | `stat.event` | 统计事件 | `hiview_config.h:37` |
| `HIVIEW_FILE_PATH_DUMP` | `dump.dat` | Dump 文件 | `hiview_config.h:39` |

**关键代码**：`hiview_config.h:30-44`

```c
#ifndef HIVIEW_FILE_DIR
#define HIVIEW_FILE_DIR                    ""
#endif

#define HIVIEW_FILE_OUT_PATH_LOG           HIVIEW_FILE_DIR"debug.log"
#define HIVIEW_FILE_OUT_PATH_UE_EVENT      HIVIEW_FILE_DIR"ue.event"
#define HIVIEW_FILE_OUT_PATH_FAULT_EVENT   HIVIEW_FILE_DIR"fault.event"
#define HIVIEW_FILE_OUT_PATH_STAT_EVENT    HIVIEW_FILE_DIR"stat.event"

#define HIVIEW_FILE_PATH_DUMP              HIVIEW_FILE_DIR"dump.dat"
```

---

## 3.8 数据结构速查

### 服务结构体

| 结构体 | 文件 | 说明 |
|--------|------|------|
| `HiviewService` | `hiview_service.h:54-57` | SAMGR 服务结构 |
| `HiviewInterface` | `hiview_service.h:48-51` | 服务接口结构 |
| `Identity` | SAMGR | 服务标识（由 SAMGR 定义） |

**关键代码**：`hiview_service.h:54-57`

```c
typedef struct {
    INHERIT_SERVICE;
    INHERIT_IUNKNOWNENTRY(HiviewInterface);
    Identity identity;
} HiviewService;
```

### 配置结构体

| 结构体 | 文件 | 说明 |
|--------|------|------|
| `HiviewConfig` | `hiview_config.h:76-85` | DFX 子系统配置 |
| `HiviewOutputOption` | `hiview_config.h:88-95` | 输出模式枚举 |

**关键代码**：`hiview_config.h:76-85`

```c
typedef struct {
    uint8 outputOption : 4;       // 控制日志输出模式
    uint8 hiviewInited : 1;       // HiView 服务是否已初始化
    uint8 level : 3;              // 控制日志输出级别
    uint8 logSwitch : 1;          // 是否启用日志组件
    uint8 eventSwitch : 1;        // 是否启用事件组件
    uint8 dumpSwitch : 1;         // 是否启用 dump 组件
    uint64 logOutputModule;       // 控制日志输出模块
    uint16 writeFailureCount;
} HiviewConfig;
```

### 缓存结构体

| 结构体 | 文件 | 说明 |
|--------|------|------|
| `HiviewCache` | `hiview_cache.h:38-46` | 循环缓冲区 |
| `HiviewCacheType` | `hiview_cache.h:28-36` | 缓存类型枚举 |

### 文件结构体

| 结构体 | 文件 | 说明 |
|--------|------|------|
| `HiviewFile` | `hiview_file.h:86-95` | 文件对象 |
| `HiviewFileHeader` | `hiview_file.h:78-84` | 文件头 |
| `FileHeaderCommon` | `hiview_file.h:70-76` | 公共文件头 |

---

## 3.9 代码统计

### 文件行数统计

| 文件 | 代码行数 | 注释行数 | 总行数 |
|------|----------|----------|--------|
| `hiview_service.c` | ~100 | ~40 | 140 |
| `hiview_service.h` | ~50 | ~23 | 73 |
| `hiview_config.c` | ~30 | ~9 | 39 |
| `hiview_config.h` | ~80 | ~28 | 108 |
| `hiview_cache.c` | ~170 | ~53 | 223 |
| `hiview_cache.h` | ~80 | ~36 | 116 |
| `hiview_file.c` | ~260 | ~81 | 341 |
| `hiview_file.h` | ~150 | ~54 | 204 |
| `hiview_util.c` | ~230 | ~83 | 313 |
| `hiview_util.h` | ~80 | ~36 | 116 |
| `hiview_def.h` | ~40 | ~21 | 61 |
| **合计** | ~1270 | ~464 | ~1734 |

### 核心函数数量

| 模块 | 公开函数数 | 内部函数数 |
|------|------------|------------|
| `hiview_service` | 4 | 5 |
| `hiview_config` | 1 | 1 |
| `hiview_cache` | 6 | 2 |
| `hiview_file` | 10 | 4 |
| `hiview_util` | 15 | 5 |
| **合计** | 36 | 17 |

---

## 3.10 下一步

| 你的目标 | 推荐阅读 |
|----------|----------|
| 了解攻击面 | [05_AttackSurface.md](./05_AttackSurface.md) |
| 了解安全风险 | [06_SecurityReview.md](./06_SecurityReview.md) |
| 了解构建配置 | [07_Build.md](./07_Build.md) |
| 了解架构设计 | [02_Architecture.md](./02_Architecture.md) |

---

*所有技术结论均有代码证据支撑，详见 [wiki/_work/NOTES.md](../_work/NOTES.md)。*
