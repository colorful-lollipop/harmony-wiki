# 关键配置项

## 概述

本文档汇总 `hidumper_lite` 项目中所有关键配置项，包括编译宏、构建配置和运行时参数。

---

## 编译宏

### OHOS_DEBUG

| 属性 | 值 |
|------|-----|
| 宏名称 | OHOS_DEBUG |
| 定义方式 | 编译时 -DOHOS_DEBUG |
| 默认状态 | 未定义 |
| 作用范围 | 调试功能条件编译 |

**控制的功能**：

| 功能 | 文件位置 | 受控行号 |
|------|----------|----------|
| DumpMemData | `lite/hidumper.c` | 120-130 |
| InjectKernelCrash | `lite/hidumper.c` | 134-140 |
| InjectUserCrash | `lite/hidumper.c` | 147-154 |
| DumpAllMem | `mini/hidumper_core.c` | 126-130 |
| DumpMemRegion | `mini/hidumper_core.c` | 138-144 |

**示例代码**：

```c
#ifdef OHOS_DEBUG
    int ret = ioctl(fd, HIDUMPER_MEM_DATA, param);
    if (ret < 0) {
        printf("Failed to ioctl [%s], error [%s]\n", HIDUMPER_DEVICE, strerror(errno));
    }
#else
    (void)fd;
    (void)param;
    printf("Unsupported!\n");
#endif
```

---

## 构建配置

### lite/BUILD.gn 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| target_type | static_library | 目标类型 |
| sources | ["hidumper.c"] | 源文件 |
| cflags | ["-Wall"] | 编译器警告 |
| include_dirs | 1 个目录 | 头文件搜索路径 |
| deps | 1 个依赖 | 链接依赖 |

### mini/BUILD.gn 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| target_type | static_library | 目标类型 |
| sources | 2 个文件 | 源文件列表 |
| cflags | ["-Wall"] | 编译器警告 |
| include_dirs | 6 个目录 | 头文件搜索路径 |
| deps | 1 个依赖 | 链接依赖 |

---

## 模块配置

### bundle.json 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| name | @ohos/hidumper_lite | 模块名称 |
| version | 4.0.2 | 版本号 |
| subsystem | hiviewdfx | 所属子系统 |
| adapted_system_type | ["mini"] | 适配系统类型 |
| rom | 26KB | ROM 占用 |
| ram | ~10KB | RAM 占用 |

---

## 设备节点配置

### 设备路径

| 属性 | 值 |
|------|-----|
| 设备路径 | /dev/hidumper |
| 定义位置 | `lite/hidumper.c:28` |
| 打开模式 | O_RDONLY |

**定义代码**：

```c
#define HIDUMPER_DEVICE  "/dev/hidumper"
```

---

## IOCTL 命令配置

### 命令定义

| 命令 | 值 | 功能 |
|------|-----|------|
| HIDUMPER_IOC_BASE | 'd' | IOCTL 基值 |
| HIDUMPER_DUMP_ALL | _IO('d', 1) | 全部信息 |
| HIDUMPER_CPU_USAGE | _IO('d', 2) | CPU 使用率 |
| HIDUMPER_MEM_USAGE | _IO('d', 3) | 内存使用率 |
| HIDUMPER_TASK_INFO | _IO('d', 4) | 任务信息 |
| HIDUMPER_INJECT_KERNEL_CRASH | _IO('d', 5) | 内核崩溃注入 |
| HIDUMPER_DUMP_FAULT_LOG | _IO('d', 6) | 故障日志 |
| HIDUMPER_MEM_DATA | _IOW('d', 7, struct MemDumpParam) | 内存数据 |

**证据位置**：`lite/hidumper.c:55-62`

---

## 内存转储参数配置

### MemDumpType 枚举

| 枚举值 | 功能 |
|--------|------|
| DUMP_TO_STDOUT | 转储到标准输出 |
| DUMP_REGION_TO_STDOUT | 转储指定区域到标准输出 |
| DUMP_TO_FILE | 转储到文件 |
| DUMP_REGION_TO_FILE | 转储指定区域到文件 |

**证据位置**：`lite/hidumper.c:41-46`

### MemDumpParam 结构体

| 成员 | 类型 | 功能 |
|------|------|------|
| type | enum MemDumpType | 转储类型 |
| start | unsigned long long | 起始地址 |
| size | unsigned long long | 转储大小 |
| filePath | char[256] | 文件路径 |

**证据位置**：`lite/hidumper.c:48-53`

---

## 参数常量定义

### 参数个数常量

| 常量 | 值 | 说明 |
|------|-----|------|
| ONE_OF_ARGC_PARAMETERS | 1 | 1 个参数 |
| TWO_OF_ARGC_PARAMETERS | 2 | 2 个参数 |
| THREE_OF_ARGC_PARAMETERS | 3 | 3 个参数 |
| FOUR_OF_ARGC_PARAMETERS | 4 | 4 个参数 |
| FIVE_OF_ARGC_PARAMETERS | 5 | 5 个参数 |

**证据位置**：
- `lite/hidumper.c:33-37`
- `mini/interfaces/native/kits/hidumper.h:25-28`

### 缓冲区大小常量

| 常量 | 值 | 说明 |
|------|-----|------|
| BUF_SIZE_16 | 16 | 十六进制转换基数 |
| PATH_MAX_LEN | 256 | 文件路径最大长度 |

**证据位置**：
- `lite/hidumper.c:39`
- `lite/hidumper.c:31`

---

## 适配器配置

### HiDumperAdapter 结构体

| 函数指针 | 参数 | 返回值 | 功能 |
|----------|------|--------|------|
| DumpSysInfo | void | int | 系统信息 |
| DumpCpuUsage | void | int | CPU 使用率 |
| DumpMemUsage | void | int | 内存使用率 |
| DumpTaskInfo | void | int | 任务信息 |
| DumpFaultLog | void | int | 故障日志 |
| DumpMemRegion | addr, size | int | 指定内存区域 |
| DumpAllMem | void | int | 所有内存数据 |

**证据位置**：`mini/interfaces/native/innerkits/hidumper.h:25-33`

---

## 初始化配置

### CORE_INIT_PRI 配置

| 配置项 | 值 |
|------|-----|
| 宏名称 | CORE_INIT_PRI |
| 函数名 | HiDumperAdapterInit |
| 优先级 | 3 |

**证据位置**：`mini/hidumper_adapter.c:102`

---

## 安全相关配置

### 地址常量

| 常量 | 值 | 说明 |
|------|-----|------|
| USER_FAULT_ADDR | 0x3 | 用户态崩溃地址（调试用） |
| USER_FAULT_VALUE | 0x4 | 用户态崩溃值（调试用） |

**证据位置**：`lite/hidumper.c:29-30`

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [04_Build.md](../04_Build.md) | 构建配置说明 |
| [05_Security.md](../05_Security.md) | 安全评审 |
| [Callgraphs.md](Callgraphs.md) | 关键调用链 |

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0.0 | 2026-02-06 | 初始版本 |
