# SecurityGuard SA 服务详解

**文档版本**：3.1.0  
**最后更新**：2026-02-06  
**维护者**：SecurityGuard Team

---

## 1. 概述

本文档详细描述 SecurityGuard 的三个核心系统能力（SA）服务，包括服务接口定义、实现细节、IPC 通信机制以及服务间依赖关系。所有关键结论均可追溯到代码证据。

**证据来源**：`services/*/include/*.h`、`interfaces/inner_api/*/*.h`、`sa_profile/*.json`

---

## 2. SA 服务概览

### 2.1 系统能力列表

| SA ID | 服务名称 | 进程 | 运行模式 | 主要功能 |
|-------|---------|------|---------|---------|
| 3523 | RiskAnalysisManager | security_guard | OnDemand | 安全模型分析、风险分类 |
| 3524 | DataCollectManager | security_guard | OnDemand | 事件上报、查询、订阅管理 |
| 3525 | SecurityCollectorManager | security_collector | OnDemand | 采集器生命周期管理、事件采集 |

**证据来源**：`sa_profile/3523.json`、`sa_profile/3524.json`、`sa_profile/3525.json`

### 2.2 SA 配置文件内容

**证据来源**：`sa_profile/3523.json`

```json
{
  "services": [{
    "name": "risk_analysis_manager",
    "path": [
      "/system/bin/sa_main",
      "/system/profile/risk_analysis_manager.json"
    ],
    "uid": "security_guard",
    "gid": ["security_guard", "shell"],
    "apl": "system_basic",
    "secon": "u:r:security_guard:s0",
    "permission": [],
    "permission_acls": [],
    "run-on-create": false,
    "start-mode": "condition"
  }]
}
```

**证据来源**：`sa_profile/3524.json`

```json
{
  "services": [{
    "name": "data_collect_manager",
    "path": [
      "/system/bin/sa_main",
      "/system/profile/data_collect_manager.json"
    ],
    "uid": "security_guard",
    "gid": ["security_guard", "shell"],
    "apl": "system_basic",
    "secon": "u:r:security_guard:s0",
    "permission": [
      "ohos.permission.COLLECT_SECURITY_EVENT",
      "ohos.permission.QUERY_SECURITY_EVENT"
    ],
    "permission_acls": [
      "ohos.permission.COLLECT_SECURITY_EVENT"
    ],
    "run-on-create": false,
    "start-mode": "condition"
  }]
}
```

**证据来源**：`sa_profile/3525.json`

```json
{
  "services": [{
    "name": "security_collector_manager",
    "path": [
      "/system/bin/sa_main",
      "/system/profile/security_collector_manager.json"
    ],
    "uid": "security_collector",
    "gid": ["security_collector", "shell"],
    "apl": "system_basic",
    "secon": "u:r:security_collector:s0",
    "permission": [
      "ohos.permission.COLLECT_SECURITY_EVENT"
    ],
    "permission_acls": [
      "ohos.permission.COLLECT_SECURITY_EVENT"
    ],
    "run-on-create": false,
    "start-mode": "condition"
  }]
}
```

---

## 3. RiskAnalysisManager 服务（SA 3523）

### 3.1 服务概述

| 属性 | 值 |
|------|-----|
| 服务名称 | RiskAnalysisManager |
| SA ID | 3523 |
| 运行进程 | security_guard |
| 主要功能 | 安全模型执行、风险分析结果返回 |
| 代码位置 | services/risk_classify/ |

### 3.2 服务接口定义

**证据来源**：`interfaces/inner_api/classify/include/i_risk_analysis_manager.h`

```cpp
// IPC 命令码定义
enum class RiskAnalysisManagerInterfaceCode {
    CMD_GET_SECURITY_MODEL_RESULT = 2,  // 获取模型结果
    CMD_SET_MODEL_STATE = 3,             // 设置模型状态
    CMD_START_MODEL = 4,                  // 启动模型
};

// 服务接口
class IRiskAnalysisManager : public IRemoteBroker {
public:
    // 请求安全模型结果
    virtual int32_t RequestSecurityModelResult(
        const std::string &devId, 
        uint32_t modelId,
        const std::string &param, 
        const sptr<IRemoteObject> &callback) = 0;

    // 设置模型状态
    virtual int32_t SetModelState(uint32_t modelId, bool enable) = 0;

    // 启动安全模型
    virtual int32_t StartSecurityModel(uint32_t modelId, const std::string &param) = 0;
};

// 回调接口
class IRiskAnalysisManagerCallback : public IRemoteBroker {
public:
    // 返回模型结果
    virtual int32_t ResponseSecurityModelResult(
        const std::string &devId, 
        uint32_t modelId, 
        std::string &result) = 0;
};
```

### 3.3 服务实现类

**证据来源**：`services/risk_classify/include/risk_analysis_manager_service.h:26-44`

```cpp
class RiskAnalysisManagerService : public SystemAbility, 
    public RiskAnalysisManagerStub, public NoCopyable {
    DECLARE_SYSTEM_ABILITY(RiskAnalysisManagerService);

public:
    RiskAnalysisManagerService(int32_t saId, bool runOnCreate);
    ~RiskAnalysisManagerService() override = default;
    
    // 系统能力生命周期
    void OnStart() override;
    void OnStop() override;
    
    // 核心接口实现
    ErrCode RequestSecurityModelResult(
        const std::string &devId, 
        uint32_t modelId,
        const std::string &param, 
        const sptr<IRemoteObject> &cb) override;
    
    ErrCode SetModelState(uint32_t modelId, bool enable) override;
    ErrCode StartSecurityModel(uint32_t modelId, const std::string &param) override;
    
    // 系统能力回调
    void OnAddSystemAbility(int32_t systemAbilityId, const std::string& deviceId) override;
    void OnRemoveSystemAbility(int32_t systemAbilityId, const std::string& deviceId) override;

private:
    // 异步任务调度
    void PushRiskAnalysisTask(uint32_t modelId, std::string param, 
        std::shared_ptr<std::promise<std::string>> promise);
    
    // 权限检查
    int32_t IsApiHasPermission(const std::string &api);
};
```

### 3.4 支持的安全模型

**证据来源**：`frameworks/js/napi/security_guard_napi.h:125-132`

```cpp
enum ModelIdType {
    ROOT_SCAN_MODEL_ID = 3001000000,           // 越狱检测模型
    DEVICE_COMPLETENESS_MODEL_ID = 3001000001, // 设备完整性检测模型
    PHYSICAL_MACHINE_DETECTION_MODEL_ID = 3001000002, // 物理机检测模型
    SECURITY_RISK_FACTOR_MODEL_ID = 3001000009, // 安全风险因子检测模型
    WLAN_RISK_DETECTION_MODEL_ID = 3001000011,  // WLAN 风险检测模型
};
```

### 3.5 服务依赖

| 依赖项 | 类型 | 说明 |
|--------|------|------|
| sg_config_manager | 服务 | 配置管理 |
| sg_collect_service | 服务 | 数据收集 |
| sg_model_manager_stamp | 静态库 | 模型管理 |
| sg_bigdata_stamp | 静态库 | 大数据上报 |

**证据来源**：`services/risk_classify/BUILD.gn`

---

## 4. DataCollectManager 服务（SA 3524）

### 4.1 服务概述

| 属性 | 值 |
|------|-----|
| 服务名称 | DataCollectManager |
| SA ID | 3524 |
| 运行进程 | security_guard |
| 主要功能 | 事件上报、查询、订阅管理 |
| 代码位置 | services/data_collect/ |

### 4.2 服务接口定义

**证据来源**：`interfaces/inner_api/collect/include/i_data_collect_manager.h`

```cpp
// IPC 命令码定义
enum class DataCollectManagerInterfaceCode {
    CMD_DATA_COLLECT = 1,              // 数据采集
    CMD_DATA_REQUEST = 2,               // 数据请求
    CMD_DATA_SUBSCRIBE = 3,             // 数据订阅
    CMD_DATA_UNSUBSCRIBE = 4,           // 取消订阅
    CMD_SECURITY_EVENT_QUERY = 5,       // 安全事件查询
    CMD_SECURITY_COLLECTOR_START = 6,  // 采集器启动
    CMD_SECURITY_COLLECTOR_STOP = 7,    // 采集器停止
    CMD_SECURITY_CONFIG_UPDATE = 8,     // 配置更新
    CMD_SECURITY_EVENT_CONFIG_QUERY = 9,// 事件配置查询
    CMD_SECURITY_EVENT_MUTE = 10,      // 事件静音
    CMD_SECURITY_EVENT_UNMUTE = 11,    // 取消静音
    CMD_SECURITY_EVENT_QUERY_BY_ID = 12,// 按 ID 查询
};

// 服务接口
class IDataCollectManager : public IRemoteBroker {
public:
    // 同步数据上报
    virtual int32_t RequestDataSubmit(
        int64_t eventId, 
        std::string &version, 
        std::string &time,
        std::string &content, 
        bool isSync = true) = 0;
    
    // 异步数据上报
    virtual int32_t RequestDataSubmitAsync(
        int64_t eventId, 
        std::string &version, 
        std::string &time,
        std::string &content) = 0;

    // 风险数据请求
    virtual int32_t RequestRiskData(
        std::string &devId, 
        std::string &eventList,
        const sptr<IRemoteObject> &callback) = 0;

    // 事件订阅
    virtual int32_t Subscribe(
        const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb, 
        const std::string &clientId) = 0;

    // 取消订阅
    virtual int32_t Unsubscribe(
        const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb, 
        const std::string &clientId) = 0;

    // 安全事件查询
    virtual int32_t QuerySecurityEvent(
        const std::vector<SecurityCollector::SecurityEventRuler> &rulers,
        const sptr<IRemoteObject> &cb, 
        const std::string &eventGroup) = 0;

    // 采集器启动
    virtual int32_t CollectorStart(
        const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) = 0;

    // 采集器停止
    virtual int32_t CollectorStop(
        const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) = 0;

    // 配置更新
    virtual int32_t ConfigUpdate(int fd, const std::string& name) = 0;

    // 事件过滤
    virtual int32_t AddFilter(
        const SecurityEventFilter &subscribeMute, 
        const std::string &clientId) = 0;
    
    virtual int32_t RemoveFilter(
        const SecurityEventFilter &subscribeMute, 
        const std::string &clientId) = 0;
};
```

### 4.3 服务实现类

**证据来源**：`services/data_collect/sa/include/data_collect_manager_service.h:32-98`

```cpp
class DataCollectManagerService : public SystemAbility, 
    public DataCollectManagerIdlStub, public NoCopyable {
    DECLARE_SYSTEM_ABILITY(DataCollectManagerService);

public:
    DataCollectManagerService(int32_t saId, bool runOnCreate);
    ~DataCollectManagerService() override = default;
    
    // 系统能力生命周期
    void OnStart() override;
    void OnStop() override;
    int Dump(int fd, const std::vector<std::u16string>& args) override;
    
    // 核心接口实现
    ErrCode RequestDataSubmit(int64_t eventId, const std::string &version, 
        const std::string &time, const std::string &content) override;
    
    ErrCode RequestDataSubmitAsync(int64_t eventId, const std::string &version, 
        const std::string &time, const std::string &content) override;
    
    ErrCode RequestRiskData(const std::string &devId, const std::string &eventList,
        const sptr<IRemoteObject> &cb) override;
    
    ErrCode Subscribe(const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb, const std::string &clientId) override;
    
    ErrCode Unsubscribe(const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb, const std::string &clientId) override;
    
    ErrCode QuerySecurityEvent(
        const std::vector<SecurityCollector::SecurityEventRuler> &rulers,
        const sptr<IRemoteObject> &cb, const std::string &eventGroup) override;
    
    ErrCode CollectorStart(const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) override;
    
    ErrCode CollectorStop(const SecurityCollector::SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) override;
    
    ErrCode ConfigUpdate(int fd, const std::string& name) override;
    
    ErrCode QuerySecurityEventConfig(std::string &result) override;
    
    ErrCode AddFilter(const SecurityEventFilter &subscribeMute, 
        const std::string &clientId) override;
    
    ErrCode RemoveFilter(const SecurityEventFilter &subscribeMute, 
        const std::string &clientId) override;

private:
    // 权限检查
    int32_t IsApiHasPermission(const std::string &api);
    int32_t IsEventGroupHasPermission(const std::string &eventGroup, 
        std::vector<int64_t> eventIds);
    int32_t IsEventGroupHasPublicPermission(const std::string &eventGroup, 
        std::vector<int64_t> eventIds);
    
    // 资源管理
    std::mutex mutex_{};
    sptr<IRemoteObject::DeathRecipient> deathRecipient_{};
    std::map<std::string, sptr<IRemoteObject>> clientCallBacks_{};
    std::atomic<int32_t> tokenBucket_{};  // Token Bucket 限流
};
```

### 4.4 服务依赖

| 依赖项 | 类型 | 说明 |
|--------|------|------|
| libsg_collector_sdk | SDK | 采集器 SDK |
| security_collector_manager | 服务 | 采集器管理 |
| sg_config_manager | 服务 | 配置管理 |
| sg_collect_service_database | 服务 | 数据库服务 |
| sg_model_manager_stamp | 静态库 | 模型管理 |
| sg_bigdata_stamp | 静态库 | 大数据上报 |

**证据来源**：`services/data_collect/BUILD.gn`

---

## 5. SecurityCollectorManager 服务（SA 3525）

### 5.1 服务概述

| 属性 | 值 |
|------|-----|
| 服务名称 | SecurityCollectorManager |
| SA ID | 3525 |
| 运行进程 | security_collector |
| 主要功能 | 采集器生命周期管理、事件采集 |
| 代码位置 | services/security_collector/ |

### 5.2 服务接口定义

**证据来源**：`interfaces/inner_api/collector/include/i_security_collector_manager.h`

```cpp
// IPC 命令码定义
enum class SecurityCollectManagerInterfaceCode {
    CMD_COLLECTOR_SUBCRIBE = 1,        // 采集器订阅
    CMD_COLLECTOR_UNSUBCRIBE = 2,       // 取消订阅
    CMD_COLLECTOR_START = 3,            // 启动采集器
    CMD_COLLECTOR_STOP = 4,             // 停止采集器
    CMD_SECURITY_EVENT_QUERY = 5,        // 事件查询
    CMD_SECURITY_EVENT_MUTE = 6,        // 静音
    CMD_SECURITY_EVENT_UNMUTE = 7,      // 取消静音
};

// 服务接口
class ISecurityCollectorManager : public IRemoteBroker {
public:
    // 订阅采集器
    virtual int32_t Subscribe(
        const SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &callback) = 0;

    // 取消订阅
    virtual int32_t Unsubscribe(const sptr<IRemoteObject> &callback) = 0;

    // 启动采集器
    virtual int32_t CollectorStart(
        const SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) = 0;

    // 停止采集器
    virtual int32_t CollectorStop(
        const SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) = 0;

    // 查询安全事件
    virtual int32_t QuerySecurityEvent(
        const std::vector<SecurityCollector::SecurityEventRuler> &rulers,
        const sptr<IRemoteObject> &cb) = 0;

    // 添加过滤器
    virtual int32_t AddFilter(
        const SecurityEventFilter &filter,
        const sptr<IRemoteObject> &callback) = 0;

    // 移除过滤器
    virtual int32_t RemoveFilter(
        const SecurityEventFilter &filter,
        const sptr<IRemoteObject> &callback) = 0;
};
```

### 5.3 服务实现类

**证据来源**：`services/security_collector/include/security_collector_manager_service.h`

```cpp
class SecurityCollectorManagerService : public SystemAbility, 
    public SecurityCollectorManagerStub, public NoCopyable {
    DECLARE_SYSTEM_ABILITY(SecurityCollectorManagerService);

public:
    SecurityCollectorManagerService(int32_t saId, bool runOnCreate);
    ~SecurityCollectorManagerService() override = default;
    
    // 系统能力生命周期
    void OnStart() override;
    void OnStop() override;
    int Dump(int fd, const std::vector<std::u16string>& args) override;
    
    // 核心接口实现
    ErrCode Subscribe(const SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &callback) override;
    ErrCode Unsubscribe(const sptr<IRemoteObject> &callback) override;
    ErrCode CollectorStart(const SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) override;
    ErrCode CollectorStop(const SecurityCollectorSubscribeInfo &subscribeInfo,
        const sptr<IRemoteObject> &cb) override;
    ErrCode QuerySecurityEvent(
        const std::vector<SecurityCollector::SecurityEventRuler> &rulers,
        const sptr<IRemoteObject> &cb) override;
    ErrCode AddFilter(const SecurityEventFilter &filter,
        const sptr<IRemoteObject> &callback) override;
    ErrCode RemoveFilter(const SecurityEventFilter &filter,
        const sptr<IRemoteObject> &callback) override;

private:
    // 权限检查
    int32_t HasPermission(const std::string &permission);
    
    // 资源管理
    std::atomic<int32_t> g_refCount{0};  // 引用计数
    bool g_flag{true};  // 运行标志
};
```

### 5.4 权限校验实现

**证据来源**：`services/security_collector/src/security_collector_manager_service.cpp:393-402`

```cpp
int32_t SecurityCollectorManagerService::HasPermission(const std::string &permission)
{
    AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int code = AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permission);
    if (code != AccessToken::PermissionState::PERMISSION_GRANTED) {
        return NO_PERMISSION;
    }
    return SUCCESS;
}
```

### 5.5 服务依赖

| 依赖项 | 类型 | 说明 |
|--------|------|------|
| libsg_collect_sdk | SDK | 数据采集 SDK |
| libsg_collector_sdk | SDK | 采集器 SDK |
| security_collector_manager | 共享库 | 采集器管理 |

**证据来源**：`services/security_collector/BUILD.gn`

---

## 6. 权限管理

### 6.1 权限列表

| 权限名称 | 用途 | 所需服务 |
|---------|------|----------|
| `ohos.permission.COLLECT_SECURITY_EVENT` | 采集安全事件 | SecurityCollectorManager |
| `ohos.permission.QUERY_SECURITY_EVENT` | 查询安全事件 | DataCollectManager |
| `ohos.permission.REPORT_SECURITY_EVENT` | 上报安全事件 | DataCollectManager |
| `ohos.permission.MANAGE_SECURITY_GUARD_CONFIG` | 管理安全配置 | DataCollectManager |

**证据来源**：`sa_profile/security_guard.cfg:29-31`、`sa_profile/security_collector.cfg`

### 6.2 权限校验调用点

**证据来源**：`services/security_collector/src/security_collector_manager_service.cpp`

| 方法 | 权限检查 | 行号 |
|------|----------|------|
| Subscribe | COLLECT_SECURITY_EVENT / QUERY_SECURITY_EVENT | 115 |
| Unsubscribe | COLLECT_SECURITY_EVENT / QUERY_SECURITY_EVENT | 149 |
| CollectorStart | COLLECT_SECURITY_EVENT | 175 |
| CollectorStop | COLLECT_SECURITY_EVENT | 226 |
| QuerySecurityEvent | QUERY_SECURITY_EVENT | 351-356 |
| AddFilter | QUERY_SECURITY_EVENT | 407 |
| RemoveFilter | QUERY_SECURITY_EVENT | 422 |

---

## 7. 服务间通信

### 7.1 IPC 通信模式

SecurityGuard 使用 OpenHarmony 标准 IPC 框架，采用 Proxy-Stub 模式：

```
客户端进程                          服务端进程
    │                                   │
    ├──► Proxy 类                      ├──► Stub 类
    │    (序列化请求)                   │    (反序列化请求)
    │                                   │
    ├──► MessageParcel                 ├──► MessageParcel
    │    WriteRemoteObject()           │    ReadRemoteObject()
    │                                   │
    │                                   ├──► 服务实现
    │                                   │    (业务逻辑)
    │                                   │
    ◄─── Reply                         ◄─── Reply
```

### 7.2 消息序列化示例

**证据来源**：`services/security_collector/src/security_collector_manager_stub.cpp`

```cpp
ErrCode SecurityCollectorManagerStub::Subscribe(
    const MessageParcel &data, const MessageParcel &reply)
{
    // 1. 接口令牌验证
    std::u16string token = data.ReadInterfaceToken();
    if (!ifaceTokenCheck(token)) {
        SGLOGE("Interface token check failed");
        return ERR_INVALID_VALUE;
    }

    // 2. 反序列化参数
    std::unique_ptr<SecurityCollectorSubscribeInfo> info(
        data.ReadParcelable<SecurityCollectorSubscribeInfo>());
    if (info == nullptr) {
        SGLOGE("Failed to read subscribe info");
        return ERR_INVALID_VALUE;
    }

    auto callback = data.ReadRemoteObject();
    if (callback == nullptr) {
        SGLOGE("Failed to read callback");
        return ERR_INVALID_VALUE;
    }

    // 3. 调用服务实现
    int32_t ret = Subscribe(*info, callback);

    // 4. 写入响应
    reply.WriteInt32(ret);
    return ERR_OK;
}
```

---

## 8. 线程模型

### 8.1 FFRT 任务调度

所有 SA 服务使用 FFRT 进行异步任务调度：

| 服务 | 线程/队列类型 | 用途 |
|------|--------------|------|
| DataCollectManager | ffrt::submit() | 异步任务提交 |
| SecurityCollectorManager | ffrt::thread | 后台卸载检查 |
| AcquireDataSubscribeManager | ffrt::queue (Serial) | 事件上传队列 |

### 8.2 Token Bucket 限流

**证据来源**：`services/data_collect/sa/data_collect_manager_service.cpp`

```cpp
// Token Bucket 限流任务
auto tokenBucketTask = [this]() {
    while (true) {
        if (tokenBucket_.load() < TOKEN_BUCKET_MAX_SIZE) {
            tokenBucket_.fetch_add(TOKEN_BUCKET_STEP_SIZE);
        }
        ffrt::this_task::sleep_for(std::chrono::milliseconds(TOKEN_BUCKET_INTERVAL_TIME));
    }
};
ffrt::submit(tokenBucketTask);
```

### 8.3 自动卸载机制

**证据来源**：`services/security_collector/src/security_collector_manager_service.cpp`

```cpp
// 无订阅时自动卸载 SA
auto task = []() {
    while (g_flag) {
        ffrt::this_task::sleep_for(std::chrono::milliseconds(SLEEP_INTERVAL));
        if (g_refCount.load() != 0) continue;  // 有订阅时不卸载
        registry->UnloadSystemAbility(SECURITY_COLLECTOR_MANAGER_SA_ID);
        break;
    }
};
g_mainThread = ffrt::thread(task);
```

---

## 9. 相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 项目概览 |
| [02_NAPI_Reference.md](./02_NAPI_Reference.md) | JS API 参考 |
| [03_Architecture.md](./03_Architecture.md) | 架构详解 |
| [04_Build.md](./04_Build.md) | 构建配置 |
| [06_Security_Review.md](./06_Security_Review.md) | 安全风险分析 |
