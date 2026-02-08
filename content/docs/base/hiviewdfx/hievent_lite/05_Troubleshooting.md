# 05_故障排查

> hievent_lite 常见构建/运行/调试问题与解决方案

## 1. 构建问题

### 1.1 编译错误: 头文件找不到

**错误信息**:
```
fatal error: 'hiview_event.h' file not found
```

**原因**:
- 构建时 include 路径未正确配置
- 依赖组件未同步构建

**解决方案**:
```bash
# 1. 确保依赖组件已构建
hb build //base/hiviewdfx/hiview_lite

# 2. 检查 include_dirs 配置
cat BUILD.gn | grep -A 10 "include_dirs"

# 3. 清理并重新构建
hb clean
hb build -f
```

**证据**: `BUILD.gn:35-42`

---

### 1.2 链接错误: 未定义引用

**错误信息**:
```
undefined reference to `HiEventCreate'
undefined reference to `HIVIEW_MemAlloc'
```

**原因**:
- 静态库未链接
- 依赖库缺失

**解决方案**:
```bash
# 1. 检查 BUILD.gn 依赖配置
cat BUILD.gn | grep -A 5 "deps"

# 2. 确保 hievent_lite 被添加到链接
# 在用户模块的 BUILD.gn 中添加:
deps = [
    "//base/hiviewdfx/hievent_lite:hievent_lite",
]

# 3. 检查是否启用了 customize_implementation
# 如果为 true，需要用户提供自定义实现
```

**证据**: `BUILD.gn:44`

---

### 1.3 配置参数不生效

**问题**: 修改 `hievent_lite_cache_size` 后大小未改变

**原因**:
- 修改后未重新触发构建
- 修改位置不对

**解决方案**:
```bash
# 1. 在正确的位置配置 (product 或公司层级的 gn 文件)
# 例如: product/my_product/config.gni

hievent_lite_cache_size = 512  # 新值

# 2. 清理并重新生成
rm -rf out/mini
gn gen out/mini --args="hievent_lite_cache_size=512"
ninja -C out/mini
```

**证据**: `BUILD.gn:14-21`

---

## 2. 运行时问题

### 2.1 事件不上报

**症状**: 调用 `HiEventReport()` 后无输出

**排查步骤**:

```c
// Step 1: 检查事件开关状态
// 在代码中添加调试打印
if (g_hiviewConfig.eventSwitch == HIVIEW_FEATURE_OFF) {
    HIVIEW_UartPrint("Event switch is OFF\n");
}

// Step 2: 检查返回值
HiEvent *event = HiEventCreate(HIEVENT_FAULT, 1001, 1);
if (event == NULL) {
    HIVIEW_UartPrint("Create failed\n");  // 检查原因
}

// Step 3: 检查事件文件是否创建
// 查看目标设备上的文件
```

**原因**:

| 原因 | 说明 | 解决 |
|------|------|------|
| eventSwitch 关闭 | 功能被禁用 | 使用命令 `hievent -c` 开启 |
| 事件类型未编译 | HIEVENT_COMPILE_TYPE 不包含该类型 | 检查编译配置 |
| 缓存未满 | DEBUG 模式下才实时输出 | 检查 outputOption 配置 |
| 文件句柄无效 | InitHiviewFile 失败 | 检查 Flash 状态 |

**证据**: `hievent_lite_command.c:83-89`

---

### 2.2 内存分配失败

**症状**: `HiEventCreate()` 返回 NULL

**排查代码**:
```c
// 在 hiview_event.c 中添加调试
HiEvent *HiEventCreate(uint8 type, uint16 eventId, uint8 num)
{
    // 添加调试打印
    HIVIEW_UartPrint("HiEventCreate: type=%d, eventId=%d, num=%d\n", 
                      type, eventId, num);
    
    if (g_hiviewConfig.eventSwitch == HIVIEW_FEATURE_OFF || 
        num > EVENT_VALUE_MAX_NUM) {
        HIVIEW_UartPrint("Invalid parameter\n");
        return NULL;
    }
    
    HiEvent *event = (HiEvent *)HIVIEW_MemAlloc(MEM_POOL_HIVIEW_ID, 
                                                 sizeof(HiEvent));
    if (event == NULL) {
        HIVIEW_UartPrint("Alloc HiEvent failed\n");
        return NULL;
    }
    // ...
}
```

**原因**:

| 原因 | 说明 | 解决 |
|------|------|------|
| 内存池耗尽 | MEM_POOL_HIVIEW_ID 内存不足 | 增加内存池大小 |
| 事件数过多 | 同时创建太多事件 | 检查事件创建逻辑 |
| 参数 num 过大 | num > 16 | 减少参数数量 |

**证据**: `hiview_event.c:73-86`

---

### 2.3 事件数据丢失

**症状**: 部分事件未被写入文件

**排查步骤**:

```c
// 1. 检查缓存是否溢出
// hiview_output_event.c:269-276

// 当缓存满时会丢弃事件
if ((c->usedSize + sizeof(HiEventCommon) + event->common.len) > EVENT_CACHE_SIZE) {
    if (WriteToCache(c, (uint8 *)&(event->common), sizeof(HiEventCommon)) 
        == sizeof(HiEventCommon)) {
        WriteToCache(c, event->payload, event->common.len);
    } else {
        printf("HiEvent have no sufficient space to write, drop event id %d\n", 
               event->common.eventId);  // ⚠️ 这里会丢弃
    }
}

// 2. 检查 EVENT_CACHE_SIZE 配置
// 默认 256 字节可能不足
```

**原因**:

| 原因 | 说明 | 解决 |
|------|------|------|
| 缓存溢出 | 事件过大或缓存太小 | 增加 EVENT_CACHE_SIZE |
| 写入失败 | WriteToFile 返回错误 | 检查 Flash 状态 |
| 文件已满 | FAULT_EVENT_FILE_SIZE 限制 | 增加文件大小或及时导出 |

**证据**: `hiview_output_event.c:243-276`

---

### 2.4 文件操作失败

**症状**: `HiEventFileProc()` 返回 -1

**排查代码**:
```c
// 添加调试信息
int HiEventFileProcImp(uint8 type, const char *dest, uint8 mode)
{
    Output2Flash(type);
    HIVIEW_MutexLock(g_eventFlushInfo.mutex);
    HiviewCache* c = NULL;
    HiviewFile* f = NULL;

    GetEventCache(type, &c, &f);
    
    HIVIEW_UartPrint("HiEventFileProc: type=%d, f=%p, dest=%s\n", 
                      type, f, dest);
    
    if (f == NULL) {
        HIVIEW_UartPrint("File handle is NULL\n");
    }
    if (f != NULL && strcmp(f->path, dest) == 0) {
        HIVIEW_UartPrint("Source and dest are same\n");
    }
    
    if (f == NULL || strcmp(f->path, dest) == 0) {
        HIVIEW_MutexUnlock(g_eventFlushInfo.mutex);
        return -1;
    }
    // ...
}
```

**原因**:

| 原因 | 说明 | 解决 |
|------|------|------|
| 文件句柄无效 | InitHiviewFile 失败 | 检查 Flash 初始化 |
| 源目标相同 | dest 与源路径相同 | 使用不同路径 |
| Flash 只读 | 文件系统只读 | 检查挂载状态 |

**证据**: `hiview_output_event.c:566-581`

---

## 3. 调试方法

### 3.1 UART 调试输出

**启用方法**:
```c
// 在代码中使用 HIVIEW_UartPrint 打印调试信息
HIVIEW_UartPrint("HiEvent init success.\n");
HIVIEW_UartPrint("Event switch is %d\n", g_hiviewConfig.eventSwitch);
```

**查看位置**: 串口控制台

**证据**: `hiview_event.c:42,46`

---

### 3.2 事件命令行

**可用命令**:
```bash
# 查看帮助
hievent -h

# 开关事件功能
hievent -c

# 示例输出
hievent [-h] [-c]
 -h            Help
 -c            Enable or disable event function
```

**实现位置**: `hievent_lite_command.c:69-90`

---

### 3.3 缓存状态查看

**代码位置**: `hiview_output_event.c:377-384`

```c
uint32 GetEventFileSize(uint8 eventType)
{
    HiviewCache *c = NULL;
    HiviewFile *f = NULL;

    GetEventCache(eventType, &c, &f);
    return GetFileUsedSize(f);
}

// 使用
uint32 size = GetEventFileSize(HIEVENT_FAULT);
HIVIEW_UartPrint("Fault event file size: %u\n", size);
```

---

## 4. 常见问题 FAQ

### Q1: 如何禁用某类事件?

**回答**: 通过 `HIEVENT_COMPILE_TYPE` 宏控制

```c
// 在 BUILD.gn 中修改
# 禁用故障事件
hievent_lite_fault_file_size = 0

# 或在代码中使用条件编译
#define HIEVENT_COMPILE_TYPE (HIEVENT_UE | HIEVENT_STAT)  // 排除 FAULT
```

**证据**: `interfaces/native/innerkits/hiview_event.h:173-178`

---

### Q2: 事件 ID 范围是多少?

**回答**: `uint16` 类型，范围 0-65535

```c
// 事件 ID 定义
uint16 eventId;  // 0-65535

// 使用示例
HiEventCreate(HIEVENT_FAULT, 2001, 1);  // ID=2001
```

**证据**: `interfaces/native/innerkits/hiview_event.h:44`

---

### Q3: 一个事件最多可以带多少参数?

**回答**: 最多 16 个参数

```c
// hiview_event.h:28
#define EVENT_VALUE_MAX_NUM    16

// 使用
HiEventCreate(HIEVENT_FAULT, 2001, 16);  // 最大
HiEventCreate(HIEVENT_FAULT, 2001, 17);   // 失败，返回 NULL
```

**证据**: `hiview_event.c:28`, `75`

---

### Q4: 如何查看缓存使用情况?

**回答**: 通过调试代码或 `GetEventFileSize()`

```c
// 添加到你的调试代码
printf("Fault cache used: %d / %d\n", 
       g_faultEventCache.usedSize, 
       EVENT_CACHE_SIZE);

printf("UE cache used: %d / %d\n", 
       g_ueEventCache.usedSize, 
       EVENT_CACHE_SIZE);

printf("Stat cache used: %d / %d\n", 
       g_statEventCache.usedSize, 
       EVENT_CACHE_SIZE);
```

**证据**: `hiview_output_event.c:35-46`

---

### Q5: 事件文件存储路径是什么?

**回答**: 路径由 `HIVIEW_FILE_PATH_*` 宏定义（在 hiview_lite 中）

```c
// hiview_output_event.c:47-70
static HiviewFile g_faultEventFile = {
    .path = HIVIEW_FILE_PATH_FAULT_EVENT,
    .outPath = HIVIEW_FILE_OUT_PATH_FAULT_EVENT,
    // ...
};

static HiviewFile g_ueEventFile = {
    .path = HIVIEW_FILE_PATH_UE_EVENT,
    .outPath = HIVIEW_FILE_OUT_PATH_UE_EVENT,
    // ...
};
```

**注意**: 实际路径定义在 hiview_lite 依赖中，需查看该模块。

---

### Q6: 系统重启后事件数据会丢失吗?

**回答**: 取决于调用时机

```c
// 情况1: 在缓存满后自动写入 - 不会丢失
// OutputEvent() 在缓存满时触发写入

// 情况2: 重启前手动刷新 - 不会丢失
HiEventFlush(TRUE);  // 同步刷新所有事件

// 情况3: 突然断电且缓存未满 - 可能丢失
// 建议在关键操作后调用 HiEventFlush()
```

**证据**: `hiview_output_event.c:546-551`

---

## 5. 调试技巧总结

### 5.1 快速检查清单

```
□ 1. eventSwitch 是否开启?
   - 查看 g_hiviewConfig.eventSwitch 值

□ 2. 事件类型是否编译?
   - 检查 HIEVENT_COMPILE_TYPE 宏

□ 3. 缓存是否充足?
   - 查看 usedSize 是否接近 EVENT_CACHE_SIZE

□ 4. 文件句柄是否有效?
   - 检查 fhandle 是否 >= 0

□ 5. 依赖服务是否就绪?
   - 检查 hiview_lite 初始化状态
```

### 5.2 推荐调试流程

```
1. 启用 UART 输出
2. 调用 HiEventReport()
3. 检查 UART 是否有输出
4. 有输出 → 检查文件写入
5. 无输出 → 检查 eventSwitch
```

---

**跳转**: [04_Security_Review.md](04_Security_Review.md) | [00_Overview.md](00_Overview.md) | [SUMMARY.md](SUMMARY.md)
