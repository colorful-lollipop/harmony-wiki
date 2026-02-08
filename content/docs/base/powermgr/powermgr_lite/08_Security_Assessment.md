# 安全风险评审

> **目的**: 基于代码证据的安全风险分析与可被利用点评估
> **适用范围**: 安全审计员、安全工程师、架构师
> **阅读时间**: 30分钟

---

## 概述

### 威胁模型

```
外部输入 (应用/JS)
    ↓
[Framework 层 - 受信任]
    ↓
[IPC 边界 - SAMGR Lite]
    ⚠️ 无权限验证
    ⚠️ 输入验证不足
    ↓
[Services 层 - 受信任]
    ⚠️ 信任 IPC 数据
    ⚠️ 缺少边界检查
    ↓
[平台层 - 最信任]
    ✓ 固定路径
    ⚠️ 无输入验证
    ↓
[内核 - 最信任]
    ✓ 无输入验证
```

### 攻击面清单

| 攻击面 | 组件 | 访问方式 | 权限检查 |
|--------|------|----------|----------|
| C API (running_lock.h) | Framework | ❌ 无 |
| C API (power_manage.h) | Framework | ❌ 无 |
| IPC 边界 | SAMGR Lite | ❌ 无 |
| JS API (battery) | JS 引擎 | ❌ 无 |
| 内核接口 | Platform | ❌ 无 |

---

## 关键漏洞

### 漏洞 1: 权限检查缺失 (严重)

**位置**: `services/src/power_manage_feature.c:82-98`

**证据**:
```c
// services/src/power_manage_feature.c:82
void OnSuspendDevice(IUnknown *iUnknown, SuspendDeviceType reason, BOOL suspendImmed)
{
    // TODO: It should check if the calling pid has permission to suspend/wakeup device
    SuspendController::DisableSuspend();
}

// services/src/power_manage_feature.c:92
void OnWakeupDevice(IUnknown *iUnknown, WakeupDeviceType reason, const char* details)
{
    // TODO: It should check if the calling pid has permission to suspend/wakeup device
    // 唤醒逻辑...
}
```

**可利用路径**:
```
恶意应用
    ↓
调用 SuspendDevice(reason, FALSE)
    ↓
[服务无条件执行]
    ↓
设备挂起 (无权限检查)
```

**影响**:
- 任何可访问 IPC 接口的进程都可以挂起或唤醒设备
- 可能导致:
  - 拒绝服务 (DoS)
  - 数据丢失 (强制挂起)
  - 电池耗尽 (频繁唤醒)
  - 用户隐私泄露 (在用户不知情时唤醒)

**修复建议**:
1. **立即**: 在 `OnSuspendDevice()` 和 `OnWakeupDevice()` 中添加权限检查:
   ```c
   // 伪代码
   if (!HasPermission(getpid(), "ohos.permission.POWER_MANAGE")) {
       HILOGE("Permission denied");
       return;
   }
   ```
2. **验证**: 获取调用者 PID (从 IPC 数据或系统调用)
3. **长期**: 实现 DAC (Discretionary Access Control) 或能力 (Capability) 系统

**严重程度**: 🔴 **严重**

---

### 漏洞 2: IPC 输入验证不足 (中等)

**位置**: `services/src/power/small/power_manage_feature_impl.c:66-115`

**证据**:
```c
// services/src/power/small/power_manage_feature_impl.c:68-70
int32_t AcquireInvoke(IServerProxy *iProxy, IpcIo *req, IpcIo *reply)
{
    uint32_t len;
    ReadUint32(req, &len);  // 从 IPC 读取长度
    RunningLockEntry *entry = (RunningLockEntry*)ReadBuffer(req, len);  // 直接转换,无验证!
    // ...
}
```

**问题**:
- `len` 来自不受信任的 IPC 数据
- `ReadBuffer()` 返回指针直接转换为 `RunningLockEntry*`
- 无边界检查确保 `len == sizeof(RunningLockEntry)`
- 无结构版本或魔数验证

**可利用路径**:
```
恶意应用
    ↓
构造恶意 IPC 消息:
  - len = 超大值 (如 0xFFFFFFFF)
  - 数据包含恶意内容
    ↓
发送到 PowerManageFeature
    ↓
[FeatureInvoke 无验证]
    ↓
ReadBuffer() 返回无效指针
    ↓
强制转换:
RunningLockEntry *entry = (RunningLockEntry*)0x...  // 可能导致:
    - 解引用 NULL/无效指针 → 崩溃
    - 栈溢出 (如果 len 小但指针计算溢出)
    - 任意内存读取 (如果指针可控)
```

**影响**:
- 服务崩溃 (拒绝服务)
- 信息泄露 (读取任意内存)
- 代码执行 (如果可以控制回调指针)

**修复建议**:
1. **立即**: 添加长度验证:
   ```c
   if (len != sizeof(RunningLockEntry)) {
       HILOGE("Invalid entry size: %u, expected %zu", len, sizeof(RunningLockEntry));
       return EC_INVALID;
   }
   ```
2. **验证**: 添加结构魔数:
   ```c
   typedef struct {
       uint32_t magic;  // 0x504D524C ("PM\0")
       RunningLockEntry entry;
   } ValidatedEntry;
   ```
3. **加固**: 限制最大 IPC 消息大小 (在 SAMGR Lite 配置中)

**严重程度**: 🟡 **中等**

---

### 漏洞 3: 锁名称长度未限制 (低)

**位置**: `frameworks/src/running_lock.c:86-93`

**证据**:
```c
// frameworks/src/running_lock.c:86
const RunningLock *CreateRunningLock(const char *name, ...)
{
    RunningLock *lock = (RunningLock *)malloc(sizeof(RunningLock));
    if (lock == NULL) {
        return NULL;
    }
    // 无长度验证!
    (void)strcpy_s(lock->name, name, RUNNING_LOCK_NAME_LEN);  // 依赖 strcpy_s 截断
    // ...
}
```

**问题**:
- `name` 参数在 `strcpy_s()` 之前未验证长度
- 如果 `name` 极长 (如 1MB),可能在堆分配时触发问题
- 虽然后 `strcpy_s()` 会截断,但 `name` 参数本身可能来自未受信任来源

**可利用路径**:
```
恶意应用
    ↓
调用 CreateRunningLock(
    "A" * 100000,  // 100KB 名称
    type,
    flag
)
    ↓
[Framework 处理]
    ↓
malloc(sizeof(RunningLock))  // OK
    ↓
strcpy_s(lock->name, name, RUNNING_LOCK_NAME_LEN)  // 截断,但 name 参数已处理
    ↓
可能问题:
  - 栈缓冲区溢出 (如果 name 从栈传入)
  - DoS (超长字符串处理)
```

**影响**:
- 轻微: `strcpy_s` 会截断
- 中等: 如果 `name` 来自栈,可能栈溢出
- DoS: 超长字符串处理消耗 CPU

**修复建议**:
1. **立即**: 添加长度验证:
   ```c
   if (name == NULL || strlen(name) >= RUNNING_LOCK_NAME_LEN) {
       HILOGE("Invalid lock name");
       return NULL;
   }
   ```
2. **加固**: 使用 `strncpy_s()` 显式长度检查

**严重程度**: 🟢 **低**

---

### 漏洞 4: 整数溢出 (低)

**位置**: `frameworks/src/small/power_manage.c:50-51`

**证据**:
```c
// frameworks/src/small/power_manage.c:50-51
static PowerManageProxy* CreateClient(IPCIo *request, ...)
{
    uint32_t size = request->dataSz;  // 从 IPC 数据读取
    uint32_t len = size + sizeof(PowerManageProxyEntry);  // 可能溢出!
    PowerManageProxyEntry *proxy = (PowerManageProxyEntry *)malloc(len);
    // ...
}
```

**问题**:
- `size` 来自不受信任的 IPC 数据
- 如果 `size` 接近 `UINT32_MAX`,`size + sizeof()` 可能溢出为小值
- `malloc(len)` 会分配小于预期的内存
- 后续 `memcpy()` 可能导致堆溢出

**可利用路径**:
```
恶意应用
    ↓
构造恶意 IPC 消息:
  - dataSz = 0xFFFFFF00  // 接近 UINT32_MAX
    ↓
发送到 PowerManageFeature
    ↓
[CreateClient 处理]
    ↓
size = 0xFFFFFF00
len = 0xFFFFFF00 + sizeof(...)  // 溢出为小值 (如 0x100)
    ↓
malloc(0x100)  // 分配 256 字节
    ↓
memcpy(proxy, request->data, 0xFFFFFF00)  // 从 request->data 复制 2GB!
    ↓
严重堆溢出
```

**影响**:
- 堆溢出
- 服务崩溃
- 可能代码执行 (如果可以控制内存布局)

**修复建议**:
1. **立即**: 添加溢出检查:
   ```c
   if (size > UINT32_MAX - sizeof(PowerManageProxyEntry)) {
       HILOGE("Invalid data size");
       return NULL;
   }
   ```
2. **加固**: 限制最大 IPC 消息大小
3. **使用安全函数**: 使用 `memcpy_s()` 代替 `memcpy()` (已使用 ✓)

**严重程度**: 🟡 **中等**

---

### 漏洞 5: 计时器回调不安全 (低)

**位置**: `utils/src/power_mgr_timer_util.c:62-71`

**证据**:
```c
// utils/src/power_mgr_timer_util.c:62-71
static void TimerHandle(union sigval value)
{
    PowerMgrTimerInfo *info = (PowerMgrTimerInfo *)value.sival_ptr;

    // 在信号上下文中访问共享数据,无同步!
    if (info != NULL) {
        info->timerCb(info->data);  // 无互斥锁保护
    }
}
```

**问题**:
- POSIX 计时器回调在信号上下文执行
- `info->timerCb` 和 `info->data` 是共享数据
- 无互斥锁保护
- 如果同时修改 `info` 结构,可能导致竞态

**可利用路径**:
```
应用 A
    ↓
PowerMgrTimerStart(timer1, callback1, data1)
PowerMgrTimerStart(timer2, callback2, data2)
    ↓
[并发修改]
    应用 A 同时修改 timer1 和 timer2
    ↓
[信号上下文]
    ↓
TimerHandle() 访问 info
    ↓
可能:
  - 解引用 NULL (timer 已销毁)
  - 调用已释放的回调
  - 竞态条件 → 未定义行为
```

**影响**:
- 崩溃
- 悬空指针解引用
- 调用已释放回调

**修复建议**:
1. **立即**: 在 `TimerHandle()` 中添加锁:
   ```c
   pthread_mutex_lock(&g_timerMutex);
   if (info != NULL && info->valid) {
       info->timerCb(info->data);
   }
   pthread_mutex_unlock(&g_timerMutex);
   ```
2. **设计**: 添加 `info->valid` 标志
3. **生命周期**: 确保先停止计时器再销毁

**严重程度**: 🟢 **低**

---

## 正面安全实践

### 使用的安全模式

| 模式 | 位置 | 证据 |
|------|------|------|
| 安全 C 函数 | 多处 | `strcpy_s()`, `memcpy_s()`, `memset_s()` |
| 空指针检查 | 多处 | `if (ptr == NULL) return error;` |
| 类型验证 | services/src/power/small/power_manage_feature_impl.c | `0 <= funcId < POWERMANAGE_FUNCID_BUTT` |
| 互斥锁保护 | 多处 | `pthread_mutex_t g_mutex` |
| 固定文件路径 | services/src/power/small/running_lock_handler.c | `/proc/power/power_lock`, `/proc/power/power_unlock` |
| 数组边界检查 | services/src/running_lock_mgr.c | `if (type < RUNNINGLOCK_BUTT)` |
| 线程安全 | 多处 | 大部分接口使用 mutex |

### 不存在的安全机制 (缺失)

| 机制 | 位置 | 状态 |
|------|------|------|
| 权限检查 | services/src/power_manage_feature.c | ❌ 未实现 (有 TODO) |
| IPC 长度限制 | services/src/power/small/power_manage_feature_impl.c | ❌ 未实现 |
| IPC 结构魔数 | services/src/power/small/power_manage_feature_impl.c | ❌ 未实现 |
| ASLR / PIE | BUILD.gn | ⚠️ 待验证 |
| Stack Canaries | BUILD.gn | ⚠️ 待验证 |
| FORTIFY_SOURCE | BUILD.gn | ⚠️ 待验证 |
| SELinux | 小系统 | ⚠️ 待验证 |

---

## 信任边界分析

### 边界 1: 应用 → Framework
**输入验证**:
- ✓ 参数非空检查
- ⚠️ 类型范围验证 (仅部分)
- ❌ 长度限制 (缺失)
- ❌ 来源验证 (缺失)

### 边界 2: Framework → IPC (SAMGR Lite)
**输入验证**:
- ❌ 权限检查 (缺失)
- ❌ 调用者身份验证 (缺失)
- ✓ 序列化 (IpcIo)

### 边界 3: IPC → Service
**输入验证**:
- ✓ 函数 ID 范围检查
- ❌ 长度验证 (缺失)
- ❌ 结构魔数 (缺失)
- ❌ 调用者身份验证 (缺失)

### 边界 4: Service → Platform
**输入验证**:
- ✓ 固定路径
- ✓ 内核接口封装
- ❌ 名称长度限制 (缺失)
- ⚠️ 内核验证 (依赖平台)

---

## 安全加固建议

### 短期 (1-2 周)
1. **实现权限检查**: 在 `OnSuspendDevice()` 和 `OnWakeupDevice()` 中
2. **添加 IPC 长度验证**: 限制 `len` 参数最大值
3. **添加结构魔数**: 验证 IPC 数据完整性
4. **添加名称长度验证**: 在 `CreateRunningLock()` 中

### 中期 (1-2 月)
1. **实现 IPC 调用者身份**: 从 IPC 数据获取 PID/Token
2. **添加日志审计**: 记录所有挂起/唤醒操作 (包含调用者)
3. **加固计时器回调**: 添加互斥锁保护
4. **添加错误码**: 返回详细错误信息

### 长期 (3-6 月)
1. **实现能力系统**: 定义细粒度权限 (如 `ohos.permission.SUSPEND_DEVICE`)
2. **实现 DAC/SELinux**: 在小系统上实现强制访问控制
3. **添加模糊测试**: 对 IPC 接口进行模糊测试
4. **代码审计**: 使用静态分析工具 (如 Coverity, Clang Static Analyzer)

---

## 相关文档

- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析、信任边界、外部输入清单
- [04_NAPI_JS_API.md](04_NAPI_JS_API.md#权限要求) - API 权限要求
- [03_Architecture.md](03_Architecture.md#信任边界) - 信任边界分析
- [09_Troubleshooting.md](09_Troubleshooting.md#安全相关问题) - 安全问题排查
