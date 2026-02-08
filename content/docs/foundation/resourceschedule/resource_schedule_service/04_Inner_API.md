# 内部 API 参考

## 目的

本文档描述 Resource Schedule Service 的 C++ 内部接口，包括客户端 API、服务接口和插件框架。

## 适用范围

- 系统服务开发者
- 插件开发者
- 框架维护者

---

## ResSchedClient - 资源调度客户端

### 类概述

`ResSchedClient` 是资源调度服务的客户端接口，封装了 IPC 调用细节，提供单例模式访问。

**头文件**: `ressched/interfaces/innerkits/ressched_client/include/res_sched_client.h`

**实现文件**: `ressched/interfaces/innerkits/ressched_client/src/res_sched_client.cpp`

### 类定义

```cpp
namespace OHOS {
namespace ResourceSchedule {

class ResSchedClient {
public:
    static ResSchedClient& GetInstance();
    
    // 事件上报
    void ReportData(uint32_t resType, int64_t value,
        const std::unordered_map<std::string, std::string>& mapPayload);
    
    // 同步事件上报
    int32_t ReportSyncEvent(const uint32_t resType, const int64_t value,
        const nlohmann::json& payload, nlohmann::json& reply);
    
    // 杀进程
    int32_t KillProcess(const std::unordered_map<std::string, std::string>& mapPayload);
    
    // 系统负载监听
    void RegisterSystemloadNotifier(const sptr<ResSchedSystemloadNotifierClient>& callbackObj);
    void UnRegisterSystemloadNotifier(const sptr<ResSchedSystemloadNotifierClient>& callbackObj);
    int32_t GetSystemloadLevel();
    
    // 应用预加载
    bool IsAllowedAppPreload(const std::string& bundleName, int32_t preloadMode);
    
    // 链接跳转
    int32_t IsAllowedLinkJump(bool& isAllowedLinkJump);
};

} // namespace ResourceSchedule
} // namespace OHOS
```

### 方法详解

#### ReportData

**功能**: 异步上报资源事件到资源调度服务。

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| resType | uint32_t | 资源类型，定义见 res_type.h |
| value | int64_t | 资源值，由调用者定义 |
| mapPayload | unordered_map | 上下文信息键值对 |

**示例**:
```cpp
#include "res_sched_client.h"

using namespace OHOS::ResourceSchedule;

// 上报应用启动事件
std::unordered_map<std::string, std::string> payload;
payload["bundleName"] = "com.example.app";
payload["pid"] = "1234";

ResSchedClient::GetInstance().ReportData(
    ResType::RES_TYPE_APP_STATE_CHANGE,  // 1
    ResType::AppStateChangeType::APP_STATE_CREATE,  // 值
    payload
);
```

**实现说明**:
- 内部使用 IPC 调用 `IResSchedService::ReportData`
- 带有请求限流保护（单 UID 250 req/s，全系统 650 req/s）
- 自动处理服务断开重连

---

#### ReportSyncEvent

**功能**: 同步上报事件并获取响应。

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| resType | uint32_t | 资源类型 |
| value | int64_t | 资源值 |
| payload | nlohmann::json | 请求数据 |
| reply | nlohmann::json | 响应数据（输出） |

**返回值**:
- `ERR_OK` (0) - 成功
- `RES_SCHED_CONNECT_FAIL` - 连接失败
- `ERR_RES_SCHED_PERMISSION_DENIED` - 权限拒绝

**示例**:
```cpp
nlohmann::json payload = {
    {"bundleName", "com.example.app"},
    {"pid", 1234}
};
nlohmann::json reply;

int32_t ret = ResSchedClient::GetInstance().ReportSyncEvent(
    ResType::SYNC_RES_TYPE_THAW_ONE_APP,
    0,
    payload,
    reply
);

if (ret == ERR_OK) {
    bool success = reply["result"].get<bool>();
}
```

**特殊处理**:
- `SYNC_RES_TYPE_CHECK_MUTEX_BEFORE_START` 类型使用 FFRT 任务队列，带 100ms 超时保护

---

#### GetSystemloadLevel

**功能**: 获取当前系统负载等级。

**返回值**: 系统负载等级 (0-7)，对应 `SystemLoadLevel` 枚举。

**示例**:
```cpp
int32_t level = ResSchedClient::GetInstance().GetSystemloadLevel();
if (level >= ResType::SystemloadLevel::HIGH) {
    // 系统负载高，降低资源消耗
}
```

---

## ResSchedExeClient - 执行器客户端

### 类概述

`ResSchedExeClient` 是资源调度执行器服务的客户端接口，用于执行具体的调度操作（如设置 cgroup、调节频率等）。

**头文件**: `ressched_executor/interfaces/innerkits/ressched_executor_client/include/res_sched_exe_client.h`

### 类定义

```cpp
namespace OHOS {
namespace ResourceSchedule {

class ResSchedExeClient {
public:
    static ResSchedExeClient& GetInstance();
    
    // 同步发送请求
    int32_t SendRequestSync(uint32_t resType, int64_t value,
        const std::unordered_map<std::string, std::string>& mapPayload,
        std::unordered_map<std::string, std::string>& reply);
    
    // 异步发送请求
    void SendRequestAsync(uint32_t resType, int64_t value,
        const std::unordered_map<std::string, std::string>& mapPayload);
    
    // 杀进程
    int32_t KillProcess(uint32_t pid);
};

} // namespace ResourceSchedule
} // namespace OHOS
```

### 与 ResSchedClient 的区别

| 特性 | ResSchedClient | ResSchedExeClient |
|------|---------------|-------------------|
| **SA ID** | 1901 | 1918 |
| **职责** | 事件接收与分发 | 具体调度操作执行 |
| **进程** | resource_schedule_service | resource_schedule_executor |
| **典型调用** | ReportData | SendRequestSync |

---

## 插件框架接口

### Plugin - 插件基类

**头文件**: `ressched/services/resschedmgr/pluginbase/include/plugin.h`

```cpp
namespace OHOS {
namespace ResourceSchedule {

class Plugin {
public:
    // 插件初始化
    // 返回: true 成功, false 失败
    virtual bool OnPluginInit(std::string& libName) = 0;
    
    // 插件禁用
    virtual void OnPluginDisable() = 0;
    
    // 事件分发
    // data: 资源事件数据
    virtual void OnDispatchResource(const std::shared_ptr<ResData>& data) = 0;
};

} // namespace ResourceSchedule
} // namespace OHOS
```

### ResData - 资源数据结构

**头文件**: `ressched/services/resschedmgr/pluginbase/include/res_data.h`

```cpp
struct ResData {
    uint32_t resType;           // 资源类型
    int64_t value;              // 资源值
    nlohmann::json payload;     // 载荷数据 (JSON 格式)
    nlohmann::json reply;       // 同步响应数据
    
    ResData(uint32_t type, int64_t val, const nlohmann::json& pay)
        : resType(type), value(val), payload(pay) {}
    
    ResData(uint32_t type, int64_t val, const nlohmann::json& pay,
            nlohmann::json& rep)
        : resType(type), value(val), payload(pay), reply(rep) {}
};
```

### 插件实现示例

```cpp
#include "plugin.h"
#include "res_data.h"
#include "res_sched_log.h"

namespace OHOS {
namespace ResourceSchedule {

class MyPlugin : public Plugin {
public:
    bool OnPluginInit(std::string& libName) override {
        RESSCHED_LOGI("MyPlugin initialized");
        // 订阅感兴趣的资源类型
        PluginMgr::GetInstance().SubscribeResource(libName, ResType::APP_STATE_CHANGE);
        return true;
    }
    
    void OnPluginDisable() override {
        RESSCHED_LOGI("MyPlugin disabled");
    }
    
    void OnDispatchResource(const std::shared_ptr<ResData>& data) override {
        // 处理事件
        if (data->resType == ResType::APP_STATE_CHANGE) {
            std::string bundleName = data->payload["bundleName"];
            RESSCHED_LOGI("App state changed: %s", bundleName.c_str());
        }
        
        // 如需返回数据（同步事件）
        data->reply["result"] = true;
    }
};

extern "C" Plugin* CreatePlugin() {
    return new MyPlugin();
}

} // namespace ResourceSchedule
} // namespace OHOS
```

---

## 关键资源类型 (ResType)

**头文件**: `ressched/interfaces/innerkits/ressched_client/include/res_type.h`

### 应用生命周期

```cpp
enum AppStateChangeType : int64_t {
    APP_STATE_CREATE = 1,
    APP_STATE_FOREGROUND = 2,
    APP_STATE_BACKGROUND = 3,
    APP_STATE_DESTROY = 4,
};

enum ProcessStateChangeType : int64_t {
    PROCESS_STATE_CREATE = 1,
    PROCESS_STATE_FOREGROUND = 2,
    PROCESS_STATE_BACKGROUND = 3,
    PROCESS_STATE_TERMINATED = 4,
};

RES_TYPE_APP_STATE_CHANGE = 1
RES_TYPE_PROCESS_STATE_CHANGE = 3
RES_TYPE_ABILITY_STATE_CHANGE = 2
```

### 窗口事件

```cpp
RES_TYPE_WINDOW_FOCUS = 6
RES_TYPE_WINDOW_VISIBILITY_CHANGE = 7
RES_TYPE_MOVE_WINDOW = 50
RES_TYPE_RESIZE_WINDOW = 51
RES_TYPE_SCENE_ROTATION = 71
```

### 用户交互

```cpp
RES_TYPE_CLICK_RECOGNIZE = 12
RES_TYPE_SLIDE_RECOGNIZE = 13
RES_TYPE_KEY_EVENT = 24
RES_TYPE_MOUSEWHEEL = 42
RES_TYPE_GESTURE_ANIMATION = 81
```

### 同步资源类型

```cpp
SYNC_RES_TYPE_THAW_ONE_APP = 200
SYNC_RES_TYPE_GET_THERMAL_DATA = 203
SYNC_RES_TYPE_CHECK_MUTEX_BEFORE_START = 204
SYNC_RES_TYPE_GET_SUSPEND_STATE_BY_UID = 222
SYNC_RES_TYPE_GET_SUSPEND_STATE_BY_PID = 223
```

---

## 错误码定义

**头文件**: `ressched/interfaces/innerkits/ressched_client/include/res_sched_errors.h`

```cpp
constexpr int32_t ERR_OK = 0;
constexpr int32_t ERR_RES_SCHED_INVALID_PARAM = 9000001;
constexpr int32_t ERR_RES_SCHED_PERMISSION_DENIED = 9000002;
constexpr int32_t ERR_RES_SCHED_PARCEL_ERROR = 9000003;
constexpr int32_t ERR_RES_SCHED_CONNECT_FAIL = 9000004;
constexpr int32_t ERR_RES_SCHED_HANDLER_NOT_FOUND = 9000005;
```

---

## IPC 接口码

**头文件**: `ressched/interfaces/innerkits/ressched_client/include/res_sched_ipc_interface_code.h`

```cpp
enum class ResSchedInterfaceCode {
    REPORT_DATA = 1,
    KILL_PROCESS = 2,
    REGISTER_SYSTEMLOAD_NOTIFIER = 3,
    UNREGISTER_SYSTEMLOAD_NOTIFIER = 4,
    GET_SYSTEMLOAD_LEVEL = 5,
    TOUCH_DOWN_APP_PRELOAD = 6,
    REPORT_SYNC_EVENT = 7,
    REGISTER_EVENT_LISTENER = 8,
    UNREGISTER_EVENT_LISTENER = 9,
    LINK_JUMP_OPTIMIZATION = 10,
};
```

---

## 代码证据

### ResSchedClient 单例实现

```cpp
// 文件: ressched/interfaces/innerkits/ressched_client/src/res_sched_client.cpp:39-43

ResSchedClient& ResSchedClient::GetInstance() {
    static ResSchedClient instance;
    return instance;
}
```

### IPC 连接建立

```cpp
// 文件: ressched/interfaces/innerkits/ressched_client/src/res_sched_client.cpp:339-374

ErrCode ResSchedClient::TryConnect() {
    std::lock_guard<ffrt::mutex> lock(mutex_);
    if (rss_) {
        return ERR_OK;
    }
    
    // 获取 SA 管理器
    sptr<ISystemAbilityManager> systemManager = 
        SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    
    // 查找服务
    remoteObject_ = systemManager->CheckSystemAbility(RES_SCHED_SYS_ABILITY_ID);  // 1901
    
    // 创建代理
    rss_ = iface_cast<IResSchedService>(remoteObject_);
    
    // 添加死亡监听
    recipient_ = new ResSchedDeathRecipient(*this);
    rss_->AsObject()->AddDeathRecipient(recipient_);
    
    return ERR_OK;
}
```

---

## 相关链接

- [概览](00_Overview.md) - 项目定位
- [架构设计](01_Architecture.md) - 数据流图
- [N-API 参考](03_NAPI_Reference.md) - JS 接口
- [GN 构建](05_GN_Targets.md) - 编译配置
