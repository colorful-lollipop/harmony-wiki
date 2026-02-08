# 内部 API 参考

> **目的**: 详细说明位置服务组件的内部 API 接口，供框架开发者参考  
> **适用范围**: 框架开发者、系统集成工程师、需要扩展定位功能的开发者  
> **最后更新**: 2026-02-05

---

## 1. 概述

内部 API（Inner API）位于 `interfaces/inner_api/` 目录下，供框架层内部模块使用。这些 API 不对应用层暴露，但为上层框架提供核心能力。

### 1.1 Inner API 定位

```
用户应用 (JS/Native)
        │
        ▼
对外接口层 (C API / JS N-API)
        │
        ▼
框架层 (locator_sdk / locator_agent / geofence_sdk)
        │
        ▼
内部 API 层 (interfaces/inner_api/)
        │
        ▼
SA 服务层 (services/location_*)
```

---

## 2. 核心接口

### 2.1 Locator 接口

**文件**: `interfaces/inner_api/include/locator.h`

**职责**: 定义定位器的基本接口

```cpp
namespace OHOS {
namespace Location {

class Locator {
public:
    virtual ~Locator() = default;
    
    // 开始定位
    virtual int StartLocating() = 0;
    
    // 停止定位
    virtual int StopLocating() = 0;
    
    // 设置定位回调
    virtual int SetLocationCallback() = 0;
    
    // 取消定位回调
    virtual int UnsetLocationCallback() = 0;
    
    // 检查定位开关
    virtual bool IsLocatingEnabled() = 0;
};

}  // namespace Location
}  // namespace OHOS
```

---

### 2.2 LocatorImpl 类

**文件**: `interfaces/inner_api/include/locator_impl.h`

**职责**: Locator 接口的默认实现

```cpp
namespace OHOS {
namespace Location {

class LocatorImpl : public Locator {
public:
    static LocatorImpl* GetInstance();
    
    int StartLocating() override;
    int StopLocating() override;
    int SetLocationCallback() override;
    int UnsetLocationCallback() override;
    bool IsLocatingEnabled() override;
    
private:
    LocatorImpl();
    ~LocatorImpl();
    static LocatorImpl* instance_;
};

}  // namespace Location
}  // namespace OHOS
```

**使用示例**:
```cpp
auto locator = LocatorImpl::GetInstance();
if (locator->IsLocatingEnabled()) {
    locator->StartLocating();
}
```

---

## 3. 请求与配置

### 3.1 RequestConfig 类

**文件**: `interfaces/inner_api/include/request_config.h`

**职责**: 定义定位请求参数

```cpp
namespace OHOS {
namespace Location {

class RequestConfig {
public:
    enum Priority {
        PRIORITY_UNSET = 0x200,
        PRIORITY_ACCURACY = 0x201,
        PRIORITY_LOW_POWER = 0x202,
        PRIORITY_FIRST_FIX = 0x203,
    };
    
    enum Scenario {
        SCENE_UNSET = 0x400,
        SCENE_NAVIGATION = 0x401,
        SCENE_SPORT = 0x402,
        SCENE_TRANSPORT = 0x403,
        SCENE_DAILY_LIFE_SERVICE = 0x404,
    };
    
    void SetPriority(Priority priority);
    Priority GetPriority() const;
    
    void SetScenario(Scenario scenario);
    Scenario GetScenario() const;
    
    void SetInterval(int interval);
    int GetInterval() const;
    
private:
    Priority priority_ = PRIORITY_UNSET;
    Scenario scenario_ = SCENE_UNSET;
    int interval_ = 1;
};

}  // namespace Location
}  // namespace OHOS
```

---

### 3.2 Request 类

**文件**: `interfaces/inner_api/include/request.h`

**职责**: 完整的定位请求

```cpp
namespace OHOS {
namespace Location {

class Request {
public:
    void SetPid(int pid);
    int GetPid() const;
    
    void SetTokenId(uint32_t tokenId);
    uint32_t GetTokenId() const;
    
    void SetUuid(const std::string& uuid);
    std::string GetUuid() const;
    
    void SetRequestConfig(std::shared_ptr<RequestConfig> config);
    std::shared_ptr<RequestConfig> GetRequestConfig() const;
    
    void SetPackageName(const std::string& packageName);
    std::string GetPackageName() const;
    
private:
    int pid_ = 0;
    uint32_t tokenId_ = 0;
    std::string uuid_;
    std::shared_ptr<RequestConfig> config_;
    std::string packageName_;
};

}  // namespace Location
}  // namespace OHOS
```

---

## 4. 位置数据

### 4.1 Location 类

**文件**: `interfaces/inner_api/include/location.h`

**职责**: 封装位置信息

```cpp
namespace OHOS {
namespace Location {

class Location {
public:
    double GetLatitude() const;
    void SetLatitude(double latitude);
    
    double GetLongitude() const;
    void SetLongitude(double longitude);
    
    double GetAltitude() const;
    void SetAltitude(double altitude);
    
    double GetAccuracy() const;
    void SetAccuracy(double accuracy);
    
    double GetSpeed() const;
    void SetSpeed(double speed);
    
    double GetDirection() const;
    void SetDirection(double direction);
    
    int64_t GetTimeForFix() const;
    void SetTimeForFix(int64_t time);
    
    int64_t GetTimeSinceBoot() const;
    void SetTimeSinceBoot(int64_t time);
    
    int GetSourceType() const;
    void SetSourceType(int sourceType);
    
private:
    double latitude_ = 0.0;
    double longitude_ = 0.0;
    double altitude_ = 0.0;
    double accuracy_ = 0.0;
    double speed_ = 0.0;
    double direction_ = 0.0;
    int64_t timeForFix_ = 0;
    int64_t timeSinceBoot_ = 0;
    int sourceType_ = 0;
};

}  // namespace Location
}  // namespace OHOS
```

---

## 5. 回调接口

### 5.1 ILocatorCallback 接口

**文件**: `interfaces/inner_api/include/i_locator_callback.h`

**职责**: 定义定位回调

```cpp
namespace OHOS {
namespace Location {

class ILocatorCallback {
public:
    virtual ~ILocatorCallback() = default;
    
    // 位置更新回调
    virtual void OnLocationReport(const std::shared_ptr<Location>& location) = 0;
    
    // 定位错误回调
    virtual void OnErrorReport(int errorCode) = 0;
    
    // 状态变化回调
    virtual void OnStatusChange(int status) = 0;
};

}  // namespace Location
}  // namespace OHOS
```

---

### 5.2 ICachedLocationsCallback 接口

**文件**: `interfaces/inner_api/include/i_cached_locations_callback.h`

**职责**: 缓存位置回调

```cpp
namespace OHOS {
namespace Location {

class ICachedLocationsCallback {
public:
    virtual ~ICachedLocationsCallback() = default;
    
    virtual void OnCacheLocationsReport(
        const std::vector<std::shared_ptr<Location>>& locations) = 0;
};

}  // namespace Location
}  // namespace OHOS
```

---

## 6. 权限管理

### 6.1 PermissionManager 类

**文件**: `interfaces/inner_api/include/permission_manager.h`

**职责**: 权限检查与验证

```cpp
namespace OHOS {
namespace Location {

class PermissionManager {
public:
    // 检查定位权限
    static bool CheckLocationPermission(uint32_t tokenId);
    
    // 检查精确定位权限
    static bool CheckApproximatelyLocationPermission(uint32_t tokenId);
    
    // 检查是否已授权
    static bool IsLocationPermitted(uint32_t tokenId);
    
    // 获取权限状态
    static int GetPermissionStatus(uint32_t tokenId);
    
private:
    PermissionManager() = default;
};

}  // namespace Location
}  // namespace OHOS
```

---

## 7. 应用身份

### 7.1 AppIdentity 类

**文件**: `interfaces/inner_api/include/app_identity.h`

**职责**: 封装应用身份信息

```cpp
namespace OHOS {
namespace Location {

class AppIdentity {
public:
    void SetPid(int pid);
    int GetPid() const;
    
    void SetTokenId(uint32_t tokenId);
    uint32_t GetTokenId() const;
    
    void SetUid(int uid);
    int GetUid() const;
    
    void SetBundleName(const std::string& bundleName);
    std::string GetBundleName() const;
    
    std::string ToString() const;
    
private:
    int pid_ = 0;
    uint32_t tokenId_ = 0;
    int uid_ = 0;
    std::string bundleName_;
};

}  // namespace Location
}  // namespace OHOS
```

---

## 8. 常量定义

### 8.1 LocationLogEventIds

**文件**: `interfaces/inner_api/include/location_log_event_ids.h`

**职责**: 定义日志事件 ID

```cpp
namespace OHOS {
namespace Location {

enum LocationLogEventId {
    LOCATOR_SERVICE_START = 0x0001,
    LOCATOR_SERVICE_STOP = 0x0002,
    LOCATION_REQUEST_START = 0x0003,
    LOCATION_REQUEST_STOP = 0x0004,
    LOCATION_REPORT = 0x0005,
    LOCATION_ERROR = 0x0006,
    PERMISSION_CHECK = 0x0007,
    SWITCH_STATE_CHANGE = 0x0008,
};

}  // namespace Location
}  // namespace OHOS
```

---

## 9. 内部模块列表

| 模块 | 头文件 | 职责 |
|------|--------|------|
| locator | `locator.h`, `locator_impl.h` | 定位器核心接口 |
| request | `request.h`, `request_config.h` | 请求配置管理 |
| location | `location.h` | 位置数据封装 |
| permission | `permission_manager.h` | 权限管理 |
| identity | `app_identity.h` | 应用身份管理 |
| callback | `i_locator_callback.h` | 回调接口定义 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [概览](index.md) | 项目定位与核心能力 |
| [系统架构](01_Architecture.md) | 架构与组件关系 |
| [C/N-API 接口](02_C_NAPI.md) | 对外 API 接口 |
| [安全风险评审](07_Security.md) | 安全分析 |

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本 |
