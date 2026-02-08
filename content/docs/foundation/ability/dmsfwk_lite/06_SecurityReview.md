# 安全风险评估

> dmsfwk_lite 分布式调度模块的安全风险深度分析，包含漏洞点、利用路径、影响评估及修复建议。

## 一、风险评估总览

本次安全评估覆盖 dmsfwk_lite 的核心模块，共识别以下风险类型：

| 风险类别 | 风险数量 | 高危 | 中危 | 低危 |
|---------|---------|------|------|------|
| 输入验证缺陷 | 3 | 1 | 2 | 0 |
| 内存安全问题 | 2 | 1 | 1 | 0 |
| 权限与鉴权 | 2 | 0 | 1 | 1 |
| 并发安全 | 1 | 0 | 1 | 0 |
| 逻辑漏洞 | 2 | 0 | 1 | 1 |
| **总计** | **10** | **2** | **6** | **2** |

---

## 二、输入验证缺陷

### R1：Want 字段缺失校验（中危）

**位置**：`source/dmslite_famgr.c:88-90`

**证据**：
```cpp
if (want == NULL || want->element == NULL || callerInfo == NULL) {
    return DMS_EC_INVALID_PARAMETER;
}
// 后续代码直接使用 want->element->bundleName 等字段
```

**问题描述**：
- 仅检查指针是否为 NULL，未检查字段内容是否有效
- `bundleName`、`abilityName`、`deviceId` 可能为空字符串
- 未检查字符串长度是否超出缓冲区限制

**触发路径**：
```
恶意应用 → StartRemoteAbility()
→ want->element->bundleName = ""
→ 消息封装为 TLV
→ 远端设备尝试启动空包名 FA
→ BMS 返回错误或启动未知 FA
```

**影响评估**：
- 可用性影响：导致启动失败，错误信息泄露包名信息
- 无法直接提权，但可探测远端设备安装的应用

**修复建议**：
```cpp
// 添加字段内容校验
if (want->element->bundleName == NULL || 
    strlen(want->element->bundleName) == 0 ||
    strlen(want->element->bundleName) > MAX_BUNDLE_NAME_LEN) {
    return DMS_EC_INVALID_PARAMETER;
}
```

---

### R2：Want 数据长度边界校验不足（中危）

**位置**：`source/dmslite_famgr.c:132-134`

**证据**：
```cpp
if (want->data != NULL && want->dataLength > 0) {
    RAWDATA_MARSHALL_HELPER(RawData, CALLER_PAYLOAD, want->data, want->dataLength);
}
```

**问题描述**：
- `want->dataLength` 类型为 `int32_t`，但只与 0 比较
- 未检查是否为负数或超过 `MAX_DMS_MSG_LENGTH`（1024）
- 超过限制时 `RAWDATA_MARSHALL_HELPER` 可能返回 false，但无错误码区分

**触发路径**：
```
恶意应用 → StartRemoteAbility()
→ want->dataLength = -1 (或极大值)
→ RAWDATA_MARSHALL_HELPER 返回 false
→ 错误信息不明确，难以调试
```

**影响评估**：
- 拒绝服务：恶意应用可触发意外的失败返回
- 信息泄露：错误处理路径可能泄露内部状态

**修复建议**：
```cpp
if (want->data != NULL) {
    if (want->dataLength < 0 || want->dataLength > MAX_DMS_MSG_LENGTH) {
        return DMS_EC_INVALID_PARAMETER;
    }
    // ...
}
```

---

### R3：TLV 节点类型无范围检查（高危）

**位置**：`source/dmslite_parser.c:89`

**证据**：
```cpp
// TLV 节点定义
typedef struct TlvNode {
    uint8_t type;        // 1 字节，无范围检查
    uint16_t length;      // 2 字节
    const uint8_t *value;
    struct TlvNode *next;
} TlvNode;

// 解析时直接赋值
node->type = *byteBuffer;  // 直接使用输入值
```

**问题描述**：
- `type` 字段使用 `uint8_t`，但仅在解析时做顺序检查
- 攻击者可发送任意类型的 TLV 节点
- `UnMarshallString` 等函数不验证 `type` 是否在预期范围内

**触发路径**：
```
远端攻击者 → 构造畸形 TLV 消息
→ type = 0xFF (预期范围外)
→ UnMarshallUint16(COMMAND_ID) 返回错误值
→ switch-case 落入 default 分支
→ 返回 DMS_EC_UNKNOWN_COMMAND_ID
```

**影响评估**：
- 可利用性：低（目前处理安全）
- 潜在风险：未来添加新命令时，未知类型可能被错误处理

**修复建议**：
```cpp
// 添加类型白名单校验
static const uint8_t ALLOWED_TYPES[] = {
    COMMAND_ID, DMS_VERSION, CALLEE_BUNDLE_NAME,
    CALLEE_ABILITY_NAME, CALLER_SIGNATURE, CALLER_PAYLOAD
};

bool IsValidTlvType(uint8_t type) {
    for (uint8_t allowed : ALLOWED_TYPES) {
        if (type == allowed) return true;
    }
    return false;
}
```

---

## 三、内存安全问题

### R4：TLV 长度字段整数溢出（中危）

**位置**：`source/dmslite_parser.c:49-78`

**证据**：
```cpp
static TlvErrorCode TlvBytesToLength(const uint8_t *bytesBuffer, uint16_t bufLength,
    uint16_t *length, uint8_t *bytesNumber)
{
    uint8_t bytesNum = 0;
    uint16_t len = 0;
    // 循环处理变长长度编码
    for (uint8_t i = 0; i < bufLength; i++) {
        TlvByteToLength(bytesBuffer[i], &len);  // 累计位移
        bytesNum++;
        // 检查是否超过最大字节数
        if (bytesNum >= TLV_MAX_LENGTH_BYTES) {  // TLV_MAX_LENGTH_BYTES = 2
            return DMS_TLV_ERR_LEN;
        }
    }
    // ...
}
```

**问题描述**：
- 长度计算使用 `uint16_t`，但位移操作可能溢出
- `TlvByteToLength` 实现：`len = (len << 7) | (byte & 0x7F)`
- 当输入为 `0xFF 0xFF` 时，`len` 值可能超出预期范围

**触发路径**：
```
远端攻击者 → 构造 TLV 消息
→ 长度字段 = 0xFF 0x7F (16383)
→ 长度计算溢出为小值
→ 解析器认为长度有效
→ 后续 memcpy 可能访问越界内存
```

**影响评估**：
- 可利用性：中（需要精心构造消息）
- 影响：内存读取越界，信息泄露或代码执行

**修复建议**：
```cpp
static TlvErrorCode TlvBytesToLength(...) {
    uint16_t len = 0;
    uint8_t bytesNum = 0;
    for (uint8_t i = 0; i < bufLength; i++) {
        if (i > 0 && ((bytesBuffer[i] & 0x80) == 0)) {
            return DMS_TLV_ERR_LEN;  // 非变长编码
        }
        TlvByteToLength(bytesBuffer[i], &len);
        bytesNum++;
        if (bytesNum > TLV_MAX_LENGTH_BYTES) {
            return DMS_TLV_ERR_LEN;
        }
        // 检查 len 是否超过缓冲区限制
        if (len > MAX_DMS_MSG_LENGTH) {
            return DMS_TLV_ERR_LEN;
        }
    }
    // ...
}
```

---

### R5：动态内存分配失败处理不足（高危）

**位置**：`source/dmslite_famgr.c:52-56, 144-146`

**证据**：
```cpp
// 位置 1
RequestData *reqdata = (RequestData *)DMS_ALLOC(sizeof(RequestData));
if (reqdata == NULL) {
    HILOGE("[mem alloc error!]");
    return DMS_EC_FAILURE;
}

// 位置 2
CallerInfo *callerData = (CallerInfo *)DMS_ALLOC(sizeof(CallerInfo));
if (callerData == NULL) {
    return DMS_EC_FAILURE;
}
```

**问题描述**：
- 内存分配失败仅返回通用错误码 `DMS_EC_FAILURE`
- 未区分不同阶段的失败，可能导致状态不一致
- 资源泄露：`reqdata` 分配失败时，上下文已分配的资源未释放

**触发路径**：
```
攻击者 → 耗尽设备内存
→ DMS_ALLOC 返回 NULL
→ dmsfwk_lite 返回 DMS_EC_FAILURE
→ 内存压力持续，设备拒绝服务
```

**影响评估**：
- 可利用性：低（需要本地权限）
- 影响：拒绝服务，内存耗尽导致系统不稳定

**修复建议**：
```cpp
// 使用清理函数封装资源释放
int32_t StartRemoteAbilityInner(...) {
    RequestData *reqdata = (RequestData *)DMS_ALLOC(sizeof(RequestData));
    if (reqdata == NULL) {
        return DMS_EC_NO_MEMORY;  // 使用专用错误码
    }
    // ...
    if (FillRequestData(reqdata, ...) != DMS_EC_SUCCESS) {
        FreeRequestData(reqdata->want, reqdata->callerInfo);
        DMS_FREE(reqdata);
        return DMS_EC_FAILURE;
    }
    // ...
}
```

---

## 四、权限与鉴权

### R6：本地调用者身份验证依赖外部系统（中危）

**位置**：`source/dmslite_permission.c:75-94`

**证据**：
```cpp
uid_t callerUid = getuid();
if (callerUid == FOUNDATION_UID) {
    /* inner-process mode */
    struct BmsServerProxy *bmsInterface = NULL;
    if (!GetBmsInterface(&bmsInterface)) {
        return DMS_EC_GET_BMS_FAILURE;
    }
    errCode = bmsInterface->GetBundleInfo(...);
} else if (callerUid == SHELL_UID) {
    /* inter-process mode (mainly called in xts testsuit process started by shell) */
    errCode = GetBundleInfo(...);
} else {
    errCode = EC_FAILURE;
}
```

**问题描述**：
- `getuid()` 仅验证调用者进程的用户 ID
- 未验证调用者进程的完整性或签名
- SHELL_UID 进程可直接调用，存在权限绕过风险

**触发路径**：
```
恶意应用（UID != FOUNDATION_UID && != SHELL_UID）
→ 直接调用 GetBundleInfo（如果导出）
→ 获取任意应用的签名信息
→ 利用签名信息伪造跨设备调用
```

**影响评估**：
- 可利用性：中（需要特定 UID）
- 影响：权限提升，可伪造合法应用身份

**修复建议**：
```cpp
// 添加调用者身份验证
if (callerUid != FOUNDATION_UID && callerUid != SHELL_UID) {
    HILOGE("[Unauthorized caller uid = %d]", callerUid);
    return DMS_EC_CHECK_PERMISSION_FAILURE;
}
```

---

### R7：Native 应用签名文件路径可预测（低危）

**位置**：`source/dmslite_permission.c:210-218`

**证据**：
```cpp
char filePath[MAX_FILE_PATH_LEN] = {0};
int32_t ret = sprintf_s(filePath, MAX_FILE_PATH_LEN, "%s%s%d%s", 
    NATIVE_APPID_DIR, APPID_FILE_PREFIX, 
    callerInfo->uid, APPID_FILE_SUFFIX);
// NATIVE_APPID_DIR = "/system/native_appid/"
// APPID_FILE_PREFIX = "uid_"
// APPID_FILE_SUFFIX = "_appid"
```

**问题描述**：
- 路径格式固定：`/system/native_appid/uid_<uid>_appid`
- 可预测的文件名，便于枚举和探测
- 文件读取操作未验证文件是否在预期目录内

**触发路径**：
```
攻击者 → 遍历 /system/native_appid/ 目录
→ 读取 uid_*_appid 文件
→ 获取 native 应用的签名信息
→ 伪造 native 应用身份
```

**影响评估**：
- 可利用性：低（需要系统权限读取 /system 目录）
- 影响：信息泄露，用于进一步攻击

**修复建议**：
```cpp
// 添加文件路径验证
char expectedPath[MAX_FILE_PATH_LEN];
sprintf_s(expectedPath, MAX_FILE_PATH_LEN, "%s%s%d%s", 
    NATIVE_APPID_DIR, APPID_FILE_PREFIX, callerInfo->uid, APPID_FILE_SUFFIX);

if (strncmp(filePath, NATIVE_APPID_DIR, strlen(NATIVE_APPID_DIR)) != 0) {
    return DMS_EC_FAILURE;
}
```

---

## 五、并发安全

### R8：全局状态非线程安全（中危）

**位置**：`source/dmslite_session.c:40-43`

**证据**：
```cpp
static int32_t g_curSessionId = INVALID_SESSION_ID;
static bool g_curBusy = false;
static time_t g_begin;
static IDmsListener *g_listener = NULL;
```

**问题描述**：
- 4 个全局变量用于管理会话状态
- 无互斥锁保护，并发访问可能导致竞态条件
- 同时多个远程启动请求可能导致状态混乱

**触发路径**：
```
场景 1：竞态条件
Thread A: 检查 g_curSessionId == INVALID_SESSION_ID
Thread B: 设置 g_curSessionId = 新会话
Thread A: 使用 g_curSessionId（已过期）
→ 发送到错误会话或丢失消息

场景 2：回调重复调用
Thread A: InvokeCallback() 被调用
Thread B: 另一个请求覆盖 g_listener
→ 回调被调用两次或调用错误的回调
```

**影响评估**：
- 可利用性：中（需要并发控制）
- 影响：消息丢失、回调错乱、状态不一致

**修复建议**：
```cpp
// 添加互斥锁保护
static pthread_mutex_t g_dmsMutex = PTHREAD_MUTEX_INITIALIZER;

void InvokeCallback(const void *data, int32_t result) {
    pthread_mutex_lock(&g_dmsMutex);
    g_curBusy = false;
    if (g_listener == NULL || g_listener->OnResultCallback == NULL) {
        pthread_mutex_unlock(&g_dmsMutex);
        return;
    }
    IDmsListener *listener = g_listener;
    g_listener = NULL;
    pthread_mutex_unlock(&g_dmsMutex);
    
    listener->OnResultCallback(data, result);
}
```

---

## 六、逻辑漏洞

### R9：会话超时后状态未完全清理（低危）

**位置**：`source/dmslite_session.c:244-258`

**证据**：
```cpp
static bool IsTimeout(void)
{
    time_t now = time(NULL);
    HILOGI("[IsTimeout diff %f]", difftime(now, g_begin));
    return ((int)difftime(now, g_begin)) - TIMEOUT >= 0;  // TIMEOUT = 60
}

bool IsDmsBusy(void)
{
    if (g_curBusy && IsTimeout() && g_curSessionId >= 0) {
        CloseDMSSession();
    }
    return g_curBusy;
}
```

**问题描述**：
- 超时检测在 `IsDmsBusy()` 中触发，由外部调用
- 若外部未调用 `IsDmsBusy()`，超时后会话不会关闭
- `CloseDMSSession()` 可能失败，但状态已改变

**触发路径**：
```
正常流程：
应用 → StartRemoteAbility()
→ 等待回调（60秒内）
→ 超时，调用 IsDmsBusy()
→ 关闭会话，清理状态

异常流程：
应用 → StartRemoteAbility()
→ 不调用 IsDmsBusy()
→ 会话保持打开状态
→ 资源泄露，可利用进行拒绝服务
```

**影响评估**：
- 可利用性：低（需要应用配合）
- 影响：资源泄露，累积后拒绝服务

**修复建议**：
```cpp
// 添加后台超时检测线程或定时器
static void *TimeoutMonitorThread(void *arg) {
    while (1) {
        sleep(10);
        pthread_mutex_lock(&g_dmsMutex);
        if (g_curBusy && IsTimeout()) {
            CloseDMSSession();
            InvokeCallback(NULL, DMS_EC_FAILURE);
        }
        pthread_mutex_unlock(&g_dmsMutex);
    }
    return NULL;
}
```

---

### R10：错误处理不完整导致信息泄露（中危）

**位置**：`source/dmslite_famgr.c:66, 77-81`

**证据**：
```cpp
// 位置 1
HILOGE("[FillRequestData failed]");  // 不返回具体错误原因

// 位置 2
HILOGD("[StartRemoteAbilityInner SendRequest errCode = %d]", result);
// 使用 HILOGD 而非 HILOGE，可能被日志级别过滤
```

**问题描述**：
- 错误日志使用不同级别，可能导致部分错误难以追溯
- 未返回详细的错误原因，难以调试和安全分析
- `memset_s` 失败后直接返回，未清理已分配资源

**触发路径**：
```
开发者 → 遇到错误
→ 查看日志
→ 只有部分错误被记录
→ 难以定位根因
→ 安全隐患难以及时发现
```

**影响评估**：
- 可利用性：低（需要本地调试）
- 影响：调试困难，安全问题难以及时发现

**修复建议**：
```cpp
// 统一错误日志级别
HILOGE("[FillRequestData failed][errCode=%d][reason=%s]", 
    ret, GetErrorReason(ret));

// 添加详细错误原因
const char* GetErrorReason(int32_t errCode) {
    switch (errCode) {
        case DMS_EC_FAILURE: return "memset_s failed";
        case DMS_EC_NO_MEMORY: return "allocation failed";
        default: return "unknown";
    }
}
```

---

## 七、风险汇总表

| ID | 风险名称 | 类型 | 风险等级 | 可用性 | 提权可能 | 修复优先级 |
|----|---------|------|---------|-------|---------|-----------|
| R1 | Want 字段缺失校验 | 输入验证 | 中 | 低 | 否 | P1 |
| R2 | 数据长度边界不足 | 输入验证 | 中 | 中 | 否 | P1 |
| R3 | TLV 类型无范围检查 | 输入验证 | 高 | 低 | 否 | P0 |
| R4 | TLV 长度溢出 | 内存安全 | 中 | 中 | 可能 | P0 |
| R5 | 内存分配失败处理 | 内存安全 | 高 | 高 | 否 | P0 |
| R6 | 本地身份验证缺陷 | 权限 | 中 | 低 | 可能 | P1 |
| R7 | 路径可预测 | 权限 | 低 | 低 | 可能 | P2 |
| R8 | 全局状态非线程安全 | 并发 | 中 | 中 | 否 | P1 |
| R9 | 超时状态未清理 | 逻辑 | 低 | 中 | 否 | P2 |
| R10 | 错误处理不完整 | 逻辑 | 中 | 低 | 否 | P1 |

---

## 八、修复优先级说明

| 优先级 | 含义 | 处理时限 |
|-------|------|---------|
| **P0** | 高危风险，必须修复 | 1 周内 |
| **P1** | 中危风险，建议修复 | 2 周内 |
| **P2** | 低危风险，择机修复 | 1 个月内 |

---

## 九、相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 项目概览 |
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析 |
| [04_Interface.md](./04_Interface.md) | 对外接口 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计 |

---

*文档版本：v1.0*  
*最后更新：2026-02-07*  
*评估方法：代码审计 + 静态分析*
