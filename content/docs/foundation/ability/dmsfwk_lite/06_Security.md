# 06_Security - 安全风险评审

## 1. 威胁模型

### 1.1 系统边界与信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              外部不可信区域                                   │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         远程设备                                       │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │  │
│  │  │   恶意应用    │  │   中间人     │  │   伪造消息   │              │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│                                      ▼                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         DSoftBus 网络层                                │  │
│  │                    （加密传输，但存在被绕过风险）                         │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           本地设备（信任边界）                                │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         dmsfwk_lite                                    │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │                    输入验证层（关键边界）                          │  │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │  │  │
│  │  │  │  TLV Parser  │  │   Session    │  │  Permission  │          │  │  │
│  │  │  │  (格式检查)   │  │  (会话管理)   │  │  (权限检查)   │          │  │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘          │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  │                              │                                        │  │
│  │                              ▼                                        │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │                    核心处理层                                     │  │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │  │  │
│  │  │  │   FA Manager │  │   BMS IPC    │  │  AbilityMS   │          │  │  │
│  │  │  │              │  │              │  │     IPC      │          │  │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘          │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│                                      ▼                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         本地应用                                       │  │
│  │  ┌──────────────┐  ┌──────────────┐                                  │  │
│  │  │  正常应用     │  │  恶意本地应用  │                                  │  │
│  │  └──────────────┘  └──────────────┘                                  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面清单

| 攻击面 | 类型 | 风险等级 | 说明 |
|--------|------|----------|------|
| 网络输入 | 外部 | 高 | 来自远程设备的 TLV 消息 |
| IPC 接口 | 内部 | 中 | SAMGR 服务/Feature 接口 |
| 文件系统 | 内部 | 中 | Native AppID 文件读取 |
| 内存操作 | 内部 | 高 | TLV 解析、数据拷贝 |
| 会话管理 | 外部 | 中 | SoftBus 会话生命周期 |

## 2. 安全风险清单

### 2.1 风险 1：TLV 长度字段未严格校验

**风险等级**: 🔴 高

**证据**:
```c
// source/dmslite_parser.c:165-227
TlvErrorCode TlvBytesToNode(const uint8_t *byteBuffer, uint16_t bufLength, TlvNode **tlv) {
    // ...
    while (true) {
        uint16_t curTlvNodeLen = 0;
        errCode = TlvFillNode(nodeStartAddr, remainingLen, curNode, &curTlvNodeLen);
        // ...
        remainingLen -= curTlvNodeLen;  // 减法可能下溢
        if (remainingLen == 0) {
            break;
        }
        // ...
        nodeStartAddr += curTlvNodeLen;  // 指针递增
    }
}
```

**问题分析**:
- `remainingLen` 是 `uint16_t` 类型，减法下溢后变成极大值
- 虽然 `curTlvNodeLen` 有校验，但复杂的 TLV 嵌套可能绕过

**触发条件**:
- 构造特殊的 TLV 消息，使 `remainingLen` 计算错误

**影响**:
- 可能导致越界读取或无限循环

**修复建议**:
```c
// 添加下溢检查
if (remainingLen < curTlvNodeLen) {
    errCode = DMS_TLV_ERR_LEN;
    break;
}
remainingLen -= curTlvNodeLen;
```

### 2.2 风险 2：全局缓冲区无并发保护

**风险等级**: 🔴 高

**证据**:
```c
// source/dmslite_packet.c:42-43
static char g_buffer[PACKET_DATA_SIZE] = {0};  // 1024 字节
static uint16_t g_counter = 0;
```

```c
// source/dmslite_session.c:40-43
static int32_t g_curSessionId = INVALID_SESSION_ID;
static bool g_curBusy = false;
static time_t g_begin;
static IDmsListener *g_listener = NULL;
```

**问题分析**:
- `g_buffer` 用于打包消息，无锁保护
- `g_curSessionId`, `g_curBusy` 等状态无锁保护
- 依赖 `IsDmsBusy()` 检查避免并发，但存在竞态窗口

**触发条件**:
- 多线程同时调用 `StartRemoteAbility()`
- 超时检查与发送操作竞态

**影响**:
- 消息内容被覆盖
- 会话状态混乱
- 回调指针错误

**修复建议**:
```c
// 添加互斥锁
static pthread_mutex_t g_dmsMutex = PTHREAD_MUTEX_INITIALIZER;

int32_t StartRemoteAbility(...) {
    pthread_mutex_lock(&g_dmsMutex);
    // ... 原有逻辑
    pthread_mutex_unlock(&g_dmsMutex);
}
```

### 2.3 风险 3：字符串操作缺乏长度限制

**风险等级**: 🟡 中

**证据**:
```c
// source/dmslite_packet.c:119-136
bool MarshallString(const char *field, uint8_t type) {
    size_t sz = strlen(field) + 1;  // 无长度限制
    if (g_counter + (TYPE_FILED_LENGTH + MAX_BYTE_NUM + sz) > PACKET_DATA_SIZE) {
        return false;
    }
    // ...
}
```

```c
// source/dmslite_tlv_common.c:133-150
const char* UnMarshallString(const TlvNode *tlvHead, uint8_t nodeType) {
    // ...
    const char* value = (const char*)tlvNode->value;
    if (value[tlvNode->length - 1] != '\0') {  // 依赖 length 字段
        return "";
    }
    // 未检查 length 是否合理
}
```

**问题分析**:
- `strlen()` 无上限，超长字符串可能导致性能问题
- 解包时依赖 `tlvNode->length`，若被篡改可能导致越界

**触发条件**:
- 构造超长包名/Ability 名的 TLV 消息

**影响**:
- 性能下降（strlen 遍历）
- 潜在的越界访问

**修复建议**:
```c
// 添加最大长度限制
#define MAX_STRING_LENGTH 256

bool MarshallString(const char *field, uint8_t type) {
    size_t len = strnlen(field, MAX_STRING_LENGTH);
    if (len >= MAX_STRING_LENGTH) {
        return false;
    }
    size_t sz = len + 1;
    // ...
}
```

### 2.4 风险 4：UID 检查可被绕过（Wearable 模式）

**风险等级**: 🟡 中

**证据**:
```c
// source/dmslite_parser.c:243-254
static bool CanCall() {
#ifndef WEARABLE_PRODUCT
    uid_t callerUid = getuid();
    if (callerUid != FOUNDATION_UID && callerUid != SHELL_UID) {
        HILOGD("[Caller uid is not allowed, uid = %u]", callerUid);
        return false;
    }
#endif
    return true;
}
```

**问题分析**:
- `WEARABLE_PRODUCT` 宏定义时完全跳过 UID 检查
- 可穿戴设备上任意应用可调用 `ProcessCommuMsg()`

**触发条件**:
- 在可穿戴产品上编译运行
- 本地恶意应用直接调用接口

**影响**:
- 权限绕过，任意应用可触发远程 FA 启动流程

**修复建议**:
- 可穿戴产品也应实施适当的权限控制
- 或明确文档说明该风险

### 2.5 风险 5：Native AppID 文件路径拼接缺乏校验

**风险等级**: 🟡 中

**证据**:
```c
// source/dmslite_permission.c:208-218
if (callerInfo->uid <= MAX_NATIVE_SERVICE_UID) {
    char filePath[MAX_FILE_PATH_LEN] = {0};
    int32_t ret = sprintf_s(filePath, MAX_FILE_PATH_LEN, "%s%s%d%s", 
        NATIVE_APPID_DIR, APPID_FILE_PREFIX,
        callerInfo->uid, APPID_FILE_SUFFIX);
    // ...
    return GetBundleInfoFromFile(filePath, bundleInfo);
}
```

**问题分析**:
- `callerInfo->uid` 来自调用方，虽然检查了 `<= 99`
- 但 `sprintf_s` 的返回值检查只判断是否 `< 0`
- 若路径构造异常（如 uid 为负数），可能读取错误文件

**触发条件**:
- 构造特殊的 `callerInfo->uid`（虽然已检查范围）

**影响**:
- 可能读取非预期的 AppID 文件

**修复建议**:
```c
// 添加 uid 范围校验
if (callerInfo->uid < 0 || callerInfo->uid > MAX_NATIVE_SERVICE_UID) {
    return DMS_EC_INVALID_PARAMETER;
}
```

### 2.6 风险 6：内存分配失败处理不完整

**风险等级**: 🟡 中

**证据**:
```c
// source/dmslite_parser.c:126-136
static inline TlvNode* MallocTlvNode() {
    TlvNode *node = (TlvNode *)malloc(sizeof(TlvNode));
    if (node == NULL) {
        HILOGE("[Out of memory]");
        return NULL;
    }
    (void) memset_s(node, sizeof(TlvNode), 0x00, sizeof(TlvNode));
    return node;
}
```

**问题分析**:
- 多处 `malloc` 后未检查返回值（虽然大部分有检查）
- `memset_s` 返回值被忽略

**触发条件**:
- 系统内存耗尽

**影响**:
- 空指针解引用导致崩溃

**修复建议**:
- 统一封装内存分配函数，确保失败处理
- 检查所有 `memset_s` 返回值

### 2.7 风险 7：会话超时时间固定且较长

**风险等级**: 🟢 低

**证据**:
```c
// source/dmslite_session.c:38
#define TIMEOUT 60
```

**问题分析**:
- 超时时间固定 60 秒，不可配置
- 长时间占用会话资源，可能导致 DoS

**触发条件**:
- 恶意应用持续发起请求不完成

**影响**:
- 拒绝服务，其他应用无法使用远程启动

**修复建议**:
- 使超时时间可配置
- 或根据操作类型设置不同超时

## 3. 安全机制分析

### 3.1 已有的安全机制

| 机制 | 位置 | 效果 |
|------|------|------|
| UID 白名单 | `dmslite_parser.c:243-254` | 限制调用者身份 |
| 签名校验 | `dmslite_permission.c:63-116` | 验证应用身份 |
| TLV 格式检查 | `dmslite_parser.c:165-227` | 验证消息格式 |
| 数据大小限制 | `dmslite_session.c:37` | 限制消息大小 1024 |
| 单会话限制 | `dmslite_session.c:251-258` | 防止并发滥用 |
| 超时机制 | `dmslite_session.c:38` | 自动释放资源 |

### 3.2 安全机制局限性

| 机制 | 局限性 |
|------|--------|
| UID 白名单 | Wearable 模式下失效 |
| 签名校验 | 依赖 BMS 安全性 |
| TLV 检查 | 长度下溢风险 |
| 单会话限制 | 无锁保护，存在竞态 |

## 4. 修复建议汇总

### 4.1 高优先级修复

1. **添加互斥锁保护全局状态**
   - 文件: `dmslite_session.c`, `dmslite_packet.c`
   - 操作: 添加 `pthread_mutex_t` 保护关键区域

2. **修复 TLV 长度下溢**
   - 文件: `dmslite_parser.c`
   - 操作: 添加 `remainingLen < curTlvNodeLen` 检查

### 4.2 中优先级修复

3. **限制字符串长度**
   - 文件: `dmslite_packet.c`, `dmslite_tlv_common.c`
   - 操作: 添加 `MAX_STRING_LENGTH` 限制

4. **完善内存分配错误处理**
   - 文件: 所有使用 `malloc` 的文件
   - 操作: 统一封装，强制检查返回值

5. **可穿戴产品权限控制**
   - 文件: `dmslite_parser.c`
   - 操作: 设计适用于可穿戴产品的轻量级权限机制

### 4.3 低优先级修复

6. **可配置超时时间**
   - 文件: `dmslite_session.c`
   - 操作: 通过宏或配置文件设置超时

7. **添加更多日志审计**
   - 文件: 所有关键操作点
   - 操作: 记录调用者身份、操作结果

## 5. 检查范围与局限性

### 5.1 已检查范围

- 所有 `source/*.c` 文件（共 10 个）
- 所有 `include/*.h` 文件（共 15 个）
- 对外接口 `interfaces/innerkits/dmsfwk_interface.h`
- 构建配置 `BUILD.gn`, `bundle.json`

### 5.2 未检查范围（局限性）

| 范围 | 原因 | 影响 |
|------|------|------|
| 依赖组件实现 | 代码不在本仓库 | BMS、SoftBus 漏洞可能影响本组件 |
| 测试代码 | 按约束忽略 | 可能遗漏测试发现的问题 |
| 实际运行环境 | 无运行时分析 | 实际攻击场景可能更复杂 |
| 历史漏洞修复 | 未分析 commit 历史 | 可能遗漏回归风险 |

### 5.3 建议的进一步检查

1. **模糊测试**: 对 TLV 解析器进行 fuzzing
2. **代码审计**: 使用静态分析工具（Coverity、CodeQL）
3. **运行时分析**: 使用 AddressSanitizer 检测内存问题
4. **渗透测试**: 模拟实际攻击场景

## 6. 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构设计
- [03_Public_API](03_Public_API.md) - 对外接口
- [04_Internal_API](04_Internal_API.md) - 内部接口
