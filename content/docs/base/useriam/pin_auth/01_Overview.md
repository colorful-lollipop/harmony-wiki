# 项目概览

> **目的**：新人 5 分钟内快速了解 pin_auth 模块的全貌
> **适用范围**：所有开发者、架构师、安全审计人员
> **关键结论**：pin_auth 是基于 Native C++ 的 System Ability，通过 IPC 和 HDI 提供安全 PIN 认证能力
> **相关文档**：[目录结构](02_Directory.md) | [架构设计](03_Architecture.md) | [对外 Native API](04_Native_API.md)

---

## 项目定位

### PIN 认证是什么

PIN（Personal Identification Number）认证是 OpenHarmony 最基础的用户身份认证方式，提供以下核心能力：

- ✅ **PIN 设置**：用户首次设置 PIN 或修改已设置的 PIN
- ✅ **PIN 删除**：删除已设置的 PIN（通常在重置设备时）
- ✅ **PIN 认证**：验证用户输入的 PIN 是否正确（如锁屏解锁）
- ✅ **PIN 修改**：配合 User IAM 框架实现 PIN 更改流程

### 在 OpenHarmony 中的位置

```
┌─────────────────────────────────────────────────────────┐
│         OpenHarmony User IAM 子系统              │
├─────────────────────────────────────────────────────────┤
│                                                   │
│  user_auth_framework (JS API 层)                   │
│         ↓                                         │
│  ┌─────────────┐                                │
│  │  pin_auth   │ ← 本模块                      │
│  │  (SA: 941)  │                                │
│  └──────┬──────┘                                │
│         ↓                                         │
│  drivers_interface_pin_auth (HDI 接口层)           │
│         ↓                                         │
│  南向厂商实现 (TEE / 安全芯片)                      │
└─────────────────────────────────────────────────────────┘
```

**关键特征**：
- pin_auth 模块本身是 **Native C++ 服务**，不提供 JavaScript N-API
- JavaScript API 由 `user_auth_framework` 组件提供，该组件内部调用 pin_auth SA
- 通过 OpenHarmony **IPC（进程间通信）** 与上层应用交互
- 通过 **HDI（硬件驱动接口）** 与南向安全环境（TEE/安全芯片）交互

### 与其他认证方式的关系

| 认证类型 | OpenHarmony 模块 | 代码仓库 |
|---------|-----------------|---------|
| PIN 认证 | pin_auth | [useriam_pin_auth](https://gitee.com/openharmony/useriam_pin_auth) |
| 人脸认证 | face_auth | [useriam_face_auth](https://gitee.com/openharmony/useriam_face_auth) |
| 指纹认证 | fingerprint_auth | (单独模块) |

pin_auth 作为基础认证执行器，通过 User IAM 框架的资源注册接口注册认证资源，并基于框架调度完成认证操作。

---

## 核心能力

### 1. PIN 输入对话框管理

系统级应用（Settings、锁屏等）通过注册 Inputer 回调来提供 PIN 输入界面：

- `RegisterInputer()` - 注册输入对话框
- `UnRegisterInputer()` - 注销输入对话框
- `OnGetData()` - 服务端请求 PIN 数据
- `OnSetData()` - 客户端返回 PIN 数据

**安全措施**：
- 输入对话框注册需要 `ohos.permission.ACCESS_PIN_AUTH` 权限
- 仅系统级应用可调用（Settings 应用、锁屏应用等）
- PIN 数据经过单向处理后传入 SA，原文不跨设备传输

### 2. PIN 认证操作

通过 HDI 接口执行以下操作：

- **注册**：设置新 PIN（enroll）
- **认证**：验证 PIN 是否正确（auth）
- **删除**：删除已设置的 PIN（delete）

**执行器类型**：
- `IAllInOneExecutor` - 全功能执行器（注册+认证一体）
- `ICollector` - 数据收集器（用于某些特殊场景）
- `IVerifier` - 验证器（仅用于验证）

### 3. 动态加载支持

支持两种加载模式（通过 `pin_auth_enable_dynamic_load` 配置）：

| 模式 | 进程 | 说明 | 场景 |
|------|------|------|------|
| 静态加载 | useriam | 默认模式，系统启动时直接加载 | 标准 OpenHarmony 设备 |
| 动态加载 | pinauth | 按需加载，支持驱动延迟初始化 | 需要优化启动时间的设备 |

动态加载模式会订阅系统事件和参数，根据需要动态启动驱动。

### 4. 硬件安全集成

HDI 接口定义了南向厂商适配接口，厂商可以在以下环境中实现：

- **TEE（可信执行环境）**：安全隔离的执行环境
- **安全芯片**：专用硬件模块

OpenHarmony 框架提供了纯软件实现供演示使用，但不包含安全存储能力。

---

## 运行环境

### 系统要求

- **OpenHarmony 版本**：4.0+
- **子系统**：useriam
- **进程**：`useriam`（默认）/ `pinauth`（动态加载）
- **SAID**：941 (`SUBSYS_USERIAM_SYS_ABILITY_PINAUTH`)
- **SELinux 上下文**：`u:r:pinauth:s0`
- **APL（能力特权级）**：`system_basic`

### 依赖组件

从 `bundle.json` 中的核心依赖：

```json
{
  "components": [
    "ability_base",           // 能力基础
    "hilog",                // 日志系统
    "ipc",                  // 进程间通信
    "safwk",               // 系统能力框架
    "samgr",                // 能力管理器
    "access_token",          // 访问令牌管理
    "user_auth_framework",    // 用户认证框架
    "drivers_interface_pin_auth",  // PIN 驱动接口
    "openssl",              // 加密库（scrypt）
    "hisysevent"           // 系统事件
  ]
}
```

**可选依赖**：
- `sensors_miscdevice` - 振动反馈（需要 `sensors_miscdevice_enable=true`）
- `enterprise_device_management` - 企业设备管理（需要 `customization_enterprise_device_management_enable=true`）

### 资源占用

从 `bundle.json` 中的声明：

- **ROM**：1024KB（代码段）
- **RAM**：6072KB（运行时内存）

---

## 关键概念

### System Ability (SA)

OpenHarmony 的系统能力机制，每个 SA 独立运行在自己的进程中。

- **注册**：通过 `SystemAbility::MakeAndRegisterAbility()` 自动注册到 SAMgr
- **发现**：客户端通过 `SystemAbilityManager::GetSystemAbility(SAID)` 获取服务代理
- **生命周期**：`OnStart()` 初始化，`OnStop()` 清理

pin_auth SA 的 SAID 是 **941**。

### IPC (Inter-Process Communication)

进程间通信机制，用于客户端和服务之间的数据传输。

**模式**：OpenHarmony 使用 **Binder** IPC 框架

**组成**：
- **Proxy**：客户端代理，发送请求到远程
- **Stub**：服务端实现，接收并处理请求
- **Descriptor**：接口描述符，用于验证和路由

pin_auth 实现了三组 IPC 接口：
1. `PinAuthInterface` - 输入器注册/注销
2. `InputerGetData` - 请求 PIN 数据
3. `InputerSetData` - 传输 PIN 数据

### HDI (Hardware Driver Interface)

硬件驱动接口，定义了操作系统与硬件之间的抽象层。

**作用**：
- 隔离上层应用与硬件实现
- 允许南向厂商定制实现
- 在 TEE 或安全芯片中执行敏感操作

**接口**：
- `IPinAuthInterface` - 主驱动接口
- `IAllInOneExecutor` / `ICollector` / `IVerifier` - 不同类型的执行器
- `IExecutorCallback` - 驱动回调接口

### Token ID

访问令牌 ID，用于标识调用者的身份。

**作用**：
- 权限检查的基础
- 调用者隔离的键
- 支持委托调用（delegation）

**获取方式**：
```cpp
// 优先检查委托令牌
uint32_t tokenId = GetFirstTokenID();
if (tokenId == 0) {
    tokenId = GetCallingTokenID();  // 回退到直接调用者
}
```

pin_auth 使用 Token ID 作为 `pinAuthInputerMap_` 的键，确保不同调用者的 Inputer 隔离。

### Death Recipient

死亡接收者，用于监控远程对象的生命周期。

**作用**：
- 客户端监控服务死亡，自动重连
- 服务监控客户端死亡，自动清理资源

pin_auth 在以下场景使用：
- **客户端**：监控 pin_auth SA 死亡（PinAuthDeathRecipient）
- **服务端**：监控注册的 Inputer 死亡（ResPinauthInputerDeathRecipient）

### Inputer（输入器）

系统级应用实现的输入对话框回调接口。

**实现者**：
- Settings 应用：PIN 设置对话框
- 锁屏应用：PIN 认证对话框

**接口**：
- `IInputer::OnGetData()` - 服务端调用此方法请求 PIN
- `IInputerData::OnSetData()` - 客户端调用此方法返回 PIN

**安全要求**：
- 需要系统签名
- 需要 `ohos.permission.ACCESS_PIN_AUTH` 权限
- PIN 数据经过单向哈希处理后传输

---

## 代码证据

### Service Ability 定义

**文件**：`services/sa/inc/pin_auth_service.h:31`

```cpp
class PinAuthService : public SystemAbility, public PinAuthStub
{
public:
    DECLEAR_SYSTEM_ABILITY(PinAuthService);
    // ...
};
```

### SA 注册

**文件**：`services/sa/src/pin_auth_service.cpp:40`

```cpp
const bool REGISTER_RESULT = SystemAbility::MakeAndRegisterAbility(
    PinAuthService::GetInstance().get()
);
```

### 公共 API 定义

**文件**：`interfaces/inner_api/pinauth_register.h:32-58`

```cpp
class PinAuthRegister {
public:
    static PinAuthRegister &GetInstance();
    virtual bool RegisterInputer(std::shared_ptr<IInputer> inputer) = 0;
    virtual void UnRegisterInputer() = 0;
};
```

### HDI 接口引用

**文件**：`services/modules/common/inc/pin_auth_hdi.h`

```cpp
using IPinAuthInterface = OHOS::HDI::PinAuth::V3_0::IPinAuthInterface;
using IAllInOneExecutor = OHOS::HDI::PinAuth::V3_0::IAllInOneExecutor;
using ICollector = OHOS::HDI::PinAuth::V3_0::ICollector;
using IVerifier = OHOS::HDI::PinAuth::V3_0::IVerifier;
```

### SA Profile 配置

**文件**：`sa_profile/default/941.json:1-13`

```json
{
    "process": "useriam",
    "systemability": [
        {
            "name": 941,
            "libpath": "libpinauthservice.z.so",
            "run-on-create": true
        }
    ]
}
```

---

## 下一步

- 了解代码组织 → [目录结构](02_Directory.md)
- 理解系统架构 → [架构设计](03_Architecture.md)
- 学习 API 使用 → [对外 Native API](04_Native_API.md)
