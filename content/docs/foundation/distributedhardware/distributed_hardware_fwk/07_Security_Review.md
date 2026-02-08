# 安全风险评审报告

**评审对象**: OpenHarmony 分布式硬件管理框架 (distributed_hardware_fwk)  
**版本**: 4.0  
**SA ID**: 4801 (dhardware 进程)  
**最后更新**: 2025-02-07

---

## 评审范围

### 已覆盖代码
- ✅ N-API 接口层 (`interfaces/kits/napi/`)
- ✅ IPC 通信层 (`services/distributedhardwarefwkservice/src/distributed_hardware_stub.cpp`)
- ✅ 服务核心逻辑 (`services/distributedhardwarefwkservice/`)
- ✅ 部件加载与动态库管理 (`componentloader/`)
- ✅ 跨设备传输层 (`transport/`)
- ✅ 资源管理器 (`resourcemanager/`)
- ✅ AV 传输引擎 (`av_transport/`)

### 未覆盖范围
- ❌ 测试代码 (`test/`, `*_test.*`, `fuzztest/`)
- ❌ HDF 驱动层
- ❌ 第三方库 (SoftBus, DeviceManager, cJSON 等)

---

## 攻击面分析

### 1. N-API 接口 (JavaScript 调用入口)

**位置**: `interfaces/kits/napi/src/native_distributedhardwarefwk_js.cpp:212-369`

**暴露的 API**:
| API 名称 | 行号 | 描述 |
|----------|------|------|
| `pauseDistributedHardware` | 212-263 | 暂停分布式硬件 |
| `resumeDistributedHardware` | 265-316 | 恢复分布式硬件 |
| `stopDistributedHardware` | 318-369 | 停止分布式硬件 |

**安全控制**:
```cpp
// native_distributedhardwarefwk_js.cpp:164-182
bool DistributedHardwareManager::Verify(napi_env env, int32_t type)
{
    if (!IsSystemApp()) {                          // 系统应用校验
        CreateBusinessErr(env, ERR_NOT_SYSTEM_APP);
        return false;
    }
    if (!HasAccessDHPermission()) {                // 权限校验
        CreateBusinessErr(env, ERR_NO_PERMISSION);
        return false;
    }
    if (!IsSupportType(type)) {                    // 类型校验
        CreateBusinessErr(env, ERR_INVALID_PARAMS);
        return false;
    }
    return true;
}
```

**攻击面评估**: 低  
**理由**: 三重校验机制（系统应用 + 权限 + 参数类型）

---

### 2. IPC 接口 (跨进程调用)

**位置**: `services/distributedhardwarefwkservice/src/distributed_hardware_stub.cpp`

**接口数量**: 26 个 IPC 方法 (Interface Code 48001-480026)

**关键入口**:
```cpp
// distributed_hardware_stub.cpp:39-54
int32_t DistributedHardwareStub::OnRemoteRequest(uint32_t code, MessageParcel &data, 
    MessageParcel &reply, MessageOption &option)
{
    if (data.ReadInterfaceToken() != GetDescriptor()) {  // Token 校验
        return ERR_INVALID_DATA;
    }
    // 特定 Code 允许远程调用(跨设备)
    if (code != NOTIFY_SOURCE_DEVICE_REMOTE_DMSDP_STARTED &&
        code != INIT_SINK_DMSDP &&
        code != NOTIFY_SINK_DEVICE_REMOTE_DMSDP_STARTED) {
        if (!IPCSkeleton::IsLocalCalling()) {            // 本地调用校验
            return ERR_DH_FWK_IS_LOCAL_PROCESS_FAIL;
        }
    }
    // ... dispatch to handlers
}
```

**攻击面评估**: 中  
**理由**: 3 个 IPC Code 允许远程调用,需验证其安全性

---

### 3. 跨设备传输接口

**位置**: `services/distributedhardwarefwkservice/src/transport/dh_transport.cpp`

**攻击面**: 网络消息接收、ACL 验证

**攻击面评估**: 中  
**理由**: 涉及网络输入处理和 ACL 验证

---

### 4. 动态库加载接口

**位置**: 
- `services/distributedhardwarefwkservice/src/componentloader/component_loader.cpp:314`
- `services/distributedhardwarefwkservice/src/hdfoperate/hdf_operate.cpp:249`

**攻击面评估**: 高  
**理由**: dlopen() 加载外部 .so 文件

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              不可信区域                                       │
│  ┌─────────────────┐  ┌──────────────────────────────────────────────────┐   │
│  │  第三方应用      │  │  远程设备                                         │   │
│  │  (无权限访问)    │  │  (需 ACL 验证)                                    │   │
│  └─────────────────┘  └──────────────────────────────────────────────────┘   │
│                              │                                               │
│                              ▼ 边界                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                              可信区域                                         │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                      N-API 层 (JS→C++)                               │  │
│  │  系统应用校验 + ACCESS_DISTRIBUTED_HARDWARE 权限                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                     IPC 层 (Inner Kit)                               │  │
│  │  IPCSkeleton 身份验证 + 权限校验                                       │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │            DistributedHardwareService (SA 4801)                      │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                 │  │
│  │  │AccessManager │ │ResourceMgr   │ │ComponentMgr  │                 │  │
│  │  └──────────────┘ └──────────────┘ └──────────────┘                 │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 安全风险评估

### R1: 动态库加载路径校验不足 (高危)

**位置**: `services/distributedhardwarefwkservice/src/componentloader/component_loader.cpp:308-321`

**证据**:
```cpp
// component_loader.cpp:308-321
void *ComponentLoader::GetHandler(const std::string &soName)
{
    if (soName.length() == 0 || soName.length() > PATH_MAX) {
        DHLOGE("File canonicalization failed, soName: %{public}s", soName.c_str());
        return nullptr;
    }
    void *pHandler = dlopen(soName.c_str(), RTLD_LAZY | RTLD_NODELETE);
    if (pHandler == nullptr) {
        DHLOGE("so: %{public}s load failed, failed reason: %{public}s", soName.c_str(), dlerror());
        return nullptr;
    }
    return pHandler;
}
```

**触发路径**:
```
设备上线事件 → ComponentManager::Enable() → GetSource()/GetSink() 
→ ComponentLoader::GetHandler(soName) → dlopen(soName.c_str())
```

其中 `soName` 来自配置文件解析:
```
ParseConfig() → Readfile() → cJSON_Parse() → compConfig.compSourceLoc/compSinkLoc
```

**问题分析**:
- `GetHandler()` 仅检查路径长度,未验证路径是否指向可信目录
- 配置文件路径通过 `GetOneCfgFile()` 获取,但实际加载时使用配置中的路径
- 若配置文件被篡改,可能加载任意路径的 .so 文件

**影响评估**:
| 维度 | 评估 |
|------|------|
| **可利用性** | 中 (需先篡改配置文件) |
| **权限提升** | 高 (代码执行在 dhardware 进程,拥有 system 权限) |
| **影响范围** | 整个分布式硬件框架 |

**修复建议**:
```cpp
// 在 GetHandler() 中增加路径白名单校验
bool IsTrustedPath(const std::string& path) {
    const std::vector<std::string> trustedPrefixes = {
        "/system/lib/",
        "/vendor/lib/",
        "/data/distributed_hardware/"
    };
    // 使用 realpath 规范化路径后比较
    char resolvedPath[PATH_MAX];
    if (realpath(path.c_str(), resolvedPath) == nullptr) {
        return false;
    }
    for (const auto& prefix : trustedPrefixes) {
        if (strncmp(resolvedPath, prefix.c_str(), prefix.length()) == 0) {
            return true;
        }
    }
    return false;
}
```

---

### R2: JSON 解析缺少字段校验 (中危)

**位置**: `services/distributedhardwarefwkservice/src/resourcemanager/capability_info.cpp:126-252`

**证据**:
```cpp
// capability_info.cpp:126-252 (简化)
int32_t CapabilityInfo::FromJsonString(const std::string &jsonStr)
{
    cJSON *jsonObj = cJSON_Parse(jsonStr.c_str());
    if (jsonObj == nullptr) {
        return ERR_DH_FWK_PARA_INVALID;
    }
    
    cJSON *compVersionObj = cJSON_GetObjectItem(jsonObj, CAP_VERSION);
    cJSON *dhIdJson = cJSON_GetObjectItem(jsonObj, CAP_DH_ID);
    cJSON *devIdJson = cJSON_GetObjectItem(jsonObj, CAP_DEV_ID);
    
    // 直接使用,缺少字段存在性校验
    std::string dhId = std::string(cJSON_GetStringValue(dhIdJson));
    std::string devId = std::string(cJSON_GetStringValue(devIdJson));
    // ...
    cJSON_Delete(jsonObj);
    return DH_FWK_SUCCESS;
}
```

**触发路径**:
```
远程设备上线 → DHTransport 接收 Capability 信息 → CapabilityInfoManager 
→ FromJsonString() 解析 JSON → 字段缺失时返回空指针 → 未检查直接使用
```

**问题分析**:
- 多处使用 cJSON 解析网络输入(设备能力信息)
- `cJSON_GetObjectItem()` 和 `cJSON_GetStringValue()` 可能返回 NULL
- 未充分检查字段存在性和类型,可能导致空指针解引用

**影响评估**:
| 维度 | 评估 |
|------|------|
| **可利用性** | 中 (需构造恶意 JSON) |
| **权限提升** | 低 (可能导致崩溃或信息泄露) |
| **影响范围** | 资源管理模块、设备上线流程 |

**修复建议**:
```cpp
// 增加字段存在性和类型校验
int32_t CapabilityInfo::FromJsonString(const std::string &jsonStr)
{
    cJSON *jsonObj = cJSON_Parse(jsonStr.c_str());
    if (jsonObj == nullptr) {
        return ERR_DH_FWK_PARA_INVALID;
    }
    
    // 必需字段校验
    cJSON *dhIdJson = cJSON_GetObjectItem(jsonObj, CAP_DH_ID);
    if (dhIdJson == nullptr || !cJSON_IsString(dhIdJson)) {
        cJSON_Delete(jsonObj);
        return ERR_DH_FWK_PARA_INVALID;
    }
    
    // 字符串长度校验
    const char* dhIdStr = cJSON_GetStringValue(dhIdJson);
    if (dhIdStr == nullptr || strlen(dhIdStr) > MAX_ID_LEN) {
        cJSON_Delete(jsonObj);
        return ERR_DH_FWK_PARA_INVALID;
    }
    
    // ... 其他字段校验
}
```

---

### R3: IPC 接口权限校验覆盖不完整 (中危)

**位置**: `services/distributedhardwarefwkservice/src/distributed_hardware_stub.cpp:47-54`

**证据**:
```cpp
// distributed_hardware_stub.cpp:47-54
if (code != static_cast<uint32_t>(DHMsgInterfaceCode::NOTIFY_SOURCE_DEVICE_REMOTE_DMSDP_STARTED) &&
    code != static_cast<uint32_t>(DHMsgInterfaceCode::INIT_SINK_DMSDP) &&
    code != static_cast<uint32_t>(DHMsgInterfaceCode::NOTIFY_SINK_DEVICE_REMOTE_DMSDP_STARTED)) {
    if (!IPCSkeleton::IsLocalCalling()) {
        DHLOGE("Invalid request, only support local, code = %{public}u.", code);
        return ERR_DH_FWK_IS_LOCAL_PROCESS_FAIL;
    }
}
```

**触发路径**:
```
远程进程 → IPC::SendRequest(code=NOTIFY_SOURCE_DEVICE_REMOTE_DMSDP_STARTED) 
→ OnRemoteRequest() → 绕过 IsLocalCalling() 检查 
→ HandleNotifySourceRemoteSinkStarted() → 触发源端逻辑
```

**问题分析**:
- 3 个 IPC Code 允许远程调用(跨设备通信需求)
- `NOTIFY_SOURCE_DEVICE_REMOTE_DMSDP_STARTED`: 通知源设备远程 Sink 已启动
- `INIT_SINK_DMSDP`: 初始化 Sink 端 DMSDP 服务
- `NOTIFY_SINK_DEVICE_REMOTE_DMSDP_STARTED`: 通知 Sink 设备远程 Source 已启动
- 虽可能有业务需求,但缺少额外的身份验证机制

**影响评估**:
| 维度 | 评估 |
|------|------|
| **可利用性** | 低 (需了解 IPC 协议和 Code 值) |
| **权限提升** | 中 (可能干扰跨设备通信状态) |
| **影响范围** | 跨设备通信功能 |

**修复建议**:
```cpp
// 增加远程调用的来源设备验证
int32_t DistributedHardwareStub::HandleNotifySourceRemoteSinkStarted(MessageParcel &data)
{
    // 验证调用者设备身份
    std::string deviceId = data.ReadString();
    if (!IsTrustedDevice(deviceId)) {
        DHLOGE("Untrusted device: %{public}s", deviceId.c_str());
        return ERR_DH_FWK_UNTRUSTED_DEVICE;
    }
    
    // 验证 ACL 权限
    if (!CheckCrossDeviceAcl(deviceId)) {
        return ERR_DH_FWK_ACCESS_DENIED;
    }
    
    // 处理业务逻辑
    return DistributedHardwareService::GetInstance().NotifySourceRemoteSinkStarted(deviceId);
}
```

---

### R4: Native SA 权限绕过风险 (中危)

**位置**: `services/distributedhardwarefwkservice/src/distributed_hardware_stub.cpp:805-833`

**证据**:
```cpp
// distributed_hardware_stub.cpp:805-833
bool DistributedHardwareStub::HasAccessDHPermission()
{
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE";
    int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    return (result == Security::AccessToken::PERMISSION_GRANTED);
}

bool DistributedHardwareStub::IsNativeSA()
{
    Security::AccessToken::AccessTokenID tokenId = IPCSkeleton::GetCallingTokenID();
    Security::AccessToken::ATokenTypeEnum tokenType = 
        Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(tokenId);
    if (tokenType == Security::AccessToken::ATokenTypeEnum::TOKEN_NATIVE) {
        return true;  // Native SA 直接通过
    }
    return false;
}
```

**触发路径**:
```
Native 进程(无权限) → IPC 调用 → OnRemoteRequest() 
→ 检查 IsNativeSA() → 返回 true → 绕过权限检查 → 执行操作
```

**问题分析**:
- `IsNativeSA()` 检查 TOKEN_NATIVE 类型,Native 系统应用可能绕过权限校验
- 虽然 Native SA 通常是可信系统服务,但逻辑上存在绕过可能
- 应显式检查所需权限而非仅依赖 Token 类型

**影响评估**:
| 维度 | 评估 |
|------|------|
| **可利用性** | 低 (需 Native 进程权限) |
| **权限提升** | 中 (绕过 ACCESS_DISTRIBUTED_HARDWARE 权限检查) |
| **影响范围** | IPC 接口访问控制 |

**修复建议**:
```cpp
// 始终验证具体权限,不依赖 Token 类型
bool DistributedHardwareStub::HasAccessDHPermission()
{
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE";
    int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    
    // 明确检查权限结果
    if (result != Security::AccessToken::PERMISSION_GRANTED) {
        DHLOGE("Permission check failed, caller token: %{public}u", callerToken);
        return false;
    }
    return true;
}

// 单独函数用于区分 Native/Hap,但不替代权限检查
bool DistributedHardwareStub::IsNativeSA()
{
    // 仅用于日志或差异化处理,不用于权限决策
    // ...
}
```

---

### R5: 整数溢出导致内存分配风险 (低危)

**位置**: `services/distributedhardwarefwkservice/src/transport/dh_transport.cpp:98-114`

**证据**:
```cpp
// dh_transport.cpp:98-114
int32_t DHTransport::SendMsg(const std::string &remoteNetworkId, const std::string &msg, 
    int32_t dataLen, int32_t channelId)
{
    // dataLen 来自外部输入(网络消息大小)
    if (dataLen > MAX_SEND_MSG_LENGTH) {
        return ERR_DH_FWK_TRANSPORT_SEND_MSG_FAIL;
    }
    
    uint8_t *buf = reinterpret_cast<uint8_t *>(calloc(dataLen + 1, sizeof(uint8_t)));
    if (buf == nullptr) {
        return ERR_DH_FWK_MALLOC_FAIL;
    }
    
    if (memcpy_s(buf, dataLen + 1, reinterpret_cast<const uint8_t *>(msg.c_str()), dataLen) != EOK) {
        free(buf);
        return ERR_DH_FWK_MEMORY_OPERATION_FAIL;
    }
    // ...
}
```

**触发路径**:
```
远程设备发送消息 → SoftBus 接收 → OnBytesReceived() → dataLen 解析
→ SendMsg() → calloc(dataLen + 1) → 若 dataLen = INT_MAX, 则溢出为 0
```

**问题分析**:
- `dataLen` 为 `int32_t` 类型,最大值约 2GB
- `dataLen + 1` 在 `dataLen = INT_MAX` 时发生整数溢出,结果为负数或 0
- 虽检查了 `MAX_SEND_MSG_LENGTH (4MB)`,但类型转换可能引入问题
- `calloc()` 参数为 `size_t`,`int32_t` 到 `size_t` 的转换在负数时产生极大值

**影响评估**:
| 维度 | 评估 |
|------|------|
| **可利用性** | 低 (需 MAX_SEND_MSG_LENGTH 检查被绕过) |
| **权限提升** | 中 (内存分配失败或异常行为) |
| **影响范围** | 传输层消息处理 |

**修复建议**:
```cpp
// 使用 size_t 类型并增加溢出检查
int32_t DHTransport::SendMsg(const std::string &remoteNetworkId, const std::string &msg, 
    int32_t dataLen, int32_t channelId)
{
    // 验证 dataLen 范围
    if (dataLen <= 0 || dataLen > MAX_SEND_MSG_LENGTH) {
        DHLOGE("Invalid dataLen: %{public}d", dataLen);
        return ERR_DH_FWK_TRANSPORT_SEND_MSG_FAIL;
    }
    
    // 使用 size_t 进行内存计算,检查溢出
    size_t allocSize = static_cast<size_t>(dataLen) + 1;
    if (allocSize > MAX_SEND_MSG_LENGTH + 1) {
        DHLOGE("Size overflow detected");
        return ERR_DH_FWK_TRANSPORT_SEND_MSG_FAIL;
    }
    
    uint8_t *buf = reinterpret_cast<uint8_t *>(calloc(allocSize, sizeof(uint8_t)));
    // ...
}
```

---

## 已实施的安全加固措施

### 1. 权限控制矩阵

| 检查点 | 位置 | 说明 |
|--------|------|------|
| 系统应用校验 | `native_distributedhardwarefwk_js.cpp:149` | `TokenIdKit::IsSystemAppByFullTokenID` |
| 权限校验 | `distributed_hardware_stub.cpp:805` | `VerifyAccessToken(ACCESS_DISTRIBUTED_HARDWARE)` |
| 本地调用校验 | `distributed_hardware_stub.cpp:50` | `IPCSkeleton::IsLocalCalling()` |
| Native SA 校验 | `distributed_hardware_stub.cpp:823` | `TOKEN_NATIVE` 类型检查 |
| Token ID 校验 | `dh_transport.cpp:130` | `IPCSkeleton::GetCallingTokenID()` |

### 2. 输入验证限制

| 验证项 | 位置 | 限制值 | 说明 |
|--------|------|--------|------|
| ID 长度 | `dh_utils_tool.cpp:306` | 256 | `MAX_ID_LEN` |
| 消息长度 | `dh_utils_tool.cpp:315` | 40MB | `MAX_MESSAGE_LEN` |
| JSON 大小 | `dh_utils_tool.cpp:324` | 40MB | `MAX_JSON_SIZE` |
| 数组长度 | `dh_utils_tool.cpp:333` | 10000 | `MAX_ARR_SIZE` |
| 密钥大小 | `dh_utils_tool.cpp:342` | 256 | `MAX_KEY_SIZE` |
| 哈希大小 | `dh_utils_tool.cpp:351` | 64 | `MAX_HASH_SIZE` |
| 网络消息 | `dh_transport.cpp:87` | 4MB | `MAX_SEND_MSG_LENGTH` |
| 路径长度 | `component_loader.cpp:310` | 系统定义 | `PATH_MAX` |
| 部件数量 | `component_loader.cpp:147` | 128 | `MAX_COMP_SIZE` |

### 3. 内存安全实践

- ✅ 所有内存拷贝使用 `memcpy_s` 而非 `memcpy`
- ✅ 缓冲区大小在拷贝前验证
- ✅ 使用 `calloc` 而非 `malloc` 初始化内存
- ✅ 异常路径释放已分配资源

### 4. 路径安全

- ✅ 配置文件路径使用 `realpath()` 规范化
- ✅ 防止路径遍历攻击
- ✅ 使用 `GetOneCfgFile()` 从预定义路径加载配置

### 5. 资源限制 (DoS 防护)

| 资源 | 限制 | 位置 |
|------|------|------|
| 数据库记录 | 10000 条 | `constants.h:MAX_DB_RECORD_SIZE` |
| 在线设备 | 10000 台 | `constants.h:MAX_ONLINE_DEVICE_SIZE` |
| DB 数据大小 | 100 条 | `constants.h:MAX_WRITE_DB_DATA_SIZE` |

---

## 总结

### 风险统计

| 级别 | 数量 | 风险项 |
|------|------|--------|
| 🔴 高危 | 1 | R1: 动态库加载路径校验不足 |
| 🟡 中危 | 3 | R2: JSON 解析缺少字段校验, R3: IPC 权限覆盖不完整, R4: Native SA 权限绕过 |
| 🟢 低危 | 1 | R5: 整数溢出导致内存分配风险 |

### 修复优先级建议

| 优先级 | 风险项 | 修复建议 |
|--------|--------|----------|
| **P0 - 立即** | R1 | 实现路径白名单和签名验证 |
| **P1 - 短期** | R2, R3 | 增加 JSON 字段校验和 IPC 来源验证 |
| **P2 - 中期** | R4 | 移除 Native SA 权限绕过 |
| **P3 - 长期** | R5 | 统一使用 size_t 并增加溢出检查 |

### 总体评估

分布式硬件管理框架整体安全设计良好,具备多层权限校验和输入验证机制。主要风险集中在:
1. **动态库加载** (R1): 需优先加固路径验证
2. **IPC 远程调用** (R3): 需补充来源设备验证
3. **数据解析** (R2): 需增加 JSON/schema 校验

建议按照修复优先级逐步实施安全加固。

---

## 参考链接

- [项目评估报告](./_work/ASSESSMENT.md)
- [架构说明](./02_Architecture.md)
- [N-API 接口文档](./03_NAPI_Reference.md)
- [IPC 接口文档](./04_Inner_API.md)
