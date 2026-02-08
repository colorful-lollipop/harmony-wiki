# 配置宏

## 目的

本文档描述 HiView Lite 的关键编译宏和 feature flags，帮助理解可配置项。

## 适用范围

本文档适用于：
- 需要修改编译配置的工程师
- 需要启用/禁用功能的维护者
- 需要了解配置项的开发者

## 相关跳转

- [GN Targets](06_GN_Targets.md) - 了解构建配置
- [项目概览](00_Overview.md) - 了解项目定位
- [常见问题](09_FAQ.md) - 了解配置问题

---

## 编译宏（来自 BUILD.gn）

### 功能开关宏

| 宏名 | GN 参数 | 默认值 | 说明 |
|------|----------|---------|------|
| `OUTPUT_OPTION` | hiview_lite_output_option | 1 | 输出模式（0:DEBUG, 1:FLOW, 2:TEXT_FILE, 3:BIN_FILE） |
| `HILOG_LITE_SWITCH` | hiview_lite_hilog_lite_log_switch | 1 | Hilog 日志开关（0:关闭, 1:开启） |
| `DUMP_LITE_SWITCH` | hiview_lite_dump_lite_dump_switch | 0 | Dump 功能开关（0:关闭, 1:开启） |
| `HIEVENT_LITE_SWITCH` | hiview_lite_hievent_lite_event_switch | 1 | HiEvent 事件开关（0:关闭, 1:开启） |
| `LOG_OUTPUT_MODULE` | hiview_lite_output_module | -1 | 输出模块（-1:所有模块） |

**证据来源**：
- BUILD.gn:49-56 - defines 生成
- BUILD.gn:15-20 - GN args 定义

### 日志级别宏

| 宏名 | GN 参数 | 默认值 | 构建类型 |
|------|----------|---------|----------|
| `OUTPUT_LEVEL` | hiview_lite_hilog_lite_level | 1 | debug 模式 |
| `OUTPUT_LEVEL` | hiview_lite_hilog_lite_level_release | 3 | release 模式 |

**说明**：根据 `ohos_build_type` 自动选择：
- debug 模式：使用 `hiview_lite_hilog_lite_level`
- release 模式：使用 `hiview_lite_hilog_lite_level_release`

**证据来源**：
- BUILD.gn:57-61 - 条件编译
- BUILD.gn:16-17 - GN args 定义

### 栈配置宏

| 宏名 | GN 参数 | 默认值 | 单位 |
|------|----------|---------|------|
| `HIVIEW_STACK_SIZE` | hiview_lite_stack_size | 4096 | 字节 |
| `HIVIEW_STACK_PRIO` | hiview_lite_stack_prio | 24 | 优先级 |

**说明**：
- `HIVIEW_STACK_SIZE`：HiView 服务线程的栈大小
- `HIVIEW_STACK_PRIO`：HiView 服务线程的优先级

**证据来源**：
- BUILD.gn:54-55 - defines 生成
- BUILD.gn:23-24 - GN args 定义

### 文件目录宏

| 宏名 | GN 参数 | 默认值 | 说明 |
|------|----------|---------|------|
| `HIVIEW_FILE_DIR` | hiview_lite_dir | "" | HiView 文件目录 |

**影响**：
- `HIVIEW_FILE_OUT_PATH_LOG` = `HIVIEW_FILE_DIR"debug.log"`
- `HIVIEW_FILE_OUT_PATH_UE_EVENT` = `HIVIEW_FILE_DIR"ue.event"`
- `HIVIEW_FILE_OUT_PATH_FAULT_EVENT` = `HIVIEW_FILE_DIR"fault.event"`
- `HIVIEW_FILE_OUT_PATH_STAT_EVENT` = `HIVIEW_FILE_DIR"stat.event"`
- `HIVIEW_FILE_PATH_DUMP` = `HIVIEW_FILE_DIR"dump.dat"`

**证据来源**：
- BUILD.gn:30 - define 生成
- BUILD.gn:22 - GN args 定义
- hiview_config.h:34-44 - 宏定义

### 禁用 CORE_INIT 宏

| 宏名 | GN 参数 | 默认值 | 说明 |
|------|----------|---------|------|
| `DISABLE_HIVIEW_LITE_CORE_INIT` | hiview_lite_disable_core_init | 未定义 | 禁用 CORE_INIT（如果为 true，定义此宏） |

**说明**：
- 如果 `hiview_lite_disable_core_init = false`：不定义此宏，启用 CORE_INIT
- 如果 `hiview_lite_disable_core_init = true`：定义此宏，禁用 CORE_INIT

**使用场景**：
- 某些平台可能需要手动初始化配置
- 集成到已有系统时可能需要禁用自动初始化

**证据来源**：
- BUILD.gn:62-64 - 条件宏定义
- BUILD.gn:26 - GN args 定义
- hiview_config.c:36-38 - 宏使用

---

## 头文件宏（来自 hiview_config.h）

### 文件路径宏

| 宏名 | 值 | 说明 |
|------|-----|------|
| `HIVIEW_FILE_OUT_PATH_LOG` | `HIVIEW_FILE_DIR"debug.log"` | 日志输出路径（正式） |
| `HIVIEW_FILE_OUT_PATH_UE_EVENT` | `HIVIEW_FILE_DIR"ue.event"` | UE 事件输出路径（正式） |
| `HIVIEW_FILE_OUT_PATH_FAULT_EVENT` | `HIVIEW_FILE_DIR"fault.event"` | 故障事件输出路径（正式） |
| `HIVIEW_FILE_OUT_PATH_STAT_EVENT` | `HIVIEW_FILE_DIR"stat.event"` | 统计事件输出路径（正式） |
| `HIVIEW_FILE_PATH_LOG` | `HIVIEW_FILE_OUT_PATH_LOG".tmp"` | 日志路径（临时） |
| `HIVIEW_FILE_PATH_UE_EVENT` | `HIVIEW_FILE_OUT_PATH_UE_EVENT".tmp"` | UE 事件路径（临时） |
| `HIVIEW_FILE_PATH_FAULT_EVENT` | `HIVIEW_FILE_OUT_PATH_FAULT_EVENT".tmp"` | 故障事件路径（临时） |
| `HIVIEW_FILE_PATH_STAT_EVENT` | `HIVIEW_FILE_OUT_PATH_STAT_EVENT".tmp"` | 统计事件路径（临时） |
| `HIVIEW_FILE_PATH_DUMP` | `HIVIEW_FILE_DIR"dump.dat"` | Dump 文件路径 |

**证据来源**：
- hiview_config.h:34-44 - 宏定义

### 缓存大小宏

| 宏名 | 默认值 | 说明 |
|------|---------|------|
| `LOG_STATIC_CACHE_SIZE` | 1024 | 日志静态缓存大小（字节） |
| `EVENT_CACHE_SIZE` | 256 | 事件缓存大小（字节） |
| `JS_LOG_CACHE_SIZE` | 512 | JS 日志缓存大小（字节） |
| `HIVIEW_HILOG_FILE_BUF_SIZE` | 512 | Hilog 文件缓冲区大小（字节） |
| `HIVIEW_HIEVENT_FILE_BUF_SIZE` | 128 | HiEvent 文件缓冲区大小（字节） |

**说明**：
- `LOG_STATIC_CACHE_SIZE` 必须大于 `HIVIEW_HILOG_FILE_BUF_SIZE`
- `EVENT_CACHE_SIZE` 必须大于 `HIVIEW_HIEVENT_FILE_BUF_SIZE`

**证据来源**：
- hiview_config.h:47-61 - 宏定义

### RAM Dump 配置宏

| 宏名 | 值 | 说明 |
|------|-----|------|
| `HIVIEW_DUMP_PRE_SIZE` | 384 * 1024 | Dump 预留大小（384 KB） |
| `HIVIEW_DUMP_HEADER_OFFSET` | 0x400 | Dump 文件头偏移（1 KB） |
| `HIVIEW_DUMP_RAM_ADDR` | 0x10000400 | Dump RAM 地址（预留 0x10000000 ~ 0x100003FF 给 NVIC） |
| `HIVIEW_DUMP_RAM_SIZE` | 384 * 1024 - 0x400 | Dump RAM 大小（383 KB） |

**证据来源**：
- hiview_config.h:64-67 - 宏定义

### 版本和常量宏

| 宏名 | 值 | 说明 |
|------|-----|------|
| `HIVIEW_FILE_HEADER_MAIN_VERSION` | 1 | 文件头主版本号 |
| `HIVIEW_FILE_HEADER_SUB_VERSION` | 10 | 文件头子版本号（lite） |
| `HIVIEW_UE_EVENT_VER` | 991231100 | UE 事件版本 |
| `HIVIEW_FAULT_EVENT_VER` | 991231000 | 故障事件版本 |
| `HIVIEW_STATIC_EVENT_VER` | 991231000 | 静态事件版本 |
| `HIVIEW_CONF_PRODUCT_VER_STR` | "1.0.0" | 产品版本字符串 |

**证据来源**：
- hiview_config.h:32-37 - 宏定义

### 超时和等待宏

| 宏名 | 值 | 说明 |
|------|-----|------|
| `OUT_PATH_WAIT_TIMEOUT` | 5 * 1000 | 文件路径等待超时（5 秒，单位：毫秒） |

**证据来源**：
- hiview_config.h:73 - 宏定义

---

## 宏依赖关系

### 日志输出配置

```
OUTPUT_OPTION
    │
    ├─> 输出模式（UART/FLOW/FILE）
    │
    └─> 影响日志和事件的输出方式


OUTPUT_LEVEL
    │
    └─> 影响日志输出的详细程度（DEBUG/INFO/WARN/ERROR）


LOG_OUTPUT_MODULE
    │
    └─> 控制哪些模块的日志被输出
```

### 功能开关配置

```
HILOG_LITE_SWITCH
    │
    └─> 控制 Hilog 组件是否启用


DUMP_LITE_SWITCH
    │
    └─> 控制 Dump 组件是否启用


HIEVENT_LITE_SWITCH
    │
    └─> 控制 HiEvent 组件是否启用
```

---

## 配置场景

### 场景 1：开发调试模式

**配置**：
```bash
hiview_lite_output_option = 0  # 直接输出到 UART（禁止商业版）
hiview_lite_hilog_lite_level = 1  # DEBUG 级别
hiview_lite_hilog_lite_log_switch = 1  # 启用日志
hiview_lite_dir = "/tmp/log/"  # 使用临时目录
```

**说明**：开发调试时，直接输出到 UART，使用 DEBUG 级别，便于调试。

### 场景 2：生产发布模式

**配置**：
```bash
hiview_lite_output_option = 2  # 输出到文本文件
hiview_lite_hilog_lite_level_release = 3  # ERROR 级别
hiview_lite_hilog_lite_log_switch = 1  # 启用日志
hiview_lite_dir = "/data/log/"  # 使用正式目录
```

**说明**：生产发布时，输出到文件，使用 ERROR 级别，减少性能影响。

### 场景 3：最小化配置

**配置**：
```bash
hiview_lite_hilog_lite_log_switch = 0  # 禁用日志
hiview_lite_dump_lite_dump_switch = 0  # 禁用 Dump
hiview_lite_hievent_lite_event_switch = 0  # 禁用事件
```

**说明**：资源受限时，可以禁用某些功能以节省资源。

### 场景 4：自定义实现

**配置**：
```bash
hiview_lite_customize_implementation = true  # 使用自定义实现
```

**说明**：只导出配置，不构建静态库，由外部提供实现。

---

## 宏影响分析

### 对 ROM 占用的影响

| 功能 | ROM 占用 | 说明 |
|------|----------|------|
| 基础服务 | ~5KB | 必需 |
| 日志功能 | ~2KB | 可通过 `HILOG_LITE_SWITCH` 禁用 |
| Dump 功能 | ~1KB | 可通过 `DUMP_LITE_SWITCH` 禁用 |
| 事件功能 | ~1KB | 可通过 `HIEVENT_LITE_SWITCH` 禁用 |
| 缓存功能 | ~1KB | 必需 |
| **总计** | **~10KB** | 与 bundle.json:33 一致 |

### 对 RAM 占用的影响

| 资源 | RAM 占用 | 说明 |
|------|----------|------|
| 全局变量 | ~2KB | 必需 |
| 缓存（LOG + EVENT） | ~4KB | 可通过禁用功能减少 |
| 服务栈 | 4KB | 可通过 `HIVIEW_STACK_SIZE` 调整 |
| 缓存文件缓冲区 | ~0.5KB | 必需 |
| **总计** | **~10.5KB** | 与 bundle.json:34 一致 |

---

## 关键结论

1. **灵活配置** - 提供多个 GN args，支持不同场景。
2. **功能开关** - 可以独立禁用日志、Dump、事件功能。
3. **资源可调** - 可以调整栈大小、缓存大小等。
4. **路径可配** - 文件目录可以通过 `hiview_lite_dir` 配置。
5. **日志级别可选** - debug 和 release 模式可以使用不同的日志级别。

---

*最后更新：2026-02-06*
