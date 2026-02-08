# 安全风险评审

## 目的

本文档描述 HiView Lite 的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

本文档适用于：
- 需要了解安全风险的审计员
- 需要修复漏洞的维护者
- 需要评估安全的集成方

## 相关跳转

- [架构说明](03_Architecture.md) - 了解组件关系
- [内部 API](05_Internal_API.md) - 了解模块接口
- [对外 API](04_External_API.md) - 了解导出接口

---

## 威胁模型

### 攻击面分析

HiView Lite 作为内部系统服务，具有以下攻击面：

| 攻击面 | 说明 | 风险等级 |
|----------|------|----------|
| 文件系统 | 日志、事件文件的读写 | 中 |
| 内存操作 | 缓存、内存分配 | 低 |
| 消息传递 | SAMGR Lite 消息 | 低 |
| Hook 机制 | 可覆盖底层函数 | 低 |

### 信任边界

```
┌─────────────────────────────────────────────────────────┐
│              不可信环境（外部模块）                │
│         (hilog_lite, hievent_lite 等)            │
└─────────────────────┬───────────────────────────────┘
                      │ 注册初始化/消息处理函数
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│            HiView Lite（可信）                    │
│  ┌─────────────────────────────────────────┐        │
│  │  配置层（hiview_config）        │        │
│  │  - 全局配置（可被外部访问）    │        │
│  └─────────────────────────────────────────┘        │
│                      │                           │
│                      ▼                           │
│  ┌─────────────────────────────────────────┐        │
│  │  文件层（hiview_file）          │        │
│  │  - 文件路径（可被配置）          │        │
│  └─────────────────────────────────────────┘        │
└─────────────────────┬───────────────────────────────┘
                      │ 文件 I/O
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│              文件系统（不可信）                    │
│    /data/log/debug.log, ue.event, fault.event     │
└─────────────────────────────────────────────────────────┘
```

**说明**：
- HiView Lite 内部是可信的
- 文件系统是不可信的（可能被其他模块或用户访问）
- 外部模块（hilog_lite、hievent_lite）是不可信的

---

## 可被利用点

### 1. 文件路径遍历

**风险等级**：中

**位置**：`hiview_file.c:291-311` - `RegisterFileWatcher`

**证据**：
```c
void RegisterFileWatcher(HiviewFile *fp, FileProc func, const char *path)
{
    // ...
    if (path == NULL || IsValidPath(path) != 0) {
        return;
    }

    int len = strlen(path) + 1;
    char* tmp = (char*)HIVIEW_MemAlloc(MEM_POOL_HIVIEW_ID, len);
    if (tmp == NULL) {
        return;
    }
    if (strcpy_s(tmp, len, path) != EOK) {
        HIVIEW_MemFree(MEM_POOL_HIVIEW_ID, tmp);
        return;
    }
    fp->outPath = tmp;
}
```

**可利用路径**：
1. 外部模块调用 `RegisterFileWatcher` 时传入恶意路径（如 `../../../etc/passwd`）
2. `IsValidPath` 只检查预定义路径，不检查路径遍历
3. `strcpy_s` 可能复制路径遍历字符串
4. 文件操作可能导致敏感文件访问

**影响**：
- 访问系统敏感文件
- 覆盖系统配置文件
- 信息泄露

**修复建议**：
```c
// 在 RegisterFileWatcher 中添加路径规范化
void RegisterFileWatcher(HiviewFile *fp, FileProc func, const char *path)
{
    // ... 现有代码 ...

    // 新增：路径规范化
    char normalizedPath[PATH_MAX] = {0};
    if (realpath(path, normalizedPath) == NULL) {
        HIVIEW_MemFree(MEM_POOL_HIVIEW_ID, tmp);
        return;
    }

    // 新增：检查路径是否在允许的目录内
    if (strncmp(normalizedPath, HIVIEW_FILE_DIR, strlen(HIVIEW_FILE_DIR)) != 0) {
        HIVIEW_MemFree(MEM_POOL_HIVIEW_ID, tmp);
        return;
    }

    // ... 其余代码 ...
}
```

---

### 2. 缓冲区溢出

**风险等级**：低

**位置**：`hiview_cache.c:76-87` - `WriteToCache`

**证据**：
```c
if ((uint32)cache->wCursor + (uint32)wLen > (uint32)cache->size) {
    firstLen = cache->size - cache->wCursor;
    if (firstLen > 0) {
        if (memcpy_s(cache->buffer + cache->wCursor, firstLen, data, firstLen) == EOK) {
            cache->wCursor += firstLen;
            cache->usedSize += firstLen;
        }
    }
    cache->wCursor = 0;
    secondLen = wLen - firstLen;
    if (secondLen > 0) {
        if (memcpy_s(cache->buffer + cache->wCursor, secondLen, data + firstLen, secondLen) == EOK) {
            cache->wCursor += secondLen;
            cache->usedSize += secondLen;
        }
    }
}
```

**可利用路径**：
1. `memcpy_s` 虽然是安全版本，但如果 `firstLen` 或 `secondLen` 计算错误，仍可能导致问题
2. 缺少对 `firstLen` 和 `secondLen` 的边界检查

**影响**：
- 缓存数据损坏
- 内存越界访问
- 潜在的代码执行

**修复建议**：
```c
// 添加更严格的边界检查
if ((uint32)cache->wCursor + (uint32)wLen > (uint32)cache->size) {
    firstLen = cache->size - cache->wCursor;
    if (firstLen > 0 && firstLen <= wLen) {
        if (memcpy_s(cache->buffer + cache->wCursor, firstLen, data, firstLen) == EOK) {
            cache->wCursor += firstLen;
            cache->usedSize += firstLen;
        }
    }
    cache->wCursor = 0;
    secondLen = wLen - firstLen;
    // 新增：检查 secondLen 的有效性
    if (secondLen > 0 && secondLen <= wLen && secondLen <= cache->size) {
        if (memcpy_s(cache->buffer + cache->wCursor, secondLen, data + firstLen, secondLen) == EOK) {
            cache->wCursor += secondLen;
            cache->usedSize += secondLen;
        }
    }
}
```

---

### 3. 文件竞态条件

**风险等级**：中

**位置**：`hiview_file.c:227-274` - `ProcFile`

**证据**：
```c
int8 ProcFile(HiviewFile *fp, const char *dest, FileProcMode mode)
{
    // ...
    if (HIVIEW_MutexLockOrWait(fp->mutex, OUT_PATH_WAIT_TIMEOUT) != 0) {
        HIVIEW_UartPrint("Procfile failed, get lock fail");
        return -1;
    }
    switch (mode) {
        case HIVIEW_FILE_RENAME: {
            HIVIEW_FileClose(fp->fhandle);
            uint8 type = fp->header.common.type;
            uint32 size = fp->configSize;
            int32 ret = HIVIEW_FileMove(fp->path, dest);
            if (InitHiviewFile(fp, (HiviewFileType)type, size) == FALSE || ret != 0) {
                HIVIEW_MutexUnlock(fp->mutex);
                HIVIEW_UartPrint("Procfile failed, type : HIVIEW_FILE_RENAME");
                return -1;
            }
            break;
        }
    }
    HIVIEW_MutexUnlock(fp->mutex);
    return 0;
}
```

**可利用路径**：
1. 线程 A 调用 `ProcFile` 重命名文件
2. 线程 B 同时访问同一文件
3. 虽然有互斥锁，但 `HIVIEW_FileClose` 和 `HIVIEW_FileMove` 之间仍有竞态窗口
4. 攻击者可能在重命名期间注入恶意文件

**影响**：
- 文件数据损坏
- 替换为恶意文件
- 数据泄露

**修复建议**：
```c
// 使用原子文件操作
case HIVIEW_FILE_RENAME: {
    // 先移动文件，再关闭
    uint8 type = fp->header.common.type;
    uint32 size = fp->configSize;

    // 新增：先移动文件，确保操作原子的
    int32 ret = HIVIEW_FileMove(fp->path, dest);
    if (ret != 0) {
        HIVIEW_MutexUnlock(fp->mutex);
        uint16 hieventID = 1;
        HiEvent *hievent = HIEVENT_CREATE(HIEVENT_FAULT, hieventID, 2);
        HIEVENT_PUT_INT_VALUE(hievent, 0, (int32)fp->header.common.type);
        HIEVENT_PUT_INT_VALUE(hievent, 1, ret);
        HIEVENT_REPORT(hievent);
        return -1;
    }

    HIVIEW_FileClose(fp->fhandle);

    if (InitHiviewFile(fp, (HiviewFileType)type, size) == FALSE) {
        HIVIEW_MutexUnlock(fp->mutex);
        HIVIEW_UartPrint("Procfile failed, type : HIVIEW_FILE_RENAME");
        return -1;
    }
    break;
}
```

---

### 4. 内存泄漏

**风险等级**：低

**位置**：`hiview_util.c:55-59` - `HIVIEW_MemAlloc`

**证据**：
```c
void *HIVIEW_MemAlloc(uint8 modId, uint32 size)
{
    (void)modId;
    return malloc(size);
}
```

**可利用路径**：
1. 持续调用 `HIVIEW_MemAlloc` 但不释放
2. 内存逐渐耗尽
3. 系统变得不稳定或崩溃

**影响**：
- 内存耗尽
- 系统不稳定
- 拒绝服务

**修复建议**：
```c
// 添加内存限制和监控
#define MAX_MEMORY_POOL_SIZE (100 * 1024) // 100KB
static uint32 g_totalAllocated = 0;

void *HIVIEW_MemAlloc(uint8 modId, uint32 size)
{
    (void)modId;

    // 新增：检查内存限制
    if (g_totalAllocated + size > MAX_MEMORY_POOL_SIZE) {
        HIVIEW_UartPrint("Memory limit exceeded");
        return NULL;
    }

    void *ptr = malloc(size);
    if (ptr != NULL) {
        g_totalAllocated += size;
    }
    return ptr;
}

void HIVIEW_MemFree(uint8 modId, void *pMem)
{
    (void)modId;
    if (pMem != NULL) {
        free(pMem);
        // 新增：更新已分配内存计数
        // 注意：这里无法获取释放的大小，需要应用层管理
    }
}
```

---

### 5. 整数溢出

**风险等级**：低

**位置**：`hiview_cache.c:68-71` - `WriteToCache`

**证据**：
```c
// cast to uint32 for prevent uint16 overflow
if ((uint32)cache->size < (uint32)wLen + (uint32)cache->usedSize) {
    HIVIEW_IntRestore(intSave);
    return -1;
}
```

**可利用路径**：
1. 传入极大的 `wLen` 值（接近 UINT16_MAX）
2. `(uint32)wLen + (uint32)cache->usedSize` 可能溢出
3. 绕过大小检查，写入过多数据

**影响**：
- 缓存溢出
- 内存损坏
- 潜在的代码执行

**修复建议**：
```c
// 添加更严格的溢出检查
if ((uint64)wLen + (uint64)cache->usedSize > (uint64)cache->size) {
    HIVIEW_IntRestore(intSave);
    return -1;
}
```

---

## 检查范围与局限性

### 已检查范围

| 安全方面 | 检查状态 | 说明 |
|----------|----------|------|
| 输入校验 | ✅ 已检查 | NULL 指针、长度校验 |
| 路径遍历 | ⚠️ 部分检查 | IsValidPath 只检查预定义路径，不检查路径遍历 |
| 权限缺失 | ✅ 不适用 | 作为内部服务，不需要权限 |
| 内存安全 | ⚠️ 部分检查 | 使用 memcpy_s，但仍有溢出风险 |
| 竞态条件 | ⚠️ 部分检查 | 有互斥锁，但仍有竞态窗口 |
| 信息泄露 | ⚠️ 未完全检查 | 文件权限未严格控制 |
| 动态加载/解析 | ✅ 不适用 | 不涉及动态加载 |

### 未检查内容

| 内容 | 原因 |
|------|------|
| 权限控制 | HiView Lite 是内部服务，不涉及用户权限 |
| 网络通信 | 不涉及网络操作 |
| 加密/解密 | 日志和事件以明文存储 |
| 访问控制 | 文件权限依赖底层文件系统 |

---

## 修复优先级

### 高优先级

1. **文件路径遍历** - 可能导致敏感文件访问
2. **文件竞态条件** - 可能导致文件数据损坏

### 中优先级

3. **缓冲区溢出** - 可能导致内存越界
4. **整数溢出** - 可能绕过大小检查

### 低优先级

5. **内存泄漏** - 长期运行可能导致内存耗尽

---

## 安全最佳实践

### 1. 输入验证

所有输入都应该进行严格的验证：

```c
// ✅ 好的做法
if (path != NULL && strlen(path) < MAX_PATH_LEN) {
    // 处理路径
}

// ❌ 不好的做法
if (path != NULL) {
    // 直接使用 path，不检查长度
}
```

### 2. 边界检查

所有数组访问都应该进行边界检查：

```c
// ✅ 好的做法
if (index < array_size && array != NULL) {
    value = array[index];
}

// ❌ 不好的做法
value = array[index]; // 可能越界
```

### 3. 使用安全函数

优先使用安全版本的字符串和内存操作函数：

```c
// ✅ 好的做法
memcpy_s(dest, dest_size, src, src_size);

// ❌ 不好的做法
memcpy(dest, src, len); // 可能溢出
```

### 4. 最小权限原则

文件和目录应该使用最小必要权限：

```c
// ✅ 好的做法
HIVIEW_FileOpen(path, O_RDWR | O_CREAT, 0600);

// ❌ 不好的做法
HIVIEW_FileOpen(path, O_RDWR | O_CREAT, 0777); // 权限过大
```

---

## 关键结论

1. **攻击面有限** - HiView Lite 作为内部服务，攻击面相对有限。
2. **文件系统风险** - 主要安全风险来自文件系统操作。
3. **路径遍历** - 需要添加路径规范化和验证。
4. **竞态条件** - 需要使用原子操作减少竞态窗口。
5. **内存安全** - 需要添加更严格的边界检查。

---

*最后更新：2026-02-06*
