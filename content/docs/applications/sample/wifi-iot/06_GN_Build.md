# GN 构建系统

## 目的

本文档详细描述项目的 GN 构建配置，包括所有 targets、依赖关系和编译选项。

## 适用范围

- 了解项目构建流程的开发者
- 需要修改构建配置的工程师
- 学习 GN 构建脚本的初学者

## 关键结论

1. **静态库为主**: 所有模块编译为 static_library 或 source_set
2. **无循环依赖**: 各模块独立，无相互依赖
3. **依赖外部组件**: 依赖 utils_lite、liteos_m、peripheral、samgr_lite
4. **简单配置**: GN 脚本简洁，易于理解和修改

## BUILD.gn 文件清单

### 根目录 BUILD.gn

| 文件 | 路径 | 说明 |
|------|------|------|
| BUILD.gn | app/BUILD.gn | 主构建脚本 |

### 子目录 BUILD.gn

| 目录 | 路径 | 说明 |
|------|------|------|
| startup | app/startup/BUILD.gn | 启动模块（空） |
| demolink | app/demolink/BUILD.gn | Demo SDK 模块 |
| iothardware | app/iothardware/BUILD.gn | IoT 硬件模块 |
| samgr | app/samgr/BUILD.gn | SAMGR 示例模块 |

证据：
- `app/BUILD.gn`: 主构建脚本
- `app/startup/BUILD.gn`: 启动模块构建
- `app/demolink/BUILD.gn`: demolink 模块构建
- `app/iothardware/BUILD.gn`: iothardware 模块构建
- `app/samgr/BUILD.gn`: samgr 模块构建

## Target 清单（按模块分组）

### 模块: startup

**文件**: `app/startup/BUILD.gn`

| Target 名称 | 类型 | 输出文件 | sources | include_dirs | deps | 备注 |
|-------------|------|---------|---------|--------------|------|------|
| startup | source_set | 无（集成到主应用） | [] | [] | 无 | 空实现，占位用 |

证据：
- `app/startup/BUILD.gn:14-18` startup target 定义

### 模块: demolink

**文件**: `app/demolink/BUILD.gn`

| Target 名称 | 类型 | 输出文件 | sources | include_dirs | deps | 备注 |
|-------------|------|---------|---------|--------------|------|------|
| example_demolink | static_library | libexample_demolink.a | demosdk.c, demosdk_adapter.c, helloworld.c | [//commonlibrary/utils_lite/include] | 无 | Demo SDK 示例 |

证据：
- `app/demolink/BUILD.gn:14-22` example_demolink target 定义

**详细配置**:
```gn
static_library("example_demolink") {
  sources = [
    "demosdk.c",
    "demosdk_adapter.c",
    "helloworld.c",
  ]

  include_dirs = [
    "//commonlibrary/utils_lite/include"
  ]
}
```

### 模块: iothardware

**文件**: `app/iothardware/BUILD.gn`

| Target 名称 | 类型 | 输出文件 | sources | include_dirs | deps | 备注 |
|-------------|------|---------|---------|--------------|------|------|
| led_example | static_library | libled_example.a | led_example.c | [//commonlibrary/utils_lite/include,<br>//kernel/liteos_m/kal/cmsis,<br>//base/iothardware/peripheral/interfaces/inner_api] | 无 | LED 控制示例 |

证据：
- `app/iothardware/BUILD.gn:14-22` led_example target 定义

**详细配置**:
```gn
static_library("led_example") {
  sources = [ "led_example.c" ]

  include_dirs = [
    "//commonlibrary/utils_lite/include",
    "//kernel/liteos_m/kal/cmsis",
    "//base/iothardware/peripheral/interfaces/inner_api",
  ]
}
```

### 模块: samgr

**文件**: `app/samgr/BUILD.gn`

| Target 名称 | 类型 | 输出文件 | sources | include_dirs | deps | 备注 |
|-------------|------|---------|---------|--------------|------|------|
| example_samgr | static_library | libexample_samgr.a | bootstrap_example.c,<br>broadcast_example.c,<br>feature_example.c,<br>maintenance_example.c,<br>samgr_maintenance.c,<br>service_example.c,<br>service_recovery_example.c,<br>specified_task_example.c,<br>task_example.c | [//commonlibrary/utils_lite/include,<br>//kernel/liteos_m/components/cmsis,<br>//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr,<br>//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast,<br>//foundation/systemabilitymgr/samgr_lite/samgr/adapter,<br>//foundation/systemabilitymgr/samgr_lite/samgr/source,<br>//test/xts/acts/distributed_schedule_lite/samgr_hal/utils] | 无 | SAMGR_Lite 框架示例 |

证据：
- `app/samgr/BUILD.gn:14-36` example_samgr target 定义

**详细配置**:
```gn
static_library("example_samgr") {
  sources = [
    "//test/xts/acts/distributed_schedule_lite/samgr_hal/utils/samgr_maintenance.c",
    "bootstrap_example.c",
    "broadcast_example.c",
    "feature_example.c",
    "maintenance_example.c",
    "service_example.c",
    "service_recovery_example.c",
    "specified_task_example.c",
    "task_example.c",
  ]

  include_dirs = [
    "//commonlibrary/utils_lite/include",
    "//kernel/liteos_m/components/cmsis",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast",
    "//foundation/systemabilitymgr/samgr_lite/samgr/adapter",
    "//foundation/systemabilitymgr/samgr_lite/samgr/source",
    "//test/xts/acts/distributed_schedule_lite/samgr_hal/utils",
  ]
}
```

### 根模块: app

**文件**: `app/BUILD.gn`

| Target 名称 | 类型 | 输出文件 | features | deps | 备注 |
|-------------|------|---------|----------|------|------|
| app | lite_component | 应用组件 | ["startup"] | 无 | 根组件，聚合所有子模块 |

证据：
- `app/BUILD.gn:16-18` app target 定义

**详细配置**:
```gn
import("//build/lite/config/component/lite_component.gni")

lite_component("app") {
  features = [ "startup" ]
}
```

## Target 依赖关系

### 依赖树

```
app (lite_component)
  └─> startup (source_set)
         └─> (无依赖)

example_demolink (static_library)
  └─> utils_lite (外部依赖)

led_example (static_library)
  └─> utils_lite (外部依赖)
  └─> liteos_m (外部依赖)
  └─> peripheral (外部依赖)

example_samgr (static_library)
  └─> utils_lite (外部依赖)
  └─> liteos_m (外部依赖)
  └─> samgr_lite (外部依赖)
  └─> samgr_maintenance.c (外部源文件)
```

### 依赖矩阵

| Target | depends on | 证据 |
|--------|-----------|------|
| startup | 无 | app/startup/BUILD.gn:17 (empty sources) |
| example_demolink | utils_lite | app/demolink/BUILD.gn:21 |
| led_example | utils_lite, liteos_m, peripheral | app/iothardware/BUILD.gn:18-20 |
| example_samgr | utils_lite, liteos_m, samgr_lite, samgr_maintenance.c | app/samgr/BUILD.gn:16, 28-34 |
| app | startup | app/BUILD.gn:17 |

**注意**: BUILD.gn 中无 `deps` 字段，依赖仅通过 `include_dirs` 体现。这表明模块间无直接编译依赖，只有头文件依赖。

## 外部依赖组件

根据 `bundle.json:23-30` 和 BUILD.gn 的 `include_dirs`：

| 组件名称 | 路径 | 说明 | 依赖的 Target |
|---------|------|------|--------------|
| utils_lite | //commonlibrary/utils_lite/include | 轻量级工具库 | example_demolink, led_example, example_samgr |
| liteos_m | //kernel/liteos_m/kal/cmsis 或 //kernel/liteos_m/components/cmsis | LiteOS-M 内核 | led_example, example_samgr |
| peripheral | //base/iothardware/peripheral/interfaces/inner_api | 外设驱动 | led_example |
| samgr_lite | //foundation/systemabilitymgr/samgr_lite/... | SAMGR_Lite 服务框架 | example_samgr |
| samgr_maintenance | //test/xts/acts/distributed_schedule_lite/samgr_hal/utils | 维护工具 | example_samgr (source 依赖) |

证据：
- `bundle.json:23-30` 组件依赖声明
- 各 BUILD.gn 文件的 `include_dirs` 字段

## 编译选项与宏定义

### 无显式 defines

检查所有 BUILD.gn 文件，未发现 `defines` 字段。

证据：
- `app/startup/BUILD.gn`: 无 defines
- `app/demolink/BUILD.gn`: 无 defines
- `app/iothardware/BUILD.gn`: 无 defines
- `app/samgr/BUILD.gn`: 无 defines

### 无显式 configs

检查所有 BUILD.gn 文件，未发现 `configs` 字段。

证据：
- 所有 BUILD.gn 文件均无 configs 字段

### 代码中定义的宏

| 宏名 | 位置 | 值 | 说明 |
|------|------|-----|------|
| LED_TEST_GPIO | app/iothardware/led_example.c:25 | 9 | LED GPIO 引脚号 |
| LED_INTERVAL_TIME_US | app/iothardware/led_example.c:22 | 300000 | LED 间隔时间（微秒） |
| LED_TASK_STACK_SIZE | app/iothardware/led_example.c:23 | 512 | LED 任务栈大小 |
| LED_TASK_PRIO | app/iothardware/led_example.c:24 | 25 | LED 任务优先级 |
| EXAMPLE_SERVICE | app/samgr/example.h:18 | "example" | 示例服务名称 |
| EXAMPLE_FEATURE | app/samgr/example.h:19 | "example" | 示例特性名称 |
| BROADCAST_TEST_SERVICE | app/samgr/broadcast_example.c:27 | "broadcast test" | 广播测试服务名称 |
| TEST_LEN | app/samgr/broadcast_example.c:25 | 10 | 测试数据长度 |
| WAIT_FEATURE_PROC | app/samgr/feature_example.c:29 | 1000 | 等待特性处理时间（毫秒） |
| WAIT_PUB_PROC | app/samgr/broadcast_example.c:26 | 1000 | 等待发布处理时间（毫秒） |
| TASK_STACK_SIZE | app/demolink/demosdk.c:21 | 1000 | Demo SDK 任务栈大小 |
| TASK_PRIO | app/demolink/demosdk.c:22 | 20 | Demo SDK 任务优先级 |
| SECOND_CNT | app/demolink/demosdk.c:23 | 1000 | 秒计数（毫秒） |

证据：
- 各源文件中的 `#define` 语句

## Target ↔ 产物映射

### 静态库产物

| Target | 产物文件 | 编译目标平台 | 备注 |
|--------|---------|-------------|------|
| example_demolink | libexample_demolink.a | HiSpark Pegasus（示例） | 静态库 |
| led_example | libled_example.a | HiSpark Pegasus（示例） | 静态库 |
| example_samgr | libexample_samgr.a | HiSpark Pegasus（示例） | 静态库 |

**注意**: 产物文件名由 GN 自动生成，格式为 `lib<target_name>.a`。

### 最终可执行文件

本项目是一个示例应用，不直接生成可执行文件。静态库会被链接到最终的系统镜像中。

**可能的最终产物**:
- 系统镜像文件（.bin, .elf 等）
- 固件包（用于刷写到硬件）

## GN 构建脚本分析

### lite_component

`lite_component` 是 OpenHarmony 提供的 GN 模板，用于定义轻量级组件。

证据：
- `app/BUILD.gn:14` `import("//build/lite/config/component/lite_component.gni")`
- `app/BUILD.gn:16-18` `lite_component("app")`

### source_set vs static_library

| 类型 | 说明 | 使用场景 |
|------|------|---------|
| source_set | 源文件集合，不生成独立库 | 当源文件需要直接链接到最终可执行文件时 |
| static_library | 生成静态库（.a 文件） | 当源文件需要编译为独立库时 |

本项目使用：
- `startup`: source_set（空实现，直接集成）
- 其他模块: static_library（生成独立静态库）

## 构建命令示例

### 编译整个应用

```bash
hb build -f -T @ohos/wifi_iot_sample_app
```

### 编译特定 target

```bash
hb build -f -T app/demolink:example_demolink
hb build -f -T app/iothardware:led_example
hb build -f -T app/samgr:example_samgr
```

### 清理构建产物

```bash
hb clean
```

## 相关跳转链接

- [项目概览](01_Project_Overview.md)
- [目录结构与模块职责](02_Directory_Structure.md)
- [编译产物](07_Build_Artifacts.md)
- [常见问题与调试](09_QA_Troubleshooting.md)
