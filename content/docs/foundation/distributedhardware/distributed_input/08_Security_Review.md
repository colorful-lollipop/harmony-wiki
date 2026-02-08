# 安全评审

**文档版本**: v1.0  
**评审日期**: 2025-02-07  
**模块版本**: 3.2

---

## 目录

- [1. 执行摘要](#1-执行摘要)
- [2. 威胁模型概述](#2-威胁模型概述)
- [3. 攻击面分析](#3-攻击面分析)
- [4. 安全机制分析](#4-安全机制分析)
- [5. 安全风险评估](#5-安全风险评估)
- [6. 安全建议](#6-安全建议)
- [7. 审计结论](#7-审计结论)

---

## 1. 执行摘要

### 1.1 评审目标

本安全评审旨在全面分析 OpenHarmony `distributed_input`（分布式输入）模块的安全特性，识别潜在的攻击面和漏洞点，为安全研究员提供详细的安全分析报告。

### 1.2 评审范围

| 范围 | 说明 |
|------|------|
| **模块** | `foundation/distributedhardware/distributed_input` |
| **版本** | 3.2 |
| **组件** | Source SA (4809)、Sink SA (4810)、传输层、InnerKit |
| **排除范围** | 测试代码（test/、tests/）、JS/N-API 层 |

### 1.3 关键发现摘要

| 风险等级 | 发现数量 | 说明 |
|---------|---------|------|
| **高危** | 0 | 无发现 |
| **中危** | 3 | 需要关注 |
| **低危** | 4 | 建议改进 |

### 1.4 安全评级：**良好**

`distributed_input` 模块实现了较为完善的安全机制，包括：
- 基于 AccessToken 的权限验证
- 同账号校验机制
- 输入参数完整性校验
- 路径规范化防止目录遍历
- 安全事件监控（HiSysEvent）

---

## 2. 威胁模型概述

### 2.1 系统角色定义

```
┌─────────────────────────────────────────────────────────────────────┐
│                        分布式输入系统                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────┐         SoftBus          ┌─────────────┐         │
│   │   源设备     │  ◄──────────────────►  │   目标设备   │         │
│   │  (Source)   │    跨设备事件传输        │   (Sink)    │         │
│   │             │                          │             │         │
│   │ 接收远程    │                          │ 采集本地    │         │
│   │ 输入事件    │                          │ 输入事件    │         │
│   │ 并注入系统  │                          │ 并发送      │         │
│   └─────────────┘                          └─────────────┘         │
│        ▲                                         │                 │
│        │                                         │                 │
│        │              ┌─────────────┐            │                 │
│        └─────────────►│  多模输入   │◄───────────┘                 │
│                       │   子系统    │                               │
│                       │  (调用方)   │                               │
│                       └─────────────┘                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 信任边界

| 边界 | 说明 | 跨越时的安全要求 |
|------|------|-----------------|
| **BB1** | 应用层 → 多模输入子系统 | 无直接调用（本模块不暴露北向 API） |
| **BB2** | 多模输入 → Source SA | IPC 调用，需权限校验 |
| **BB3** | Source → Sink (跨设备) | SoftBus 传输，需同账号校验 |
| **BB4** | Sink → 本地输入设备 | 驱动层访问，需 uhid/input 组权限 |
| **BB5** | Source → 虚拟设备 | 写入 /dev/uinput，需 uhid 组权限 |

### 2.3 资产分类

| 资产 | 分类 | 敏感级别 | 说明 |
|------|------|---------|------|
| **用户输入事件** | 数据 | 高 | 键盘、鼠标、触摸事件 |
| **设备描述符** | 配置 | 中 | 设备标识符 |
| **白名单配置** | 配置 | 中 | 组合键过滤规则 |
| **SA 权限** | 配置 | 高 | 系统能力访问权限 |
| **会话密钥** | 密钥 | 高 | SoftBus 传输加密 |

---

## 3. 攻击面分析

### 3.1 外部输入入口

#### 3.1.1 IPC 接口输入

**Source SA (ID: 4809)**

| 接口 | 命令码 | 输入参数 | 风险等级 |
|------|-------|---------|---------|
| `InitDistributedHardware` | 0xf001 | - | 低 |
| `RegisterDistributedHardware` | 0xf003 | devId, dhId, parameters | **中** |
| `UnregisterDistributedHardware` | 0xf004 | devId, dhId | 低 |
| `PrepareRemoteInput` | 0xf005 | deviceId | **中** |
| `UnprepareRemoteInput` | 0xf006 | deviceId | 低 |
| `StartRemoteInput` | 0xf007 | deviceId, inputTypes | **中** |
| `StopRemoteInput` | 0xf008 | deviceId, inputTypes | 低 |
| `StartRelayRemoteInput` | 0xf00a-0xf011 | srcId, sinkId, dhIds | **高** |
| `RegisterSimulationEventListener` | 0xf017 | - | 低 |

**Sink SA (ID: 4810)**

| 接口 | 命令码 | 输入参数 | 风险等级 |
|------|-------|---------|---------|
| `Init` | 0xf011 | - | 低 |
| `NotifyStartDScreen` | 0xf013 | srcScreenInfo | **中** |
| `NotifyStopDScreen` | 0xf014 | srcScreenInfoKey | 低 |
| `RegisterSharingDhIdListener` | 0xf015 | listener | 低 |

**证据**: `common/include/dinput_ipc_interface_code.h:24-58`

```cpp
// Source 命令码定义
enum class IDInputSourceInterfaceCode : uint32_t {
    INIT = 0xf001U,
    RELEASE = 0xf002U,
    REGISTER_REMOTE_INPUT = 0xf003U,
    UNREGISTER_REMOTE_INPUT = 0xf004U,
    PREPARE_REMOTE_INPUT = 0xf005U,
    // ...
    START_DHID_REMOTE_INPUT = 0xf00eU,
    STOP_DHID_REMOTE_INPUT = 0xf00fU,
    // ...
};
```

#### 3.1.2 跨设备传输输入

**事件数据格式**

| 字段 | 类型 | 最大长度 | 说明 |
|------|------|---------|------|
| `RawEvent.when` | int64_t | 8B | 时间戳 |
| `RawEvent.type` | uint32_t | 4B | 事件类型 |
| `RawEvent.code` | uint32_t | 4B | 事件代码 |
| `RawEvent.value` | int32_t | 4B | 事件值 |
| `RawEvent.descriptor` | string | 256B | 设备描述符 |
| `RawEvent.path` | string | 256B | 设备路径 |

**证据**: `common/include/constants_dinput.h:248-261`

```cpp
struct RawEvent {
    int64_t when;           // 时间戳
    uint32_t type;          // 事件类型 (EV_KEY, EV_ABS, etc.)
    uint32_t code;          // 事件代码
    int32_t value;          // 事件值
    std::string descriptor; // 设备描述符
    std::string path;       // 设备路径
};
```

**消息大小限制**

| 限制 | 值 | 文件位置 |
|------|------|---------|
| `MSG_MAX_SIZE` | 32KB | `services/common/include/dinput_softbus_define.h` |
| `IPC_VECTOR_MAX_SIZE` | 32 | `common/include/constants_dinput.h` |
| `DEV_ID_LENGTH_MAX` | 256 | `common/include/constants_dinput.h` |
| `DH_ID_LENGTH_MAX` | 256 | `common/include/constants_dinput.h` |

#### 3.1.3 配置文件输入

**白名单配置文件**

| 配置项 | 路径 | 安全机制 |
|--------|------|---------|
| 白名单文件 | `/etc/minidp_dinput_whitelist.cfg` | 路径规范化、行数限制、长度限制 |

**证据**: `common/include/white_list_util.cpp:37-42`

```cpp
const int32_t MAX_LINE_NUM = 100;           // 最大行数
const int32_t MAX_CHAR_PER_LINE_NUM = 100;   // 每行最大字符数
const int32_t MAX_SPLIT_COMMA_NUM = 4;       // 最大逗号分隔数
const int32_t MAX_SPLIT_LINE_NUM = 12;       // 最大行分割数
const int32_t MAX_KEY_CODE_NUM = 4;          // 每个键的最大代码数

// 路径规范化
if (strlen(whiteListFilePath) == 0 || strlen(whiteListFilePath) > PATH_MAX ||
    realpath(whiteListFilePath, path) == nullptr) {
    DHLOGE("File connicailization failed.");
    return false;
}
```

### 3.2 敏感操作

#### 3.2.1 系统调用

| 操作 | 文件 | 权限要求 | 风险 |
|------|------|---------|------|
| `open("/dev/uinput")` | `virtual_device.cpp:127` | uhid 组 | **高** - 虚拟设备创建 |
| `open("/dev/input/*")` | `input_hub.cpp` | input 组 | **高** - 原始设备访问 |
| `ioctl` | `virtual_device.cpp` | uhid 组 | **高** - 设备配置 |
| `write` (虚拟设备) | `virtual_device.cpp` | uhid 组 | **高** - 事件注入 |

**证据**: `services/source/inputinject/src/virtual_device.cpp:127`

```cpp
fd_ = open("/dev/uinput", O_WRONLY | O_NONBLOCK);
if (fd_ < 0) {
    DHLOGE("open /dev/uinput failed, errno: %{public}d", errno);
    return false;
}
```

#### 3.2.2 网络操作

| 操作 | 文件 | 加密 | 认证 |
|------|------|------|------|
| `Socket()` | `distributed_input_transport_base.cpp` | SoftBus 提供 | 同账号校验 |
| `SetAccessInfo()` | `softbus_permission_check.cpp` | - | tokenId 绑定 |

**证据**: `services/transportbase/src/softbus_permission_check.cpp:206-233`

```cpp
bool SoftBusPermissionCheck::SetAccessInfoToSocket(const int32_t sessionId)
{
    AccountInfo accountInfo;
    if (!GetLocalAccountInfo(accountInfo)) {
        return false;
    }
    SocketAccessInfo accessInfo;
    accessInfo.userId = accountInfo.userId_;
    accessInfo.localTokenId = accountInfo.tokenId_;
    // ...
    if (SetAccessInfo(sessionId, accessInfo) != 0) {
        return false;
    }
}
```

### 3.3 攻击面汇总

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            攻击面矩阵                                         │
├────────────────────┬────────────────────┬────────────────────┬─────────────┤
│     攻击面         │      输入类型       │     防护机制        │   风险等级  │
├────────────────────┼────────────────────┼────────────────────┼─────────────┤
│ IPC 命令参数       │ devId, dhId,       │ AccessToken 校验    │    中       │
│                    │ inputTypes         │ 参数长度校验        │             │
├────────────────────┼────────────────────┼────────────────────┼─────────────┤
│ 跨设备事件数据     │ RawEvent 结构      │ MSG_MAX_SIZE 限制   │    低       │
│                    │ (type, code, value)│ SoftBus 传输加密    │             │
├────────────────────┼────────────────────┼────────────────────┼─────────────┤
│ 设备描述符         │ string (256B)      │ 长度校验            │    低       │
├────────────────────┼────────────────────┼────────────────────┼─────────────┤
│ 白名单配置文件     │ 文本文件           │ realpath 规范化     │    低       │
│                    │                    │ 行数/长度限制       │             │
├────────────────────┼────────────────────┼────────────────────┼─────────────┤
│ 虚拟设备创建       │ ioctl 命令         │ uhid 组权限         │    高       │
├────────────────────┼────────────────────┼────────────────────┼─────────────┤
│ SoftBus 会话       │ networkId          │ 同账号校验          │    中       │
└────────────────────┴────────────────────┴────────────────────┴─────────────┘
```

---

## 4. 安全机制分析

### 4.1 权限验证机制

#### 4.1.1 AccessToken 权限校验

**验证位置**: `interfaces/ipc/src/distributed_input_source_stub.cpp:36-52`

**权限定义**

| 权限名 | 用途 | 保护的操作 |
|--------|------|-----------|
| `ohos.permission.ENABLE_DISTRIBUTED_HARDWARE` | 启用分布式硬件 | Init、Register、Unregister |
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | 访问分布式硬件 | Prepare、Start、Stop |

**实现代码**

```cpp
bool DistributedInputSourceStub::HasEnableDHPermission()
{
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ENABLE_DISTRIBUTED_HARDWARE";
    int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    return (result == Security::AccessToken::PERMISSION_GRANTED);
}

bool DistributedInputSourceStub::HasAccessDHPermission()
{
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE";
    int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    return (result == Security::AccessToken::PERMISSION_GRANTED);
}
```

**权限检查点**

| 操作 | 检查函数 | 错误码 | 文件位置 |
|------|---------|--------|---------|
| Init | `HasEnableDHPermission()` | `ERR_DH_INPUT_SRC_ENABLE_PERMISSION_CHECK_FAIL` | `:56-58` |
| Register | `HasEnableDHPermission()` | `ERR_DH_INPUT_SRC_ENABLE_PERMISSION_CHECK_FAIL` | `:96-98` |
| Prepare | `HasAccessDHPermission()` | `ERR_DH_INPUT_SRC_ACCESS_PERMISSION_CHECK_FAIL` | `:139-141` |
| Start | `HasAccessDHPermission()` | `ERR_DH_INPUT_SRC_ACCESS_PERMISSION_CHECK_FAIL` | `:179-181` |

#### 4.1.2 SA 权限配置

**证据**: `sa_profile/dinput.cfg`

```json
{
    "services": [{
        "name": "dinput",
        "uid": "dinput",
        "gid": ["dinput", "uhid", "input"],
        "apl": "system_basic",
        "permission": [
            "ohos.permission.DISTRIBUTED_DATASYNC",
            "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE"
        ],
        "secon": "u:r:dinput:s0"
    }]
}
```

### 4.2 输入验证机制

#### 4.2.1 参数校验

**校验实现**: `common/include/input_check_param.cpp`

| 校验项 | 限制值 | 错误处理 |
|--------|--------|---------|
| deviceId 长度 | > 0 && <= 256 | `DHLOGE` + 返回 false |
| dhId 长度 | <= 256 | `DHLOGE` + 返回 false |
| inputTypes | 1-7 (有效设备类型) | `DHLOGE` + 返回 false |
| 回调指针 | 非 null | `DHLOGE` + 返回 false |

**实现代码**

```cpp
if (deviceId.empty() || deviceId.size() > DEV_ID_LENGTH_MAX) {
    DHLOGE("CheckParam deviceId is empty or deviceId size too long.");
    return false;
}

if (inputTypes > static_cast<uint32_t>(DInputDeviceType::ALL) ||
    inputTypes == static_cast<uint32_t>(DInputDeviceType::NONE) ||
    !(inputTypes & static_cast<uint32_t>(DInputDeviceType::ALL))) {
    DHLOGE("CheckParam, inputTypes is invalids.");
    return false;
}
```

#### 4.2.2 IPC 数据校验

**IPC 向量大小限制**

```cpp
uint32_t vecSize = data.ReadUint32();
if (vecSize > IPC_VECTOR_MAX_SIZE) {
    DHLOGE("HandleStartDhidRemoteInput vecSize too large");
    return ERR_DH_INPUT_IPC_READ_VALID_FAIL;
}
```

**IPC 接口令牌校验**

```cpp
if (data.ReadInterfaceToken() != GetDescriptor()) {
    DHLOGE("DistributedInputSourceStub read token valid failed");
    return ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL;
}
```

### 4.3 传输层安全

#### 4.3.1 同账号校验

**实现**: `services/transportbase/src/softbus_permission_check.cpp`

```cpp
bool SoftBusPermissionCheck::CheckSrcPermission(const std::string &sinkNetworkId)
{
    AccountInfo localAccountInfo;
    if (!GetLocalAccountInfo(localAccountInfo)) {
        return false;
    }
#ifdef SUPPORT_SAME_ACCOUNT
    if (!CheckSrcIsSameAccount(sinkNetworkId, localAccountInfo)) {
        DHLOGE("Check src same account failed");
        return false;
    }
#endif
    return true;
}
```

#### 4.3.2 访问控制列表校验

```cpp
bool SoftBusPermissionCheck::CheckSrcAccessControl(const std::string &sinkNetworkId,
    const AccountInfo &localAccountInfo)
{
    DmAccessCaller caller = {
        .accountId = localAccountInfo.accountId_,
        .networkId = localAccountInfo.networkId_,
        .userId = localAccountInfo.userId_,
    };
    DmAccessCallee callee = { .networkId = sinkNetworkId };
    if (!DeviceManager::GetInstance().CheckSrcAccessControl(caller, callee)) {
        DHLOGE("Check src acl failed");
        return false;
    }
    return true;
}
```

### 4.4 日志与监控

#### 4.4.1 HiSysEvent 安全事件

**证据**: `hisysevent.yaml`

| 事件 | 类型 | 级别 | 说明 |
|------|------|------|------|
| `DINPUT_INIT` | BEHAVIOR | CRITICAL | SA 初始化 |
| `DINPUT_REGISTER` | BEHAVIOR | CRITICAL | 硬件注册 |
| `DINPUT_PREPARE` | BEHAVIOR | CRITICAL | 准备远程输入 |
| `DINPUT_START_USE` | BEHAVIOR | CRITICAL | 开始使用 |
| `DINPUT_STOP_USE` | BEHAVIOR | CRITICAL | 停止使用 |
| `DINPUT_INIT_FAIL` | FAULT | CRITICAL | 初始化失败 |
| `DINPUT_REGISTER_FAIL` | FAULT | CRITICAL | 注册失败 |
| `DINPUT_OPT_FAIL` | FAULT | CRITICAL | 操作失败 |

#### 4.4.2 日志脱敏

**证据**: 错误日志中使用 `GetAnonyString()` 进行脱敏

```cpp
DHLOGE("CheckDevice called, deviceId: %{public}s is not exist.", 
    GetAnonyString(devId).c_str());
```

---

## 5. 安全风险评估

### 5.1 风险清单

#### R1: 设备描述符注入风险 【中危】

**位置**: `common/include/constants_dinput.h:248-261`

**风险描述**: RawEvent 结构中的 `descriptor` 和 `path` 字段为字符串类型，在跨设备传输后可能被注入到系统中。

**触发条件**:
```
Sink 设备 → 构造恶意 RawEvent → 修改 descriptor 字段 
→ 发送到 Source → Source 注入事件时使用恶意 descriptor
```

**现有防护**:
- ✅ 设备描述符长度限制 (256B)
- ⚠️ 无内容白名单校验

**影响评估**:
- **可利用性**: 中 - 需要控制跨设备传输
- **权限提升**: 低 - 事件注入已有权限控制
- **影响范围**: 受限 - 虚拟设备隔离

**修复建议**:
```cpp
// 在 InjectInputEvent 前校验 descriptor 格式
bool IsValidDescriptor(const std::string& descriptor) {
    // 只允许字母、数字、下划线
    std::regex pattern("^[a-zA-Z0-9_]+$");
    return std::regex_match(descriptor, pattern);
}
```

#### R2: 组合键白名单绕过风险 【低危】

**位置**: `common/include/white_list_util.cpp`

**风险描述**: 白名单配置文件解析使用 `std::stoi()`，未处理异常情况。

**触发条件**:
```
攻击者 → 修改白名单配置文件 → 注入非数字字符
→ std::stoi 抛出异常 → 解析失败，可能导致绕过
```

**现有防护**:
- ✅ 路径规范化 (`realpath`)
- ⚠️ 无异常处理

**影响评估**:
- **可利用性**: 低 - 需要系统权限修改配置文件
- **权限提升**: 无

**修复建议**:
```cpp
// 使用安全的字符串转整数函数
int32_t SafeStrToInt(const std::string& str) {
    try {
        size_t pos;
        int32_t result = std::stoi(str, &pos);
        if (pos != str.length()) {
            return INVALID_VALUE;
        }
        return result;
    } catch (...) {
        return INVALID_VALUE;
    }
}
```

#### R3: 虚拟设备权限提升风险 【中危】

**位置**: `services/source/inputinject/src/virtual_device.cpp:127`

**风险描述**: Source SA 进程有权创建虚拟输入设备 (`/dev/uinput`)，如果存在漏洞，可用于权限提升。

**触发条件**:
```
攻击者 (有 dinput 权限) → 利用虚拟设备 API → 
→ 创建恶意设备 → 注入任意输入事件
```

**现有防护**:
- ✅ SA 运行于 dinput 用户
- ⚠️ 无设备创建速率限制
- ⚠️ 无设备属性白名单

**影响评估**:
- **可利用性**: 中 - 需要 dinput 权限
- **权限提升**: 中 - 可注入系统级输入事件
- **影响范围**: 高 - 可控制整个系统输入

**修复建议**:
```cpp
// 添加设备创建速率限制
static constexpr int MAX_DEVICES_PER_MINUTE = 10;
static std::map<uid_t, int> deviceCreationCount;

if (++deviceCreationCount[getuid()] > MAX_DEVICES_PER_MINUTE) {
    DHLOGE("Device creation rate exceeded");
    return ERR_DH_INPUT_RATE_LIMIT;
}
```

#### R4: 拒绝服务攻击风险 【低危】

**位置**: `services/common/include/dinput_softbus_define.h`

**风险描述**: `STRING_MAX_SIZE` 限制为 40MB，可能被用于触发大内存分配。

**现有防护**:
- ✅ MSG_MAX_SIZE = 32KB（实际消息限制）
- ⚠️ STRING_MAX_SIZE = 40MB（理论限制）

**影响评估**:
- **可利用性**: 低 - 需要 SoftBus 传输控制
- **影响范围**: 单次会话 DoS

**修复建议**:
考虑将 STRING_MAX_SIZE 降低到更合理的值（如 4KB）。

#### R5: 事件注入速率无限制 【低危】

**风险描述**: 未对跨设备事件注入速率进行限制，可能被用于事件洪泛攻击。

**影响评估**:
- **可利用性**: 低 - 需要 Sink 设备协作
- **影响范围**: 用户体验降级

**修复建议**:
```cpp
// 添加事件注入速率限制
static constexpr uint32_t MAX_EVENTS_PER_SECOND = 1000;
static EventRateLimiter rateLimiter;

if (!rateLimiter.TryAcquire()) {
    DHLOGE("Event injection rate exceeded");
    return ERR_DH_INPUT_RATE_LIMITED;
}
```

### 5.2 风险矩阵

| ID | 风险 | 可能性 | 影响 | 风险等级 | 状态 |
|----|------|--------|------|---------|------|
| R1 | 设备描述符注入 | 中 | 中 | **中危** | 需关注 |
| R2 | 白名单解析异常 | 低 | 低 | 低危 | 已知 |
| R3 | 虚拟设备权限提升 | 中 | 高 | **中危** | 需关注 |
| R4 | 大小限制过宽 | 低 | 低 | 低危 | 已知 |
| R5 | 事件速率无限制 | 低 | 低 | 低危 | 建议改进 |

---

## 6. 安全建议

### 6.1 短期改进（高优先级）

#### 6.1.1 添加事件注入速率限制

**优先级**: 高  
**工作量**: 1-2 人天

**建议**: 在 `DistributedInputInject` 类中添加事件注入速率限制器。

#### 6.1.2 完善白名单解析异常处理

**优先级**: 高  
**工作量**: 0.5 人天

**建议**: 将 `white_list_util.cpp:106-109` 的 `std::stoi()` 替换为带异常处理的版本。

### 6.2 中期改进（中优先级）

#### 6.2.1 添加设备描述符白名单

**优先级**: 中  
**工作量**: 2-3 人天

**建议**: 在事件注入前校验 descriptor 格式，只允许已注册的设备描述符。

#### 6.2.2 添加虚拟设备创建审计

**优先级**: 中  
**工作量**: 1 人天

**建议**: 记录所有虚拟设备创建操作到 HiSysEvent。

### 6.3 长期改进（建议）

| 改进项 | 说明 | 复杂度 |
|--------|------|--------|
| 设备沙箱 | 为虚拟设备创建独立的 uhid namespace | 高 |
| 事件签名 | 对跨设备事件进行数字签名 | 中 |
| 运行时监控 | 实时检测异常事件注入模式 | 中 |

---

## 7. 审计结论

### 7.1 总体评价

`distributed_input` 模块实现了较为完善的安全机制，包括：

| 方面 | 评估 | 说明 |
|------|------|------|
| **权限控制** | ✅ 良好 | AccessToken 双重校验 |
| **输入验证** | ✅ 良好 | 多层参数校验 |
| **传输安全** | ✅ 良好 | SoftBus 同账号校验 |
| **日志监控** | ✅ 良好 | HiSysEvent 全面覆盖 |
| **代码质量** | ✅ 良好 | 规范的安全编码实践 |

### 7.2 符合性检查

| 安全要求 | 符合性 | 证据 |
|---------|--------|------|
| 最小权限原则 | ✅ | 两个权限分别控制初始化和运行时 |
| 深度防御 | ✅ | 多层校验 (IPC + 参数 + 设备) |
| 安全默认值 | ✅ | 默认禁用、需要明确启用 |
| 审计追踪 | ✅ | HiSysEvent 完整覆盖 |

### 7.3 建议优先处理

1. **立即处理**: 白名单解析异常处理 (R2)
2. **本月处理**: 事件注入速率限制 (R5)
3. **下季度处理**: 设备描述符白名单 (R1)

---

## 附录

### A. 错误码参考

| 错误码 | 定义 | 含义 |
|--------|------|------|
| `-67000` | `ERR_DH_INPUT_IPC_INVALID_DESCRIPTOR` | IPC 描述符无效 |
| `-67045` | `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | IPC 写入令牌校验失败 |
| `-67046` | `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | IPC 读取令牌校验失败 |
| `-67047` | `ERR_DH_INPUT_IPC_READ_VALID_FAIL` | IPC 读取数据校验失败 |
| `-67061` | `ERR_DH_INPUT_SRC_ENABLE_PERMISSION_CHECK_FAIL` | Source ENABLE 权限校验失败 |
| `-67062` | `ERR_DH_INPUT_SRC_ACCESS_PERMISSION_CHECK_FAIL` | Source ACCESS 权限校验失败 |
| `-67063` | `ERR_DH_INPUT_SINK_ENABLE_PERMISSION_CHECK_FAIL` | Sink ENABLE 权限校验失败 |
| `-65042` | `ERR_DH_INPUT_SERVER_SOURCE_TRANSPORT_PERMISSION_DENIED` | 传输层权限被拒 |

### B. 相关文件索引

| 文件 | 说明 | 安全相关内容 |
|------|------|-------------|
| `interfaces/ipc/src/distributed_input_source_stub.cpp` | Source IPC 实现 | 权限校验 |
| `interfaces/ipc/src/distributed_input_sink_stub.cpp` | Sink IPC 实现 | 权限校验 |
| `services/transportbase/src/softbus_permission_check.cpp` | 传输权限检查 | 同账号校验 |
| `common/include/input_check_param.cpp` | 参数校验 | 输入验证 |
| `common/include/white_list_util.cpp` | 白名单解析 | 配置安全 |
| `services/source/inputinject/src/virtual_device.cpp` | 虚拟设备 | 设备注入 |
| `hisysevent.yaml` | 安全事件定义 | 监控覆盖 |

---

**文档版本**: v1.0  
**评审人**: OpenHarmony Security Wiki Generator  
**下次评审**: 建议每季度评审一次
