# 项目概览

## 项目定位

HiTrace 是 OpenHarmony DFX 子系统的调用链追踪框架，提供跨设备、跨进程、跨线程的服务调用链追踪能力。

**证据来源**: `README.md:15` - "HiTrace provides APIs to implement call chain tracing throughout a service process"

## 核心能力

### 调用链追踪
- **TraceId 生成与传播**: 在分布式调用中传递唯一追踪标识
- **Span 管理**: 支持嵌套调用链的层级追踪
- **日志关联**: 自动将 traceId 关联到日志输出

### 追踪模式
- **同步追踪**: 追踪同步调用链路
- **异步追踪**: 追踪异步操作（需启用 `HITRACE_FLAG_INCLUDE_ASYNC`）
- **跨设备追踪**: 支持设备间的调用链关联（需启用 `HITRACE_FLAG_D2D_TP_INFO`）
- **故障触发**: 可通过故障事件触发追踪（`HITRACE_FLAG_FAULT_TRIGGER`）

## 运行环境

| 组件 | 要求 |
|------|------|
| 系统类型 | small, standard |
| 依赖子系统 | hiviewdfx, init, hilog |
| 运行时 | Native (C/C++) + JS/ArkTS |

**证据来源**: `bundle.json:27-30`

## 关键概念

### TraceId
调用链的唯一标识符，包含以下字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| chainId | uint64 | 调用链 ID |
| spanId | uint64 | Span ID |
| parentSpanId | uint64 | 父 Span ID |
| flags | uint32 | 追踪标志位 |

**证据来源**: `interfaces/native/innerkits/include/hitrace/hitraceid.h:25-94`

### HiTraceFlag
追踪行为控制标志：

| 标志 | 说明 |
|------|------|
| `HITRACE_FLAG_DEFAULT` | 默认行为 |
| `HITRACE_FLAG_INCLUDE_ASYNC` | 追踪异步调用 |
| `HITRACE_FLAG_DONOT_CREATE_SPAN` | 不创建 Span |
| `HITRACE_FLAG_TP_INFO` | 输出追踪点信息 |
| `HITRACE_FLAG_NO_BE_INFO` | 不输出开始/结束信息 |
| `HITRACE_FLAG_DONOT_ENABLE_LOG` | 不关联日志 |
| `HITRACE_FLAG_FAULT_TRIGGER` | 故障触发 |
| `HITRACE_FLAG_D2D_TP_INFO` | 跨设备追踪点信息 |

**证据来源**: `interfaces/js/kits/napi/src/napi_hitrace_init.cpp` - Enum 初始化

### 调用链通信模式

| 模式 | 说明 |
|------|------|
| `HITRACE_CM_DEFAULT` | 默认模式 |
| `HITRACE_CM_THREAD` | 线程内通信 |
| `HITRACE_CM_PROCESS` | 进程间通信 |
| `HITRACE_CM_DEVICE` | 设备间通信 |

## 目录结构

```
/base/hiviewdfx/hitrace
├── cmd                    # 命令行工具
│   └── hitrace_cmd.cpp   # hitrace 命令实现
├── common                 # 公共定义
│   ├── common_define.h   # 通用定义
│   ├── hitrace_define.h  # HiTrace 定义
│   └── smart_fd.h        # 智能文件描述符
├── config                 # 配置文件
├── example               # 示例代码
├── frameworks            # 框架实现
│   ├── native            # Native 实现
│   │   ├── hitracechain.cpp      # 调用链核心
│   │   ├── hitracechainc.c       # C 接口实现
│   │   ├── hitraceid.cpp         # TraceId 实现
│   │   ├── hitracechain_inner.h  # 内部接口
│   │   ├── dynamic_buffer.cpp    # 动态缓冲区
│   │   ├── c_wrapper/            # C 包装器
│   │   ├── tracedump_executor/   # 转储执行器
│   │   └── trace_factory/       # 追踪工厂
│   └── hitrace_ndk/       # NDK 接口
├── interfaces            # 对外接口
│   ├── js/kits/napi      # N-API 实现
│   │   ├── src/          # hiTraceChain N-API
│   │   ├── hitracemeter/ # hiTraceMeter N-API
│   │   └── bytrace_napi/ # 遗留 bytrace 接口
│   ├── native/innerkits  # Native Inner API
│   │   ├── include/hitrace/       # 调用链头文件
│   │   ├── include/hitrace_meter/ # 性能追踪头文件
│   │   ├── include/hitrace_option/ # 配置选项头文件
│   │   ├── include/hitrace_dump.h  # 转储头文件
│   │   └── src/           # Native 实现
│   ├── ets/ani           # ETS/ANI 接口 (ArkTS)
│   ├── cj/kits           # CJ (C-JavaScript) 接口
│   └── rust/innerkits    # Rust 接口
├── tools                 # 工具
│   └── hitrace_converter # 转换工具
└── utils                 # 工具类
    ├── common_utils.cpp/h  # 通用工具
    ├── file_ageing_utils.cpp/h  # 文件老化
    ├── trace_json_parser.cpp/h  # JSON 解析
    └── trace_file_utils.cpp/h  # 文件工具
```

**证据来源**: 目录结构分析 + `bundle.json:10-11`

## 依赖关系

### 系统依赖

| 组件 | 用途 |
|------|------|
| hilog | 日志输出 |
| hisysevent | 系统事件 |
| hiview | DFX 框架 |
| init | 初始化系统 |
| c_utils | C 工具库 |
| bounds_checking_function | 安全函数 |
| runtime_core | 运行时核心 |

**证据来源**: `bundle.json:34-47`

### 外部依赖

| 依赖 | 用途 |
|------|------|
| zlib | 压缩 |
| faultloggerd | 故障日志 |
| cJSON | JSON 解析 |
