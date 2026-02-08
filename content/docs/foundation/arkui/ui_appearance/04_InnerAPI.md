# 内部 API

> UI Appearance 内部模块接口与实现细节

## 模块概览

```
services/
├── include/
│   ├── ui_appearance_ability.h         # SA 主类
│   ├── ui_appearance_ability_client.h  # 客户端代理
│   ├── ui_appearance_types.h           # 类型定义
│   ├── dark_mode_manager.h            # 深色模式管理
│   ├── dark_mode_temp_state_manager.h # 临时状态管理
│   ├── screen_switch_operator_manager.h # 屏幕开关管理
│   ├── smart_gesture_manager.h        # 智能手势管理
│   └── background_app_color_switch_settings.h
├── src/
│   ├── ui_appearance_ability.cpp       # SA 主逻辑
│   ├── ui_appearance_ability_client.cpp # 客户端实现
│   ├── dark_mode_manager.cpp
│   ├── dark_mode_temp_state_manager.cpp
│   ├── screen_switch_operator_manager.cpp
│   ├── smart_gesture_manager.cpp
│   └── background_app_color_switch_settings.cpp
├── utils/
│   ├── include/
│   │   ├── setting_data_manager.h     # 设置数据管理
│   │   ├── setting_data_observer.h    # 数据观察者
│   │   ├── parameter_wrap.h           # 参数封装
│   │   ├── alarm_timer.h              # 定时器
│   │   ├── alarm_timer_manager.h      # 定时器管理
│   │   └── json_utils.h               # JSON工具
│   └── src/
└── IUiAppearanceAbility.idl           # IPC 接口定义
```

---

## UiAppearanceAbility

### 类信息

| 属性 | 值 |
|------|-----|
| **类名** | UiAppearanceAbility |
| **基类** | SystemAbility, UiAppearanceAbilityStub |
| **SA ID** | 7002 |
| **进程** | ui_service |
| **注册宏** | `REGISTER_SYSTEM_ABILITY_BY_ID(UiAppearanceAbility, ARKUI_UI_APPEARANCE_SERVICE_ID, true)` |

**代码位置**: `services/src/ui_appearance_ability.cpp:115`

### 公共接口

```cpp
class UiAppearanceAbility : public SystemAbility, public UiAppearanceAbilityStub {
public:
    struct UiAppearanceParam {
        DarkMode darkMode = DarkMode::ALWAYS_LIGHT;
        std::string fontScale = "1";
        std::string fontWeightScale = "1";
    };

    // 深色模式
    ErrCode SetDarkMode(int32_t mode, int32_t& funcResult);
    ErrCode GetDarkMode(int32_t& funcResult);
    
    // 字体缩放
    ErrCode GetFontScale(std::string& fontScale, int32_t& funcResult);
    ErrCode SetFontScale(const std::string& fontScale, int32_t& funcResult);
    
    // 字重缩放
    ErrCode GetFontWeightScale(std::string& fontWeightScale, int32_t& funcResult);
    ErrCode SetFontWeightScale(const std::string& fontWeightScale, int32_t& funcResult);
```

**代码位置**: `services/include/ui_appearance_ability.h:47-64`

### 关键成员变量

| 成员 | 类型 | 说明 |
|------|------|------|
| usersParam_ | std::map<int32_t, UiAppearanceParam> | 用户配置映射（线程安全） |
| userSwitchUpdateConfigurationOnceFlag_ | std::set<int32_t> | 防止重复更新标志 |
| uiAppearanceEventSubscriber_ | std::shared_ptr<UiAppearanceEventSubscriber> | 公共事件订阅者 |
| isNeedDoCompatibleProcess_ | std::atomic<bool> | 是否需要兼容处理 |
| isInitializationFinished_ | std::atomic<bool> | 初始化是否完成 |

### 生命周期接口

```cpp
protected:
    void OnStart() override;                    // SA 启动
    void OnStop() override;                     // SA 停止
    void OnAddSystemAbility(int32_t systemAbilityId, const std::string& deviceId) override;
    void OnRemoveSystemAbility(int32_t systemAbilityId, const std::string& deviceId) override;
```

### 核心方法详解

#### OnStart

**功能**: SA 启动初始化

**代码位置**: `services/src/ui_appearance_ability.cpp:153-164`

```cpp
void UiAppearanceAbility::OnStart()
{
    // 1. 发布 SA 到 SAMGR
    bool res = Publish(this);
    if (!res) {
        LOGE("publish failed.");
        return;
    }
    
    // 2. 监听 APP_MGR_SERVICE_ID
    LOGI("AddSystemAbilityListener start.");
    AddSystemAbilityListener(APP_MGR_SERVICE_ID);
}
```

#### OnAddSystemAbility

**功能**: 依赖 SA 就绪后的初始化

**代码位置**: `services/src/ui_appearance_ability.cpp:346-380`

**主要职责**:
- 初始化 DarkModeManager
- 初始化 SmartGestureManager
- 订阅公共事件
- 加载用户配置
- 执行兼容处理

#### VerifyAccessToken

**功能**: 权限验证

**代码位置**: `services/src/ui_appearance_ability.cpp:142-151`

```cpp
bool UiAppearanceAbility::VerifyAccessToken(const std::string& permissionName)
{
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, permissionName);
    if (ret == Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        return true;
    }
    LOGE("permission %{private}s denied, callerToken : %{public}u", 
        permissionName.c_str(), callerToken);
    return false;
}
```

#### UpdateConfiguration

**功能**: 更新系统配置

**代码位置**: `services/src/ui_appearance_ability.cpp:455-501`

```cpp
bool UiAppearanceAbility::UpdateConfiguration(
    const AppExecFwk::Configuration& configuration, 
    const int32_t userId,
    const std::vector<std::int32_t>& effectiveUserIds)
{
    auto appManagerInstance = GetAppManagerInstance();
    
    // 调用 AppMgr 更新配置
    if (effectiveUserIds.size() > 1) {
        errcode = appManagerInstance->UpdateConfigurationByUserIds(
            configuration, effectiveUserIds);
    } else {
        errcode = appManagerInstance->UpdateConfiguration(configuration, userId);
    }
    
    // 更新后台应用颜色切换
    if (!configuration.GetItem(AAFwk::GlobalConfigurationKey::SYSTEM_COLORMODE).empty()) {
        BackGroundAppColorSwitch(appManagerInstance, userId);
    }
    return true;
}
```

---

## UiAppearanceAbilityClient

### 类信息

| 属性 | 值 |
|------|-----|
| **类名** | UiAppearanceAbilityClient |
| **基类** | RefBase |
| **可见性** | `__attribute__((visibility("default")))` |

**代码位置**: `services/include/ui_appearance_ability_client.h`

### 公共接口

```cpp
class __attribute__((visibility("default"))) UiAppearanceAbilityClient : public RefBase {
public:
    UiAppearanceAbilityClient();
    ~UiAppearanceAbilityClient() = default;
    static sptr<UiAppearanceAbilityClient> GetInstance();

    int32_t SetDarkMode(DarkMode mode);
    int32_t GetDarkMode();
    int32_t GetFontScale(std::string& fontScale);
    int32_t SetFontScale(std::string& fontScale);
    int32_t GetFontWeightScale(std::string& fontWeightScale);
    int32_t SetFontWeightScale(std::string& fontWeightScale);
    void OnRemoteSaDied(const wptr<IRemoteObject>& object);
```

### 获取 SA Proxy

```cpp
private:
    sptr<IUiAppearanceAbility> GetUiAppearanceServiceProxy();
    sptr<IUiAppearanceAbility> CreateUiAppearanceServiceProxy();
    
    std::mutex serviceProxyLock_;
    sptr<IUiAppearanceAbility> uiAppearanceServiceProxy_;
};
```

---

## DarkModeManager

### 类信息

| 属性 | 值 |
|------|-----|
| **类名** | DarkModeManager |
| **设计模式** | Singleton |

**代码位置**: `services/include/dark_mode_manager.h`

### 深色模式枚举

```cpp
enum DarkModeMode {
    DARK_MODE_INVALID = -1,
    DARK_MODE_ALWAYS_LIGHT = 0,
    DARK_MODE_ALWAYS_DARK = 1,
    DARK_MODE_CUSTOM_AUTO = 2,      // 自定义时间段
    DARK_MODE_SUNRISE_SUNSET = 3,    // 日出日落
    DARK_MODE_SIZE,
};
```

### 状态结构

```cpp
struct DarkModeState {
    DarkModeMode settingMode = DARK_MODE_INVALID;
    int32_t settingStartTime = -1;
    int32_t settingEndTime = -1;
    int32_t settingSunsetTime = SUNSET_TIME_DEFAULT;   // 默认 18:00
    int32_t settingSunriseTime = SUNRISE_TIME_DEFAULT; // 默认 7:00
};
```

### 公共接口

```cpp
static DarkModeManager &GetInstance();

ErrCode Initialize(const std::function<void(bool, int32_t)>& updateCallback);
ErrCode LoadUserSettingData(int32_t userId, bool needUpdateCallback, 
    bool &isDarkMode, const bool bootLoadFlag);
void NotifyDarkModeUpdate(int32_t userId, bool isDarkMode);
ErrCode OnSwitchUser(int32_t userId);
void ScreenOnCallback();
void ScreenOffCallback();
ErrCode RestartTimer();
void Dump();
bool GetSettingTime(const int32_t userId, int32_t& settingStartTime, 
    int32_t& settingEndTime);
bool IsColorModeNormal(const int32_t userId);
void DoSwitchTemporaryColorMode(const int32_t userId, bool isDarkMode);
```

### 默认时间常量

```cpp
constexpr int32_t HOUR_TO_MINUTE = 60;
constexpr int32_t DAY_TO_MINUTE = 24 * 60;
constexpr int32_t SUNSET_TIME_DEFAULT = 18 * HOUR_TO_MINUTE;      // 18:00
constexpr int32_t SUNRISE_TIME_DEFAULT = 7 * HOUR_TO_MINUTE + DAY_TO_MINUTE;  // 次日 7:00
```

---

## SmartGestureManager

### 类信息

| 属性 | 值 |
|------|-----|
| **类名** | SmartGestureManager |
| **设计模式** | Singleton |

### 公共接口

```cpp
class SmartGestureManager final : public NoCopyable {
public:
    static SmartGestureManager &GetInstance();
    ErrCode Initialize(const std::function<void(bool, int32_t)>& updateCallback);
    void RegisterSettingDataObserver();
    void UpdateSmartGestureInitialValue();
};
```

---

## SettingDataManager

### 公共接口

```cpp
class SettingDataManager : public NoCopyable {
public:
    static SettingDataManager &GetInstance();
    ErrCode Initialize();
    bool IsInitialized();
    
    // 获取值
    ErrCode GetStringValue(const std::string& key, std::string& value, int32_t userId);
    ErrCode GetInt32Value(const std::string& key, int32_t& value, int32_t userId);
    ErrCode GetInt32ValueStrictly(const std::string& key, int32_t& value, int32_t userId);
    
    // 设置值
    ErrCode SetStringValue(const std::string& key, const std::string& value, int32_t userId);
    
    // 观察者
    ErrCode RegisterObserver(const std::string& key, 
        const std::function<void(const std::string&, int32_t)>& callback, 
        int32_t userId);
    ErrCode UnregisterObserver(const std::string& key, int32_t userId);
};
```

### 设置键名常量

```cpp
const std::string SETTING_DARK_MODE_MODE = "settings.uiappearance.darkmode_mode";
const std::string SETTING_DARK_MODE_START_TIME = "settings.uiappearance.darkmode_starttime";
const std::string SETTING_DARK_MODE_END_TIME = "settings.uiappearance.darkmode_endtime";
const std::string SETTING_DARK_MODE_SUN_SET = "settings.display.sun_set";
const std::string SETTING_DARK_MODE_SUN_RISE = "settings.display.sun_rise";
```

---

## AlarmTimerManager

### 公共接口

```cpp
class AlarmTimerManager : public NoCopyable {
public:
    static AlarmTimerManager &GetInstance();
    
    ErrCode SetAlarmTimer(const std::string& pkgName, const std::string& name, 
        uint64_t triggerTime, const AlarmTimerCallback& callback);
    ErrCode CancelAlarmTimer(const std::string& pkgName, const std::string& name);
    ErrCode SetScheduleTime(int32_t startTime, int32_t endTime, int32_t userId,
        const AlarmTimerCallback& startCallback, const AlarmTimerCallback& endCallback);
    ErrCode ClearTimerByUserId(int32_t userId);
    ErrCode RestartAllTimer();
    bool IsWithinTimeInterval(int32_t startTime, int32_t endTime);
    void Dump();
};
```

---

## ParameterWrap

### 公共接口

```cpp
class ParameterWrap {
public:
    static bool GetParameterWrap(const std::string& key, std::string& value);
    static bool SetParameterWrap(const std::string& key, const std::string& value);
};
```

### 参数键名

```cpp
// 深色模式
static const std::string PERSIST_DARKMODE_KEY = "persist.ace.darkmode";
static const std::string PERSIST_DARKMODE_KEY_FOR_NONE = "persist.ace.darkmode.";

// 字体缩放
static const std::string FONT_SCAL_FOR_USER0 = "persist.sys.font_scale_for_user0";
static const std::string FONT_SCAL_FOR_NONE = "persist.sys.font_scale_for_user.";

// 字重缩放
static const std::string FONT_Weight_SCAL_FOR_USER0 = "persist.sys.font_wght_scale_for_user0";
static const std::string FONT_WEIGHT_SCAL_FOR_NONE = "persist.sys.font_wght_scale_for_user.";
```

---

## 依赖方向

```
                    ┌─────────────────┐
                    │ UiAppearance    │
                    │    Ability      │
                    │   (SA 主类)     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
    ┌─────────────────┐ ┌───────────┐ ┌─────────────────┐
    │  DarkModeManager│ │ SmartGesture│ │  SettingData    │
    │                 │ │ Manager    │ │  Manager        │
    └────────┬────────┘ └─────┬─────┘ └────────┬────────┘
             │                │               │
             │       ┌────────┴────────┐      │
             │       │                 │      │
             ▼       ▼                 ▼      ▼
    ┌─────────────────┐ ┌───────────┐ ┌─────────────────┐
    │ AlarmTimerManager│ │ Parameter │ │  CommonEvent    │
    │                  │ │   Wrap    │ │  Subscriber     │
    └─────────────────┘ └───────────┘ └─────────────────┘
```

---

## 稳定性标注

| 模块/接口 | 稳定性 | 标注依据 |
|-----------|--------|----------|
| UiAppearanceAbility | 稳定 | 对外 SA 接口，版本稳定 |
| UiAppearanceAbilityClient | 稳定 | N-API 内部调用 |
| DarkModeManager | 稳定 | 内部实现，无公开 API 变化 |
| SettingDataManager | 稳定 | 工具类，接口固定 |
| AlarmTimerManager | 稳定 | 工具类，接口固定 |
| ParameterWrap | 稳定 | 工具类，接口固定 |

---

*相关内容: [N-API 接口](03_NAPI.md) | [构建配置](05_Build.md)*
