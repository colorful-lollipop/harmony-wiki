# 架构说明

## 目的

本文档详细说明指纹认证组件的架构设计，包括组件图、数据流、线程模型和关键时序。

## 适用范围

- 架构师
- 需要深入理解组件设计原理的开发者
- 性能优化人员

## 关键结论

1. **分层架构**：组件采用清晰的分层设计，从上到下依次为：UserAuth Framework → FingerprintAuthService → HDI 适配层 → Hardware Driver。

2. **通信模式**：
   - **下行调用**：Framework 通过 HDI Proxy 调用 Driver，执行操作
   - **上行回调**：Driver 通过 IExecutorCallback 返回结果
   - **命令流**：Driver 通过 ISaCommandCallback 发送命令到 SA

3. **线程模型**：服务运行在 SA 主线程，HDI 回调通过 HDF 线程池异步执行，UI 渲染在专门的渲染线程。

4. **扩展机制**：通过 `SaCommandManager` 和动态库加载实现功能扩展，支持厂商自定义命令。

---

## 组件图

### 整体架构

```mermaid
graph TB
    subgraph "应用层"
        App[系统应用]
    end

    subgraph "UserAuth Framework"
        UA[UserAuth Framework<br/>权限检查<br/>JS API]
    end

    subgraph "FingerprintAuthService (SA 943)"
        SA[FingerprintAuthService<br/>SystemAbility 单例]
        DM[DriverManager<br/>执行器管理]
        CMD[SaCommandManager<br/>命令管理]
        SIM[SensorIlluminationManager<br/>传感器照明]
    end

    subgraph "services_ex (扩展)"
        SSM[ScreenStateMonitor<br/>屏幕监听]
        SIT[SensorIlluminationTask<br/>UI 渲染]
    end

    subgraph "HDI Layer"
        HProxy[IFingerprintAuthInterface<br/>HDI Proxy]
        HDI[IAllInOneExecutor<br/>HDI Interface]
        CB[IExecutorCallback<br/>回调接口]
        SCB[ISaCommandCallback<br/>SA 命令回调]
    end

    subgraph "Hardware Driver"
        Driver[HDF Driver<br/>指纹驱动]
    end

    subgraph "Hardware"
        Sensor[指纹传感器]
    end

    App -->|1. 调用 JS API| UA
    UA -->|2. 调度执行器| SA
    SA --> DM
    DM -->|3. 获取执行器| HProxy
    DM -->|4. 注册命令处理器| CMD

    CMD -.->|5. 照明命令| SIM
    SIM -.->|6. 动态加载| SIT
    SSM -.->|7. 屏幕事件| SIM

    DM -->|8. 执行操作| HDI
    HDI --> Driver
    Driver --> Sensor

    Driver -->|9. 回调结果| CB
    CB -->|10. 转发结果| UA

    Driver -->|11. 发送命令| SCB
    SCB -->|12. 处理命令| CMD

    style SA fill:#e1f5fe
    style Driver fill:#ffecb3
    style UA fill:#fff9c4
```

### 模块交互图

```mermaid
sequenceDiagram
    participant App as 应用
    participant UA as UserAuth Framework
    participant SA as FingerprintAuthService
    participant DM as DriverHdi
    participant HDI as HDI Driver
    participant Cmd as SaCommandManager
    participant Task as SensorIlluminationTask

    Note over App,Task: 1. 录入流程
    App->>UA: enroll()
    UA->>SA: Enroll(scheduleId, token)
    SA->>DM: 获取执行器
    DM->>HDI: Enroll()
    HDI-->>DM: 回调 OnResult(SUCCESS)
    DM-->>UA: 返回结果
    UA-->>App: Promise resolve

    Note over App,Task: 2. 认证流程 + 照明
    App->>UA: authenticate()
    UA->>SA: Authenticate(scheduleId, token)
    SA->>DM: 获取执行器
    DM->>HDI: Authenticate()
    HDI->>Cmd: SendCommand(TURN_ON_ILLUMINATION)
    Cmd->>Task: 开启照明
    Task-->>Cmd: 完成
    Cmd-->>HDI: 命令响应
    HDI-->>DM: 回调 OnResult(SUCCESS/FAIL)
    DM->>Cmd: SendCommand(TURN_OFF_ILLUMINATION)
    Cmd->>Task: 关闭照明
```

---

## 核心组件说明

### 1. FingerprintAuthService（主服务）

**文件路径**：
- 头文件：`services/inc/fingerprint_auth_service.h`
- 实现：`services/src/fingerprint_auth_service.cpp`

**职责**：
- 作为 System Ability 943 注册到系统
- 管理服务生命周期（OnStart/OnStop）
- 初始化 DriverManager
- 单例模式，全局唯一实例

**关键代码**：
```cpp
// services/src/fingerprint_auth_service.cpp:51
const bool REGISTER_RESULT = SystemAbility::MakeAndRegisterAbility(
    FingerprintAuthService::GetInstance().get()
);

// services/src/fingerprint_auth_service.cpp:68
FingerprintAuthService::FingerprintAuthService()
    : SystemAbility(SUBSYS_USERIAM_SYS_ABILITY_FINGERPRINTAUTH, true)
{
}
```

**证据**：
- 文件：`services/src/fingerprint_auth_service.cpp:16-110`
- SA 注册：第 51 行
- SA ID 定义：第 68 行

### 2. FingerprintAuthDriverHdi（HDI 驱动适配器）

**文件路径**：
- 头文件：`services/inc/fingerprint_auth_driver_hdi.h`
- 实现：`services/src/fingerprint_auth_driver_hdi.cpp`

**职责**：
- 实现 UserAuth Framework 的 `IAuthDriverHdi` 接口
- 管理 HDI 执行器列表
- 处理执行器获取和回调注册
- 支持 All-in-One Executor 模式

**关键方法**：
```cpp
// 获取执行器列表
std::vector<std::shared_ptr<IAuthExecutorHdi>> GetExecutors();

// 注册 SA 命令处理器
void RegisterSaCommandCallback(const sptr<ISaCommandCallback> &callback);
```

**证据**：
- 文件：`services/src/fingerprint_auth_driver_hdi.cpp:47-93`

### 3. FingerprintAllInOneExecutorHdi（All-in-One 执行器）

**文件路径**：
- 头文件：`services/inc/fingerprint_auth_all_in_one_executor_hdi.h`
- 实现：`services/src/fingerprint_auth_all_in_one_executor_hdi.cpp`

**职责**：
- 实现 UserAuth Framework 的 `IAuthExecutorHdi` 接口
- 封装 HDI 的 `IAllInOneExecutor` 接口
- 提供所有执行器操作：Enroll、Authenticate、Identify、Delete、Cancel
- 处理 SA 命令（SendCommand）

**关键方法**：
```cpp
// services/src/fingerprint_auth_all_in_one_executor_hdi.cpp

ResultCode Enroll(uint64_t scheduleId, const std::vector<uint8_t> &extraInfo) override;      // 101-116 行
ResultCode Authenticate(uint64_t scheduleId, const std::vector<uint8_t> &extraInfo) override; // 118-134 行
ResultCode Identify(uint64_t scheduleId, const std::vector<uint8_t> &extraInfo) override;   // 136-151 行
ResultCode Delete(uint64_t scheduleId, const std::vector<uint64_t> &templateIds) override; // 153-163 行
ResultCode Cancel(uint64_t scheduleId) override;                                        // 165-175 行
ResultCode SendCommand(uint64_t commandId, const std::vector<uint8_t> &extraInfo);        // 177-198 行
```

**证据**：
- 文件：`services/src/fingerprint_auth_all_in_one_executor_hdi.cpp:101-198`

### 4. SaCommandManager（SA 命令管理器）

**文件路径**：
- 头文件：`services/inc/sa_command_manager.h`
- 实现：`services/src/sa_command_manager.cpp`

**职责**：
- 管理 SA 命令处理器
- 注册/注销命令处理器
- 分发来自 HDI 驱动的命令
- 支持多个命令处理器并存

**关键接口**：
```cpp
// services/inc/isa_command_processor.h
class ISaCommandProcessor {
public:
    virtual ~ISaCommandProcessor() = default;
    virtual ResultCode ProcessSaCommand(const SaCommand &command) = 0;
    virtual void OnHdiDisconnect() {}
};
```

**支持的命令处理器**：
- `SensorIlluminationManager`：处理传感器照明相关命令

**证据**：
- 文件：`services/inc/sa_command_manager.h`
- 实现：`services/src/sa_command_manager.cpp:20-98`

### 5. SensorIlluminationManager（传感器照明管理器）

**文件路径**：
- 头文件：`services/inc/sensor_illumination_manager.h`
- 实现：`services/src/sensor_illumination_manager.cpp`

**职责**：
- 实现 `ISaCommandProcessor` 接口
- 管理传感器照明状态
- 动态加载 `services_ex` 库
- 协调 `SensorIlluminationTask` 执行 UI 渲染

**支持的命令**：
```cpp
// 传感器照明命令
ENABLE_SENSOR_ILLUMINATION    // 启用照明
DISABLE_SENSOR_ILLUMINATION   // 禁用照明
TURN_ON_SENSOR_ILLUMINATION    // 打开照明
TURN_OFF_SENSOR_ILLUMINATION   // 关闭照明
```

**动态库加载**：
```cpp
// services/src/service_ex_manager.cpp:30-52
void ServiceExManager::Load()
{
    handle_ = dlopen("libfingerprintauthservice_ex.z.so", RTLD_NOW);
    auto createFunc = reinterpret_cast<CreateSensorIlluminationTaskFunc>(
        dlsym(handle_, "GetSensorIlluminationTask")
    );
    task_ = createFunc();
}
```

**证据**：
- 文件：`services/src/sensor_illumination_manager.cpp:1-206`
- 动态加载：`services/src/service_ex_manager.cpp:30-52`

---

## 数据流

### 1. 录入流程（Enroll）

```mermaid
flowchart TD
    A[UserAuth Framework<br/>验证权限] --> B[FingerprintAuthService<br/>接收 Enroll 请求]
    B --> C[DriverHdi<br/>获取执行器]
    C --> D[AllInOneExecutorHdi<br/>转换参数]
    D --> E[HDI Driver<br/>IAllInOneExecutor::Enroll]
    E --> F{指纹采集成功?}
    F -->|是| G[HDI 回调<br/>OnResult SUCCESS]
    F -->|否| H[HDI 回调<br/>OnResult FAIL/TIMEOUT]
    G --> I[ExecutorCallbackHdi<br/>转换结果]
    H --> I
    I --> J[DriverHdi<br/>转发回调]
    J --> K[UserAuth Framework<br/>处理结果]
    K --> L[返回给应用<br/>Promise resolve]

    style E fill:#ffecb3
    style K fill:#fff9c4
```

**关键代码路径**：
```
UserAuth Framework
  → services/src/fingerprint_auth_service.cpp:68 (SA 构造)
  → services/src/fingerprint_auth_driver_hdi.cpp:47 (获取执行器)
  → services/src/fingerprint_auth_all_in_one_executor_hdi.cpp:101 (Enroll 实现)
  → HDI Driver (IAllInOneExecutor::Enroll)
  → HDI Driver 回调 (IExecutorCallback::OnResult)
  → services/src/fingerprint_auth_executor_callback_hdi.cpp:16 (回调处理)
  → UserAuth Framework
```

**参数转换**：
```cpp
// services/src/fingerprint_auth_all_in_one_executor_hdi.cpp:101-116
ResultCode FingerprintAllInOneExecutorHdi::Enroll(
    uint64_t scheduleId,
    const std::vector<uint8_t> &extraInfo)
{
    // 转换为 HDI 参数格式
    EnrollParam param {
        .scheduleId = scheduleId,
        .extraInfo = extraInfo
    };

    // 调用 HDI 接口
    auto ret = executor_->Enroll(param, callback_);

    // 转换结果码
    return ConvertToResultCode(ret);
}
```

### 2. 认证流程（Authenticate）+ 传感器照明

```mermaid
flowchart TD
    A[UserAuth Framework<br/>验证权限] --> B[AllInOneExecutorHdi<br/>Authenticate]
    B --> C[HDI Driver<br/>Authenticate]
    C --> D[驱动触发<br/>SEND_COMMAND]
    D --> E[SaCommandManager<br/>接收命令]
    E --> F{命令类型?}
    F -->|TURN_ON_ILLUMINATION| G[SensorIlluminationManager<br/>ProcessSaCommand]
    F -->|TURN_OFF_ILLUMINATION| H[SensorIlluminationManager<br/>ProcessSaCommand]
    G --> I[SensorIlluminationTask<br/>渲染 UI]
    H --> J[SensorIlluminationTask<br/>关闭 UI]
    I --> K[HDI Driver<br/>继续认证]
    J --> K
    K --> L{认证结果?}
    L -->|SUCCESS| M[HDI 回调<br/>OnResult SUCCESS]
    L -->|FAIL| N[HDI 回调<br/>OnResult FAIL]
    M --> O[UserAuth Framework<br/>验证通过]
    N --> P[UserAuth Framework<br/>验证失败]

    style E fill:#ffecb3
    style I fill:#e1f5fe
```

**传感器照明时序**：
```mermaid
sequenceDiagram
    participant App as 应用
    participant UA as UserAuth Framework
    participant Exe as AllInOneExecutorHdi
    participant HDI as HDI Driver
    participant Cmd as SaCommandManager
    participant SIM as SensorIlluminationManager
    participant Task as SensorIlluminationTask

    App->>UA: authenticate()
    UA->>Exe: Authenticate(scheduleId, token)
    Exe->>HDI: Authenticate()
    HDI->>Cmd: SendCommand(TURN_ON_ILLUMINATION)
    Cmd->>SIM: ProcessSaCommand()
    SIM->>Task: TurnOn()
    Task-->>SIM: Done
    SIM-->>HDI: 回调
    HDI->>HDI: 执行指纹采集
    HDI-->>Exe: OnResult(SUCCESS/FAIL)
    HDI->>Cmd: SendCommand(TURN_OFF_ILLUMINATION)
    Cmd->>SIM: ProcessSaCommand()
    SIM->>Task: TurnOff()
    Task-->>SIM: Done
    SIM-->>HDI: 回调
    Exe-->>UA: 返回结果
```

### 3. 命令流（SA Command）

```mermaid
flowchart TD
    A[HDI Driver<br/>厂商自定义逻辑] --> B[发送 SA 命令<br/>IExecutorCallback::OnSaCommands]
    B --> C[ExecutorCallbackHdi<br/>接收命令]
    C --> D[转换命令格式<br/>HDI → Framework]
    D --> E[SaCommandManager<br/>ProcessSaCommands]
    E --> F[遍历命令处理器]
    F --> G[SensorIlluminationManager<br/>ProcessSaCommand]
    G --> H{命令类型?}
    H -->|ENABLE| I[EnableSensorIllumination]
    H -->|DISABLE| J[DisableSensorIllumination]
    H -->|TURN_ON| K[TurnOnSensorIllumination]
    H -->|TURN_OFF| L[TurnOffSensorIllumination]
    I --> M[设置参数<br/>centerX, centerY, radius]
    K --> N[SensorIlluminationTask<br/>渲染 UI]
    J --> O[释放资源]
    L --> O
    N --> P[返回结果码]
    O --> P

    style E fill:#e1f5fe
    style N fill:#ffecb3
```

**命令结构**（来自 HDI 定义）：
```cpp
// services/inc/fingerprint_auth_hdi.h (HDI 类型别名)
using SaCommand = OHOS::HDI::FingerprintAuth::V2_0::SaCommand;
using SaCommandId = OHOS::HDI::FingerprintAuth::V2_0::SaCommandId;

struct SaCommand {
    SaCommandId id;           // 命令 ID
    std::vector<uint8_t> payload;  // 命令载荷
};
```

---

## 线程模型

### 线程概览

| 线程类型 | 职责 | 创建时机 | 生命周期 |
|----------|------|----------|---------|
| **SA 主线程** | 服务生命周期管理、命令分发 | 系统启动 | 持久运行 |
| **HDF 回调线程池** | HDI 操作回调 | HDF 初始化 | 服务运行期间 |
| **事件订阅线程** | 屏幕状态事件处理 | `ScreenStateMonitor` 初始化 | 服务运行期间 |
| **UI 渲染线程** | 传感器照明 UI 渲染 | Rosen 渲染服务 | 渲染任务期间 |

### 线程交互图

```mermaid
sequenceDiagram
    participant Main as SA 主线程
    participant HDF as HDF 线程池
    participant Event as 事件订阅线程
    participant UI as UI 渲染线程

    Note over Main,UI: 服务启动
    Main->>Main: OnStart()
    Main->>Event: 订阅屏幕事件

    Note over Main,UI: 录入流程
    Main->>HDF: Enroll() 调用
    HDF->>HDF: 异步执行
    Main->>Main: 继续处理其他请求
    HDF-->>Main: OnResult() 回调 (异步)

    Note over Main,UI: 认证 + 照明
    Main->>HDF: Authenticate()
    HDF-->>Main: OnSaCommands(TURN_ON_ILLUMINATION)
    Main->>Main: ProcessSaCommands()
    Main->>UI: TurnOn() (异步渲染)
    HDF->>HDF: 执行指纹采集
    HDF-->>Main: OnResult(SUCCESS)
    HDF-->>Main: OnSaCommands(TURN_OFF_ILLUMINATION)
    Main->>Main: ProcessSaCommands()
    Main->>UI: TurnOff()

    Note over Main,UI: 屏幕事件
    Event->>Event: COMMON_EVENT_SCREEN_ON
    Event->>Main: 处理屏幕亮起
    Event->>Event: COMMON_EVENT_SCREEN_OFF
    Event->>Main: 处理屏幕关闭
```

### 线程安全

| 资源 | 保护机制 | 证据 |
|--------|---------|------|
| `FingerprintAuthService` 单例 | `std::mutex mutex_` | `services/src/fingerprint_auth_service.cpp:39` |
| SensorIlluminationManager 执行器映射 | `std::map` + `std::mutex` | `services/src/sensor_illumination_manager.cpp:107-114` |
| ServiceExManager 动态库句柄 | `std::mutex` + RAII | `services/src/service_ex_manager.cpp:1-79` |
| HDF 回调 | 通过 HDF 线程池异步执行 | HDI 框架保证 |

---

## 扩展机制

### 1. SA 命令扩展

**原理**：通过 `SaCommandManager` 注册自定义命令处理器。

**扩展步骤**：

1. **定义命令 ID**（在 HDI 接口中）：
```cpp
// drivers_interface/fingerprint_auth/idl/*.idl
enum SaCommandId {
    VENDOR_COMMAND_BEGIN = 1000,
    VENDOR_CUSTOM_CMD_1 = 1001,
    VENDOR_CUSTOM_CMD_2 = 1002
};
```

2. **实现命令处理器**：
```cpp
// services/inc/my_vendor_processor.h
class MyVendorProcessor : public ISaCommandProcessor {
public:
    ResultCode ProcessSaCommand(const SaCommand &command) override {
        if (command.id == VENDOR_CUSTOM_CMD_1) {
            // 处理自定义命令
            return SUCCESS;
        }
        return GENERAL_ERROR;
    }
};
```

3. **注册处理器**：
```cpp
// services/src/fingerprint_auth_service.cpp
auto myProcessor = std::make_shared<MyVendorProcessor>();
SaCommandManager::GetInstance().RegisterProcessor(myProcessor);
```

### 2. 动态库扩展（services_ex）

**原理**：通过 `ServiceExManager` 动态加载扩展库。

**扩展库要求**：

1. **导出工厂函数**：
```cpp
// services_ex/src/sensor_illumination_task.cpp
extern "C" {
    ISensorIlluminationTask *GetSensorIlluminationTask()
    {
        return new SensorIlluminationTask();
    }
}
```

2. **定义符号可见性**：
```
// services_ex/fingerprint_auth_service_ex_map
{
    global:
        GetSensorIlluminationTask*;
    local:
        *;
};
```

3. **加载扩展库**：
```cpp
// services/src/service_ex_manager.cpp
void ServiceExManager::Load()
{
    handle_ = dlopen("libfingerprintauthservice_ex.z.so", RTLD_NOW);
    auto createFunc = reinterpret_cast<CreateSensorIlluminationTaskFunc>(
        dlsym(handle_, "GetSensorIlluminationTask")
    );
    task_ = createFunc();
}
```

---

## 性能考量

### 1. 异步操作

**设计原则**：所有 HDI 操作都是异步的，避免阻塞 SA 主线程。

**证据**：
- HDI 回调通过 HDF 线程池执行：`services/src/fingerprint_auth_executor_callback_hdi.cpp:16-141`
- 用户认证流程不阻塞框架：使用 `OnResult()` 回调返回结果

### 2. 内存管理

**智能指针使用**：
- 使用 `std::shared_ptr` 管理执行器实例：`services/src/fingerprint_auth_driver_hdi.cpp:47-93`
- 使用 `sptr`（Strong Pointer）管理 HDI 接口：HDF 框架要求

**内存保护**：
- `MemoryGuard` RAII 包装器：`services/inc/memory_guard.h`
- CFI（Control Flow Integrity）启用：`services/BUILD.gn:34`

### 3. 错误处理

**防御性编程**：
- 参数检查宏：`common/utils/iam_check.h`
- 空指针检查：所有公共方法入口

**错误码转换**：
- HDI 错误码转换为框架标准错误码：`services/src/fingerprint_auth_all_in_one_executor_hdi.cpp:101-198`

---

## 相关跳转

- [01_Overview.md](./01_Overview.md) - 组件全貌
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [04_HDI_Interfaces.md](./04_HDI_Interfaces.md) - HDI 接口详解
- [05_Internal_APIs.md](./05_Internal_APIs.md) - 内部 API 详解
