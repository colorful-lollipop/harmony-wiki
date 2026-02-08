# 安全风险评估

**本文档对 Work Scheduler 模块进行深度安全分析，识别潜在风险并提供修复建议。**

---

## 目录

- [评估方法论](#评估方法论)
- [R1: 输入验证缺陷](#r1-输入验证缺陷)
- [R2: 权限与鉴权](#r2-权限与鉴权)
- [R3: 内存安全](#r3-内存安全)
- [R4: 并发安全](#r4-并发安全)
- [R5: 逻辑漏洞](#r5-逻辑漏洞)
- [风险矩阵](#风险矩阵)
- [修复建议汇总](#修复建议汇总)

---

## 评估方法论

### 评估维度

| 维度 | 检查内容 | 分析方法 |
|------|----------|----------|
| **输入验证** | 参数类型、范围、格式、编码 | 代码审查、边界分析 |
| **权限控制** | 身份验证、授权检查、访问控制 | 调用链分析、绕过测试 |
| **内存安全** | 缓冲区、生命周期、释放安全 | 模式匹配、危险函数识别 |
| **并发安全** | 竞态条件、死锁、TOCTOU | 锁分析、时序分析 |
| **逻辑漏洞** | 业务流程、异常处理、资源管理 | 控制流分析、状态机验证 |

### 风险评级标准

| 等级 | 定义 | 示例 |
|------|------|------|
| **🔴 严重** | 无需交互即可利用，可导致系统完全 compromise | 远程代码执行 |
| **🟠 高危** | 需要一定条件，可导致权限提升或数据泄露 | 权限绕过 |
| **🟡 中危** | 需要特定条件，影响有限或需要用户交互 | 拒绝服务 |
| **🟢 低危** | 影响极小，利用条件苛刻 | 信息泄露 |

---

## R1: 输入验证缺陷

### R1.1: WorkInfo JSON 解析风险

**风险等级**: 🟡 中危

**位置**: 
- `services/native/src/work_scheduler_service.cpp:430`
- `frameworks/src/work_info.cpp:550-597`

**证据**:
```cpp
// services/native/src/work_scheduler_service.cpp:430
nlohmann::json::parse(data, nullptr, false);  // 禁用异常
if (root.is_discarded()) {
    WS_HILOGE("Parse json failed");
    return false;
}
```

```cpp
// frameworks/src/work_info.cpp:553-597
bool WorkInfo::ParseFromJson(const nlohmann::json &value) {
    if (value.contains("workId") && value["workId"].is_number()) {
        workId_ = value["workId"].get<int32_t>();
    }
    // ... 其他字段解析
}
```

**触发路径**:
```
应用调用 startWork() 
→ N-API StartWork() [start_work.cpp:27]
→ WorkInfo JSON 序列化 [common.cpp]
→ IPC 传递到服务端
→ StartWorkInner() [work_scheduler_service.cpp]
→ ParseFromJson() 反序列化
→ 字段逐一解析
```

**问题分析**:
1. **缺乏 Schema 验证**: JSON 解析仅检查字段存在性和类型，不验证业务逻辑
2. **深度嵌套风险**: `parameters` 字段支持任意嵌套，可能导致递归深度过大
3. **数字溢出**: `is_number()` 不检查数值范围，可能导致整数溢出

**影响评估**:
- **可利用性**: 需要应用权限注册任务
- **影响**: 可能导致服务崩溃（DoS）或内存占用过高

**修复建议**:
```cpp
// 建议增加深度限制和范围检查
bool WorkInfo::ParseFromJson(const nlohmann::json &value, int maxDepth) {
    if (maxDepth <= 0) {
        WS_HILOGE("JSON nesting too deep");
        return false;
    }
    
    if (value.contains("workId")) {
        if (!value["workId"].is_number_integer()) {
            WS_HILOGE("workId must be integer");
            return false;
        }
        auto workId = value["workId"].get<int64_t>();
        if (workId < 0 || workId > INT32_MAX) {
            WS_HILOGE("workId out of range");
            return false;
        }
        workId_ = static_cast<int32_t>(workId);
    }
    // ...
}
```

---

### R1.2: 字符串长度未严格限制

**风险等级**: 🟢 低危

**位置**: `interfaces/kits/js/napi/src/common.cpp:375`

**证据**:
```cpp
// interfaces/kits/js/napi/src/common.cpp:375
char nameBuffer[NAME_MAXIMUM_LIMIT + 1] = {0};  // 128+1
napi_get_value_string_utf8(env, value, nameBuffer, NAME_MAXIMUM_LIMIT + 1, &len);
```

**问题分析**:
- 虽然设置了 128 字节限制，但 `napi_get_value_string_utf8` 在截断时不会报错
- 可能导致业务逻辑中 bundleName/abilityName 被静默截断

**修复建议**:
```cpp
// 获取实际长度并验证
napi_get_value_string_utf8(env, value, nullptr, 0, &len);
if (len > NAME_MAXIMUM_LIMIT) {
    WS_HILOGE("String too long: %zu", len);
    return ERR_INVALID_VALUE;
}
```

---

### R1.3: 路径遍历风险

**风险等级**: 🟡 中危

**位置**: `services/native/src/work_scheduler_service.cpp:428`

**证据**:
```cpp
// services/native/src/work_scheduler_service.cpp:428
bool WorkSchedulerService::GetJsonFromFile(const char *filePath, nlohmann::json& root) {
    std::string realPath;
    if (!ConvertFullPath(filePath, realPath)) {  // 使用 realpath 规范化
        WS_HILOGE("Convert full path failed");
        return false;
    }
    std::string data;
    if (!LoadStringFromFile(realPath.c_str(), data)) {
        return false;
    }
    // ...
}
```

**缓解措施**:
- 使用了 `ConvertFullPath()` → `realpath()` 进行路径规范化
- 路径限制在 `/data/service/el1/public/WorkScheduler/` 下

**潜在风险**:
- 如果 `ConvertFullPath` 存在漏洞，仍可能被绕过
- 符号链接攻击（如果目标目录存在符号链接）

**验证结果**: ✅ 已有适当缓解

---

## R2: 权限与鉴权

### R2.1: 进程名白名单可被绕过

**风险等级**: 🟡 中危

**位置**: `services/native/src/work_scheduler_service.cpp:1514-1527`

**证据**:
```cpp
// services/native/src/work_scheduler_service.cpp:1514
bool WorkSchedulerService::CheckProcessName() {
    std::string processName;
    // 获取调用者进程名
    if (!GetProcessNameByToken(GetCallingTokenID(), processName)) {
        return false;
    }
    // 白名单检查
    if (processName == "resource_schedule_service" || 
        processName == "hidumper_service") {
        return true;
    }
    return false;
}
```

**问题分析**:
1. **进程名伪造风险**: 某些情况下进程名可被修改或伪装
2. **缺乏二次验证**: 仅依赖进程名，未结合其他身份标识
3. **白名单过小**: 仅2-3个服务，容易被枚举

**影响评估**:
- **可利用性**: 需要已存在的系统服务被 compromise
- **影响**: 可暂停/恢复其他应用的任务调度

**修复建议**:
```cpp
// 建议增加多因子验证
bool CheckPrivilegedOperation() {
    // 1. 检查进程名
    if (!CheckProcessName()) return false;
    
    // 2. 检查 UID
    auto uid = GetCallingUid();
    if (uid != SYSTEM_UID && uid != ROOT_UID) return false;
    
    // 3. 检查 Token 类型
    if (!CheckCallingToken()) return false;
    
    // 4. 审计日志
    AuditLog("Privileged operation from: %s", processName.c_str());
    
    return true;
}
```

---

### R2.2: Dump 接口信息泄露

**风险等级**: 🟢 低危

**位置**: `services/native/src/work_scheduler_service.cpp:1014-1107`

**证据**:
```cpp
// services/native/src/work_scheduler_service.cpp:1016
bool WorkSchedulerService::AllowDump() {
    Security::AccessToken::AccessTokenID tokenId = GetCallingTokenID();
    int result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenId, "ohos.permission.DUMP");
    return result == Security::AccessToken::PERMISSION_GRANTED;
}
```

**信息泄露点**:
- 可获取所有任务的 bundleName、abilityName、UID
- 可获取能效资源白名单
- 可获取系统配置参数

**影响评估**:
- 需要 `ohos.permission.DUMP` 权限
- 通常只有系统应用和调试工具拥有

---

### R2.3: SA 任务停止缺乏细粒度控制

**风险等级**: 🟡 中危

**位置**: `services/native/src/work_scheduler_service.cpp:StopWorkForSA`

**证据**:
```cpp
// services/native/src/work_scheduler_service.cpp:1769
bool WorkSchedulerService::CheckCallingToken() {
    Security::AccessToken::AccessTokenID tokenId = GetCallingTokenID();
    auto tokenType = Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(tokenId);
    if (tokenType == Security::AccessToken::TOKEN_NATIVE ||
        tokenType == Security::AccessToken::TOKEN_SHELL) {
        return true;
    }
    return false;
}
```

**问题分析**:
- 任何 Native 或 Shell Token 都可以停止 SA 任务
- 缺乏 SA 级别的所有权检查

**修复建议**:
```cpp
ErrCode StopWorkForSA(int32_t saId) {
    // 检查调用者是否是该 SA 的所有者
    auto callerSaId = GetCallingSaId();
    if (callerSaId != saId && !IsSystemService()) {
        return ERR_PERMISSION_DENIED;
    }
    // ... 停止任务
}
```

---

## R3: 内存安全

### R3.1: FFI 边界内存管理

**风险等级**: 🟡 中危

**位置**: `interfaces/kits/cj/work_scheduler/work_scheduler_ffi.cpp`

**证据**:
```cpp
// interfaces/kits/cj/work_scheduler/work_scheduler_ffi.cpp
int32_t CJ_StartWork(RetWorkInfo workInfo) {
    // 转换 FFI 数据为内部类型
    auto workInfoPtr = std::make_shared<WorkInfo>();
    // ... 数据填充
    auto ret = WorkSchedulerSrvClient::GetInstance().StartWork(*workInfoPtr);
    return static_cast<int32_t>(ret);
}
```

**问题分析**:
- FFI 层涉及 C++ ↔ Cangjie 内存布局转换
- 复杂类型（如 WantParams）的转换可能存在内存对齐问题
- 缺乏显式的边界检查

**缓解措施**:
- 使用智能指针管理生命周期
- 通过 IPC 而非直接内存共享

---

### R3.2: Watchdog 超时处理

**风险等级**: 🟢 低危

**位置**: `services/native/src/watchdog.cpp`

**证据**:
```cpp
// services/native/src/watchdog.cpp
void Watchdog::StartWatchdog(const std::shared_ptr<WorkStatus>& workStatus) {
    // 启动定时器，120秒后触发
    auto timer = std::make_shared<TimerInfo>(...);
    timer->SetTimeout(120000);  // 120秒
    // ...
}
```

**安全考虑**:
- 超时机制防止资源滥用
- 但强制停止可能导致数据不一致

---

## R4: 并发安全

### R4.1: 多线程任务状态竞争

**风险等级**: 🟡 中危

**位置**: `services/native/src/work_scheduler_service.cpp`

**证据**:
```cpp
// services/native/include/work_scheduler_service.h:378-407
class WorkSchedulerService {
private:
    std::set<int32_t> whitelist_;
    ffrt::mutex whitelistMutex_;  // FFRT 互斥锁
    std::map<std::string, std::shared_ptr<WorkInfo>> persistedMap_;
    ffrt::mutex mutex_;  // 通用互斥锁
    std::atomic<bool> ready_ {false};
    // ...
}
```

**并发访问点**:
| 资源 | 锁保护 | 风险 |
|------|--------|------|
| `whitelist_` | `whitelistMutex_` | ✅ 已保护 |
| `persistedMap_` | `mutex_` | ✅ 已保护 |
| `deepIdleTimeMap_` | `deepIdleTimeMutex_` | ✅ 已保护 |
| `specialMap_` | `specialMutex_` | ✅ 已保护 |

**潜在风险**:
- `ready_` 原子变量使用正确
- 但某些操作序列可能存在逻辑竞争

---

### R4.2: IPC 回调死锁风险

**风险等级**: 🟢 低危

**位置**: `services/native/src/work_conn_manager.cpp`

**证据**:
```cpp
// services/native/src/work_conn_manager.cpp
void WorkConnManager::ConnectAbility(const std::shared_ptr<WorkStatus>& workStatus) {
    std::lock_guard<ffrt::mutex> lock(connMapMutex_);
    // ... 可能触发 IPC 调用
    abilityManager->ConnectAbility(...);
}
```

**分析**:
- 持有锁时进行 IPC 调用是危险模式
- 如果服务端回调需要同一把锁，会导致死锁

**修复建议**:
```cpp
void WorkConnManager::ConnectAbility(const std::shared_ptr<WorkStatus>& workStatus) {
    std::shared_ptr<WorkStatus> workCopy;
    {
        std::lock_guard<ffrt::mutex> lock(connMapMutex_);
        workCopy = workStatus;
        // 只进行必要的状态检查
    }
    // 释放锁后再进行 IPC 调用
    abilityManager->ConnectAbility(...);
}
```

---

## R5: 逻辑漏洞

### R5.1: 任务频率限制绕过

**风险等级**: 🟡 中危

**位置**: `services/native/src/work_policy_manager.cpp`

**证据**:
```cpp
// services/native/src/work_policy_manager.cpp
bool WorkPolicyManager::IsWorkFrequencyAllowed(const std::shared_ptr<WorkStatus>& workStatus) {
    // 检查应用分组
    int32_t group = GetAppGroup(workStatus->GetUid());
    // 根据分组返回允许/拒绝
}
```

**潜在绕过**:
1. **卸载重装**: 应用分组信息可能丢失，重装后重新计数
2. **UID 复用**: 如果 UID 被其他应用复用，可能继承频率限制
3. **系统时间篡改**: 时间检查可能受系统时间影响

---

### R5.2: 持久化任务恢复顺序

**风险等级**: 🟢 低危

**位置**: `services/native/src/work_scheduler_service.cpp`

**问题分析**:
```cpp
// 系统启动时
void WorkSchedulerService::RefreshPersistedWorks() {
    auto works = ReadPersistedWorks();  // 读取文件
    for (auto& work : works) {
        AddWorkInner(*work);  // 逐个添加
    }
}
```

**风险**:
- 大量持久化任务可能导致启动延迟
- 任务恢复顺序不可控，可能导致资源竞争

---

## 风险矩阵

| 风险ID | 类型 | 等级 | 利用难度 | 影响范围 | 修复优先级 |
|--------|------|------|----------|----------|------------|
| R1.1 | JSON解析 | 🟡 中 | 中 | 服务可用性 | P2 |
| R1.2 | 字符串处理 | 🟢 低 | 高 | 功能异常 | P3 |
| R1.3 | 路径遍历 | 🟢 低 | 高 | 文件访问 | P3 |
| R2.1 | 权限绕过 | 🟡 中 | 高 | 任务控制 | P2 |
| R2.2 | 信息泄露 | 🟢 低 | 高 | 敏感信息 | P3 |
| R2.3 | 越权操作 | 🟡 中 | 中 | 任务管理 | P2 |
| R3.1 | FFI内存 | 🟡 中 | 中 | 内存安全 | P2 |
| R3.2 | 资源管理 | 🟢 低 | 低 | 可用性 | P3 |
| R4.1 | 并发竞争 | 🟢 低 | 高 | 数据一致性 | P3 |
| R4.2 | 死锁 | 🟢 低 | 低 | 可用性 | P3 |
| R5.1 | 逻辑绕过 | 🟡 中 | 中 | 策略控制 | P2 |
| R5.2 | 性能问题 | 🟢 低 | 低 | 启动性能 | P3 |

---

## 修复建议汇总

### 高优先级 (P1-P2)

| 建议 | 目标风险 | 实现难度 | 预期收益 |
|------|----------|----------|----------|
| 增加 JSON Schema 验证 | R1.1 | 中 | 防止畸形输入 |
| 强化特权操作验证 | R2.1, R2.3 | 低 | 防止权限绕过 |
| FFI 边界检查增强 | R3.1 | 高 | 提高内存安全 |
| 应用分组防绕过 | R5.1 | 中 | 保证策略有效性 |

### 中优先级 (P3)

| 建议 | 目标风险 | 实现难度 |
|------|----------|----------|
| 字符串长度显式检查 | R1.2 | 低 |
| Dump 接口数据脱敏 | R2.2 | 低 |
| IPC 调用锁优化 | R4.2 | 中 |
| 启动任务分批加载 | R5.2 | 低 |

---

**文档版本**: 1.0  
**更新日期**: 2026-02-07  
**分析范围**: 排除测试代码，覆盖核心服务实现
