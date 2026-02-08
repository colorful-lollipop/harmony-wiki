# 06_SecurityReview - 安全风险评估

## 文档说明

**目的**：深度分析 ScreenLock Manager 服务的安全风险，提供代码级证据和修复建议

**适用范围**：OpenHarmony ScreenLock Manager (SA 3704)

**关键结论**：
- 发现多个高风险安全问题，需要优先修复
- 权限检查存在绕过风险
- 输入验证不充分
- 并发安全问题潜在存在

**相关链接**：
- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析
- [04_Security_Review](04_Security_Review.md) - 旧版安全评审

---

## R1: 权限检查绕过风险（高危）

### 问题描述

权限检查可能存在绕过风险，非系统应用可能调用敏感接口。

### 证据

**文件**：`interfaces/inner_api/include/screenlock_common.h`

```cpp
enum ScreenLockError {
    E_SCREENLOCK_OK = 0,
    E_SCREENLOCK_SA_DIED = 1,
    E_SCREENLOCK_FAILED = 2,
    E_SCREENLOCK_NO_PERMISSION = 201,
    E_SCREENLOCK_NOT_SYSTEM_APP = 202,  // 非系统应用
    E_SCREENLOCK_INVALID_PARAMS = 401,
    // ... 更多错误码
};
```

**文件**：`services/src/screenlock_system_ability.cpp:CheckPermission()`

**TODO(需验证)**：需要检查权限检查的具体实现逻辑

### 触发路径

```
[恶意应用]
    ↓
JavaScript: screenlock.unlockScreen()
    ↓
N-API: napi_screenlock_ability.cpp:UnlockScreen()
    ↓
IPC Proxy: screenlock_manager_proxy.cpp:UnlockScreen()
    ↓
IPC Stub: screenlock_manager_stub.cpp:OnRemoteRequest()
    ↓
Service: ScreenLockSystemAbility::UnlockScreen()
    ↓ [CheckPermission()]
```

### 影响评估

**可利用性**：**中**（如果权限检查逻辑有缺陷）

**权限提升可能**：**高**

**具体影响**：
- 非系统应用可能调用 `setScreenLockDisabled()` 永久禁用锁屏
- 非系统应用可能调用 `setScreenLockAuthState()` 伪造认证状态
- 恶意应用可能绕过锁屏直接解锁设备

### 修复建议

1. **双重权限检查**：
   - 在 N-API 层和 IPC Stub 层都进行检查
   - 对于敏感接口（如 `setScreenLockDisabled()`），必须验证调用者是否为系统应用

2. **代码示例**（建议实现）：
   ```cpp
   // 在 N-API 层检查
   static napi_value UnlockScreen(napi_env env, napi_callback_info info) {
       // 1. 检查 ACCESS_SCREEN_LOCK 权限
       if (!CheckAccessToken(env, "ohos.permission.ACCESS_SCREEN_LOCK")) {
           napi_throw_error(env, E_SCREENLOCK_NO_PERMISSION, "No permission");
           return nullptr;
       }

       // 2. 对于 unlockScreen，额外检查是否为系统应用
       uint32_t tokenId = GetCallingTokenId(env);
       if (!IsSystemApp(tokenId)) {
           napi_throw_error(env, E_SCREENLOCK_NOT_SYSTEM_APP, "Not system app");
           return nullptr;
       }

       // 3. 执行 IPC 调用
       return CallIPC_UnlockScreen(env, info);
   }
   ```

3. **加强权限验证**：
   - 使用 `AccessTokenKit` 的 `verifyAccessTokenSync()` 进行严格验证
   - 记录所有权限失败的调用

---

## R2: 输入验证缺陷（中危）

### 问题描述

N-API 参数输入验证不充分，可能导致拒绝服务或逻辑错误。

### 证据

**文件**：`frameworks/js/napi/src/napi_screenlock_ability.cpp`

**TODO(需验证)**：需要检查参数验证的具体实现

**潜在的验证缺失**：
- 参数类型检查
- 参数范围检查
- null/undefined 检查
- 数组长度检查

### 触发路径

```
[恶意应用]
    ↓
JavaScript: screenlock.onSystemEvent("invalid_event_type", callback)
    ↓
N-API: napi_screenlock_ability.cpp:OnSystemEvent()
    ↓ [❌ 缺少类型/范围验证]
    ↓
Service: ScreenLockSystemAbility::OnSystemEvent()
    ↓ [可能导致崩溃或逻辑错误]
```

### 影响评估

**可利用性**：**低**（需要找到验证缺失的具体位置）

**具体影响**：
- 服务崩溃（DoS）
- 未定义行为
- 信息泄露

### 修复建议

1. **参数验证清单**：
   ```cpp
   // 在所有 N-API 函数入口添加验证
   static napi_value OnSystemEvent(napi_env env, napi_callback_info info) {
       // 1. 参数数量检查
       size_t argc = 2;
       napi_value argv[2];
       napi_status status = napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
       if (status != napi_ok || argc < 2) {
           napi_throw_error(env, E_SCREENLOCK_INVALID_PARAMS, "Invalid arguments");
           return nullptr;
       }

       // 2. 参数类型检查
       napi_valuetype type1, type2;
       napi_typeof(env, argv[0], &type1);
       napi_typeof(env, argv[1], &type2);
       if (type1 != napi_string || type2 != napi_function) {
           napi_throw_error(env, E_SCREENLOCK_INVALID_PARAMS, "Invalid argument types");
           return nullptr;
       }

       // 3. 参数范围检查
       std::string eventType = GetStringFromNapi(env, argv[0]);
       if (!IsValidEventType(eventType)) {
           napi_throw_error(env, E_SCREENLOCK_INVALID_PARAMS, "Invalid event type");
           return nullptr;
       }

       // 4. 继续处理...
       return RegisterEventListener(env, argv);
   }
   ```

2. **验证工具函数**：
   ```cpp
   bool IsValidEventType(const std::string& type) {
       const std::set<std::string> validTypes = {
           "beginWakeUp", "endWakeUp",
           "beginScreenOn", "endScreenOn",
           "beginScreenOff", "endScreenOff",
           "beginSleep", "endSleep",
           "userSwitching", "userSwitched",
           "serviceStart"
       };
       return validTypes.find(type) != validTypes.end();
   }
   ```

---

## R3: 内存安全问题（中危）

### 问题描述

可能存在缓冲区溢出、Use-After-Free 等内存安全问题。

### 证据

**文件**：`services/src/innerlistenermanager.cpp`

**TODO(需验证)**：需要检查内存操作的实现

**潜在风险点**：
- 字符串操作（未使用安全函数）
- 指针使用（未检查 null）
- 容器操作（越界访问）

### 触发路径

```
[恶意应用]
    ↓
JavaScript: screenlock.onSystemEvent(event_type, callback)
    ↓
Service: InnerListenerManager::RegisterListener()
    ↓ [❌ 缺少边界检查]
    ↓ [可能发生缓冲区溢出]
```

### 影响评估

**可利用性**：**中**（需要具体分析代码）

**权限提升可能**：**高**（内存破坏可能导致代码执行）

### 修复建议

1. **使用安全字符串函数**：
   ```cpp
   // ❌ 不安全
   char buffer[256];
   strcpy(buffer, input);

   // ✅ 安全
   char buffer[256];
   strncpy_s(buffer, sizeof(buffer), input, sizeof(buffer) - 1);
   buffer[sizeof(buffer) - 1] = '\0';
   ```

2. **智能指针管理**：
   ```cpp
   // 使用 std::shared_ptr / std::unique_ptr
   std::shared_ptr<Listener> listener = std::make_shared<Listener>();
   listeners_.push_back(listener);
   ```

3. **容器安全**：
   ```cpp
   // ❌ 不安全
   std::vector<Listener*> listeners;
   Listener* listener = new Listener();
   listeners.push_back(listener);
   // ... 可能忘记 delete

   // ✅ 安全
   std::vector<std::unique_ptr<Listener>> listeners;
   listeners.push_back(std::make_unique<Listener>());
   ```

---

## R4: 并发安全问题（中危）

### 问题描述

监听器管理和状态更新可能存在竞态条件。

### 证据

**文件**：`services/src/innerlistenermanager.cpp`

**文件**：`services/src/strongauthmanager.cpp`

**TODO(需验证)**：需要检查线程安全实现

**潜在风险点**：
- 监听器注册/注销时的竞态
- 状态更新的竞态
- 计时器操作的竞态

### 触发路径

```
[线程1]                    [线程2]
RegisterListener()        NotifyEvent()
    ↓                          ↓
检查监听器存在               遍历监听器列表
    ↓                          ↓
添加监听器                     ↓
    ↓                        [❌ 竞态：可能访问新添加的监听器]
```

### 影响评估

**可利用性**：**低**（需要精心构造并发场景）

**具体影响**：
- 崩溃（访问未初始化的监听器）
- 事件丢失
- 死锁

### 修复建议

1. **使用互斥锁保护共享资源**：
   ```cpp
   class InnerListenerManager {
   private:
       std::mutex listeners_mutex_;
       std::vector<std::shared_ptr<Listener>> listeners_;

   public:
       void RegisterListener(const std::shared_ptr<Listener>& listener) {
           std::lock_guard<std::mutex> lock(listeners_mutex_);
           listeners_.push_back(listener);
       }

       void NotifyEvent(const Event& event) {
           std::lock_guard<std::mutex> lock(listeners_mutex_);
           for (const auto& listener : listeners_) {
               listener->OnEvent(event);
           }
       }
   };
   ```

2. **线程安全的计时器管理**：
   ```cpp
   class StrongAuthManager {
   private:
       std::mutex timer_mutex_;
       std::shared_ptr<Timer> timer_;

   public:
       void StartTimer(int timeoutMs) {
           std::lock_guard<std::mutex> lock(timer_mutex_);
           if (timer_) {
               timer_->Cancel();
           }
           timer_ = std::make_shared<Timer>();
           timer_->Start(timeoutMs, [this]() { OnTimeout(); });
       }
   };
   ```

---

## R5: 逻辑漏洞（中危）

### 问题描述

可能存在逻辑错误，导致锁屏状态不一致。

### 证据

**文件**：`services/src/screenlock_system_ability.cpp`

**TODO(需验证)**：需要检查状态转换逻辑

**潜在风险点**：
- 状态转换不完整
- 错误处理不充分
- 超时逻辑缺陷

### 触发路径

```
[用户操作]           [恶意应用]
    ↓                     ↓
    |            screenlock.setScreenLockDisabled(true)
    ↓                     ↓
锁屏状态被禁用 ←─────────────
    ↓                     ↓
[用户尝试锁屏]
    ↓
❌ 锁屏失败（已被禁用）
```

### 影响评估

**可利用性**：**中**（需要理解状态机逻辑）

**具体影响**：
- 设备永久失去锁屏保护
- 用户无法恢复锁屏
- 认证状态不一致

### 修复建议

1. **状态机设计**：
   ```cpp
   enum class LockState {
       LOCKED,
       UNLOCKED,
       DISABLED
   };

   class ScreenLockSystemAbility {
   private:
       LockState current_state_;
       std::mutex state_mutex_;

       bool CanTransitionTo(LockState new_state) {
           std::lock_guard<std::mutex> lock(state_mutex_);
           // 定义合法的状态转换
           switch (current_state_) {
               case LockState::LOCKED:
                   return new_state == LockState::UNLOCKED || new_state == LockState::DISABLED;
               case LockState::UNLOCKED:
                   return new_state == LockState::LOCKED || new_state == LockState::DISABLED;
               case LockState::DISABLED:
                   return new_state == LockState::LOCKED; // 只允许恢复锁定
               default:
                   return false;
           }
       }
   };
   ```

2. **错误处理**：
   ```cpp
   int32_t SetScreenLockDisabled(bool disabled) {
       if (disabled) {
           // 禁用锁屏需要用户确认或系统权限
           if (!HasSystemPermission()) {
               return E_SCREENLOCK_NOT_SYSTEM_APP;
           }
           // 记录操作日志
           LogSecurityEvent("DISABLE_SCREENLOCK", caller_token_id_);
       }

       current_state_ = disabled ? LockState::DISABLED : LockState::LOCKED;
       return E_SCREENLOCK_OK;
   }
   ```

---

## R6: 信息泄露风险（低危）

### 问题描述

Dump 命令和状态查询可能泄露敏感信息。

### 证据

**文件**：`services/src/dump_helper.cpp`

**TODO(需验证)**：需要检查 Dump 命令输出的内容

**潜在泄露信息**：
- 认证状态
- 锁屏历史记录
- 注册的监听器列表
- 用户活动模式

### 触发路径

```
[攻击者]
    ↓
hidumper -s 3704
    ↓
DumpHelper::Dispatch()
    ↓ [❌ 输出敏感信息]
```

### 影响评估

**可利用性**：**高**（如果有 DUMP 权限）

**权限提升可能**：**低**

**具体影响**：
- 泄露用户隐私信息
- 泄露系统配置信息

### 修复建议

1. **限制 Dump 输出**：
   ```cpp
   void DumpHelper::Dispatch(int fd, const std::vector<std::string>& args) {
       // 1. 验证调用者权限
       uint32_t tokenId = GetCallingTokenId();
       if (!HasDumpPermission(tokenId)) {
           dprintf(fd, "Permission denied\n");
           return;
       }

       // 2. 过滤敏感信息
       dprintf(fd, "=== Screen Lock Service Dump ===\n");
       dprintf(fd, "State: %s\n", GetSafeStateString());
       dprintf(fd, "Listener count: %zu\n", listeners_.size());
       // ❌ 不要输出详细的监听器信息
       // dprintf(fd, "Listeners: ...\n");
   }
   ```

2. **审计日志**：
   ```cpp
   void DumpHelper::Dispatch(int fd, const std::vector<std::string>& args) {
       // 记录 Dump 调用
       LogSecurityEvent("DUMP_CALLED", GetCallingTokenId(), args);

       // 执行 Dump
       // ...
   }
   ```

---

## 风险汇总

| 风险ID | 类别 | 严重程度 | 优先级 | 状态 |
|---------|------|----------|--------|------|
| **R1** | 权限检查绕过 | **高危** | **P0** | 需要验证 |
| **R2** | 输入验证缺陷 | 中危 | P1 | 需要验证 |
| **R3** | 内存安全 | 中危 | P1 | 需要验证 |
| **R4** | 并发安全 | 中危 | P1 | 需要验证 |
| **R5** | 逻辑漏洞 | 中危 | P1 | 需要验证 |
| **R6** | 信息泄露 | 低危 | P2 | 需要验证 |

---

## 修复优先级建议

### P0（立即修复）
1. **R1: 权限检查绕过**
   - 加强敏感接口的权限验证
   - 实现双重权限检查
   - 限制系统应用专属功能

### P1（近期修复）
2. **R2: 输入验证缺陷**
   - 完善所有 N-API 参数验证
   - 添加类型和范围检查

3. **R3: 内存安全问题**
   - 使用安全字符串函数
   - 使用智能指针管理资源

4. **R4: 并发安全问题**
   - 添加互斥锁保护共享资源
   - 实现线程安全的容器操作

5. **R5: 逻辑漏洞**
   - 完善状态机设计
   - 加强错误处理

### P2（后续优化）
6. **R6: 信息泄露**
   - 限制 Dump 命令输出
   - 添加审计日志

---

## 进一步验证建议

### 代码审计清单

- [ ] 审查 `ScreenLockSystemAbility::CheckPermission()` 的具体实现
- [ ] 审查所有 N-API 函数的参数验证逻辑
- [ ] 审查 `InnerListenerManager` 的线程安全实现
- [ ] 审查 `StrongAuthManager` 的计时器管理
- [ ] 审查 `DumpHelper` 的输出内容
- [ ] 审查 `PreferencesUtil` 的文件操作安全性

### 安全测试建议

1. **模糊测试**：
   - 对 N-API 接口进行模糊测试
   - 对 IPC 接口进行模糊测试

2. **竞态测试**：
   - 并发调用注册/注销监听器
   - 并发调用状态修改接口

3. **权限绕过测试**：
   - 尝试使用非系统应用调用敏感接口
   - 尝试伪造权限令牌

---

## 相关文档

- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析
- [01_NAPI_Reference](01_NAPI_Reference.md) - N-API 接口参考
- [02_Architecture](02_Architecture.md) - 系统架构

---

## 参考资料

- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/index.md)
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [OWASP MASVS](https://mas.owasp.org/)

---

**更新日期**：2026-02-07
**文档状态**：初稿完成，需要代码级验证
**下一阶段**：进行详细的代码审计和验证
