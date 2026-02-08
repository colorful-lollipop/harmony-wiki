# 接口文档

## 概述

本文档描述 `hidumper_lite` 项目的所有编程接口，包括命令行工具、AT 命令接口以及平台适配接口。本项目**不涉及 N-API（JS API）**，所有接口均为 C/C++ 接口。

---

## 命令行工具（LiteOS_A）

### 工具概述

LiteOS_A 版本提供独立的命令行工具 `hidumper`，通过 `/dev/hidumper` 设备节点与内核通信。

**证据位置**：`lite/hidumper.c`

### 命令格式

#### 基本命令

| 命令格式 | 功能描述 |
|----------|----------|
| `hidumper` | 转储 CPU 使用率、内存使用率和所有任务信息 |
| `hidumper -dc` | 转储 CPU 使用率 |
| `hidumper -dm` | 转储内存使用率 |
| `hidumper -dt` | 转储所有任务信息 |
| `hidumper -df` | 转储故障日志 |
| `hidumper -h` | 打印帮助信息 |

#### 调试命令（仅调试版本）

| 命令格式 | 功能描述 |
|----------|----------|
| `hidumper -ikc` | 注入内核崩溃 |
| `hidumper -iuc` | 注入用户态崩溃 |
| `hidumper -m` | 转储所有内存数据到标准输出 |
| `hidumper -m filepath` | 转储所有内存数据到指定文件 |
| `hidumper -m memstart memsize` | 转储指定内存区域到标准输出 |
| `hidumper -m memstart memsize filepath` | 转储指定内存区域到文件 |

**证据位置**：`lite/hidumper.c:64-83`（Usage 函数）

### 程序入口

```c
int main(int argc, const char *argv[])
```

**功能**：程序入口函数

**参数**：
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| argc | int | 是 | 参数个数 |
| argv | const char** | 是 | 参数数组 |

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| -1 | 失败（无法打开设备或参数错误） |

**执行流程**：
1. 打开 `/dev/hidumper` 设备节点
2. 调用 `ParameterMatching()` 解析参数
3. 关闭设备描述符
4. 返回结果

**证据位置**：`lite/hidumper.c:205-217`

### IOCTL 命令

#### 命令定义

```c
#define HIDUMPER_IOC_BASE            'd'
#define HIDUMPER_DUMP_ALL            _IO(HIDUMPER_IOC_BASE, 1)
#define HIDUMPER_CPU_USAGE           _IO(HIDUMPER_IOC_BASE, 2)
#define HIDUMPER_MEM_USAGE           _IO(HIDUMPER_IOC_BASE, 3)
#define HIDUMPER_TASK_INFO           _IO(HIDUMPER_IOC_BASE, 4)
#define HIDUMPER_INJECT_KERNEL_CRASH _IO(HIDUMPER_IOC_BASE, 5)
#define HIDUMPER_DUMP_FAULT_LOG      _IO(HIDUMPER_IOC_BASE, 6)
#define HIDUMPER_MEM_DATA            _IOW(HIDUMPER_IOC_BASE, 7, struct MemDumpParam)
```

**证据位置**：`lite/hidumper.c:55-62`

#### MemDumpParam 结构体

```c
enum MemDumpType {
    DUMP_TO_STDOUT,
    DUMP_REGION_TO_STDOUT,
    DUMP_TO_FILE,
    DUMP_REGION_TO_FILE
};

struct MemDumpParam {
    enum MemDumpType type;
    unsigned long long start;
    unsigned long long size;
    char filePath[PATH_MAX_LEN];  // PATH_MAX_LEN = 256
};
```

**证据位置**：`lite/hidumper.c:41-53`

#### 执行函数

```c
static void ExecAction(int fd, unsigned int cmd)
```

**功能**：执行 IOCTL 命令

**参数**：
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| fd | int | 是 | 设备文件描述符 |
| cmd | unsigned int | 是 | IOCTL 命令 |

**返回值**：无

**证据位置**：`lite/hidumper.c:85-96`

---

## AT 命令接口（LiteOS_M）

### 接口概述

LiteOS_M 版本通过 AT 框架提供命令行接口，入口函数为 `at_hidumper()`。

**证据位置**：`mini/hidumper_core.c:155-165`

### at_hidumper

```c
unsigned int at_hidumper(unsigned int argc, const char **argv)
```

**功能**：AT 命令入口函数，由 AT 框架调用

**参数**：
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| argc | unsigned int | 是 | 参数个数 |
| argv | const char** | 是 | 参数数组 |

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 1 | 失败（适配器未注册） |

**前置条件**：
- 必须已调用 `HiDumperRegisterAdapter()` 注册适配器
- 全局变量 `g_isAdapterRegistered` 必须为 1

**证据位置**：`mini/hidumper_core.c:155-165`

### AT 命令格式

| AT 命令格式 | 功能描述 |
|-------------|----------|
| `AT+HIDUMPER=` | 转储 CPU、内存和所有任务信息 |
| `AT+HIDUMPER=-dc` | 转储 CPU 使用率 |
| `AT+HIDUMPER=-dm` | 转储内存使用率 |
| `AT+HIDUMPER=-dt` | 转储所有任务信息 |
| `AT+HIDUMPER=-df` | 转储最新故障日志 |
| `AT+HIDUMPER=-h` | 打印帮助信息 |
| `AT+HIDUMPER=-ikc` | 注入内核崩溃 |
| `AT+HIDUMPER=-m` | 转储内存数据（调试版本） |
| `AT+HIDUMPER=-m,memstart,memsize` | 转储指定内存区域（调试版本） |

**证据位置**：`mini/hidumper_core.c:38-55`

---

## 平台适配接口

### 概述

平台适配接口定义了 `hidumper_lite` 与各芯片平台之间的契约。各平台必须实现这些接口函数，并通过 `HiDumperRegisterAdapter()` 注册适配器。

**证据位置**：
- 接口声明：`mini/interfaces/native/innerkits/hidumper_adapter.h`
- 接口实现：`mini/hidumper_adapter.c`

### HiDumperAdapter 结构体

```c
struct HiDumperAdapter {
    int (*DumpSysInfo)(void);
    int (*DumpCpuUsage)(void);
    int (*DumpMemUsage)(void);
    int (*DumpTaskInfo)(void);
    int (*DumpFaultLog)(void);
    int (*DumpMemRegion)(unsigned long long addr, unsigned long long size);
    int (*DumpAllMem)(void);
};
```

**证据位置**：`mini/interfaces/native/innerkits/hidumper.h:25-33`

### HiDumperRegisterAdapter

```c
int HiDumperRegisterAdapter(struct HiDumperAdapter *pAdapter)
```

**功能**：注册平台适配器

**参数**：
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| pAdapter | struct HiDumperAdapter* | 是 | 适配器结构体指针 |

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| -1 | 失败（参数为空或函数指针为空） |

**参数校验**：
- 检查 `pAdapter != NULL`
- 检查所有函数指针均不为 NULL

**证据位置**：`mini/hidumper_core.c:79-103`

### DumpSysInfo

```c
WEAK int DumpSysInfo(void)
```

**功能**：打印系统信息

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败（平台自定义错误码） |

**默认实现**：打印 "Please implement the interface according to the platform!"

**平台实现示例**：`device/soc/hisilicon/hi3861v100/sdk_liteos/components/at/src/hidumper_adapter_impl.c`

**证据位置**：`mini/hidumper_adapter.c:33-37`

### DumpCpuUsage

```c
WEAK int DumpCpuUsage(void)
```

**功能**：打印 CPU 使用率信息

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败 |

**默认实现**：打印提示信息

**证据位置**：`mini/hidumper_adapter.c:39-43`

### DumpMemUsage

```c
WEAK int DumpMemUsage(void)
```

**功能**：打印内存使用率信息

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败 |

**默认实现**：打印提示信息

**证据位置**：`mini/hidumper_adapter.c:45-50`

### DumpTaskInfo

```c
WEAK int DumpTaskInfo(void)
```

**功能**：打印所有任务信息

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败 |

**默认实现**：打印提示信息

**证据位置**：`mini/hidumper_adapter.c:52-56`

### DumpFaultLog

```c
WEAK int DumpFaultLog(void)
```

**功能**：打印故障日志

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败 |

**默认实现**：打印提示信息

**证据位置**：`mini/hidumper_adapter.c:58-62`

### DumpMemRegion

```c
WEAK int DumpMemRegion(unsigned long long addr, unsigned long long size)
```

**功能**：打印指定内存区域的数据

**参数**：
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| addr | unsigned long long | 是 | 内存起始地址（十六进制） |
| size | unsigned long long | 是 | 内存大小（十六进制） |

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败 |

**默认实现**：打印提示信息

**证据位置**：`mini/hidumper_adapter.c:64-70`

### DumpAllMem

```c
WEAK int DumpAllMem(void)
```

**功能**：打印所有内存数据（十六进制格式）

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败 |

**默认实现**：打印提示信息

**证据位置**：`mini/hidumper_adapter.c:72-76`

### PlatformHiDumperIinit

```c
WEAK int PlatformHiDumperIinit(void)
```

**功能**：平台特定初始化

**返回值**：
| 返回值 | 描述 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败 |

**默认实现**：打印提示信息

**证据位置**：`mini/hidumper_adapter.c:78-82`

---

## 接口使用示例

### 示例一：注册平台适配器

```c
#include "hidumper.h"
#include "hidumper_adapter.h"

static int MyDumpCpuUsage(void)
{
    // 平台特定的 CPU 使用率采集实现
    printf("CPU Usage: 25%%\n");
    return 0;
}

static int MyDumpMemUsage(void)
{
    // 平台特定的内存使用率采集实现
    printf("Memory Usage: 1024KB / 4096KB\n");
    return 0;
}

// ... 其他函数实现

static struct HiDumperAdapter g_myAdapter = {
    .DumpSysInfo = MyDumpSysInfo,
    .DumpCpuUsage = MyDumpCpuUsage,
    .DumpMemUsage = MyDumpMemUsage,
    .DumpTaskInfo = MyDumpTaskInfo,
    .DumpFaultLog = MyDumpFaultLog,
    .DumpMemRegion = MyDumpMemRegion,
    .DumpAllMem = MyDumpAllMem,
};

void MyPlatformInit(void)
{
    HiDumperRegisterAdapter(&g_myAdapter);
    PlatformHiDumperIinit();
}
```

### 示例二：AT 命令调用

```c
// 在 AT 任务中
const char *argv[] = {"-dc"};
at_hidumper(1, argv);  // 输出 CPU 使用率

const char *argv2[] = {"-dm"};
at_hidumper(1, argv2);  // 输出内存使用率
```

### 示例三：命令行工具调用

```bash
# 转储所有信息
hidumper

# 仅转储 CPU 使用率
hidumper -dc

# 转储指定内存区域
hidumper -m 0x20000000 0x100

# 转储到文件
hidumper -m 0x20000000 0x100 /data/mem_dump.bin
```

---

## 错误码说明

### 全局错误码

| 错误码 | 宏定义 | 描述 |
|--------|--------|------|
| -1 | 无 | 参数无效或操作失败 |

### 错误处理策略

1. **参数校验错误**：打印错误信息并返回 -1
2. **设备打开失败**：打印错误信息（包含 errno）并返回 -1
3. **IOCTL 失败**：打印错误信息（包含 errno）但不终止程序
4. **适配器未注册**：打印 "No adapter has been registered!" 并返回 1

**证据位置**：
- 参数校验：`lite/hidumper.c:178-181, 193-196`
- 设备打开失败：`lite/hidumper.c:209-211`
- IOCTL 失败：`lite/hidumper.c:92-95`
- 适配器未注册：`mini/hidumper_core.c:157-160`

---

## 线程安全性

### 线程安全说明

| 接口 | 线程安全性 | 说明 |
|------|------------|------|
| `main()` | 线程安全 | 独立进程，无并发问题 |
| `at_hidumper()` | 线程安全 | 在 AT 任务中串行执行 |
| `HiDumperRegisterAdapter()` | 线程安全 | 仅在初始化阶段调用 |
| 适配器函数指针 | 取决于实现 | 平台需确保线程安全 |

### 资源共享

- `g_isAdapterRegistered`：全局只读标志
- `g_hidumperAdapter`：全局只读结构体

**证据位置**：`mini/hidumper_core.c:35-36`

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [02_Architecture.md](02_Architecture.md) | 架构说明 |
| [04_Build.md](04_Build.md) | 构建配置说明 |
| [06_Troubleshooting.md](06_Troubleshooting.md) | 问题排查指南 |
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 |

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0.0 | 2026-02-06 | 初始版本 |
