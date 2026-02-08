# 目录结构与模块职责

## 目的

本文档说明 hiperf 项目的目录结构、各模块职责和文件组织方式。

## 适用范围

- 新加入的开发者
- 需要理解代码结构的贡献者
- 进行代码审查的人员

## 顶层目录结构

```
developtools/hiperf/
├── BUILD.gn              # 主构建文件
├── hiperf.gni            # 构建配置定义
├── bundle.json           # 组件配置
├── LICENSE               # 许可证文件
├── README.md             # 项目说明（英文）
├── README_zh.md          # 项目说明（中文）
│
├── demo/                 # 示例程序
├── etc/                  # 配置文件
├── figures/              # 文档图片
├── include/              # 公共头文件
├── interfaces/           # API 接口
├── proto/                # ProtoBuf 定义
├── script/               # Python 脚本
└── src/                  # 源代码
```

## 各目录详细说明

### 1. demo/ - 示例程序

**路径**: `demo/`

**职责**: 提供 hiperf API 使用示例

```
demo/
├── cpp/                  # C++ 示例
│   ├── BUILD.gn
│   ├── hiperf_demo.cpp           # 基础 API 使用示例
│   ├── hiperf_example_cmd.cpp    # 测试命令（模拟采样场景）
│   └── hiperf_malloc_demo.cpp    # 内存分配采样示例
│
└── js/                   # JS 示例（仅展示命令行调用）
    ├── build.gradle
    ├── package.json
    └── entry/src/main/js/MainAbility/
        └── pages/
            ├── index/index.js    # 首页示例
            └── second/second.js  # 第二页示例
```

**重要说明**: 
- JS demo 仅展示如何通过命令行调用 hiperf，**hiperf 不提供 N-API 接口**
- C++ demo 展示 `hiperf_client` API 的正确使用方式

### 2. etc/ - 配置文件

**路径**: `etc/`

**职责**: 系统配置文件

| 文件 | 说明 | 安装路径 |
|------|------|----------|
| `hiperf.cfg` | init 配置 | `/system/etc/init/` |
| `hiperf.para` | 系统参数 | `/system/etc/param/` |
| `hiperf.para.dac` | DAC 配置 | `/system/etc/param/` |

**hiperf.cfg** (`etc/hiperf.cfg`):
```json
{
    "jobs": [{
        "name": "post-fs-data",
        "cmds": [
            "mkdir /data/log/hiperflog 0770 shell log",
            "restorecon /data/log/hiperflog",
            "chmod 0666 /dev/lperf"
        ]
    }]
}
```

### 3. include/ - 公共头文件

**路径**: `include/`

**职责**: 定义公共接口和数据结构

**头文件清单** (42个):

| 头文件 | 说明 | 关键定义 |
|--------|------|----------|
| `callstack.h` | 调用链回溯 | `CallStack` 类 |
| `command.h` | 命令基类 | `Command` 类 |
| `command_reporter.h` | 命令报告 | `CommandReporter` 类 |
| `debug_logger.h` | 调试日志 | `HLOGD/HLOGI/HLOGE` 宏 |
| `dwarf_encoding.h` | DWARF 编码 | DWARF 解析相关 |
| `hashlist.h` | 哈希列表 | `HashList` 类 |
| `hiperf_hilog.h` | HiLog 日志 | 日志模块封装 |
| `hiperf_libreport.h` | Host 报告库 | `HiperfLibReport` 类 |
| `ipc_utilities.h` | IPC 工具 | `IsDebugableApp()` 等 |
| `noncopyable.h` | 不可复制基类 | `NonCopyable` 类 |
| `option.h` | 命令选项 | `Option` 类 |
| `option_debug.h` | 调试选项 | 调试专用选项 |
| `perf_event_record.h` | Perf 事件记录 | `PerfEventRecord` 类 |
| `perf_events.h` | Perf 事件管理 | `PerfEvents` 类 |
| `perf_file_format.h` | 文件格式 | 数据文件格式定义 |
| `perf_file_reader.h` | 文件读取 | `PerfFileReader` 类 |
| `perf_file_writer.h` | 文件写入 | `PerfFileWriter` 类 |
| `perf_pipe.h` | 管道通信 | `PerfPipe` 类 |
| `perf_record_format.h` | 记录格式 | 记录类型定义 |
| `register.h` | 寄存器定义 | 寄存器相关结构 |
| `report.h` | 报告生成 | `Report` 类 |
| `report_json_file.h` | JSON 报告 | `ReportJSONFile` 类 |
| `report_protobuf_file.h` | ProtoBuf 报告 | `ReportProtobufFile` 类 |
| `ring_buffer.h` | 环形缓冲区 | `RingBuffer` 类 |
| `spe_decoder.h` | SPE 解码 | `SpeDecoder` 类 |
| `subcommand.h` | 子命令基类 | `SubCommand` 类 |
| `subcommand_dump.h` | dump 子命令 | `SubCommandDump` 类 |
| `subcommand_help.h` | help 子命令 | `SubCommandHelp` 类 |
| `subcommand_list.h` | list 子命令 | `SubCommandList` 类 |
| `subcommand_record.h` | record 子命令 | `SubCommandRecord` 类 |
| `subcommand_report.h` | report 子命令 | `SubCommandReport` 类 |
| `subcommand_stat.h` | stat 子命令 | `SubCommandStat` 类 |
| `symbols_file.h` | 符号文件 | `SymbolsFile` 类 |
| `tracked_command.h` | 追踪命令 | `TrackedCommand` 类 |
| `unique_stack_table.h` | 唯一栈表 | `UniqueStackTable` 类 |
| `utilities.h` | 工具函数 | 各种工具函数 |
| `virtual_runtime.h` | 虚拟运行时 | `VirtualRuntime` 类 |
| `virtual_thread.h` | 虚拟线程 | `VirtualThread` 类 |

**nonlinux/** - 非 Linux 平台适配:
- `asm/byteorder.h` - 字节序适配
- `linux/ioctl.h` - ioctl 适配
- `linux/perf_event_host.h` - Host 端 perf_event 定义
- `linux/types.h` - 类型定义
- `MingW64Fix.h` - Windows MinGW 适配

### 4. interfaces/ - API 接口

**路径**: `interfaces/innerkits/native/`

**职责**: 对外暴露的 C++ API

```
interfaces/innerkits/native/
├── hiperf_client/              # 完整功能客户端 API
│   ├── BUILD.gn
│   ├── include/
│   │   └── hiperf_client.h     # 头文件
│   └── src/
│       └── hiperf_client.cpp   # 实现
│
└── hiperf_local/               # 轻量级本地采样 API
    ├── BUILD.gn
    ├── hiperf_local.map        # 符号导出控制
    ├── include/
    │   └── lperf.h             # 头文件
    └── src/
        └── lperf.cpp           # 实现
```

**API 对比**:

| 特性 | hiperf_client | hiperf_local |
|------|---------------|--------------|
| 功能 | 完整采样控制 | 轻量级堆栈采样 |
| 通信 | 管道 + fork/exec | 直接调用 |
| 依赖 | hiperf 二进制 | libdfx_dumpcatcher |
| 输出 | perf.data 文件 | 字符串堆栈 |
| 使用场景 | 完整性能分析 | 实时轻量级采样 |

### 5. proto/ - ProtoBuf 定义

**路径**: `proto/`

**职责**: 定义 ProtoBuf 数据格式

| 文件 | 说明 |
|------|------|
| `report_sample.proto` | 采样数据格式定义 |
| `build_proto.sh` | 编译脚本 |

### 6. script/ - Python 脚本

**路径**: `script/`

**职责**: Host 端辅助脚本

| 文件 | 说明 |
|------|------|
| `command_script.py` | 采样命令包装脚本 |
| `hiperf_utils.py` | 工具函数库 |
| `make_report.py` | 生成 HTML 报告 |
| `recv_binary_cache.py` | 收集符号表 |
| `make_diff.py` | 生成对比报告 |
| `loadlib_test.py` | 库加载测试 |
| `report.html` | HTML 报告模板 |

### 7. src/ - 源代码

**路径**: `src/`

**职责**: 核心功能实现

#### 7.1 程序入口

| 文件 | 说明 |
|------|------|
| `main.cpp` | 程序入口，命令分发 |

#### 7.2 命令框架

| 文件 | 说明 | 关键类/函数 |
|------|------|-------------|
| `command.cpp` | 命令管理 | `Command::DispatchCommands()` |
| `command_reporter.cpp` | 命令报告 | `CommandReporter` |
| `subcommand.cpp` | 子命令基类 | `SubCommand` |
| `subcommand_help.cpp` | help 命令 | `SubCommandHelp` |
| `subcommand_list.cpp` | list 命令 | `SubCommandList` |
| `subcommand_stat.cpp` | stat 命令 | `SubCommandStat` |
| `subcommand_record.cpp` | record 命令 | `SubCommandRecord` |
| `subcommand_dump.cpp` | dump 命令 | `SubCommandDump` |
| `subcommand_report.cpp` | report 命令 | `SubCommandReport` |

#### 7.3 核心功能

| 文件 | 说明 | 关键类/函数 |
|------|------|-------------|
| `perf_events.cpp` | perf 事件管理 | `PerfEvents` |
| `perf_event_record.cpp` | 事件记录处理 | `PerfEventRecord` |
| `perf_file_reader.cpp` | 数据文件读取 | `PerfFileReader` |
| `perf_file_writer.cpp` | 数据文件写入 | `PerfFileWriter` |
| `virtual_runtime.cpp` | 虚拟运行时 | `VirtualRuntime` |
| `virtual_thread.cpp` | 虚拟线程 | `VirtualThread` |
| `symbols_file.cpp` | 符号文件处理 | `SymbolsFile` |
| `callstack.cpp` | 调用链回溯 | `CallStack` |
| `ring_buffer.cpp` | 环形缓冲区 | `RingBuffer` |
| `spe_decoder.cpp` | SPE 解码 | `SpeDecoder` |

#### 7.4 报告生成

| 文件 | 说明 |
|------|------|
| `report.cpp` | 报告基类 |
| `report_json_file.cpp` | JSON 格式报告 |
| `report_protobuf_file.cpp` | ProtoBuf 格式报告 |

#### 7.5 工具与辅助

| 文件 | 说明 |
|------|------|
| `utilities.cpp` | 通用工具函数 |
| `option.cpp` | 命令选项处理 |
| `option_debug.cpp` | 调试选项 |
| `debug_logger.cpp` | 调试日志 |
| `ipc_utilities.cpp` | IPC 工具（权限检查） |
| `register.cpp` | 寄存器处理 |
| `dwarf_encoding.cpp` | DWARF 编码处理 |
| `unique_stack_table.cpp` | 唯一栈表 |
| `perf_pipe.cpp` | 管道通信 |
| `tracked_command.cpp` | 追踪命令 |

#### 7.6 Host 端专用

| 文件 | 说明 |
|------|------|
| `hiperf_libreport.cpp` | Host 端报告库 |
| `hiperf_libreport_demo.cpp` | Host 端库示例 |
| `mingw_adapter.cpp` | Windows MinGW 适配 |

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                        应用层                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  demo/cpp   │  │  demo/js    │  │   script/   │          │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │
└─────────┼────────────────┼────────────────┼─────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────┐
│                        API 层                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              interfaces/innerkits/                   │   │
│  │  ┌─────────────┐        ┌─────────────┐             │   │
│  │  │hiperf_client│        │hiperf_local │             │   │
│  │  └─────────────┘        └─────────────┘             │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────┐
│                        命令层                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                     src/main.cpp                     │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │   │
│  │  │  list   │ │  stat   │ │ record  │ │ report  │   │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────┐
│                        核心层                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  PerfEvents  │  │VirtualRuntime│  │ SymbolsFile  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   CallStack  │  │  RingBuffer  │  │   SpeDecoder │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────┐
│                        系统层                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  perf_event  │  │    /proc     │  │   IPC/SAMGR  │      │
│  │   (内核)      │  │  (进程信息)   │  │  (系统服务)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## 关键结论

1. **无 N-API**: hiperf 仅提供 C++ API，不提供 JS API
2. **分层清晰**: 从 API 层到系统层，职责分明
3. **命令模式**: 使用子命令模式组织功能（list/stat/record/report/dump）
4. **数据流**: perf_event → RingBuffer → VirtualRuntime → 输出文件
5. **Host/Target 分离**: 支持设备端和 Host 端两种构建目标

## 相关跳转

- [项目定位](01_Overview.md) - 了解项目边界
- [架构说明](03_Architecture.md) - 深入了解架构设计
- [对外 API](04_Public_API.md) - API 使用说明
- [GN Targets](06_GN_Targets.md) - 构建目标说明
