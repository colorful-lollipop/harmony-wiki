# 安全分析

## 概述

本文档对 `drivers_liteos` 仓库进行安全风险评审，重点分析 **hievent** 模块的潜在攻击面、信任边界和安全风险。

**分析范围**:
- ✅ hievent 模块 (完整开源)
- ⚠️ tzdriver 模块 (部分开源，代码不可见)
- ❌ 其他模块 (位于外部仓库)

**分析依据**: 基于源码逆向分析 (`src/*.c`, `include/*.h`)

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                    不可信区域                                │
│  ┌─────────────────────────────────────────────────────┐  │
│  │          用户空间进程 (任意 UID)                       │  │
│  └─────────────────────────────────────────────────────┘  │
│                         ▲                                   │
│                         │ open()/write()/read()            │
├─────────────────────────────────────────────────────────────┤
│                      边界: /dev/hwlog_exception             │
├─────────────────────────────────────────────────────────────┤
│                    可信区域 (内核空间)                        │
│  ┌─────────────────────────────────────────────────────┐  │
│  │          hievent 驱动 (环形缓冲区)                    │  │
│  └─────────────────────────────────────────────────────┘  │
│                         ▲                                   │
│                         │ LogBufToException()              │
├─────────────────────────────────────────────────────────────┤
│                    边界: hievent_driver.c                  │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

| 方向 | 数据类型 | 信任度 |
|-----|---------|-------|
| 用户 → 内核 | 事件数据 | 不可信，需校验 |
| 内核 → 用户 | 环形缓冲区内容 | 相对可信 |
| 内核 → 内核 | Payload 链表 | 可信 |

## 攻击面清单

| 攻击面 | 类型 | 暴露方式 | 风险等级 |
|-------|------|---------|---------|
| `/dev/hwlog_exception` | 字符设备 | 全局可读写 (0666) | 高 |
| `write()` | 系统调用 | 用户可控数据 | 高 |
| `read()` | 系统调用 | 读取内核缓冲区 | 中 |
| `HiviewHieventPutString()` | API | 字符串键值对 | 中 |
| `HiviewHieventAddFilePath()` | API | 文件路径 | 中 |
| `LOS_IsUserAddressRange()` | 检查 | 地址验证逻辑缺陷 | 高 |

## 风险清单

### 风险 1：设备节点权限过宽

**风险等级**: 🔴 高

**证据**: `hievent_driver.c:57`, `hievent_driver.c:384-385`

```c
#define DRIVER_MODE 0666

register_driver("/dev/hwlog_exception", &g_hieventFops,
                DRIVER_MODE, &g_hieventDev);
```

**问题描述**:
- 设备节点 `/dev/hwlog_exception` 使用 `0666` 权限
- **任何用户** (包括非特权用户) 都可以读写该设备
- 可能导致未授权的事件注入

**触发条件**:
```c
// 任意用户可执行
int fd = open("/dev/hwlog_exception", O_WRONLY);
write(fd, malicious_data, len);
```

**影响**:
1. 未授权用户可注入虚假事件日志
2. 可能干扰系统日志分析和故障排查
3. 占用环形缓冲区空间，影响正常事件记录

**修复建议**:
```c
// 建议改为 0660，仅 wheel/root 组可访问
#define DRIVER_MODE 0660
```

---

### 风险 2：用户地址校验逻辑缺陷

**风险等级**: 🔴 高

**证据**: `hievent_driver.c:127-134`

```c
static int HieventBufferCopy(unsigned char *dst, unsigned dstLen,
                             unsigned char *src, size_t srcLen)
{
    int retval = -1;

    size_t minLen = dstLen > srcLen ? srcLen : dstLen;

    if (LOS_IsUserAddressRange((vaddr_t)(uintptr_t)dst, minLen) &&
        LOS_IsUserAddressRange((vaddr_t)(uintptr_t)src, minLen)) {
        return retval;  // ⚠️ 逻辑错误：两个都是用户地址时返回 -1
    }
    // ...
}
```

**问题描述**:
- 检查逻辑与预期**相反**
- 当 `dst` 和 `src` **都是**用户地址时，函数直接返回 `-1`
- 当只有 **一个**是用户地址时，反而会执行复制操作

**正确逻辑** (推测):
```c
// 预期：两个都是内核地址时返回 -1（不允许内核→内核）
// 实际：两个都是用户地址时返回 -1（不允许用户→用户）
if (LOS_IsKernelAddressRange(dst) && LOS_IsKernelAddressRange(src)) {
    return retval;
}
```

**触发条件**:
- 内核内部调用 `HieventBufferCopy()` 可能失败
- 正常的内核→内核数据传输被阻止

**影响**:
1. 内核内部数据复制可能异常
2. 可能导致环形缓冲区数据损坏

**修复建议**:
```c
// 修正检查逻辑
if (LOS_IsKernelAddressRange((vaddr_t)(uintptr_t)dst) && 
    LOS_IsKernelAddressRange((vaddr_t)(uintptr_t)src)) {
    return retval;  // 两个都是内核地址才返回错误
}
```

---

### 风险 3：CHECK_CODE 强度不足

**风险等级**: 🟡 中

**证据**: `hievent_driver.h:37`, `hievent_driver.c:286-290`

```c
#define CHECK_CODE 0x7BCDABCD

// 写入时校验
int checkCode = *((int *)buffer);
if (checkCode != CHECK_CODE) {
    retval = -EINVAL;
    goto out;
}
```

**问题描述**:
- 使用固定魔数 `0x7BCDABCD` 作为校验码
- 攻击者可轻易绕过校验

**触发条件**:
```c
// 构造包含 CHECK_CODE 的恶意数据
char malicious[1024];
*(int *)malicious = 0x7BCDABCD;  // CHECK_CODE
write(fd, malicious, sizeof(malicious));
```

**影响**:
1. 校验形同虚设
2. 无法防止恶意事件注入

**修复建议**:
```c
// 方案1: 使用随机化校验码
// 方案2: 引入 HMAC 签名验证
// 方案3: 限制只有特定 UID 可写入
```

---

### 风险 4：路径遍历风险

**风险等级**: 🟡 中

**证据**: `hiview_hievent.c:270-291`

```c
#define MAX_PATH_LEN 256

static int AppendArrayItem(char **pool, int poolLen, const char *path)
{
    if (strlen(path) > MAX_PATH_LEN) {
        HWLOG_ERR("file path over max: %d", MAX_PATH_LEN);
        return -EINVAL;
    }
    // ...
    pool[i] = strdup(path);  // ⚠️ 未校验路径内容
}
```

**问题描述**:
- 仅检查路径长度，未检查路径合法性
- 可能导致路径遍历攻击

**触发条件**:
```c
// 注入恶意路径
HiviewHieventAddFilePath(event, "../../../etc/passwd");
HiviewHieventAddFilePath(event, "/proc/self/mem");
```

**影响**:
1. 可能访问敏感文件
2. 可能泄露系统信息

**修复建议**:
```c
// 1. 检查路径前缀，必须在允许目录内
// 2. 拒绝包含 ".." 的路径
// 3. 仅允许特定白名单目录
```

---

### 风险 5：字符串长度截断风险

**风险等级**: 🟢 低

**证据**: `hiview_hievent.c:234-238`

```c
#define MAX_STR_LEN (10 * 1024)

// HiviewHieventPutString()
len = strlen(value);
if (len > MAX_STR_LEN) {
    len = MAX_STR_LEN;  // ⚠️ 静默截断
}
```

**问题描述**:
- 字符串超长时静默截断，不通知调用者
- 可能导致数据丢失或格式错误

**触发条件**:
```c
// 传入超长字符串
char long_str[MAX_STR_LEN + 100];
HiviewHieventPutString(event, "key", long_str);  // 被截断
```

**影响**:
1. 重要信息可能被截断丢失
2. 可能导致 Payload 格式异常

**修复建议**:
```c
// 返回实际长度或错误码
if (len > MAX_STR_LEN) {
    return -EINVAL;  // 或返回截断后的实际长度
}
```

---

### 风险 6：NULL 指针检查不完整

**风险等级**: 🟡 中

**证据**: `hiview_hievent.c:121-136`

```c
static struct HiviewHieventPayload *HiviewHieventGetPayload(
    struct HiviewHieventPayload *head, const char *key)
{
    struct HiviewHieventPayload *p = head;

    while (p) {
        if (key && p->key) {  // ⚠️ key 和 p->key 分开检查
            if (strcmp(p->key, key) == 0) {
                return p;
            }
        }
        p = p->next;
    }
    return NULL;
}
```

**问题描述**:
- `key` 和 `p->key` 分开检查
- 但在调用处 `HiviewHieventPutIntegral()` 的参数检查可能不完整

**触发条件**:
- 传入 NULL key 值

**影响**:
1. 可能导致空指针解引用
2. 可能导致 `strcmp(NULL, ...)` crash

**修复建议**:
```c
// 统一在入口处检查
if (!key) {
    return -EINVAL;
}
```

---

## 安全加固建议

### 立即修复

| 优先级 | 问题 | 建议 |
|-------|-----|------|
| P0 | 设备权限过宽 | 改为 0660，限制可写用户 |
| P0 | 地址校验逻辑 | 修正检查条件 |
| P1 | CHECK_CODE | 引入签名验证或用户限制 |
| P1 | 路径遍历 | 添加路径白名单/前缀检查 |

### 长期改进

| 改进项 | 说明 |
|-------|------|
| 审计日志 | 记录所有写入操作，包括用户 ID |
| 速率限制 | 限制单用户/单进程的写入频率 |
| 输入过滤 | 对关键字段进行白名单校验 |
| 安全测试 | 添加模糊测试 (fuzzing) |

## 安全检查清单

- [ ] 设备节点权限是否为最小权限原则
- [ ] 用户输入是否经过完整校验
- [ ] 路径操作是否防止遍历攻击
- [ ] 字符串处理是否避免溢出和截断
- [ ] 内存分配失败是否有合理处理
- [ ] 是否有审计日志记录安全相关操作

---

*分析依据*: `hievent_driver.c`, `hiview_hievent.c`, `hievent_driver.h`, `hiview_hievent.h`
*分析方法*: 代码逆向分析 + 威胁建模
*局限性*: tzdriver 模块代码不可见，无法完整评估
