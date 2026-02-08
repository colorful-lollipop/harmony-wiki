# 目录结构与模块职责

> **目的**: 详细说明 battery_manager 项目的目录结构、每个目录的职责和关键文件

**适用范围**: 顶层目录结构、模块职责、关键文件清单

---

## 完整目录树（不含测试）

```
base/powermgr/battery_manager
├── charger/                     # 关机充电模块（可选）
│   ├── include/                # 充电模块头文件
│   ├── resources/              # 充电动画资源
│   ├── sa_profile/             # 充电 SA 配置
│   └── src/                   # 充电模块源代码
├── figures/                    # 架构图资源
├── frameworks/                 # Framework 层
│   ├── napi/                 # N-API 绑定层（JS API）
│   ├── native/               # Native 客户端实现
│   ├── capi/                # C API
│   ├── cj/                  # CJ (ArkUI-X) 绑定
│   └── ets/taihe/          # Taihe (ArkTS) 模块
│       ├── batteryInfo/        # 电池信息 ArkTS 模块
│       └── charger/          # 充电 ArkTS 模块
├── interfaces/                 # 接口层
│   ├── inner_api/            # 内部 API（C++）
│   └── kits/                # C Kit API
├── sa_profile/                 # SA 配置文件
├── services/                   # 服务层
│   ├── native/               # Native 服务实现
│   │   ├── include/          # 服务头文件
│   │   ├── notification/     # 通知功能
│   │   ├── profile/          # 配置文件
│   │   ├── resources/        # 资源文件
│   │   └── src/             # 服务源代码
│   └── zidl/                 # ZIDL 接口层（IPC）
├── utils/                      # 工具和通用层
│   ├── hookmgr/             # Hook 管理器
│   └── native/              # 通用工具
├── batterymgr.gni              # GN 构建配置
├── batterymgr.yaml             # HiSysEvent 配置
└── bundle.json                 # 组件配置
```

---

## interfaces/ - 接口层

### 职责

定义内部 API，供模块内部跨模块调用，以及暴露给外部模块使用的接口。

### 目录结构

#### inner_api/ - 内部 API

| 子目录 | 职责 | 关键文件 |
|--------|------|---------|
| `native/include/` | C++ 内部 API 头文件 | battery_info.h, battery_srv_client.h, battery_srv_errors.h, napi_utils.h |
| `native/src/` | Native 客户端实现（在 frameworks/native） | - |

#### kits/ - C Kit API

| 子目录 | 职责 | 关键文件 |
|--------|------|---------|
| `c/` | C 语言 API | ohbattery_info.h, ohbattery_info.cpp |

### 关键头文件

| 文件 | 说明 | 证据 |
|------|------|------|
| `battery_info.h` | 电池信息类定义（BatteryInfo、枚举） | `interfaces/inner_api/native/include/battery_info.h:16-474` |
| `battery_srv_client.h` | 电池服务客户端接口（BatterySrvClient） | `interfaces/inner_api/native/include/battery_srv_client.h:16-126` |
| `battery_srv_errors.h` | 电池服务错误码定义（BatteryError） | `interfaces/inner_api/native/include/battery_srv_errors.h` |
| `napi_utils.h` | N-API 工具函数定义（NapiUtils 类） | `interfaces/inner_api/native/include/napi_utils.h` |

---

## services/ - 服务层

### 职责

实现电池服务核心逻辑，监听 HDI 驱动事件，处理 IPC 客户端请求，上报状态变化。

### 目录结构

#### native/ - Native 服务实现

| 子目录 | 职责 | 关键文件 |
|--------|------|---------|
| `include/` | 服务头文件 | battery_service.h, battery_callback.h, battery_config.h, battery_dump.h, battery_light.h, battery_notify.h, charging_sound.h |
| `src/` | 服务实现 | battery_service.cpp, battery_callback.cpp, battery_config.cpp, battery_dump.cpp, battery_light.cpp, battery_notify.cpp, charging_sound.cpp |
| `notification/` | 通知模块 | notification_center.h, notification_manager.h, notification_decorator.h, notification_locale.h, ibattery_notification.h, button_event.h |
| `profile/` | 配置文件 | battery_config.json, battery_vibrator.json |
| `resources/` | 资源文件 | locale_path.json, string.json (多语言) |

#### zidl/ - ZIDL 接口层

| 文件 | 说明 |
|------|------|
| `IBatterySrv.idl` | 电池服务 IPC 接口定义（18 个方法） |
| `BUILD.gn` | ZIDL 编译配置 |

### 关键头文件

| 文件 | 说明 | 证据 |
|------|------|------|
| `battery_service.h` | 电池服务主类（BatteryService） | `services/native/include/battery_service.h:16-210` |
| `battery_callback.h` | HDI 电池回调接口（BatteryCallback） | `services/native/include/battery_callback.h` |
| `battery_config.h` | 电池配置管理类 | `services/native/include/battery_config.h` |
| `battery_notify.h` | 电池通知管理类（BatteryNotify） | `services/native/include/battery_notify.h` |
| `battery_light.h` | 电池指示灯控制类（BatteryLight） | `services/native/include/battery_light.h` |

---

## frameworks/ - 框架层

### 职责

实现客户端代码，连接到电池服务，并向应用层（JS/Native/C）提供接口。

### 目录结构

#### napi/ - N-API 绑定层

| 文件 | 说明 |
|------|------|
| `src/battery_info.cpp` | batteryInfo 模块（13 个 GETTER + 3 个配置函数） |
| `src/system_battery.cpp` | battery 模块（异步 getStatus 函数） |
| `src/charger.cpp` | charger 模块（ChargeType 枚举） |
| `src/napi_utils.cpp` | N-API 工具函数实现 |
| `src/napi_error.cpp` | N-API 错误处理实现 |
| `include/` | N-API 头文件 |

#### native/ - Native 客户端

| 文件 | 说明 |
|------|------|
| `src/battery_srv_client.cpp` | BatterySrvClient 实现（IPC 客户端） |

#### capi/ - C API

| 文件 | 说明 |
|------|------|
| `battery_info/ohbattery_info.cpp` | C API 实现 |

#### cj/ - CJ FFI

| 文件 | 说明 |
|------|------|
| `src/battery_info_ffi.cpp` | CJ FFI 实现（10 个导出函数） |
| `include/battery_info_ffi.h` | CJ FFI 头文件 |

#### ets/taihe/ - Taihe (ArkTS)

| 子目录 | 文件 | 说明 |
|--------|------|------|
| `batteryInfo/` | ohos.batteryInfo.impl.cpp, ani_constructor.cpp | 电池信息 ArkTS 模块（17 个导出函数） |
| `charger/` | ohos.charger.impl.cpp, ani_constructor.cpp | 充电 ArkTS 模块（空实现） |

---

## charger/ - 关机充电模块（可选）

### 职责

提供设备关机状态下的充电界面和功能，包括充电动画、电量显示、低电量提示等。

### 目录结构

| 子目录 | 职责 | 关键文件 |
|--------|------|---------|
| `include/` | 充电模块头文件 | charger_thread.h, battery_thread.h, charger_animation.h, charger_graphic_engine.h, animation_config.h, battery_config.h |
| `include/dev/` | 设备驱动头文件 | graphic_dev.h, fbdev_driver.h, display_drv.h, drm_driver.h |
| `src/` | 充电模块实现 | charger.cpp, charger_thread.cpp, battery_thread.cpp, charger_animation.cpp, charger_graphic_engine.cpp |
| `src/dev/` | 设备驱动实现 | graphic_dev.cpp, fbdev_driver.cpp, drm_driver.cpp |
| `sa_profile/` | 充电动画配置 | animation.json |
| `resources/` | 充电动画资源 | 图片资源 |

---

## utils/ - 工具和通用层

### 职责

提供通用工具函数、日志、单例模式、内存保护等基础设施支持。

### 目录结构

#### native/ - 通用工具

| 文件 | 说明 |
|------|------|
| `include/battery_log.h` | 日志工具（BATTERY_HILOG* 宏） |
| `include/power_common.h` | 通用定义和常量 |
| `include/power_mgr_errors.h` | 电源管理错误码 |
| `include/sp_singleton.h` | 延迟初始化单例模板 |
| `include/battery_xcollie.h` | XCollie 监控工具 |
| `include/memory_guard.h` | 内存保护工具 |
| `include/battery_mgr_cjson_utils.h` | JSON 工具函数 |
| `src/battery_xcollie.cpp` | XCollie 监控实现 |

#### hookmgr/ - Hook 管理器

| 文件 | 说明 |
|------|------|
| `include/battery_hookmgr.h` | Hook 管理器接口 |
| `src/battery_hookmgr.cpp` | Hook 管理器实现 |

---

## sa_profile/ - SA 配置

### 职责

定义 System Ability 配置，包括 SA ID、进程名、库路径等。

### 关键文件

| 文件 | 说明 | 证据 |
|------|------|------|
| `3302.json` | 电池服务（SA 3302）配置 | `sa_profile/3302.json:1-13` |

---

## 配置文件

### 项目级配置

| 文件 | 说明 | 证据 |
|------|------|------|
| `batterymgr.gni` | GN 构建配置（路径、条件编译、特性开关） | `batterymgr.gni:16-96` |
| `batterymgr.yaml` | HiSysEvent 事件配置（BATTERY 域） | `batterymgr.yaml:14-31` |
| `bundle.json` | 组件配置（依赖、构建目标、系统能力） | `bundle.json:1-133` |

### 服务级配置

| 文件 | 说明 | 证据 |
|------|------|------|
| `services/native/profile/battery_config.json` | 电池服务配置（电量等级阈值、温度阈值） | `services/native/profile/battery_config.json` |
| `services/native/profile/battery_vibrator.json` | 电池振动配置 | `services/native/profile/battery_vibrator.json` |

### 充电动画配置

| 文件 | 说明 | 证据 |
|------|------|------|
| `charger/sa_profile/animation.json` | 充电动画配置（组件位置、颜色、资源路径） | `charger/sa_profile/animation.json:1-66` |

---

## 代码统计（不含测试）

| 类型 | 数量 |
|------|------|
| 源文件 (.cpp/.cc) | 93 |
| 总代码文件 (.cpp/.cc/.h/.hpp) | 89 |
| BUILD.gn 文件（不含 test） | 18 |
| .gni 文件 | 1 |

---

## 相关跳转

- [系统架构](03_Architecture.md)
- [N-API 文档](04_NAPI_API.md)
- [GN Targets](06_GN_Targets.md)

---

**返回**: [导航](SUMMARY.md)
