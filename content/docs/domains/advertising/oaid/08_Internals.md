# OAID 内部实现细节

## 核心类职责

### 类关系图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        OAID 核心类架构                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    OAIDService                               │   │
│  │  [SystemAbility + OAIDServiceStub]                          │   │
│  │                                                              │   │
│  │  • 服务生命周期管理 (OnStart/OnStop)                         │   │
│  │  • OAID 生成与管理 (GetOAID/GainOAID/ResetOAID)              │   │
│  │  • KVStore 操作 (InitKvStore/Read/Write)                     │   │
│  │  • 内存缓存 (oaid_)                                          │   │
│  └──────────────────────────┬──────────────────────────────────┘   │
│                             │                                       │
│              ┌──────────────┼──────────────┐                       │
│              ▼              ▼              ▼                       │
│  ┌─────────────────┐ ┌─────────────┐ ┌─────────────────────┐      │
│  │  OAIDServiceStub │ │ ConnectAds  │ │ OaidObserverManager │      │
│  │                  │ │   Manager   │ │                     │      │
│  │  IPC 请求处理     │ │             │ │  观察者管理         │      │
│  │  权限校验         │ │  Ads 连接    │ │  配置变更通知       │      │
│  └─────────────────┘ └─────────────┘ └─────────────────────┘      │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    OAIDServiceClient                         │   │
│  │                                                              │   │
│  │  • 单例模式管理                                               │   │
│  │  • 服务发现与连接                                             │   │
│  │  • 死亡监听与恢复                                             │   │
│  │  • 客户端权限预检                                             │   │
│  └──────────────────────────┬──────────────────────────────────┘   │
│                             │                                       │
│                             ▼                                       │
│                    ┌─────────────────┐                             │
│                    │  OAIDServiceProxy │                            │
│                    │                   │                            │
│                    │  IPC 代理封装      │                            │
│                    │  SendRequest      │                            │
│                    └─────────────────┘                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 核心类详解

### OAIDService

**文件**: `services/oaid_manager/include/oaid_service.h`, `services/oaid_manager/src/oaid_service.cpp`

**职责**: 核心业务逻辑实现

#### 类定义

```cpp
class OAIDService : public SystemAbility, public OAIDServiceStub {
    DECLARE_SYSTEM_ABILITY(OAIDService);
public:
    DISALLOW_COPY_AND_MOVE(OAIDService);
    static sptr<OAIDService> GetInstance();
    
    // 业务接口
    std::string GetOAID() override;
    int32_t ResetOAID() override;
    
    // KVStore 管理
    bool InitKvStore(std::string storeIdStr);
    bool ReadValueFromKvStore(const std::string &key, std::string &value);
    bool WriteValueToKvStore(const std::string &key, const std::string &value);
    
protected:
    void OnStart() override;
    void OnStop() override;
    void OnAddSystemAbility(int32_t systemAbilityId, const std::string& deviceId) override;
    
private:
    ServiceRunningState state_;
    static std::mutex mutex_;
    static sptr<OAIDService> instance_;
    
    std::shared_ptr<DistributedKv::SingleKvStore> oaidKvStore_;
    std::shared_ptr<DistributedKv::SingleKvStore> oaidUnderAgeKvStore_;
    std::mutex updateMutex_;
    std::string oaid_;  // 内存缓存
};
```

#### 关键方法

| 方法 | 职责 | 行号 |
|------|------|------|
| `OnStart` | 服务启动，发布 SA | 123 |
| `OnStop` | 服务停止 | 153 |
| `Init` | 初始化，发布服务 | 140 |
| `GetOAID` | 对外接口，返回 OAID | 287 |
| `GainOAID` | 核心逻辑，生成/获取 OAID | 238 |
| `ResetOAID` | 重置 OAID | 295 |
| `InitKvStore` | 初始化 KVStore | 327 |
| `OnAddSystemAbility` | 监听 SA 启动事件 | 163 |

---

### OAIDServiceStub

**文件**: `services/oaid_manager/include/oaid_service_stub.h`, `services/oaid_manager/src/oaid_service_stub.cpp`

**职责**: IPC 请求处理与权限校验

#### 类定义

```cpp
class OAIDServiceStub : public IRemoteStub<IOAIDService> {
public:
    OAIDServiceStub();
    virtual ~OAIDServiceStub();
    
    int32_t OnRemoteRequest(
        uint32_t code, 
        MessageParcel &data, 
        MessageParcel &reply, 
        MessageOption &option
    ) override;
    
protected:
    // 业务处理
    int32_t OnGetOAID(MessageParcel &data, MessageParcel &reply);
    int32_t OnResetOAID(MessageParcel &data, MessageParcel &reply);
    
    // 权限校验
    bool CheckPermission(const std::string &permissionName);
    bool CheckSystemApp();
    int32_t ValidateResetOAIDPermission(std::string bundleName, MessageParcel &reply);
    
    // 服务管理
    void ExitIdleState();
    void PostDelayUnloadTask();
    
private:
    std::shared_ptr<AppExecFwk::EventHandler> unloadHandler_;
    std::mutex init_eventHandler_Mutex_;
};
```

#### 权限校验流程

```
OnRemoteRequest
    │
    ├──► GetCallingUid() → GetBundleNameByUid()
    │
    ├──► [GET_OAID] CheckPermission(APP_TRACKING_CONSENT)
    │
    ├──► [RESET_OAID] ValidateResetOAIDPermission()
    │       │
    │       ├──► LoadAndCheckOaidTrustList() → 检查白名单
    │       └──► CheckSystemApp() → 检查系统应用
    │
    ├──► Verify InterfaceToken
    │
    └──► SendCode() → 分发到业务处理
```

---

### OAIDServiceClient

**文件**: `interfaces/innerkits/include/oaid_service_client.h`, `interfaces/innerkits/src/oaid_service_client.cpp`

**职责**: 客户端封装，单例管理

#### 类定义

```cpp
class OAIDServiceClient : public RefBase {
public:
    static sptr<OAIDServiceClient> GetInstance();
    
    // 业务接口
    std::string GetOAID();
    int32_t ResetOAID();
    int32_t RegisterObserver(const sptr<IRemoteConfigObserver>& observer);
    
    // 连接管理
    bool LoadService();
    void OnRemoteSaDied(const wptr<IRemoteObject>& remote);
    
private:
    bool CheckPermission(const std::string &permissionName);
    void LoadServerSuccess(const sptr<IRemoteObject>& remoteObject);
    void LoadServerFail();
    
    static std::mutex instanceLock_;
    static sptr<OAIDServiceClient> instance_;
    
    std::mutex getOaidProxyMutex_;
    sptr<IOAIDService> oaidServiceProxy_;
    
    std::mutex loadServiceLock_;
    std::condition_variable loadServiceCondition_;
    bool loadServiceReady_ = false;
    
    sptr<OAIDSaDeathRecipient> deathRecipient_;
};
```

#### 服务加载流程

```
GetOAID/ResetOAID
    │
    ├──► LoadService()
    │       │
    │       ├──► SystemAbilityManager::LoadSystemAbility(6101, callback)
    │       │
    │       ├──► 等待回调 (condition_variable, 10s 超时)
    │       │
    │       └──► 收到 OnLoadSystemAbilitySuccess()
    │               │
    │               └──► LoadServerSuccess()
    │                       │
    │                       ├──► AddDeathRecipient()
    │                       └──► iface_cast<IOAIDService>()
    │
    └──► 使用 oaidServiceProxy_ 调用 IPC
```

---

### ConnectAdsManager

**文件**: `services/oaid_manager/include/connect_ads_stub.h`, `services/oaid_manager/src/connect_ads_stub.cpp`

**职责**: 连接 Ads Service，管理未成年人检查

#### 类定义

```cpp
class ConnectAdsManager {
public:
    static ConnectAdsManager* GetInstance();
    
    // 检查是否允许获取 OAID
    bool checkAllowGetOaid();
    
    // 通知 Ads Service
    void notifyKit(int32_t code);
    
    // 获取连接
    sptr<ConnectAdsStub> getConnection();
    
private:
    ConnectAdsManager();
    ~ConnectAdsManager();
    
    Want getWantInfo();  // 从配置读取
    int32_t DisconnectService();
    
    sptr<ConnectAdsStub> connectObject_;
    std::mutex connectMutex_;
};

class ConnectAdsStub : public AbilityConnectionStub {
public:
    void OnAbilityConnectDone(const ElementName &element, 
                              const sptr<IRemoteObject> &remoteObject, 
                              int resultCode) override;
    void OnAbilityDisconnectDone(const ElementName &element, int resultCode) override;
    
    void SendMessage(int32_t code);
    void AddMessageToQueue(int32_t code);
    void ProcessMessageQueue();
    
private:
    enum class ConnectionState { DISCONNECTED, CONNECTING, CONNECTED };
    
    ConnectionState connectionState_;
    std::mutex stateMutex_;
    
    std::queue<int32_t> messageQueue_;
    std::unordered_set<int32_t> messageSet_;
    std::mutex queueMutex_;
    
    sptr<IRemoteObject> proxy_;
    std::mutex proxyMutex_;
};
```

#### 连接管理流程

```
checkAllowGetOaid()
    │
    ├──► 从 KVStore 读取 allowGetOaid 和 updateTime
    │
    ├──► 检查时间戳是否过期 (6小时)
    │
    ├──► [过期] notifyKit(GET_ALLOW_OAID_CODE)
    │       │
    │       ├──► 检查连接状态
    │       ├──► [未连接] ConnectServiceExtensionAbility()
    │       └──► 发送消息队列
    │
    └──► 返回允许状态
```

---

## 资源生命周期

### OAID 生命周期

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ 生成     │────►│ 存储     │────►│ 使用     │────►│ 重置     │
│          │     │          │     │          │     │          │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                │                │                │
     ▼                ▼                ▼                ▼
  GetUUID()      KVStore Put    应用层读取      GetUUID()
  (OpenSSL)      加密存储       内存缓存        KVStore Put
                                                      │
                                                      ▼
                                                通知观察者
```

### 服务对象生命周期

| 对象 | 创建时机 | 销毁时机 | 生命周期管理 |
|------|---------|---------|-------------|
| `OAIDService` | 系统启动/按需 | 系统关闭/空闲超时 | SystemAbility 框架 |
| `OAIDServiceClient` | 首次调用 | 进程结束 | 单例模式 |
| `ConnectAdsManager` | 首次使用 | 系统关闭 | 单例模式 |
| `KVStore` | OnAddSystemAbility | OnStop | shared_ptr |

---

## 内部 API 契约

### 稳定接口

以下接口向后兼容，可安全使用：

| 接口 | 位置 | 稳定性 |
|------|------|--------|
| `OAIDServiceClient::GetOAID()` | `interfaces/innerkits/` | 稳定 |
| `OAIDServiceClient::ResetOAID()` | `interfaces/innerkits/` | 稳定 |
| `IOAIDService` | `interfaces/innerkits/include/oaid_service_interface.h` | 稳定 |

### 内部实现细节

以下实现可能变更，不应直接依赖：

| 实现 | 位置 | 说明 |
|------|------|------|
| `GainOAID()` 内部逻辑 | `oaid_service.cpp` | 实现细节 |
| KVStore 配置 | `oaid_service.cpp:getOptions()` | 可能调整 |
| 延迟卸载时间 | `oaid_service_stub.cpp:DELAY_TIME` | 可能调整 |

---

## 关键算法

### UUID v4 生成

```cpp
// services/oaid_manager/src/oaid_service.cpp:50-93
std::string GetUUID()
{
    // 生成 16 字节随机数
    unsigned char uuid[16] = {0};
    RAND_bytes(uuid, sizeof(uuid));
    
    // UUID v4 格式设置
    // xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx
    // M = 4 (版本)
    // N = 8,9,a,b (变体)
    uuid[6] = (uuid[6] & 0x0F) | 0x40;  // 版本 4
    uuid[8] = (uuid[8] & 0x3F) | 0x80;  // 变体 RFC 4122
    
    // 格式化为字符串
    std::string formatUuid = "";
    for (size_t i = 0; i < sizeof(uuid); i++) {
        if (i >= 4 && i <= 10 && i % 2 == 0) {
            formatUuid += "-";
        }
        formatUuid += HexToChar(uuid[i] >> 4);
        formatUuid += HexToChar(uuid[i] & 0x0F);
    }
    return formatUuid;
}
```

### 缓存策略

```cpp
// OAID 双重缓存
class OAIDService {
    std::string oaid_;  // 内存缓存
    // 持久化存储在 KVStore
};

// 读取策略
std::string OAIDService::GainOAID()
{
    // 1. 检查更新文件
    if (IsFileExsit(OAID_UPDATE)) {
        // 读取新 OAID 并更新缓存和 KVStore
    }
    
    // 2. 检查内存缓存
    if (!oaid_.empty()) {
        return oaid_;
    }
    
    // 3. 读取 KVStore
    ReadValueFromKvStore(OAID_KVSTORE_KEY, oaidKvStoreStr);
    
    // 4. 生成新 OAID（首次）
    if (oaidKvStoreStr empty) {
        oaid_ = GetUUID();
        WriteValueToKvStore(OAID_KVSTORE_KEY, oaid_);
    }
    
    return oaid_;
}
```

---

## 性能优化

### 延迟卸载机制

```cpp
void OAIDServiceStub::PostDelayUnloadTask()
{
    // 延迟 290 秒后卸载
    const int32_t DELAY_TIME = 290000;  // ms
    
    unloadHandler_->PostTask([this]() {
        samgrProxy->UnloadSystemAbility(OAID_SYSTME_ID);
    }, TASK_ID, DELAY_TIME);
}
```

### 异步处理

```cpp
// N-API 异步工作
void GetOAIDExecuteCallBack(napi_env env, void *data)
{
    AsyncCallbackInfoOAID *info = (AsyncCallbackInfoOAID *)data;
    // 在 LibUV 线程池执行
    info->oaid = OAIDServiceClient::GetInstance()->GetOAID();
}

void GetOAIDCompleteCallBack(napi_env env, napi_status status, void *data)
{
    // 在主线程回调 JS
    ReturnCallbackPromise(env, info, result);
}
```

---

## 相关文档

- [项目概览](01_Overview.md)
- [架构分析](02_Architecture.md)
- [代码地图](03_CodeMap.md)
- [构建配置](07_Build.md)
