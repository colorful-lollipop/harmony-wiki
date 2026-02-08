# 02_API 参考

> hievent_lite C API 接口清单、参数说明与使用示例

## 1. API 分类概览

| 分类 | 接口数 | 主要函数 |
|------|--------|----------|
| [事件创建](#21-事件创建函数) | 2 | `HiEventCreate()`, `HiEventPrintf()` |
| [事件上报](#22-事件上报函数) | 3 | `HiEventReport()`, `HiEventFlush()` |
| [参数添加](#23-参数添加函数) | 1 | `HiEventPutInteger()` |
| [文件操作](#24-文件操作函数) | 6 | `HiEventFileProc()`, `HiEventFileAddWatcher()` |
| [回调注册](#25-回调注册函数) | 2 | `HiEventRegisterProc()`, `HiEventUnRegisterProc()` |
| [锁操作](#26-锁操作函数) | 2 | `HiEventOutputFileLock()`, `HiEventOutputFileUnLock()` |

## 2. 事件创建函数

### 2.1 HiEventCreate - 创建多参数事件

**函数签名**:
```c
HiEvent *HiEventCreate(uint8 type, uint16 eventId, uint8 num);
```

**参数说明**:

| 参数 | 类型 | 范围 | 说明 |
|------|------|------|------|
| `type` | `uint8` | 1, 2, 4 | 事件类型 (FAULT/UE/STAT) |
| `eventId` | `uint16` | 0-65535 | 事件 ID（全局唯一） |
| `num` | `uint8` | 2-16 | 参数数量 |

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `HiEvent*` | 成功，返回事件对象指针 |
| `NULL` | 失败，eventSwitch 关闭或参数越界 |

**代码位置**: `frameworks/hiview_event.c:73-95`

**实现细节**:
```c
HiEvent *HiEventCreate(uint8 type, uint16 eventId, uint8 num)
{
    // 检查开关和参数
    if (g_hiviewConfig.eventSwitch == HIVIEW_FEATURE_OFF || num > EVENT_VALUE_MAX_NUM) {
        return NULL;
    }
    // 分配事件对象
    HiEvent *event = (HiEvent *)HIVIEW_MemAlloc(MEM_POOL_HIVIEW_ID, sizeof(HiEvent));
    if (event == NULL) {
        return NULL;
    }
    // 分配 payload 缓存
    event->payload = (uint8 *)HIVIEW_MemAlloc(MEM_POOL_HIVIEW_ID, SINGLE_VALUE_MAX_LEN * num);
    if (event->payload == NULL) {
        HIVIEW_MemFree(MEM_POOL_HIVIEW_ID, (void *)event);
        return NULL;
    }
    // 初始化公共字段
    event->common.mark = num;   // 临时存储参数数量
    event->common.eventId = eventId;
    event->common.time = (uint32)(HIVIEW_GetCurrentTime() / MS_PER_SECOND);
    event->common.len = 0;
    event->type = type;
    return event;
}
```

**使用示例**:
```c
// 创建包含 3 个参数的用户行为事件
HiEvent *event = HiEventCreate(HIEVENT_UE, 1001, 3);
if (event != NULL) {
    HiEventPutInteger(event, 0, 100);   // 参数 key=0, value=100
    HiEventPutInteger(event, 1, 200);   // 参数 key=1, value=200
    HiEventPutInteger(event, 2, 300);   // 参数 key=2, value=300
    HiEventReport(event);
}
```

---

### 2.2 HiEventPrintf - 单参数事件快速上报

**函数签名**:
```c
void HiEventPrintf(uint8 type, uint16 eventId, int8 key, int32 value);
```

**参数说明**:

| 参数 | 类型 | 范围 | 说明 |
|------|------|------|------|
| `type` | `uint8` | 1, 2, 4 | 事件类型 |
| `eventId` | `uint16` | 0-65535 | 事件 ID |
| `key` | `int8` | -1 或 0-15 | 参数键，-1 表示无参数 |
| `value` | `int32` | 任意 | 参数值 |

**返回值**: 无（直接输出）

**代码位置**: `frameworks/hiview_event.c:51-71`

**实现特点**:
- 无需手动创建/释放事件对象
- 内部自动创建栈上事件并上报
- 适用于单参数场景

**使用示例**:
```c
// 上报单参数故障事件
HiEventPrintf(HIEVENT_FAULT, 2001, 0, -1);

// 上报无参数事件
HiEventPrintf(HIEVENT_STAT, 3001, -1, 0);
```

---

## 3. 参数添加函数

### 3.1 HiEventPutInteger - 添加整数参数

**函数签名**:
```c
void HiEventPutInteger(HiEvent *event, int8 key, int32 value);
```

**参数说明**:

| 参数 | 类型 | 范围 | 说明 |
|------|------|------|------|
| `event` | `HiEvent*` | 非 NULL | 事件对象指针 |
| `key` | `int8` | 0-15 | 参数键（对应 XML 定义） |
| `value` | `int32` | 任意 | 参数值 |

**前置条件**:
- `event != NULL`
- `event->payload != NULL`
- `key >= 0`
- `event->common.mark > 0` (还有未添加的参数)

**代码位置**: `frameworks/hiview_event.c:97-112`

**实现细节**:
```c
void HiEventPutInteger(HiEvent *event, int8 key, int32 value)
{
    // 参数校验
    if (g_hiviewConfig.eventSwitch == HIVIEW_FEATURE_OFF || 
        event == NULL || event->payload == NULL ||
        key < 0 || event->common.mark == 0) {
        return;
    }
    
    uint8 encodeLen;
    // 根据是否是最后一个参数设置 last 标志
    if (event->common.mark <= 1) {
        encodeLen = HiEventEncode((uint8)key, value, 1, 
                                  event->payload + event->common.len);
    } else {
        encodeLen = HiEventEncode((uint8)key, value, 0, 
                                  event->payload + event->common.len);
    }
    event->common.len += encodeLen;
    event->common.mark -= 1;  // 递减剩余参数计数
}
```

**使用示例**:
```c
HiEvent *event = HiEventCreate(HIEVENT_UE, 1001, 3);
HiEventPutInteger(event, 0, 100);   // 第一个参数
HiEventPutInteger(event, 1, 200);   // 第二个参数
HiEventPutInteger(event, 2, 300);   // 第三个参数 (last=1)
HiEventReport(event);
```

---

## 4. 事件上报函数

### 4.1 HiEventReport - 上报并释放事件

**函数签名**:
```c
void HiEventReport(HiEvent *event);
```

**参数说明**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `event` | `HiEvent*` | 事件对象指针 |

**前置条件**: `event != NULL` 且 `event->payload != NULL`

**代码位置**: `frameworks/hiview_event.c:114-127`

**行为**:
1. 检查所有参数已添加 (`event->common.mark == 0`)
2. 恢复事件头标记
3. 调用 `OutputEvent()` 输出
4. 释放 payload 和事件对象内存

---

### 4.2 HiEventFlush - 刷新事件到存储

**函数签名**:
```c
void HiEventFlush(boolean syncFlag);
```

**参数说明**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `syncFlag` | `boolean` | `TRUE`: 同步刷新; `FALSE`: 异步刷新 |

**代码位置**: `frameworks/hiview_event.c:173-176`

**使用场景**: 系统重启前确保事件数据持久化

**使用示例**:
```c
// 同步刷新所有事件
HiEventFlush(TRUE);

// 异步刷新所有事件
HiEventFlush(FALSE);
```

---

## 5. 文件操作函数

### 5.1 HiEventFileProc - 处理事件文件

**函数签名**:
```c
int HiEventFileProc(uint8 type, const char *dest, uint8 mode);
```

**参数说明**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `uint8` | 事件类型 |
| `dest` | `const char*` | 目标文件路径 |
| `mode` | `uint8` | 处理模式 (0: copy, 1: rename) |

**返回值**:

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 失败 (文件为空或路径相同) |

**代码位置**: `frameworks/hiview_event.c:198-201`

**模式说明**:
- `mode=0`: 复制文件到目标路径，保留源文件
- `mode=1`: 重命名文件到目标路径，删除源文件

---

### 5.2 HiEventFileAddWatcher - 添加文件满监控

**函数签名**:
```c
void HiEventFileAddWatcher(uint8 type, FileProc func, const char *dest);
```

**参数说明**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `uint8` | 事件类型 |
| `func` | `FileProc` | 回调函数指针 |
| `dest` | `const char*` | 目标文件路径 |

**代码位置**: `frameworks/hiview_event.c:188-191`

**回调类型定义**:
```c
typedef void (*FileProc)(const char *path, uint8 type, uint8 event);
```

---

### 5.3 HiEventFileRemoveWatcher - 移除文件监控

**函数签名**:
```c
void HiEventFileRemoveWatcher(uint8 type, FileProc func);
```

**代码位置**: `frameworks/hiview_event.c:193-196`

---

## 6. 回调注册函数

### 6.1 HiEventRegisterProc - 注册事件处理回调

**函数签名**:
```c
void HiEventRegisterProc(HieventProc func);
```

**参数类型**:
```c
typedef boolean (*HieventProc)(const HiEvent *event);
```

**说明**: 注册全局事件处理回调，在 `OutputEvent()` 中优先调用

---

### 6.2 HiEventUnRegisterProc - 注销事件处理回调

**函数签名**:
```c
void HiEventUnRegisterProc(HieventProc func);
```

---

## 7. 锁操作函数

### 7.1 HiEventOutputFileLock - 锁定输出文件

**函数签名**:
```c
void HiEventOutputFileLock(void);
```

**说明**: 获取输出文件互斥锁，防止并发访问

---

### 7.2 HiEventOutputFileUnLock - 解锁输出文件

**函数签名**:
```c
void HiEventOutputFileUnLock(void);
```

**说明**: 释放输出文件互斥锁

---

## 8. 宏定义接口

### 8.1 事件类型判断宏

**证据**: `interfaces/native/innerkits/hiview_event.h:177`

```c
#define IS_COMPILE_EVENT(t) (((HIEVENT_COMPILE_TYPE) & (t)) == (t))
```

### 8.2 快速上报宏

| 宏 | 定义 | 说明 |
|----|------|------|
| `HIEVENT_FAULT_REPORT(id, k, v)` | `HiEventPrintf(HIEVENT_FAULT, (id), (k), (v))` | 故障事件快速上报 |
| `HIEVENT_UE_REPORT(id, k, v)` | `HiEventPrintf(HIEVENT_UE, (id), (k), (v))` | 用户事件快速上报 |
| `HIEVENT_STAT_REPORT(id, k, v)` | `HiEventPrintf(HIEVENT_STAT, (id), (k), (v))` | 统计事件快速上报 |

**代码位置**: `interfaces/native/innerkits/hiview_event.h:186-202`

**条件编译**: 当 `HIEVENT_COMPILE_TYPE` 中包含对应类型时才启用

**使用示例**:
```c
// 使用宏上报故障事件
HIEVENT_FAULT_REPORT(2001, 0, -1);

// 使用宏上报用户事件
HIEVENT_UE_REPORT(1001, 0, 100);
```

### 8.3 多参数事件宏

| 宏 | 说明 |
|----|------|
| `HIEVENT_CREATE(type, id, num)` | 创建事件 |
| `HIEVENT_PUT_INT_VALUE(pEvent, k, v)` | 添加参数 |
| `HIEVENT_REPORT(pEvent)` | 上报事件 |

**代码位置**: `interfaces/native/innerkits/hiview_event.h:211-219`

**使用示例**:
```c
HiEvent *event = HIEVENT_CREATE(HIEVENT_UE, 1001, 3);
HIEVENT_PUT_INT_VALUE(event, 0, 100);
HIEVENT_PUT_INT_VALUE(event, 1, 200);
HIEVENT_PUT_INT_VALUE(event, 2, 300);
HIEVENT_REPORT(event);
```

---

## 9. 数据结构

### 9.1 HiEventCommon - 公共头部

**证据**: `interfaces/native/innerkits/hiview_event.h:40-46`

```c
#pragma pack(1)
typedef struct {
    uint8  mark;      /* payload length 或 事件头标记 */
    uint8  len;       /* payload length */
    uint16 eventId;   /* 0-65535 */
    uint32 time;      /* 时间戳 (秒) */
} HiEventCommon;
#pragma pack()
```

### 9.2 HiEvent - 完整事件结构

**证据**: `interfaces/native/innerkits/hiview_event.h:48-53`

```c
typedef struct {
    HiEventCommon common;  /* 公共头部 (8字节) */
    uint8 type;            /* 事件类型 */
    uint8 *payload;        /* TLV 编码数据 */
} HiEvent;
```

---

## 10. 错误码与异常处理

### 10.1 返回值说明

| 场景 | 返回值 | 说明 |
|------|--------|------|
| eventSwitch 关闭 | 无/CAPI 静默返回 | 事件被丢弃 |
| HiEventCreate 失败 | NULL | 内存分配失败或 num 越界 |
| HiEventFileProc 失败 | -1 | 文件为空或路径相同 |
| HiEventCreate num 越界 | NULL | num > 16 |

### 10.2 安全机制

| 机制 | 位置 | 说明 |
|------|------|------|
| 参数校验 | 各函数入口 | NULL 检查、范围检查 |
| 内存检查 | `HiEventCreate()` | malloc 返回值检查 |
| 安全字符串函数 | `HiEventEncode()` | memcpy_s 防止溢出 |
| 互斥锁 | 输出层 | 防止并发访问冲突 |

---

**跳转**: [01_Architecture.md](01_Architecture.md) | [03_Build_System.md](03_Build_System.md) | [SUMMARY.md](SUMMARY.md)
