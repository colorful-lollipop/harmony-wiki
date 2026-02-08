# 内部 API (Inner Kits)

## 概述

Inner Kits 是供系统内部其他服务调用的 C++ 接口，相比对外 API 提供更丰富的功能。

## StandbyServiceClient

**头文件**：`interfaces/innerkits/include/standby_service_client.h`

**职责**：待机服务的客户端代理，单例模式

### 获取实例

```cpp
// 获取客户端实例
StandbyServiceClient& client = StandbyServiceClient::GetInstance();
```

### API 清单

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `SubscribeStandbyCallback` | `sptr<IStandbyServiceSubscriber>` | `ErrCode` | 订阅待机状态变化 |
| `UnsubscribeStandbyCallback` | `sptr<IStandbyServiceSubscriber>` | `ErrCode` | 取消订阅 |
| `ApplyAllowResource` | `sptr<ResourceRequest>` | `ErrCode` | 申请豁免资源 |
| `UnapplyAllowResource` | `sptr<ResourceRequest>` | `ErrCode` | 释放豁免资源 |
| `GetAllowList` | `uint32_t allowType, vector<AllowInfo>&, uint32_t reasonCode` | `ErrCode` | 获取允许列表 |
| `GetRestrictList` | `uint32_t restrictType, vector<AllowInfo>&, uint32_t reasonCode` | `ErrCode` | 获取限制列表 |
| `IsDeviceInStandby` | `bool& isStandby` | `ErrCode` | 查询设备待机状态 |
| `ReportWorkSchedulerStatus` | `bool, int32_t uid, string` | `ErrCode` | 上报工作调度状态 |
| `ReportDeviceStateChanged` | `DeviceStateType, bool` | `ErrCode` | 上报设备状态变化 |
| `IsStrategyEnabled` | `const string&, bool&` | `ErrCode` | 查询策略状态 |
| `SetNatInterval` | `uint32_t&, bool&, uint32_t&` | `ErrCode` | 设置 NAT 超时 |
| `HandleEvent` | `shared_ptr<ResData>` | `ErrCode` | 统一事件处理 |
| `ReportPowerOverused` | `const string&, uint32_t` | `ErrCode` | 上报功耗过载 |
| `DelayHeartBeat` | `int64_t` | `ErrCode` | 延迟心跳 |
| `ReportSceneInfo` | `uint32_t, int64_t, const string&` | `ErrCode` | 上报场景信息 |
| `PushProxyStateChanged` | `uint32_t, bool` | `ErrCode` | Push 代理状态变化 |
| `HeartBeatValueChanged` | `const string&, int32_t` | `ErrCode` | 心跳值变化 |

### 实现位置

| 方法 | 实现文件 |
|------|----------|
| `SubscribeStandbyCallback` | `interfaces/innerkits/src/standby_service_client.cpp` |
| `UnsubscribeStandbyCallback` | `interfaces/innerkits/src/standby_service_client.cpp` |
| `ApplyAllowResource` | `interfaces/innerkits/src/standby_service_client.cpp:68` |
| `UnapplyAllowResource` | `interfaces/innerkits/src/standby_service_client.cpp:83` |
| `IsDeviceInStandby` | `interfaces/innerkits/src/standby_service_client.cpp:113` |

## 核心数据结构

### ResourceRequest

**头文件**：`interfaces/innerkits/include/resource_request.h`

```cpp
struct ResourceRequest {
    uint32_t allowType_;      // 资源类型
    int32_t uid_;            // 应用 UID
    std::string name_;       // 应用名称
    int32_t duration_;      // 豁免时长（秒）
    std::string reason_;     // 申请原因
    uint32_t reasonCode_;    // 原因代码
};
```

### AllowInfo

**头文件**：`interfaces/innerkits/include/allow_info.h`

```cpp
struct AllowInfo {
    uint32_t allowType_;      // 允许类型
    std::string name_;        // 应用名称
    int32_t uid_;            // 应用 UID
    int64_t beginTime_;      // 开始时间
    int64_t endTime_;        // 结束时间
};
```

### AllowType

**头文件**：`interfaces/innerkits/include/allow_type.h`

```cpp
enum class AllowType : uint32_t {
    NETWORK = 1,
    RUNNING_LOCK = 2,
    TIMER = 4,
    WORK_SCHEDULER = 8,
    AUTO_SYNC = 16,
    PUSH = 32,
    FREEZE = 64,
};
```

## IStandbyServiceSubscriber

**头文件**：`frameworks/include/istandby_service_subscriber.h`

**职责**：待机状态订阅者的接口定义

### 接口方法

```cpp
// 状态变化回调
virtual void OnStandbyStateChanged(bool isStandby) = 0;

// 允许列表变化回调
virtual void OnAllowListChanged(uint32_t allowType, bool added) = 0;
```

### 注册位置

| 方法 | 实现类 |
|------|--------|
| `SubscribeStandbyCallback` | `StandbyServiceImpl` (`services/core/src/standby_service_impl.cpp`) |
| `UnsubscribeStandbyCallback` | `StandbyServiceImpl` |

## 稳定性标注

| 接口 | 稳定性 | 适用场景 |
|------|--------|----------|
| `StandbyServiceClient` | 较稳定 | 系统内部服务 |
| `IStandbyServiceSubscriber` | 较稳定 | 系统内部订阅者 |
| `ResourceRequest` | 稳定 | 数据结构 |

## 依赖方向

```
StandbyServiceClient
    ↓ IPC
StandbyService (SA)
    ↓
StandbyServiceImpl
    ↓
Plugin Layer
    ↓
System Services
```

## 使用示例

### 订阅待机状态

```cpp
#include "standby_service_client.h"
#include "istandby_service_subscriber.h"

class MySubscriber : public IStandbyServiceSubscriber {
public:
    void OnStandbyStateChanged(bool isStandby) override {
        // 处理状态变化
    }

    void OnAllowListChanged(uint32_t allowType, bool added) override {
        // 处理允许列表变化
    }
};

// 订阅待机状态
auto& client = StandbyServiceClient::GetInstance();
sptr<MySubscriber> subscriber = new MySubscriber();
client.SubscribeStandbyCallback(subscriber);
```

### 查询设备状态

```cpp
auto& client = StandbyServiceClient::GetInstance();
bool isStandby = false;
ErrCode ret = client.IsDeviceInStandby(isStandby);
if (ret == ERR_OK) {
    // 使用 isStandby 状态
}
```

## 内部错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | `ERR_OK` | 成功 |
| -1 | `ERR_STANDBY_SYS_NOT_READY` | 系统未就绪 |
| -2 | `ERR_RESOURCE_TYPES_INVALID` | 无效的资源类型 |
| -3 | `ERR_DURATION_INVALID` | 无效的时长 |
| -4 | `ERR_PERMISSION_ERROR` | 权限错误 |
| -5 | `ERR_STANDBY_ALLOWLIST_FULL` | 允许列表已满 |

**定义位置**：`utils/common/include/standby_service_errors.h`
