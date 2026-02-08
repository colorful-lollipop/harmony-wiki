# 对外接口 (Native SDK)

## 目的与适用范围

本文档描述分布式屏幕对外提供的Native C++ SDK接口。

**重要说明**: 分布式屏幕**不提供JS/TS API**，仅提供Native C++接口。上层JS应用通过分布式硬件框架间接使用本能力。

---

## 接口概述

### 接口类型

| SDK | 适用角色 | 主要功能 |
|-----|----------|----------|
| `distributed_screen_source_sdk` | 主控端 (Source) | 使能/去使能远程屏幕 |
| `distributed_screen_sink_sdk` | 被控端 (Sink) | 订阅/取消订阅本地屏幕 |

### 接口位置

| SDK | 头文件路径 |
|-----|-----------|
| Source | `interfaces/innerkits/native_cpp/screen_source/include/idscreen_source.h` |
| Sink | `interfaces/innerkits/native_cpp/screen_sink/include/idscreen_sink.h` |

---

## Source端接口

### 接口类定义

**文件**: `interfaces/innerkits/native_cpp/screen_source/include/idscreen_source.h:25-40`

```cpp
class IDScreenSource : public OHOS::IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedscreensource");

    ~IDScreenSource() override {}
    
    // 初始化Source端
    virtual int32_t InitSource(const std::string &params, 
                               const sptr<IDScreenSourceCallback> &callback) = 0;
    
    // 释放Source端资源
    virtual int32_t ReleaseSource() = 0;
    
    // 注册分布式硬件（使能远程屏幕）
    virtual int32_t RegisterDistributedHardware(const std::string &devId, 
                                                const std::string &dhId,
                                                const EnableParam &param, 
                                                const std::string &reqId) = 0;
    
    // 注销分布式硬件（去使能远程屏幕）
    virtual int32_t UnregisterDistributedHardware(const std::string &devId, 
                                                  const std::string &dhId,
                                                  const std::string &reqId) = 0;
    
    // 配置分布式硬件参数
    virtual int32_t ConfigDistributedHardware(const std::string &devId, 
                                              const std::string &dhId,
                                              const std::string &key, 
                                              const std::string &value) = 0;
    
    // 通知事件
    virtual void DScreenNotify(const std::string &devId, 
                               int32_t eventCode,
                               const std::string &eventContent) = 0;
};
```

### 接口方法详解

#### InitSource

**功能**: 初始化Source端服务

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| params | `const std::string&` | 初始化参数（JSON格式）|
| callback | `const sptr<IDScreenSourceCallback>&` | 回调接口 |

**返回值**: `int32_t` - 错误码，0表示成功

**错误码**:
- `DH_SUCCESS` (0) - 成功
- `ERR_DH_SCREEN_SA_INIT_SOURCE_FAIL` (-50038) - 初始化失败

**调用链**:
```
DScreenSourceHandler::InitSource()
  └── DScreenSourceProxy::InitSource() [IPC]
       └── DScreenSourceStub::OnRemoteRequest()
            └── DScreenSourceStub::InitSourceInner()
                 └── DScreenSourceService::InitSource()
                      └── DScreenManager::Init()
```

---

#### RegisterDistributedHardware

**功能**: 注册分布式硬件（使能远程屏幕）

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| devId | `const std::string&` | 目标设备ID |
| dhId | `const std::string&` | 分布式硬件ID |
| param | `const EnableParam&` | 使能参数 |
| reqId | `const std::string&` | 请求ID |

**EnableParam结构** (来自分布式硬件框架):
```cpp
struct EnableParam {
    std::string sourceAttrs;    // 源端属性
    std::string sinkAttrs;      // 接收端属性
    std::string subtype;        // 子类型
};
```

**返回值**: `int32_t` - 错误码

**错误码**:
- `ERR_DH_SCREEN_SA_ENABLE_FAILED` (-50016) - 使能失败
- `ERR_DH_SCREEN_SA_CHECK_ENABLE_PERMISSION_FAIL` (-50039) - 权限检查失败

**权限要求**: 需要 `ohos.permission.ENABLE_DISTRIBUTED_HARDWARE`

---

#### UnregisterDistributedHardware

**功能**: 注销分布式硬件（去使能远程屏幕）

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| devId | `const std::string&` | 目标设备ID |
| dhId | `const std::string&` | 分布式硬件ID |
| reqId | `const std::string&` | 请求ID |

**返回值**: `int32_t` - 错误码

**错误码**:
- `ERR_DH_SCREEN_SA_DISABLE_FAILED` (-50017) - 去使能失败

---

### 回调接口

**文件**: `interfaces/innerkits/native_cpp/screen_source/include/idscreen_source_callback.h`

```cpp
class IDScreenSourceCallback : public OHOS::IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.dscreensourcecallback");

    // 注册结果通知
    virtual int32_t OnNotifyRegResult(const std::string &devId, 
                                      const std::string &dhId,
                                      const std::string &reqId, 
                                      int32_t status, 
                                      const std::string &data) = 0;
    
    // 注销结果通知
    virtual int32_t OnNotifyUnregResult(const std::string &devId, 
                                        const std::string &dhId,
                                        const std::string &reqId, 
                                        int32_t status, 
                                        const std::string &data) = 0;
    
    // 远程请求通知
    virtual int32_t OnRemoteRequest(uint32_t code, 
                                    MessageParcel &data, 
                                    MessageParcel &reply, 
                                    MessageOption &option) = 0;
};
```

---

## Sink端接口

### 接口类定义

**文件**: `interfaces/innerkits/native_cpp/screen_sink/include/idscreen_sink.h:23-34`

```cpp
class IDScreenSink : public OHOS::IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedscreensink");

    IDScreenSink() = default;
    ~IDScreenSink() override = default;
    
    // 初始化Sink端
    virtual int32_t InitSink(const std::string &params) = 0;
    
    // 释放Sink端资源
    virtual int32_t ReleaseSink() = 0;
    
    // 订阅本地硬件
    virtual int32_t SubscribeLocalHardware(const std::string &dhId, 
                                           const std::string &param) = 0;
    
    // 取消订阅本地硬件
    virtual int32_t UnsubscribeLocalHardware(const std::string &dhId) = 0;
    
    // 通知事件
    virtual void DScreenNotify(const std::string &devId, 
                               int32_t eventCode, 
                               const std::string &eventContent) = 0;
};
```

### 接口方法详解

#### InitSink

**功能**: 初始化Sink端服务

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| params | `const std::string&` | 初始化参数（JSON格式）|

**返回值**: `int32_t` - 错误码

---

#### SubscribeLocalHardware

**功能**: 订阅本地硬件（允许远程设备使用本机屏幕）

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| dhId | `const std::string&` | 分布式硬件ID |
| param | `const std::string&` | 订阅参数 |

**返回值**: `int32_t` - 错误码

---

#### UnsubscribeLocalHardware

**功能**: 取消订阅本地硬件

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| dhId | `const std::string&` | 分布式硬件ID |

**返回值**: `int32_t` - 错误码

---

## 使用方式

### Source端使用示例

```cpp
#include "idscreen_source.h"
#include "dscreen_source_handler.h"

// 1. 获取Handler实例
DScreenSourceHandler& handler = DScreenSourceHandler::GetInstance();

// 2. 初始化Source
std::string params = "{}";  // JSON参数
sptr<IDScreenSourceCallback> callback = new MySourceCallback();
int32_t ret = handler.InitSource(params, callback);
if (ret != DH_SUCCESS) {
    // 错误处理
}

// 3. 使能远程屏幕
std::string devId = "目标设备ID";
std::string dhId = "硬件ID";
EnableParam enableParam;
enableParam.sourceAttrs = "...";
enableParam.sinkAttrs = "...";
std::string reqId = "请求ID";
ret = handler.RegisterDistributedHardware(devId, dhId, enableParam, reqId);

// 4. 等待回调结果
// OnNotifyRegResult() 将被调用

// 5. 去使能远程屏幕
ret = handler.UnregisterDistributedHardware(devId, dhId, reqId);

// 6. 释放资源
handler.ReleaseSource();
```

### Sink端使用示例

```cpp
#include "idscreen_sink.h"
#include "dscreen_sink_handler.h"

// 1. 获取Handler实例
DScreenSinkHandler& handler = DScreenSinkHandler::GetInstance();

// 2. 初始化Sink
std::string params = "{}";
int32_t ret = handler.InitSink(params);

// 3. 订阅本地硬件
std::string dhId = "硬件ID";
std::string param = "参数";
ret = handler.SubscribeLocalHardware(dhId, param);

// 4. 取消订阅
ret = handler.UnsubscribeLocalHardware(dhId);

// 5. 释放资源
handler.ReleaseSink();
```

---

## IPC通信机制

### 接口描述符

| 接口 | 描述符字符串 |
|------|-------------|
| IDScreenSource | `u"ohos.distributedhardware.distributedscreensource"` |
| IDScreenSink | `u"ohos.distributedhardware.distributedscreensink"` |
| IDScreenSourceCallback | `u"ohos.distributedhardware.dscreensourcecallback"` |

### IPC命令码

**文件**: `common/include/dscreen_ipc_interface_code.h:23-40`

**Source端 (SA 4807)**:
| 命令码 | 值 | 对应方法 |
|--------|-----|----------|
| INIT_SOURCE | 0 | InitSource |
| RELEASE_SOURCE | 1 | ReleaseSource |
| REGISTER_DISTRIBUTED_HARDWARE | 2 | RegisterDistributedHardware |
| UNREGISTER_DISTRIBUTED_HARDWARE | 3 | UnregisterDistributedHardware |
| CONFIG_DISTRIBUTED_HARDWARE | 4 | ConfigDistributedHardware |
| DSCREEN_NOTIFY | 5 | DScreenNotify |

**Sink端 (SA 4808)**:
| 命令码 | 值 | 对应方法 |
|--------|-----|----------|
| INIT_SINK | 0 | InitSink |
| RELEASE_SINK | 1 | ReleaseSink |
| SUBSCRIBE_DISTRIBUTED_HARDWARE | 2 | SubscribeLocalHardware |
| UNSUBSCRIBE_DISTRIBUTED_HARDWARE | 3 | UnsubscribeLocalHardware |
| DSCREEN_NOTIFY | 4 | DScreenNotify |

---

## 权限要求

### 权限声明

```xml
<!-- 在应用的config.json中声明 -->
"reqPermissions": [
    {
        "name": "ohos.permission.ENABLE_DISTRIBUTED_HARDWARE",
        "reason": "需要使能分布式硬件"
    }
]
```

### 权限检查实现

**文件**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_stub.cpp`

```cpp
bool DScreenSourceStub::HasEnableDHPermission() {
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ENABLE_DISTRIBUTED_HARDWARE";
    int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    return (result == Security::AccessToken::PERMISSION_GRANTED);
}
```

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目基本信息
- [架构设计](01_Architecture.md) - 架构说明
- [内部接口](04_Inner_APIs.md) - 内部模块接口
- [安全风险](07_Security.md) - 安全分析