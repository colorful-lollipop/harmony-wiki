# 内部 API

> utils_lite 的 C/C++ 内部接口完整参考。

## 文件操作 API

**头文件**：`include/utils_file.h`
**实现**：`file/src/file_impl_hal/file.c`

### API 清单

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `UtilsFileOpen` | `const char* path, int oflag, int mode` | `int` (fd 或 -1) | 打开/创建文件 (行162) |
| `UtilsFileClose` | `int fd` | `int` (0 或 -1) | 关闭文件 (行172) |
| `UtilsFileRead` | `int fd, char* buf, unsigned int len` | `int` (字节数或 -1) | 读取文件 (行185) |
| `UtilsFileWrite` | `int fd, const char* buf, unsigned int len` | `int` (字节数或 -1) | 写入文件 (行197) |
| `UtilsFileDelete` | `const char* path` | `int` (0 或 -1) | 删除文件 (行209) |
| `UtilsFileStat` | `const char* path, unsigned int* fileSize` | `int` (0 或 -1) | 获取文件大小 (行220) |
| `UtilsFileSeek` | `int fd, int offset, unsigned int whence` | `int` (位置或 -1) | 调整读写位置 (行241) |
| `UtilsFileCopy` | `const char* src, const char* dest` | `int` (0 或 -1) | 复制文件 (行254) |
| `UtilsFileMove` | `const char* src, const char* dest` | `int` (0 或 -1) | 移动文件 (行267) |

### UtilsFileOpen

**功能**：打开或创建文件

**参数**：
- `path`：文件路径
- `oflag`：打开模式标志（可组合）
- `mode`：兼容参数，实际无效

**oflag 标志**：
| 标志 | 值 | 说明 |
|------|------|------|
| `O_RDONLY_FS` | 00 | 只读模式 |
| `O_WRONLY_FS` | 01 | 只写模式 |
| `O_RDWR_FS` | 02 | 读写模式 |
| `O_CREAT_FS` | 0100 | 文件不存在时创建 |
| `O_EXCL_FS` | 0200 | 配合 O_CREAT，检查文件是否存在 |
| `O_TRUNC_FS` | 01000 | 截断文件 |
| `O_APPEND_FS` | 02000 | 从文件末尾开始读写 |

**返回**：文件描述符（>=0）或 -1

### UtilsFileCopy

**功能**：复制源文件到目标文件

**实现**：使用 128 字节缓冲区循环读写

**证据**：`file/src/file_impl_hal/file.c:61-100`

---

## HAL File API

**头文件**：`hals/file/hal_file.h`
**实现**：`hals/file/hal_file.c`

### API 清单

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `HalFileOpen` | `const char* path, int oflag, int mode` | `int` | 底层打开文件 (行25) |
| `HalFileClose` | `int fd` | `int` | 底层关闭文件 (行27) |
| `HalFileRead` | `int fd, char* buf, unsigned int len` | `int` | 底层读取 (行29) |
| `HalFileWrite` | `int fd, const char* buf, unsigned int len` | `int` | 底层写入 (行31) |
| `HalFileDelete` | `const char* path` | `int` | 底层删除 (行33) |
| `HalFileStat` | `const char* path, unsigned int* fileSize` | `int` | 底层状态获取 (行35) |
| `HalFileSeek` | `int fd, int offset, unsigned int whence` | `int` | 底层定位 (行37) |

**实现**：直接封装 POSIX 文件操作

**证据**：`hals/file/hal_file.c:21-40`

---

## KV 存储 API

**头文件**：`include/kv_store.h`

### API 清单

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `UtilsGetValue` | `const char* key, char* value, unsigned int len` | `int` | 获取值 (行65) |
| `UtilsSetValue` | `const char* key, const char* value` | `int` | 设置值 (行80) |
| `UtilsDeleteValue` | `const char* key` | `int` | 删除值 (行93) |
| `ClearKVCache` | `void` | `int` | 清除缓存 (行105) |

### UtilsGetValue

**功能**：从文件系统或缓存获取值

**参数**：
- `key`：键名，仅小写字母、数字、下划线、点，长度 1-32 字节
- `value`：输出缓冲区
- `len`：缓冲区大小

**返回值**：
| 值 | 说明 |
|------|------|
| >0 | 值的长度 |
| 0 | 从缓存获取 |
| -1 | 操作失败 |
| -9 | 参数错误 |

### UtilsSetValue

**功能**：设置或更新键值对

**参数**：
- `key`：键名，同上
- `value`：值，长度 1-128 字节

**返回值**：
| 值 | 说明 |
|------|------|
| 0 | 成功 |
| -1 | 失败 |
| -9 | 参数错误 |

### 配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `FEATURE_KV_CACHE` | - | 启用缓存支持 |
| `MAX_CACHE_SIZE` | 10 | 缓存条目数 |
| `MAX_KV_SUM` | 50 | 每应用最大 KV 对数 |

---

## 定时器 API

### Timer Task API

**头文件**：`timer_task/include/nativeapi_timer_task.h`

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `InitTimerTask` | `void` | `int` | 初始化定时器任务 (行29) |
| `StartTimerTask` | `bool isPeriodic, unsigned int delay, void* userCallback, void* userContext, timerHandle_t* timerHandle` | `int` | 启动定时器 (行30) |
| `StopTimerTask` | `const timerHandle_t timerHandle` | `int` | 停止定时器 (行32) |

### KAL Timer API

**头文件**：`kal/timer/include/kal.h`

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `KalTimerCreate` | `KalTimerProc func, KalTimerType type, void* arg, unsigned int millisec` | `KalTimerId` | 创建定时器 (行39) |
| `KalTimerStart` | `KalTimerId timerId` | `KalErrCode` | 启动定时器 (行40) |
| `KalTimerChange` | `KalTimerId timerId, unsigned int millisec` | `KalErrCode` | 修改定时器 (行41) |
| `KalTimerStop` | `KalTimerId timerId` | `KalErrCode` | 停止定时器 (行42) |
| `KalTimerDelete` | `KalTimerId timerId` | `KalErrCode` | 删除定时器 (行43) |
| `KalTimerIsRunning` | `KalTimerId timerId` | `unsigned int` | 检查运行状态 (行44) |

### 类型定义

```cpp
typedef void (*KalTimerProc)(union sigval);  // 定时器回调
typedef void *KalTimerId;                     // 定时器句柄

typedef enum {
    KAL_TIMER_ONCE = 0,    // 单次定时器
    KAL_TIMER_PERIODIC = 1  // 周期性定时器
} KalTimerType;

typedef enum {
    KAL_OK = 0,
    KAL_ERR_PARA = 1,           // 参数错误
    KAL_ERR_INNER = 2,          // 内部错误
    KAL_ERR_TIMER_STATE = 0x100  // 定时器状态错误
} KalErrCode;
```

### KAL 实现

基于 POSIX 实时定时器：

```c
timer_t timerId;
timer_create(CLOCK_REALTIME, &sevp, &timerId);
timer_settime(timerId, 0, &its, NULL);
timer_delete(timerId);
```

**证据**：`kal/timer/src/kal.c:50-76`

---

## 内存池 API

**头文件**：`memory/include/ohos_mem_pool.h`

### 内存类型

| 类型 | 说明 |
|------|------|
| `UIKIT` | UI 框架内存 |
| `UIKIT_LSRAM` | UI 框架 LSRAM 内存 |
| `APPFMK` | 应用框架内存 |
| `APPFMK_LSRAM` | 应用框架 LSRAM 内存 |
| `ACE` | ACE 引擎内存 |
| `ACE_LSRAM` | ACE LSRAM 内存 |
| `JERRY` | JerryScript 内存 |
| `JERRY_LSRAM` | JerryScript LSRAM 内存 |
| `JERRY_HEAP` | JerryScript 堆 |
| `HICHAIN` | 安全内存 |
| `SOFTBUS_LSRAM` | 软总线内存 |
| `I18N_LSRAM` | 国际化内存 |
| `CJSON_LSRAM` | CJSON 内存 |
| `APP_VERIFY_LSRAM` | 应用验证内存 |

### API 清单

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `OhosMalloc` | `MemType type, uint32 size` | `void*` | 按类型分配内存 (行85) |
| `OhosFree` | `void* ptr` | `void` | 释放内存 (行96) |

---

## 链表 API

**头文件**：`include/utils_list.h`

**注意**：非线程安全

### 链表结构

```cpp
typedef struct UtilsList {
    struct UtilsList* prev;
    struct UtilsList* next;
} UTILS_DL_LIST;
```

### 操作 API

| 函数 | 参数 | 说明 |
|------|------|------|
| `UtilsListInit` | `UTILS_DL_LIST* list` | 初始化链表 (行59) |
| `UtilsListAdd` | `UTILS_DL_LIST* list, UTILS_DL_LIST* node` | 添加节点 (行172) |
| `UtilsListTailInsert` | `UTILS_DL_LIST* list, UTILS_DL_LIST* node` | 尾部插入 (行199) |
| `UtilsListHeadInsert` | `UTILS_DL_LIST* list, UTILS_DL_LIST* node` | 头部插入 (行223) |
| `UtilsListDelete` | `UTILS_DL_LIST* node` | 删除节点 (行247) |
| `UtilsListEmpty` | `UTILS_DL_LIST* list` | 检查空链表 (行276) |

### 遍历宏

```cpp
UTILS_DL_LIST_FOR_EACH(item, list)      // 遍历
UTILS_DL_LIST_ENTRY(item, type, member)  // 获取结构体指针
```

---

## 模块依赖方向

```
JS Builtin (js/builtin/)
    │
    ├── deviceinfokit/ ────────────────┐
    │                                  │
    ├── filekit/ ──────────────────────┼──> 外部依赖
    │                                  │
    └── kvstorekit/ ───────────────────┘
           │
           │   public_deps
           ▼
    ┌─────────────────────────────────────────┐
    │           C/C++ API Layer               │
    │  ┌────────────┐  ┌─────────────────┐    │
    │  │ utils_file │  │   kv_store      │    │
    │  └────┬───────┘  └────────┬────────┘    │
    │       │                   │             │
    └───────┼───────────────────┼─────────────┘
            │                   │
            ▼                   ▼
    ┌─────────────────────────────────────────┐
    │           HAL / KAL Layer               │
    │  ┌────────────┐  ┌─────────────────┐    │
    │  │ hal_file   │  │   kal_timer     │    │
    │  └────────────┘  └─────────────────┘    │
    └─────────────────────────────────────────┘
                    │
                    ▼
            POSIX / 文件系统 / 定时器
```

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [目录结构](01_Directory_Structure.md) - 模块布局
- [架构说明](02_Architecture.md) - 组件关系
- [N-API 参考](03_NAPI_Reference.md) - JS 接口
- [GN 构建](05_GN_Build.md) - 构建配置
