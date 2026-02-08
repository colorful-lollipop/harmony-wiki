# 内部 API

## 模块接口概览

```
┌─────────────────────────────────────────────────────────────────┐
│                     face_auth 内部模块结构                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      IPC 层                               │  │
│  │  ┌─────────────────┐    ┌─────────────────────────────┐ │  │
│  │  │ FaceAuthClient  │───▶│ FaceAuthClientImpl          │ │  │
│  │  │ (接口)           │    │ (单例实现)                   │ │  │
│  │  └─────────────────┘    └──────────────┬──────────────┘ │  │
│  │                                         │                   │  │
│  │  ┌─────────────────┐    ┌─────────────────────────────┐ │  │
│  │  │ FaceAuthProxy   │    │ FaceAuthStub                │ │  │
│  │  │ (序列化/发送)    │    │ (反序列化/分发)              │ │  │
│  │  └─────────────────┘    └─────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                    │                             │
│                                    ▼                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      SA 层                                │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                   FaceAuthService                   │  │  │
│  │  │  - OnStart() / OnStop()                            │  │  │
│  │  │  - SetBufferProducer()                             │  │  │
│  │  │  - IsPermissionGranted()                            │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                    │                             │
│                                    ▼                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      HDI 适配层                            │  │
│  │  ┌─────────────────┐    ┌─────────────────────────────┐   │  │
│  │  │ FaceAuthDriver │    │ FaceAuthAllInOneExecutorHDI │   │  │
│  │  │ HDI             │    │                             │   │  │
│  │  └─────────────────┘    └─────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## FaceAuthClient 接口

**头文件**: `frameworks/ipc/inc/face_auth_client.h`

**接口定义**:

```cpp
class FaceAuthClient {
public:
    static FaceAuthClient &GetInstance();

    // 设置 Buffer 生产者
    virtual int32_t SetBufferProducer(sptr<IBufferProducer> &producer) = 0;

protected:
    virtual ~FaceAuthClient() = default;
};
```

**稳定性**: 稳定（platformsdk 标注）

**调用方**: N-API 层 (`face_auth_napi.cpp`)

## IFaceAuth 接口

**头文件**: `frameworks/ipc/inc/iface_auth.h`

**接口定义**:

```cpp
class IFaceAuth : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.faceauth.IFaceAuth");

    // 设置 Buffer 生产者
    virtual int32_t SetBufferProducer(sptr<IBufferProducer> &producer) = 0;
};
```

**接口描述符**: `ohos.faceauth.IFaceAuth`

**稳定性**: 稳定

## FaceAuthProxy

**头文件**: `frameworks/ipc/inc/face_auth_proxy.h`

**实现**: `frameworks/ipc/src/face_auth_proxy.cpp`

**核心方法**:

```cpp
class FaceAuthProxy : public IRemoteProxy<IFaceAuth> {
public:
    explicit FaceAuthProxy(const sptr<IRemoteObject> &object);

    // 设置 Buffer 生产者
    int32_t SetBufferProducer(sptr<IBufferProducer> &producer) override;
};
```

**序列化逻辑**:

```cpp
// face_auth_proxy.cpp
int32_t FaceAuthProxy::SetBufferProducer(sptr<IBufferProducer> &producer)
{
    MessageParcel data;
    MessageParcel reply;

    // 1. 写入接口令牌
    data.WriteInterfaceToken(FaceAuthProxy::GetDescriptor());

    // 2. 序列化远程对象
    if (producer != nullptr) {
        data.WriteRemoteObject(producer->AsObject());
    }

    // 3. 发送 IPC 请求
    SendRequest(CODE, data, reply);

    // 4. 读取返回结果
    return reply.ReadInt32();
}
```

## FaceAuthStub

**头文件**: `frameworks/ipc/inc/face_auth_stub.h`

**实现**: `frameworks/ipc/src/face_auth_stub.cpp`

**核心方法**:

```cpp
class FaceAuthStub : public IRemoteStub<IFaceAuth> {
private:
    // 处理 FACE_AUTH_SET_BUFFER_PRODUCER 请求
    int32_t FaceAuthSetBufferProducer(MessageParcel &data, MessageParcel &reply);

    // 消息分发
    int32_t OnRemoteRequest(uint32_t code, MessageParcel &data,
                           MessageParcel &reply, MessageOption &option) override;
};
```

**反序列化逻辑**:

```cpp
int32_t FaceAuthStub::FaceAuthSetBufferProducer(MessageParcel &data, MessageParcel &reply)
{
    // 1. 读取远程对象
    sptr<IRemoteObject> remoteObj = data.ReadRemoteObject();

    // 2. 转换为 IBufferProducer
    sptr<IBufferProducer> buffer = iface_cast<IBufferProducer>(remoteObj);

    // 3. 调用实际实现
    return SetBufferProducer(buffer);
}
```

## FaceAuthService

**头文件**: `services/inc/face_auth_service.h`

**实现**: `services/src/face_auth_service.cpp`

**类定义**:

```cpp
class FaceAuthService : public SystemAbility, public FaceAuthStub {
    DECLEAR_SYSTEM_ABILITY(FaceAuthService);

public:
    FaceAuthService();
    ~FaceAuthService() override = default;
    static std::shared_ptr<FaceAuthService> GetInstance();

    // IFaceAuth 接口实现
    int32_t SetBufferProducer(sptr<IBufferProducer> &producer) override;

protected:
    void OnStart() override;  // 注册到 SAMgr
    void OnStop() override;

private:
    static std::mutex mutex_;
    static std::shared_ptr<FaceAuthService> instance_;
    void StartDriverManager();
    bool IsPermissionGranted(const std::string &permission);
};
```

**SA ID**: 942 (`services/inc/face_auth_service.h:31`)

## FaceAuthDriverHdi

**头文件**: `services/inc/face_auth_driver_hdi.h`

**实现**: `services/src/face_auth_driver_hdi.cpp`

**职责**: HDI 接口封装，与设备厂商驱动交互

**依赖**: `drivers/interface/face_auth V2_0`

## 稳定性标注

| 模块 | 稳定性 | 依据 |
|------|--------|------|
| `faceauth_framework` | 稳定 | `innerapi_tags = ["platformsdk"]` |
| `FaceAuthClient` | 稳定 | platformsdk 接口 |
| `IFaceAuth` | 稳定 | 接口定义完整 |
| `FaceAuthProxy` | 稳定 | IPC 框架标准实现 |
| `FaceAuthStub` | 稳定 | IPC 框架标准实现 |
| `FaceAuthService` | 稳定 | SA 框架标准实现 |

## 可替换点

| 可替换组件 | 替换方式 | 风险 |
|-----------|----------|------|
| HDI 实现层 | 替换 `drivers/interface/face_auth` 厂商实现 | 低（接口稳定） |
| BufferProducer 来源 | 修改 SurfaceId 获取逻辑 | 中（依赖 graphic_surface） |
