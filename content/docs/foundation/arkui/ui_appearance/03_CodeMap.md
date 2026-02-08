# 代码地图

> UI Appearance 子系统目录结构、核心文件定位与功能导航

## 文档目的

本文档为开发者和安全研究员提供代码导航工具，帮助快速定位：

1. **顶层目录职责**: 每个目录的功能和包含内容
2. **核心文件定位**: 关键实现文件的精确位置
3. **代码导航图**: 功能 → 文件路径的映射表
4. **快速索引**: 按功能/符号查找代码位置

**适用受众**: 开发者、安全研究员
**阅读时间**: 约 5-10 分钟

---

## 目录结构概览

### 顶层目录职责

```
arkui/ui_appearance/
├── interfaces/                    # 对外接口层
│   ├── kits/napi/                  # N-API (Node.js/JS) 接口实现
│   │   ├── include/js_ui_appearance.h       # N-API 头文件 (AsyncContext, 函数声明)
│   │   ├── src/js_ui_appearance.cpp        # N-API 实现文件 (6 个接口函数)
│   │   └── BUILD.gn                      # N-API 构建配置 (libuiappearance.z.so)
│   └── ets/ani/                       # ANI (ArkTS Native Interface) 接口
│       ├── @ohos.uiAppearance.ets           # ETS 接口定义 (暴露给 ArkTS)
│       ├── src/ui_appearance.cpp             # ANI 实现文件
│       └── BUILD.gn                        # ANI 构建配置
│
├── services/                       # 系统服务实现层
│   ├── include/                           # 头文件目录
│   │   ├── ui_appearance_ability.h         # System Ability 主类 (SA 7002)
│   │   ├── ui_appearance_ability_client.h  # IPC 客户端代理类
│   │   ├── ui_appearance_types.h          # 类型定义 (DarkMode, ErrorCode)
│   │   ├── ui_appearance_log.h            # 日志宏定义
│   │   ├── dark_mode_manager.h            # 深色模式管理器
│   │   ├── dark_mode_temp_state_manager.h  # 临时颜色状态管理
│   │   ├── screen_switch_operator_manager.h # 屏幕开关操作管理
│   │   ├── smart_gesture_manager.h          # 智能手势管理器
│   │   └── background_app_color_switch_settings.h # 后台应用颜色切换设置
│   │
│   ├── src/                                # 实现源码目录
│   │   ├── ui_appearance_ability.cpp       # System Ability 主实现 (6 个 IPC 方法)
│   │   ├── ui_appearance_ability_client.cpp  # IPC 客户端代理实现
│   │   ├── dark_mode_manager.cpp           # 深色模式管理器实现
│   │   ├── dark_mode_temp_state_manager.cpp # 临时颜色状态管理
│   │   ├── screen_switch_operator_manager.cpp # 屏幕开关操作
│   │   ├── smart_gesture_manager.cpp        # 智能手势管理器实现
│   │   ├── background_app_color_switch_settings.cpp # 后台应用颜色切换
│   │   └── ... (其他工具实现)
│   │
│   ├── utils/                               # 工具类目录
│   │   ├── include/                        # 工具类头文件
│   │   │   ├── parameter_wrap.h            # 系统参数封装
│   │   │   ├── setting_data_manager.h      # 设置数据管理器
│   │   │   ├── setting_data_observer.h       # 设置数据观察者
│   │   │   ├── alarm_timer.h                # 定时器工具
│   │   │   ├── alarm_timer_manager.h       # 定时器管理器
│   │   │   └── json_utils.h                 # JSON 工具
│   │   └── src/                           # 工具类实现
│   │       ├── parameter_wrap.cpp            # GetParameterWrap/SetParameterWrap
│   │       ├── setting_data_manager.cpp      # RegisterObserver/NotifyChange
│   │       ├── setting_data_observer.cpp       # OnChange 回调
│   │       ├── alarm_timer.cpp                # 定时器创建/启动/停止
│   │       ├── alarm_timer_manager.cpp       # 定时器调度
│   │       └── json_utils.cpp                 # JSON 解析工具
│   │
│   ├── IUiAppearanceAbility.idl           # IPC 接口定义 (6 个方法)
│   └── BUILD.gn                           # 系统服务构建配置
│
├── sa_profile/                     # 系统能力 (System Ability) 配置
│   ├── 7002.json                          # SA 7002 配置 (进程名、库路径)
│   └── BUILD.gn                           # SA 配置文件构建
│
├── etc/                            # 持久化配置文件
│   └── para/                              # 系统参数目录
│       ├── ui_appearance.para             # 参数默认值
│       ├── ui_appearance.para.dac         # DAC 权限配置
│       └── BUILD.gn                       # 参数文件构建
│
├── test/                           # 测试目录 (本文档不覆盖)
│   └── unittest/                       # 单元测试
│
├── wiki/                           # 文档目录 (本文档)
└── BUILD.gn                       # 根构建配置
```

---

## 核心文件定位

### 1. 系统服务主文件

| 文件路径 | 核心类/职责 | 关键方法 | 行号 |
|----------|--------------|---------|------|
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` (SA 7002 主类) | SetDarkMode | 549 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | GetDarkMode | 598 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | SetFontScale | 641 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | GetFontScale | 660 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | SetFontWeightScale | 706 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | GetFontWeightScale | 725 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | VerifyAccessToken | 142 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | UpdateConfiguration | 455 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | GetCallingUserId | 391 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | GetAppManagerInstance | 119 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | SubscribeCommonEvent | 326 |
| **services/src/ui_appearance_ability.cpp** | `UiAppearanceAbility` | OnStart/OnStop | 153,166 |

---

### 2. N-API 实现文件

| 文件路径 | 核心函数 | 行号 | 说明 |
|----------|---------|------|------|
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `JSSetDarkMode` | 289 | N-API setDarkMode 入口 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `JSGetDarkMode` | 334 | N-API getDarkMode 入口 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `JSSetFontScale` | 381 | N-API setFontScale 入口 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `JSGetFontScale` | 357 | N-API getFontScale 入口 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `JSSetFontWeightScale` | 449 | N-API setFontWeightScale 入口 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `JSGetFontWeightScale` | 425 | N-API getFontWeightScale 入口 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `CheckCallerIsSystemApp` | 280 | 系统应用检查 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `CheckArgs` | 204 | 参数类型/数量校验 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `CheckFontScaleArgs` | 236 | 字体缩放参数校验 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `ConvertJsDarkMode2Enum` | 268 | DarkMode 枚举转换 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `OnExecute` | 69 | SetDarkMode 异步执行 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `OnSetFontScale` | 88 | SetFontScale 异步执行 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `OnSetFontWeightScale` | 117 | SetFontWeightScale 异步执行 |
| **interfaces/kits/napi/src/js_ui_appearance.cpp** | `OnComplete` | 148 | 异步操作完成回调 |

---

### 3. ANI 实现文件

| 文件路径 | 核心函数 | 行号 | 说明 |
|----------|---------|------|------|
| **interfaces/ets/ani/src/ui_appearance.cpp** | `SetFontScale` | 268 | ANI setFontScale 入口 |
| **interfaces/ets/ani/src/ui_appearance.cpp** | `GetFontScale` | 298 | ANI getFontScale 入口 |
| **interfaces/ets/ani/src/ui_appearance.cpp** | `SetFontWeightScale` | 334 | ANI setFontWeightScale 入口 |
| **interfaces/ets/ani/src/ui_appearance.cpp** | `GetFontWeightScale` | 357 | ANI getFontWeightScale 入口 |
| **interfaces/ets/ani/src/ui_appearance.cpp** | `SetDarkMode` | 203 | ANI setDarkMode 入口 |
| **interfaces/ets/ani/src/ui_appearance.cpp** | `GetDarkMode` | 242 | ANI getDarkMode 入口 |
| **interfaces/ets/ani/src/ui_appearance.cpp** | `ConvertJsDarkMode2Enum` | 145 | DarkMode 枚举转换 |

**注意**: ANI 层缺少 `CheckCallerIsSystemApp()` 检查 (参考 [安全评审](06_Security.md))

---

### 4. 辅助管理器文件

| 文件路径 | 核心类/职责 | 关键方法 | 行号 |
|----------|--------------|---------|------|
| **services/src/dark_mode_manager.cpp** | `DarkModeManager` | Initialize | 56 |
| **services/src/dark_mode_manager.cpp** | `DarkModeManager` | LoadUserSettingData | 39 |
| **services/src/dark_mode_manager.cpp** | `DarkModeManager` | RestartTimer | 49 |
| **services/src/dark_mode_manager.cpp** | `DarkModeManager` | ScreenOnCallback | 317 |
| **services/src/dark_mode_manager.cpp** | `DarkModeManager` | ScreenOffCallback | 318 |
| **services/src/smart_gesture_manager.cpp** | `SmartGestureManager` | Initialize | 32 |
| **services/src/smart_gesture_manager.cpp** | `SmartGestureManager` | RegisterSettingDataObserver | 60 |
| **services/utils/src/setting_data_manager.cpp** | `SettingDataManager` | RegisterObserver | 72 |
| **services/utils/src/setting_data_manager.cpp** | `SettingDataManager` | NotifyChange | 85 |

---

### 5. 工具类文件

| 文件路径 | 核心类/职责 | 关键方法 | 说明 |
|----------|--------------|---------|------|
| **services/utils/src/parameter_wrap.cpp** | `ParameterWrap` | GetParameterWrap | 封装 GetParameter |
| **services/utils/src/parameter_wrap.cpp** | `ParameterWrap` | SetParameterWrap | 封装 SetParameter |
| **services/utils/src/setting_data_observer.cpp** | `SettingDataObserver` | OnChange | 设置数据变化回调 |

---

## 代码导航图

### 按功能模块查找

| 功能 | 相关文件 | 核心符号 |
|------|---------|----------|
| **N-API 接口注册** | `interfaces/kits/napi/src/js_ui_appearance.cpp:518-531` | `UiAppearanceExports()`, `ui_appearance_module` |
| **N-API setDarkMode** | `interfaces/kits/napi/src/js_ui_appearance.cpp:289` | `JSSetDarkMode()` |
| **N-API getDarkMode** | `interfaces/kits/napi/src/js_ui_appearance.cpp:334` | `JSGetDarkMode()` |
| **N-API setFontScale** | `interfaces/kits/napi/src/js_ui_appearance.cpp:381` | `JSSetFontScale()` |
| **N-API getFontScale** | `interfaces/kits/napi/src/js_ui_appearance.cpp:357` | `JSGetFontScale()` |
| **N-API setFontWeightScale** | `interfaces/kits/napi/src/js_ui_appearance.cpp:449` | `JSSetFontWeightScale()` |
| **N-API getFontWeightScale** | `interfaces/kits/napi/src/js_ui_appearance.cpp:425` | `JSGetFontWeightScale()` |
| **权限校验** | `services/src/ui_appearance_ability.cpp:142` | `VerifyAccessToken()` |
| **系统应用检查** | `interfaces/kits/napi/src/js_ui_appearance.cpp:280` | `CheckCallerIsSystemApp()` |
| **SA 初始化** | `services/src/ui_appearance_ability.cpp:153` | `OnStart()`, `Publish()` |
| **SA SetDarkMode** | `services/src/ui_appearance_ability.cpp:549` | `SetDarkMode()` |
| **SA GetDarkMode** | `services/src/ui_appearance_ability.cpp:598` | `GetDarkMode()` |
| **SA SetFontScale** | `services/src/ui_appearance_ability.cpp:641` | `SetFontScale()` |
| **SA GetFontScale** | `services/src/ui_appearance_ability.cpp:660` | `GetFontScale()` |
| **深色模式管理** | `services/src/dark_mode_manager.cpp:56` | `DarkModeManager::Initialize()` |
| **参数持久化** | `services/utils/src/parameter_wrap.cpp:20` | `GetParameterWrap()`, `SetParameterWrap()` |
| **AMS 通信** | `services/src/ui_appearance_ability.cpp:119` | `GetAppManagerInstance()` |
| **配置更新** | `services/src/ui_appearance_ability.cpp:455` | `UpdateConfiguration()` |
| **事件订阅** | `services/src/ui_appearance_ability.cpp:326` | `SubscribeCommonEvent()` |
| **SA 7002 配置** | `sa_profile/7002.json:1` | `systemability` 数组 |

---

### 按类型查找

#### 枚举与常量

| 类型 | 定义位置 | 说明 |
|------|---------|------|
| **DarkMode 枚举** | `services/include/ui_appearance_types.h:22-26` | `ALWAYS_DARK=0`, `ALWAYS_LIGHT=1`, `UNKNOWN=2` |
| **UiAppearanceAbilityErrCode** | `services/include/ui_appearance_types.h:28-34` | `SUCCEEDED=0`, `PERMISSION_ERR=201`, `NOT_SYSTEM_APP=202`, `INVALID_ARG=401`, `SYS_ERR=500001` |
| **权限字符串常量** | `services/src/ui_appearance_ability.cpp:42` | `PERMISSION_UPDATE_CONFIGURATION = "ohos.permission.UPDATE_CONFIGURATION"` |
| **字体缩放常量** | `interfaces/kits/napi/src/js_ui_appearance.cpp:27-30` | `MIN_FONT_SCALE=0`, `MAX_FONT_SCALE=5` |

#### 类与结构体

| 类名 | 定义位置 | 职责 |
|------|---------|------|
| **UiAppearanceAbility** | `services/include/ui_appearance_ability.h:47-112` | System Ability 7002 主服务类 |
| **UiAppearanceAbilityClient** | `services/include/ui_appearance_ability_client.h:18-32` | IPC 客户端代理类 |
| **DarkModeManager** | `services/include/dark_mode_manager.h:33-131` | 深色模式管理器 (单例) |
| **SmartGestureManager** | `services/include/smart_gesture_manager.h:24-38` | 智能手势管理器 (单例) |
| **UiAppearanceEventSubscriber** | `services/include/ui_appearance_ability.h:31-45` | 公共事件订阅者类 |
| **AsyncContext** | `interfaces/kits/napi/include/js_ui_appearance.h:28-40` | N-API 异步上下文结构 |

---

### 按错误码查找

| 错误码 | 值 | 定义位置 | 说明 |
|-------|-----|---------|------|
| **SUCCEEDED** | 0 | `services/include/ui_appearance_types.h:29` | 成功 |
| **PERMISSION_ERR** | 201 | `services/include/ui_appearance_types.h:30` | 权限拒绝 |
| **NOT_SYSTEM_APP** | 202 | `services/include/ui_appearance_types.h:31` | 非系统应用 (N-API 特有) |
| **INVALID_ARG** | 401 | `services/include/ui_appearance_types.h:32` | 参数错误 |
| **SYS_ERR** | 500001 | `services/include/ui_appearance_types.h:33` | 系统错误 |

---

## 快速查找索引

### 按符号查找

| 符号名称 | 文件路径 | 行号 |
|----------|----------|------|
| `JSSetDarkMode` | `interfaces/kits/napi/src/js_ui_appearance.cpp` | 289 |
| `JSGetDarkMode` | `interfaces/kits/napi/src/js_ui_appearance.cpp` | 334 |
| `JSSetFontScale` | `interfaces/kits/napi/src/js_ui_appearance.cpp` | 381 |
| `JSGetFontScale` | `interfaces/kits/napi/src/js_ui_appearance.cpp` | 357 |
| `UiAppearanceAbility::SetDarkMode` | `services/src/ui_appearance_ability.cpp` | 549 |
| `UiAppearanceAbility::GetDarkMode` | `services/src/ui_appearance_ability.cpp` | 598 |
| `UiAppearanceAbility::VerifyAccessToken` | `services/src/ui_appearance_ability.cpp` | 142 |
| `UiAppearanceAbility::UpdateConfiguration` | `services/src/ui_appearance_ability.cpp` | 455 |
| `UiAppearanceAbility::GetAppManagerInstance` | `services/src/ui_appearance_ability.cpp` | 119 |
| `UiAppearanceAbility::GetCallingUserId` | `services/src/ui_appearance_ability.cpp` | 391 |
| `UiAppearanceAbility::SubscribeCommonEvent` | `services/src/ui_appearance_ability.cpp` | 326 |
| `DarkModeManager::GetInstance` | `services/include/dark_mode_manager.h:35` | 单例获取 |
| `DarkModeManager::Initialize` | `services/src/dark_mode_manager.cpp:56` | 初始化深色模式管理器 |
| `GetParameterWrap` | `services/utils/src/parameter_wrap.cpp:20` | 读取系统参数 |
| `SetParameterWrap` | `services/utils/src/parameter_wrap.cpp:37` | 写入系统参数 |
| `ARKUI_UI_APPEARANCE_SERVICE_ID` | `sa_profile/7002.json:5` | 7002 |

---

### 按文件扩展名查找

| 文件扩展名 | 说明 | 位置 |
|----------|------|------|
| `.cpp` | C++ 源文件 | 所有 `src/` 目录 |
| `.h` | C++ 头文件 | 所有 `include/` 目录 |
| `.idl` | IDL 接口定义 | `services/IUiAppearanceAbility.idl` |
| `.ets` | ArkTS/Ets 接口 | `interfaces/ets/ani/ets/` |
| `.gn` | 构建配置 | 所有目录 |
| `.json` | SA 配置 | `sa_profile/7002.json` |
| `.para` | 系统参数 | `etc/para/` |

---

## 证据库索引

| 主题 | 相关文件 | 证据位置 |
|------|---------|----------|
| **N-API 接口** | `interfaces/kits/napi/src/js_ui_appearance.cpp` | 全文件 |
| **ANI 接口** | `interfaces/ets/ani/src/ui_appearance.cpp` | 全文件 |
| **SA 主逻辑** | `services/src/ui_appearance_ability.cpp` | 全文件 |
| **深色模式管理** | `services/src/dark_mode_manager.cpp` | 全文件 |
| **IPC 通信** | `services/src/ui_appearance_ability_client.cpp` | 全文件 |
| **参数持久化** | `services/utils/src/parameter_wrap.cpp` | 全文件 |
| **SA 配置** | `sa_profile/7002.json` | 全文件 |
| **类型定义** | `services/include/ui_appearance_types.h` | 全文件 |
| **权限常量** | `services/src/ui_appearance_ability.cpp:42` | Line 42 |

---

**相关文档**:
- [项目概述](01_Overview.md) - 项目定位和边界
- [架构设计](02_Architecture.md) - 组件图和数据流
- [N-API 接口](03_NAPI.md) - API 详细文档
- [内部 API](04_InnerAPI.md) - 模块接口
- [攻击面分析](05_AttackSurface.md) - 攻击点映射
- [安全评审](06_Security.md) - 详细风险评估
