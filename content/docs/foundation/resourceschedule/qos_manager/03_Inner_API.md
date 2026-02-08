# 内部 API

> 本文档描述 qos_manager 面向系统内部模块的 C++ API。

## 1. 概述

### 1.1 API 分层

| 层级 | 头文件 | 访问权限 | 描述 |
|------|--------|----------|------|
| **NDK (对外)** | `interfaces/kits/c/qos.h` | 公开 | Native 应用使用 |
| **Inner API (对内)** | `interfaces/inner_api/*` | 系统内部 | 子系统间调用 |
| **Private** | `services/include/*`, `frameworks/*/include/*` | 模块私有 | 内部实现 |

### 1.2 Inner API 列表

| 头文件 | 用途 | 稳定性 |
|--------|------|--------|
| `concurrent_task_client.h` | 并发任务客户端 | 稳定 |
| `concurrent_task_type.h` | 类型定义 | 稳定 |
| `qos.h` | QoS 控制器 | 稳定 |
| `pi_mutex.h` | PI 互斥锁 | 稳定 |

---

## 2. ConcurrentTaskClient API

### 2.1 类概述

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/inner_api/concurrent_task_client.h` |
| **命名空间** | `OHOS::ConcurrentTask` |
| **设计模式** | 单例 |
| **线程安全** | 是 (内部互斥锁) |

### 2.2 类定义

```cpp
// 证据: interfaces/inner_api/concurrent_task_client.h:31-139
class ConcurrentTaskClient {
public:
    static ConcurrentTaskClient& GetInstance();

    // 上报场景数据
    void ReportData(uint32_t resType, int64_t value,
        const std::unordered_map<std::string, std::string>& mapPayload);

    // 上报场景信息
    void ReportSceneInfo(uint32_t type,
        const std::unordered_map<std::string, std::string>& mapPayload);

    // 查询 RTG 间隔信息
    void QueryInterval(int queryItem, IntervalReply& queryRs);

    // 查询截止时间
    void QueryDeadline(int queryItem, DeadlineReply& ddlReply,
        const std::unordered_map<pid_t, uint32_t>& mapPayload);

    // 查询截止时间 (重载)
    void QueryDeadline(int queryItem, DeadlineReply& ddlReply,
        const std::unordered_map<std::string, std::string>& mapPayload);

    // 设置音频截止时间
    void SetAudioDeadline(int queryItem, int tid, int grpId, IntervalReply& queryRs);

    // 请求权限
    void RequestAuth(const std::unordered_map<std::string, std::string>& mapPayload);

    // 设置系统 QoS
    int SetSystemQoS(int tid, int level);

    // 停止远程对象
    void StopRemoteObject();

private:
    ConcurrentTaskClient();
    ~ConcurrentTaskClient();

    class ConcurrentTaskDeathRecipient : public IRemoteObject::DeathRecipient {
        void OnRemoteDied(const wptr<IRemoteObject>& object) override;
    };

    ErrCode TryConnect();
    std::mutex mutex_;
    sptr<ConcurrentTaskDeathRecipient> recipient_;
    sptr<IRemoteObject> remoteObject_;
    sptr<IConcurrentTaskService> clientService_;
};
```

### 2.3 核心方法详解

#### 2.3.1 GetInstance

```cpp
static ConcurrentTaskClient& GetInstance();
```

| 说明 | 返回值 |
|------|--------|
| 获取客户端单例实例 | `ConcurrentTaskClient&` |

**使用示例**:
```cpp
auto& client = ConcurrentTaskClient::GetInstance();
```

#### 2.3.2 ReportData

```cpp
// 证据: interfaces/inner_api/concurrent_task_client.h:47
void ReportData(uint32_t resType, int64_t value,
    const std::unordered_map<std::string, std::string>& mapPayload);
```

| 参数 | 类型 | 描述 |
|------|------|------|
| `resType` | `uint32_t` | 资源类型 |
| `value` | `int64_t` | 值 (如 uid) |
| `mapPayload` | `map<string, string>` | 负载信息 |

**调用链**:
```
ConcurrentTaskClient::ReportData()
    ↓ IPC 调用
IConcurrentTaskService::ReportData()
    ↓ Binder IPC
ConcurrentTaskService::ReportData()
    ↓
TaskControllerInterface::ReportData()
```

#### 2.3.3 SetSystemQoS

```cpp
// 证据: interfaces/inner_api/concurrent_task_client.h:114
int SetSystemQoS(int tid, int level);
```

| 参数 | 类型 | 描述 |
|------|------|------|
| `tid` | `int` | 目标线程 ID |
| `level` | `int` | QoS 等级 |

| 返回值 | 含义 |
|--------|------|
| 0 | 成功 |
| < 0 | 失败 |

---

## 3. QosController API

### 3.1 类概述

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/inner_api/qos.h` |
| **命名空间** | `OHOS::QOS` |
| **设计模式** | 单例 |
| **头文件依赖** | 无 (自包含) |

### 3.2 QoS 等级枚举

```cpp
// 证据: interfaces/inner_api/qos.h:21-30
enum class QosLevel {
    QOS_BACKGROUND,
    QOS_UTILITY,
    QOS_DEFAULT,
    QOS_USER_INITIATED,
    QOS_DEADLINE_REQUEST,
    QOS_USER_INTERACTIVE,
    QOS_KEY_BACKGROUND,
    QOS_MAX,
};
```

**注意**: Inner API 使用 `QosLevel` (C++ enum class)，NDK API 使用 `QoS_Level` (C enum)。

### 3.3 类定义

```cpp
// 证据: interfaces/inner_api/qos.h:32-48
class QosController {
public:
    static QosController& GetInstance();

    // 为其他线程设置 QoS
    int SetThreadQosForOtherThread(enum QosLevel level, int tid);
    // 重置其他线程 QoS
    int ResetThreadQosForOtherThread(int tid);
    // 获取其他线程 QoS
    int GetThreadQosForOtherThread(enum QosLevel &level, int tid);

private:
    QosController() = default;
    ~QosController() = default;
    // 删除拷贝构造
    QosController(const QosController&) = delete;
    QosController& operator=(const QosController&) = delete;
};
```

### 3.4 全局函数

```cpp
// 证据: interfaces/inner_api/qos.h:50-59
namespace OHOS {
namespace QOS {

// 设置当前线程 QoS
int SetThreadQos(enum QosLevel level);

// 为其他线程设置 QoS
int SetQosForOtherThread(enum QosLevel level, int tid);

// 重置当前线程 QoS
int ResetThreadQos();

// 重置其他线程 QoS
int ResetQosForOtherThread(int tid);

// 获取当前线程 QoS
int GetThreadQos(enum QosLevel &level);

// 获取其他线程 QoS
int GetQosForOtherThread(enum QosLevel &level, int tid);

// RTG 相关
int AddThreadToProcRtg(int tid);
int AddThreadsToProcRtg(int tid[5], int size);
int RemoveThreadFromProcRtg(int tid);
int RemoveThreadsFromProcRtg(int tid[5], int size);

} // namespace QOS
} // namespace OHOS
```

### 3.5 RTG 函数实现状态

```cpp
// 证据: qos/qos.cpp:130-148
// 注意: 以下函数当前返回 0 (未完全实现)
int AddThreadToProcRtg([[maybe_unused]] int tid)
{
    return 0;  // 未实现
}

int AddThreadsToProcRtg([[maybe_unused]] int tid[5], [[maybe_unused]] int size)
{
    return 0;  // 未实现
}

int RemoveThreadFromProcRtg([[maybe_unused]] int tid)
{
    return 0;  // 未实现
}

int RemoveThreadsFromProcRtg([[maybe_unused]] int tid[5], [[maybe_unused]] int size)
{
    return 0;  // 未实现
}
```

---

## 4. 类型定义

### 4.1 消息类型

```cpp
// 证据: interfaces/inner_api/concurrent_task_type.h:25-44
enum MsgType {
    MSG_FOREGROUND = 0,           // 前台
    MSG_BACKGROUND,               // 后台
    MSG_APP_START,                // 应用启动
    MSG_APP_KILLED,               // 应用终止
    MSG_CONTINUOUS_TASK_START,    // 连续任务开始
    MSG_CONTINUOUS_TASK_END,      // 连续任务结束
    MSG_GET_FOCUS,                // 获取焦点
    MSG_LOSE_FOCUS,               // 失去焦点
    MSG_ENTER_INTERACTION_SCENE,  // 进入交互场景
    MSG_EXIT_INTERACTION_SCENE,   // 退出交互场景
    MSG_SUB_FOCUS,                // 子焦点
    MSG_GROUP_CHANGE,             // 组变化
    MSG_SYSTEM_MAX,
    MSG_APP_START_TYPE = 100,
    MSG_REG_RENDER = MSG_APP_START_TYPE,
    MSG_REG_UI,
    MSG_REG_KEY_THERAD,
    MSG_TYPE_MAX
};
```

### 4.2 查询项类型

```cpp
// 证据: interfaces/inner_api/concurrent_task_type.h:52-62
enum QueryIntervalItem {
    QUERY_UI = 0,                  // UI 线程
    QUERY_RENDER,                 // 渲染线程
    QUERY_RENDER_SERVICE,         // 渲染服务
    QUERY_COMPOSER,               // 合成器
    QUERY_HARDWARE,               // 硬件线程
    QUERY_EXECUTOR_START,         // 执行器启动
    QUERY_RENDER_SERVICE_MAIN,    // 渲染服务主线程
    QUERY_RENDER_SERVICE_RENDER, // 渲染服务渲染线程
    QURRY_TYPE_MAX
};
```

### 4.3 音频截止时间类型

```cpp
// 证据: interfaces/inner_api/concurrent_task_type.h:64-69
enum AudioDeadlineType {
    AUDIO_DDL_CREATE_GRP = 0,     // 创建组
    AUDIO_DDL_DESTROY_GRP,        // 销毁组
    AUDIO_DDL_ADD_THREAD,         // 添加线程
    AUDIO_DDL_REMOVE_THREAD,      // 移除线程
};
```

### 4.4 截止时间类型

```cpp
// 证据: interfaces/inner_api/concurrent_task_type.h:71-74
enum DeadlineType {
    DDL_RATE = 0,                 // 帧率
    MSG_GAME = 1,                 // 游戏
};
```

### 4.5 RTG 间隔回复

```cpp
// 证据: interfaces/inner_api/concurrent_task_type.h:76-82
struct IntervalReply {
    int rtgId;                     // RTG 组 ID
    int tid;                       // 线程 ID
    int paramA;                    // 参数 A
    int paramB;                    // 参数 B
    std::string bundleName;        // 包名
};
```

### 4.6 截止时间回复

```cpp
// 证据: interfaces/inner_api/concurrent_task_type.h:84-86
struct DeadlineReply {
    bool setStatus;                // 设置状态
};
```

---

## 5. 模块依赖方向

### 5.1 依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                      interfaces/inner_api/                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  concurrent_task_client.h                                       │
│      ↓ 依赖                                                     │
│  concurrent_task_type.h (类型定义)                              │
│      ↓ 依赖                                                     │
│  IConcurrentTaskService (IDL 生成)                             │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  qos.h (QosController)                                         │
│      ↓ 依赖                                                     │
│  qos.cpp                                                       │
│      ↓ 依赖                                                     │
│  qos_interface.h (services/include/)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 稳定性标注

| 头文件 | 稳定性 | 证据 |
|--------|--------|------|
| `concurrent_task_client.h` | 稳定 | 位于 `interfaces/inner_api/` 目录 |
| `concurrent_task_type.h` | 稳定 | 位于 `interfaces/inner_api/` 目录 |
| `qos.h` | 稳定 | 位于 `interfaces/inner_api/` 目录 |
| `pi_mutex.h` | 稳定 | 位于 `interfaces/inner_api/` 目录 |

**稳定性判断依据**: 位于 `interfaces/inner_api/` 目录下，属于系统内部稳定接口。

---

## 6. IPC 接口定义

### 6.1 IConcurrentTaskService 接口

**IDL 文件**: `frameworks/concurrent_task_client/idl/IConcurrentTaskService.idl`

| 方法 | 类型 | 描述 |
|------|------|------|
| `ReportData` | oneway | 上报数据 |
| `ReportSceneInfo` | oneway | 上报场景信息 |
| `QueryInterval` | sync | 查询间隔 |
| `QueryDeadline` | oneway | 查询截止时间 |
| `SetAudioDeadline` | sync | 设置音频截止时间 |
| `RequestAuth` | sync | 请求权限 |

### 6.2 IPC 返回值

```cpp
// 证据: services/include/concurrent_task_service.h
ErrCode ReportData(...) override;
ErrCode ReportSceneInfo(...) override;
ErrCode QueryInterval(...) override;
ErrCode QueryDeadline(...) override;
ErrCode SetAudioDeadline(...) override;
ErrCode RequestAuth(...) override;
```

| 返回值类型 | 描述 |
|------------|------|
| `ERR_OK` | 成功 |
| 其他 | 见 `native/inc/errors.h` |

---

## 7. 相关文档

| 文档 | 描述 |
|------|------|
| [01_Architecture.md](./01_Architecture.md) | 架构图和模块关系 |
| [02_NDK_API.md](./02_NDK_API.md) | NDK C 接口 |
| [04_Build_Targets.md](./04_Build_Targets.md) | 构建配置 |
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 详细调用链 |
