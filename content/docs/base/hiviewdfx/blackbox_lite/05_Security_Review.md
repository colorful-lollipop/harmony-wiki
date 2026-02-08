# 05_Security_Review - 安全风险评审

## 5.1 评审范围与方法

### 5.1.1 代码范围

本次评审覆盖 `blackbox_lite` 模块的以下文件：

| 文件 | 评审范围 |
|------|----------|
| `blackbox_core.c` | 核心逻辑、故障处理、线程安全 |
| `blackbox_adapter.c` | WEAK 适配层 |
| `blackbox_detector.c` | 事件上报 |
| `blackbox.h` | 数据结构定义 |
| `blackbox_adapter.h` | 接口定义 |
| `BUILD.gn` | 构建配置 |

### 5.1.2 评审方法

- **代码审计**：静态分析代码漏洞模式
- **数据流分析**：追踪外部输入的处理路径
- **威胁建模**：识别攻击面与信任边界

### 5.1.3 局限性说明

- 未涵盖平台适配层实现（由各芯片厂商提供）
- 未进行运行时渗透测试
- 依赖 hiview_lite 的部分未审计

## 5.2 威胁模型

### 5.2.1 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  blackbox_lite  (可信代码区)                         │    │
│  │  - blackbox_core.c (内核态)                         │    │
│  │  - blackbox_adapter.c (WEAK 默认实现)               │    │
│  └─────────────────────────────────────────────────────┘    │
│                            ▲                               │
│                            │ 边界                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  平台实现区 (半可信)                                 │    │
│  │  - blackbox_adapter_impl.c (厂商实现)               │    │
│  │  - 文件系统操作                                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                            ▲                               │
│                            │ 边界                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  外部输入区 (不可信)                                 │    │
│  │  - 故障中断/异常上下文                               │    │
│  │  - 用户空间传入的事件参数                           │    │
│  │  - 文件系统状态                                      │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 5.2.2 攻击面清单

| 攻击面 | 来源 | 风险等级 |
|--------|------|----------|
| **参数注入** | `BBoxNotifyError()` 的字符串参数 | 高 |
| **缓冲区边界** | `strncpy_s`/`snprintf_s` 截断处理 | 中 |
| **内存分配** | `malloc` 失败处理 | 低 |
| **竞态条件** | `g_opsList` 链表操作 | 中 |
| **文件路径** | `GetFaultLogPath()` 返回路径 | 高 |
| **空指针解引用** | 多处指针参数校验 | 中 |

## 5.3 风险点分析

### 5.3.1 [高危] 字符串截断导致信息丢失

**证据**：`blackbox_core.c:83-94`

```c
// 问题代码
if (strncpy_s(info->event, sizeof(info->event), event,
    Min(strlen(event), sizeof(info->event) - 1)) != EOK) {
    BBOX_PRINT_ERR("strncpy_s failed or the info->event is not enough!\n");
}
```

**问题**：
- 使用 `Min(strlen(event), ...)` 计算拷贝长度，若 event 长度超过缓冲区，会静默截断
- 日志中只保存了部分信息，可能丢失关键故障线索

**触发条件**：
- 调用 `BBoxNotifyError()` 时传入超过 `EVENT_MAX_LEN` (32字节) 的 event 字符串

**影响**：
- 故障信息不完整，影响问题定位
- 可能导致后续逻辑基于不完整的 event 进行匹配

**修复建议**：
```c
size_t eventLen = strlen(event);
if (eventLen >= sizeof(info->event)) {
    BBOX_PRINT_ERR("Event too long: %zu > %zu\n", eventLen, sizeof(info->event) - 1);
    // 可选择拒绝处理或截断并告警
}
```

---

### 5.3.2 [高危] 路径遍历风险

**证据**：`blackbox_core.c:50-69`

```c
static void GetDirName(char *dirBuf, unsigned int dirBufSize, const char *path)
{
    // ...
    const char *end = path + strlen(path);
    while (*end != '/' && end >= path) {
        end--;
    }
    // ...
}
```

**问题**：
- 从 `GetFaultLogPath()` 返回的路径中提取目录
- 若平台实现的 `GetFaultLogPath()` 返回恶意路径（如 `../../../etc`），可能导致目录遍历

**触发条件**：
- 平台适配层 `GetFaultLogPath()` 返回包含 `../` 的恶意路径

**影响**：
- 故障日志可能被写入非预期目录
- 可能覆盖系统文件或泄露敏感信息

**修复建议**：
```c
// 在使用路径前进行规范化检查
char *realPath = realpath(path, NULL);
if (realPath == NULL || !IsPathInAllowedDir(realPath)) {
    BBOX_PRINT_ERR("Invalid path: %s\n", path);
    return;
}
// 使用 realPath 进行后续操作
free(realPath);
```

---

### 5.3.3 [中危] 信号量竞态窗口

**证据**：`blackbox_core.c:269-274`

```c
if (needSysReset == 0) {
    WaitForLogRootDir(dirName);
    if (LOS_SemPend(g_opsListSem, LOS_NO_WAIT) != 0) {
        BBOX_PRINT_ERR("Request g_opsListSem failed!\n");
        goto __out;
    }
}
```

**问题**：
- 当 `needSysReset == 0` 时，使用 `LOS_NO_WAIT` 获取信号量
- 失败后直接跳过链表操作，但未重试或等待

**触发条件**：
- 系统负载高时信号量被长时间占用

**影响**：
- 某些故障场景下可能无法正确处理
- 降低系统可靠性

**修复建议**：
- 考虑使用带超时的等待，而非 `LOS_NO_WAIT`
- 或在信号量获取失败时记录错误但不跳过处理

---

### 5.3.4 [中危] 内存分配失败静默处理

**证据**：`blackbox_core.c:150-154, 262-266`

```c
info = malloc(sizeof(*info));
if (info == NULL) {
    BBOX_PRINT_ERR("malloc failed!\n");
    return NULL;  // 或 return -1
}
```

**问题**：
- 内存分配失败后仅打印日志
- 在故障处理这种关键路径上，分配失败可能导致问题定位困难

**触发条件**：
- 系统内存严重不足

**影响**：
- 故障信息丢失
- 无法进行正常的故障恢复流程

**修复建议**：
```c
info = malloc(sizeof(*info));
if (info == NULL) {
    BBOX_PRINT_ERR("malloc failed!\n");
    // 尝试使用栈上的临时 buffer
    struct ErrorInfo stackInfo;
    FormatErrorInfo(&stackInfo, event, module, errorDesc);
    // 使用 stackInfo 进行后续处理
    return -1;
}
```

---

### 5.3.5 [中危] 空指针解引用风险

**证据**：`blackbox_core.c:77-79`

```c
if (info == NULL || event == NULL || module == NULL || errorDesc == NULL) {
    BBOX_PRINT_ERR("info: %p, event: %p, module: %p, errorDesc: %p\n",
        info, event, module, errorDesc);
    return;  // 空指针导致直接返回
}
```

**问题**：
- 参数校验不完整，某些路径下仍可能访问空指针
- `FormatErrorInfo()` 返回 `void`，调用者无法得知是否成功

**触发条件**：
- 传入 NULL 参数

**影响**：
- 潜在的空指针解引用崩溃
- 故障处理中断

**修复建议**：
```c
// 返回错误码而非静默返回
int FormatErrorInfo(struct ErrorInfo *info, ...) {
    if (info == NULL || ...) return -1;
    // ...
    return 0;
}
```

---

### 5.3.6 [低危] 日志打印信息泄露

**证据**：`blackbox_core.c:54, 131-137`

```c
BBOX_PRINT_ERR("dirBuf: %p, dirBufSize: %u, path: %p!\n", ...);
BBOX_PRINT_INFO("[%s] starts uploading event [%s]\n", info->module, info->event);
```

**问题**：
- 打印的信息可能包含敏感内容（如内存地址、文件路径）
- 在生产环境中泄露系统信息

**触发条件**：
- 故障发生时打印调试信息

**影响**：
- 信息泄露（内存地址、路径等）
- 攻击者可利用信息进行进一步攻击

**修复建议**：
- 在生产版本中使用 `BLACKBOX_DEBUG` 宏控制调试日志
- 避免打印敏感信息

---

### 5.3.7 [低危] 模块重复注册未清理

**证据**：`blackbox_core.c:232-238`

```c
UTILS_DL_LIST_FOR_EACH_ENTRY(temp, &g_opsList, struct BBoxOps, opsList) {
    if (strcmp(temp->ops.module, ops->module) == 0) {
        BBOX_PRINT_ERR("[%s] has been registered!\n", ops->module);
        (void)LOS_SemPost(g_opsListSem);
        free(newOps);  // 分配的内存被丢弃
        return -1;
    }
}
```

**问题**：
- 检测到重复注册后释放了 `newOps`，但调用者可能未处理
- 没有通知调用方具体失败原因

**影响**：
- 内存浪费（每次重复注册都会 malloc 后 free）
- API 使用者无法区分失败原因

---

## 5.4 安全加固建议

### 5.4.1 输入验证

| 检查项 | 建议 |
|--------|------|
| 字符串长度 | 使用 `sizeof()` 而非 `strlen()` 计算边界 |
| 路径合法性 | 拒绝包含 `../` 的路径 |
| 参数有效性 | 拒绝 NULL 指针，使用错误码而非 `void` 返回 |

### 5.4.2 内存安全

| 检查项 | 建议 |
|--------|------|
| malloc 失败 | 提供 fallback 方案或上报错误 |
| 缓冲区截断 | 记录截断事件便于排查 |
| 字符串操作 | 统一使用 `_s` 安全函数 |

### 5.4.3 竞态防护

| 检查项 | 建议 |
|--------|------|
| 信号量获取 | 使用合理超时，避免 `LOS_NO_WAIT` |
| 链表操作 | 确保所有路径都释放信号量 |

### 5.4.4 信息泄露防护

| 检查项 | 建议 |
|--------|------|
| 调试日志 | 默认关闭，使用编译宏控制 |
| 敏感数据 | 不打印路径、地址等内容 |

## 5.5 平台适配安全要求

平台实现 `blackbox_adapter_impl.c` 时应：

1. **验证路径来源**：`GetFaultLogPath()` 返回固定路径，不接受外部输入
2. **文件权限**：日志文件应设置合适的访问权限
3. **重启前清理**：在 `RebootSystem()` 中清理敏感数据
4. **异常隔离**：适配层代码崩溃不应影响整个系统

---

## 参考文档

- [01_Overview](01_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构说明
- [03_API_Reference](03_API_Reference.md) - API 参考
- [06_Troubleshooting](06_Troubleshooting.md) - 常见问题
