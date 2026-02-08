# 构建与产物

> **文档版本**: 1.0  
> **最后更新**: 2026-02-07  
> **代码版本**: v3.1

## 7.1 GN 目标清单

### 7.1.1 目标概览

| 目标名称 | 类型 | 依赖 | 产物 |
|----------|------|------|------|
| `hiview_lite_config` | config | 无 | 配置定义 |
| `hiview_lite_static` | static_library | hiview_lite_config | `libhiview_lite_static.a` |
| `hiview_lite` | group | hiview_lite_static 或自定义 | 组目标 |

**证据来源**：`BUILD.gn:29-80`

---

### 7.1.2 配置目标

**目标名称**：`hiview_lite_config`

**类型**：config

**定义**：`BUILD.gn:29-38`

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
- 定义编译宏 `HIVIEW_FILE_DIR`
- 指定头文件搜索路径

---

### 7.1.3 静态库目标

**目标名称**：`hiview_lite_static`

**类型**：static_library

**源文件**：`BUILD.gn:41-47`

```gn
sources = [
  "hiview_cache.c",
  "hiview_config.c",
  "hiview_file.c",
  "hiview_service.c",
  "hiview_util.c",
]
```

**编译宏定义**：`BUILD.gn:48-56`

```gn
defines = [
  "OUTPUT_OPTION=$hiview_lite_output_option",
  "HILOG_LITE_SWITCH=$hiview_lite_hilog_lite_log_switch",
  "DUMP_LITE_SWITCH=$hiview_lite_dump_lite_dump_switch",
  "HIEVENT_LITE_SWITCH=$hiview_lite_hievent_lite_event_switch",
  "LOG_OUTPUT_MODULE=$hiview_lite_output_module",
  "HIVIEW_STACK_SIZE=$hiview_lite_stack_size",
  "HIVIEW_STACK_PRIO=$hiview_lite_stack_prio",
]
```

**条件编译**：`BUILD.gn:57-64`

```gn
if (ohos_build_type == "debug") {
  defines += [ "OUTPUT_LEVEL=$hiview_lite_hilog_lite_level" ]
} else {
  defines += [ "OUTPUT_LEVEL=$hiview_lite_hilog_lite_level_release" ]
}

if (hiview_lite_disable_core_init) {
  defines += [ "DISABLE_HIVIEW_LITE_CORE_INIT" ]
}
```

**平台特定配置**：`BUILD.gn:65-70`

```gn
if (board_toolchain_type == "iccarm") {
  cflags = [
    "--diag_suppress",
    "Pe546",
  ]
}
```

---

### 7.1.4 组目标

**目标名称**：`hiview_lite`

**类型**：group

**定义**：`BUILD.gn:74-80`

```gn
group("hiview_lite") {
  if (hiview_lite_customize_implementation) {
    public_configs = [ ":hiview_lite_config" ]
  } else {
    public_deps = [ ":hiview_lite_static" ]
  }
}
```

**作用**：
- 当 `hiview_lite_customize_implementation` 为 true 时，只暴露配置
- 当为 false 时，依赖静态库

---

## 7.2 编译产物

### 7.2.1 静态库

| 产物 | 格式 | 大小 | 说明 |
|------|------|------|------|
| `libhiview_lite_static.a` | 静态库 | ~10KB | 编译后的目标文件集合 |

**产物位置**：
```
out/hi3516dv300/libs/
```

### 7.2.2 符号表

| 符号类型 | 数量 | 说明 |
|----------|------|------|
| 公开函数 | ~36 | 对外 API |
| 内部函数 | ~17 | 内部使用 |
| 全局变量 | ~10 | 配置和状态 |

**主要公开符号**：

| 符号 | 类型 | 说明 |
|------|------|------|
| `HiviewConfigInit` | 函数 | 配置初始化 |
| `HiviewRegisterInitFunc` | 函数 | 注册组件初始化 |
| `HiviewRegisterMsgHandle` | 函数 | 注册消息处理 |
| `HiviewSendMessage` | 函数 | 发送消息 |
| `InitHiviewFile` | 函数 | 初始化文件 |
| `WriteToFile` | 函数 | 写入文件 |
| `ReadFromFile` | 函数 | 读取文件 |
| `InitHiviewCache` | 函数 | 初始化缓存 |
| `WriteToCache` | 函数 | 写入缓存 |

---

## 7.3 Feature 开关

### 7.3.1 Feature 清单

| Feature | 默认值 | 类型 | 说明 |
|---------|--------|------|------|
| `hiview_lite_output_option` | 1 | int | 输出选项 |
| `hiview_lite_hilog_lite_level` | 1 | int | Debug 日志级别 |
| `hiview_lite_hilog_lite_level_release` | 3 | int | Release 日志级别 |
| `hiview_lite_hilog_lite_log_switch` | 1 | int | 日志组件开关 |
| `hiview_lite_dump_lite_dump_switch` | 0 | int | Dump 组件开关 |
| `hiview_lite_hievent_lite_event_switch` | 1 | int | 事件组件开关 |
| `hiview_lite_output_module` | -1 | int | 输出模块 |
| `hiview_lite_dir` | "" | string | 文件目录 |
| `hiview_lite_stack_size` | 4096 | int | 栈大小 |
| `hiview_lite_stack_prio` | 24 | int | 栈优先级 |
| `hiview_lite_customize_implementation` | false | bool | 自定义实现开关 |
| `hiview_lite_disable_core_init` | false | bool | 禁用 CORE_INIT |

**证据来源**：`BUILD.gn:14-27`

---

### 7.3.2 输出选项

**Feature**：`hiview_lite_output_option`

**默认值**：1

**可选值**：

| 值 | 枚举 | 说明 |
|---|------|------|
| 0 | `OUTPUT_OPTION_DEBUG` | 直接输出到 UART（商业版禁止） |
| 1 | `OUTPUT_OPTION_FLOW` | 通过 SAMGR 输出到 UART |
| 2 | `OUTPUT_OPTION_TEXT_FILE` | 输出到文本文件 |
| 3 | `OUTPUT_OPTION_BIN_FILE` | 输出到二进制文件 |
| 8 | `OUTPUT_OPTION_PRINT` | 打印输出 |

**证据来源**：`hiview_config.h:88-95`

---

### 7.3.3 日志级别

**Feature**：`hiview_lite_hilog_lite_level` (Debug) / `hiview_lite_hilog_lite_level_release` (Release)

**默认值**：Debug=1, Release=3

**日志级别定义**：

| 级别 | 值 | 说明 |
|------|-----|------|
| DEBUG | 0 | 调试信息 |
| INFO | 1 | 普通信息 |
| WARN | 2 | 警告信息 |
| ERROR | 3 | 错误信息 |
| FATAL | 4 | 致命错误 |

---

### 7.3.4 组件开关

| 组件 | Feature | 默认值 | 说明 |
|------|---------|--------|------|
| 日志 | `hiview_lite_hilog_lite_log_switch` | 1 | 启用日志组件 |
| Dump | `hiview_lite_dump_lite_dump_switch` | 0 | 启用 Dump 组件 |
| 事件 | `hiview_lite_hievent_lite_event_switch` | 1 | 启用事件组件 |

**证据来源**：`BUILD.gn:18-20`, `hiview_config.h:70-71`

```c
#define HIVIEW_FEATURE_ON                  1
#define HIVIEW_FEATURE_OFF                 0
```

---

### 7.3.5 任务配置

**栈大小**：`hiview_lite_stack_size`

- 默认值：4096 字节
- 最小值：2048 字节（建议）
- 最大值：16384 字节

**栈优先级**：`hiview_lite_stack_prio`

- 默认值：24
- 范围：0-31（LiteOS-M 优先级范围）

**证据来源**：`BUILD.gn:23-24`

---

## 7.4 构建命令

### 7.4.1 标准构建

```bash
# 进入构建目录
cd build

# Debug 构建
python build.py -p <product> -b debug

# Release 构建
python build.py -p <product> -b release
```

### 7.4.2 自定义 Feature

```bash
# 设置自定义 Feature
python build.py -p <product> -b debug \
  --gn-args hiview_lite_output_option=2 \
            hiview_lite_dump_lite_dump_switch=1 \
            hiview_lite_dir="/data/log/"
```

### 7.4.3 禁用 CORE_INIT

```bash
# 禁用 CORE_INIT 阶段
python build.py -p <product> -b debug \
  --gn-args hiview_lite_disable_core_init=true
```

---

## 7.5 依赖关系

### 7.5.1 系统依赖

| 依赖 | 说明 | 来源 |
|------|------|------|
| `liteos_m` | LiteOS-M 内核接口 | 系统组件 |
| `samgr_lite` | 轻量级服务管理器 | `include_dirs` |
| `hilog_lite` | 日志组件接口 | `include_dirs` |
| `hievent_lite` | 事件组件接口 | `include_dirs` |
| `utils_lite` | 公共工具库 | `include_dirs` |

### 7.5.2 第三方依赖

| 依赖 | 说明 | 用途 |
|------|------|------|
| `bounds_checking_function` | 边界检查函数库 | 安全内存操作 |

**证据来源**：`bundle.json:39-41`

```json
"third_party": [
  "bounds_checking_function"
]
```

---

## 7.6 编译配置示例

### 7.6.1 最小配置

```gn
# 只包含 hiview_lite
hiview_lite_output_option = 1
hiview_lite_hilog_lite_log_switch = 1
hiview_lite_hievent_lite_event_switch = 1
hiview_lite_dump_lite_dump_switch = 0
```

### 7.6.2 日志优先配置

```gn
# 最大化日志输出
hiview_lite_output_option = 1
hiview_lite_hilog_lite_level = 0  # DEBUG 级别
hiview_lite_hilog_lite_log_switch = 1
hiview_lite_hievent_lite_event_switch = 1
hiview_lite_dump_lite_dump_switch = 1  # 启用 dump
```

### 7.6.3 资源受限配置

```gn
# 最小化资源占用
hiview_lite_output_option = 2  # 文件输出
hiview_lite_hilog_lite_level = 3  # ERROR 级别
hiview_lite_hilog_lite_log_switch = 1
hiview_lite_hievent_lite_event_switch = 0  # 禁用事件
hiview_lite_dump_lite_dump_switch = 0  # 禁用 dump
hiview_lite_stack_size = 2048  # 减小栈
hiview_lite_stack_prio = 28  # 降低优先级
```

---

## 7.7 下一步

| 你的目标 | 推荐阅读 |
|----------|----------|
| 了解项目定位 | [01_Overview.md](./01_Overview.md) |
| 了解架构设计 | [02_Architecture.md](./02_Architecture.md) |
| 了解安全风险 | [06_SecurityReview.md](./06_SecurityReview.md) |
| 了解代码结构 | [03_CodeMap.md](./03_CodeMap.md) |

---

*所有技术结论均有代码证据支撑，详见 [wiki/_work/NOTES.md](../_work/NOTES.md)。*
