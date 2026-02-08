# Inner API 文档

## 5.1 模块概述

Inner API 是面向 **Native（C/C++）** 开发者的内部接口，位于 `interfaces/innerkits/` 目录下。这些接口主要用于：
- 系统服务开发
- 底层 IPC 框架扩展
- 跨进程通信定制

### 头文件目录

| 目录 | 职责 |
|------|------|
| `interfaces/innerkits/ipc_core/` | 核心 IPC 接口 |
| `interfaces/innerkits/ipc_single/` | 单进程 IPC 库 |
| `interfaces/innerkits/c_api/` | C API 接口 |
| `interfaces/innerkits/libdbinder/` | DBinder 接口 |

## 5.2 核心类体系

### 类继承关系

```
RefBase
└── IRemoteObject
    ├── IPCObjectStub (服务端存根)
    │   └── IRemoteStub<T> (模板)
    └── IPCObjectProxy (客户端代理)
        └── IRemoteProxy<T> (模板)
            └── IRemoteBroker (接口基类)
```

### 关键头文件

| 头文件 | 职责 | 行号 |
|--------|------|------|
| `iremote_object.h` | 远程对象基类 | `ipc_core/include/:34` |
| `iremote_proxy.h` | 远程代理模板 | `ipc_core/include/:24` |
| `iremote_stub.h` | 远程存根模板 | `ipc_core/include/:24` |
| `iremote_broker.h` | 远程代理接口基类 | `ipc_core/include/` |
| `ipc_skeleton.h` | IPC 骨架 | `ipc_core/include/:22` |
| `message_parcel.h` | 消息数据包 | `ipc_core/include/:26` |
| `message_option.h` | 消息选项 | `ipc_core/include/:21` |

## 5.3 IRemoteBroker 接口

### 接口定义

```cpp
#include "iremote_broker.h"

class IRemoteBroker {
public:
    virtual sptr<IRemoteObject> AsObject() = 0;
};
```

### 使用方式

业务接口继承 `IRemoteBroker`，用于解耦业务接口与 IPC 实现：

```cpp
// 1. 定义业务接口
class ITestAbility : public IRemoteBroker {
public:
    virtual int TestPing(const std::u16string &dummy) = 0;
    DECLARE_INTERFACE_DESCRIPTOR(u"test.ITestAbility");
};

// 2. Stub 端返回 RemoteObject
class TestAbilityStub : public IRemoteStub<ITestAbility> {
public:
    sptr<IRemoteObject> AsObject() override {
        return this;
    }
};

// 3. Proxy 端返回代理对象
class TestAbilityProxy : public IRemoteProxy<ITestAbility> {
public:
    sptr<IRemoteObject> AsObject() override {
        return Remote();
    }
};
```

**证据**: `README_zh.md:321-333` - 接口定义示例

## 5.4 IRemoteProxy 模板

### 类定义

```cpp
template <typename T>
class IRemoteProxy : public T, public PeerHolder {
public:
    explicit IRemoteProxy(const sptr<IRemoteObject> &impl);
    virtual sptr<IRemoteObject> AsObject() override;
};
```

### 核心方法

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `SendRequest()` | `code`, `data`, `reply`, `option` | `int` | 发送请求 |
| `AsObject()` | 无 | `sptr<IRemoteObject>` | 获取底层代理 |

### 使用示例

```cpp
// 定义业务接口
class ITestAbility : public IRemoteBroker {
public:
    virtual int TestPing(const std::u16string &dummy) = 0;
};

// 实现 Proxy
class TestAbilityProxy : public IRemoteProxy<ITestAbility> {
public:
    explicit TestAbilityProxy(const sptr<IRemoteObject> &impl)
        : IRemoteProxy<ITestAbility>(impl) {}

    int TestPing(const std::u16string &dummy) override {
        MessageOption option;
        MessageParcel data, reply;
        data.WriteInterfaceToken(GetDescriptor());
        data.WriteString16(dummy);
        int error = Remote()->SendRequest(TRANS_ID_PING, data, reply, option);
        return (error == ERR_NONE) ? reply.ReadInt32() : -1;
    }

private:
    static inline BrokerDelegator<TestAbilityProxy> delegator_;
};
```

**证据**: `README_zh.md:388-414` - Proxy 实现示例

## 5.5 IRemoteStub 模板

### 类定义

```cpp
template <typename T>
class IRemoteStub : public IPCObjectStub {
public:
    virtual sptr<IRemoteObject> AsObject() override;
    virtual sptr<IInterface> AsInterface(const sptr<IRemoteObject> &impl);
};
```

### 核心方法

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `OnRemoteRequest()` | `code`, `data`, `reply`, `option` | `int` | 处理远程请求 |
| `AsObject()` | 无 | `sptr<IRemoteObject>` | 获取对象 |
| `AsInterface()` | `IRemoteObject` | `sptr<IInterface>` | 转换为业务接口 |

### 使用示例

```cpp
// 实现 Stub
class TestAbilityStub : public IRemoteStub<ITestAbility> {
public:
    int OnRemoteRequest(uint32_t code, MessageParcel &data,
                       MessageParcel &reply, MessageOption &option) override {
        // 校验接口令牌
        if (data.ReadInterfaceToken() != GetDescriptor()) {
            return -1;
        }
        switch (code) {
            case TRANS_ID_PING: {
                std::u16string dummy = data.ReadString16();
                int result = TestPing(dummy);
                reply.WriteInt32(result);
                return 0;
            }
            default:
                return IPCObjectStub::OnRemoteRequest(code, data, reply, option);
        }
    }
};
```

**证据**: `README_zh.md:342-365` - Stub 实现示例

## 5.6 IPCSkeleton 静态接口

### 头文件

```cpp
#include "ipc_skeleton.h"
```

### 核心静态方法

| 方法 | 返回值 | 描述 |
|------|--------|------|
| `GetCallingPid()` | `int` | 获取调用方 PID |
| `GetCallingUid()` | `int` | 获取调用方 UID |
| `GetCallingTokenID()` | `uint32_t` | 获取调用方 TokenID |
| `GetLocalDeviceID()` | `std::string` | 获取本地设备 ID |
| `GetCallingDeviceID()` | `std::string` | 获取调用方设备 ID |
| `IsLocalCalling()` | `bool` | 是否本地调用 |
| `GetContextObject()` | `sptr<IRemoteObject>` | 获取上下文对象 |
| `ResetCallingIdentity()` | `bool` | 重置调用身份 |
| `SetCallingIdentity()` | `bool` | 设置调用身份 |

### 线程管理

| 方法 | 描述 |
|------|------|
| `SetMaxWorkThreadNum(int)` | 设置 IPC 线程池最大线程数 |
| `JoinWorkThread()` | 当前线程加入 IPC 工作线程池 |
| `StopWorkThread()` | 停止 IPC 工作线程 |

**证据**: `ipc_skeleton.h:22-178`

## 5.7 MessageParcel 接口

### 头文件

```cpp
#include "message_parcel.h"
```

### 读写方法

| 分类 | 方法 | 描述 |
|------|------|------|
| **对象** | `WriteRemoteObject()` / `ReadRemoteObject()` | 远程对象序列化 |
| **令牌** | `WriteInterfaceToken()` / `ReadInterfaceToken()` | 接口令牌 |
| **基本类型** | `WriteInt32()` / `ReadInt32()` | 32 位整数 |
| | `WriteInt64()` / `ReadInt64()` | 64 位整数 |
| | `WriteFloat()` / `ReadFloat()` | 浮点数 |
| | `WriteDouble()` / `ReadDouble()` | 双精度浮点数 |
| | `WriteBool()` / `ReadBool()` | 布尔值 |
| **字符串** | `WriteString16()` / `ReadString16()` | UTF-16 字符串 |
| | `WriteString()` / `ReadString()` | UTF-8 字符串 |
| **数组** | `WriteInt32Array()` / `ReadInt32Array()` | 整数数组 |
| **原始数据** | `WriteRawData()` / `ReadRawData()` | 原始数据块 |
| **共享内存** | `WriteAshmem()` / `ReadAshmem()` | Ashmem 对象 |
| **文件描述符** | `WriteFileDescriptor()` / `ReadFileDescriptor()` | FD |
| **异常** | `WriteNoException()` / `ReadException()` | 异常标记 |

### 使用示例

```cpp
MessageParcel data, reply;

// 写入数据
data.WriteInterfaceToken(GetDescriptor());
data.WriteInt32(100);
data.WriteString16(u"test");
data.WriteRemoteObject(proxy);

// 读取数据
int32_t value = data.ReadInt32();
std::u16string str = data.ReadString16();
sptr<IRemoteObject> obj = data.ReadRemoteObject();
```

**证据**: `message_parcel.h:26-164`

## 5.8 MessageOption 接口

### 头文件

```cpp
#include "message_option.h"
```

### 标志定义

| 标志 | 值 | 描述 |
|------|-----|------|
| `TF_SYNC` | `0x00` | 同步调用（默认） |
| `TF_ASYNC` | `0x01` | 异步调用 |
| `TF_STATUS_CODE` | `0x08` | 包含状态码 |
| `TF_ACCEPT_FDS` | `0x10` | 接受文件描述符 |
| `TF_WAIT_TIME` | `0x8` | 等待时间标志 |

### 使用示例

```cpp
MessageOption option;

// 同步调用
option.setFlags(MessageOption::TF_SYNC);

// 异步调用
option.setFlags(MessageOption::TF_ASYNC);

// 设置超时（毫秒）
option.setWaitTime(5000);

// 组合标志
option.setFlags(MessageOption::TF_SYNC | MessageOption::TF_ACCEPT_FDS);
```

**证据**: `message_option.h:21-65`

## 5.9 DBinder Service 接口

### 头文件

```cpp
#include "dbinder_service.h"
```

### 核心方法

| 方法 | 返回值 | 描述 |
|------|--------|------|
| `GetInstance()` | `DBinderService*` | 获取单例 |
| `RegisterRemoteProxy()` | `int` | 注册远程代理 |
| `MakeRemoteBinder()` | `sptr<IRemoteObject>` | 创建远程 Binder |
| `OnRemoteMessageTask()` | `int` | 处理远程消息 |

### 使用示例

```cpp
auto dbinder = DBinderService::GetInstance();
int ret = dbinder->RegisterRemoteProxy(serviceName, remoteProxy);
```

**证据**: `dbinder_service.h:126-165`

## 5.10 模块依赖方向

```
                    ┌─────────────────┐
                    │   System App    │
                    └────────┬────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────┐
│                      Native Service                           │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  │
│  │ IRemoteStub    │  │ MessageParcel  │  │ MessageOption  │  │
│  │ (服务端)       │  │ (消息序列化)    │  │ (调用配置)     │  │
│  └───────┬────────┘  └────────────────┘  └────────────────┘  │
└──────────┼──────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────────┐
│                     IPC Framework                              │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  │
│  │ IPCObjectStub  │  │ IPCObjectProxy │  │ IPCSkeleton    │  │
│  │ (存根实现)     │  │ (代理实现)      │  │ (骨架工具)     │  │
│  └───────┬────────┘  └────────────────┘  └────────────────┘  │
└──────────┼──────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────────┐
│                   Binder Driver                               │
│                   (内核层)                                    │
└──────────────────────────────────────────────────────────────┘
```

---

*证据来源*:
- `README_zh.md:319-465` - Native API 使用说明
- `interfaces/innerkits/ipc_core/include/` - 头文件定义
- Phase 1 全局扫描结果
