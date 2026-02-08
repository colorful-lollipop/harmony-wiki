# SAFWK API 接口

## 重要说明

**SAFWK 不提供 N-API (Node.js API)**

SAFWK 是 OpenHarmony 的原生 C++ 框架，**不提供** JavaScript/TypeScript/ArkTS API。JS 层 API 由其他仓库（如 `samgr`）提供。

本文档覆盖:
- **C++ SDK**: `interfaces/innerkits/safwk/` 中的头文件
- **Rust 绑定**: `interfaces/innerkits/safwk/rust/` 中的 FFI 接口

---

## C++ SDK

### SystemAbility 基类

**头文件**: `interfaces/innerkits/safwk/system_ability.h`

**继承关系**:
```
SystemAbility (基类)
    ↓ 继承
所有具体 SA 实现
```

#### 注册宏

| 宏 | 位置 | 用途 |
|----|------|------|
| `REGISTER_SYSTEM_ABILITY_BY_ID` | `system_ability.h:29` | 注册 SA 到框架 |
| `DECLARE_SYSTEM_ABILITY` | `system_ability.h:43-47` | 声明 SA 类名 |
| `DECLARE_BASE_SYSTEM_ABILITY` | `system_ability.h:49-51` | 声明抽象基类 |

**使用示例** (`system_ability.h:29-31`):
```cpp
#define REGISTER_SYSTEM_ABILITY_BY_ID(abilityClassName, systemAbilityId, runOnCreate) \
    const bool abilityClassName##_##RegisterResult = \
    SystemAbility::MakeAndRegisterAbility(new abilityClassName(systemAbilityId, runOnCreate));

// 用法:
REGISTER_SYSTEM_ABILITY_BY_ID(MyAbility, MY_ABILITY_ID, true);
```

#### 生命周期方法

| 方法签名 | 描述 | 继承要求 |
|----------|------|----------|
| `virtual void OnStart()` | SA 启动 | 必须覆盖 |
| `virtual void OnStop()` | SA 停止 | 必须覆盖 |
| `virtual void OnStart(const SystemAbilityOnDemandReason&)` | 按需启动 | 可选 |
| `virtual void OnStop(const SystemAbilityOnDemandReason&)` | 按需停止 | 可选 |
| `virtual int32_t OnIdle(const SystemAbilityOnDemandReason&)` | 空闲回调 | 可选 |
| `virtual void OnActive(const SystemAbilityOnDemandReason&)` | 激活回调 | 可选 |
| `virtual void OnDump()` | Dump 信息 | 可选 |

**代码位置**: `interfaces/innerkits/safwk/system_ability.h:103-143`

#### 静态方法

| 方法签名 | 描述 | 返回值 |
|----------|------|--------|
| `static bool MakeAndRegisterAbility(SystemAbility*)` | 注册 SA | 是否成功 |
| `static void StopAbility(int32_t systemAbilityId)` | 停止指定 SA | void |

#### 实例方法

| 方法签名 | 描述 | 返回值 |
|----------|------|--------|
| `bool Publish(sptr<IRemoteObject> systemAbility)` | 发布 SA | 是否成功 |
| `sptr<IRemoteObject> GetSystemAbility(int32_t systemAbilityId)` | 获取 SA | IRemoteObject |
| `bool AddSystemAbilityListener(int32_t systemAbilityId)` | 添加监听 | 是否成功 |
| `bool RemoveSystemAbilityListener(int32_t systemAbilityId)` | 移除监听 | 是否成功 |
| `bool CancelIdle()` | 取消空闲 | 是否成功 |
| `SystemAbilityState GetAbilityState()` | 获取状态 | SystemAbilityState |

### 按需启动原因

**头文件**: `interfaces/innerkits/safwk/system_ability_ondemand_reason.h`

**类**: `SystemAbilityOnDemandReason`

| 方法 | 描述 |
|------|------|
| `int32_t GetReason()` | 获取触发原因 |
| `const std::string& GetReasonStr()` | 获取原因字符串 |
| `int32_t GetLastTime()` | 获取上次触发时间 |
| `void SetReason(int32_t reason)` | 设置原因 |
| `void SetReasonStr(const std::string& reason)` | 设置原因字符串 |

### ApiCacheManager

**头文件**: `interfaces/innerkits/safwk/api_cache_manager.h`

#### API 清单表

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `GetInstance()` | - | ApiCacheManager& | 单例访问 |
| `AddCacheApi()` | descriptor, apiCode, expireTimeSec | void | 添加缓存 |
| `DelCacheApi()` | descriptor, apiCode | void | 删除缓存 |
| `ClearCache()` | - | void | 清空所有 |
| `ClearCache()` | descriptor | void | 按描述符清空 |
| `ClearCache()` | descriptor, apiCode | void | 按 API 清空 |
| `PreSendRequest()` | descriptor, apiCode, data, reply | bool | 发送前处理 |
| `PostSendRequest()` | descriptor, apiCode, data, reply | bool | 发送后处理 |

**稳定性**: 不稳定接口，用于性能优化

### IRemoteBroker 接口

**头文件**: (IPC 框架提供，引用)

**继承要求**: 所有 SA 的 IPC 接口必须继承此基类

```cpp
class IMyAbility : public IRemoteBroker {
public:
    virtual int32_t DoSomething(int32_t param) = 0;
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.test.IMyAbility");  // 必须
};
```

### IRemoteProxy / IRemoteStub

**头文件**: (IPC 框架提供)

**Proxy (客户端)**:
```cpp
class MyAbilityProxy : public IRemoteProxy<IMyAbility> {
public:
    int32_t DoSomething(int32_t param);
};
```

**Stub (服务端)**:
```cpp
class MyAbilityStub : public IRemoteStub<IMyAbility> {
public:
    int32_t DoSomething(int32_t param) override;
    int32_t OnRemoteRequest(uint32_t code, MessageParcel& data, 
                            MessageParcel& reply, MessageOption& option) override;
};
```

---

## Rust 绑定

### SystemAbility Trait

**文件**: `interfaces/innerkits/safwk/rust/src/ability.rs`

**Rust FFI 定义**:
```rust
#[cxx::bridge]
mod ffi {
    unsafe impl SystemAbility for SystemAbilityWrapper {
        fn on_start(&mut self, start_reason: SystemAbilityOnDemandReason);
        fn on_stop(&mut self, stop_reason: SystemAbilityOnDemandReason);
        // ... 其他生命周期方法
    }
}
```

### FFI 包装器

**文件**: `interfaces/innerkits/safwk/rust/src/wrapper.rs`

**关键结构**:

| 结构 | 描述 |
|------|------|
| `SystemAbilityWrapper` | 继承 SystemAbility 的 Rust 包装器 |
| `AbilityStub` | Rust IPC Stub 实现 |
| `SystemAbilityOnDemandReason` | 按需启动原因 (CXX 共享类型) |

### Rust 示例

**位置**: `interfaces/innerkits/safwk/rust/examples/`

| 示例 | 描述 |
|------|------|
| `audio_rust_sa/` | Rust SA 实现示例 |
| `listen_rust_sa/` | Rust 监听 SA 示例 |

---

## 开发步骤

### 步骤 1: 定义 IPC 接口

```cpp
// IMyAbility.h
#include "iremote_broker.h"

class IMyAbility : public IRemoteBroker {
public:
    virtual int32_t Add(int32_t a, int32_t b) = 0;
    virtual int32_t Sub(int32_t a, int32_t b) = 0;
    
    enum {
        ADD = 1,
        SUB = 2,
    };
    
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.test.IMyAbility");
};
```

### 步骤 2: 实现 Stub

```cpp
// MyAbilityStub.cpp
int32_t MyAbilityStub::OnRemoteRequest(uint32_t code, 
    MessageParcel& data, MessageParcel& reply, MessageOption& option)
{
    switch (code) {
        case ADD: {
            int32_t a = data.ReadInt32();
            int32_t b = data.ReadInt32();
            reply.WriteInt32(Add(a, b));
            return 0;
        }
        case SUB: {
            // ...
        }
        default:
            return IPCObjectStub::OnRemoteRequest(code, data, reply, option);
    }
}
```

### 步骤 3: 实现 SA

```cpp
// MyAbility.cpp
#include "system_ability.h"

class MyAbility : public SystemAbility {
public:
    MyAbility(int32_t saId, bool runOnCreate) 
        : SystemAbility(saId, runOnCreate) {}
    
    void OnStart() override {
        Publish(this);  // 必须调用
    }
    
    void OnStop() override {
        // 清理资源
    }
    
    int32_t Add(int32_t a, int32_t b) override {
        return a + b;
    }
    
private:
    DECLARE_SYSTEM_ABILITY(MyAbility);
};

// 注册 SA
REGISTER_SYSTEM_ABILITY_BY_ID(MyAbility, MY_ABILITY_ID, false);
```

### 步骤 4: 配置 SA Profile

```json
// sa_profile/1234.json
{
    "process": "my_service",
    "systemability": [
        {
            "name": 1234,
            "libpath": "libmy_ability.z.so",
            "run-on-create": false,
            "distributed": true,
            "dump_level": 1
        }
    ]
}
```

---

## 错误码参考

| 错误码 | 定义位置 | 含义 |
|--------|----------|------|
| 0 | - | 成功 |
| `-1` | SystemAbility | SA 不存在 |
| `-2` | LocalAbilityManager | 状态错误 |

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 架构设计 | [01_Architecture.md](01_Architecture.md) |
| 构建系统 | [03_Build.md](03_Build.md) |
| 安全评审 | [04_Security.md](04_Security.md) |
