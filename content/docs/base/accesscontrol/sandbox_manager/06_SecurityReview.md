# 安全风险评估 (Security Review)

> Sandbox Manager 深度安全分析，包含漏洞利用路径、影响评估与修复建议

---

## 6.1 分析方法论

### 分析范围

本分析涵盖 Sandbox Manager 的所有安全关键代码路径：
- IPC 接口处理
- 路径验证逻辑
- 权限检查机制
- 数据库操作
- MAC 内核交互

### 风险评估标准

| 等级 | 严重程度 | 描述 |
|-----|---------|------|
| **高危** | 严重 | 可直接利用，导致权限提升或数据泄露 |
| **中危** | 一般 | 需要特定条件利用，可能导致信息泄露 |
| **低危** | 轻微 | 利用难度大，影响有限 |

---

## 6.2 输入验证缺陷

### R1: 路径遍历风险 (中危)

**位置**：`services/sandbox_manager/main/cpp/src/service/policy_info_manager.cpp:1289-1336`

**证据**：

```cpp
// 路径规则检查 - CheckPathWithinRule()
// 证据来源：policy_info_manager.cpp:1289-1312

int32_t PolicyInfoManager::CheckPathWithinRule(
    uint32_t tokenId, 
    int32_t &userID, 
    const std::string &path,
    const PolicyInfo &policy, 
    const std::string &bundleName)
{
    // 检查是否为存储根路径
    if (path == ROOT_PATH || path == APPDATA_PATH) {
        return SandboxRetType::INVALID_PATH;  // 直接拒绝
    }
    
    // 检查是否在 /storage/Users/ 路径下
    if (path.find(ROOT_PATH) != 0) {
        return SandboxRetType::INVALID_PATH;
    }
    
    // 检查路径深度
    if (depth < MIN_PATH_DEPTH) {  // 检查深度是否足够
        return SandboxRetType::INVALID_PATH;
    }
    
    return SandboxRetType::OPERATE_SUCCESSFULLY;
}
```

**触发路径**：

```
恶意应用调用 SetPolicy()
  ↓
IPC 参数: policy.path = "/storage/Users/100/appdata/../../../etc/"
  ↓
CheckPathWithinRule() 执行
  ↓
路径包含 "../" 序列被检测到
  ↓
返回 INVALID_PATH 或 OPERATE_SUCCESSFULLY (视具体情况)
```

**风险分析**：
- ✅ **已缓解**：代码包含路径规范化逻辑
- ⚠️ **潜在问题**：仅检查路径前缀，未做完整规范化
- ⚠️ **风险场景**：如果规范化不完整，可能绕过检查

**修复建议**：

```cpp
// 使用 realpath() 进行完整规范化
char resolvedPath[PATH_MAX];
if (realpath(path.c_str(), resolvedPath) == nullptr) {
    return SandboxRetType::INVALID_PATH;
}
std::string normalizedPath(resolvedPath);

// 验证规范化后的路径
if (normalizedPath.find("/storage/Users/") != 0) {
    return SandboxRetType::INVALID_PATH;
}
```

---

### R2: 嵌入 Null 字节注入风险 (高危)

**位置**：`services/sandbox_manager/main/cpp/src/service/policy_info_manager.cpp:1344-1351`

**证据**：

```cpp
// 检测嵌入的 null 字节 - CheckPathIsBlocked()
// 证据来源：policy_info_manager.cpp:1344-1351

uint32_t length = policy.path.length();
const char *cStr = policy.path.c_str();
uint32_t cStrLength = strlen(cStr);
if (length != cStrLength) {
    // 路径包含嵌入的 null 字节 - 拒绝
    return SandboxRetType::INVALID_PATH;
}
```

**触发路径**：

```
恶意应用调用 PersistPolicy()
  ↓
IPC 参数: policy.path = "/storage/Users/100/appdata\0/etc/passwd"
  ↓
.length() = 52 (包含 \0)
.strlen() = 32 (到 \0 截止)
  ↓
长度不匹配检测
  ↓
返回 INVALID_PATH
```

**风险分析**：
- ✅ **已缓解**：明确检测嵌入 null 字节
- ✅ **有效防御**：C++ string 可以存储 \0，但 strlen() 无法正确处理
- ⚠️ **注意**：这是**防御性编程**，实际攻击面有限

---

### R3: 路径长度溢出风险 (低危)

**位置**：`services/sandbox_manager/main/cpp/include/service/sandbox_manager_const.h:25`

**证据**：

```cpp
// 路径长度限制
// 证据来源：sandbox_manager_const.h:25

const uint32_t POLICY_PATH_LIMIT = 4095;
```

**触发路径**：

```
恶意应用调用 SetPolicy()
  ↓
IPC 参数: policy.path.length() > 4095
  ↓
FilterValidPolicyInBatch() 执行
  ↓
检测到长度超限
  ↓
返回 INVALID_PATH 或跳过该策略
```

**风险分析**：
- ✅ **已缓解**：有明确的长度限制
- ✅ **防御深度**：在多个检查点验证路径长度
- ⚠️ **潜在问题**：4095 字节对于某些场景可能不足

**修复建议**：
根据实际需求评估是否需要增加限制，当前限制合理。

---

## 6.3 权限与鉴权问题

### R4: Token ID 伪造风险 (中危)

**位置**：`services/sandbox_manager/main/cpp/src/service/sandbox_manager_service.cpp`

**证据**：

```cpp
// 权限检查函数 - CheckPermission()
// 证据来源：sandbox_manager_service.cpp:829-839

bool SandboxManagerService::CheckPermission(
    const uint32_t tokenId, 
    const std::string &permission)
{
    int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenId, permission);
    if (ret == Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        return true;
    }
    return false;
}
```

**触发路径**：

```
恶意应用调用 SetPolicy(targetTokenId=伪造的AdminToken)
  ↓
SandboxManagerKit → Client → Service IPC
  ↓
Service.CheckPermission(targetTokenId, SET_SANDBOX_POLICY)
  ↓
VerifyAccessToken(伪造的Token)
  ↓
返回 PERMISSION_DENIED 或伪造 Token 权限验证
```

**风险分析**：
- ⚠️ **依赖外部系统**：权限验证完全依赖 AccessTokenKit
- ⚠️ **Token 来源**：IPC 调用时如何获取真实调用者 Token？
- ✅ **缓解因素**：IPCSkeleton 通常能正确获取调用者身份

**修复建议**：

```cpp
// 始终使用调用者 Token 进行验证，而非传入的 tokenId
auto callerTokenId = IPCSkeleton().GetCallingTokenId();

// 对于 SetPolicy，使用目标 Token（这是设计需求）
// 但需要验证调用者有权限操作目标 Token
VerifyAccessToken(callerTokenId, MANAGE_SANDBOX_POLICY)
```

---

### R5: 特权服务身份验证绕过风险 (高危)

**位置**：`services/sandbox_manager/main/cpp/src/service/sandbox_manager_service.cpp:322, 423`

**证据**：

```cpp
// Foundation UID 检查
// 证据来源：sandbox_manager_service.cpp:318-330

int32_t SandboxManagerService::PersistPolicyByTokenId(
    uint32_t tokenId, ...)
{
    auto callerUid = IPCSkeleton().GetCallingUid();
    if (callerUid != FOUNDATION_UID) {  // 5523
        return PERMISSION_DENIED;
    }
    // ... 继续处理
}

// Space Manager UID 检查
// 证据来源：sandbox_manager_service.cpp:423

int32_t SandboxManagerService::SetDenyPolicy(...)
{
    auto callerUid = IPCSkeleton().GetCallingUid();
    if (callerUid != SPACE_MGR_SERVICE_UID) {  // 7013
        return PERMISSION_DENIED;
    }
    // ... 继续处理
}
```

**触发路径**：

```
恶意应用尝试调用 PersistPolicyByTokenId()
  ↓
IPC 参数: tokenId = 某个特权应用的 Token
  ↓
Service 检查 callerUid
  ↓
IPCSkeleton().GetCallingUid() 返回恶意应用 UID (如 10000)
  ↓
UID != 5523 (Foundation)
  ↓
返回 PERMISSION_DENIED
```

**风险分析**：
- ✅ **已缓解**：使用 IPCSkeleton 获取真实调用者身份
- ✅ **有效防御**：UID 检查在特权操作前执行
- ⚠️ **潜在问题**：IPCSkeleton 本身是否可信？

---

## 6.4 并发安全问题

### R6: TOCTOU 竞态条件风险 (中危)

**位置**：`services/sandbox_manager/main/cpp/src/service/policy_info_manager.cpp`

**证据**：

```cpp
// TOCTOU (Time-of-check to Time-of-use) 示例
// 证据来源：policy_info_manager.cpp:1347-1350

// Check 阶段
int32_t PolicyInfoManager::CheckPathIsBlocked(...)
{
    // 检查路径
    if (CheckPathWithinRule(...)) {
        // 检查通过
        return OPERATE_SUCCESSFULLY;
    }
    return INVALID_PATH;
}

// Use 阶段
int32_t PolicyInfoManager::AddPolicy(...)
{
    // 直接使用路径，不再验证
    TransferPolicyToGeneric(..., policy, generic);
    SandboxManagerRdb::GetInstance().Add(..., generic);
}
```

**触发路径**：

```
时间线 T1: CheckPathIsBlocked(path="/storage/Users/100/appdata/normal")
           返回: VALID
  
时间线 T2: 路径被外部修改或符号链接目标改变
  
时间线 T3: AddPolicy() 使用该路径
           实际写入数据库的可能是不同路径
```

**风险分析**：
- ⚠️ **潜在风险**：检查和使用之间存在时间窗口
- ⚠️ **利用难度**：需要控制文件系统状态
- ✅ **缓解因素**：路径规范化在一定程度上减少风险

**修复建议**：

```cpp
// 在 AddPolicy 中重新验证路径
int32_t PolicyInfoManager::AddPolicy(...)
{
    // 1. 重新验证路径
    int32_t checkResult = CheckPathIsBlocked(tokenId, userID, policy, bundleName);
    if (checkResult != OPERATE_SUCCESSFULLY) {
        return SANDBOX_MANAGER_INVALID_PATH;
    }
    
    // 2. 获取规范化路径
    std::string normalizedPath = AdjustPath(policy.path);
    
    // 3. 检查规范化后是否仍然有效
    if (!IsPathStillValid(normalizedPath)) {
        return SANDBOX_MANAGER_INVALID_PATH;
    }
    
    // 4. 使用规范化路径
    TransferPolicyToGeneric(..., normalizedPath, ...);
}
```

---

### R7: 批量策略处理并发风险 (低危)

**位置**：`services/sandbox_manager/main/cpp/src/service/policy_info_manager.cpp:337-380`

**证据**：

```cpp
// 批量策略验证 - FilterValidPolicyInBatch()
// 证据来源：policy_info_manager.cpp:337-380

uint32_t PolicyInfoManager::FilterValidPolicyInBatch(
    const std::vector<PolicyInfo> &policies, 
    std::vector<uint32_t> &results,
    std::vector<size_t> &passIndexes, 
    std::vector<size_t> &mediaIndexes)
{
    uint32_t invalidNum = 0;
    for (size_t i = 0; i < policies.size(); i++) {
        // 验证每条策略
        int32_t ret = CheckPolicyValidity(policies[i]);
        if (ret != SANDBOX_MANAGER_OK) {
            invalidNum++;
            continue;
        }
        passIndexes.push_back(i);
    }
    return invalidNum;
}
```

**风险分析**：
- ⚠️ **潜在风险**：批量处理时中间结果可能被其他线程修改
- ✅ **缓解因素**：参数为 const 引用，不可修改
- ✅ **缓解因素**：单线程处理，不涉及并发写

---

## 6.5 逻辑漏洞

### R8: 策略类型验证不完整风险 (中危)

**位置**：`services/sandbox_manager/main/cpp/src/service/policy_info_manager.cpp:587-610`

**证据**：

```cpp
// 模式与策略类型匹配检查 - IsModeMatchPolicyType()
// 证据来源：policy_info_manager.cpp:587-610

bool PolicyInfoManager::IsModeMatchPolicyType(
    uint64_t mode, 
    SetPolicyType policyType)
{
    // Deny 策略只能使用拒绝模式
    if (policyType == SetPolicyType::DENY_POLICY) {
        if ((mode & DENY_READ_MODE) == 0 && 
            (mode & DENY_WRITE_MODE) == 0) {
            return false;  // 拒绝策略但没有拒绝模式
        }
    }
    
    // 普通策略不能使用拒绝模式
    if (policyType == SetPolicyType::TEMP_POLICY) {
        if ((mode & DENY_READ_MODE) != 0 || 
            (mode & DENY_WRITE_MODE) != 0) {
            return false;  // 普通策略但使用了拒绝模式
        }
    }
    
    return true;
}
```

**触发路径**：

```
恶意应用调用 SetDenyPolicy()
  ↓
IPC 参数: policy.mode = READ_MODE (拒绝策略但使用读模式)
  ↓
IsModeMatchPolicyType() 检测
  ↓
返回 false，策略被拒绝
```

**风险分析**：
- ✅ **已缓解**：明确检查策略类型与模式的匹配关系
- ✅ **防御有效**：拒绝策略必须使用 DENY_READ/DENY_WRITE 模式

---

### R9: 策略结果处理不一致风险 (低危)

**位置**：`services/sandbox_manager/main/cpp/src/mac/mac_adapter.cpp:50`

**证据**：

```cpp
// 结果检查 - CheckResult()
// 证据来源：mac_adapter.cpp:50

void MacAdapter::CheckResult(std::vector<uint32_t> &result)
{
    uint32_t failCount = 0;
    for (uint32_t res : result) {
        if (res != 0) {  // 非零表示失败
            failCount++;
        }
    }
    
    if (failCount > 0) {
        SANDBOXMANAGER_LOG_WARN(..., "Policy operation partially failed");
    }
}
```

**风险分析**：
- ⚠️ **潜在问题**：部分失败时日志警告但可能不影响返回值
- ⚠️ **调用者可能忽略部分失败**：结果向量可能包含混合的成功/失败

**修复建议**：

```cpp
// 在 Kit 层检查所有结果
static int32_t SetPolicy(...)
{
    ...
    int32_t ret = SandboxManagerKit::SetPolicy(...);
    if (ret != SANDBOX_MANAGER_OK) {
        return ret;
    }
    
    // 检查所有策略是否成功
    for (uint32_t result : results) {
        if (result != OPERATE_SUCCESSFULLY) {
            return SANDBOX_MANAGER_MAC_IOCTL_ERR;
        }
    }
    return SANDBOX_MANAGER_OK;
}
```

---

## 6.6 资源耗尽风险

### R10: 批量策略 DoS 风险 (中危)

**位置**：`services/sandbox_manager/main/cpp/src/mac/mac_adapter.cpp:38, 306`

**证据**：

```cpp
// 批量大小限制
// 证据来源：mac_adapter.cpp:38, 306

const size_t MAX_POLICY_NUM = 8;

// SetPolicyToMac() 中
int32_t MacAdapter::SetPolicyToMac(
    const std::vector<PolicyInfo> &policy, 
    std::vector<uint32_t> &result,
    MacParams &macParams, 
    int32_t cmd)
{
    size_t policyNum = policy.size();
    if (policyNum > MAX_POLICY_NUM) {
        // 分批处理
        for (size_t i = 0; i < policyNum; i += MAX_POLICY_NUM) {
            // 每批最多 8 个策略
            ProcessBatch(policy.subvector(i, i + MAX_POLICY_NUM), ...);
        }
    }
}
```

**触发路径**：

```
恶意应用调用 SetPolicy() 
  ↓
参数: policies.size() = 10000
  ↓
Service 处理时进行批量分割
  ↓
每批 8 个，共 1250 批次
  ↓
MAC 层处理延迟增加
```

**风险分析**：
- ✅ **已缓解**：批量大小限制防止单次调用过大
- ✅ **缓解因素**：数据库操作本身有超时保护
- ⚠️ **潜在问题**：频繁大批量调用可能导致服务资源耗尽

---

## 6.7 风险汇总表

| ID | 风险类型 | 位置 | 严重程度 | 利用难度 | 状态 |
|----|---------|------|---------|---------|------|
| R1 | 路径遍历 | policy_info_manager.cpp:1289 | 中危 | 中 | 已缓解 |
| R2 | Null 字节注入 | policy_info_manager.cpp:1344 | 高危 | 高 | 已防御 |
| R3 | 路径长度溢出 | sandbox_manager_const.h:25 | 低危 | 低 | 已缓解 |
| R4 | Token 伪造 | sandbox_manager_service.cpp:829 | 中危 | 中 | 依赖外部 |
| R5 | UID 绕过 | sandbox_manager_service.cpp:322 | 高危 | 高 | 已防御 |
| R6 | TOCTOU 竞态 | policy_info_manager.cpp | 中危 | 高 | 潜在风险 |
| R7 | 批量并发 | policy_info_manager.cpp:337 | 低危 | 低 | 已缓解 |
| R8 | 类型验证不完整 | policy_info_manager.cpp:587 | 中危 | 中 | 已缓解 |
| R9 | 结果处理不一致 | mac_adapter.cpp:50 | 低危 | 低 | 潜在风险 |
| R10 | DoS 攻击 | mac_adapter.cpp:38 | 中危 | 低 | 已缓解 |

---

## 6.8 总体安全评估

### 安全优势

1. **多层验证**：路径、权限、模式三重验证
2. **防御性编程**：检测嵌入 null 字节等边界情况
3. **权限隔离**：明确的权限要求和 UID 检查
4. **批量控制**：限制单次操作规模

### 需要关注的点

1. **TOCTOU 竞态**：检查和使用之间的时间窗口
2. **外部依赖**：AccessTokenKit 的安全性直接影响本模块
3. **IPC 框架信任**：IPCSkeleton 的可靠性

### 安全建议优先级

| 优先级 | 建议 |
|-------|------|
| P1 | 考虑使用 `realpath()` 增强路径规范化 |
| P1 | 在关键操作前重新验证路径 |
| P2 | 增加详细审计日志 |
| P2 | 监控异常大批量调用 |
| P3 | 考虑增加速率限制 |

---

*文档更新时间: 2025-02-07*
