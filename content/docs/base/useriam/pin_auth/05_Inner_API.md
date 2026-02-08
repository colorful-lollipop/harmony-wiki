# 内部 API 文档

> **目的**：了解 pin_auth 模块内部模块的接口、依赖关系和稳定性
> **适用范围**：系统开发者、架构师、模块维护者
> **关键结论**：模块间依赖清晰，无循环依赖，接口相对稳定
> **相关文档**：[概览](index.md) | [架构设计](03_Architecture.md) | [对外 API](04_Native_API.md)

---

## 模块接口定义

### 1. SA 核心模块

#### PinAuthService

**文件**：
- 头文件：`services/sa/inc/pin_auth_service.h`
- 实现：`services/sa/src/pin_auth_service.cpp`

**接口**：

| 方法 | 签名 | 说明 | 稳定性 |
|------|------|------|--------|
| GetInstance | `static std::shared_ptr<PinAuthService> GetInstance()` | 获取单例 | 稳定 |
| RegisterInputer | `bool RegisterInputer(const sptr<InputerGetData> &inputer)` | 注册 Inputer | 稳定 |
| UnRegisterInputer | `void UnRegisterInputer()` | 注销 Inputer | 稳定 |
| CheckPermission | `bool CheckPermission(const std::string &permission)` | 权限检查 | 稳定 |
| OnStart | `void OnStart() override` | SA 启动 | 稳定 |
| OnStop | `void OnStop() override` | SA 停止 | 稳定 |

**依赖**：
- `PinAuthStub` - IPC 接口
- `PinAuthManager` - Inputer 管理
- `PinAuthDriverHdi` - HDI 驱动
- `LoadModeHandler` - 加载模式

**代码证据**：
- `services/sa/inc/pin_auth_service.h:31` - 类定义

---

### 2. Inputer 管理模块

#### PinAuthManager

**文件**：
- 头文件：`services/modules/inputters/inc/pin_auth_manager.h`
- 实现：`services/modules/inputters/src/pin_auth_manager.cpp`

**接口**：

| 方法 | 签名 | 说明 | 稳定性 |
|------|------|------|--------|
| GetInstance | `static PinAuthManager &GetInstance()` | 获取单例 | 稳定 |
| RegisterInputer | `bool RegisterInputer(uint32_t tokenId, const sptr<InputerGetData> &inputer)` | 注册 Inputer（带 Token 隔离） | 稳定 |
| UnRegisterInputer | `void UnRegisterInputer()` | 注销当前 Token 的 Inputer | 稳定 |
| GetInputer | `sptr<InputerGetData> GetInputer(uint32_t tokenId)` | 根据 Token ID 获取 Inputer | 稳定 |

**数据结构**：
```cpp
std::map<uint32_t, sptr<InputerGetData>> pinAuthInputerMap_;
std::mutex mapMutex_;
```

**依赖**：
- `InputerGetData` - IPC 接口
- 无其他模块依赖（独立）

**代码证据**：
- `services/modules/inputters/inc/pin_auth_manager.h:25-33` - 接口定义

---

### 3. HDI 驱动模块

#### PinAuthDriverHdi

**文件**：
- 头文件：`services/modules/driver/inc/pin_auth_driver_hdi.h`
- 实现：`services/modules/driver/src/pin_auth_driver_hdi.cpp`

**接口**：

| 方法 | 签名 | 说明 | 稳定性 |
|------|------|------|--------|
| Start | `void Start()` | 启动 HDI 驱动 | 稳定 |
| Stop | `void Stop()` | 停止 HDI 驱动 | 稳定 |
| GetExecutor | `std::shared_ptr<IExecutor> GetExecutor(uint16_t authType)` | 获取执行器 | 稳定 |
| GetExecutorList | `std::vector<std::shared_ptr<IExecutor>> GetExecutorList()` | 获取执行器列表 | 稳定 |

**依赖**：
- `IPinAuthInterface` - HDI 接口（来自 `drivers_interface_pin_auth`）
- `PinAuthInterfaceAdapter` - HDI 适配器

**代码证据**：
- `services/modules/driver/inc/pin_auth_driver_hdi.h:28-46` - 接口定义

#### PinAuthInterfaceAdapter

**文件**：
- 头文件：`services/modules/driver/inc/pin_auth_interface_adapter.h`
- 实现：`services/modules/driver/src/pin_auth_interface_adapter.cpp`

**职责**：HDI 接口的适配层，将 HDI 类型转换为内部类型

**接口**：（TODO：需要详细分析）

**代码证据**：
- `services/modules/driver/inc/pin_auth_interface_adapter.h:24` - 类定义

---

### 4. 执行器模块

#### PinAuthAllInOneHdi

**文件**：
- 头文件：`services/modules/executors/inc/pin_auth_all_in_one_hdi.h`
- 实现：`services/modules/executors/src/pin_auth_all_in_one_hdi.cpp`

**接口**：

| 方法 | 签名 | 说明 | 稳定性 |
|------|------|------|--------|
| Begin | `int32_t Begin(uint64_t scheduleId, const ExecutorInfo &info, const std::vector<uint8_t> &authToken)` | 开始认证会话 | 稳定 |
| Cancel | `int32_t Cancel(uint64_t scheduleId)` | 取消认证会话 | 稳定 |
| Delete | `int32_t Delete(uint64_t scheduleId, const std::vector<uint8_t> &authToken)` | 删除凭证 | 稳定 |

**依赖**：
- `IAllInOneExecutor` - HDI 接口
- `IExecutorCallbackHdi` - 执行器回调

**代码证据**：
- `services/modules/executors/inc/pin_auth_all_in_one_hdi.h:28-42` - 接口定义

#### PinAuthCollectorHdi

**文件**：
- 头文件：`services/modules/executors/inc/pin_auth_collector_hdi.h`
- 实现：`services/modules/executors/src/pin_auth_collector_hdi.cpp`

**接口**：（TODO：需要详细分析）

**代码证据**：
- `services/modules/executors/inc/pin_auth_collector_hdi.h:28` - 类定义

#### PinAuthVerifierHdi

**文件**：
- 头文件：`services/modules/executors/inc/pin_auth_verifier_hdi.h`
- 实现：`services/modules/executors/src/pin_auth_verifier_hdi.cpp`

**接口**：（TODO：需要详细分析）

**代码证据**：
- `services/modules/executors/inc/pin_auth_verifier_hdi.h:28` - 类定义

#### IExecutorCallbackHdi

**文件**：
- 头文件：`services/modules/executors/inc/pin_auth_executor_callback_hdi.h`
- 实现：`services/modules/executors/src/pin_auth_executor_callback_hdi.cpp`

**接口**：

| 方法 | 签名 | 说明 | 稳定性 |
|------|------|------|--------|
| OnGetData | `int32_t OnGetData(uint64_t scheduleId, const std::vector<uint8_t> &authToken)` | HDI 回调：请求数据 | 稳定 |
| OnResult | `int32_t OnResult(uint64_t scheduleId, int32_t result, const std::vector<uint8_t> &extraInfo)` | HDI 回调：返回结果 | 稳定 |

**关键成员**：
```cpp
uint32_t tokenId_;      // 用于 Inputer 查找
uint64_t scheduleId_;   // 会话 ID
```

**代码证据**：
- `services/modules/executors/inc/pin_auth_executor_callback_hdi.h:32-45` - 接口定义

---

### 5. 加载模式模块

#### LoadModeHandler

**文件**：
- 头文件：`services/modules/load_mode/inc/load_mode_handler.h`
- 实现：`services/modules/load_mode/src/load_mode_handler.cpp`

**接口**：

| 方法 | 签名 | 说明 | 稳定性 |
|------|------|------|--------|
| Start | `virtual void Start() = 0` | 启动加载模式 | 稳定 |
| Stop | `virtual void Stop() = 0` | 停止加载模式 | 稳定 |

**子类**：
- `LoadModeHandlerDefault` - 静态加载
- `LoadModeHandlerDynamic` - 动态加载

**代码证据**：
- `services/modules/load_mode/inc/load_mode_handler.h:31-38` - 基类定义

#### DriverLoadManager

**文件**：
- 头文件：`services/modules/load_mode/inc/driver_load_manager.h`
- 实现：`services/modules/load_mode/src/driver_load_manager.cpp`

**接口**：（TODO：需要详细分析）

**代码证据**：
- `services/modules/load_mode/inc/driver_load_manager.h:30` - 类定义

#### SystemAbilityListener

**文件**：
- 头文件：`services/modules/load_mode/inc/system_ability_listener.h`
- 实现：`services/modules/load_mode/src/system_ability_listener.cpp`

**接口**：（TODO：需要详细分析）

**代码证据**：
- `services/modules/load_mode/inc/system_ability_listener.h:27` - 类定义

#### SystemParamManager

**文件**：
- 头文件：`services/modules/load_mode/inc/system_param_manager.h`
- 实现：`services/modules/load_mode/src/system_param_manager.cpp`

**接口**：（TODO：需要详细分析）

**代码证据**：
- `services/modules/load_mode/inc/system_param_manager.h:28` - 类定义

---

## 依赖关系图

### 模块依赖层次

```
┌─────────────────────────────────────┐
│     PinAuthService (SA 核心)         │
│  依赖：                          │
│  - PinAuthStub                  │
│  - PinAuthManager                │
│  - PinAuthDriverHdi              │
│  - LoadModeHandler               │
└──────────┬──────────────────────┘
           │
    ┌──────┴──────┬──────────────┐
    │             │              │
    ↓             ↓              ↓
PinAuthManager  PinAuthDriverHdi  LoadModeHandler
  依赖           依赖            依赖
  ↓              ↓              ↓
InputerGetData  IPinAuthInterface  子类实现
                ↓
          PinAuthInterfaceAdapter
                ↓
          IAllInOneExecutor 等
                ↓
          IExecutorCallbackHdi
                ↓
          PinAuthManager（回调）
```

### 依赖方向（无环）

| 模块 | 依赖 | 级别 |
|------|------|------|
| PinAuthService | PinAuthStub、PinAuthManager、PinAuthDriverHdi、LoadModeHandler | Level 1 |
| PinAuthManager | InputerGetData | Level 2 |
| PinAuthDriverHdi | IPinAuthInterface、PinAuthInterfaceAdapter | Level 2 |
| IExecutorCallbackHdi | PinAuthManager（通过回调） | Level 3 |
| LoadModeHandler | DriverLoadManager、SystemAbilityListener、SystemParamManager | Level 2 |

**结论**：依赖关系清晰，无循环依赖。

---

## 稳定性标注

### 稳定接口

这些接口已稳定，不建议修改：

1. **PinAuthService 公共接口**
   - 位置：`services/sa/inc/pin_auth_service.h`
   - 依据：继承自 SystemAbility 和 PinAuthStub（OpenHarmony 标准接口）
   - 变更影响：破坏与 user_auth_framework 和应用的兼容性

2. **PinAuthManager 接口**
   - 位置：`services/modules/inputters/inc/pin_auth_manager.h`
   - 依据：被多个执行器回调引用
   - 变更影响：所有认证流程

3. **HDI 执行器接口**
   - 位置：`services/modules/executors/inc/*.h`
   - 依据：对应 `drivers_interface_pin_auth` 接口
   - 变更影响：南向厂商适配

4. **公共 API 接口**
   - 位置：`interfaces/inner_api/*.h`
   - 依据：作为 Inner Kits 暴露到子系统
   - 变更影响：所有集成方（Settings、锁屏等）

### 不稳定接口

这些接口可能随内部实现变化：

1. **LoadModeHandler 及其子类**
   - 位置：`services/modules/load_mode/inc/*.h`
   - 依据：加载模式策略可能调整
   - 变更影响：仅影响内部加载逻辑

2. **PinAuthInterfaceAdapter**
   - 位置：`services/modules/driver/inc/pin_auth_interface_adapter.h`
   - 依据：适配器实现可能随 HDI 版本更新
   - 变更影响：仅影响驱动交互层

3. **HDI 类型别名**
   - 位置：`services/modules/common/inc/pin_auth_hdi.h`
   - 依据：依赖 `drivers_interface_pin_auth` 版本
   - 变更影响：HDI 版本升级时需同步更新

---

## 可替换点

### 1. 加载模式策略

**位置**：`services/modules/load_mode/`

**当前实现**：
- 静态加载（默认）
- 动态加载（可选）

**可扩展点**：
- 添加新的加载模式（如条件加载、延迟加载）
- 实现 `LoadModeHandler` 接口

**证据**：
- `services/modules/load_mode/inc/load_mode_handler.h:31-38` - 虚基类定义

---

### 2. HDI 执行器类型

**位置**：`services/modules/executors/`

**当前实现**：
- AllInOne - 全功能
- Collector - 收集器
- Verifier - 验证器

**可扩展点**：
- 添加新的执行器类型
- 实现 `IExecutor` 接口

**证据**：
- `services/modules/common/inc/pin_auth_hdi.h` - HDI 类型定义

---

### 3. 死亡恢复策略

**位置**：
- 客户端：`frameworks/client/src/pinauth_register_impl.cpp`
- 服务端：`services/modules/inputters/src/pin_auth_manager.cpp`

**当前实现**：
- 自动清理
- 需要手动重连

**可扩展点**：
- 添加自动重连机制
- 添加指数退避重试

**证据**：
- `frameworks/client/src/pinauth_register_impl.cpp` - Death Recipient 实现
- `services/modules/inputters/src/pin_auth_manager.cpp:40-48` - Death Recipient 添加

---

## 内部通信机制

### IPC 通信

**接口对**：

| 接口 | Proxy | Stub | 代码位置 |
|------|--------|------|---------|
| PinAuthInterface | `frameworks/ipc/src/pin_auth_proxy.cpp` | `frameworks/ipc/src/pin_auth_stub.cpp` | `frameworks/ipc/src/` |
| InputerGetData | `frameworks/ipc/src/inputer_get_data_proxy.cpp` | `frameworks/ipc/src/inputer_get_data_stub.cpp` | `frameworks/ipc/src/` |
| InputerSetData | `frameworks/ipc/src/inputer_set_data_proxy.cpp` | `frameworks/ipc/src/inputer_set_data_stub.cpp` | `frameworks/ipc/src/` |

**调用流程**：

```
客户端 Proxy
    ↓ (IPC Binder)
服务端 Stub
    ↓ (OnRemoteRequest)
具体实现
```

### 回调机制

**方向**：

```
SA → 应用：OnGetData()（通过 InputerGetData IPC）
应用 → SA：OnSetData()（通过 InputerSetData IPC）
```

---

## 代码证据索引

| 模块 | 头文件 | 实现文件 |
|------|---------|----------|
| SA 核心 | `services/sa/inc/pin_auth_service.h` | `services/sa/src/pin_auth_service.cpp` |
| Inputer 管理 | `services/modules/inputters/inc/pin_auth_manager.h` | `services/modules/inputters/src/pin_auth_manager.cpp` |
| HDI 驱动 | `services/modules/driver/inc/pin_auth_driver_hdi.h` | `services/modules/driver/src/pin_auth_driver_hdi.cpp` |
| 全功能执行器 | `services/modules/executors/inc/pin_auth_all_in_one_hdi.h` | `services/modules/executors/src/pin_auth_all_in_one_hdi.cpp` |
| 执行器回调 | `services/modules/executors/inc/pin_auth_executor_callback_hdi.h` | `services/modules/executors/src/pin_auth_executor_callback_hdi.cpp` |
| 加载模式 | `services/modules/load_mode/inc/load_mode_handler.h` | `services/modules/load_mode/src/load_mode_handler.cpp` |
| 驱动适配器 | `services/modules/driver/inc/pin_auth_interface_adapter.h` | `services/modules/driver/src/pin_auth_interface_adapter.cpp` |

---

## 下一步

- 了解构建系统 → [GN Targets](06_GN_Targets.md)
- 了解编译产物 → [编译产物](07_Build_Artifacts.md)
