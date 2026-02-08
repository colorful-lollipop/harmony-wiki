# 附录 B: 配置宏与 Feature Flags

## 目的

本文档列出代码中的关键配置宏、常量定义和 Feature Flags。

## 适用范围

- 需要修改配置的开发者
- 了解配置项用途的维护者

## 配置宏清单

### iothardware 模块配置

| 宏名 | 值 | 位置 | 说明 |
|------|-----|------|------|
| LED_TEST_GPIO | 9 | app/iothardware/led_example.c:25 | LED GPIO 引脚号（HiSpark Pegasus） |
| LED_INTERVAL_TIME_US | 300000 | app/iothardware/led_example.c:22 | LED 间隔时间（微秒，即 300ms） |
| LED_TASK_STACK_SIZE | 512 | app/iothardware/led_example.c:23 | LED 任务栈大小（字节） |
| LED_TASK_PRIO | 25 | app/iothardware/led_example.c:24 | LED 任务优先级 |

**代码证据**:
```c
#define LED_INTERVAL_TIME_US 300000  // [led_example.c:22]
#define LED_TASK_STACK_SIZE 512       // [led_example.c:23]
#define LED_TASK_PRIO 25              // [led_example.c:24]
#define LED_TEST_GPIO 9               // for hispark_pegasus [led_example.c:25]
```

### demolink 模块配置

| 宏名 | 值 | 位置 | 说明 |
|------|-----|------|------|
| TASK_STACK_SIZE | 1000 | app/demolink/demosdk.c:21 | Demo SDK 任务栈大小（字节） |
| TASK_PRIO | 20 | app/demolink/demosdk.c:22 | Demo SDK 任务优先级 |
| SECOND_CNT | 1000 | app/demolink/demosdk.c:23 | 秒计数（毫秒，即 1 秒） |

**代码证据**:
```c
#define TASK_STACK_SIZE 1000  // [demosdk.c:21]
#define TASK_PRIO 20         // [demosdk.c:22]
#define SECOND_CNT 1000       // [demosdk.c:23]
```

### samgr 模块配置（服务/特性名称）

| 宏名 | 值 | 位置 | 说明 |
|------|-----|------|------|
| EXAMPLE_SERVICE | "example" | app/samgr/example.h:18 | 示例服务名称 |
| EXAMPLE_FEATURE | "example" | app/samgr/example.h:19 | 示例特性名称 |
| BOOT_SYS_SERVICE1 | "sys_s1" | app/samgr/example.h:21 | 引导系统服务 1 |
| BOOT_SYS_SERVICE2 | "sys_s2" | app/samgr/example.h:22 | 引导系统服务 2 |
| BOOT_SYS_FEATURE1 | "sys_f1" | app/samgr/example.h:23 | 引导系统特性 1 |
| BOOT_SYS_FEATURE2 | "sys_f2" | app/samgr/example.h:24 | 引导系统特性 2 |
| BOOT_SYSEX_SERVICE1 | "sysex_s1" | app/samgr/example.h:26 | 引导系统扩展服务 1 |
| BOOT_SYSEX_SERVICE2 | "sysex_s2" | app/samgr/example.h:27 | 引导系统扩展服务 2 |
| BOOT_SYSEX_FEATURE1 | "sysex_f1" | app/samgr/example.h:28 | 引导系统扩展特性 1 |
| BOOT_SYSEX_FEATURE2 | "sysex_f2" | app/samgr/example.h:29 | 引导系统扩展特性 2 |
| TASK_SERVICE1 | "task_s1" | app/samgr/example.h:31 | 任务服务 1 |
| TASK_SERVICE2 | "task_s2" | app/samgr/example.h:32 | 任务服务 2 |
| TASK_SERVICE3 | "task_s3" | app/samgr/example.h:33 | 任务服务 3 |
| TASK_SERVICE4 | "task_s4" | app/samgr/example.h:34 | 任务服务 4 |

**代码证据**:
```c
#define EXAMPLE_SERVICE "example"       // [example.h:18]
#define EXAMPLE_FEATURE "example"       // [example.h:19]
// ... (其他宏定义)
```

### samgr 模块配置（广播示例）

| 宏名 | 值 | 位置 | 说明 |
|------|-----|------|------|
| BROADCAST_TEST_SERVICE | "broadcast test" | app/samgr/broadcast_example.c:27 | 广播测试服务名称 |
| TEST_LEN | 10 | app/samgr/broadcast_example.c:25 | 测试数据长度 |
| WAIT_PUB_PROC | 1000 | app/samgr/broadcast_example.c:26 | 等待发布处理时间（毫秒） |

**代码证据**:
```c
#define TEST_LEN 10                           // [broadcast_example.c:25]
#define WAIT_PUB_PROC 1000                    // [broadcast_example.c:26]
#define BROADCAST_TEST_SERVICE "broadcast test"  // [broadcast_example.c:27]
```

### samgr 模块配置（特性示例）

| 宏名 | 值 | 位置 | 说明 |
|------|-----|------|------|
| WAIT_FEATURE_PROC | 1000 | app/samgr/feature_example.c:29 | 等待特性处理时间（毫秒） |

**代码证据**:
```c
#define WAIT_FEATURE_PROC 1000  // [feature_example.c:29]
```

### samgr 模块配置（服务/特性名称 - 维护示例）

| 宏名 | 值 | 位置 | 说明 |
|------|-----|------|------|
| MAINTEN_SERVICE1 | "mainten_s1" | app/samgr/maintenance_example.c:15 | 维护服务 1（推断） |
| MAINTEN_SERVICE2 | "mainten_s2" | app/samgr/maintenance_example.c:18 | 维护服务 2（推断） |
| MAINTEN_FEATURE1 | "mainten_f1" | app/samgr/maintenance_example.c:21 | 维护特性 1（推断） |

**代码证据**: `app/samgr/maintenance_example.c` 中的字符串常量（需要进一步确认宏定义）

## Feature Flags

本项目无显式的 Feature Flags（条件编译选项）。所有代码默认启用。

如需添加 Feature Flags，可在 `BUILD.gn` 中添加 `defines` 字段：

```gn
static_library("led_example") {
  sources = [ "led_example.c" ]
  include_dirs = [ ... ]

  defines = [
    "ENABLE_LED_BLINK=1",      # 启用 LED 闪烁
    "LED_GPIO_PIN=9",           # GPIO 引脚号
  ]
}
```

## 修改配置建议

### 1. 修改 GPIO 引脚号

根据硬件板型修改 GPIO 引脚号：

```c
// HiSpark Pegasus
#define LED_TEST_GPIO 9

// 其他板型
#define LED_TEST_GPIO 2  // 修改为实际引脚
```

**证据**: `app/iothardware/led_example.c:25`

### 2. 修改任务优先级

调整任务优先级以适应系统需求：

```c
// LED 任务优先级
#define LED_TASK_PRIO 25  // 数值越大，优先级越高

// Demo SDK 任务优先级
#define TASK_PRIO 20
```

**证据**:
- `app/iothardware/led_example.c:24`
- `app/demolink/demosdk.c:22`

### 3. 修改任务栈大小

根据任务需求调整栈大小：

```c
// LED 任务栈大小
#define LED_TASK_STACK_SIZE 512  // 字节

// Demo SDK 任务栈大小
#define TASK_STACK_SIZE 1000    // 字节
```

**证据**:
- `app/iothardware/led_example.c:23`
- `app/demolink/demosdk.c:21`

### 4. 修改时间间隔

调整 LED 闪烁间隔或 SDK 睡眠时间：

```c
// LED 间隔时间（微秒）
#define LED_INTERVAL_TIME_US 300000  // 300ms

// SDK 睡眠时间（毫秒）
#define SECOND_CNT 1000  // 1 秒
```

**证据**:
- `app/iothardware/led_example.c:22`
- `app/demolink/demosdk.c:23`

### 5. 修改服务/特性名称

根据需求自定义服务/特性名称：

```c
// 服务名称
#define EXAMPLE_SERVICE "my_custom_service"

// 特性名称
#define EXAMPLE_FEATURE "my_custom_feature"
```

**证据**: `app/samgr/example.h:18-19`

## 编译时配置

### GN 构建配置

虽然当前 `BUILD.gn` 文件中没有 `defines` 字段，但可以添加：

```gn
static_library("led_example") {
  sources = [ "led_example.c" ]
  include_dirs = [ ... ]

  # 编译时定义
  defines = [
    "LED_GPIO_PIN=9",
    "LED_BLINK_INTERVAL=300",
  ]
}
```

### 运行时配置

本项目不支持运行时配置（无配置文件）。所有配置通过编译时宏定义。

## 配置依赖关系

| 配置项 | 依赖配置项 | 说明 |
|-------|-----------|------|
| LED_TEST_GPIO | 硬件板型 | 不同板型 GPIO 引脚不同 |
| LED_TASK_PRIO | 系统调度 | 需要考虑其他任务优先级 |
| TASK_STACK_SIZE | 可用内存 | 不能超过系统可用内存 |

## 配置验证

### 1. GPIO 引脚号验证

在修改 GPIO 引脚号后，确保：

1. 引脚号在有效范围内
2. 引脚未被其他功能占用
3. 硬件连接正确

### 2. 任务栈大小验证

在修改任务栈大小后，确保：

1. 栈大小足够任务使用
2. 不会导致栈溢出
3. 不超过系统可用内存

### 3. 任务优先级验证

在修改任务优先级后，确保：

1. 优先级在有效范围内
2. 不会导致优先级反转
3. 系统调度正确

## 相关跳转链接

- [目录结构与模块职责](02_Directory_Structure.md)
- [GN 构建系统](06_GN_Build.md)
- [附录 A: 关键调用链](appendix/Callgraphs.md)
