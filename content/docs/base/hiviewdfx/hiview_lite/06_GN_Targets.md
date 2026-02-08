# GN Targets

## 目的

本文档描述 HiView Lite 的 GN 构建目标，包括 targets 列表、类型、依赖、产物和编译开关。

## 适用范围

本文档适用于：
- 需要理解构建配置的工程师
- 需要修改编译选项的维护者
- 需要集成到其他项目的集成方

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目定位
- [编译产物](07_Build_Artifacts.md) - 了解编译输出
- [常见问题](09_FAQ.md) - 了解构建问题

---

## 关键构建文件

### BUILD.gn

**路径**：`/Volumes/lexar/code/d/work/oh/base/hiviewdfx/hiview_lite/BUILD.gn`

**行数**：81 行

**说明**：HiView Lite 的主构建文件，定义了编译参数和构建目标。

**证据来源**：
- BUILD.gn:14-27 - `declare_args()` 定义构建参数
- BUILD.gn:29-38 - `config()` 定义配置
- BUILD.gn:40-72 - `static_library()` 定义静态库
- BUILD.gn:74-80 - `group()` 定义组

---

## Build Args（构建参数）

### 参数列表

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|---------|------|
| `hiview_lite_output_option` | int | 1 | 输出选项（0:DEBUG, 1:FLOW, 2:TEXT_FILE, 3:BIN_FILE） |
| `hiview_lite_hilog_lite_level` | int | 1 | Hilog 日志级别（debug 模式） |
| `hiview_lite_hilog_lite_level_release` | int | 3 | Hilog 日志级别（release 模式） |
| `hiview_lite_hilog_lite_log_switch` | int | 1 | Hilog 日志开关（0:关闭, 1:开启） |
| `hiview_lite_dump_lite_dump_switch` | int | 0 | Dump 功能开关（0:关闭, 1:开启） |
| `hiview_lite_hievent_lite_event_switch` | int | 1 | HiEvent 事件开关（0:关闭, 1:开启） |
| `hiview_lite_output_module` | int | -1 | 输出模块（-1:所有模块） |
| `hiview_lite_dir` | string | "" | HiView 文件目录 |
| `hiview_lite_stack_size` | int | 4096 | 栈大小（字节） |
| `hiview_lite_stack_prio` | int | 24 | 栈优先级 |
| `hiview_lite_customize_implementation` | bool | false | 是否使用自定义实现 |
| `hiview_lite_disable_core_init` | bool | false | 是否禁用 CORE_INIT |

**证据来源**：
- BUILD.gn:14-27 - `declare_args()` 定义

### 参数使用

#### hiview_lite_output_option

**说明**：控制日志输出模式

| 值 | 宏定义 | 输出模式 |
|-----|----------|----------|
| 0 | OUTPUT_OPTION_DEBUG | 直接输出到 UART（商业版禁止） |
| 1 | OUTPUT_OPTION_FLOW | 通过 SAMGR 输出到 UART |
| 2 | OUTPUT_OPTION_TEXT_FILE | 输出到文本文件 |
| 3 | OUTPUT_OPTION_BIN_FILE | 输出到二进制文件 |

**证据来源**：
- BUILD.gn:15 - `hiview_lite_output_option = 1`
- hiview_config.h:49 - `OUTPUT_OPTION = OUTPUT_OPTION`

#### hiview_lite_hilog_lite_level / hiview_lite_hilog_lite_level_release

**说明**：控制日志输出级别

| 构建类型 | 使用的参数 | 默认值 |
|----------|------------|---------|
| debug | `hiview_lite_hilog_lite_level` | 1（HILOG_LV_DEBUG） |
| release | `hiview_lite_hilog_lite_level_release` | 3（HILOG_LV_ERROR） |

**证据来源**：
- BUILD.gn:16-17 - 日志级别参数
- BUILD.gn:57-61 - 根据 `ohos_build_type` 选择级别

#### hiview_lite_dir

**说明**：HiView 文件目录，用于存储日志和事件文件

**默认值**：空字符串 `""`

**影响**：
- `HIVIEW_FILE_DIR` 宏
- 日志文件路径：`HIVIEW_FILE_DIR"debug.log"`
- 事件文件路径：`HIVIEW_FILE_DIR"*.event"`
- Dump 文件路径：`HIVIEW_FILE_DIR"dump.dat"`

**证据来源**：
- BUILD.gn:22 - `hiview_lite_dir = ""`
- BUILD.gn:30 - `defines = [ "HIVIEW_FILE_DIR=\"$hiview_lite_dir\"" ]`
- hiview_config.h:34-44 - 文件路径定义

#### hiview_lite_stack_size / hiview_lite_stack_prio

**说明**：控制 HiView 服务的线程栈大小和优先级

| 参数 | 默认值 | 宏定义 | 作用 |
|------|---------|----------|------|
| `hiview_lite_stack_size` | 4096 | HIVIEW_STACK_SIZE | 栈大小（字节） |
| `hiview_lite_stack_prio` | 24 | HIVIEW_STACK_PRIO | 栈优先级 |

**证据来源**：
- BUILD.gn:23-24 - 栈大小和优先级
- BUILD.gn:54-55 - `defines += [ "HIVIEW_STACK_SIZE=$hiview_lite_stack_size", "HIVIEW_STACK_PRIO=$hiview_lite_stack_prio" ]`
- hiview_service.c:91 - `TaskConfig config = { LEVEL_LOW, HIVIEW_STACK_PRIO, HIVIEW_STACK_SIZE, 10, SINGLE_TASK }`

#### hiview_lite_customize_implementation

**说明**：是否使用自定义实现

| 值 | 行为 |
|-----|------|
| false | 使用默认实现，构建 `hiview_lite_static` 静态库 |
| true | 使用自定义实现，只导出配置，不构建静态库 |

**证据来源**：
- BUILD.gn:25 - `hiview_lite_customize_implementation = false`
- BUILD.gn:75-79 - 根据 `hiview_lite_customize_implementation` 决定行为

#### hiview_lite_disable_core_init

**说明**：是否禁用 CORE_INIT

| 值 | 行为 |
|-----|------|
| false | 启用 CORE_INIT，调用 `HiviewConfigInit` |
| true | 禁用 CORE_INIT，不调用 `HiviewConfigInit` |

**使用场景**：
- 某些平台可能需要手动初始化配置
- 集成到已有系统中时可能需要禁用自动初始化

**证据来源**：
- BUILD.gn:26 - `hiview_lite_disable_core_init = false`
- BUILD.gn:62-64 - `if (hiview_lite_disable_core_init) { defines += [ "DISABLE_HIVIEW_LITE_CORE_INIT" ] }`
- hiview_config.c:36-38 - `#ifndef DISABLE_HIVIEW_LITE_CORE_INIT` / `CORE_INIT_PRI(HiviewConfigInit, 0)`

---

## Targets 清单

### hiview_lite_config

**类型**：`config`

**说明**：配置 target，定义 include_dirs 和 defines

**内容**：

```gn
config("hiview_lite_config") {
  defines = [ "HIVIEW_FILE_DIR=\"$hiview_lite_dir\"" ]
  include_dirs = [
    "//base/hiviewdfx/hiview_lite",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//base/hiviewdfx/hievent_lite/interfaces/native/innerkits",
    "//commonlibrary/utils_lite/include",
  ]
}
```

**作用**：
- 定义 `HIVIEW_FILE_DIR` 宏
- 定义 include 路径，供其他模块使用

**证据来源**：
- BUILD.gn:29-38 - `config("hiview_lite_config")`

---

### hiview_lite_static

**类型**：`static_library`

**说明**：静态库 target，包含所有源文件

**sources**：

| 源文件 | 行数 | 说明 |
|---------|------|------|
| hiview_cache.c | 223 | 缓存实现 |
| hiview_config.c | 39 | 配置实现 |
| hiview_file.c | 341 | 文件操作实现 |
| hiview_service.c | 140 | 服务实现 |
| hiview_util.c | 313 | 工具函数实现 |

**defines**：

| 宏 | 值来源 | 说明 |
|-----|----------|------|
| OUTPUT_OPTION | $hiview_lite_output_option | 输出选项 |
| HILOG_LITE_SWITCH | $hiview_lite_hilog_lite_log_switch | Hilog 日志开关 |
| DUMP_LITE_SWITCH | $hiview_lite_dump_lite_dump_switch | Dump 开关 |
| HIEVENT_LITE_SWITCH | $hiview_lite_hievent_lite_event_switch | HiEvent 开关 |
| LOG_OUTPUT_MODULE | $hiview_lite_output_module | 输出模块 |
| HIVIEW_STACK_SIZE | $hiview_lite_stack_size | 栈大小 |
| HIVIEW_STACK_PRIO | $hiview_lite_stack_prio | 栈优先级 |
| OUTPUT_LEVEL | $hiview_lite_hilog_lite_level 或 $hiview_lite_hilog_lite_level_release | 输出级别（根据构建类型） |
| DISABLE_HIVIEW_LITE_CORE_INIT | 条件定义 | 禁用 CORE_INIT（如果 hiview_lite_disable_core_init=true） |

**public_configs**：

- `:hiview_lite_config` - 导出配置，供依赖方使用

**cflags**（条件编译）：

- `#if (board_toolchain_type == "iccarm")` - IAR 编译器特殊标志

**证据来源**：
- BUILD.gn:40-72 - `static_library("hiview_lite_static")`

---

### hiview_lite

**类型**：`group`

**说明**：组 target，根据 `hiview_lite_customize_implementation` 决定行为

**行为**：

| hiview_lite_customize_implementation | 行为 |
|--------------------------------|------|
| false | `public_deps = [ ":hiview_lite_static" ]` - 依赖静态库 |
| true | `public_configs = [ ":hiview_lite_config" ]` - 只导出配置 |

**证据来源**：
- BUILD.gn:74-80 - `group("hiview_lite")`

---

## Include Dirs

### 核心头文件

| 路径 | 说明 |
|--------|------|
| //base/hiviewdfx/hiview_lite | 本项目头文件 |
| //foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr | SAMGR Lite 接口 |
| //base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite | Hilog Lite 接口 |
| //base/hiviewdfx/hievent_lite/interfaces/native/innerkits | HiEvent Lite 接口 |
| //commonlibrary/utils_lite/include | Utils Lite 公共头文件 |

**证据来源**：
- BUILD.gn:31-37 - `include_dirs`

---

## 依赖关系

### 模块依赖

| 目标 | 依赖类型 | 依赖 | 说明 |
|------|----------|------|------|
| hiview_lite_static | 外部依赖 | liteos_m | RTOS 内核 |
| hiview_lite_static | 第三方依赖 | bounds_checking_function | 边界检查函数库 |
| hiview_lite_static | 公共库 | utils_lite | 工具库 |

**证据来源**：
- bundle.json:36-42 - 依赖定义
- BUILD.gn:36 - include_dirs

### Target 依赖

| 目标 | public_deps | 说明 |
|------|-------------|------|
| hiview_lite | :hiview_lite_static | 依赖静态库（当 hiview_lite_customize_implementation=false） |

**证据来源**：
- BUILD.gn:78 - `public_deps = [ ":hiview_lite_static" ]`

---

## 编译开关

### 功能开关

| 开关 | 宏定义 | 控制内容 |
|------|----------|----------|
| 日志开关 | HILOG_LITE_SWITCH | 是否启用日志组件 |
| Dump 开关 | DUMP_LITE_SWITCH | 是否启用 Dump 组件 |
| 事件开关 | HIEVENT_LITE_SWITCH | 是否启用事件组件 |
| CORE_INIT | DISABLE_HIVIEW_LITE_CORE_INIT | 是否禁用 CORE_INIT |

**证据来源**：
- BUILD.gn:50-53 - `defines += [ "HILOG_LITE_SWITCH=$hiview_lite_hilog_lite_log_switch", "DUMP_LITE_SWITCH=$hiview_lite_dump_lite_dump_switch", "HIEVENT_LITE_SWITCH=$hiview_lite_hievent_lite_event_switch" ]`
- BUILD.gn:62-64 - `if (hiview_lite_disable_core_init) { defines += [ "DISABLE_HIVIEW_LITE_CORE_INIT" ] }`

### 输出级别

| 构建类型 | 宏定义 | 默认值 |
|----------|----------|---------|
| debug | OUTPUT_LEVEL | 1（HILOG_LV_DEBUG） |
| release | OUTPUT_LEVEL | 3（HILOG_LV_ERROR） |

**证据来源**：
- BUILD.gn:57-61 - 根据 `ohos_build_type` 选择级别

---

## Target ↔ 产物映射

### 静态库产物

**Target**：`hiview_lite_static`

**产物**：`libhiview_lite.a`

**路径**：`out/<board>/<product>/obj/base/hiviewdfx/hiview_lite/hiview_lite_static/libhiview_lite.a`

**说明**：
- 包含所有源文件编译的 .o 文件
- 打包为静态库 .a 文件
- 可被其他模块链接

**证据来源**：
- BUILD.gn:40 - `static_library("hiview_lite_static")`
- GN 构建系统输出规范

### 组 Target 产物

**Target**：`hiview_lite`

**产物**：无直接产物（虚拟 target）

**作用**：
- 当 `hiview_lite_customize_implementation=false` 时，依赖 `hiview_lite_static`
- 当 `hiview_lite_customize_implementation=true` 时，只导出 `hiview_lite_config` 配置

**证据来源**：
- BUILD.gn:74-80 - `group("hiview_lite")`

---

## 关键结论

1. **轻量级配置** - ROM 占用 10KB，RAM 占用 ~10KB。
2. **可配置性强** - 提供多个构建参数，支持不同场景的配置。
3. **灵活初始化** - 支持禁用 CORE_INIT，便于集成到已有系统。
4. **自定义实现** - 支持 `hiview_lite_customize_implementation`，便于平台适配。
5. **静态库输出** - 默认输出静态库，便于集成。

---

*最后更新：2026-02-06*
