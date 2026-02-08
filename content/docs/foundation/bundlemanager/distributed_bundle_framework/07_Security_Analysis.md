# 安全分析

---

## 目的

本文档分析分布式包管理服务 (DBMS) 的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

---

## 适用范围

- ✅ 攻击面分析
- ✅ 信任边界定义
- ✅ 已知风险点（基于代码证据）
- ✅ 修复建议

---

## 威胁模型

### 外部输入 → 敏感操作

```
外部攻击者
    │
    ├─→ JS API 调用（参数注入、权限绕过）
    ├─→ IPC 伪造（接口令牌伪造）
    ├─→ 设备欺骗（device ID 伪造）
    └─→ 跨设备访问控制绕过（ACL 伪造）
              │
              ▼
         ┌─────────────┐
         │  DBMS 服务  │
         │  SA = 402    │
         └─────────────┘
              │
              ▼
         敏感操作
         ├─→ 权限验证
         ├─→ 跨设备数据访问
         ├─→ Bundle 信息查询
         └─→ 图标资源访问
```

---

## 攻击面分析

### 1. JS API 参数输入

| 攻击面 | 风险 | 证据 |
|---------|------|------|
| ElementName 参数注入 | deviceId, bundleName, abilityName 未做充分验证 | distributed_bundle_mgr.cpp:211-255 |
| 数组长度溢出 | 批量查询最大 10，但可能被绕过 | distributed_bundle.cpp:262-266 |
| locale 参数注入 | 未验证语言代码格式 | distributed_bundle_mgr.cpp:347-348 |

### 2. IPC 通信

| 攻击面 | 风险 | 证据 |
|---------|------|------|
| 接口令牌伪造 | 仅使用字符串比较，未加密验证 | distributed_bms_host.cpp:47-52 |
| Parcel 序列化注入 | 未做深度验证，可能构造恶意 Parcel | distributed_bms_proxy.cpp:123-131 |
| IPC 重放攻击 | 无请求 ID 或时间戳验证 | - |

### 3. 权限验证

| 攻击面 | 风险 | 证据 |
|---------|------|------|
| 权限绕过 | Native/Shell Token 可直接通过 | distributed_bms.cpp:592-608 |
| 权限提升 | 未充分验证调用者 Token 类型 | distributed_bms.cpp:610-636 |
| ACL 伪造 | 跨设备 ACL 信息可被构造 | dbms_device_manager.cpp:102-132 |

### 4. 跨设备安全

| 攻击面 | 风险 | 证据 |
|---------|------|------|
| Device ID 伪造 | 未验证 Device ID 真实性 | distributed_bms.cpp:660-661 |
| 信任设备列表污染 | 可添加恶意设备到信任列表 | dbms_device_manager.cpp:102-132 |
| 跨设备数据泄露 | 远程查询可能泄露敏感信息 | distributed_bms.cpp:510-546 |

### 5. 资源处理

| 攻击面 | 风险 | 证据 |
|---------|------|------|
| Base64 解码注入 | 手动 Base64 解码可能存在漏洞 | distributed_bms.cpp:548-590 |
| 图片资源路径遍历 | 图标文件路径未做严格验证 | distributed_bms.cpp:502-508 |
| 内存溢出 | EncodeBase64 函数未做长度检查 | distributed_bms.cpp:548-590 |

---

## 信任边界

### 边界 1：应用 → DBMS SA

**边界**: 通过 N-API 调用 DBMS 系统服务

**保护机制**:
- 权限验证：`VerifyCallingPermission()`
- Token 验证：`VerifySystemApp()`
- ACL 检查：`VerifyCallingPermissionOrAclCheck()`

**证据**: `services/dbms/src/distributed_bms.cpp:638-671`

### 边界 2：DBMS SA → Bundle Manager SA

**边界**: DBMS 查询本地 Bundle Manager 服务

**保护机制**:
- 通过系统 API 查询（自动权限）
- 无额外的权限验证（依赖 BMS 的权限检查）

**证据**: `services/dbms/src/distributed_bms.cpp:178-197`

### 边界 3：DBMS SA → Device Manager SA

**边界**: DBMS 查询设备信息

**保护机制**:
- 通过 Device Manager 系统服务查询
- 设备 ID 验证（依赖 Device Manager 的验证）

**证据**: `services/dbms/src/dbms_device_manager.cpp:87-100`

### 边界 4：跨设备 → 远程 DBMS SA

**边界**: 跨设备查询远程 DBMS 服务

**保护机制**:
- ACL 检查：`CheckAclData()`
- 跨设备验证

**证据**: `services/dbms/src/dbms_device_manager.cpp:102-132`

---

## 可被利用点

### 风险点 1：参数验证不足

**严重程度**: 🔴 高

**位置**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:211-255`

**问题**: ElementName 参数的 deviceId, bundleName, abilityName 字段仅做非空检查，未验证：
- 字符串长度限制
- 特殊字符过滤
- 格式规范验证

**可利用路径**:
```
1. 攻击者构造超长 deviceId/bundleName/abilityName
2. 导致内部缓冲区溢出或内存耗尽
3. 或注入特殊字符触发路径遍历
```

**影响**:
- 可能导致 DoS 攻击
- 可能绕过某些验证逻辑
- 可能导致信息泄露

**修复建议**:
```cpp
// 添加长度限制
const size_t MAX_DEVICE_ID_LENGTH = 128;
const size_t MAX_BUNDLE_NAME_LENGTH = 128;
const size_t MAX_ABILITY_NAME_LENGTH = 128;

if (deviceId.length() > MAX_DEVICE_ID_LENGTH) {
    return false;
}

// 添加格式验证
static bool IsValidString(const std::string &str) {
    // 检查是否包含恶意字符
    // 检查是否为空
    return !str.empty() && str.find_first_of("..") == std::string::npos;
}
```

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:211-255`

---

### 风险点 2：权限验证可绕过

**严重程度**: 🟠 中

**位置**: `services/dbms/src/distributed_bms.cpp:592-608`

**问题**: `VerifySystemApp()` 函数允许 Native Token 和 Shell Token 直接通过，未做额外验证

**可利用路径**:
```
1. 攻击者使用 Native Token 调用系统 API
2. 绕过权限检查，直接访问敏感数据
3. 或使用 Shell Token 进行系统级操作
```

**影响**:
- 权限提升
- 访问受限资源
- 绕过安全策略

**修复建议**:
```cpp
bool DistributedBms::VerifySystemApp()
{
    int32_t callingUid = IPCSkeleton::GetCallingUid();
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();

    // 即使是 Native/Shell Token，也应验证权限
    if (VerifyTokenNative(callerToken) || VerifyTokenShell(callerToken)) {
        // 添加额外的验证
        if (!VerifyCallingPermission(Constants::PERMISSION_GET_BUNDLE_INFO_PRIVILEGED)) {
            APP_LOGE("Native/Shell token still needs permission");
            return false;
        }
    }

    // 其他验证逻辑...
    return true;
}
```

**证据**: `services/dbms/src/distributed_bms.cpp:592-608`

---

### 风险点 3：ACL 检查可被绕过

**严重程度**: 🟠 中

**位置**: `services/dbms/src/dbms_device_manager.cpp:102-132`

**问题**: `CheckAclData()` 函数依赖外部 Device Manager 验证，如果 Device Manager 被攻击者控制，ACL 检查失效

**可利用路径**:
```
1. 攻击者控制或影响 Device Manager 服务
2. 绕过 ACL 检查，访问未授权的跨设备资源
3. 伪造信任设备列表
```

**影响**:
- 跨设备访问控制绕过
- 未授权的设备通信
- 数据泄露

**修复建议**:
```cpp
bool DbmsDeviceManager::CheckAclData(DistributedBmsAclInfo info)
{
#ifdef ACCOUNT_ENABLE
    // 添加本地验证，不只依赖外部服务
    if (info.networkId.empty() || info.accountId.empty() || info.pkgName.empty()) {
        APP_LOGE("Invalid ACL info");
        return false;
    }

    // 验证 networkId 格式
    if (!IsValidNetworkId(info.networkId)) {
        APP_LOGE("Invalid networkId format");
        return false;
    }

    // 原有逻辑...
    return DistributedHardware::DeviceManager::GetInstance().CheckAccessControl(dmSrecaller, dmDstCallee);
#else
    return false;
#endif
}
```

**证据**: `services/dbms/src/dbms_device_manager.cpp:102-132`

---

### 风险点 4：Base64 编码内存风险

**严重程度**: 🟡 中低

**位置**: `services/dbms/src/distributed_bms.cpp:548-590`

**问题**: `EncodeBase64()` 函数中：
- 未检查输入长度
- 使用固定大小的数组操作
- 可能在边界条件下导致问题

**可利用路径**:
```
1. 攻击者构造超大的图标数据
2. 触发 EncodeBase64() 内存问题
3. 可能导致缓冲区溢出或崩溃
```

**影响**:
- 服务崩溃
- 拒绝服务
- 可能的代码执行

**修复建议**:
```cpp
std::unique_ptr<char[]> DistributedBms::EncodeBase64(std::unique_ptr<uint8_t[]> &data, int srcLen)
{
    // 添加长度检查
    const int MAX_ENCODE_SIZE = 10 * 1024 * 1024; // 10MB
    if (srcLen < 0 || srcLen > MAX_ENCODE_SIZE) {
        APP_LOGE("Invalid data length: %{public}d", srcLen);
        return nullptr;
    }

    // 安全的数组分配
    // 使用安全函数替代手动计算
    std::vector<char> result;
    result.resize(outLen + DECODE_VALUE_ONE);

    // 使用标准库函数进行 Base64 编码
    // 而不是手动实现
}
```

**证据**: `services/dbms/src/distributed_bms.cpp:548-590`

---

### 风险点 5：设备 ID 验证不足

**严重程度**: 🟠 中

**位置**: `services/dbms/src/distributed_bms.cpp:660-661`

**问题**: `VerifyCallingPermissionOrAclCheck()` 仅检查 callingDeviceID 是否为空或为本地设备，未验证deviceId 格式和有效性

**可利用路径**:
```
1. 攻击者构造格式错误的 deviceId
2. 绕过本地设备检查
3. 可能触发后续验证逻辑的异常行为
```

**影响**:
- 逻辑绕过
- 未授权访问
- 状态不一致

**修复建议**:
```cpp
bool DistributedBms::VerifyCallingPermissionOrAclCheck(DistributedBmsAclInfo *info)
{
    DistributedHardware::DmDeviceInfo dmDeviceInfo;
    if (!GetLocalDevice(dmDeviceInfo)) {
        return false;
    }
    std::string callingDeviceID = IPCSkeleton::GetCallingDeviceID();

    // 添加 deviceId 格式验证
    if (!callingDeviceID.empty() && !IsValidDeviceId(callingDeviceID)) {
        APP_LOGE("Invalid callingDeviceID format");
        return false;
    }

    if (callingDeviceID.empty() || std::string(dmDeviceInfo.networkId) == callingDeviceID) {
        if (!VerifyCallingPermission(Constants::PERMISSION_GET_BUNDLE_INFO_PRIVILEGED)) {
            return false;
        }
    } else if (info != nullptr && !CheckAclData(*info)) {
        return false;
    }
    return true;
}
```

**证据**: `services/dbms/src/distributed_bms.cpp:660-661`

---

## 修复优先级

### 高优先级（🔴）

| 风险点 | 建议修复时间 | 修复复杂度 |
|---------|------------|----------|
| 参数验证不足 | 1 周 | 低（添加验证函数）|
| Base64 编码内存风险 | 2 周 | 中（重构编码逻辑）|

### 中优先级（🟠）

| 风险点 | 建议修复时间 | 修复复杂度 |
|---------|------------|----------|
| 权限验证可绕过 | 2 周 | 中（增强验证逻辑）|
| ACL 检查可被绕过 | 3 周 | 高（增加本地验证）|

### 低优先级（🟡）

| 风险点 | 建议修复时间 | 修复复杂度 |
|---------|------------|----------|
| 设备 ID 验证不足 | 4 周 | 低（添加格式验证）|

---

## 安全最佳实践

### 1. 输入验证

**建议**:
- ✅ 对所有外部输入做长度限制
- ✅ 验证输入格式（正则表达式）
- ✅ 检查特殊字符（.., /, 空字符等）
- ✅ 使用安全的字符串处理函数

**证据**: 基于 `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:211-255` 的分析

### 2. 权限验证

**建议**:
- ✅ 不要仅依赖 Token 类型判断
- ✅ 对所有敏感操作进行显式权限检查
- ✅ 记录权限拒绝事件到安全日志
- ✅ 使用最小权限原则

**证据**: 基于 `services/dbms/src/distributed_bms.cpp:592-608` 的分析

### 3. 跨设备安全

**建议**:
- ✅ 验证跨设备请求的合法性
- ✅ 不完全信任外部设备管理器
- ✅ 使用加密的设备标识符
- ✅ 实施设备白名单机制

**证据**: 基于 `services/dbms/src/dbms_device_manager.cpp:102-132` 的分析

### 4. 内存安全

**建议**:
- ✅ 使用标准库函数替代手动实现
- ✅ 对所有缓冲区操作做边界检查
- ✅ 使用智能指针管理内存生命周期
- ✅ 启用编译器的安全检查选项

**证据**: 基于 `services/dbms/src/distributed_bms.cpp:548-590` 的分析

### 5. IPC 安全

**建议**:
- ✅ 验证接口令牌
- ✅ 使用请求 ID 防止重放攻击
- ✅ 实施消息大小限制
- ✅ 记录异常 IPC 调用

**证据**: 基于 `services/dbms/src/distributed_bms_host.cpp:47-52` 的分析

---

## 权限清单

### SA 运行时权限

**文件**: `services/dbms/sa_profile/distributedbms.cfg:16-23`

| 权限 | 用途 |
|--------|------|
| ohos.permission.DISTRIBUTED_DATASYNC | 分布式数据同步 |
| ohos.permission.GET_BUNDLE_INFO_PRIVILEGED | 获取 Bundle 信息（特权）|
| ohos.permission.GET_INSTALLED_BUNDLE_LIST | 获取已安装列表 |
| ohos.permission.ACCESS_SERVICE_DM | 访问设备管理服务 |
| ohos.permission.MANAGE_LOCAL_ACCOUNTS | 管理本地账号 |
| ohos.permission.GET_BUNDLE_RESOURCES | 获取 Bundle 资源 |

**证据**: `services/dbms/sa_profile/distributedbms.cfg:16-23`

### API 级权限检查

**文件**: `services/dbms/src/distributed_bms.cpp`

| 方法 | 检查的权限 | 行号 |
|------|-----------|------|
| GetRemoteAbilityInfo | PERMISSION_GET_BUNDLE_INFO_PRIVILEGED | 257 |
| GetRemoteAbilityInfos | PERMISSION_GET_BUNDLE_INFO_PRIVILEGED | 297 |
| GetAbilityInfo | VerifyCallingPermissionOrAclCheck | 371 |
| GetAbilityInfos | VerifyCallingPermissionOrAclCheck | 476 |
| GetDistributedBundleInfo | PERMISSION_GET_BUNDLE_INFO_PRIVILEGED | 513 |
| GetDistributedBundleName | PERMISSION_GET_BUNDLE_INFO_PRIVILEGED | 532 |

**证据**: `services/dbms/src/distributed_bms.cpp:257, 297, 371, 476, 513, 532`

---

## 检查范围与局限性

### 已检查的代码范围

- ✅ JS API 参数解析（interfaces/kits/js/）
- ✅ IPC 接口和实现（interfaces/inner_api/, services/dbms/）
- ✅ 权限验证逻辑（services/dbms/src/distributed_bms.cpp）
- ✅ ACL 检查逻辑（services/dbms/src/dbms_device_manager.cpp）
- ✅ SA 配置（services/dbms/sa_profile/）

### 未检查的内容

- ❌ 外部依赖的安全实现（bundle_framework, device_manager 等）
- ❌ 网络通信加密层
- ❌ 设备认证的加密机制
- ❌ 测试用例中的安全场景

### 局限性

1. **时间点**: 安全分析基于 2026-02-06 的代码快照，后续代码变更可能引入新的风险或修复已有风险
2. **范围限制**: 未进行深入的安全审计或渗透测试
3. **依赖分析**: 假设外部依赖的实现是安全的

---

## 证据索引

| 风险点 | 证据来源 |
|---------|----------|
| 参数验证不足 | interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:211-255 |
| 权限验证可绕过 | services/dbms/src/distributed_bms.cpp:592-608 |
| ACL 检查可被绕过 | services/dbms/src/dbms_device_manager.cpp:102-132 |
| Base64 编码内存风险 | services/dbms/src/distributed_bms.cpp:548-590 |
| 设备 ID 验证不足 | services/dbms/src/distributed_bms.cpp:660-661 |
| SA 权限配置 | services/dbms/sa_profile/distributedbms.cfg:16-23 |
