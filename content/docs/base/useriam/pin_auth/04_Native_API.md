# 对外 Native API 文档

> **目的**：提供完整的对外 Native API 参考（C++ 接口，非 N-API）
> **适用范围**：应用开发者、系统集成者
> **重要说明**：pin_auth 模块不提供 JavaScript N-API，Native C++ 接口仅供内部子系统使用
> **相关文档**：[概览](index.md) | [架构设计](03_Architecture.md) | [内部 API](05_Inner_API.md)

---

## 重要说明

### N-API vs Native API

**pin_auth 模块不提供 JavaScript N-API 绑定**。

| 模块 | 提供的 API | 调用者 |
|------|------------|--------|
| pin_auth | Native C++ 接口 | 内部子系统（Settings、锁屏） |
| user_auth_framework | JavaScript N-API | 应用开发者（ArkTS/ETS） |

**JavaScript API 路径**：

```
应用（ArkTS/ETS）
    ↓ 调用
user_auth_framework (N-API 层)
    ↓ IPC
pin_auth (Native C++ 服务)
    ↓ HDI
南向厂商实现（TEE/安全芯片）
```

### 谁应该使用此 API

**目标用户**：
- ✅ 系统级应用开发者（Settings、锁屏）
- ✅ OpenHarmony 子系统集成者
- ❌ 普通应用开发者（应使用 user_auth_framework 的 JS API）

**权限要求**：
- `ohos.permission.ACCESS_PIN_AUTH` - 注册/注销 Inputer 必需
- 系统签名

---

## API 清单表

### PinAuthRegister 接口

| API 名称 | 命名空间 | 同步/异步 | 权限要求 | C++ 入口 | 绑定位置 |
|---------|---------|----------|----------|----------|---------|
| `GetInstance()` | `OHOS::UserIam::PinAuth` | 同步 | 无 | `PinAuthRegisterImpl::Instance()` | `frameworks/client/src/pinauth_register_impl.cpp` |
| `RegisterInputer()` | `OHOS::UserIam::PinAuth` | 同步 | ACCESS_PIN_AUTH | `PinAuthRegisterImpl::RegisterInputer()` | `frameworks/client/src/pinauth_register_impl.cpp` |
| `UnRegisterInputer()` | `OHOS::UserIam::PinAuth` | 同步 | ACCESS_PIN_AUTH | `PinAuthRegisterImpl::UnRegisterInputer()` | `frameworks/client/src/pinauth_register_impl.cpp` |

### IInputer 接口

| API 名称 | 命名空间 | 同步/异步 | 权限要求 | C++ 入口 | 绑定位置 |
|---------|---------|----------|----------|----------|---------|
| `OnGetData()` | `OHOS::UserIam::PinAuth` | 回调（异步） | 无 | 由应用实现 | - |

### IInputerData 接口

| API 名称 | 命名空间 | 同步/异步 | 权限要求 | C++ 入口 | 绑定位置 |
|---------|---------|----------|----------|----------|---------|
| `OnSetData()` | `OHOS::UserIam::PinAuth` | 回调（异步） | 无 | 由应用实现 | - |

---

## PinAuthRegister 接口详细说明

### 头文件位置

**文件**：`interfaces/inner_api/pinauth_register.h`

### 类定义

```cpp
namespace OHOS {
namespace UserIam {
namespace PinAuth {
class PinAuthRegister {
public:
    /**
     * @brief 获取 PinAuthRegister 单例
     * @return PinAuthRegister 实例引用
     */
    static PinAuthRegister &GetInstance();

    /**
     * @brief 析构函数
     */
    virtual ~PinAuthRegister() = default;

    /**
     * @brief 注册 Inputer（用于获取 PIN 数据）
     * @param inputer 输入器回调对象
     * @return 是否注册成功
     */
    virtual bool RegisterInputer(std::shared_ptr<IInputer> inputer) = 0;

    /**
     * @brief 注销 Inputer
     */
    virtual void UnRegisterInputer() = 0;
};
}}}
```

### API 1：GetInstance()

**签名**：
```cpp
static PinAuthRegister &GetInstance();
```

**说明**：
- 获取 `PinAuthRegister` 单例
- 线程安全

**返回值**：
- `PinAuthRegister` 实例引用

**前置条件**：
- 无

**错误处理**：
- 无（总是返回有效实例）

**代码位置**：
- 声明：`interfaces/inner_api/pinauth_register.h:39`
- 实现：`frameworks/client/src/pinauth_register_impl.cpp:24-30`

---

### API 2：RegisterInputer()

**签名**：
```cpp
virtual bool RegisterInputer(std::shared_ptr<IInputer> inputer);
```

**参数**：
- `inputer`：Inputer 回调对象指针，不能为 nullptr

**返回值**：
- `true`：注册成功
- `false`：注册失败（参数无效、已注册、权限不足等）

**前置条件**：
- `inputer` 不能为 nullptr
- 调用者必须持有 `ohos.permission.ACCESS_PIN_AUTH` 权限
- 同一 Token ID 只能注册一个 Inputer

**参数校验**：

| 检查项 | 位置 | 处理 |
|---------|------|------|
| nullptr 检查 | `frameworks/ipc/src/pin_auth_stub.cpp:43-45` | 返回 false |
| 权限检查 | `services/sa/src/pin_auth_service.cpp:115-128` | 返回 false |
| 重复注册检查 | `services/modules/inputters/src/pin_auth_manager.cpp:31-34` | 返回 false |

**错误码**：
- 无标准错误码，通过返回值表示成功/失败
- 失败原因通过日志输出（IAM_LOGE）

**调用示例**：

```cpp
#include "pinauth_register.h"
#include "i_inputer.h"

using namespace OHOS::UserIam::PinAuth;

class MyInputer : public IInputer {
public:
    void OnGetData(int32_t authSubType, 
                 std::vector<uint8_t> challenge, 
                 std::shared_ptr<IInputerData> inputerData) override {
        // 显示 PIN 输入对话框
        // 获取用户输入后调用 inputerData->OnSetData()
    }
};

// 注册 Inputer
auto &register = PinAuthRegister::GetInstance();
std::shared_ptr<IInputer> inputer = std::make_shared<MyInputer>();
bool result = register.RegisterInputer(inputer);
if (!result) {
    // 处理注册失败
}
```

**代码位置**：
- 声明：`interfaces/inner_api/pinauth_register.h:52`
- 实现入口：`frameworks/client/src/pinauth_register_impl.cpp:56-80`
- IPC Stub：`frameworks/ipc/src/pin_auth_stub.cpp:37-57`
- SA 处理：`services/sa/src/pin_auth_service.cpp:115-128`

---

### API 3：UnRegisterInputer()

**签名**：
```cpp
virtual void UnRegisterInputer();
```

**参数**：
- 无

**返回值**：
- 无

**前置条件**：
- 必须先成功注册 Inputer

**行为**：
- 注销当前 Token ID 注册的 Inputer
- 清理 Death Recipient
- 不影响其他 Token ID 的 Inputer

**调用示例**：

```cpp
auto &register = PinAuthRegister::GetInstance();
register.UnRegisterInputer();
```

**代码位置**：
- 声明：`interfaces/inner_api/pinauth_register.h:57`
- 实现入口：`frameworks/client/src/pinauth_register_impl.cpp:82-100`
- IPC Stub：`frameworks/ipc/src/pin_auth_stub.cpp:59-72`
- SA 处理：`services/sa/src/pin_auth_service.cpp:130-139`

---

## IInputer 接口详细说明

### 头文件位置

**文件**：`interfaces/inner_api/i_inputer.h`

### 类定义

```cpp
namespace OHOS {
namespace UserIam {
namespace PinAuth {
class IInputer {
public:
    /**
     * @brief 获取 PIN 数据
     * @param authSubType PIN 认证子类型
     * @param challenge PIN 认证挑战
     * @param inputerData 输入数据回调对象
     */
    virtual void OnGetData(
        int32_t authSubType, 
        std::vector<uint8_t> challenge, 
        std::shared_ptr<IInputerData> inputerData) = 0;
};
}}}
```

### API：OnGetData()

**签名**：
```cpp
virtual void OnGetData(int32_t authSubType, 
                   std::vector<uint8_t> challenge, 
                   std::shared_ptr<IInputerData> inputerData);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `authSubType` | `int32_t` | PIN 认证子类型（普通 PIN、私有 PIN、恢复密钥等） |
| `challenge` | `std::vector<uint8_t>` | 认证挑战（防重放） |
| `inputerData` | `std::shared_ptr<IInputerData>` | 数据传输回调对象，用于返回 PIN 数据 |

**返回值**：
- 无

**同步/异步**：
- 这是一个回调函数，在 SA 线程中同步调用
- 应用应快速返回，避免阻塞 SA 线程

**authSubType 值**：

| 常量 | 值 | 说明 |
|------|------|------|
| SCHEDULE_PIN | 0 | 普通 PIN |
| SCHEDULE_PIN_MIX | 1 | 混合 PIN（含数字+字母） |
| ... | ... | 其他类型（参考 user_auth_framework 定义） |

**调用时机**：
- User Auth Framework 调用 `BeginAuthentication()` 后
- SA 通过 HDI 接收到 `OnGetData()` 回调
- SA 查找对应 Token ID 的 Inputer
- SA 调用此方法请求 PIN 数据

**调用示例**：

```cpp
#include "i_inputer.h"
#include "i_inputer_data.h"

using namespace OHOS::UserIam::PinAuth;

class MyInputer : public IInputer {
public:
    void OnGetData(int32_t authSubType, 
                 std::vector<uint8_t> challenge, 
                 std::shared_ptr<IInputerData> inputerData) override {
        
        // 1. 显示 PIN 输入对话框
        std::string pin = ShowPinInputDialog(authSubType);
        
        // 2. 转换为字节数组
        std::vector<uint8_t> pinData(pin.begin(), pin.end());
        
        // 3. 返回 PIN 数据
        inputerData->OnSetData(authSubType, pinData);
    }

private:
    std::string ShowPinInputDialog(int32_t authSubType) {
        // 实现对话框逻辑
        return "1234";  // 示例
    }
};
```

**代码位置**：
- 声明：`interfaces/inner_api/i_inputer.h:42-43`
- 由应用实现
- SA 调用位置：`services/modules/executors/src/pin_auth_executor_callback_hdi.cpp:44-66`

---

## IInputerData 接口详细说明

### 头文件位置

**文件**：`interfaces/inner_api/i_inputer_data.h`

### 类定义

```cpp
namespace OHOS {
namespace UserIam {
namespace PinAuth {
class IInputerData : public NoCopyable {
public:
    IInputerData() = default;
    ~IInputerData() override = default;

    /**
     * @brief 传输 PIN 数据到 pin_auth SA
     * @param authSubType PIN 认证子类型
     * @param data PIN 数据（已加密处理）
     */
    virtual void OnSetData(int32_t authSubType, std::vector<uint8_t> data) = 0;
};
}}}
```

### API：OnSetData()

**签名**：
```cpp
virtual void OnSetData(int32_t authSubType, std::vector<uint8_t> data);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `authSubType` | `int32_t` | PIN 认证子类型 |
| `data` | `std::vector<uint8_t>` | PIN 数据（单向哈希处理后） |

**返回值**：
- 无

**调用时机**：
- 应用在 `OnGetData()` 回调中显示 PIN 输入对话框
- 用户输入 PIN 后，应用调用此方法返回 PIN 数据
- SA 接收数据后传输到 HDI 进行验证

**安全说明**：
- `data` 参数已经过 scrypt 单向哈希处理
- 原文 PIN 不跨 IPC 传输
- TEE 内部进行 PIN 比较

**调用示例**：

参见 `IInputer::OnGetData()` 示例。

**代码位置**：
- 声明：`interfaces/inner_api/i_inputer_data.h:54`
- 实现：`services/modules/inputters/src/i_inputer_data_impl.cpp:41-58`
- 调用位置：由应用在 `IInputer::OnGetData()` 中调用

---

## 参数校验与错误码

### 空指针检查

**宏定义**（`common/utils/iam_check.h`）：

```cpp
#define IF_FALSE_LOGE_AND_RETURN(cond) \
    do { \
        if (!(cond)) { \
            IAM_LOGE("(" #cond ") check fail, return"); \
            return; \
        } \
    } while (0)

#define IF_FALSE_LOGE_AND_RETURN_VAL(cond, retVal) \
    do { \
        if (!(cond)) { \
            IAM_LOGE("(" #cond ") check fail, return"); \
            return (retVal); \
        } \
    } while (0)
```

**使用示例**：

```cpp
// framework/ipc/src/pin_auth_stub.cpp:43-45
if (inputer == nullptr) {
    IAM_LOGE("inputer is nullptr");
    return false;
}
```

### 权限检查

**实现位置**：`services/sa/src/pin_auth_service.cpp:107-113`

```cpp
bool PinAuthService::CheckPermission(const std::string &permission)
{
    IAM_LOGI("start");
    using namespace Security::AccessToken;
    uint32_t tokenId = GetTokenId();
    return AccessTokenKit::VerifyAccessToken(tokenId, permission) == RET_SUCCESS;
}
```

**使用的权限**：

| 权限 | 用途 | 检查位置 |
|------|------|---------|
| `ohos.permission.ACCESS_PIN_AUTH` | 注册/注销 Inputer | `services/sa/src/pin_auth_service.cpp:115` |
| `ohos.permission.ACCESS_AUTH_RESPOOL` | 访问认证资源池 | `sa_profile/dynamic_load/pinauth_sa_profile.cfg` |
| `ohos.permission.VIBRATE` | 振动反馈 | `sa_profile/dynamic_load/pinauth_sa_profile.cfg` |

### IPC 描述符验证

所有 Stub 实现验证接口描述符：

```cpp
// framework/ipc/src/pin_auth_stub.cpp:27-30
if (PinAuthStub::GetDescriptor() != data.ReadInterfaceToken()) {
    IAM_LOGE("descriptor is not matched");
    return UserAuth::GENERAL_ERROR;
}
```

### 错误返回

| API | 失败返回 | 说明 |
|------|---------|------|
| `RegisterInputer()` | `false` | 参数无效、权限不足、已注册 |
| `UnRegisterInputer()` | 无（void） | 无返回值 |
| IPC 错误 | `UserAuth::GENERAL_ERROR` | 描述符不匹配、数据读取失败 |

**日志级别**：
- `IAM_LOGI`：信息日志
- `IAM_LOGE`：错误日志
- 基于 OpenHarmony hilog

---

## 调用链路

### 注册流程调用链

```
应用层（Settings）
  ↓
PinAuthRegister::RegisterInputer()
  ↓ [frameworks/client/src/pinauth_register_impl.cpp:56]
PinAuthProxy::RegisterInputer()
  ↓ [IPC，跨进程]
PinAuthStub::OnRemoteRequest() → PinAuthInterfaceCode::REGISTER_INPUTER
  ↓ [frameworks/ipc/src/pin_auth_stub.cpp:37]
PinAuthService::RegisterInputer()
  ↓ [services/sa/src/pin_auth_service.cpp:115]
CheckPermission(ACCESS_PIN_AUTH)
  ↓
PinAuthManager::RegisterInputer(tokenId, inputer)
  ↓ [services/modules/inputters/src/pin_auth_manager.cpp:36]
添加到 pinAuthInputerMap_[tokenId]
  ↓
添加 Death Recipient
  ↓
返回 true
```

### 认证流程调用链

```
User Auth Framework
  ↓
PinAuthService (通过 user_auth_framework 调用 HDI)
  ↓
IAllInOneExecutor::Begin(scheduleId, executor, authToken)
  ↓ [HDI，跨安全边界]
南向厂商实现（TEE/安全芯片）
  ↓
IExecutorCallback::OnGetData(scheduleId, authToken)
  ↓ [services/modules/executors/src/pin_auth_executor_callback_hdi.cpp:44]
PinAuthManager::GetInputer(tokenId)
  ↓
Inputer::OnGetData(authSubType, challenge, inputerData)
  ↓ [应用实现]
显示 PIN 输入对话框
  ↓
IInputerData::OnSetData(authSubType, pinData)
  ↓ [services/modules/inputters/src/i_inputer_data_impl.cpp:41]
通过 InputerSetData IPC 传输到 SA
  ↓
Scrypt 处理（单向哈希）
  ↓
传输到 HDI 进行验证
  ↓
返回认证结果
```

---

## 代码证据总结

| API | 头文件位置 | 实现位置 | IPC 位置 |
|------|------------|----------|---------|
| PinAuthRegister | `interfaces/inner_api/pinauth_register.h` | `frameworks/client/src/pinauth_register_impl.cpp` | - |
| RegisterInputer | `interfaces/inner_api/pinauth_register.h:52` | `frameworks/client/src/pinauth_register_impl.cpp:56` | `frameworks/ipc/src/pin_auth_stub.cpp:37` |
| UnRegisterInputer | `interfaces/inner_api/pinauth_register.h:57` | `frameworks/client/src/pinauth_register_impl.cpp:82` | `frameworks/ipc/src/pin_auth_stub.cpp:59` |
| IInputer | `interfaces/inner_api/i_inputer.h` | 由应用实现 | - |
| OnGetData | `interfaces/inner_api/i_inputer.h:42` | 由应用实现 | SA 调用位置：`services/modules/executors/src/pin_auth_executor_callback_hdi.cpp:44` |
| IInputerData | `interfaces/inner_api/i_inputer_data.h` | `services/modules/inputters/src/i_inputer_data_impl.cpp` | - |
| OnSetData | `interfaces/inner_api/i_inputer_data.h:54` | `services/modules/inputters/src/i_inputer_data_impl.cpp:41` | - |

---

## 下一步

- 了解内部模块接口 → [内部 API](05_Inner_API.md)
- 了解安全机制 → [安全风险评审](08_Security_Review.md)
