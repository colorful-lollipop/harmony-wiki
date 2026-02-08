# 安全风险评估

> 最后更新：2026-02-07
> 版本：v3.0.0
> 评估范围：主要源代码（排除 test/ 目录）

## 6.1 评估概述

### 评估方法

本安全风险评估基于以下方法论：

| 评估维度 | 说明 |
|----------|------|
| **代码审计** | 静态分析源代码，识别潜在安全漏洞 |
| **架构分析** | 分析模块间调用关系和信任边界 |
| **威胁建模** | 识别攻击面和潜在威胁 |
| **渗透测试思路** | 从攻击者角度分析可利用路径 |

### 评估范围

| 包含 | 不包含 |
|------|--------|
| `interfaces/` | `test/` |
| `services/` | 第三方依赖 |
| `oem_property/` | 运行时动态加载的插件 |
| `baselib/` | 操作系统内核 |

---

## 6.2 输入验证缺陷

### R1: MessageParcel 反序列化边界检查不充分

**风险等级**：中危

**位置**：`services/sa/standard/dslm_ipc_process.cpp:62-80`

**证据**：

```cpp
// services/sa/standard/dslm_ipc_process.cpp:62-80
int32_t DslmIpcProcess::DslmGetRequestFromParcel(
    MessageParcel &data, 
    DeviceIdentify &identify,
    RequestOption &option,
    sptr<IRemoteObject> &callback,
    uint32_t &cookie)
{
    // 从 data 读取 DeviceIdentify
    uint32_t len = data.ReadUint32();  // [风险点] len 未验证
    if (len > DEVICE_ID_MAX_LEN) {     // [防护措施] 检查上限
        return ERR_INVALID_LEN_PARA;
    }
    // 后续使用 len 作为 ReadBuffer 的长度参数
    uint32_t readLen = data.ReadBuffer(&identify.identity, len);
    if (readLen != len) {
        return ERR_IPC_RET_PARCEL_ERR;
    }
    identify.length = len;
    
    option.challenge = data.ReadUint64();
    option.timeout = data.ReadUint32();
    option.extra = data.ReadUint32();
    
    callback = data.ReadRemoteObject();  // [风险点] 未验证 callback 有效性
    cookie = data.ReadUint32();
    
    return SUCCESS;
}
```

**触发路径**：

```
恶意应用进程
    │
    ▼
构造畸形 MessageParcel（len > DEVICE_ID_MAX_LEN）
    │
    ▼
调用 IPC 接口 RequestDeviceSecurityLevel()
    │
    ▼
DslmGetRequestFromParcel() 解析数据
    │
    ▼
边界检查通过（已有上限检查）
    │
    ▼
但 ReadBuffer() 返回值未充分检查
```

**影响评估**：

| 维度 | 评估 |
|------|------|
| **可利用性** | 中 - 需要 Binder 访问权限 |
| **权限提升** | 无 - 无法直接提升权限 |
| **数据泄露** | 可能读取越界内存 |
| **服务可用性** | 可能导致服务崩溃 |

**修复建议**：

```cpp
// 改进的边界检查
uint32_t len = data.ReadUint32();
if (len == 0 || len > DEVICE_ID_MAX_LEN) {
    SECURITY_LOG_ERROR("invalid identify length: %{public}u", len);
    return ERR_INVALID_LEN_PARA;
}

uint8_t *buffer = static_cast<uint8_t *>(malloc(len));
if (buffer == NULL) {
    return ERR_NO_MEMORY;
}

uint32_t readLen = data.ReadBuffer(buffer, len);
if (readLen != len) {
    free(buffer);
    return ERR_IPC_RET_PARCEL_ERR;
}

// 使用后立即擦除敏感数据
memset_s(buffer, len, 0, len);
free(buffer);
```

---

### R2: JSON 解析空指针风险

**风险等级**：中危

**位置**：`oem_property/common/dslm_credential_utils.c:89-150`

**证据**：

```c
// oem_property/common/dslm_credential_utils.c:89-150
int32_t ParseDslmCredential(
    const DslmCredBuff *credBuff, 
    DslmCredInfo *credInfo)
{
    // 解析 JWS 头部
    JSON *header = cJSON_ParseWithLength(
        credBuff->credBuff, 
        credBuff->credBuffSize);  // [风险点] credBuff 可能为 NULL
    
    if (header == NULL) {
        SECURITY_LOG_ERROR("parse header failed");
        return ERR_JSON_ERR;
    }
    
    // 获取类型字段
    JSON *typeJson = cJSON_GetObjectItem(header, "type");
    if (typeJson == NULL) {  // [防护措施] 存在空指针检查
        SECURITY_LOG_ERROR("no type field");
        cJSON_Delete(header);
        return ERR_JSON_ERR;
    }
    
    // 获取载荷
    JSON *payload = cJSON_Parse(credBuff->payload);
    // ...
}
```

**触发路径**：

```
攻击者
    │
    ▼
构造恶意 JWS 凭证 (credBuff = NULL 或畸形格式)
    │
    ▼
发送凭证请求
    │
    ▼
ParseDslmCredential() 解析
    │
    ▼
cJSON_ParseWithLength() 失败返回 NULL
    │
    ▼
后续访问 NULL 指针 → 崩溃
```

**影响评估**：

| 维度 | 评估 |
|------|------|
| **可利用性** | 中 - 需要能够发送凭证 |
| **权限提升** | 无 |
| **数据泄露** | 无 |
| **服务可用性** | 高 - 导致服务崩溃 (DoS) |

**修复建议**：

```c
int32_t ParseDslmCredential(
    const DslmCredBuff *credBuff, 
    DslmCredInfo *credInfo)
{
    // 增加输入参数验证
    if (credBuff == NULL || credBuff->credBuff == NULL) {
        SECURITY_LOG_ERROR("invalid credBuff parameter");
        return ERR_INVALID_PARA;
    }
    
    if (credBuff->credBuffSize == 0 || 
        credBuff->credBuffSize > CRED_BUFF_MAX_SIZE) {
        SECURITY_LOG_ERROR("invalid credBuffSize: %{public}u", 
                          credBuff->credBuffSize);
        return ERR_INVALID_LEN_PARA;
    }
    
    JSON *header = cJSON_ParseWithLength(
        credBuff->credBuff, 
        credBuff->credBuffSize);
    
    if (header == NULL) {
        SECURITY_LOG_ERROR("parse header failed");
        return ERR_JSON_ERR;
    }
    
    // 建议：使用 cJSON_ParseWithLengthCSL 增加长度检查
    // ...
}
```

---

## 6.3 内存安全问题

### R3: 设备列表内存泄漏风险

**风险等级**：中危

**位置**：`services/dslm/dslm_device_list.c`

**证据**：

```c
// services/slm/dslm_device_list.c:89-120
DslmDeviceInfo *CreatOrGetDslmDeviceInfo(const DeviceIdentify *identify)
{
    DslmDeviceInfo *device = GetDslmDeviceInfo(identify);
    
    if (device == NULL) {
        // 分配新设备
        device = (DslmDeviceInfo *)DslmMalloc(sizeof(DslmDeviceInfo));
        if (device == NULL) {
            return NULL;
        }
        
        // 初始化设备信息
        memset_s(device, sizeof(DslmDeviceInfo), 0, sizeof(DslmDeviceInfo));
        
        // 添加到链表
        DslmUtilsLock(&globalMutex_);
        ListNode *node = &(device->linkNode);
        ListAdd(&globalDeviceList_, node);
        DslmUtilsUnlock(&globalMutex_);
    }
    
    return device;
}

// 问题：设备下线后未释放内存
void OnPeerStatusReceiver(..., uint32_t onlineStatus)
{
    if (onlineStatus == OFFLINE) {
        // 只修改状态，未释放内存
        device->onlineStatus = OFFLINE;
        // TODO: 缺少内存释放逻辑
    }
}
```

**触发路径**：

```
设备频繁上下线
    │
    ▼
CreatOrGetDslmDeviceInfo() 创建设备
    │
    ▼
OnPeerStatusReceiver() 设备下线
    │
    ▼
只修改状态，设备对象未释放
    │
    ▼
长期运行 → 内存耗尽
```

**影响评估**：

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 需要特定场景触发 |
| **权限提升** | 无 |
| **数据泄露** | 无 |
| **服务可用性** | 中 - 长期运行后的 DoS |

**修复建议**：

```c
// 设备下线时释放内存
void OnPeerStatusReceiver(..., uint32_t onlineStatus)
{
    DslmDeviceInfo *device = GetDslmDeviceInfo(identify);
    if (device == NULL) {
        return;
    }
    
    if (onlineStatus == OFFLINE) {
        // 从链表中移除
        DslmUtilsLock(&globalMutex_);
        ListNode *node = &(device->linkNode);
        ListRemove(&globalDeviceList_, node);
        DslmUtilsUnlock(&globalMutex_);
        
        // 释放设备对象
        DestroyDslmDeviceInfo(device);  // 需要实现此函数
    }
}

void DestroyDslmDeviceInfo(DslmDeviceInfo *device)
{
    if (device == NULL) {
        return;
    }
    
    // 释放内部链表
    DestroyList(&device->notifyList);
    DestroyList(&device->historyList);
    
    // 清除敏感数据
    memset_s(device, sizeof(DslmDeviceInfo), 0, sizeof(DslmDeviceInfo));
    
    // 释放对象
    DslmFree(device);
}
```

---

### R4: 回调列表内存管理

**风险等级**：低危

**位置**：`services/dslm/dslm_core_process.c:150-180`

**证据**：

```c
// services/dslm/dslm_core_process.c:150-180
int32_t OnRequestDeviceSecLevelInfo(..., DslmNotifyListNode *node)
{
    DslmDeviceInfo *device = CreatOrGetDslmDeviceInfo(identify);
    if (device == NULL) {
        return ERR_NO_MEMORY;
    }
    
    // 检查回调列表大小
    if (device->notifyListSize >= MAX_NOTIFY_SIZE) {
        SECURITY_LOG_ERROR("notify list full");
        return ERR_MSG_NEIGHBOR_FULL;
    }
    
    // 添加到回调链表
    ListNode *next = &(node->linkNode);
    ListNode *head = &(device->notifyList);
    ListAddInsert(head, next, head);
    
    device->notifyListSize++;  // 增加计数
    
    // 调度状态机
    ScheduleDslmStateMachine(EVENT_SDK_GET);
    
    return SUCCESS;
}

// 问题：请求完成后未清理回调节点
void NotifyResultAndClean(..., uint32_t result)
{
    // 遍历通知列表
    ListNode *node = NULL;
    LIST_FOR_EACH(node, &device->notifyList) {
        DslmNotifyListNode *notifyNode = LIST_ENTRY(
            node, DslmNotifyListNode, linkNode);
        
        // 调用回调
        notifyNode->callback(identify, info);
    }
    
    // 问题：只清理了链表，未释放节点内存
    ListInit(&device->notifyList);
    device->notifyListSize = 0;
}
```

---

## 6.4 权限与鉴权

### R5: IPC 接口令牌验证缺陷

**风险等级**：低危

**位置**：`services/sa/standard/dslm_service.cpp:131-134`

**证据**：

```cpp
// services/sa/standard/dslm_service.cpp:131-134
int32_t DslmService::OnRemoteRequest(
    uint32_t code, 
    MessageParcel &data, 
    MessageParcel &reply, 
    MessageOption &option)
{
    SetSystemAbilityUnloadSchedule(unloadTimerHandle_);
    
    do {
        if (IDeviceSecurityLevel::GetDescriptor() != 
            data.ReadInterfaceToken()) {  // [检查点] 接口令牌验证
            SECURITY_LOG_ERROR("local descriptor is not equal remote");
            break;
        }
        
        switch (code) {
            case CMD_GET_DEVICE_SECURITY_LEVEL:
                return ProcessGetDeviceSecurityLevel(data, reply);
            default:
                return IPCObjectStub::OnRemoteRequest(
                    code, data, reply, option);
        }
    } while (false);
    
    return ERR_REQUEST_CODE_ERR;
}
```

**评估结论**：

| 评估项 | 状态 |
|--------|------|
| **接口令牌验证** | ✅ 已实现 |
| **调用者身份追踪** | ✅ IPCSkeleton::GetCallingPid() |
| **权限声明** | ✅ profile/dslm_service.cfg |

**现有安全措施**：

```cpp
// profile/dslm_service.cfg 配置
{
    "uid": 3046,
    "gid": 3046,
    "apl": "system_basic",
    "permissions": [
        "ohos.permission.ACCESS_IDS",
        "ohos.permission.sec.ACCESS_UDID",
        "ohos.permission.ACCESS_SERVICE_DM",
        "ohos.permission.DISTRIBUTED_DATASYNC"
    ]
}
```

---

## 6.5 并发安全

### R6: 定时器回调线程安全

**风险等级**：低危

**位置**：`services/sa/standard/dslm_service.cpp:46-66`

**证据**：

```cpp
// services/sa/standard/dslm_service.cpp:46-66
static void TimerProcessUnloadSystemAbility(const void *context)
{
    (void)context;
    
    // 检查设备类型
    if (!JudgeListDeviceType()) {
        return;
    }
    
    auto samgrProxy = SystemAbilityManagerClient::GetInstance()
        .GetSystemAbilityManager();
    
    if (samgrProxy == nullptr) {
        SECURITY_LOG_ERROR("get samgr failed");
        return;
    }
    
    int32_t ret = samgrProxy->UnloadSystemAbility(
        DEVICE_SECURITY_LEVEL_MANAGER_SA_ID);
    
    if (ret != ERR_OK) {
        SECURITY_LOG_ERROR("unload system ability failed");
        return;
    }
    
    SECURITY_LOG_INFO("unload system ability succeed");
}
```

**评估结论**：

| 评估项 | 状态 |
|--------|------|
| **定时器回调线程** | ⚠️ 独立线程执行 |
| **全局状态访问** | ⚠️ 未加锁保护 |
| **竞态条件** | ⚠️ 存在潜在风险 |

**风险说明**：

定时器回调在独立线程中执行，可能与主线程竞争全局状态。

---

## 6.6 逻辑漏洞

### R7: 回调重复调用风险

**风险等级**：低危

**位置**：`services/dslm/dslm_fsm_process.c`

**证据**：

```c
// services/dslm/dslm_fsm_process.c:200-280
void ProcessEventWithDevice(uint32_t event, DslmDeviceInfo *device)
{
    switch (device->machine.state) {
        case STATE_INIT:
            if (event == EVENT_SDK_GET) {
                // 发送请求并切换状态
                SendRequestToPeer(device);
                SetState(device, STATE_WAITING_CRED_RSP);
            }
            break;
            
        case STATE_WAITING_CRED_RSP:
            if (event == EVENT_SDK_GET) {
                // 问题：重复处理 SDK 请求
                // 可能导致回调被多次调用
                SendRequestToPeer(device);
            }
            break;
            
        case STATE_SUCCESS:
            if (event == EVENT_SDK_GET) {
                // 直接返回已缓存结果
                NotifyWithCachedResult(device);
            }
            break;
    }
}
```

**触发路径**：

```
SDK 请求 1 → STATE_INIT → STATE_WAITING_CRED_RSP
    │
    ▼
SDK 请求 2 (超时前到达)
    │
    ▼
仍在 STATE_WAITING_CRED_RSP 状态
    │
    ▼
再次调用 SendRequestToPeer()
    │
    ▼
可能产生重复回调
```

---

## 6.7 风险汇总表

| ID | 风险名称 | 风险等级 | 位置 | 可利用性 | 影响 |
|----|----------|----------|------|----------|------|
| R1 | MessageParcel 反序列化边界检查不充分 | 中危 | dslm_ipc_process.cpp:62-80 | 中 | 越界访问 |
| R2 | JSON 解析空指针风险 | 中危 | dslm_credential_utils.c:89-150 | 中 | 服务崩溃 |
| R3 | 设备列表内存泄漏 | 中危 | dslm_device_list.c | 低 | 资源耗尽 |
| R4 | 回调列表内存管理 | 低危 | dslm_core_process.c:150-180 | 低 | 内存泄漏 |
| R5 | IPC 接口令牌验证 | 低危 | dslm_service.cpp:131-134 | 低 | 已防护 |
| R6 | 定时器回调线程安全 | 低危 | dslm_service.cpp:46-66 | 低 | 竞态条件 |
| R7 | 回调重复调用风险 | 低危 | dslm_fsm_process.c | 低 | 逻辑错误 |

---

## 6.8 安全加固措施

### 已启用的安全加固

| 加固项 | 启用平台 | 配置位置 |
|--------|----------|----------|
| **CFI (Control Flow Integrity)** | Standard | bundle.json, sanitizer config |
| **UBSan (Undefined Behavior Sanitizer)** | Standard | bundle.json, sanitizer config |
| **Integer Overflow Sanitize** | Standard | bundle.json, sanitizer config |
| **PacRet (Pointer Authentication)** | Standard | BUILD.gn, pac_ret |

**代码证据**：`services/sa/standard/BUILD.gn`

```gn
# Standard 版本启用安全加固
if (defined(ohos sanitizer config)) {
    cflags_cc += ohos.sanitizer_config.cflags_cc
    ldflags += ohos.sanitizer_config.ldflags
}

# 分支保护
if (enable branch_protector) {
    cflags_cc += [ "-fvisibility=hidden", "-fsanitize=cfi" ]
    ldflags += [ "-fsanitize=cfi" ]
}
```

### 建议的额外加固

| 建议 | 优先级 | 说明 |
|------|--------|------|
| 增加 Fuzzing 测试 | 高 | 对 MessageParcel 解析进行模糊测试 |
| 完善内存释放逻辑 | 高 | 清理未使用的设备对象 |
| 增加 AddressSanitizer | 中 | 开发测试阶段启用 ASan |
| 完善竞态检测 | 中 | 使用 ThreadSanitizer 检测 |

---

## 6.9 审计日志

### HiEvent 配置

```yaml
# hisysevent.yaml
SECURITY_DEVICE_SECURITY_LEVEL:
  type: security
  level: SEC_INFO
  tag: DEVICE_SECURITY_LEVEL
  msg: ""
```

**日志位置**：`services/dslm/dslm_hievent.c`

| 事件类型 | 记录位置 | 说明 |
|----------|----------|------|
| 请求开始 | `dslm_hievent.c` | 记录设备标识和选项 |
| 请求完成 | `dslm_hievent.c` | 记录结果和耗时 |
| 错误事件 | `dslm_hievent.c` | 记录错误码和上下文 |

---

## 下一章

- [07_Build.md](./07_Build.md) - 构建与产物
- [08_Internals.md](./08_Internals.md) - 内部实现细节
