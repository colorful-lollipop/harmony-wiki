# 04_安全评审

> hievent_lite 安全风险分析、攻击面识别与修复建议

## 1. 评审概述

### 1.1 评审范围

| 范围 | 说明 |
|------|------|
| **代码范围** | `frameworks/`, `command/` 目录下所有 `.c/.h` 文件 |
| **接口范围** | `interfaces/native/innerkits/` 对外 C API |
| **构建配置** | `BUILD.gn`, `bundle.json` |

### 1.2 评审方法

- 静态代码分析
- 数据流追踪
- 威胁建模

### 1.3 代码证据范围

| 文件 | 分析状态 |
|------|----------|
| `frameworks/hiview_event.c` | ✅ 已分析 |
| `frameworks/hiview_output_event.c` | ✅ 已分析 |
| `command/hievent_lite_command.c` | ✅ 已分析 |
| `interfaces/native/innerkits/hiview_event.h` | ✅ 已分析 |

---

## 2. 攻击面清单

### 2.1 外部数据入口点

| 入口 | 函数 | 文件:行号 | 输入类型 | 信任级别 |
|------|------|-----------|----------|----------|
| **命令输入** | `HieventCmdProc()` | `hievent_lite_command.c:39` | Shell 字符串 | 高 (仅系统调用) |
| **事件创建** | `HiEventCreate()` | `hiview_event.c:73` | 结构化参数 | 中 (应用调用) |
| **事件上报** | `HiEventPrintf()` | `hiview_event.c:51` | 整数参数 | 中 (应用调用) |
| **参数添加** | `HiEventPutInteger()` | `hiview_event.c:97` | 整数参数 | 中 (应用调用) |
| **事件输出** | `OutputEvent()` | `hiview_output_event.c:220` | 事件数据指针 | 中 (内部调用) |
| **文件路径** | `HiEventFileProc()` | `hiview_event.c:198` | 文件路径字符串 | 低 (需验证) |
| **回调注册** | `HiEventFileAddWatcher()` | `hiview_event.c:188` | 函数指针 | 中 (受控注册) |

### 2.2 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  内部可信区域 (本模块实现)                                 │   │
│  │  ├── hiview_event.c  (事件逻辑)                         │   │
│ hiview_output_event.c (输出逻辑)                    │  ├── │   │
│  │  └── hiview_event.h   (API 定义)                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            ▲                                    │
│                            │                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  边界接口 (需校验)                                        │   │
│  │  ├── HiEventFileProc(dest) - dest 需验证                │   │
│  │  ├── HieventCmdProc(cmd) - 已校验                       │   │
│  │  └── HiEventCreate() - 参数已校验                       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  外部不可信区域                                          │   │
│  │  ├── 用户应用 (调用 HiEvent* API)                       │   │
│  │  ├── 文件系统 (ReadFromFile/WriteToFile)               │   │
│  │  └── hiview_lite 依赖模块                               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 安全风险项

### 3.1 风险项 1: 路径遍历漏洞 (中等)

**风险 ID**: SEC-001

**证据**: `hiview_output_event.c:566-581`

```c
int HiEventFileProcImp(uint8 type, const char *dest, uint8 mode)
{
    Output2Flash(type);
    HIVIEW_MutexLock(g_eventFlushInfo.mutex);
    HiviewCache* c = NULL;
    HiviewFile* f = NULL;

    GetEventCache(type, &c, &f);
    if (f == NULL || strcmp(f->path, dest) == 0) {
        HIVIEW_MutexUnlock(g_eventFlushInfo.mutex);
        return -1;
    }
    int ret = ProcFile(f, dest, (FileProcMode)mode);  // ⚠️ dest 未验证
    HIVIEW_MutexUnlock(g_eventFlushInfo.mutex);
    return ret;
}
```

**问题描述**:
- `dest` 参数直接传递给 `ProcFile()`
- 未验证 `dest` 是否包含路径遍历字符 (`../`, `..\`)

**触发条件**:
```c
// 恶意应用调用
HiEventFileProc(HIEVENT_FAULT, "../../../etc/passwd", 1);
```

**潜在影响**:
- 事件文件被写入敏感位置
- 覆盖系统文件 (如果 ProcFile 支持)

**修复建议**:
```c
// 在传递 dest 前验证路径
#include <stdlib.h>

static boolean IsValidPath(const char *path) {
    if (path == NULL) return FALSE;
    // 检查路径遍历
    for (const char *p = path; *p != '\0'; p++) {
        if ((p[0] == '.' && p[1] == '.') && 
            (p[2] == '/' || p[2] == '\\' || p[2] == '\0')) {
            return FALSE;
        }
    }
    // 确保是绝对路径
    if (path[0] != '/') return FALSE;
    return TRUE;
}

// 使用
if (!IsValidPath(dest)) {
    HIVIEW_UartPrint("Invalid path detected.\n");
    return -1;
}
```

---

### 3.2 风险项 2: 整数溢出 (低)

**风险 ID**: SEC-002

**证据**: `hiview_event.c:82`

```c
event->payload = (uint8 *)HIVIEW_MemAlloc(MEM_POOL_HIVIEW_ID, 
    SINGLE_VALUE_MAX_LEN * num);  // ⚠️ 可能溢出
```

**问题描述**:
- `SINGLE_VALUE_MAX_LEN * num` 未检查溢出
- 如果 `num > UINT_MAX / SINGLE_VALUE_MAX_LEN`，结果截断

**触发条件**:
```c
// num 极大时触发
HiEventCreate(HIEVENT_FAULT, 1, 0x10000000);  // 异常值
```

**潜在影响**:
- 分配过小内存
- 后续写入越界

**修复建议**:
```c
// 检查溢出
#define SINGLE_VALUE_MAX_LEN 5

if (num == 0 || num > (UINT_MAX / SINGLE_VALUE_MAX_LEN)) {
    return NULL;
}
size_t allocSize = SINGLE_VALUE_MAX_LEN * num;
```

---

### 3.3 风险项 3: 回调函数未校验 (低)

**风险 ID**: SEC-003

**证据**: `hiview_output_event.c:583-592`

```c
void HiviewRegisterHieventFileWatcher(uint8 type, FileProc func, const char *path)
{
    if (func == NULL) {
        return;
    }
    HiviewCache* c = NULL;
    HiviewFile* f = NULL;
    GetEventCache(type, &c, &f);
    RegisterFileWatcher(f, func, path);  // ⚠️ func 未校验
}
```

**问题描述**:
- `func` 函数指针直接注册
- 未验证 `func` 是否为有效函数指针

**潜在影响**:
- 注册空指针导致崩溃
- 恶意回调 (可能性低，需先有调用权限)

**修复建议**:
```c
// 检查 func 是否为有效地址 (平台相关)
#include <stdlib.h>

static boolean IsValidFunctionPointer(FileProc func) {
    if (func == NULL) return FALSE;
    // 简化检查：确保不是 NULL
    // 更严格检查需要平台支持
    return TRUE;
}
```

---

### 3.4 风险项 4: 事件 ID 缺少范围检查 (信息)

**风险 ID**: SEC-004

**证据**: `hiview_event.c:59`

```c
e.common.eventId = eventId;  // ⚠️ 直接赋值，无范围检查
```

**问题描述**:
- `eventId` 为 `uint16`，但未检查是否在有效范围内

**潜在影响**:
- 事件 ID 冲突 (ID 重复)
- 日志分析困难

**修复建议**:
```c
// 建议在公共库层面统一分配和校验事件 ID
// 或添加调试断言
void HiEventCreate(uint8 type, uint16 eventId, uint8 num) {
    HIVIEW_DEBUG_ASSERT(eventId < MAX_EVENT_ID, "Event ID overflow");
    // ...
}
```

---

### 3.5 风险项 5: 时间戳精度依赖系统 (信息)

**风险 ID**: SEC-005

**证据**: `hiview_event.c:60,90`

```c
e.common.time = (uint32)(HIVIEW_GetCurrentTime() / MS_PER_SECOND);
```

**问题描述**:
- 时间戳依赖 `HIVIEW_GetCurrentTime()`
- 如果系统时钟被篡改，事件时序不可信

**潜在影响**:
- 事件顺序错乱
- 故障定位困难

**缓解措施**:
- 这是嵌入式系统常见限制
- 依赖系统层面的时钟同步

---

## 4. 安全亮点

### 4.1 正面安全实践

| 实践 | 说明 | 证据 |
|------|------|------|
| **安全字符串函数** | 使用 `memcpy_s()`, `snprintf_s()` | `hiview_event.c:147`, `hiview_output_event.c:424` |
| **边界长度检查** | `strnlen()` 防止缓冲区溢出 | `hievent_lite_command.c:45` |
| **参数校验** | NULL 检查、范围检查贯穿各 API | `hiview_event.c:75,99,116` |
| **白名单验证** | 命令行使用字符白名单 | `hievent_lite_command.c:94-100` |
| **互斥锁保护** | 线程安全机制 | `hiview_output_event.c:72-83` |
| **条件编译** | 可禁用不需要的事件类型 | `hiview_event.h:186-202` |

### 4.2 代码示例: 安全字符串操作

**证据**: `hiview_event.c:147`

```c
// ✅ 使用安全变体
(void)memcpy_s(encodeOut, sizeof(HiEventTag), (void *)&tag, sizeof(HiEventTag));
```

**证据**: `hiview_output_event.c:424`

```c
// ✅ snprintf_s 防止缓冲区溢出
len = snprintf_s(outStr, outStrLen, outStrLen - 1,
    "EVENT: time=%02u:%02u:%02u id=%u type=%u data=null",
    hour, mte, sec, event->common.eventId, event->type);
```

### 4.3 代码示例: 命令行白名单

**证据**: `hievent_lite_command.c:92-102`

```c
static boolean CheckCmdStr(const char *cmd)
{
    // ✅ 白名单验证：只允许特定字符
    while (*cmd != '\0') {
        if (!(isalnum(*cmd) || (*cmd == ' ') || (*cmd == '\n') ||
            (*cmd == '=') || (*cmd == '-'))) {
            return FALSE;
        }
        cmd++;
    }
    return TRUE;
}
```

---

## 5. 依赖模块安全

### 5.1 hiview_lite 依赖

| 依赖项 | 安全考量 |
|--------|----------|
| **内存管理** | `HIVIEW_MemAlloc/MemFree` 实现需审计 |
| **文件系统** | `InitHiviewFile`, `WriteToFile` 实现需审计 |
| **互斥锁** | `HIVIEW_MutexInit/Lock/Unlock` 实现需审计 |
| **消息队列** | `HiviewSendMessage` 实现需审计 |

**注意**: 本 Wiki 仅覆盖 hievent_lite 代码，依赖模块需单独评审。

### 5.2 外部接口

| 接口 | 用途 | 安全责任 |
|------|------|----------|
| **FileProc 回调** | 文件满通知 | 调用者负责校验 |
| **HieventProc 回调** | 事件前置处理 | 调用者负责实现 |
| **命令字符串** | shell 命令 | 已白名单校验 |

---

## 6. 安全建议总结

### 6.1 优先级排序

| 优先级 | 风险项 | 建议 |
|--------|--------|------|
| **P1** | SEC-001 路径遍历 | 实施路径白名单验证 |
| **P2** | SEC-002 整数溢出 | 添加溢出检查 |
| **P3** | SEC-003 回调校验 | 添加函数指针验证 |

### 6.2 长期建议

1. **增加模糊测试** - 对 API 参数进行 Fuzzing 测试
2. **静态分析集成** - 在 CI 中集成 Coverity 或 SonarQube
3. **安全编码规范** - 遵循 MISRA-C 或 CERT C
4. **依赖审计** - 定期审计 hiview_lite 等依赖

---

## 7. 局限性说明

### 7.1 评审局限

| 局限 | 说明 |
|------|------|
| **运行时分析** | 本评审为静态分析，未覆盖运行时行为 |
| **依赖模块** | hiview_lite, hilog_lite 等依赖未在本次评审范围 |
| **系统集成** | 与系统其他部分的交互未详细分析 |

### 7.2 未覆盖代码

| 目录 | 原因 |
|------|------|
| `test/` | 按规范忽略 |
| 依赖模块代码 | 需单独评审 |

---

**跳转**: [03_Build_System.md](03_Build_System.md) | [05_Troubleshooting.md](05_Troubleshooting.md) | [SUMMARY.md](SUMMARY.md)
