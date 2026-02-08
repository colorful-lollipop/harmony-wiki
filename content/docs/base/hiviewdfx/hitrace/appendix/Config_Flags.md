# 附录：配置参数

## Feature Flags

### 全局配置 (hitrace.gni)

**文件位置**: `/base/hiviewdfx/hitrace/hitrace.gni`

#### 功能开关

| 参数 | 类型 | 默认值 | 范围 | 说明 |
|------|------|--------|------|------|
| `hitrace_support_executable_file` | bool | true | - | 是否支持可执行文件 |
| `hitrace_snapshot_tracebuffer_size` | int | 0 | 0+ | 快照追踪缓冲区大小 (KB) |
| `hitrace_snapshot_file_limit` | int | 0 | 0+ | 快照文件大小限制 |
| `hitrace_record_file_limit` | int | 0 | 0+ | 记录文件大小限制 |
| `hitrace_feature_enable_pgo` | bool | false | - | 是否启用 PGO 优化 |
| `hitrace_feature_pgo_path` | string | "" | 路径 | PGO 数据路径 |
| `use_shared_libz` | bool | true | - | 是否使用共享 zlib |
| `hitrace_feature_support_usr_symlink` | bool | false | - | 是否支持 /usr/bin 符号链接 |
| `hiview_enable` | bool | auto | - | 是否启用 HiView 集成 |

**证据来源**: `hitrace.gni:23-37`

---

## 系统参数

### hitrace.para

**文件位置**: `config/hitrace.para`

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `hitrace.support_executable_file` | bool | true | 支持可执行文件 |
| `hitrace.snapshot.tracebuffer_size` | int | 0 | 缓冲区大小 |
| `hitrace.snapshot.file_limit` | int | 0 | 文件限制 |
| `hitrace.record.file_limit` | int | 0 | 记录限制 |

### hitrace.para.dac

**文件位置**: `config/hitrace.para.dac`

DAC (Discretionary Access Control) 相关参数配置。

---

## 追踪标志位

### HiTraceFlag

**头文件**: `hitrace/hitracechainc.h`

| 标志位 | 值 | 用途 |
|--------|-----|------|
| `HITRACE_FLAG_DEFAULT` | 0x00 | 默认行为 |
| `HITRACE_FLAG_INCLUDE_ASYNC` | 0x01 | 追踪异步调用 |
| `HITRACE_FLAG_DONOT_CREATE_SPAN` | 0x02 | 不创建 Span |
| `HITRACE_FLAG_TP_INFO` | 0x04 | 输出追踪点信息 |
| `HITRACE_FLAG_NO_BE_INFO` | 0x08 | 不输出开始/结束信息 |
| `HITRACE_FLAG_DONOT_ENABLE_LOG` | 0x10 | 不关联日志 |
| `HITRACE_FLAG_FAULT_TRIGGER` | 0x20 | 故障触发追踪 |
| `HITRACE_FLAG_D2D_TP_INFO` | 0x40 | 跨设备追踪点信息 |

**组合使用示例**:
```c
// 异步追踪 + 输出追踪点
unsigned int flags = HITRACE_FLAG_INCLUDE_ASYNC | HITRACE_FLAG_TP_INFO;
```

---

## 通信模式

### HiTraceCommunicationMode

**头文件**: `hitrace/hitracechainc.h`

| 模式 | 值 | 用途 |
|------|-----|------|
| `HITRACE_CM_DEFAULT` | 0 | 默认模式 |
| `HITRACE_CM_THREAD` | 1 | 线程内通信 |
| `HITRACE_CM_PROCESS` | 2 | 进程间通信 (IPC) |
| `HITRACE_CM_DEVICE` | 3 | 设备间通信 |

---

## 追踪点类型

### HiTraceTracepointType

**头文件**: `hitrace/hitracechainc.h`

| 类型 | 值 | 用途 |
|------|-----|------|
| `HITRACE_TP_CS` | 0 | 客户端发送 (Client Send) |
| `HITRACE_TP_CR` | 1 | 客户端接收 (Client Receive) |
| `HITRACE_TP_SS` | 2 | 服务端发送 (Server Send) |
| `HITRACE_TP_SR` | 3 | 服务端接收 (Server Receive) |
| `HITRACE_TP_GENERAL` | 4 | 通用信息 |

---

## HiTraceMeter 配置

### HiTraceOutputLevel

**头文件**: `hitrace_meter.h`

| 级别 | 值 | 用途 |
|------|-----|------|
| `DEBUG` | 0 | 调试级别追踪 |
| `INFO` | 1 | 信息级别追踪 |
| `CRITICAL` | 2 | 关键级别追踪 |
| `COMMERCIAL` | 3 | 商用级别追踪 |
| `MAX` | 4 | 最大级别追踪 |

---

## 日志配置

### LOG_DOMAIN

**值**: `0xD002D33`

**用途**: HiTrace 模块的 HiLog 日志域

### LOG_TAG

| 模块 | 值 |
|------|-----|
| hiTraceChain | `"HitraceChainNapi"` |
| hiTraceMeter | `"HitraceMeterNapi"` |
| bytrace | `"HITRACE_METER_JS"` |

---

## Trace Buffer 配置

### 默认大小

| 组件 | 默认值 | 可配置 |
|------|--------|--------|
| 快照缓冲区 | 0 (禁用) | `hitrace_snapshot_tracebuffer_size` |
| 快照文件 | 0 (禁用) | `hitrace_snapshot_file_limit` |
| 记录文件 | 0 (禁用) | `hitrace_record_file_limit` |

---

## 编译时配置

### PGO 优化

```gn
hitrace_feature_enable_pgo = true
hitrace_feature_pgo_path = "/path/to/profdata"
```

**条件**: 仅在 `is_ohos && is_clang && enhanced_opt` 时可用

---

## 运行时配置

### 环境变量

| 变量 | 说明 |
|------|------|
| `HITRACE_DEBUG` | 启用调试模式 |

---

## 缓冲区限制

### String 缓冲区

| 用途 | 大小 | 位置 |
|------|------|------|
| 名称字符串 | 64 字节 | `napi_hitrace_js.cpp:34` |
| 追踪名称 | 1024 字节 | `napi_hitrace_meter.cpp:80` |

### TraceId 序列化

| 属性 | 大小 |
|------|------|
| HiTraceIdStruct | 16 字节 |
| ChainId | 8 字节 |
| SpanId | 8 字节 |

---

## Syscap 配置

### SystemCapability

**bundle.json 配置**:
```json
{
  "syscap": [
    "SystemCapability.HiviewDFX.HiTrace"
  ]
}
```
