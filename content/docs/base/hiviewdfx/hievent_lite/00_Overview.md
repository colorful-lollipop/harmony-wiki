# 00_项目概览

> hievent_lite 项目定位、边界与核心能力说明

## 1. 项目定位

### 1.1 概述

**hievent_lite** 是 OpenHarmony DFX（Diagnostics, Fault-tolerance, eXtensibility）子系统下的**轻量级事件日志组件**，专为 **LiteOS-M** 内核设计。

**证据**: `bundle.json:3` - `"description": "event log for liteos-m kernel"`

```
子系统: hiviewdfx (DFX)
组件名: @ohos/hievent_lite
版本: 3.1
适配系统: mini (轻量级设备)
```

### 1.2 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| **事件创建** | 支持三类事件的结构化创建 | `frameworks/hiview_event.c:73` - `HiEventCreate()` |
| **快速上报** | 单参数事件快速上报接口 | `frameworks/hiview_event.c:51` - `HiEventPrintf()` |
| **TLV 编码** | 1-4字节变长整数编码 | `frameworks/hiview_event.c:129` - `HiEventEncode()` |
| **缓存管理** | 三类事件内存缓存 | `frameworks/hiview_output_event.c:35-46` |
| **文件持久化** | 二进制格式写入 Flash | `frameworks/hiview_output_event.c:320` - `Output2Flash()` |
| **UART 实时输出** | 调试日志实时输出 | `frameworks/hiview_output_event.c:279` - `OutputEventRealtime()` |

### 1.3 资源占用

| 资源 | 大小 | 来源 |
|------|------|------|
| ROM | 26 KB | `bundle.json:27` |
| RAM | ~10 KB | `bundle.json:28` |

## 2. 项目边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      hievent_lite 项目边界                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  核心功能（本项目实现）                                    │    │
│  │  ├── 事件对象管理 (HiEvent)                              │    │
│  │  ├── TLV 二进制序列化                                    │    │
│  │  ├── 三类事件缓存 (Fault/UE/Stat)                        │    │
│  │  ├── 事件文件输出                                         │    │
│  │  └── 命令行控制                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌──────────────────┐      ┌──────────────────┐               │
│  │  上游依赖         │      │  下游消费          │               │
│  │  hiview_lite     │ ───▶ │  事件文件 (.bin)   │               │
│  │  hilog_lite      │      │  UART 输出         │               │
│  │  samgr_lite      │      │  上传组件 (外部)    │               │
│  │  utils_lite      │      │                   │               │
│  └──────────────────┘      └──────────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.1 上游依赖

| 依赖组件 | 用途 | 来源 |
|----------|------|------|
| **hiview_lite** | 基础服务、配置文件、文件系统抽象 | `BUILD.gn:44` - `deps` |
| **hilog_lite** | 日志输出 | `BUILD.gn:37-38` - `include_dirs` |
| **samgr_lite** | 系统能力管理 | `BUILD.gn:41` - `include_dirs` |
| **utils_lite** | 通用工具函数 | `BUILD.gn:39` - `include_dirs` |

**证据**: `BUILD.gn:35-44`

### 2.2 下游输出

| 输出类型 | 说明 | 格式 |
|----------|------|------|
| **故障事件文件** | Flash 存储 | 二进制 TLV 格式 |
| **用户行为事件文件** | Flash 存储 | 二进制 TLV 格式 |
| **统计事件文件** | Flash 存储 | 二进制 TLV 格式 |
| **UART 实时输出** | 调试信息 | ASCII 文本格式 |

## 3. 事件类型

hievent_lite 定义了三类事件：

| 类型 | 值 | 常量定义 | 说明 | 代码位置 |
|------|-----|----------|------|----------|
| **故障事件** | 1 | `HIEVENT_FAULT` | 系统故障、异常捕获 | `hiview_event.h:31` |
| **用户行为事件** | 2 | `HIEVENT_UE` | 用户操作、行为追踪 | `hiview_event.h:34` |
| **统计事件** | 4 | `HIEVENT_STAT` | 性能统计、数据采集 | `hiview_event.h:37` |

**证据**: `interfaces/native/innerkits/hiview_event.h:27-38`

```c
#ifndef HIEVENT_NONE
#define HIEVENT_NONE   0   /* none event            */
#endif
#ifndef HIEVENT_FAULT
#define HIEVENT_FAULT  1   /* fault event           */
#endif
#ifndef HIEVENT_UE
#define HIEVENT_UE     2   /* user behavior event   */
#endif
#ifndef HIEVENT_STAT
#define HIEVENT_STAT   4   /* statistics event      */
#endif
```

## 4. 目录结构

```
hievent_lite/
├── .gitee/                              # Gitee 平台配置
├── BUILD.gn                             # 根构建配置
├── bundle.json                          # 组件描述文件
├── LICENSE                              # Apache 2.0
├── README.md / README_zh.md             # 项目说明
│
├── command/                             # [模块] 命令行工具
│   ├── BUILD.gn
│   ├── hievent_lite_command.c          # 命令实现
│   └── hievent_lite_command.h          # 命令头文件
│
├── frameworks/                          # [模块] 核心实现
│   ├── hiview_event.c                  # 事件创建/上报
│   ├── hiview_output_event.c           # 事件输出/文件
│   └── hiview_output_event.h           # 输出头文件
│
└── interfaces/                          # [模块] 对外接口
    └── native/
        └── innerkits/
            ├── event.h                # 兼容头文件
            └── hiview_event.h          # 主 API 头文件
```

### 4.1 模块职责

| 目录 | 职责 | 核心文件 |
|------|------|----------|
| **command/** | Shell 命令处理、运行时开关 | `hievent_lite_command.c` |
| **frameworks/** | 事件生命周期、编码、输出 | `hiview_event.c`, `hiview_output_event.c` |
| **interfaces/** | C API 定义 | `hiview_event.h` |

## 5. 关键概念

### 5.1 HiEvent 结构体

```
┌─────────────────────────────────────────┐
│              HiEvent                     │
├─────────────────────────────────────────┤
│  HiEventCommon common                   │  ← 公共头部 (8字节)
│  ├── uint8  mark                        │     事件头标记
│  ├── uint8  len                        │     数据长度
│  ├── uint16 eventId                    │     事件 ID
│  └── uint32 time                       │     时间戳
├─────────────────────────────────────────┤
│  uint8 type                            │  ← 事件类型
│  uint8 *payload                       │  ← TLV 编码数据
└─────────────────────────────────────────┘
```

**证据**: `interfaces/native/innerkits/hiview_event.h:40-53`

### 5.2 TLV 编码格式

事件数据采用 **TLV (Type-Length-Value)** 变长编码：

```
┌──────────┬──────────┬────────────┐
│ 1 bit    │ 3 bits   │  1-4 bytes │
│  (last)  │  (key)   │   (value)  │
└──────────┴──────────┴────────────┘
```

**证据**: `frameworks/hiview_event.c:129-171` - `HiEventEncode()`

| Value 范围 | 编码长度 |
|------------|----------|
| 0x00 - 0xFF | 1 字节 |
| 0x100 - 0xFFFF | 2 字节 |
| 0x10000 - 0xFFFFFF | 3 字节 |
| > 0xFFFFFF | 4 字节 |

### 5.3 编译时开关

通过 `HIEVENT_COMPILE_TYPE` 宏控制编译的事件类型：

**证据**: `interfaces/native/innerkits/hiview_event.h:173-178`

```c
#ifndef HIEVENT_COMPILE_TYPE
#define HIEVENT_COMPILE_TYPE (HIEVENT_FAULT | HIEVENT_UE | HIEVENT_STAT)
#endif
```

## 6. 运行环境

### 6.1 适配系统

| 系统类型 | 说明 | 配置值 |
|----------|------|--------|
| **mini** | 轻量级设备 (LiteOS-M) | `bundle.json:17` |

### 6.2 初始化流程

**证据**: `frameworks/hiview_event.c:40-49`

```c
static void HiEventInit(void)
{
    HIVIEW_UartPrint("hievent will init.\n");
    if (g_hiviewConfig.eventSwitch == HIVIEW_FEATURE_ON && 
        HIEVENT_COMPILE_TYPE > HIEVENT_NONE) {
        InitCoreEventOutput();
        HiviewRegisterInitFunc(HIVIEW_CMP_TYPE_EVENT, InitEventOutput);
        HIVIEW_UartPrint("hievent init success.");
    }
}
CORE_INIT_PRI(HiEventInit, 1);  // 优先级 1
```

### 6.3 功能开关

| 状态 | 常量 | 说明 |
|------|------|------|
| **开启** | `HIVIEW_FEATURE_ON` | 事件功能可用 |
| **关闭** | `HIVIEW_FEATURE_OFF` | 事件静默丢弃 |

**证据**: `command/hievent_lite_command.c:83-89` - `HieventSetProc()`

## 7. 相关仓库

| 仓库 | 关系 | 说明 |
|------|------|------|
| [hiviewdfx_hilog_lite](https://gitee.com/openharmony/hiviewdfx_hilog_lite) | 依赖 | 日志输出组件 |
| [hiviewdfx_hiview_lite](https://gitee.com/openharmony/hiviewdfx_hiview_lite) | 依赖 | 基础服务组件 |
| [DFX 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md) | 所属 | 父级子系统 |

---

**跳转**: [01_Architecture.md](01_Architecture.md) | [02_API_Reference.md](02_API_Reference.md) | [SUMMARY.md](SUMMARY.md)
