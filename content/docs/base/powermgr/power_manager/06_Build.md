# 构建配置与产物

本文档描述 Power Manager 模块的 GN 构建配置、编译产物和依赖关系。

## 1. GN 构建系统

### 1.1 根配置

**配置文件**: `powermgr.gni`

```gni
# 路径定义
powermgr_root_path = "//base/powermgr/power_manager"
powermgr_service_path = "${powermgr_root_path}/services"
powermgr_inner_api = "${powermgr_root_path}/interfaces/inner_api"
powermgr_utils_path = "${powermgr_root_path}/utils"

# 特性开关
power_manager_feature_runninglock = true
power_manager_feature_shutdown_reboot = true
power_manager_feature_screen_on_off = true
```

### 1.2 bundle.json 构建配置

**文件**: `bundle.json`

```json
{
  "build": {
    "group_type": {
      "base_group": [
        "//base/powermgr/power_manager/etc/init:powermgr_cfg",
        "//base/powermgr/power_manager/etc/para:powermgr_para"
      ],
      "fwk_group": [
        "//base/powermgr/power_manager/frameworks:power_napi",
        "//base/powermgr/power_manager/interfaces/inner_api:powermgr_client"
      ],
      "service_group": [
        "//base/powermgr/power_manager/sa_profile:powermgr_sa_profile",
        "//base/powermgr/power_manager/services:service"
      ]
    }
  }
}
```

---

## 2. 关键 Targets

### 2.1 服务层 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `powermgrservice` | ohos_shared_library | libpowermgrservice.so | 核心服务 |
| `service` | group | - | 聚合目标 |
| `powermgr_stub` | ohos_source_set | - | IPC Stub |
| `powermgr_proxy` | ohos_source_set | - | IPC Proxy |
| `powermgr_interface` | idl_gen_interface | 生成 | IDL 接口 |

**Source 文件**:
- `power_mgr_service.cpp` (113KB)
- `power_state_machine.cpp` (121KB)
- `power_mgr_ipc_adapter.cpp`
- `runninglock/*.cpp`
- `suspend/*.cpp`
- `shutdown/*.cpp`
- 等等

### 2.2 框架层 Targets

| Target | 类型 | 输出 | 安装路径 | 说明 |
|--------|------|------|----------|------|
| `power` | ohos_shared_library | libpower.so | system/module/ | Power N-API |
| `runninglock` | ohos_shared_library | librunninglock.so | system/module/ | RunningLock N-API |
| `power_napi` | ohos_shared_library | libpower_napi.so | - | Power N-API 库 |
| `runninglock_napi` | ohos_shared_library | librunninglock_napi.so | - | RunningLock N-API 库 |

### 2.3 工具层 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `power_utils` | ohos_source_set | - | 公共工具 |
| `power_ffrt` | ohos_shared_library | libpower_ffrt.so | FFRT 工具 |
| `power_hookmgr` | ohos_shared_library | libpower_hookmgr.so | 钩子管理 |
| `power_setting` | ohos_shared_library | libpower_setting.so | 设置工具 |
| `power_permission` | ohos_shared_library | libpower_permission.so | 权限工具 |
| `power_sysparam` | ohos_shared_library | libpower_sysparam.so | 系统参数 |
| `power_vibrator` | ohos_shared_library | libpower_vibrator.so | 振动器 |
| `power_ability` | ohos_shared_library | libpower_ability.so | 能力工具 |
| `power-shell` | ohos_executable | power-shell | Shell 工具 |

### 2.4 Taihe/ETS Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `power_manager_taihe_native` | taihe_shared_library | libpower_manager_taihe_native.so | Taihe 原生桥 |
| `power_abc` | generate_static_abc | power_abc.abc | ArkTS 字节码 |
| `power_manager_runninglock_taihe_native` | taihe_shared_library | libpower_manager_runninglock_taihe_native.so | Taihe 锁桥 |
| `runningLock_abc` | generate_static_abc | runningLock_abc.abc | ArkTS 字节码 |

### 2.5 Cangjie FFI Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cj_power_ffi` | ohos_shared_library | libcj_power_ffi.so | Power FFI |
| `cj_running_lock_ffi` | ohos_shared_library | libcj_running_lock_ffi.so | RunningLock FFI |

### 2.6 其他 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `power_dialog_hap` | ohos_app | power_dialog.hap | 电源对话框 |
| `powermgr_sa_profile` | ohos_sa_profile | 3301.json | SA 配置 |
| `powermgr_client` | ohos_shared_library | libpowermgr_client.so | 客户端库 |
| `power_napi_utils` | ohos_source_set | - | N-API 工具 |

---

## 3. 编译产物清单

### 3.1 系统库 (.so)

| 产物 | 来源 Target | 安装路径 | 用途 |
|------|-------------|----------|------|
| `libpowermgrservice.so` | powermgrservice | system/lib/ | 核心服务 |
| `libpowermgr_client.so` | powermgr_client | system/lib/ | 客户端库 |
| `libpower.so` | power | system/module/ | Power N-API |
| `librunninglock.so` | runninglock | system/module/ | RunningLock N-API |
| `libpower_ffrt.so` | power_ffrt | system/lib/ | FFRT 工具 |
| `libpower_hookmgr.so` | power_hookmgr | system/lib/ | 钩子管理 |
| `libpower_setting.so` | power_setting | system/lib/ | 设置工具 |
| `libpower_permission.so` | power_permission | system/lib/ | 权限检查 |
| `libpower_sysparam.so` | power_sysparam | system/lib/ | 系统参数 |
| `libpower_vibrator.so` | power_vibrator | system/lib/ | 振动器 |
| `libpower_ability.so` | power_ability | system/lib/ | 能力集成 |
| `libpower_manager_taihe_native.so` | taihe | system/lib/ | Taihe 桥接 |
| `libcj_power_ffi.so` | cj_power_ffi | system/lib/ | CJ FFI |

### 3.2 可执行文件

| 产物 | 来源 Target | 说明 |
|------|-------------|------|
| `power-shell` | power-shell | Shell 调试工具 |

### 3.3 HAP 文件

| 产物 | 来源 Target | 安装路径 | 说明 |
|------|-------------|----------|------|
| `power_dialog.hap` | power_dialog_hap | app/com.ohos.powerdialog | 电源对话框 |

### 3.4 ABC 文件

| 产物 | 来源 Target | 说明 |
|------|-------------|------|
| `power_abc.abc` | power_abc | ArkTS 字节码 |
| `runningLock_abc.abc` | runningLock_abc | ArkTS 字节码 |

### 3.5 配置文件

| 产物 | 来源 Target | 安装路径 |
|------|-------------|----------|
| `power_mode_config.xml` | power_mode_config | system/etc/power_config/ |
| `power_suspend.json` | power_suspend_config | system/etc/power_config/ |
| `power_wakeup.json` | power_wakeup_config | system/etc/power_config/ |
| `power_vibrator.json` | power_vibrator_config | system/etc/power_config/ |
| `3301.json` | powermgr_sa_profile | system/profile/ |
| `power_manager.cfg` | powermgr_cfg | etc/init/ |

---

## 4. 依赖关系

### 4.1 外部依赖

```json
// bundle.json dependencies
"ability_runtime",       // 能力运行时
"access_token",          // 权限
"battery_manager",       // 电池
"common_event_service",  // 公共事件
"display_manager",       // 显示管理
"drivers_interface_power", // Power HDI
"ets_runtime",           // ETS 运行时
"ffrt",                 // 异步任务
"hiview",               // 日志
"ipc",                  // IPC 框架
"safwk",                // SA 框架
"samgr",                // 服务管理
"sensor",               // 传感器
"window_manager",       // 窗口管理
```

### 4.2 内部依赖

```
services/BUILD.gn
├── powermgrservice
│   ├── deps: utils/*, interfaces/*
│   └── external_deps: ability_runtime, ipc, safwk, samgr

frameworks/napi/BUILD.gn
├── power
│   ├── deps: interfaces/inner_api
│   └── external_deps: napi
└── runninglock
    ├── deps: interfaces/inner_api
    └── external_deps: napi

interfaces/inner_api/BUILD.gn
├── powermgr_client
│   └── deps: zidl
├── powermgr_stub
│   └── deps: interfaces
└── powermgr_proxy
    └── deps: interfaces
```

---

## 5. Feature Flags

### 5.1 Feature Flags 列表

| 特性开关 | 默认值 | 说明 |
|----------|--------|------|
| `power_manager_feature_allow_interrupting_powerkey_off` | true | 允许电源键中断关机 |
| `power_manager_feature_tv_dreaming` | false | TV dreaming |
| `power_manager_feature_poweroff_charge` | false | 关机充电 |
| `power_manager_feature_runninglock` | true | 运行锁功能 |
| `power_manager_feature_shutdown_reboot` | true | 关机重启功能 |
| `power_manager_feature_screen_on_off` | true | 亮灭屏功能 |
| `power_manager_feature_power_state` | true | 电源状态 |
| `power_manager_feature_power_mode` | true | 电源模式 |
| `power_manager_feature_wakeup_action` | false | 唤醒动作 |
| `power_manager_feature_power_dialog` | true | 电源对话框 |
| `power_manager_feature_enable_s4` | false | 启用 S4 休眠 |
| `power_manager_feature_enable_suspend_with_tag` | false | 带标签挂起 |
| `power_manager_feature_doubleclick` | true | 双击唤醒 |
| `power_manager_feature_pickup` | true | 抬腕唤醒 |
| `power_manager_feature_movement` | true | 运动唤醒 |
| `power_manager_feature_force_sleep_broadcast` | false | 强制休眠广播 |

### 5.2 条件编译

```gni
# 条件编译示例
if (power_manager_feature_enable_s4) {
  defines += [ "POWER_MANAGER_POWER_ENABLE_S4" ]
}

if (power_manager_feature_enable_suspend_with_tag) {
  defines += [ "POWER_MANAGER_ENABLE_SUSPEND_WITH_TAG" ]
}
```

---

## 6. 构建命令

### 6.1 完整构建

```bash
# 使用 hb 工具
hb set -p <platform>
hb build -f

# 使用 gn + ninja
gn gen out/powermgr
ninja -C out/powermgr //base/powermgr/power_manager:services
```

### 6.2 单独构建模块

```bash
# 构建服务
ninja -C out //base/powermgr/power_manager/services:powermgrservice

# 构建 N-API
ninja -C out //base/powermgr/power_manager/frameworks/napi/power:power
ninja -C out //base/powermgr/power_manager/frameworks/napi/runninglock:runninglock

# 构建工具库
ninja -C out //base/powermgr/power_manager/utils/ffrt:power_ffrt
ninja -C out //base/powermgr/power_manager/utils/hookmgr:power_hookmgr
```

### 6.3 查看依赖

```bash
# 查看 target 依赖
gn desc out //base/powermgr/power_manager/services:powermgrservice

# 查看所有 targets
gn ls //base/powermgr/power_manager/
```

---

## 7. 安装路径

### 7.1 系统库

```
/system/lib/
├── libpowermgrservice.so
├── libpowermgr_client.so
├── libpower_ffrt.so
├── libpower_hookmgr.so
├── libpower_setting.so
├── libpower_permission.so
├── libpower_sysparam.so
├── libpower_vibrator.so
├── libpower_ability.so
├── libpower_manager_taihe_native.so
└── libcj_power_ffi.so
```

### 7.2 模块库

```
/system/module/
├── libpower.so
├── librunninglock.so
└── libcj_running_lock_ffi.so
```

### 7.3 配置文件

```
/system/etc/power_config/
├── power_mode_config.xml
├── power_suspend.json
├── power_wakeup.json
└── power_vibrator.json

/system/profile/
└── 3301.json

/etc/init/
└── power_manager.cfg
```

---

## 8. 产物验证

### 8.1 检查构建产物

```bash
# 列出构建产物
ls -la out/powermgr/lib/
ls -la out/powermgr/system/lib/
ls -la out/powermgr/system/module/
```

### 8.2 检查符号表

```bash
# 查看导出符号
nm -D out/powermgr/system/module/libpower.so | grep T

# 查看依赖
ldd out/powermgr/system/lib/libpowermgrservice.so
```

---

## 9. 关键 BUILD.gn 文件

| 功能 | 文件路径 |
|------|----------|
| 根配置 | `powermgr.gni` |
| 服务构建 | `services/BUILD.gn` |
| N-API Power | `frameworks/napi/power/BUILD.gn` |
| N-API RunningLock | `frameworks/napi/runninglock/BUILD.gn` |
| Inner API | `interfaces/inner_api/BUILD.gn` |
| Utils | `utils/BUILD.gn` |
| ETs Taihe | `frameworks/ets/taihe/power/BUILD.gn` |
| Cangjie FFI | `frameworks/cj/power/BUILD.gn` |
