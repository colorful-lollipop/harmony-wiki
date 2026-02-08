# 内部实现细节

**Distributed Data Manager Service 核心实现分析**

---

## 目录

- [核心类图](#核心类图)
- [关键数据结构](#关键数据结构)
- [Feature 生命周期](#feature-生命周期)
- [数据库句柄管理](#数据库句柄管理)
- [元数据管理](#元数据管理)
- [事件系统](#事件系统)
- [线程模型](#线程模型)
- [安全机制](#安全机制)

---

## 核心类图

### 整体类结构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Framework Layer                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                        FeatureSystem (Singleton)                     │  │
│  │  - RegisterCreator(name, creator, flag)                              │  │
│  │  - GetCreator(name) → shared_ptr<Feature>                            │  │
│  │  - RegisterStaticActs(name, acts)                                    │  │
│  └────────────────────┬─────────────────────────────────────────────────┘  │
│                       │                                                     │
│                       │ uses                                                │
│                       ▼                                                     │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  Feature (Abstract)                                                   │  │
│  │  ├─ OnRemoteRequest(code, data, reply)                               │  │
│  │  ├─ OnInitialize() / OnBind() / OnAppExit()                          │  │
│  │  ├─ OnUserChange() / Online() / Offline()                            │  │
│  │  └─ OnScreenUnlocked()                                               │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                        AutoCache (Singleton)                         │  │
│  │  - GetDBStore(meta, watchers) → pair<status, store>                  │  │
│  │  - CloseStore(storeId)                                               │  │
│  │  - RegCreator(creator)                                               │  │
│  └────────────────────┬─────────────────────────────────────────────────┘  │
│                       │                                                     │
│                       ▼                                                     │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  GeneralStore (Abstract)                                              │  │
│  │  ├─ Insert/Replace/Update/Delete                                      │  │
│  │  ├─ Query() → Cursor                                                  │  │
│  │  ├─ Sync(devices, mode, callback)                                     │  │
│  │  ├─ Watch/Unwatch                                                     │  │
│  │  └─ Execute(sql)                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                     MetaDataManager (Singleton)                      │  │
│  │  - SaveMeta(key, value)                                               │  │
│  │  - LoadMeta(key, value)                                               │  │
│  │  - DelMeta(key)                                                       │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                      EventCenter (Singleton)                         │  │
│  │  - Subscribe(evtId, handler)                                          │  │
│  │  - PostEvent(event)                                                   │  │
│  │  - Unsubscribe(evtId)                                                 │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ implements
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Service Layer                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐       │
│  │ RdbFeature   │ │KvDbFeature   │ │CloudFeature  │ │...           │       │
│  │ extends      │ │extends       │ │extends       │ │              │       │
│  │ Feature      │ │Feature       │ │Feature       │ │              │       │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────────────┘       │
│         │                │                │                                │
│         └────────────────┴────────────────┘                                │
│                          │                                                  │
│                          ▼                                                  │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                     RdbGeneralStore / KvDbGeneralStore               │  │
│  │                     extends GeneralStore                             │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### IPC 类层次

```
IRemoteBroker (OpenHarmony IPC 基类)
    │
    ├─ FeatureStub
    │       └── FeatureStubImpl : IRemoteStub<FeatureStub>
    │               └── wraps Feature
    │
    ├─ IRdbService
    │       └── RdbServiceStub : IRemoteStub<IRdbService>
    │
    ├─ IKVDBService
    │       └── KVDBServiceStub : IRemoteStub<IKVDBService>
    │
    └─ ... (其他 Feature 接口)
```

---

## 关键数据结构

### StoreMetaData

**文件**: `services/distributeddataservice/framework/include/metadata/store_meta_data.h`

```cpp
struct StoreMetaData : public Serializable {
    // 身份标识
    std::string appId;           // 应用唯一标识
    std::string bundleName;      // Bundle 名称
    std::string storeId;         // 存储 ID
    int32_t user;                // 用户 ID
    std::string deviceId;        // 设备 ID
    int32_t instanceId;          // 实例 ID (多开)
    
    // 存储类型
    int32_t storeType;           // 存储类型 (KV/RDB)
    int32_t securityLevel;       // 安全等级 (S0-S4)
    int32_t area;                // 数据分区 (EL0-EL5)
    
    // 功能开关
    bool isAutoSync;             // 自动同步
    bool isBackup;               // 备份
    bool isEncrypt;              // 加密
    bool enableCloud;            // 云同步
    bool cloudAutoSync;          // 自动云同步
    
    // 路径
    std::string dataDir;         // 数据目录
    std::string schema;          // Schema 定义
    
    // 时间戳
    int64_t createTime;
    int64_t modifyTime;
    
    // 序列化
    bool Marshal(json &node) const override;
    bool Unmarshal(const json &node) override;
    std::string GetKey() const;
};
```

### SecretKeyMetaData

```cpp
struct SecretKeyMetaData : public Serializable {
    int32_t time;           // 密钥版本时间
    std::vector<uint8_t> sKey;   // 加密密钥
    std::vector<uint8_t> nonce;  // 随机数
    
    std::string GetKey(const std::string &storeKey) const;
};
```

### StoreInfo

```cpp
struct StoreInfo {
    uint32_t tokenId;           // 访问令牌 ID
    std::string bundleName;     // Bundle 名
    std::string storeName;      // 存储名
    int32_t instanceId;         // 实例 ID
    int32_t user;               // 用户
    std::string deviceId;       // 设备
};
```

---

## Feature 生命周期

### 注册阶段

```cpp
// 1. 在静态初始化时注册
// services/distributeddataservice/service/rdb/rdb_service_impl.cpp

__attribute__((used)) static bool RdbFeatureRegistered = []() {
    FeatureSystem::GetInstance().RegisterCreator(
        "relational_store",  // Feature 名称
        []() { 
            return std::shared_ptr<Feature>(new RdbFeature());
        },
        FeatureSystem::BIND_NOW  // 或 BIND_LAZY
    );
    return true;
}();
```

### 创建阶段

```cpp
// 2. App 层获取 Feature 时创建
// services/distributeddataservice/app/src/kvstore_data_service.cpp:165

sptr<IRemoteObject> KvStoreDataService::GetFeatureInterface(
    const std::string &name) {
    
    // 从 FeatureSystem 获取创建器
    auto creator = FeatureSystem::GetInstance().GetCreator(name);
    
    // 创建 Feature 实例
    auto feature = creator();
    
    // 包装在 FeatureStubImpl 中
    sptr<FeatureStubImpl> stub = new FeatureStubImpl(feature);
    
    // 初始化
    feature->OnInitialize(executors_);
    
    return stub;
}
```

### 生命周期回调

```
Create() 
    ↓
OnInitialize()      // 初始化 Feature
    ↓
OnBind(info)        // 绑定客户端
    ↓
    ├─ [业务运行] 
    │     ├─ OnRemoteRequest()  // 处理 RPC 请求
    │     ├─ OnAppExit()        // 应用退出
    │     ├─ OnUserChange()     // 用户切换
    │     ├─ Online/Offline()   // 设备上线/下线
    │     └─ OnScreenUnlocked() // 屏幕解锁
    ↓
OnUnbind()          // 解绑客户端
    ↓
OnRelease()         // 释放资源
```

### 代码示例

```cpp
class RdbFeature : public FeatureSystem::Feature {
public:
    int32_t OnInitialize() override {
        // 初始化 RDB 相关资源
        // 注册到 AutoCache
        // 初始化同步管理器
        return E_OK;
    }
    
    int32_t OnBind(const BindInfo &info) override {
        // 保存客户端信息
        selfName_ = info.selfName;
        selfTokenId_ = info.selfTokenId;
        executors_ = info.executors;
        return E_OK;
    }
    
    int32_t OnAppExit(pid_t uid, pid_t pid, uint32_t tokenId, 
                      const std::string &bundleName) override {
        // 清理该应用相关的资源
        // 关闭打开的数据库
        return E_OK;
    }
    
    int32_t OnUserChange(uint32_t code, const std::string &user, 
                         const std::string &account) override {
        // 处理用户切换
        // 可能关闭旧用户的数据库
        return E_OK;
    }
    
    int32_t Online(const std::string &device) override {
        // 设备上线，准备同步
        return E_OK;
    }
    
    int32_t Offline(const std::string &device) override {
        // 设备下线，清理相关状态
        return E_OK;
    }
};
```

---

## 数据库句柄管理

### AutoCache 设计

**核心思想**: 服务器端数据库句柄的缓存和生命周期管理

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              AutoCache                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                         Cache Map                                    │  │
│  │  Key: StoreMetaData → Value: shared_ptr<Delegate>                    │  │
│  │                                                                      │  │
│  │  Store A ────→ Delegate A ────→ shared_ptr<GeneralStore>            │  │
│  │  Store B ────→ Delegate B ────→ shared_ptr<GeneralStore>            │  │
│  │  Store C ────→ Delegate C ────→ shared_ptr<GeneralStore>            │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                      Garbage Collector                               │  │
│  │  - 定时扫描未使用的句柄                                               │  │
│  │  - 关闭超时未访问的数据库                                             │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 使用模式

```cpp
// ✅ 正确用法: 获取句柄，使用后立即释放
void ProcessData(const StoreMetaData &meta) {
    auto &cache = AutoCache::GetInstance();
    
    // 获取句柄
    auto [status, store] = cache.GetDBStore(meta, watchers);
    if (status != E_OK) {
        return;
    }
    
    // 使用句柄
    store->Insert("table", value);
    
    // 句柄在作用域结束时自动释放
    // 不要长期持有!
}

// ❌ 错误用法: 长期持有句柄
class BadService {
    std::shared_ptr<GeneralStore> store_;  // 不要这样做!
    
public:
    void Init() {
        auto &cache = AutoCache::GetInstance();
        store_ = cache.GetDBStore(meta, {}).second;  // 危险!
    }
};
```

### 实现要点

```cpp
// services/distributeddataservice/framework/include/store/auto_cache.h
class API_EXPORT AutoCache {
public:
    static AutoCache &GetInstance();
    
    // 获取数据库句柄
    std::pair<int32_t, std::shared_ptr<GeneralStore>> 
    GetDBStore(const StoreMetaData &meta, 
               const std::vector<GeneralWatcher*> &watchers);
    
    // 关闭指定存储
    int32_t CloseStore(const std::string &storeId);
    
    // 注册存储创建器
    using Creator = std::function<std::shared_ptr<GeneralStore>(
        const StoreMetaData &, const std::vector<GeneralWatcher*> &)>;
    int32_t RegCreator(Creator creator);
    
    // 配置
    void Configure(const StoreConfig &config);
    
private:
    ConcurrentMap<std::string, std::shared_ptr<Delegate>> delegates_;
    std::atomic<bool> gcRunning_{false};
};
```

---

## 元数据管理

### MetaDataManager

**职责**: 统一管理所有元数据的持久化和缓存

```cpp
// services/distributeddataservice/framework/include/metadata/meta_data_manager.h
class API_EXPORT MetaDataManager {
public:
    static MetaDataManager &GetInstance();
    
    // 保存元数据
    template<typename T>
    bool SaveMeta(const std::string &key, const T &value, bool isLocal = false);
    
    // 加载元数据
    template<typename T>
    bool LoadMeta(const std::string &key, T &value, bool isLocal = false);
    
    // 删除元数据
    bool DelMeta(const std::string &key, bool isLocal = false);
    
    // 订阅变更
    using Observer = std::function<void(const std::string &key, 
                                          const std::string &value)>;
    bool Subscribe(const std::string &prefix, Observer observer);
    bool Unsubscribe(const std::string &prefix);
    
    // 同步元数据到设备
    int32_t Sync(const std::string &deviceId, const std::vector<std::string> &keys);
    
private:
    std::shared_ptr<DistributedDB> metaDB_;  // 底层元数据数据库
    ConcurrentMap<std::string, std::vector<Observer>> observers_;
};
```

### 元数据存储结构

```
Metadata DB (DistributedDB)
│
├─ /app/{bundleName}/{storeId}           → StoreMetaData
├─ /secret/{bundleName}/{storeId}        → SecretKeyMetaData
├─ /user/{userId}                        → UserMetaData
├─ /device/{deviceId}                    → DeviceMetaData
├─ /capability/{deviceId}                → CapMetaData
├─ /strategy/{bundleName}                → StrategyMetaData
└─ /switches/{bundleName}                → SwitchesMetaData
```

### 使用示例

```cpp
// 保存 StoreMetaData
StoreMetaData meta;
meta.bundleName = "com.example.app";
meta.storeId = "test_store";
meta.user = 100;
// ... 设置其他字段

std::string key = meta.GetKey();  // 生成唯一 key
MetaDataManager::GetInstance().SaveMeta(key, meta);

// 加载 StoreMetaData
StoreMetaData loaded;
if (MetaDataManager::GetInstance().LoadMeta(key, loaded)) {
    // 使用 loaded
}
```

---

## 事件系统

### EventCenter 架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            EventCenter                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                        Event Types                                   │  │
│  │  EVT_INITED      - 初始化完成                                         │  │
│  │  EVT_UPDATE      - 数据更新                                           │  │
│  │  EVT_CLOUD       - 云相关事件                                         │  │
│  │  EVT_BIND        - 绑定事件                                           │  │
│  │  EVT_CHANGE      - 数据变更                                           │  │
│  │  EVT_CUSTOM      - 自定义事件                                         │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                        Subscribers Map                               │  │
│  │  EventId → List<Observer>                                            │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 使用模式

```cpp
// 1. 订阅事件
auto &center = EventCenter::GetInstance();

center.Subscribe(Event::EVT_CLOUD, [](const Event &event) {
    auto &cloudEvent = static_cast<const CloudEvent &>(event);
    
    switch (cloudEvent.GetEventId()) {
        case CloudEvent::CLOUD_SYNC:
            // 处理云同步事件
            break;
        case CloudEvent::DATA_CHANGE:
            // 处理数据变更
            break;
    }
});

// 2. 发布事件
center.PostEvent(std::make_unique<CloudEvent>(
    CloudEvent::CLOUD_SYNC,
    bundleName,
    storeId
));

// 3. RAII 自动取消订阅
{
    auto defer = center.Subscribe(Event::EVT_CHANGE, handler);
    // 作用域结束时自动取消订阅
}
```

### CloudEvent 层次

```cpp
// services/distributeddataservice/framework/include/cloud/cloud_event.h
class CloudEvent : public Event {
public:
    enum EventId {
        FEATURE_INIT,       // Feature 初始化
        GET_SCHEMA,         // 获取 Schema
        LOCAL_CHANGE,       // 本地变更
        CLEAN_DATA,         // 清理数据
        CLOUD_SYNC,         // 云同步
        DATA_CHANGE,        // 数据变更
        CLOUD_SHARE,        // 云分享
    };
    
    EventId GetEventId() const;
    std::string GetBundleName() const;
    std::string GetStoreId() const;
    // ...
};

// 数据变更事件 (继承自 CloudEvent)
class DataChangeEvent : public CloudEvent {
public:
    struct EventInfo {
        std::string bundleName;
        std::string storeId;
        std::vector<std::string> devices;
        // ...
    };
    
    EventInfo GetEventInfo() const;
};
```

---

## 线程模型

### 线程池配置

```cpp
// services/distributeddataservice/service/bootstrap/src/bootstrap.cpp
bool Bootstrap::LoadThread() {
    // 从配置加载线程参数
    int ipcThreadNum = config["ipcThreadNum"].get<int>();
    int minThreadNum = config["minThreadNum"].get<int>();
    int maxThreadNum = config["maxThreadNum"].get<int>();
    
    // 创建线程池
    executors_ = std::make_shared<ExecutorPool>(
        minThreadNum, maxThreadNum
    );
    
    return true;
}
```

### 线程类型

| 线程类型 | 数量 | 用途 | 配置项 |
|---------|------|------|-------|
| **IPC 线程** | N | 处理 IPC 请求 | `ipcThreadNum` |
| **工作线程** | min-max | 业务逻辑处理 | `minThreadNum` / `maxThreadNum` |
| **定时器线程** | 1 | 备份/同步调度 | 内置 |
| **GC 线程** | 1 | AutoCache 垃圾回收 | 内置 |

### 线程安全组件

| 组件 | 线程安全机制 |
|-----|-------------|
| **AutoCache** | 内部使用 `ConcurrentMap` |
| **MetaDataManager** | 内部使用 `ConcurrentMap` |
| **EventCenter** | 订阅/发布使用锁保护 |
| **FeatureSystem** | `ConcurrentMap` 存储 creators |

### 并发集合

```cpp
// ConcurrentMap - 线程安全的 Map
ConcurrentMap<Key, Value> map;

// 基本操作
map.Insert(key, value);
map.Erase(key);
map.Find(key, value);
map.ForEach([](const auto &k, auto &v) { /* ... */ });

// 常用线程安全模式
std::shared_ptr<Value> value;
if (map.Find(key, value)) {
    // 使用 value
}
```

---

## 安全机制

### 权限检查链

```
IPC Request
    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. AccessTokenKit 检查                                                      │
│    - GetTokenTypeFlag(tokenId) → 获取 token 类型                            │
│    - GetHapTokenInfo(tokenId) → 获取 HAP 信息                               │
│    - VerifyAccessToken(tokenId, permission) → 验证权限                      │
└─────────────────────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ 2. BundleChecker 检查                                                       │
│    - 验证 bundleName 格式                                                   │
│    - 检查信任/不信任列表                                                    │
│    - 缓存验证结果                                                           │
└─────────────────────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ 3. PermissionValidator 检查                                                 │
│    - CheckSyncPermission() → DISTRIBUTED_DATASYNC                          │
│    - IsCloudConfigPermit() → CLOUDDATA_CONFIG                              │
└─────────────────────────────────────────────────────────────────────────────┘
    ↓
Business Logic
```

### 数据隔离

```cpp
// 数据隔离三元组
struct DataIsolation {
    int32_t user;           // 用户 ID
    std::string appId;      // 应用 ID
    std::string storeId;    // 存储 ID
};

// 访问控制检查
bool CheckAccess(const StoreMetaData &meta, uint32_t tokenId) {
    // 1. 获取调用者信息
    HapTokenInfo info;
    AccessTokenKit::GetHapTokenInfo(tokenId, info);
    
    // 2. 验证用户匹配
    if (info.userID != meta.user) {
        // 检查跨账户权限
        return VerifyAcrossAccountsPermission(tokenId);
    }
    
    // 3. 验证应用匹配
    if (info.bundleName != meta.bundleName) {
        // 检查数据共享权限
        return VerifyDataSharePermission(tokenId, meta);
    }
    
    return true;
}
```

### 加密管理

```cpp
// CryptoManager - 密钥管理
class CryptoManager {
public:
    // 生成密钥
    std::vector<uint8_t> GenerateKey(int32_t len);
    
    // 加密/解密
    std::vector<uint8_t> Encrypt(const std::vector<uint8_t> &key,
                                    const std::vector<uint8_t> &data);
    std::vector<uint8_t> Decrypt(const std::vector<uint8_t> &key,
                                    const std::vector<uint8_t> &data);
    
    // 密钥派生
    std::vector<uint8_t> DeriveKey(const std::string &password,
                                    const std::vector<uint8_t> &salt);
};
```

---

## 相关文档

- [架构设计](01_Architecture.md) - 整体架构
- [IPC 接口清单](04_Interface.md) - RPC 方法参考
- [攻击面分析](05_AttackSurface.md) - 安全分析

---

**代码位置**:
- Framework: `services/distributeddataservice/framework/include/`
- Service: `services/distributeddataservice/service/`
- App: `services/distributeddataservice/app/src/`
