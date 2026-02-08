# 目录结构与模块职责

> Settings 应用的完整目录结构、模块职责与边界

---

## 目的

本文档详细说明 Settings 应用的目录结构、各模块的职责和边界。

## 适用范围

- 目标读者：开发者、架构师
- 项目：@ohos/settings (Settings 3.1)
- 覆盖范围：所有模块（除 test/）

---

## 顶层目录结构

```
/applications/standard/settings
├── AppScope/              # 应用范围资源
├── ani/                   # ANI（Ark Native Interface）模块
│   ├── settings/          # Settings ANI 实现
│   └── intelligentscene/   # 智能场景 ANI 实现
├── cj/                    # CJ（C-JSON FFI）模块
│   └── settings/          # Settings CJ FFI 实现
├── common/                # 公共代码和工具
│   ├── component/         # UI 组件库
│   ├── search/           # 搜索功能
│   ├── settingsBase/      # 设置基础库
│   └── utils/            # 工具函数
├── figures/               # 图片资源（架构图等）
├── hvigor/                # 构建工具配置
├── napi/                  # N-API 模块
│   ├── settings/          # Settings N-API 实现
│   └── intelligentscene/   # 智能场景 N-API 实现
├── native/                # Native C++ 实现代码
│   └── settings/          # Settings Native 实现
├── product/               # 产品特定代码
│   ├── phone/            # 手机产品（激活）
│   └── wearable/          # 可穿戴设备（预留）
├── signature/             # 签名证书
└── wiki/                 # Wiki 文档
```

---

## 模块详细分析

### 1. N-API 模块（napi/）

#### napi/settings/

**职责**：提供 JS ↔ C++ 绑定，实现设置管理 API

**文件结构**：
```
napi/settings/
├── native_module.cpp          # 模块注册点（napi_module_register）
├── napi_settings.cpp          # 主要实现（get/set value）
├── napi_settings_init.cpp     # N-API 类初始化
├── napi_settings_observer.cpp # 观察者实现
├── napi_settings.h           # 头文件（API 声明）
├── napi_settings_log.h       # 日志工具
├── open_network_settings/     # 网络设置相关
│   ├── napi_open_network_settings.cpp
│   ├── napi_open_settings_page_util.cpp
│   ├── napi_open_network_settings.h
│   └── napi_open_settings_page_util.h
└── util/                    # 工具函数
```

**导出 API**（13 个）：
- getURI, getValue, setValue
- getUriSync, getValueSync, setValueSync
- enableAirplaneMode, canShowFloating
- registerKeyObserver, unregisterKeyObserver
- openNetworkManagerSettings, openInputMethodSettings, openInputMethodDetail

**模块名**：settings
**注册点**：napi/settings/native_module.cpp:77
**产物**：libsettings.z.so

#### napi/intelligentscene/

**职责**：提供智能场景 API（免打扰模式等）

**文件结构**：
```
napi/intelligentscene/
├── native_module.cpp          # 模块注册点
├── napi_nodisturb.cpp        # 免打扰模式实现
├── napi_intelligent_scene_log.h # 日志工具
└── common/                  # 公共代码
    ├── common_utils.cpp
    ├── intelligence_inner_errors.cpp
    └── intelligence_inner_errors.h
```

**导出 API**（2 个）：
- isDoNotDisturbEnabled
- isNotifyAllowedInDoNotDisturb

**模块名**：intelligentscene
**注册点**：napi/intelligentscene/native_module.cpp:59
**产物**：libintelligentscene.z.so

---

### 2. ANI 模块（ani/）

#### ani/settings/

**职责**：提供 ArkTS ↔ C++ 绑定，高性能设置管理 API

**文件结构**：
```
ani/settings/
├── ani_settings.cpp          # 主要实现
├── ani_settings_observer.cpp # 观察者实现
├── ani_init_module.cpp     # 模块初始化
├── ani_settings.h           # 头文件（API 声明）
├── ani_settings_observer.h
├── open_network_settings/     # 网络设置相关
│   ├── ani_open_network_settings.cpp
│   ├── api_open_settings_page_util.cpp
│   ├── ani_open_network_settings.h
│   └── api_open_settings_page_util.h
├── util/                    # 工具函数
└── ets/                     # ArkTS 文件
    └── @ohos.settings.ets  # ANI 入口
```

**导出 ANI API**（16 个）：
- ani_get_value, ani_get_value_ext
- ani_set_value, ani_set_value_ext
- ani_get_value_sync, ani_get_value_sync_ext
- ani_set_value_sync, ani_set_value_sync_ext
- ani_get_uri_sync
- ani_register_key_observer, ani_unregister_key_observer
- ani_enable_airplane_mode
- ani_can_show_floating

**产物**：
- libsettings_ani.z.so - ANI 动态库
- settings.abc - ArkTS 字节码（/system/framework/settings.abc）

#### ani/intelligentscene/

**职责**：提供智能场景 ANI API

**文件结构**：
```
ani/intelligentscene/
├── ani_init_module.cpp      # 模块初始化
├── ani_nodisturb.cpp        # 免打扰模式实现
├── ani_nodisturb.h        # 头文件
├── common/                  # 公共代码
│   ├── ani_throw_error.cpp   # 错误处理
│   ├── ani_throw_error.h
│   ├── ani_intelligent_scene_log.h
│   └── ani_common.h
└── ets/                     # ArkTS 文件
    └── @ohos.intelligentscene.ets  # ANI 入口
```

**导出 ANI API**（2 个）：
- ani_is_do_not_disturb_enabled
- ani_is_notify_allowed

**产物**：
- libintelligentscene_ani.z.so - ANI 动态库
- intelligentscene.abc - ArkTS 字节码（/system/framework/intelligentscene.abc）

---

### 3. CJ FFI 模块（cj/）

#### cj/settings/

**职责**：提供 C ABI 绑定，兼容性最好的设置管理 API

**文件结构**：
```
cj/settings/
├── cj_settings.cpp           # 主要实现
├── cj_settings_observer.cpp   # 观察者实现
├── settings_ffi.cpp         # FFI 导出
├── cj_settings.h            # 头文件（内部 API）
├── cj_settings_observer.h
├── settings_ffi.h          # FFI 头文件
├── cj_settings_log.h        # 日志工具
└── cj_settings_utils.h       # 工具函数
```

**导出 FFI 函数**（5 个）：
- FfiSettingsSetValue
- FfiSettingsGetValue
- FfiSettingsRegisterKeyObserver
- FfiSettingsUnregisterKeyObserver
- FfiSettingsGetUriSync

**内部 API**（详见 [05_Inner_API.md](05_Inner_API.md)）：
- Settings::SetValue
- Settings::GetValue
- Settings::GetUriSync
- GetDataShareHelper
- GetUserIdStr
- GetStageUriStr

**产物**：libcj_settings_ffi.z.so

---

### 4. Native 模块（native/）

#### native/settings/

**职责**：提供公共 Native C++ 实现，被 N-API/ANI/CJ FFI 复用

**文件结构**：
```
native/settings/
├── src/
│   ├── napi_bundle_util.cpp         # Bundle 工具实现
│   ├── napi_sys_event_util.cpp       # 系统事件工具实现
│   └── include/
│       ├── napi_bundle_util.h        # Bundle 工具头文件
│       ├── napi_sys_event_util.h      # 系统事件工具头文件
│       └── napi_settings_log.h        # 日志工具头文件
└── BUILD.gn                        # 构建配置
```

**对外接口**（详见 [05_Inner_API.md](05_Inner_API.md)）：
1. **BundleUtil**：
   - GetCurrentBundleName()
   - GetCurrentVersionName()
   - InitCurrentBundleInfo()

2. **SysEventUtil**：
   - 系统事件上报（待确认具体 API）

**产物**：libsettings_common.z.so

**依赖的外部 SA**：
- BUNDLE_MGR_SERVICE_SYS_ABILITY_ID（通过 SystemAbilityManager）
- 用途：获取当前应用 Bundle 信息

---

### 5. 公共模块（common/）

#### common/component/

**职责**：UI 组件库，提供可复用的 UI 组件

**文件数**：19 个 ETS 文件

**主要组件**：
- settingItemComponent - 设置项组件
- switchComponent - 开关组件
- sliderComponent - 滑块组件
- radioListComponent - 单选列表组件
- dialogComponent - 对话框组件
- ...（详见 NOTES.md）

**控制器**：
- SwitchController - 开关控制器
- ISettingsController - 设置控制器接口
- BaseSettingsController - 基础设置控制器

#### common/search/

**职责**：搜索功能

**文件数**：19 个 ETS 文件

**主要组件**：
- searchHeader - 搜索头部
- resultComponent - 结果组件
- SearchDataProvider - 搜索数据提供者
- SearchModel - 搜索模型
- SearchData - 搜索数据

**数据存储**：
- 使用 RDB（关系型数据库）
- SQL 操作：DELETE（SearchConfig.ts:38）

#### common/settingsBase/

**职责**：设置基础功能

**文件数**：待详细分析

#### common/utils/

**职责**：工具函数和基础类

**文件数**：19 个 ETS 文件

**主要模块**：
- BaseModel - 基础模型
- BasicDataSource - 基础数据源
- GlobalResourceManager - 全局资源管理器
- DateAndTimeUtil - 日期时间工具
- SubscriberUtil - 订阅工具
- Secure - 安全工具

---

### 6. 产品模块（product/）

#### product/phone/

**职责**：手机产品的 UI 实现

**主入口**：product/phone/src/main/ets/MainAbility/

**主页面**（product/phone/src/main/ets/pages/）：
- settingList.ets - 主设置列表（首页）
- screenAndBrightness.ets - 屏幕与亮度
- applicationInfo.ets - 应用信息
- storage.ets - 存储
- privacy.ets - 隐私
- vpn*.ets - VPN 相关页面（多个）
- accessibility*.ets - 无障碍功能（多个）
- ...（共 59 个 ETS 文件）

**模型层**（product/phone/src/main/ets/model/）：
- bluetoothImpl/ - 蓝牙模型
- wifiImpl/ - Wi-Fi 模型
- displayAndBrightnessImpl/ - 显示与亮度模型
- dateAndTimeImpl/ - 日期与时间模型
- vpnImpl/ - VPN 模型
- accessibilityImpl/ - 无障碍模型
- dataRdbImpl/ - 数据库模型

**权限声明**（product/phone/src/main/module.json5）：
```json
{
  "name": "ohos.permission.UPDATE_CONFIGURATION",
  "reason": "$string:UPDATE_CONFIGURATION"
}
```

**主 Ability**：
- MainAbility - 主能力
- AppInfoAbility - 应用信息能力
- BluetoothAbility - 蓝牙能力
- ...（多个辅助 Ability）

#### product/wearable/

**职责**：可穿戴设备产品（预留）

**状态**：已注释（build-profile.json5）

---

## 模块依赖关系

### 依赖层次

```
应用层（product/phone）
    ↓
公共层（common/）
    ↓
API 层（napi/ + ani/ + cj/）
    ↓
Native 层（native/settings）
    ↓
系统服务层（DataShare + BundleManager）
```

### 关键依赖

| 依赖类型 | 提供者 | 消费者 |
|----------|--------|----------|
| Native C++ | native/settings | napi/, ani/, cj/ |
| DataShare Helper | DataShare 框架 | napi/, ani/, cj/ |
| Bundle Manager | BundleManagerService | native/settings |
| UI 组件 | common/component | product/phone |
| 工具函数 | common/utils | product/phone, common/ |

---

## 模块边界与接口

### 对外接口

1. **N-API 模块**：
   - 对外 API：13 个 JS 函数
   - 文件：napi/settings/native_module.cpp:77
   - 稳定性：稳定（公共 API）

2. **ANI 模块**：
   - 对外 API：16 个 ANI 函数
   - 文件：ani/settings/ani_settings.cpp
   - 稳定性：稳定（公共 API）

3. **CJ FFI 模块**：
   - 对外 API：5 个 FFI 函数
   - 文件：cj/settings/src/settings_ffi.h
   - 稳定性：稳定（标注 innerapi_tags = "platformsdk"）

4. **Native 模块**：
   - 对外 API：2 个工具类
   - 文件：native/settings/src/include/
   - 稳定性：稳定（内部 API）

### 内部接口

- **公共组件**（common/component）：被 product/phone 使用
- **公共搜索**（common/search）：被 product/phone 使用
- **公共工具**（common/utils）：被所有模块使用
- **Native 实现**（native/settings）：被 N-API/ANI/CJ 复用

---

## 数据流

### 设置数据流

```
应用调用 API（getValue/setValue）
    ↓
N-API/ANI/CJ FFI 层
    ↓
调用 Native 函数
    ↓
创建 DataShareHelper
    ↓
访问 DataAbility（global/system/secure 表）
    ↓
返回结果
```

### UI 数据流

```
用户操作 UI（product/phone）
    ↓
调用公共组件（common/component）
    ↓
调用模型层（product/phone/model）
    ↓
通过 N-API/ANI 调用 API
    ↓
更新数据存储
    ↓
触发观察者回调
    ↓
刷新 UI
```

---

## 相关跳转

- **[00_Overview.md](00_Overview.md)** - 项目概览
- **[01_Positioning.md](01_Positioning.md)** - 项目定位与边界
- **[03_Architecture.md](03_Architecture.md)** - 架构说明

---

**最后更新**：2026-02-06 00:11:23
