# 安全风险评估

> **文档版本**: 1.0  
> **最后更新**: 2026-02-07  
> **代码版本**: v3.1  
> **审计者**: 安全研究员

---

## 摘要

本报告对 hiview_lite 代码库进行了安全风险评估，识别了以下风险：

| 风险 ID | 风险名称 | 风险等级 | 类型 |
|---------|----------|----------|------|
| R1 | 路径遍历风险 | 中 | 输入验证缺陷 |
| R2 | IPC 消息验证不充分 | 中 | 输入验证缺陷 |
| R3 | 组件注册无验证 | 中 | 输入验证缺陷 |
| R4 | 缓存竞态条件 | 低 | 并发安全 |
| R5 | 文件描述符泄漏风险 | 低 | 资源管理 |

---

## R1: 路径遍历风险（中危）

### 位置

`hiview_config.h:34-37`

### 证据

```c
// 直接使用配置宏拼接路径，无路径规范化
#define HIVIEW_FILE_OUT_PATH_LOG           HIVIEW_FILE_DIR"debug.log"
#define HIVIEW_FILE_OUT_PATH_UE_EVENT      HIVIEW_FILE_DIR"ue.event"
#define HIVIEW_FILE_OUT_PATH_FAULT_EVENT   HIVIEW_FILE_DIR"fault.event"
#define HIVIEW_FILE_OUT_PATH_STAT_EVENT    HIVIEW_FILE_DIR"stat.event"
```

### 触发路径

```
编译配置 → HIVIEW_FILE_DIR="../../../etc/" 
         → WriteToFile() 
         → ../../../etc/debug.log
```

### 影响评估

**可利用性**：低
- HIVIEW_FILE_DIR 是编译时宏，非运行时输入
- 需要修改编译配置才能利用

**权限提升**：无
- 写入位置受限于编译时配置
- 不会提升权限

**影响范围**：文件写入位置可控

### 修复建议

```c
// 在使用路径前进行规范化
char* NormalizePath(const char* path) {
    // 1. 移除 .. 遍历
    // 2. 确保路径在允许的目录内
    // 3. 返回规范化后的路径
}
```

---

## R2: IPC 消息验证不充分（中危）

### 位置

`hiview_service.c:75-86`

### 证据

```c
static BOOL MessageHandle(Service *service, Request *request)
{
    (void)service;
    if ((request == NULL) || (request->msgId >= HIVIEW_MSG_MAX)) {
        return TRUE;  // 只检查 msgId 范围
    }
    if (g_hiviewMsgHandleList[request->msgId] != NULL) {
        (*(g_hiviewMsgHandleList[request->msgId]))(request);  // data 和 len 未验证
    }
    return TRUE;
}
```

### 触发路径

```
外部组件 → HiviewSendMessage(msgId, msgValue, data, len)
       → MessageHandle(request)
       → g_hiviewMsgHandleList[msgId](request)
       → 处理函数直接使用 data 和 len
```

### 影响评估

**可利用性**：中
- data 和 len 未验证，依赖处理函数
- 处理函数可能存在缓冲区溢出

**权限提升**：可能
- hiview_lite 以系统服务权限运行
- 利用成功可能获取服务权限

### 修复建议

```c
static BOOL MessageHandle(Service *service, Request *request)
{
    (void)service;
    if ((request == NULL) || (request->msgId >= HIVIEW_MSG_MAX)) {
        return TRUE;
    }
    
    // 添加 data 和 len 验证
    if (request->data == NULL || request->len == 0) {
        HIVIEW_UartPrint("Invalid message data");
        return TRUE;
    }
    
    // 添加长度上限检查
    if (request->len > MAX_MESSAGE_LEN) {
        HIVIEW_UartPrint("Message too long");
        return TRUE;
    }
    
    if (g_hiviewMsgHandleList[request->msgId] != NULL) {
        (*(g_hiviewMsgHandleList[request->msgId]))(request);
    }
    return TRUE;
}
```

---

## R3: 组件注册无验证（中危）

### 位置

`hiview_service.c:117-125`

### 证据

```c
void HiviewRegisterInitFunc(HiviewComponentType type, HiviewInitFunc func)
{
    g_hiviewInitFuncList[type] = func;  // 无 type 范围检查
}                                               // 无 func NULL 检查

void HiviewRegisterMsgHandle(HiviewInnerMessage type, HiviewMsgHandle func)
{
    g_hiviewMsgHandleList[type] = func;  // 无 type 范围检查
}                                                // 无 func NULL 检查
}
```

### 触发路径

```
恶意组件 → HiviewRegisterInitFunc(-1, malicious_func)
       → g_hiviewInitFuncList[-1] = malicious_func
       → InitHiviewComponent() 
       → 调用 g_hiviewInitFuncList[-1]
       → 崩溃或任意代码执行
```

### 影响评估

**可利用性**：中
- 需要在同一进程内有代码执行能力
- 恶意注册可能导致服务崩溃

**权限提升**：可能
- 利用成功可导致服务拒绝
- 可能执行任意函数

### 修复建议

```c
void HiviewRegisterInitFunc(HiviewComponentType type, HiviewInitFunc func)
{
    // 添加 type 范围检查
    if (type < 0 || type >= HIVIEW_CMP_TYPE_MAX) {
        HIVIEW_UartPrint("Invalid component type");
        return;
    }
    
    // 添加 func NULL 检查
    if (func == NULL) {
        HIVIEW_UartPrint("NULL init function");
        return;
    }
    
    g_hiviewInitFuncList[type] = func;
}
```

---

## R4: 缓存竞态条件（低危）

### 位置

`hiview_cache.c:58-107`

### 证据

```c
int32 WriteToCache(HiviewCache *cache, const uint8 *data, uint32 len)
{
    HIVIEW_IntLock();  // 使用中断锁保护
    
    uint32 freeSize = cache->size - cache->usedSize;
    if (len > freeSize) {
        HIVIEW_IntRestore();
        return 0;
    }
    
    // ... 写入逻辑
    
    cache->wCursor = (cache->wCursor + len) % cache->size;
    cache->usedSize = MIN(cache->usedSize + len, cache->size);
    
    HIVIEW_IntRestore();
    return len;
}
```

### 触发路径

```
多任务 → 同时调用 WriteToCache
      → 中断锁保护
      → 理论上安全
```

### 影响评估

**可利用性**：低
- 已使用中断锁保护
- 在 LiteOS-M 环境下相对安全

**潜在问题**：
- 中断锁可能影响系统响应性
- 长时间占用中断锁可能导致中断丢失

### 修复建议

考虑使用自旋锁替代中断锁：

```c
// 伪代码
int32 WriteToCache(HiviewCache *cache, const uint8 *data, uint32 len)
{
    uint32_t ticket = HIVIEW_SpinLockTry(&cache->spinlock);
    if (ticket == 0) {
        return 0;  // 获取锁失败
    }
    
    // ... 写入逻辑
    
    HIVIEW_SpinUnlock(&cache->spinlock, ticket);
    return len;
}
```

---

## R5: 文件描述符泄漏风险（低危）

### 位置

`hiview_file.c:40-96`

### 证据

```c
boolean InitHiviewFile(HiviewFile *fp, HiviewFileType type, uint32 size)
{
    // 打开文件
    fp->fhandle = HIVIEW_FileOpen(fp->path, O_RDWR | O_CREAT, 0666);
    if (fp->fhandle < 0) {
        return FALSE;  // 打开失败返回
    }
    
    // 读取或创建文件头
    int32 ret = ReadFileHeader(fp);
    if (ret == FALSE) {
        // 没有文件头，创建新的
        ret = WriteFileHeader(fp);
        if (ret == FALSE) {
            return FALSE;  // 创建失败返回，但未关闭文件描述符
        }
    }
    
    return TRUE;
}
```

### 触发路径

```
InitHiviewFile() → 打开文件
               → ReadFileHeader() 失败
               → WriteFileHeader() 失败
               → 返回 FALSE
               → 文件描述符 fp->fhandle 未关闭
```

### 影响评估

**可利用性**：低
- 需要多次失败才能累积泄漏
- LiteOS-M 资源有限，泄漏容易被发现

**影响**：资源泄漏，可能导致拒绝服务

### 修复建议

```c
boolean InitHiviewFile(HiviewFile *fp, HiviewFileType type, uint32 size)
{
    fp->fhandle = HIVIEW_FileOpen(fp->path, O_RDWR | O_CREAT, 0666);
    if (fp->fhandle < 0) {
        return FALSE;
    }
    
    int32 ret = ReadFileHeader(fp);
    if (ret == FALSE) {
        ret = WriteFileHeader(fp);
        if (ret == FALSE) {
            HIVIEW_FileClose(fp->fhandle);  // 关闭文件描述符
            return FALSE;
        }
    }
    
    return TRUE;
}
```

---

## 风险汇总

| ID | 风险名称 | 风险等级 | 可利用性 | 影响范围 | 修复优先级 |
|----|----------|----------|----------|----------|------------|
| R1 | 路径遍历风险 | 中 | 低 | 文件写入 | 中 |
| R2 | IPC 消息验证不充分 | 中 | 中 | 消息处理 | 高 |
| R3 | 组件注册无验证 | 中 | 中 | 组件管理 | 高 |
| R4 | 缓存竞态条件 | 低 | 低 | 缓存操作 | 低 |
| R5 | 文件描述符泄漏风险 | 低 | 低 | 资源管理 | 中 |

---

## 安全评估结论

### 整体安全状况

| 维度 | 评分 | 说明 |
|------|------|------|
| 输入验证 | ⭐⭐ | 存在多处验证不足 |
| 内存安全 | ⭐⭐⭐ | 使用 memcpy_s，边界检查较好 |
| 并发安全 | ⭐⭐⭐⭐ | 中断锁保护，较为安全 |
| 资源管理 | ⭐⭐⭐ | 基本安全，存在泄漏风险 |
| 代码质量 | ⭐⭐⭐ | 代码结构清晰，注释完善 |

### 建议修复优先级

1. **高优先级**：R2 (IPC 消息验证) - 容易被利用
2. **高优先级**：R3 (组件注册验证) - 可能导致崩溃
3. **中优先级**：R1 (路径遍历) - 编译时配置，风险可控
4. **中优先级**：R5 (文件描述符) - 资源泄漏
5. **低优先级**：R4 (竞态条件) - 影响较小

---

## 参考资料

| 文档 | 说明 |
|------|------|
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析 |
| [02_Architecture.md](./02_Architecture.md) | 架构与数据流 |
| [03_CodeMap.md](./03_CodeMap.md) | 代码地图 |

---

*所有技术结论均有代码证据支撑，详见 [wiki/_work/NOTES.md](../_work/NOTES.md)。*
