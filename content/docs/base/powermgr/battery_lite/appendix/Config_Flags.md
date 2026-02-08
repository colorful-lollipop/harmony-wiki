# 配置开关文档

> **目的**: 梳理 battery_lite 项目中的所有可配置参数、宏定义和 feature flags。  
> **适用范围**: 需要定制化编译或优化性能的开发者。  
> **关键结论**: 项目配置主要通过 GN 构建参数控制，支持 mini/small 系统类型切换。

---

## 1. 系统级配置

### 1.1 batterymgr.gni 核心参数

**文件**: `base/powermgr/battery_lite/batterymgr.gni`

```gn
# 路径变量
batterymgr_path = "//base/powermgr/battery_lite"
batterymgr_frameworks_path = "${batterymgr_path}/frameworks/native"
batterymgr_interfaces_path = "${batterymgr_path}/interfaces"
batterymgr_innerkits_path = "${batterymgr_interfaces_path}/innerkits"
batterymgr_kits_path = "${batterymgr_interfaces_path}/kits"
batterymgr_services_path = "${batterymgr_path}/services"
batterymgr_utils_path = "//base/powermgr/powermgr_lite/utils"

# 系统类型参数 (通过 ohos_kernel_type 自动判断)
declare_args() {
  battery_mini_system = false
  battery_small_system = false
}

if (ohos_kernel_type == "liteos_m") {
  battery_mini_system = true
  battery_library_type = "static_library"   # 静态库
  battery_system_type = "mini"               # mini 系统标识
} else {
  battery_small_system = true
  battery_library_type = "shared_library"   # 共享库
  battery_system_type = "small"              # small 系统标识
}
```

**证据**: `batterymgr.gni:17-44`。

### 1.2 config.gni 功能开关

**文件**: `base/powermgr/battery_lite/config.gni`

```gn
declare_args() {
  # 当前为空，未定义功能开关
  # enable_screensaver = false  # 示例：屏保功能开关
}
```

**证据**: `config.gni:14-16`（当前为空配置）。

---

## 2. 服务配置参数

### 2.1 任务配置常量

**文件**: `services/include/battery_device.h:42-43`

```c
#define TASK_CONFIG_STACK_SIZE 0x800   // 栈大小: 2048 字节 (2KB)
#define TASK_CONFIG_QUEUE_SIZE 20       // 消息队列深度: 20
```

| 参数 | 值 | 说明 |
|------|-----|------|
| `TASK_CONFIG_STACK_SIZE` | `0x800` (2048) | 服务线程栈大小 |
| `TASK_CONFIG_QUEUE_SIZE` | `20` | 消息队列最大容量 |

**优先级配置** (`services/src/battery_device.c:54-59`):

```c
TaskConfig GetTaskConfig(Service *service)
{
    TaskConfig config = {
        LEVEL_HIGH,              // 优先级级别：高
        PRI_BELOW_NORMAL,        // 线程优先级：低于普通
        TASK_CONFIG_STACK_SIZE,  // 栈大小
        TASK_CONFIG_QUEUE_SIZE,  // 队列大小
        SHARED_TASK             // 共享任务模式
    };
    return config;
}
```

### 2.2 错误码常量

**文件**: `services/include/battery_device.h:37-40`

```c
#define BATTERY_ERROR_UNKNOWN (-1)       // 未知错误
#define BATTERY_ERROR_INVALID_ID (-2)     // 无效 ID
#define BATTERY_ERROR_INVALID_PARAM (-3)  // 无效参数
#define BATTERY_OK 0                      // 成功
```

### 2.3 服务名常量

**文件**: `frameworks/native/include/battery_mgr.h:24-25`

```c
#define BATTERY_INNER  "battery_feature"  // 特征名称
#define BATTERY_SERVICE "battery_service" // 服务名称
```

| 常量 | 值 | 用途 |
|------|-----|------|
| `BATTERY_INNER` | `"battery_feature"` | SAMgr Feature 标识 |
| `BATTERY_SERVICE` | `"battery_service"` | SAMgr Service 标识 |

### 2.4 电池技术字符串长度

**文件**: `services/include/ibattery.h:24`

```c
#define BATTECHNOLOGY_LEN 64  // 电池技术字符串最大长度
```

---

## 3. 构建目标配置

### 3.1 框架层配置

**mini 系统** (`frameworks/native/src/mini/BUILD.gn`):

```gn
static_library("battery_impl") {
  sources = [ "battery_framework.c" ]
  include_dirs = [
    "${batterymgr_kits_path}",
    "${batterymgr_frameworks_path}/include",
    "${batterymgr_services_path}/include",
    "${batterymgr_frameworks_path}/include/${battery_system_type}",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
  ]
  deps = [
    "//device/soc/hisilicon/hi3861v100/hi3861_adapter/kal/posix:posix",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]
}
```

**small 系统** (`frameworks/BUILD.gn:28-34`):

```gn
local_deps += [
  "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
  "//base/powermgr/battery_lite/frameworks/native/src/small:battery_impl",
  "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
]
```

### 3.2 JS 模块配置

**文件**: `frameworks/js/BUILD.gn`

```gn
lite_component("ace_battery_kits") {
  features = [ "builtin:ace_kit_battery" ]
}
```

---

## 4. 组件配置

### 4.1 bundle.json 配置

**文件**: `bundle.json`

```json
{
  "name": "@ohos/battery_lite",
  "version": "3.1",
  "subsystem": "powermgr",
  "syscap": [
    "SystemCapability.PowerManager.BatteryManager.Lite"
  ],
  "adapted_system_type": [
    "mini",
    "small"
  ],
  "rom": "22KB",
  "ram": "~10KB",
  "deps": {
    "components": [
      "utils_lite",
      "samgr_lite",
      "ipc",
      "hilog_lite"
    ],
    "third_party": [
      "bounds_checking_function"
    ]
  }
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `subsystem` | `powermgr` | 所属子系统 |
| `adapted_system_type` | `["mini", "small"]` | 支持的系统类型 |
| `rom` | `22KB` | ROM 占用 |
| `ram` | `~10KB` | RAM 占用 |

---

## 5. 条件编译

### 5.1 系统类型判断

```c
// batterymgr.gni 中判断
if (ohos_kernel_type == "liteos_m") {
  battery_mini_system = true
} else {
  battery_small_system = true
}
```

### 5.2 头文件包含路径

| 系统类型 | 包含路径 |
|----------|----------|
| mini | `include/mini/` |
| small | `include/small/` |

**证据**: `frameworks/BUILD.gn:51` 使用 `${battery_system_type}` 变量。

---

## 6. 功能开关建议

### 6.1 可选功能

| 功能 | 开关变量 | 默认值 | 说明 |
|------|----------|--------|------|
| LED 控制 | `ENABLE_LED_CONTROL` | 未定义 | 启用 LED 控制接口 |
| 电池数据模拟 | `ENABLE_BATTERY_MOCK` | 已启用 | 使用模拟数据而非 HAL |
| 详细日志 | `ENABLE_DEBUG_LOG` | 未定义 | 启用调试日志 |
| 错误码扩展 | `ENABLE_EXTENDED_ERROR` | 未定义 | 返回详细错误信息 |

### 6.2 配置示例

```c
// 在编译时添加 -DENABLE_DEBUG_LOG 启用调试日志
hb build -b debug --gn-args "enable_battery_debug=true"
```

---

## 7. 性能相关配置

### 7.1 线程优先级

| 参数 | 值 | 影响 |
|------|-----|------|
| `LEVEL_HIGH` | 高优先级级别 | 服务响应速度 |
| `PRIORITY_BELOW_NORMAL` | 低于普通优先级 | CPU 占用控制 |

### 7.2 资源限制

| 资源 | 值 | 说明 |
|------|-----|------|
| 栈大小 | 2KB | 限制函数调用深度 |
| 队列大小 | 20 | 限制并发请求数 |
| ROM | 22KB | 代码存储空间 |
| RAM | 10KB | 运行时内存 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [GN 构建](05_GN_Build.md) | 构建 Targets 详细说明 |
| [目录结构](01_Directory_Structure.md) | 代码组织结构 |
| [架构说明](02_Architecture.md) | 系统架构 |
