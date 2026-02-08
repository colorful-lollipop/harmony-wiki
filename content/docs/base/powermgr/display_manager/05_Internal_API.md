# 内部 API - display_manager

> 本文档说明 display_manager 模块的内部 API 接口，包括 IPC 接口、Client API 和回调接口

---

## 文档目的

本文档提供：
- IPC 接口的完整定义和方法说明
- Client API 的使用方法和调用约定
- 回调接口的注册和使用方式
- 接口稳定性评估

## 适用范围

- **适用对象**：系统服务开发者、需要调用 display_manager 的 Native 模块开发者
- **前置知识**：熟悉 ZIDL、IPC 通信、OpenHarmony System Ability

---

## IPC 接口

### IDisplayPowerMgr（主服务接口）

**定义文件**：`state_manager/service/IDisplayPowerMgr.idl`

**接口说明**：display_manager 的核心 IPC 接口，通过 ZIDL 工具生成 Proxy/Stub 代码，用于跨进程调用。

**继承关系**：
```
IRemoteBroker (OpenHarmony IPC 基类)
    └── IDisplayPowerMgr (IDL 定义接口)
```

#### 显示状态管理方法

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `SetDisplayState` | id, state, reason | bResult | 设置显示状态（ON/OFF） |
| `GetDisplayState` | id | displayState | 获取显示状态 |
| `GetDisplayIds` | - | ids[] | 获取所有显示器 ID |
| `GetMainDisplayId` | - | id | 获取主显示器 ID |
| `SetScreenDisplayState` | screenId, state, reason | - | 设置屏幕显示状态 |
| `SetScreenPowerOffStrategy` | strategy, reason, token | retCode | 设置关屏策略 |
| `NotifyScreenPowerStatus` | displayId, powerStatus | retCode | 通知屏幕电源状态 |

**代码位置**：`IDisplayPowerMgr.idl:23-27, 59-65`

---

#### 亮度控制方法

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `SetBrightness` | value, displayId, continuous | bResult, retCode | 设置亮度 |
| `GetBrightness` | displayId | brightness | 获取当前亮度 |
| `GetDefaultBrightness` | - | defaultBrightness | 获取默认亮度 |
| `GetMaxBrightness` | - | maxBrightness | 获取最大亮度 |
| `GetMinBrightness` | - | minBrightness | 获取最小亮度 |
| `AdjustBrightness` | id, value, duration | bResult | 调整亮度（相对值） |
| `DiscountBrightness` | discount, displayId | bResult | 折扣亮度 |
| `OverrideBrightness` | value, displayId, duration | bResult | 临时覆盖亮度 |
| `RestoreBrightness` | displayId, duration | bResult | 恢复亮度 |
| `SetMaxBrightness` | value, enterTestMode | bResult, retCode | 设置最大亮度 |
| `SetMaxBrightnessNit` | maxNit, enterTestMode | bResult, retCode | 设置最大亮度（nit） |
| `SetCoordinated` | coordinated, displayId | bResult | 设置协调模式 |
| `SetScreenOnBrightness` | - | bResult | 设置亮屏亮度 |

**代码位置**：`IDisplayPowerMgr.idl:28-44`

---

#### 自动亮度方法

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `AutoAdjustBrightness` | enable | bResult | 开关自动亮度 |
| `IsAutoAdjustBrightness` | - | bResult | 查询自动亮度状态 |
| `SetLightBrightnessThreshold` | threshold[], callback | retCode | 设置亮度阈值 |

**代码位置**：`IDisplayPowerMgr.idl:40-41, 47-48`

---

#### 亮度增强方法

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `BoostBrightness` | timeoutMs, displayId | bResult | 临时提升亮度 |
| `CancelBoostBrightness` | displayId | bResult | 取消亮度提升 |
| `GetDeviceBrightness` | displayId, useHbm | deviceBrightness | 获取设备亮度 |
| `OverrideDisplayOffDelay` | delayMs | bResult | 覆盖关屏延迟 |
| `WaitDimmingDone` | - | - | 等待渐变完成 |

**代码位置**：`IDisplayPowerMgr.idl:43-45, 62`

---

#### 回调注册方法

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `RegisterCallback` | callback | bResult | 注册电源回调 |
| `RegisterDataChangeListener` | listener, type, callerId, params | retCode | 注册数据监听 |
| `UnregisterDataChangeListener` | type, callerId | retCode | 注销数据监听 |

**代码位置**：`IDisplayPowerMgr.idl:42, 50-54`

---

#### 调试方法

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `RunJsonCommand` | request | strResult | 执行 JSON 调试命令 |

**代码位置**：`IDisplayPowerMgr.idl:49`

---

### 回调接口定义

#### IDisplayPowerCallback（电源状态回调）

**定义文件**：`state_manager/interfaces/inner_api/native/include/idisplay_power_callback.h`

```cpp
class IDisplayPowerCallback : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.DisplayPowerMgr.IDisplayPowerCallback");
    
    virtual void OnDisplayStateChanged(uint32_t displayId, DisplayState state, 
                                       uint32_t reason) = 0;
};
```

**使用场景**：监听屏幕开关状态变化

**注册方法**：`DisplayPowerMgrClient::RegisterCallback()`

---

#### IDisplayBrightnessCallback（亮度回调）

**定义文件**：`state_manager/interfaces/inner_api/native/include/idisplay_brightness_callback.h`

```cpp
class IDisplayBrightnessCallback : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.DisplayPowerMgr.IDisplayBrightnessCallback");
    
    virtual void OnLightBrightnessChanged(uint32_t brightnessLevel, 
                                          uint32_t brightnessValue) = 0;
};
```

**使用场景**：监听亮度阈值触发事件

**注册方法**：`DisplayPowerMgrClient::SetLightBrightnessThreshold()`

---

#### IDisplayBrightnessListener（亮度数据监听）

**定义文件**：`state_manager/interfaces/inner_api/native/include/idisplay_brightness_listener.h`

```cpp
class IDisplayBrightnessListener : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.DisplayPowerMgr.IDisplayBrightnessListener");
    
    virtual void OnDataChanged(const std::string& params) = 0;
};
```

**使用场景**：监听亮度数据变化（APS 集成、UI 实时更新）

**监听类型**（`DisplayDataChangeListenerType`）：
- `LIGHT_OR_BRIGHTNESS_FOR_APS` - 通知 APS 光或亮度变化
- `STABLE_LUX` - 通知稳定环境光变化
- `FORCE_EXIT_OVERRIDDEN_MODE` - 通知强制退出覆盖状态
- `BRIGHTNESS_FOR_UI` - 通知所有亮度更新（包含渐变）
- `BRIGHTNESS_TARGET` - 仅通知最终目标亮度变化

**注册方法**：`DisplayPowerMgrClient::RegisterDataChangeListener()`

---

## Client API

### DisplayPowerMgrClient（客户端单例）

**定义文件**：`state_manager/interfaces/inner_api/native/include/display_power_mgr_client.h`

**实现文件**：`state_manager/frameworks/native/display_power_mgr_client.cpp`

**设计模式**：DelayedRefSingleton 延迟单例

```cpp
class DisplayPowerMgrClient : public DelayedRefSingleton<DisplayPowerMgrClient> {
    DECLARE_DELAYED_REF_SINGLETON(DisplayPowerMgrClient);
public:
    // 显示状态
    bool SetScreenDisplayState(uint64_t screenId, DisplayState status, uint32_t reason);
    bool SetDisplayState(DisplayState state, PowerMgr::StateChangeReason reason = ..., uint32_t id = 0);
    DisplayState GetDisplayState(uint32_t id = 0);
    std::vector<uint32_t> GetDisplayIds();
    int32_t GetMainDisplayId();
    
    // 亮度控制
    bool SetBrightness(uint32_t value, uint32_t displayId = 0, bool continuous = false);
    bool SetMaxBrightness(double value, uint32_t enterTestMode = 0);
    bool DiscountBrightness(double discount, uint32_t displayId = 0);
    bool OverrideBrightness(uint32_t value, uint32_t displayId = 0, uint32_t duration = 500);
    bool RestoreBrightness(uint32_t displayId = 0, uint32_t duration = 500);
    uint32_t GetBrightness(uint32_t displayId = 0);
    uint32_t GetDefaultBrightness();
    uint32_t GetMaxBrightness();
    uint32_t GetMinBrightness();
    bool AdjustBrightness(uint32_t value, uint32_t duration, uint32_t id = 0);
    
    // 自动亮度
    bool AutoAdjustBrightness(bool enable);
    bool IsAutoAdjustBrightness();
    
    // 回调注册
    bool RegisterCallback(sptr<IDisplayPowerCallback> callback);
    int32_t RegisterDataChangeListener(const sptr<IDisplayBrightnessListener>& listener, 
                                       DisplayDataChangeListenerType listenerType, 
                                       const std::string& callerId = "", 
                                       const std::string& params = "");
    int32_t UnregisterDataChangeListener(DisplayDataChangeListenerType listenerType, 
                                         const std::string& callerId = "");
    uint32_t SetLightBrightnessThreshold(std::vector<int32_t> threshold, 
                                          sptr<IDisplayBrightnessCallback> callback);
    
    // 亮度增强
    bool BoostBrightness(int32_t timeoutMs, uint32_t displayId = 0);
    bool CancelBoostBrightness(uint32_t displayId = 0);
    uint32_t GetDeviceBrightness(uint32_t displayId = 0, bool useHbm = false);
};
```

---

#### 常量定义

```cpp
static constexpr int32_t INVALID_DISPLAY_ID {-1};
static constexpr int32_t DEFAULT_MAIN_DISPLAY_ID {0};
static constexpr uint32_t BRIGHTNESS_OFF {0};
static constexpr uint32_t BRIGHTNESS_DEFAULT {102};
static constexpr uint32_t BRIGHTNESS_MAX {255};
static constexpr uint32_t BRIGHTNESS_MIN {1};
```

---

#### 错误码

**定义文件**：`state_manager/interfaces/inner_api/native/include/display_mgr_errors.h`

| 错误码 | 值 | 说明 |
|--------|------|------|
| `ERR_OK` | 0 | 成功 |
| `ERR_FAILED` | 4700100 | 通用失败 |
| `ERR_CONNECTION_FAIL` | 4700101 | 服务连接失败 |
| `ERR_PERMISSION_DENIED` | 201 | 权限被拒绝 |
| `ERR_SYSTEM_API_DENIED` | 202 | 系统 API 被拒绝 |
| `ERR_PARAM_INVALID` | 401 | 参数无效 |

---

#### 使用示例

```cpp
#include "display_power_mgr_client.h"

using namespace OHOS::DisplayPowerMgr;

// 设置亮度
bool success = DisplayPowerMgrClient::GetInstance().SetBrightness(128);

// 获取亮度
uint32_t brightness = DisplayPowerMgrClient::GetInstance().GetBrightness();

// 注册显示状态回调
class MyDisplayCallback : public IDisplayPowerCallback {
public:
    void OnDisplayStateChanged(uint32_t displayId, DisplayState state, uint32_t reason) override {
        // 处理显示状态变化
    }
};

sptr<IDisplayPowerCallback> callback = new MyDisplayCallback();
DisplayPowerMgrClient::GetInstance().RegisterCallback(callback);
```

---

## 内部模块接口

### BrightnessManager（亮度管理器）

**定义文件**：`brightness_manager/include/brightness_manager.h`

**说明**：全局亮度管理单例，提供亮度控制的核心接口。

```cpp
class BrightnessManager {
public:
    static BrightnessManager& Get();
    
    // 生命周期
    void Init(uint32_t defaultMax, uint32_t defaultMin);
    void DeInit();
    
    // 显示状态
    void SetDisplayState(uint32_t id, DisplayState state, uint32_t reason);
    DisplayState GetState();
    
    // 自动亮度
    bool IsSupportLightSensor(void);
    bool IsAutoAdjustBrightness(void);
    bool AutoAdjustBrightness(bool enable);
    
    // 亮度控制
    bool SetBrightness(uint32_t value, uint32_t gradualDuration = 0, bool continuous = false);
    bool DiscountBrightness(double discount);
    double GetDiscount() const;
    bool OverrideBrightness(uint32_t value, uint32_t gradualDuration = 0);
    bool RestoreBrightness(uint32_t gradualDuration = 0);
    bool BoostBrightness(uint32_t timeoutMs, uint32_t gradualDuration = 0);
    bool CancelBoostBrightness(uint32_t gradualDuration = 0);
    uint32_t GetBrightness();
    uint32_t GetDeviceBrightness(bool useHbm = false);
    void WaitDimmingDone() const;
    
    // 监听注册
    int32_t RegisterDataChangeListener(...);
    int32_t UnregisterDataChangeListener(...);
    uint32_t SetLightBrightnessThreshold(...);
    
    // 最大亮度
    bool SetMaxBrightness(double value);
    bool SetMaxBrightnessNit(uint32_t nit);
};
```

---

### ScreenController（屏幕控制器）

**定义文件**：`state_manager/service/native/include/screen_controller.h`

**说明**：每显示器的屏幕状态和亮度控制器。

```cpp
class ScreenController {
public:
    // 状态管理
    DisplayState GetState();
    DisplayState SetDelayOffState();
    DisplayState SetOnState();
    bool UpdateState(DisplayState state, uint32_t reason);
    bool IsScreenOn();
    
    // 亮度控制
    bool SetBrightness(uint32_t value, uint32_t gradualDuration = 0, bool continuous = false);
    uint32_t GetBrightness();
    uint32_t GetDeviceBrightness();
    bool DiscountBrightness(double discount, uint32_t gradualDuration = 0);
    bool OverrideBrightness(uint32_t value, uint32_t gradualDuration = 0);
    bool RestoreBrightness(uint32_t gradualDuration = 0);
    bool BoostBrightness(uint32_t timeoutMs, uint32_t gradualDuration = 0);
    bool CancelBoostBrightness(uint32_t gradualDuration = 0);
    bool IsBrightnessOverridden() const;
    bool IsBrightnessBoosted() const;
    void SetCoordinated(bool coordinated);
};
```

---

## 接口稳定性

### 稳定接口（推荐使用的公共 API）

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `DisplayPowerMgrClient` | 稳定 | 客户端 API，对外暴露，版本兼容保证 |
| `IDisplayPowerMgr` | 稳定 | IPC 接口，IDL 定义，版本控制 |
| `IDisplayPowerCallback` | 稳定 | 回调接口，IDL 定义 |
| `IDisplayBrightnessListener` | 稳定 | 监听接口，IDL 定义 |

### 内部接口（可能变化的实现细节）

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `BrightnessManager` | 中等 | 静态库内部接口，可能随版本变化 |
| `ScreenController` | 中等 | 服务内部实现，不推荐外部依赖 |
| `BrightnessService` | 低 | 内部实现细节，频繁变化 |
| `CalculationManager` | 低 | 亮度计算算法，可能优化调整 |

---

## 接口版本兼容性

### 版本信息

- **当前版本**：3.1（bundle.json）
- **API 级别**：System API（仅系统应用可用）
- **兼容性策略**：IDL 接口向后兼容，Client API 版本控制

### 变更历史

| 版本 | 变更 | 影响 |
|------|------|------|
| 3.1 | 新增 ETS/ANI 接口 | 新增 @ohos.brightness |
| 3.0 | 新增数据监听接口 | RegisterDataChangeListener |
| 2.0 | 新增亮度增强接口 | BoostBrightness, OverrideBrightness |

---

## 相关链接

- **架构文档**：[03_Architecture.md](03_Architecture.md)
- **N-API 文档**：[04_NAPI_Interface.md](04_NAPI_Interface.md)
- **GN 构建**：[06_GN_Targets.md](06_GN_Targets.md)
- **安全分析**：[08_Security_Analysis.md](08_Security_Analysis.md)

---

## 文档更新记录

- **2026-02-07**：初始版本 v1.0，基于代码扫描生成
