# 安全风险评审

> **目的**: 分析 Work Scheduler 模块的安全攻击面和潜在风险
> **适用范围**: 安全审计、渗透测试、代码审查

---

## 攻击面清单

### 1. 对外 API 接口

| API | 暴露内容 | 风险级别 |
|-----|----------|----------|
| `startWork(workInfo)` | 注册延迟任务 | 🔴 高 |
| `stopWork(workInfo)` | 停止任务 | 🟡 中 |
| `getWorkStatus(workId)` | 查询任务状态 | 🟡 中 |
| `obtainAllWorks()` | 获取所有任务 | 🟡 中 |
| `stopAndClearWorks()` | 清空所有任务 | 🟡 中 |
| `isLastWorkTimeOut(workId)` | 检查超时 | 🟢 低 |

### 2. IPC 接口

| 接口 | 暴露内容 | 风险级别 |
|------|----------|----------|
| `IWorkSchedService` (SA 1904) | 13 个服务方法 | 🔴 高 |
| `IWorkScheduler` (Extension 回调) | 2 个回调方法 | 🟡 中 |

### 3. 系统交互

| 交互对象 | 交互内容 | 风险级别 |
|---------|----------|----------|
| **Bundle Manager** | 获取应用信息、验证 Ability | 🟡 中 |
| **IPC** | 跨进程通信 | 🔴 高 |
| **Ability Runtime** | 启动 Ability | 🟡 中 |
| **Common Event Service** | 订阅系统事件 | 🟡 中 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────┐
│                 不信任应用                    │
│  ┌────────────────────────────────────────┐          │
│  │ 应用进程（第三方应用）          │          │
│  └───────────┬────────────────────┘          │
│              ↓ IPC 调用                │
│  ┌────────────────────────────────────────┐          │
│  │ Work Scheduler Service (SA 1904)  │          │
│  │  - 权限检查                │          │
│  │  - 参数校验                │          │
│  │  - 身份验证                │          │
│  └───────────┬────────────────────┘          │
│              ↓ 系统服务调用                │
│  ┌────────────────────────────────────────┐          │
│  │ Bundle Manager, IPC, Ability      │          │
│  │ Runtime, Event Service         │          │
│  │  - 权限验证系统              │          │
│  └────────────────────────────────────┘          │
│                                              │
└──────────────────────────────────────────────────────┘

系统服务（受信）
- SAMGR
- Bundle Manager
- IPC
- Ability Runtime
- Event Handler
```

---

## 可被利用点

### 🔴 高风险点

#### 1. WorkId 假冒或篡改

**位置**: `interfaces/kits/js/napi/src/start_work.cpp:46`
**代码证据**:
```cpp
WorkInfo workInfo = WorkInfo();
if (Common::GetWorkInfo(env, argv[WORK_INFO_INDEX], workInfo)) {
    ErrCode errCode = WorkSchedulerSrvClient::GetInstance().StartWork(workInfo);
```

**可利用路径**:
```
恶意应用
  ↓
构造虚假 WorkId（如 0 或极小值）
  ↓
调用 startWork({workId: 0, bundleName: "com.victim.app", ...})
  ↓
WorkScheduler 服务接受并存储
  ↓
受害者应用调用 stopWork({workId: 0, ...}) 或 getWorkStatus(0)
  ↓
可能停止或查询到恶意应用的任务
```

**触发条件**:
- 应用未对 WorkId 进行有效范围检查
- WorkId 全局唯一性验证不足

**影响**:
- ✅ 任务劫持：恶意应用可以停止受害者的任务
- ✅ 信息泄露：可以查询其他应用的任务信息
- ✅ 拒绝服务：通过伪造任务占用资源

**修复建议**:
1. **加强 WorkId 校验**:
   ```cpp
   // 在 CheckWorkInfo 中添加更严格的验证
   if (workInfo.workId <= 0) {
       return E_WORKID_ERR;
   }
   ```

2. **实现 WorkId 命名空间隔离**:
   - 按 bundleName + workId 组合生成全局唯一 ID
   - 或者使用随机 UUID 替代简单数字

3. **增加所有权验证**:
   - 在服务端验证 bundleName 和 UID 的匹配关系
   - 防止跨应用任务操作

**证据位置**: `services/native/src/work_scheduler_service.cpp:661-678` (CheckWorkInfo)

---

#### 2. BundleName/abilityName 伪造攻击

**位置**: `interfaces/kits/js/napi/src/common.cpp:58` (GetWorkInfo)

**代码证据**:
```cpp
static bool GetBaseWorkInfo(napi_env env, napi_value objValue, WorkInfo& workInfo)
{
    workInfo.bundleName = GetStringProperty(env, objValue, "bundleName", errCode);
    workInfo.abilityName = GetStringProperty(env, objValue, "abilityName", errCode);
}
```

**可利用路径**:
```
恶意应用
  ↓
构造 WorkInfo
  bundleName: "com.victim.app"
  abilityName: "com.victim.app.WorkAbility"
  ↓
调用 startWork(workInfo)
  ↓
服务端验证通过（如果权限检查不足）
  ↓
系统尝试启动受害者的 Ability
  ↓
受害者 Ability 被恶意应用触发执行
```

**触发条件**:
- BundleName 和 AbilityName 仅做字符串存在性检查
- 未验证调用者是否是该包的所有者

**影响**:
- ✅ 能力滥用：恶意应用可以启动其他应用的 Ability
- ✅ 隐私泄露：可以触发其他应用的后台任务
- ✅ 权限提升：如果受害者 Ability 有更高权限

**修复建议**:
1. **强制 UID 匹配**:
   ```cpp
   // 在 CheckWorkInfo 中添加 UID 验证
   int32_t callerUid = IPCSkeleton::GetCallingUid();
   int32_t targetUid = GetUidByBundleName(workInfo.bundleName);
   if (callerUid != targetUid) {
       return E_CHECK_PERMISSION_FAILED;
   }
   ```

2. **扩展权限检查**:
   - 在启动 Ability 前检查调用者是否有该应用的权限
   - 使用 Access Token 验证包所有权

3. **增加 Bundle 签名验证**:
   - 验证调用者是否为系统应用或具有相同签名
   - 使用现有的 `IsInActiveGroupWhitelist` 机制

**证据位置**: `services/native/src/work_scheduler_service.cpp:661-678` (CheckWorkInfo)

---

#### 3. 参数类型混淆攻击

**位置**: `interfaces/kits/js/napi/src/common.cpp:217` (GetExtrasInfo)

**代码证据**:
```cpp
static bool GetExtrasInfo(napi_env env, napi_value objValue, WorkInfo& workInfo)
{
    napi_valuetype valueType = napi_undefined;
    napi_typeof(env, objValue, &valueType);
    // 缺少对 valueType 的严格检查
    // 直接解析为 parameters
}
```

**可利用路径**:
```
恶意应用
  ↓
构造恶意 parameters 对象
  - 包含 Object（应该只支持基本类型）
  - 包含复杂嵌套结构
  - 包含 Function（可能被执行）
  ↓
调用 startWork({parameters: maliciousObject})
  ↓
服务端解析 parameters 时可能
  - 触发类型转换漏洞
  - 解析序列化对象时造成内存破坏
  - 在 Ability 中执行恶意代码
```

**触发条件**:
- 参数类型检查不严格
- 缺少对嵌套对象的深度限制
- 缺少对 Object 大小的限制

**影响**:
- ✅ 拒绝服务：参数解析失败导致服务崩溃
- ✅ 远程代码执行：如果恶意对象在 Ability 中被反序列化并执行
- ✅ 内存破坏：导致服务进程崩溃或任意代码执行

**修复建议**:
1. **严格类型检查**:
   ```cpp
   static bool GetExtrasInfo(napi_env env, napi_value objValue, WorkInfo& workInfo)
   {
       if (MatchValueType(env, objValue, napi_object)) {
           // 检查对象的每个属性类型
           if (!IsValidParameterType(env, objValue)) {
               return false;
           }
       }
       // 解析...
   }
   ```

2. **限制参数大小**:
   ```cpp
   // 限制 parameters 对象的大小和复杂度
   if (GetParameterSize(env, objValue) > MAX_PARAMETERS_SIZE) {
       return false;
   }
   ```

3. **使用安全的序列化**:
   - 避免直接解析复杂对象
   - 使用标准的 Want 序列化机制
   - 在 Ability 侧验证参数结构

**证据位置**: `interfaces/kits/js/napi/include/common.h:218` (声明)

---

### 🟡 中风险点

#### 4. 频率限制绕过

**位置**: `services/native/src/work_sched_config.cpp:53-63`

**代码证据**:
```cpp
bool IsInActiveGroupWhitelist(const string& bundleName)
{
    // 检查 bundleName 是否在白名单中
    // 白名单是静态配置的，可能过时或不完整
    std::set<string> whitelist = {"com.example.app1", ...};
    return whitelist.find(bundleName) != whitelist.end();
}
```

**可利用路径**:
```
恶意应用
  ↓
频繁调用 startWork()
  ↓
应用被分配到 active group (最小间隔 2 小时)
  ↓
实际使用大量系统资源
  ↓
绕过频率限制机制
```

**触发条件**:
- 应用分组逻辑依赖于静态白名单
- 未正确识别应用的活跃度
- 应用频繁注册/注销以绕过分组机制

**影响**:
- ✅ 资源滥用：恶意应用可以频繁触发任务
- ✅ 系统性能下降：大量并发任务影响系统响应
- ✅ 耗电增加：频繁唤醒设备

**修复建议**:
1. **使用动态分组**:
   ```cpp
   // 从 Device Usage Statistics 实时获取应用活跃度
   // 而不是使用静态白名单
   std::string groupName = GetAppGroupFromStats(bundleName);
   int32_t minInterval = GetIntervalByGroup(groupName);
   ```

2. **限制任务并发数**:
   ```cpp
   // 添加全局任务并发限制
   if (GetRunningWorkCountByUid(uid) > MAX_CONCURRENT_WORKS) {
       return ERR_TOO_MANY_WORKS;
   }
   ```

3. **记录异常行为**:
   - 检测频繁的 startWork/stopWork 调用
   - 对可疑应用添加冷却时间
   - 上报 HiSysEvent 供审计

**证据位置**: `services/native/include/work_sched_config.h:15`

---

#### 5. 权限检查绕过

**位置**: `services/native/src/work_scheduler_service.cpp:1014-1023` (AllowDump)

**代码证据**:
```cpp
bool AllowDump()
{
    auto tokenID = IPCSkeleton::GetFirstTokenID();
    AccessTokenID accessToken = tokenID;
    // 仅检查 DUMP 权限
    uint64_t permissionFlag = 0;
    bool result = AccessTokenKit::VerifyAccessToken(accessToken, "ohos.permission.DUMP", permissionFlag);
    return result;
}
```

**可利用路径**:
```
恶意系统应用（或有漏洞的应用）
  ↓
获取 Native/Shell Token
  ↓
不声明 DUMP 权限
  ↓
调用内部接口（如 GetAllRunningWorks, PauseRunningWorks 等）
  ↓
绕过权限检查获取敏感信息
```

**触发条件**:
- 内部接口仅检查 Token 类型（Native/Shell）
- 未验证具体权限
- 系统应用权限范围过广

**影响**:
- ✅ 信息泄露：获取其他应用的任务信息
- ✅ 系统操控：暂停/恢复其他应用的任务
- ✅ 提权攻击：通过系统应用执行恶意操作

**修复建议**:
1. **细化权限检查**:
   ```cpp
   bool AllowDump()
   {
       // 添加更严格的权限检查
       // 不仅检查 Token 类型，还检查具体权限
       bool isNativeToken = CheckTokenType(tokenID);
       if (isNativeToken) {
           // Native 应用需要明确权限
           if (!HasPermission(tokenID, "ohos.permission.DUMP")) {
               return false;
           }
       }
       return true;
   }
   ```

2. **最小化内部接口暴露**:
   - 内部接口应最小化，仅用于系统组件
   - 移除不必要的 Get/Pause/Resume 接口
   - 使用明确的权限声明

3. **审计日志**:
   - 记录所有内部接口调用（包括调用者 UID）
   - 上报 HiSysEvent 供安全审计
   - 检测异常调用模式

**证据位置**: `services/native/src/work_scheduler_service.cpp:1014`

---

#### 6. 条件绕过攻击

**位置**: `services/native/src/conditions/condition_checker.cpp`

**代码证据**:
```cpp
bool ConditionChecker::CheckCondition(const WorkInfo& workInfo)
{
    // 检查条件是否满足
    // 可能存在逻辑错误，导致条件误判
    if (workInfo.networkType != NETWORK_TYPE_ANY) {
        if (!networkListener_->CheckNetworkType(workInfo.networkType)) {
            return false;
        }
    }
    // 其他条件检查...
}
```

**可利用路径**:
```
恶意应用
  ↓
构造 WorkInfo，设置宽松条件
  networkType: NETWORK_TYPE_ANY
  isCharging: false (任意状态)
  repeatCycleTime: 20 (最小值)
  ↓
调用 startWork()
  ↓
服务端条件检查通过
  ↓
任务立即执行（绕过调度意图）
  ↓
频繁执行，消耗资源
```

**触发条件**:
- 条件检查逻辑存在漏洞（如 OR 逻辑错误）
- 条件监听器返回错误状态
- 未考虑所有条件的组合情况

**影响**:
- ✅ 调度绕过：恶意任务可以立即执行
- ✅ 资源滥用：频繁执行消耗 CPU/内存
- ✅ 耗电增加：绕过系统节能策略

**修复建议**:
1. **严格条件验证**:
   ```cpp
   bool ConditionChecker::CheckCondition(const WorkInfo& workInfo)
   {
       // 确保所有条件都满足
       // 使用 AND 逻辑而非 OR
       // 验证每个条件的实际值
       if (!ValidateNetworkCondition(workInfo)) return false;
       if (!ValidateBatteryCondition(workInfo)) return false;
       // ...
   }
   ```

2. **添加条件互斥检查**:
   ```cpp
   // 检查冲突的条件设置
   if (workInfo.isRepeat && workInfo.repeatCount == 0) {
       return false; // 重复任务必须设置 repeatCount
   }
   ```

3. **条件状态验证**:
   - 从系统监听器获取真实状态
   - 与应用设置的条件进行比对
   - 不信任应用提供的参数

**证据位置**: `services/native/include/conditions/condition_checker.h`

---

### 🟢 低风险点

#### 7. 任务超时处理不当

**位置**: `services/native/src/work_scheduler_service.cpp` (Watchdog 实现)

**代码证据**:
```cpp
void Watchdog::WatchDogThread()
{
    // 监控任务执行时间
    if (workDuration > WATCHDOG_TIME) {
        // 强制停止任务
        StopWork(workInfo);
    }
}
```

**触发条件**:
- 任务执行超过 120 秒
- Watchdog 超时
- 系统负载高导致任务延迟

**影响**:
- 🟢 任务异常终止：正常任务被强制停止
- 🟢 数据丢失：任务未完成清理

**修复建议**:
- 提供任务超时配置
- 允许应用主动延长超时时间
- 提供超时前的警告回调

**证据位置**: `services/native/src/watchdog.cpp`

---

## 输入校验总结

### 当前校验机制

| 检查点 | 位置 | 严格程度 |
|---------|------|---------|
| WorkId 验证 | ❌ 仅非空检查 | 低 |
| BundleName 验证 | ❌ 仅字符串存在性 | 低 |
| AbilityName 验证 | ❌ 仅字符串存在性 | 低 |
| 条件设置 | 🟡 基本类型检查 | 中 |
| Parameters 类型 | 🟡 基本类型检查 | 中 |
| UID 匹配 | ✅ 在服务端检查 | 高 |
| 权限检查 | ✅ Access Token 验证 | 高 |

### 建议改进

1. **WorkId 规范化**:
   - 使用 UUID 或组合 ID (bundleName + timestamp + random)
   - 禁止使用简单数字

2. **强制所有权验证**:
   - 所有 API 必须验证调用者 UID 与目标 bundleName 匹配
   - 内部接口也需要添加 UID 检查

3. **参数深度限制**:
   - 限制 parameters 对象的嵌套层级
   - 限制对象总大小（如 4KB）
   - 白名单允许的参数键名

4. **条件状态真实性**:
   - 所有条件检查必须从系统监听器获取真实状态
   - 不信任应用传递的参数值

5. **频率限制增强**:
   - 动态获取应用活跃度（不依赖静态白名单）
   - 添加全局任务并发限制
   - 添加单应用任务总数限制

---

## 防御建议

### 代码层面

| 防御措施 | 实现难度 | 优先级 |
|-----------|-----------|-------|
| 参数类型严格校验 | 🟢 低 | 🔴 高 |
| UID/BundleName 匹配验证 | 🟡 中 | 🔴 高 |
| WorkId 命名空间隔离 | 🟢 低 | 🟡 中 |
| 条件真实性验证 | 🟡 中 | 🟡 中 |
| 权限细化检查 | 🟢 低 | 🟡 中 |
| 频率限制动态化 | 🟡 中 | 🟡 中 |
| 异常行为检测 | 🟢 低 | 🟡 中 |

### 运行时层面

| 防御措施 | 实现方式 | 优先级 |
|-----------|-----------|-------|
| SELinux 策略 | 系统 | 🔴 高 |
| 权限声明 | 应用配置 | 🔴 高 |
| 系统签名验证 | Bundle 签名 | 🟡 中 |
| 审计日志 | HiSysEvent | 🟡 中 |

---

## 安全测试建议

### 渗透测试场景

1. **WorkId 篡改测试**:
   - 使用 0、负数、极大值测试
   - 验证是否有竞态条件

2. **BundleName 伪造测试**:
   - 尝试启动其他应用的 Ability
   - 测试 UID 检查是否有效

3. **参数注入测试**:
   - 传递复杂对象、Function、Symbol
   - 测试大小限制

4. **频率限制绕过测试**:
   - 频繁调用 startWork
   - 测试冷却时间是否有效

5. **权限绕过测试**:
   - 尝试调用内部接口
   - 测试 Token 类型检查

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 模块概览
- [04_External_API.md](04_External_API.md) - API 文档

---

**证据索引**:

| 结论 | 证据 |
|------|------|
| 参数校验 | `interfaces/kits/js/napi/src/start_work.cpp:46` |
| 权限检查 | `services/native/src/work_scheduler_service.cpp:1014-1023` |
| 条件检查 | `services/native/include/conditions/condition_checker.h` |
| 频率限制 | `services/native/src/work_sched_config.cpp:53-63` |
| Watchdog | `services/native/src/watchdog.cpp` |
| UID 检查 | `services/native/src/work_scheduler_service.cpp:608-636` |
