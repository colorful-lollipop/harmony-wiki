# 08_Internals - 内部实现细节

本文档深入解析 SOC 统一调频部件的内部实现细节，帮助开发者理解核心类设计、资源生命周期和内部 API 契约。

## 核心类设计

### SocPerf - 调频仲裁器

**文件**：`services/core/include/socperf.h`

**职责**：调频请求处理、仲裁决策、状态管理

#### 公共接口

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `Init()` | - | `bool` | 初始化，调用配置加载 |
| `PerfRequest` | `cmdId: int32_t`, `msg: std::string` | `void` | 性能提频 |
| `PerfRequestEx` | `cmdId: int32_t`, `onOffTag: bool`, `msg: std::string` | `void` | 带开关提频 |
| `PowerLimitBoost` | `onOffTag: bool`, `msg: std::string` | `void` | 功耗限频 |
| `ThermalLimitBoost` | `onOffTag: bool`, `msg: std::string` | `void` | 热限频 |
| `LimitRequest` | `clientId`, `tags`, `configs`, `msg` | `void` | 多项限频 |
| `SetRequestStatus` | `status: bool`, `msg: std::string` | `void` | 服务开关 |
| `SetThermalLevel` | `level: int32_t` | `void` | 设置热等级 |
| `RequestDeviceMode` | `mode: std::string`, `status: bool` | `void` | 设备模式 |
| `RequestCmdIdCount` | `msg: std::string` | `std::string` | 统计查询 |

#### 成员变量

| 变量名 | 类型 | 用途 |
|--------|------|------|
| `enabled_` | `bool` | 服务使能状态 |
| `perfRequestEnable_` | `volatile bool` | 提频使能（volatile 防止优化） |
| `thermalLvl_` | `int32_t` | 当前热等级 |
| `socperfThreadWrap_` | `std::shared_ptr<SocPerfThreadWrap>` | 线程封装实例 |
| `socPerfConfig_` | `SocPerfConfig&` | 配置管理单例引用 |
| `limitRequest_` | `std::vector<std::unordered_map<int32_t, int32_t>>` | 限频状态（按 ActionType 索引） |
| `boostCmdCount_` | `std::unordered_map<int32_t, uint32_t>` | 提频命令计数 |
| `boostTime_` | `std::unordered_map<int32_t, uint64_t>` | 提频时间戳 |
| `recordDeviceMode_` | `std::set<std::string>` | 设备模式记录 |

#### 内部方法

| 方法 | 可见性 | 说明 |
|------|--------|------|
| `CreateThreadWraps()` | private | 创建线程封装实例 |
| `InitThreadWraps()` | private | 初始化线程封装 |
| `DoFreqActions()` | private | 执行频率动作 |
| `DoPerfRequestThremalLvl()` | private | 处理热等级 |
| `SendLimitRequestEvent()` | private | 发送限频事件 |
| `CheckTimeInterval()` | private | 检查时间间隔 |
| `CompleteEvent()` | private | 完成事件处理 |
| `GetActionsInfo()` | private | 获取动作信息 |

### SocPerfConfig - 配置管理器

**文件**：`services/core/include/socperf_config.h`

**职责**：XML 配置加载、解析、验证

#### 公共接口

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `GetInstance()` | - | `SocPerfConfig&` | 获取单例 |
| `Init()` | - | `bool` | 初始化加载 |
| `GetActionsInfo()` | `cmdId: int32_t` | `std::shared_ptr<Actions>` | 获取动作 |
| `GetResource()` | `resId: int32_t` | `std::shared_ptr<ResourceNode>` | 获取资源 |

#### 配置加载流程

```
SocPerfConfig::Init()
    ↓
LoadAllConfigXmlFile("socperf_resource_config.xml")
    ├─ xmlReadFile() → 解析 XML
    ├─ ParseResourceXmlFile()
    │   ├─ LoadResource() → 频率资源
    │   ├─ LoadGovResource() → 治理资源
    │   └─ LoadSceneResource() → 场景资源
    ↓
LoadAllConfigXmlFile("socperf_boost_config.xml")
    ├─ ParseBoostXmlFile()
    │   └─ LoadConfig()
    │       ├─ LoadCmdInfo() → 命令配置
    │       └─ LoadInterAction() → 弱交互配置
```

### SocPerfThreadWrap - 线程封装

**文件**：`services/core/include/socperf_thread_wrap.h`

**职责**：FFRT 任务队列、内核接口调用

#### 公共接口

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `Execute()` | - | `bool` | 执行任务队列 |
| `AddTask()` | `task: std::function<void()>` | `void` | 添加异步任务 |

## 数据结构

### ActionType 枚举

**文件**：`interfaces/inner_api/socperf_client/include/socperf_action_type.h`

```cpp
enum ActionType : uint32_t {
    ACTION_TYPE_PERF,      // 性能提频
    ACTION_TYPE_POWER,     // 功耗限频
    ACTION_TYPE_THERMAL,   // 热限频
    ACTION_TYPE_PERFLVL,   // 性能级别
    ACTION_TYPE_BATTERY,   // 电池相关
    ACTION_TYPE_MAX
};
```

### Actions - 动作集合

**文件**：`services/core/include/socperf_common.h`

```cpp
class Actions {
public:
    std::vector<std::shared_ptr<Action>> actionList_;  // 动作列表
    std::map<int32_t, std::vector<int32_t>> modeMap_;  // 模式映射
};
```

### Action - 单个动作

**文件**：`services/core/include/socperf_common.h`

```cpp
class Action {
public:
    int32_t cmdId_;           // 命令 ID
    int32_t actionType_;      // 动作类型
    int32_t resId_;           // 资源 ID
    int64_t value_;           // 目标值
    int32_t duration_;        // 持续时间
    int32_t thermalLvl_;      // 热等级
    int32_t thermalCmdId_;    // 热命令 ID
    int64_t endTime_;         // 结束时间
    bool onOffTag_;           // 开关标记
};
```

## 资源生命周期

### SocPerfServer 生命周期

```
SocPerfServer 构造
    ↓
OnStart()
    ├─ SocPerf::Init()
    │   ├─ SocPerfConfig::Init()
    │   │   ├─ LoadAllConfigXmlFile("resource_config.xml")
    │   │   └─ LoadAllConfigXmlFile("boost_config.xml")
    │   ├─ CreateThreadWraps()
    │   └─ InitThreadWraps()
    └─ 注册 SA 到 Samgr
    ↓
OnStop()
    ├─ 清理线程封装
    └─ 取消 SA 注册
```

### 客户端连接生命周期

```
SocPerfClient::GetInstance()
    ↓ (首次调用)
CheckClientValid()
    ├─ GetSystemAbilityManager()
    ├─ GetSystemAbility(SOC_PERF_SERVICE_SA_ID)
    ├─ iface_cast<ISocPerf>()
    └─ AddDeathRecipient()
    ↓
业务调用 (PerfRequest 等)
    ↓
服务端死亡
OnRemoteDied()
    └─ client_ = nullptr
    ↓ (下次调用)
CheckClientValid() 重连
```

## 内部 API 契约

### 稳定接口（对外）

以下接口定义在 IDL 中，对外稳定：

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `PerfRequest` | 高 | 性能请求接口 |
| `PerfRequestEx` | 高 | 带开关请求 |
| `PowerLimitBoost` | 高 | 功耗限频 |
| `ThermalLimitBoost` | 高 | 热限频 |
| `LimitRequest` | 高 | 多项限频 |

### 内部接口（不稳定）

以下接口仅内部使用，可能变更：

| 接口 | 文件 | 使用范围 |
|------|------|----------|
| `DoFreqActions()` | `socperf.cpp` | SocPerf 内部 |
| `CheckTimeInterval()` | `socperf.cpp` | SocPerf 内部 |
| `SendLimitRequestEvent()` | `socperf.cpp` | SocPerf 内部 |
| `GetActionsInfo()` | `socperf.cpp` | SocPerf 内部 |
| `Execute()` | `socperf_thread_wrap.cpp` | SocPerfThreadWrap 内部 |

## 线程模型详解

### SA 主线程

**职责**：接收并分发 IPC 消息

**证据**：`services/server/src/socperf_server.cpp:115-200`

```cpp
ErrCode SocPerfServer::PerfRequest(int32_t cmdId, const std::string& msg)
{
    // 权限检查
    if (!HasPerfPermission()) {
        return PERMISSION_DENIED;
    }
    // 转发到核心逻辑
    socPerf_.PerfRequest(cmdId, msg);
    return ERR_OK;
}
```

### FFRT 线程池

**职责**：异步执行调频任务

**证据**：`services/core/src/socperf_thread_wrap.cpp`

```cpp
bool SocPerfThreadWrap::Execute()
{
    std::function<void()> task = nullptr;
    {
        std::lock_guard<std::mutex> lock(mutex_);
        if (!taskQueue_.empty()) {
            task = taskQueue_.front();
            taskQueue_.pop();
        }
    }
    if (task) {
        task();
    }
    return true;
}
```

## 锁使用策略

| 锁 | 保护对象 | 粒度 |
|----|----------|------|
| `mutex_` | 通用数据（PerfRequest） | 粗粒度 |
| `mutexDeviceMode_` | `recordDeviceMode_` | 细粒度 |
| `mutexBoostCmdCount_` | `boostCmdCount_` | 细粒度 |
| `mutexBoostTime_` | `boostTime_` | 细粒度 |

**设计原则**：

- 高并发场景使用细粒度锁减少竞争
- 读多写少场景考虑读写锁（当前使用互斥锁）

---

## 相关文档

- 架构设计：[02_Architecture](02_Architecture.md)
- 代码地图：[03_CodeMap](03_CodeMap.md)
- 构建说明：[07_Build](07_Build.md)

---

*文档版本：v1.0*
*最后更新：2026-02-07*
