# 编译配置开关

## 概述

本文档列出 KV Store 相关的编译配置开关（Feature Flags 和编译宏）。

## Feature Flags

### bundle.json 中定义

**路径**：`bundle.json:42-45`

```json
"features": [
  "kv_store_cloud",
  "kv_store_device"
]
```

| 开关 | 描述 | 依赖 |
|-----|------|------|
| `kv_store_cloud` | 启用云同步功能 | `USE_DISTRIBUTEDDB_CLOUD` 宏 |
| `kv_store_device` | 启用设备间同步功能 | `USE_DISTRIBUTEDDB_DEVICE` 宏 |

## 编译宏

### distributeddb 配置

**路径**：`frameworks/libs/distributeddb/BUILD.gn:49-74`

```gn
config("distrdb_config") {
  defines = [
    "_LARGEFILE64_SOURCE",
    "_FILE_OFFSET_BITS=64",
    "SQLITE_HAS_CODEC",
    "SQLITE_ENABLE_JSON1",
    "USING_HILOG_LOGGER",
    "USE_SQLITE_SYMBOLS",
    "USING_DB_JSON_EXTRACT_AUTOMATICALLY",
    "JSONCPP_USE_BUILDER",
    "OMIT_FLATBUFFER",
    "OMIT_MULTI_VER",
    "RELATIONAL_STORE",
    "SQLITE_DISTRIBUTE_RELATIONAL",
    "USE_DFX_ABILITY",
    "SQLITE_ENABLE_DROPTABLE_CALLBACK",
    "OPENSSL_SUPPRESS_DEPRECATED",
  ]

  if (is_debug) {
    defines += [ "TRACE_SQLITE_EXECUTE" ]
  }

  if (kv_store_cloud) {
    defines += [ "USE_DISTRIBUTEDDB_CLOUD" ]
  }

  if (kv_store_device) {
    defines += [ "USE_DISTRIBUTEDDB_DEVICE" ]
  }
}
```

### 宏说明

| 宏 | 描述 | 用途 |
|---|------|------|
| `_LARGEFILE64_SOURCE` | 启用 64 位文件 API | 支持大文件 |
| `_FILE_OFFSET_BITS=64` | 64 位文件偏移 | 大文件支持 |
| `SQLITE_HAS_CODEC` | SQLite 加密扩展 | 数据库加密 |
| `SQLITE_ENABLE_JSON1` | JSON1 扩展 | JSON 查询支持 |
| `USING_HILOG_LOGGER` | 使用 HiLog | 日志输出 |
| `USE_SQLITE_SYMBOLS` | 使用 SQLite 符号 | 链接优化 |
| `USING_DB_JSON_EXTRACT_AUTOMATICALLY` | 自动 JSON 提取 | JSON 查询 |
| `OMIT_FLATBUFFER` | 禁用 FlatBuffer | 减小代码体积 |
| `OMIT_MULTI_VER` | 禁用多版本 | 单版本模式 |
| `RELATIONAL_STORE` | 启用关系存储 | RDB 功能 |
| `SQLITE_DISTRIBUTE_RELATIONAL` | 分布式关系存储 | 跨设备 RDB |
| `USE_DFX_ABILITY` | DFX 能力 | 调试和监控 |
| `SQLITE_ENABLE_DROPTABLE_CALLBACK` | DROP 表回调 | 清理逻辑 |
| `OPENSSL_SUPPRESS_DEPRECATED` | 抑制弃用警告 | 编译兼容性 |
| `TRACE_SQLITE_EXECUTE` | 追踪 SQL 执行 | Debug 模式 |
| `USE_DISTRIBUTEDDB_CLOUD` | 云同步 | 云功能 |
| `USE_DISTRIBUTEDDB_DEVICE` | 设备同步 | 分布式功能 |

## 编译器安全选项

**路径**：`frameworks/libs/distributeddb/BUILD.gn:115-120`

```gn
cflags_cc = [
  "-fvisibility=hidden",
  "-Os",
  "-D_FORTIFY_SOURCE=2",
  "-flto",
]
```

| 选项 | 描述 | 用途 |
|------|------|------|
| `-fvisibility=hidden` | 隐藏符号导出 | 减少攻击面 |
| `-Os` | 优化大小 | 减少代码体积 |
| `-D_FORTIFY_SOURCE=2` | 运行时检查 | 缓冲区溢出防护 |
| `-flto` | 链接时优化 | 性能优化 |

## Sanitizer 配置

**路径**：`frameworks/libs/distributeddb/BUILD.gn:100-106`

```gn
sanitize = {
  ubsan = true
  boundary_sanitize = true
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

| Sanitizer | 描述 | 用途 |
|-----------|------|------|
| `ubsan` | 未定义行为检测 | UB 调试 |
| `boundary_sanitize` | 边界检查 | 内存安全 |
| `cfi` | 控制流完整性 | 防止 ROP 攻击 |
| `cfi_cross_dso` | 跨 DSO CFI | 跨库调用安全 |

## 条件编译

### kv_store.gni

**路径**：`kv_store.gni:26-44`

```gn
declare_args() {
  if (!defined(global_parts_info) || defined(global_parts_info.ability_dmsfwk)) {
    dms_service_enable = true
  } else {
    dms_service_enable = false
  }

  if (!defined(global_parts_info) || defined(global_parts_info.distributedhardware_device_manager)) {
    dm_service_enable = true
  } else {
    dm_service_enable = false
  }

  if (device_company != "qemu") {
    qemu_disable = true
  } else {
    qemu_disable = false
  }
}
```

| 变量 | 描述 | 默认值 |
|-----|------|-------|
| `dms_service_enable` | DMS 服务 | true |
| `dm_service_enable` | 设备管理服务 | true |
| `qemu_disable` | QEMU 禁用 | device_company != "qemu" |

## 性能相关配置

### 日志级别控制

```cpp
// log_print.h
#define LOG_TAG "JsKVManager"

// 日志宏
ZLOGD()  // Debug
ZLOGI()  // Info
ZLOGW()  // Warning
ZLOGE()  // Error
```

### 追踪开关

```cpp
#include "hitrace/hitrace_meter.h"

HiTraceBegin("OperationName", HiTraceFlag::INPUT_PARAM)
// ... 操作代码 ...
HiTraceEnd(traceId);
```

## 相关文档

- [GN 构建指南](05_Build_GN.md)
- [安全风险评审](06_Security_Review.md)
- [架构设计](02_Architecture.md)
