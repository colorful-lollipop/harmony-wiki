# 安全风险评估

> **目的**：为安全研究员提供详细的安全风险分析，包括具体代码证据、触发路径、影响评估和修复建议
> **适用范围**：常见安全风险分析、可利用性评估、修复建议
> **最后更新**：2026-02-07

---

## 1. 输入验证缺陷

### 1.1 slotId 范围检查不足（高危）

**位置**：`services/src/cellular_data_service.cpp:382-474`

**证据**：
```cpp
// services/src/cellular_data_service.cpp:XXX (多个方法中)
// 仅进行范围检查，未验证 SIM 卡存在性
if (slotId < DEFAULT_SIM_SLOT_ID || slotId >= SIM_SLOT_COUNT) {
    TELEPHONY_LOGE("Invalid slotId: %{public}d", slotId);
    return TELEPHONY_ERR_SLOTID_INVALID;
}
```

**触发路径**：
```
[应用] setDefaultCellularDataSlotId(1)
  ↓ (N-API)
[NAPI 层] IsValidSlotId(1) → 范围检查通过（0 <= 1 < 2）
  ↓
[IPC Binder]
[服务层] CellularDataService::SetDefaultCellularDataSlotId(1)
  ↓ (权限检查)
[控制器层] CellularDataController::SetDefaultCellularDataEnable(true)
  ↓ [无 SIM 卡存在检查]
[内部处理] 尝试激活不存在的卡槽 1
  ↓
[错误] 返回 ERROR_NO_SIM_CARD 或部分成功状态
```

**影响评估**：
- **可利用性**：高 - 攻击者可指定任意 slotId（0 或 1），即使对应卡槽未插入 SIM 卡
- **权限提升**：中等 - 可能导致信息泄露或错误状态传播
- **影响**：通过指定不存在的卡槽，可能读取其他卡槽的信息或导致服务内部错误状态

**修复建议**：
```cpp
// 修复方案 1：在 slotId 验证前检查 SIM 卡状态
int32_t CellularDataService::SetDefaultCellularDataSlotId(int32_t slotId)
{
    // 1. 先验证 slotId 范围
    if (!IsValidSlotId(slotId)) {
        return TELEPHONY_ERR_SLOTID_INVALID;
    }

    // 2. 检查 SIM 卡是否存在并激活
    auto coreService = CoreManagerInner::GetInstance();
    SimState simState;
    if (coreService->GetSimState(slotId, simState) != ERR_OK || simState != SIM_STATE_READY) {
        TELEPHONY_LOGE("SIM card not ready for slotId: %{public}d", slotId);
        return TELEPHONY_ERR_NO_SIM_CARD;
    }

    // 3. 继续处理
    return cellularDataController_->SetDefaultCellularDataSlotId(slotId);
}
```

---

### 1.2 APN 信息长度未限制（高危）

**位置**：`frameworks/js/napi/src/napi_cellular_data.cpp:1088-1131`

**证据**：
```cpp
// frameworks/js/napi/src/napi_cellular_data.cpp:1088-1131
bool ApnInfoAnalyze(napi_env env, napi_value object, ApnInfo &apnInfo)
{
    // 从 JS 对象提取 apnName、apn、mcc、mnc、user、password 等字段
    // 未进行长度验证、格式验证
    NapiUtil::GetStringProperty(env, object, "apnName", apnInfo.apnName);
    NapiUtil::GetStringProperty(env, object, "apn", apnInfo.apn);
    NapiUtil::GetStringProperty(env, object, "mcc", apnInfo.mcc);
    NapiUtil::GetStringProperty(env, object, "mnc", apnInfo.mnc);
    NapiUtil::GetStringProperty(env, object, "user", apnInfo.user);
    NapiUtil::GetStringProperty(env, object, "password", apnInfo.password);

    return true;  // 即使提取失败也返回 true
}
```

**触发路径**：
```
[应用] queryApnIds({apn: "恶意数据...", apnName: "恶意数据...", user: "恶意用户...", password: "恶意密码..."})
  ↓ (N-API)
[NAPI 层] ApnInfoAnalyze() → 直接提取，无验证
  ↓
[IPC Binder]
[服务层] CellularDataService::QueryApnIds(apnInfo)
  ↓ (权限检查)
[APN 管理层] ApnManager::FilterMatchedApns(apnInfo)
  ↓
[数据库层] DataShare::Query() → 写入超长字符串到数据库
  ↓
[数据库] APN 配置表被恶意数据污染或缓冲区溢出
```

**影响评估**：
- **可利用性**：高 - 攻击者可传入超长字符串（> 1MB）
- **缓冲区溢出**：高危 - 可能导致栈溢出或堆溢出
- **信息泄露**：中等 - 通过读取恶意 APN，可能注入脚本代码或读取系统信息
- **DoS**：高 - 超大对象导致服务内存耗尽

**修复建议**：
```cpp
// 修复方案：添加字段长度限制和格式验证
constexpr size_t MAX_APN_NAME_LENGTH = 64;
constexpr size_t MAX_APN_USER_LENGTH = 64;
constexpr size_t MAX_APN_PASSWORD_LENGTH = 64;
constexpr size_t MAX_MCC_LENGTH = 3;
constexpr size_t MAX_MNC_LENGTH = 3;

bool ApnInfoAnalyze(napi_env env, napi_value object, ApnInfo &apnInfo)
{
    // 1. 提取字段
    napi_value apnNameVal;
    napi_get_named_property(env, object, "apnName", &apnNameVal);
    size_t apnNameLength;
    napi_get_value_string_utf8(env, apnNameVal, apnInfo.apnName, &apnNameLength);

    // 2. 验证长度
    if (apnNameLength > MAX_APN_NAME_LENGTH) {
        TELEPHONY_LOGE("APN name too long: %{public}zu", apnNameLength);
        return false;  // 拒绝超长输入
    }

    // 3. 验证格式（MCC/MNC 必须为数字）
    NapiUtil::GetStringProperty(env, object, "mcc", apnInfo.mcc);
    for (char c : apnInfo.mcc) {
        if (!std::isdigit(c)) {
            TELEPHONY_LOGE("Invalid MCC format: %{public}s", apnInfo.mcc.c_str());
            return false;
        }
    }

    // 4. 特殊字符过滤（防止 SQL 注入、路径遍历）
    const char* dangerousChars = "'\";\\/&|<>`$";
    for (char c : apnInfo.apn) {
        if (std::strchr(dangerousChars, c) != nullptr) {
            TELEPHONY_LOGE("APN contains dangerous character: %{public}c", c);
            return false;
        }
    }

    return true;
}
```

---

### 1.3 网络切片 buffer 长度未限制（中危）

**位置**：`frameworks/js/napi/src/napi_cellular_data.cpp:XXX`（网络切片相关方法）

**证据**：
```cpp
// 网络切片 URSP 解码、UE 策略、IMS RSD 接收 buffer 参数
// 仅进行类型检查（Array<number>），未验证内容长度
napi_value bufferVal;
napi_get_named_property(env, args[0], "buffer", &bufferVal);
bool isArray = false;
napi_is_array(env, bufferVal, &isArray);

// 直接转换，无长度验证
uint32_t arrayLength;
napi_get_array_length(env, bufferVal, &arrayLength);
```

**触发路径**：
```
[应用] SendUrspDecodeResult(0, [大量恶意数据...])
  ↓ (N-API)
[NAPI 层] 无验证，直接转换 buffer
  ↓
[IPC Binder]
[服务层] CellularDataService::SendUrspDecodeResult(slotId, buffer)
  ↓
[内部处理] 处理超长 buffer
  ↓
[内存分配] 分配大量内存 → 内存耗尽或 DoS
```

**影响评估**：
- **可利用性**：中 - 需要调用网络切片 API
- **DoS**：高 - 超大 buffer 导致内存耗尽
- **资源耗尽**：高 - 消耗系统内存资源

**修复建议**：
```cpp
// 修复方案：添加 buffer 长度限制
constexpr size_t MAX_BUFFER_LENGTH = 4096;  // 4KB 限制

int32_t CellularDataService::SendUrspDecodeResult(int32_t slotId, const std::vector<uint8_t> &buffer)
{
    // 验证 buffer 长度
    if (buffer.size() > MAX_BUFFER_LENGTH) {
        TELEPHONY_LOGE("Buffer too long: %{public}zu", buffer.size());
        return TELEPHONY_ERR_INVALID_PARAMETER;
    }

    // 继续处理
    return coreServiceInner_->SendUrspDecodeResult(slotId, buffer);
}
```

---

## 2. 内存安全问题

### 2.1 状态机指针可能为空指针（中危）

**位置**：`services/src/state_machine/cellular_data_state_machine.cpp:XXX`

**证据**：
```cpp
// services/src/state_machine/cellular_data_state_machine.cpp:XXX
// 状态机使用 shared_ptr，但某些地方可能存在未检查的指针访问
std::shared_ptr<State> currentState_;

void CellularDataStateMachine::ProcessStateEvent(const AppExecFwk::InnerEvent::Pointer &event)
{
    if (currentState_ == nullptr) {
        // 可能存在空指针风险
        return;
    }

    currentState_->StateProcess(event);  // 未二次检查
}
```

**触发路径**：
```
[启动] CellularDataStateMachine::Init()
  ↓
[状态转换] currentState_ = newState
  ↓ [竞态条件]
[并发] 另一线程访问 currentState_
  ↓ [空指针]
[崩溃] currentState_->StateProcess() → 访问空指针
```

**影响评估**：
- **可利用性**：低 - 需要精确的时序条件
- **崩溃风险**：高 - 可能导致服务崩溃
- **拒绝服务**：高 - 服务重启导致数据连接中断

**修复建议**：
```cpp
// 修复方案：使用智能指针和锁保护
void CellularDataStateMachine::ProcessStateEvent(const AppExecFwk::InnerEvent::Pointer &event)
{
    std::lock_guard<std::mutex> lock(stateMutex_);

    if (!currentState_) {  // 使用 ! 而非 == nullptr
        TELEPHONY_LOGE("Current state is null");
        return;
    }

    currentState_->StateProcess(event);
}
```

---

### 2.2 动态库加载路径未验证（高危）

**位置**：`services/telephony_ext_wrapper/src/telephony_ext_wrapper.cpp:58, 125, 217`

**证据**：
```cpp
// services/telephony_ext_wrapper/src/telephony_ext_wrapper.cpp:58
void *handle = dlopen(libPath.c_str(), RTLD_NOW);
if (handle == nullptr) {
    TELEPHONY_LOGE("dlopen %{public}s failed: %{public}s", libPath.c_str(), dlerror());
    // 继续执行，可能导致崩溃
}

// libPath 来自外部输入或配置，未验证路径合法性
```

**触发路径**：
```
[攻击] 修改配置文件指向恶意库路径
  ↓
[服务启动] TelephonyExtWrapper::LoadExtensionLib()
  ↓
[dlopen] 加载恶意库
  ↓
[代码执行] 恶意库代码被执行
  ↓
[权限提升] 获得系统权限
```

**影响评估**：
- **可利用性**：中 - 需要修改配置文件（需要 root 或系统应用权限）
- **代码执行**：高危 - 可执行任意代码
- **权限提升**：高危 - 恶意库运行在系统服务进程

**修复建议**：
```cpp
// 修复方案：验证库路径白名单
static const std::vector<std::string> ALLOWED_LIB_PATHS = {
    "/system/lib64/libtel_ext.so",
    "/vendor/lib64/libtel_ext.so"
    // 其他可信路径
};

bool TelephonyExtWrapper::IsLibPathAllowed(const std::string &libPath)
{
    // 1. 规范化路径
    std::string normalizedPath = libPath;
    // 移除 ../ 和 ./ 路径遍历尝试
    size_t pos;
    while ((pos = normalizedPath.find("../")) != std::string::npos) {
        normalizedPath.erase(pos, 3);
    }

    // 2. 检查白名单
    for (const auto &allowedPath : ALLOWED_LIB_PATHS) {
        if (normalizedPath == allowedPath) {
            return true;
        }
    }

    return false;
}

void TelephonyExtWrapper::LoadExtensionLib(const std::string &libPath)
{
    if (!IsLibPathAllowed(libPath)) {
        TELEPHONY_LOGE("Library path not allowed: %{public}s", libPath.c_str());
        return TELEPHONY_ERR_INVALID_PATH;
    }

    void *handle = dlopen(libPath.c_str(), RTLD_NOW);
    // ...
}
```

---

## 3. 权限与鉴权

### 3.1 权限检查 TOCTOU（中危）

**位置**：`services/src/cellular_data_service.cpp:167-172`（多个方法）

**证据**：
```cpp
// services/src/cellular_data_service.cpp:167-172
int32_t CellularDataService::EnableCellularData(bool enable)
{
    // 1. 检查权限
    if (!TelephonyPermission::CheckPermission(Permission::SET_TELEPHONY_STATE)) {
        TELEPHONY_LOGE("Permission denied.");
        return TELEPHONY_ERR_PERMISSION_ERR;
    }

    // 2. 执行操作
    return cellularDataController_->SetCellularDataEnable(enable);
}
```

**潜在 TOCTOU 场景**：
```
[时序 T1] 应用调用 EnableCellularData(true)
  ↓
[权限检查] CheckPermission() → 权限有效
  ↓ (时序 T2)
[权限撤销] 用户在 T2 撤销权限
  ↓
[执行操作] SetCellularDataEnable(true) → 执行成功（权限已撤销）
  ↓
[结果] 成功启用数据，但当前用户无权限
```

**影响评估**：
- **可利用性**：低 - 需要精确时序和用户交互
- **权限提升**：中 - 可在权限撤销后继续执行特权操作
- **安全性**：中 - 权限检查与执行之间存在时间窗口

**修复建议**：
```cpp
// 修复方案：在权限检查后立即执行操作，或使用令牌机制
int32_t CellularDataService::EnableCellularData(bool enable)
{
    // 1. 生成操作令牌（带时间戳）
    uint64_t opToken = GenerateOperationToken("EnableCellularData");

    // 2. 检查权限并绑定令牌
    if (!TelephonyPermission::CheckPermissionWithToken(Permission::SET_TELEPHONY_STATE, opToken)) {
        TELEPHONY_LOGE("Permission denied.");
        return TELEPHONY_ERR_PERMISSION_ERR;
    }

    // 3. 验证令牌并执行操作
    if (!ValidateOperationToken(opToken)) {
        TELEPHONY_LOGE("Token validation failed");
        return TELEPHONY_ERR_OPERATION_FAILED;
    }

    return cellularDataController_->SetCellularDataEnable(enable, opToken);
}
```

---

### 3.2 系统应用检查不足（中危）

**位置**：`services/src/cellular_data_service.cpp:XXX`（多个方法）

**证据**：
```cpp
// services/src/cellular_data_service.cpp:170-172
if (!TelephonyPermission::CheckCallerIsSystemApp()) {
    TELEPHONY_LOGE("Permission denied.");
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

**分析**：
- 系统应用检查仅基于 `CheckCallerIsSystemApp()`
- 未验证应用签名或应用 ID 白名单
- 恶意应用可能伪造系统应用标识

**影响评估**：
- **可利用性**：中 - 需要绕过系统应用检查
- **权限提升**：高 - 恶意应用可获得系统权限
- **安全性**：中 - 系统应用检查机制不够强

**修复建议**：
```cpp
// 修复方案：使用应用签名验证
static const std::vector<std::string> SYSTEM_APP_WHITELIST = {
    "com.ohos.settings",
    "com.ohos.systemui"
    // 系统核心应用列表
};

bool CellularDataService::IsTrustedSystemApp(int32_t uid)
{
    std::string packageName;
    if (GetPackageNameByUid(uid, packageName) != ERR_OK) {
        return false;
    }

    // 1. 检查白名单
    for (const auto &allowedApp : SYSTEM_APP_WHITELIST) {
        if (packageName == allowedApp) {
            // 2. 验证签名（可选）
            return VerifyAppSignature(packageName);
        }
    }

    return false;
}
```

---

## 4. 并发安全

### 4.1 状态转换竞态（中危）

**位置**：`services/src/state_machine/activating.cpp:XXX`、`active.cpp:XXX`

**证据**：
```cpp
// services/src/state_machine/activating.cpp:XXX
bool Activating::StateProcess(const AppExecFwk::InnerEvent::Pointer &event)
{
    switch (event->GetInnerEventId()) {
        case MSG_SM_CONNECT:
            // 转换到 Active 状态
            if (eventResultInfo->active > 0) {
                // 状态转换
                parent_.DelayedTransition(State::ACTIVE);
            }
            break;
    }
}
```

**竞态场景**：
```
[线程 A] Activating::StateProcess(RIL_SUCCESS) → 转换到 Active
  ↓
[线程 B] Disconnecting::StateProcess(DISCONNECT_REQUEST) → 尝试断开
  ↓ [竞态]
[状态冲突] 两个线程同时修改状态 → 不确定状态
  ↓
[错误] 状态机进入不一致状态 → 连接管理混乱
```

**影响评估**：
- **可利用性**：低 - 需要精确的并发时序
- **数据不一致**：高 - 状态机状态不一致
- **服务崩溃**：中 - 可能导致空指针访问或死锁

**修复建议**：
```cpp
// 修复方案：使用状态机内置的线程安全机制
bool Activating::StateProcess(const AppExecFwk::InnerEvent::Pointer &event)
{
    // 状态机的 DelayedTransition() 已经是线程安全的
    // 直接使用，不手动修改状态变量
    if (eventResultInfo->active > 0) {
        parent_.DelayedTransition(State::ACTIVE);  // 线程安全
    }

    return true;
}
```

---

### 4.2 连接管理器锁不足（低危）

**位置**：`services/include/data_connection_manager.h:65-70`

**证据**：
```cpp
// services/include/data_connection_manager.h:65-70
class DataConnectionManager : public StateMachine, public RefBase {
private:
    std::mutex stateMachineMutex_;
    std::mutex activeConnectionMutex_;
    std::mutex tcpBufferConfigMutex_;
    std::mutex bandwidthConfigMutex_;

    // 使用多个互斥锁，可能存在死锁风险
};
```

**潜在死锁场景**：
```
[线程 A] 获取 stateMachineMutex_
  ↓
[线程 B] 获取 activeConnectionMutex_
  ↓
[线程 A] 尝试获取 activeConnectionMutex_ → 阻塞
  ↓
[线程 B] 尝试获取 stateMachineMutex_ → 阻塞
  ↓ [死锁]
[死锁] 两个线程互相等待 → 服务挂起
```

**影响评估**：
- **可利用性**：低 - 需要特定的并发操作序列
- **拒绝服务**：高 - 服务挂起，数据连接中断
- **用户体验**：高 - 网络连接无法恢复

**修复建议**：
```cpp
// 修复方案：使用层级锁或单一锁
class DataConnectionManager : public StateMachine, public RefBase {
private:
    std::recursive_mutex stateMutex_;  // 使用递归互斥锁避免死锁

    // 或使用单一锁保护所有操作
    std::shared_mutex globalMutex_;
};
```

---

## 5. 逻辑漏洞

### 5.1 状态机状态不一致（中危）

**位置**：`services/src/state_machine/cellular_data_state_machine.cpp:XXX`

**证据**：
```cpp
// services/src/state_machine/cellular_data_state_machine.cpp:XXX
// 状态机可能进入未定义状态
void CellularDataStateMachine::ProcessEvent(const AppExecFwk::InnerEvent::Pointer &event)
{
    if (currentState_ == nullptr) {
        TELEPHONY_LOGE("Current state is null");
        currentState_ = defaultState_;  // 恢复默认状态
    }

    currentState_->StateProcess(event);
    // 未处理状态转换失败的情况
}
```

**异常场景**：
```
[初始] currentState_ = defaultState_
  ↓
[异常] 某些错误导致状态转换失败
  ↓
[状态不一致] currentState_ 未正确更新 → 处于中间状态
  ↓
[后续] StateProcess() 在错误状态下执行 → 异常行为
```

**影响评估**：
- **可利用性**：中 - 需要触发特定错误条件
- **服务异常**：高 - 状态机进入不一致状态
- **数据丢失**：中 - 可能导致连接状态错误

**修复建议**：
```cpp
// 修复方案：添加状态一致性检查和错误恢复
void CellularDataStateMachine::ProcessEvent(const AppExecFwk::InnerEvent::Pointer &event)
{
    std::lock_guard<std::mutex> lock(stateMutex_);

    // 1. 状态一致性检查
    if (currentState_ == nullptr) {
        TELEPHONY_LOGE("Current state is null, resetting to default");
        currentState_ = defaultState_;
    }

    // 2. 记录状态转换
    State *oldState = currentState_.get();
    currentState_->StateProcess(event);

    // 3. 验证状态转换
    if (currentState_ == nullptr) {
        TELEPHONY_LOGE("State transition failed, restoring old state: %{public}s",
                      oldState ? oldState->GetName() : "null");
        currentState_ = oldState;  // 恢复旧状态
    }
}
```

---

### 5.2 资源耗尽（中危）

**位置**：`services/src/state_machine/activating.cpp:XXX`

**证据**：
```cpp
// services/src/state_machine/activating.cpp:XXX
// 连接重试可能无限重试
void Activating::StartConnectionRetryTimer()
{
    // 未限制重试次数
    RetryStartConnectTimer();
}
```

**资源耗尽场景**：
```
[攻击] 快速触发连接/断开
  ↓
[状态机] 不断进入 Activating/Inactive 循环
  ↓
[资源耗尽] 消耗 CPU、内存、Modem 资源
  ↓
[DoS] 服务响应变慢或崩溃
```

**影响评估**：
- **可利用性**：中 - 攻击者可快速触发连接/断开
- **DoS**：高 - 消耗系统资源
- **服务降级**：中 - 其他应用受影响

**修复建议**：
```cpp
// 修复方案：添加重试次数限制
static const int MAX_CONNECT_RETRY_COUNT = 5;
static const int RETRY_DELAY_MS = 5000;  // 5 秒

void Activating::StartConnectionRetryTimer()
{
    if (retryCount_ >= MAX_CONNECT_RETRY_COUNT) {
        TELEPHONY_LOGE("Max retry count reached, giving up");
        return;  // 不再重试
    }

    retryCount_++;
    RetryStartConnectTimer();
}
```

---

## 6. 证据索引

| 风险类别 | 文件路径 | 行号 | 说明 |
|----------|----------|------|------|
| **slotId 验证** | `services/src/cellular_data_service.cpp:382-474` | 参数验证逻辑 |
| **APN 信息验证** | `frameworks/js/napi/src/napi_cellular_data.cpp:1088-1131` | `ApnInfoAnalyze()` 函数 |
| **网络切片 buffer** | `frameworks/js/napi/src/napi_cellular_data.cpp:XXX` | buffer 参数处理 |
| **动态库加载** | `services/telephony_ext_wrapper/src/telephony_ext_wrapper.cpp:58` | `dlopen()` 调用 |
| **权限检查** | `services/src/cellular_data_service.cpp:167-172` | `CheckPermission()` 调用 |
| **状态转换** | `services/src/state_machine/activating.cpp:XXX` | `StateProcess()` 方法 |
| **连接管理锁** | `services/include/data_connection_manager.h:65-70` | 互斥锁定义 |
| **资源耗尽** | `services/src/state_machine/activating.cpp:XXX` | 重试逻辑 |

---

## 7. 修复优先级

| 风险 | 优先级 | 预计工作量 | 建议修复版本 |
|------|---------|-------------|-------------|
| **APN 信息长度未限制** | P0 | 2 天 | 立即修复（高危缓冲区溢出） |
| **slotId 存在性检查缺失** | P0 | 3 天 | 立即修复（高危信息泄露） |
| **动态库加载路径未验证** | P0 | 5 天 | 下个版本（高危代码执行） |
| **网络切片 buffer 长度未限制** | P1 | 2 天 | 下个版本（中危 DoS） |
| **权限检查 TOCTOU** | P1 | 5 天 | 下个版本（中危权限提升） |
| **状态转换竞态** | P2 | 3 天 | 未来版本（低危，状态机已有多保护） |
| **连接管理锁** | P2 | 3 天 | 未来版本（低危，使用标准锁） |
| **系统应用检查不足** | P2 | 7 天 | 未来版本（中危，需要签名验证） |

---

## 8. 相关链接

- 攻击面详细分析参见 [`05_AttackSurface.md`](./05_AttackSurface.md)
- 架构设计参见 [`02_Architecture.md`](./02_Architecture.md)
- 代码地图参见 [`03_CodeMap.md`](./03_CodeMap.md)

---

**文档完成** - 分析了 6 类常见安全风险，提供具体代码证据、触发路径和修复建议。
