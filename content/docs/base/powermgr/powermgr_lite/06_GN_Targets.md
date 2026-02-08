# GN Targets 与构建配置

> **目的**: 完整的 GN 构建系统文档,targets 依赖和产物映射
> **适用范围**: 构建工程师、平台工程师
> **阅读时间**: 35分钟

---

## 概述

powermgr_lite 使用 **GN (Generate Ninja)** 构建系统,支持 **mini** (LiteOS-M) 和 **small** (LiteOS-A) 两个系统变体。

### 构建入口
- **根文件**: `BUILD.gn`
- **依赖**: `powermgr.gni`, `config.gni`
- **输出**: `powermgr_lite` 组件

---

## 系统类型配置

### 配置文件: powermgr.gni

**位置**: `powermgr.gni`

**关键变量**:
```gni
# 系统类型检测
if (ohos_kernel_type == "liteos_m") {
  is_liteos_m = true
  lite_library_type = "static_library"
  system_type = "mini"
} else {
  is_liteos_a = true
  lite_library_type = "shared_library"
  system_type = "small"
}

# 路径定义
powermgr_path = "//base/powermgr/powermgr_lite"
powermgr_frameworks_path = "${powermgr_path}/frameworks"
powermgr_interfaces_path = "${powermgr_path}/interfaces"
powermgr_innerkits_path = "${powermgr_interfaces_path}/innerkits"
powermgr_kits_path = "${powermgr_interfaces_path}/kits"
powermgr_services_path = "${powermgr_path}/services"
powermgr_utils_path = "${powermgr_path}/utils"
```

### 配置文件: config.gni

**位置**: `config.gni`

**Feature Flags**:
```gni
declare_args() {
  enable_screensaver = false  # 屏保功能,默认关闭
}
```

**效果**:
- `enable_screensaver = true`: 编译屏保功能 (仅 small 系统)
- `enable_screensaver = false`: 完全排除屏保代码

---

## 根 Target

### powermgr_lite

**位置**: `BUILD.gn`

**定义**:
```gni
lite_component("powermgr_lite") {
  features = [
    "frameworks:powermgr",
    "services:powermgrservice",
  ]
}
```

**类型**: `lite_component` (元 target)
**依赖**:
- `frameworks:powermgr`
- `services:powermgrservice`

**输出**: 无 (仅聚合)

---

## Frameworks Targets

### powermgr

**位置**: `frameworks/BUILD.gn`

**定义**:
```gni
lite_library("powermgr") {
  target_type = lite_library_type  # "static_library" (mini) or "shared_library" (small)

  sources = [ "src/running_lock.c" ]

  include_dirs = [
    "include",
    "include/${system_type}",
    "//commonlibrary/utils_lite/include",
  ]
  # mini: kernel 头文件
  # small: ipc 头文件

  public_configs = [ ":powermgr_public_config" ]

  deps = [
    "src/${system_type}:powermanage_impl",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]
  # small: hilog_shared, ipc_single

  if (enable_screensaver) {
    deps += [ "src/${system_type}:screensaver_impl" ]
  }
}
```

**Sources**:
- `src/running_lock.c` - 运行锁客户端

**依赖**:
- `src/${system_type}:powermanage_impl` - 平台特定实现
- `samgr` - 系统能力管理器
- **条件**: `src/${system_type}:screensaver_impl` (屏保)

**输出**:
- Mini: `libpowermgr.a`
- Small: `libpowermgr.so`

### powermgr_public_config

**定义**:
```gni
config("powermgr_public_config") {
  include_dirs = [
    "${powermgr_innerkits_path}",
    "${powermgr_kits_path}",
  ]
}
```

**用途**: 导出公共 API 头文件路径

### powermanage_impl (Mini)

**位置**: `frameworks/src/mini/BUILD.gn`

**定义**:
```gni
static_library("powermanage_impl") {
  sources = [
    "power_manage.c",
    "power_screen_saver.c",
  ]

  include_dirs = [
    "${powermgr_frameworks_path}/include",
    "${powermgr_innerkits_path}",
    "${powermgr_kits_path}",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
  ]
}
```

**Sources**:
- `power_manage.c` - 直接调用实现
- `power_screen_saver.c` - 屏保实现

**输出**: `libpowermanage_impl.a` (静态链接到 libpowermgr.a)

### powermanage_impl (Small)

**位置**: `frameworks/src/small/BUILD.gn`

**定义**:
```gni
source_set("powermanage_impl") {
  sources = [
    "power_manage.c",
    "power_screen_saver.c",
  ]

  include_dirs = [
    "${powermgr_frameworks_path}/include",
    "${powermgr_innerkits_path}",
    "${powermgr_kits_path}",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc/include",
  ]

  cflags = [ "-fPIC" ]  # 位置无关代码 (用于共享库)
}
```

**类型**: `source_set` (仅对象文件,用于共享库)

### screensaver_impl (Mini)

**位置**: `frameworks/src/mini/BUILD.gn`

**定义**:
```gni
static_library("screensaver_impl") {
  sources = [ "power_screen_saver.c" ]
  # 基础屏保实现
}
```

### screensaver_impl (Small)

**位置**: `frameworks/src/small/BUILD.gn`

**定义**:
```gni
source_set("screensaver_impl") {
  sources = [ "power_screen_saver.c" ]
  # 完整屏保实现

  cflags = [ "-fPIC" ]
}
```

---

## Services Targets

### powermgrservice

**位置**: `services/BUILD.gn`

**定义**:
```gni
lite_library("powermgrservice") {
  target_type = lite_library_type

  sources = [
    "src/power/auto_suspend.c",
    "src/power/running_lock_hub.c",
    "src/power/suspend_controller.c",
    "src/power_manage_feature.c",
    "src/power_manage_service.c",
    "src/running_lock_mgr.c",
  ]

  # small: 屏保 sources
  # sources += local_sources

  include_dirs = [
    "include",
    "${powermgr_frameworks_path}/include",
    "${powermgr_frameworks_path}/include/${system_type}",
    "${powermgr_innerkits_path}",
    "${powermgr_kits_path}",
    "//commonlibrary/utils_lite/include",
  ]

  deps = [
    "src/power/${system_type}:powermanage_feature_impl",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]

  # small: hilog_shared, ipc_single, screensaver_featrue_impl
}
```

**Sources**:
- `src/power/auto_suspend.c` - 自动挂起线程
- `src/power/running_lock_hub.c` - 运行锁中心
- `src/power/suspend_controller.c` - 挂起控制器
- `src/power_manage_feature.c` - 电源 Feature
- `src/power_manage_service.c` - 主服务
- `src/running_lock_mgr.c` - 运行锁管理器
- **Small only**: 屏保相关 sources

**依赖**:
- `src/power/${system_type}:powermanage_feature_impl`
- `samgr`
- **Small only**: `hilog_shared`, `ipc_single`, `screensaver_featrue_impl`

**输出**:
- Mini: `libpowermgrservice.a`
- Small: `libpowermgrservice.so`

### powermanage_feature_impl (Mini)

**位置**: `services/src/power/mini/BUILD.gn`

**定义**:
```gni
static_library("powermanage_feature_impl") {
  sources = [
    "auto_suspend_loop.c",
    "power_manage_feature_impl.c",
    "running_lock_handler.c",
  ]

  include_dirs = [
    "${powermgr_services_path}/include",
    "${powermgr_frameworks_path}/include/${system_type}",
    "${powermgr_innerkits_path}",
    "${powermgr_kits_path}",
    "${powermgr_services_path}/include/power",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//kernel/liteos_m/kal/los_base/include",
  ]
}
```

**Sources**:
- `auto_suspend_loop.c` - LiteOS-M 挂起循环
- `power_manage_feature_impl.c` - Feature 实现 (直接调用)
- `running_lock_handler.c` - LiteOS-M 内核接口

**输出**: `libpowermanage_feature_impl.a`

### powermanage_feature_impl (Small)

**位置**: `services/src/power/small/BUILD.gn`

**定义**:
```gni
source_set("powermanage_feature_impl") {
  sources = [
    "auto_suspend_loop.c",
    "power_manage_feature_impl.c",
    "running_lock_handler.c",
  ]

  include_dirs = [
    "${powermgr_services_path}/include",
    "${powermgr_frameworks_path}/include/${system_type}",
    "${powermgr_innerkits_path}",
    "${powermgr_kits_path}",
    "${powermgr_services_path}/include/power",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc/include",
    "//third_party/bounds_checking_function/include",
  ]

  cflags = [ "-fPIC" ]
}
```

**Sources**:
- `auto_suspend_loop.c` - Linux 挂起循环
- `power_manage_feature_impl.c` - IPC Stub 实现
- `running_lock_handler.c` - /proc 文件操作

**输出**: 对象文件 (链接到 libpowermgrservice.so)

### screensaver_featrue_impl (Small)

**位置**: `services/src/screensaver/small/BUILD.gn`

**定义**:
```gni
source_set("screensaver_featrue_impl") {
  sources = [
    "screen_saver_feature_impl.c",
    "screen_saver_handler.cpp",
    "screen_saver_mgr.cpp",
  ]

  include_dirs = [
    "${powermgr_services_path}/include",
    "${powermgr_frameworks_path}/include/small",
    "${powermgr_frameworks_path}/include",
    "${powermgr_services_path}/include/small",
    "${powermgr_services_path}/include",
    "${powermgr_kits_path}/battery/js/builtin/include",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc/include",
    "//foundation/aafwk/ability/ability_lite/interfaces/innerkits/ability_manager/include",
    "//foundation/ace/ace_engine_lite/interfaces/inner_api/builtin/jsi",
    "//foundation/ace/ace_engine_lite/interfaces/inner_api/builtin/base",
    "//foundation/window/window_manager_lite/interfaces/innerkits",
    "//third_party/bounds_checking_function/include",
  ]

  cflags = [
    "-fPIC",
    "-Wno-unused-variable",
    "-DABILITY_WINDOW_SUPPORT",
    "-DOHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
    "-DENABLE_WINDOW=1",
  ]
}
```

**类型**: `source_set`

**Sources**:
- `screen_saver_feature_impl.c` - IPC Stub (C)
- `screen_saver_handler.cpp` - 计时器处理 (C++)
- `screen_saver_mgr.cpp` - 屏保管理器 (C++)

**输出**: 对象文件 (链接到 libpowermgrservice.so)

---

## Utils Targets

### powermgr_utils

**位置**: `utils/BUILD.gn`

**定义 (Mini)**:
```gni
static_library("powermgr_utils") {
  sources = [
    "src/power_mgr_timer_util.c",
    "src/power_mgr_time_util.c",
  ]

  include_dirs = [
    "include",
    "//commonlibrary/utils_lite/include",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//kernel/liteos_m/kal/los_base/include",
  ]
}
```

**定义 (Small)**:
```gni
source_set("powermgr_utils") {
  sources = [
    "src/power_mgr_timer_util.c",
    "src/power_mgr_time_util.c",
  ]

  include_dirs = [
    "include",
    "//commonlibrary/utils_lite/include",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
  ]

  cflags = [ "-fPIC" ]
}
```

**Sources**:
- `src/power_mgr_timer_util.c` - POSIX 计时器封装
- `src/power_mgr_time_util.c` - 时间转换工具

**输出**:
- Mini: `libpowermgr_utils.a`
- Small: 对象文件 (链接到 libpowermgrservice.so)

---

## Battery JS API Target

### libnativeapi_battery_simulator

**位置**: `interfaces/kits/battery/js/builtin/BUILD.gn`

**定义**:
```gni
ohos_static_library("libnativeapi_battery_simulator") {
  public_configs = [ ":nativeapi_battery_simulator_config" ]

  sources = [ "src/battery_module.cpp" ]

  include_dirs = [
    "//third_party/bounds_checking_function/include",
    "include",
    "//foundation/arkui/ace_engine_lite/interfaces/inner_api/builtin/jsi",
    "//foundation/arkui/ace_engine_lite/interfaces/inner_api/builtin/base",
  ]

  cflags = [ "-Wno-unused-variable" ]

  deps = []
}
```

**类型**: `ohos_static_library`

**Sources**:
- `src/battery_module.cpp` - JS 模块实现

**输出**: `libnativeapi_battery_simulator.a`

### nativeapi_battery_simulator_config

**定义**:
```gni
config("nativeapi_battery_simulator_config") {
  include_dirs = [
    "//base/powermgr/powermgr_lite/interfaces/kits/battery/js/builtin/include",
  ]
}
```

---

## 依赖图

### 完整依赖树

```
powermgr_lite (root)
├── frameworks:powermgr
│   ├── src/mini:powermanage_impl OR src/small:powermanage_impl
│   │   ├── power_manage.c
│   │   └── power_screen_saver.c
│   ├── samgr:samgr
│   └── [可选] src/mini:screensaver_impl OR src/small:screensaver_impl
│
└── services:powermgrservice
    ├── src/power/mini:powermanage_feature_impl OR src/power/small:powermanage_feature_impl
    │   ├── auto_suspend_loop.c
    │   ├── power_manage_feature_impl.c
    │   └── running_lock_handler.c
    ├── samgr:samgr
    ├── [small only] hilog_lite:hilog_shared
    ├── [small only] ipc:ipc_single
    └── [small only + enable_screensaver] src/screensaver/small:screensaver_featrue_impl
        ├── screen_saver_feature_impl.c
        ├── screen_saver_handler.cpp
        └── screen_saver_mgr.cpp
            └── utils:powermgr_utils (源文件链接)
                ├── power_mgr_timer_util.c
                └── power_mgr_time_util.c
```

### 层级依赖

| 层级 | Target | 依赖 | 被依赖 |
|-------|--------|------|--------|
| Root | powermgr_lite | 无 | 无 |
| 1 | powermgr | powermgr_lite | 2, 3 |
| 1 | powermgrservice | powermgr_lite | 2, 3 |
| 2 | powermanage_impl | powermgr | powermgrservice |
| 2 | screensaver_impl | powermgr | powermgrservice |
| 2 | powermanage_feature_impl | powermgrservice | 无 |
| 2 | screensaver_featrue_impl | powermgrservice | 无 |
| 2 | powermgr_utils | 无 | screensaver_featrue_impl |
| 3 | samgr | 1, 2 | 无 |
| 3 | hilog_shared | 2 | 无 |
| 3 | ipc_single | 2 | 无 |

---

## 编译选项

### Mini 系统编译选项
```bash
ohos_kernel_type="liteos_m"
enable_screensaver=false  # 或 true
```

**配置结果**:
- `is_liteos_m = true`
- `system_type = "mini"`
- `lite_library_type = "static_library"`
- 无 IPC 支持
- 静态链接 hilog

### Small 系统编译选项
```bash
ohos_kernel_type="liteos_a"  # 或任何非 "liteos_m" 值
enable_screensaver=true   # 或 false
```

**配置结果**:
- `is_liteos_a = true`
- `system_type = "small"`
- `lite_library_type = "shared_library"`
- IPC 支持 (ipc_single)
- 共享链接 hilog_shared
- 位置无关代码 (`-fPIC`)

### 条件编译 (Screensaver)

**配置**: `enable_screensaver = true`

**影响**:
- 编译屏保相关 targets:
  - `screensaver_impl` (frameworks)
  - `screensaver_featrue_impl` (services)
- 添加 defines:
  - `ABILITY_WINDOW_SUPPORT`
  - `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER`
  - `ENABLE_WINDOW=1`

**依赖**:
- `window_manager_lite` - 窗口管理器
- `ams` - Ability 管理服务

---

## 产物清单

### Main 产物

| Target | Mini 产物 | Small 产物 | 大小估算 |
|--------|-----------|-----------|----------|
| powermgr | libpowermgr.a | libpowermgr.so | ~8KB |
| powermgrservice | libpowermgrservice.a | libpowermgrservice.so | ~14KB |
| powermgr_utils | libpowermgr_utils.a | (链接到 service) | ~2KB |
| libnativeapi_battery_simulator | libnativeapi_battery_simulator.a | libnativeapi_battery_simulator.a | ~4KB |

### 输出位置 (bundle.json)

**Mini 系统**:
- 静态库链接到可执行文件
- 无独立 .so 文件

**Small 系统**:
- 共享库: `/usr/lib/libpowermgr.so`
- 共享库: `/usr/lib/libpowermgrservice.so`
- 静态库: `/usr/lib/libnativeapi_battery_simulator.a` (链接到 JS 引擎)

---

## 构建命令

### 编译完整组件
```bash
# Mini 系统
hb build -f powermgr/powermgr_lite --build-type debug --ccache

# Small 系统
hb build -f powermgr/powermgr_lite --build-type release --ccache --enable-screensaver=true
```

### 编译特定 target
```bash
# 仅编译 frameworks
hb build -f powermgr/powermgr_lite/frameworks:powermgr

# 仅编译 services
hb build -f powermgr/powermgr_lite/services:powermgrservice
```

### 查看构建图
```bash
hb build -f powermgr/powermgr_lite --graph
```

---

## 相关文档

- [01_Project_Boundaries.md](01_Project_Boundaries.md#feature-flags) - Feature Flags
- [02_Directory_Structure.md](02_Directory_Structure.md) - BUILD.gn 文件位置
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 产物与安装
