# 安全风险评审

> **目的**：基于代码证据，分析 pin_auth 模块的安全风险、攻击面和可被利用点
> **适用范围**：安全审计人员、架构师、模块维护者
> **关键结论**：模块实施了多层安全防护（权限、Token 隔离、IPC 验证），但仍有潜在风险点需要关注
> **相关文档**：[架构设计](03_Architecture.md) | [对外 API](04_Native_API.md) | [常见问题](09_FAQ.md)

---

## 威胁模型

```
外部输入（应用/用户）
    ↓
pin_auth 模块
    ├── 输入校验层（参数验证、权限检查）
    ├── Token 隔离层（调用者隔离）
    ├── IPC 通信层（描述符验证）
    └── HDI 接口层（TEE/安全芯片）
        ↓
    敏感操作（PIN 存储、PIN 验证）
```

**假设**：
- 攻击者可以获取系统签名（如设备已 root）
- 攻击者可以注入恶意代码到 useriam 或 pinauth 进程
- 攻击者可以监听 IPC 通信
- TEE/安全芯片被认为是可信的（南向厂商实现）

---

## 攻击面清单

### 1. IPC 攻击面

**接口**：
- PinAuthInterface（REGISTER_INPUTER、UNREGISTER_INPUTER）
- InputerGetData（ON_GET_DATA）
- InputerSetData（ON_SET_DATA）

**潜在攻击**：
- IPC 描述符伪造
- IPC 数据篡改
- 恶意输入器注册

**代码证据**：
- IPC Stub：`frameworks/ipc/src/*.cpp`
- 接口码定义：`frameworks/ipc/common_defines/*_ipc_interface_code.h`

### 2. 权限绕过攻击面

**所需权限**：
- `ohos.permission.ACCESS_PIN_AUTH`
- `ohos.permission.ACCESS_AUTH_RESPOOL`

**潜在攻击**：
- 权限提升
- Token ID 伪造
- 委托令牌滥用

**代码证据**：
- 权限检查：`services/sa/src/pin_auth_service.cpp:107-113`
- Token ID 获取：`services/sa/src/pin_auth_service.cpp:79-86`

### 3. 输入验证攻击面

**输入来源**：
- `authSubType` - 认证子类型
- `challenge` - 认证挑战
- `pinData` - PIN 数据

**潜在攻击**：
- 类型混淆攻击
- 缓冲区溢出
- 格式错误

**代码证据**：
- 参数检查宏：`common/utils/iam_check.h`
- Inputer 实现：由应用实现

### 4. 内存安全攻击面

**潜在攻击**：
- Use-After-Free
- Double-Free
- 空指针解引用
- 释放后使用

**代码证据**：
- 智能指针使用：`common/utils/iam_ptr.h`
- NoCopyable 基类：`common/utils/nocopyable.h`

### 5. 并发攻击面

**潜在攻击**：
- 竞态条件（Race Condition）
- 死锁
- 优先级反转

**代码证据**：
- Mutex 保护：`services/modules/inputters/src/pin_auth_manager.cpp:30`
- 回调线程：`services/modules/executors/src/pin_auth_executor_callback_hdi.cpp`

---

## 可被利用点（基于代码证据）

### 1. IPC 描述符验证不足（中等风险）

**证据**：
- 文件：`frameworks/ipc/src/pin_auth_stub.cpp:27-30`
- 代码：
  ```cpp
  if (PinAuthStub::GetDescriptor() != data.ReadInterfaceToken()) {
      IAM_LOGE("descriptor is not matched");
      return UserAuth::GENERAL_ERROR;
  }
  ```

**问题描述**：
- 仅验证接口描述符，未验证消息内容的完整性
- 可能被中间人攻击篡改数据

**利用路径**：
1. 攻击者监听 IPC 通道
2. 拦截并修改 `pinData` 内容
3. 描述符匹配，数据通过验证

**影响**：
- 中等：可能导致认证绕过或 PIN 泄露

**修复建议**：
```cpp
// 添加消息完整性校验（如 HMAC）
if (!VerifyMessageIntegrity(data)) {
    IAM_LOGE("message integrity check failed");
    return UserAuth::GENERAL_ERROR;
}

if (PinAuthStub::GetDescriptor() != data.ReadInterfaceToken()) {
    IAM_LOGE("descriptor is not matched");
    return UserAuth::GENERAL_ERROR;
}
```

---

### 2. Token ID 伪造风险（低风险）

**证据**：
- 文件：`services/sa/src/pin_auth_service.cpp:79-86`
- 代码：
  ```cpp
  inline uint32_t PinAuthService::GetTokenId()
  {
      uint32_t tokenId = this->GetFirstTokenID();
      if (tokenId == 0) {
          tokenId = this->GetCallingTokenID();
      }
      return tokenId;
  }
  ```

**问题描述**：
- 优先使用 `GetFirstTokenID()`（委托令牌）
- 如果委托令牌机制有漏洞，可能导致权限提升
- 未验证 Token ID 的合法性（如是否在有效范围内）

**利用路径**：
1. 攻击者创建恶意应用
2. 通过某种方式伪造或获取委托 Token
3. 使用伪造 Token 注册 Inputer
4. 窃取其他应用的 PIN 数据

**影响**：
- 低：需要先突破委托令牌机制
- 但一旦突破，可跨应用窃取 PIN

**修复建议**：
```cpp
inline uint32_t PinAuthService::GetTokenId()
{
    uint32_t tokenId = this->GetFirstTokenID();
    if (tokenId == 0) {
        tokenId = this->GetCallingTokenID();
    }
    
    // 添加 Token ID 合法性验证
    if (tokenId == 0 || !IsTokenIdValid(tokenId)) {
        IAM_LOGE("invalid tokenId: %{public}u", tokenId);
        return 0;  // 或抛出异常
    }
    
    return tokenId;
}
```

---

### 3. Inputer 重复注册检查不完整（低风险）

**证据**：
- 文件：`services/modules/inputters/src/pin_auth_manager.cpp:31-34`
- 代码：
  ```cpp
  auto it = pinAuthInputerMap_.find(tokenId);
  if (it != pinAuthInputerMap_.end()) {
      IAM_LOGE("inputer already registered for token %{public}u", tokenId);
      return false;
  }
  ```

**问题描述**：
- 仅检查 Token ID 是否存在
- 未检查旧 Inputer 的有效性（如 Death Recipient 是否仍存活）
- 可能导致僵尸 Inputer 残留

**利用路径**：
1. 应用 A 注册 Inputer
2. 应用 A 崩溃，但 Death Recipient 未及时触发
3. 应用 A 重新启动，尝试注册 Inputer
4. 检测到已注册，返回失败
5. 僵尸 Inputer 仍然接收回调

**影响**：
- 低：可能导致 PIN 数据发送到错误的 Inputer

**修复建议**：
```cpp
// 在注册前清理僵尸 Inputer
auto it = pinAuthInputerMap_.find(tokenId);
if (it != pinAuthInputerMap_.end()) {
    IAM_LOGW("cleaning up stale inputer for token %{public}u", tokenId);
    pinAuthInputerMap_.erase(it);
    // 添加 Death Recipient 清理逻辑
}

// 然后注册新 Inputer
pinAuthInputerMap_[tokenId] = inputer;
```

---

### 4. scheduleId 会话隔离不足（中等风险）

**证据**：
- 文件：`services/modules/executors/inc/pin_auth_executor_callback_hdi.h:32-45`
- 代码：
  ```cpp
  class IExecutorCallbackHdi : public IExecutorCallback {
  public:
      IExecutorCallbackHdi(uint32_t tokenId, uint64_t scheduleId)
          : tokenId_(tokenId), scheduleId_(scheduleId) {}
      
      int32_t OnGetData(uint64_t scheduleId, const std::vector<uint8_t> &authToken) override;
      
  private:
      uint32_t tokenId_;
      uint64_t scheduleId_;
  };
  ```

**问题描述**：
- `scheduleId` 仅用于 HDI 回调匹配
- 未在 `IInputerDataImpl` 中验证会话完整性
- 可能导致会话混淆攻击

**利用路径**：
1. 应用 A 开始认证，获得 scheduleId = 100
2. 应用 B 同时开始认证，获得 scheduleId = 200
3. 攻击者通过某种方式交换或伪造 scheduleId
4. 应用 A 的 PIN 数据发送到应用 B 的会话

**影响**：
- 中等：可能导致跨会话数据泄露

**修复建议**：
```cpp
// 在 IInputerDataImpl 中添加会话验证
class IInputerDataImpl : public IInputerData {
public:
    IInputerDataImpl(uint32_t tokenId, uint64_t scheduleId)
        : tokenId_(tokenId), scheduleId_(scheduleId) {
        // 绑定 scheduleId 和 tokenId
    }
    
    void OnSetData(int32_t authSubType, std::vector<uint8_t> data) override {
        std::lock_guard<std::mutex> lock(mutex_);
        
        // 验证 scheduleId 匹配
        if (scheduleId_ != expectedScheduleId_) {
            IAM_LOGE("scheduleId mismatch: %{public}lu vs %{public}lu", 
                      scheduleId_, expectedScheduleId_);
            return;
        }
        
        // 原有逻辑...
    }
    
private:
    uint32_t tokenId_;
    uint64_t scheduleId_;
    uint64_t expectedScheduleId_;
    std::mutex mutex_;
};
```

---

### 5. Scrypt 参数硬编码（低风险）

**证据**：
- 文件：`frameworks/scrypt/src/scrypt.cpp`
- 代码：（TODO：需要查看 scrypt 实现细节）

**问题描述**：
- Scrypt 参数（N、r、p、dkLen）可能硬编码
- 如果参数不够强，容易被暴力破解

**利用路径**：
1. 攻击者获取加密后的 PIN 数据
2. 使用标准硬件暴力破解
3. 如果 scrypt 参数较弱，可以快速破解

**影响**：
- 低：需要先获取加密数据
- 取决于 scrypt 参数强度

**修复建议**：
```cpp
// 将 scrypt 参数作为可配置项
// 从系统参数或安全策略读取
struct ScryptParams {
    uint64_t N;
    uint32_t r;
    uint32_t p;
    uint32_t dkLen;
};

ScryptParams params = GetSecureScryptParamsFromSystem();
// 使用参数进行 scrypt 运算
```

---

## 信任边界

### 1. 用户空间 → System Ability

**边界**：IPC Binder

**信任假设**：
- SA 信任调用者的 Token ID
- SA 信任权限检查结果

**安全措施**：
- Token 隔离（每个 Token ID 对应独立 Inputer）
- 权限检查（ACCESS_PIN_AUTH）
- IPC 描述符验证

### 2. System Ability → TEE/安全芯片

**边界**：HDI 接口

**信任假设**：
- SA 信任 HDI 驱动
- HDI 驱动信任 TEE

**安全措施**：
- 南向厂商在 TEE 中实现
- PIN 原文不跨 HDI 传输
- Scrypt 单向哈希处理

---

## 检查范围与局限性

### 已检查范围

✅ **已检查**：
- IPC 接口和描述符验证
- 权限检查机制
- Token ID 隔离
- Inputer 注册管理
- HDI 回调处理
- 内存安全（智能指针、NoCopyable）
- 并发保护（Mutex）

❌ **未检查**（超出本模块范围）：
- TEE/安全芯片内部实现（南向厂商）
- SELinux 策略配置（位于其他仓库）
- user_auth_framework 的 N-API 层
- 系统签名验证机制

### 局限性

1. **南向厂商依赖**：
   - TEE/安全芯片的安全由厂商实现
   - 本文档假设 TEE 是可信的
   - 如果厂商实现有漏洞，无法通过框架层防护

2. **OpenHarmony 系统信任**：
   - 依赖 OpenHarmony 的安全框架（access_token、hilog 等）
   - 如果框架层有漏洞，可能影响 pin_auth

3. **动态加载模式**：
   - 动态加载引入了额外的攻击面
   - 未详细分析动态加载的安全机制

---

## 安全建议汇总

### 高优先级

1. **添加 IPC 消息完整性校验**
   - 使用 HMAC 或类似机制
   - 防止中间人攻击

2. **增强 Token ID 验证**
   - 验证 Token ID 范围
   - 添加白名单机制

### 中优先级

3. **改进 Inputer 注册检查**
   - 清理僵尸 Inputer
   - 添加健康检查机制

4. **增强 scheduleId 会话隔离**
   - 在 IInputerDataImpl 中验证会话
   - 防止会话混淆

### 低优先级

5. **可配置 scrypt 参数**
   - 从系统参数读取
   - 支持动态调整安全级别

---

## 代码证据索引

| 风险点 | 证据文件 | 行号 |
|---------|----------|------|
| IPC 描述符验证不足 | `frameworks/ipc/src/pin_auth_stub.cpp` | 27-30 |
| Token ID 伪造风险 | `services/sa/src/pin_auth_service.cpp` | 79-86 |
| Inputer 重复注册检查不完整 | `services/modules/inputters/src/pin_auth_manager.cpp` | 31-34 |
| scheduleId 会话隔离不足 | `services/modules/executors/inc/pin_auth_executor_callback_hdi.h` | 32-45 |
| Scrypt 参数硬编码 | `frameworks/scrypt/src/scrypt.cpp` | (TODO) |

---

## 下一步

- 常见安全问题 → [常见问题](09_FAQ.md)
