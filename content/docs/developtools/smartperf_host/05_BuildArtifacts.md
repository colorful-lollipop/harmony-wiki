# SmartPerf 编译产物

## 概述

SmartPerf 编译后会生成多种产物，包括设备端可执行文件、HAP 应用包、Host 端工具和测试二进制文件。

**产物路径配置来源**: `bundle.json`

```json
{
  "rom": "188KB",
  "ram": "2000KB"
}
```

## 设备端产物

### SP_daemon

**类型**: 可执行文件 (ELF)

**构建命令**:
```bash
# 完整构建
hb build -f

# 仅构建 smartperf_host
hb build -f --parts smartperf_host
```

**产物路径**:
```
out/[device_type]/libs/
└── libsmartperf_daemon.z.so          # SP_daemon 可执行文件
```

**产物大小**: ~188KB (ROM), ~2000KB (RAM)

**运行时加载关系**:
```
SP_daemon (ELF)
├── libhilog.z.so                    # 日志
├── libhisysevent.z.so              # HiSysEvent
├── libipc_core.z.so                # IPC
├── libsamgr_proxy.z.so             # SAMgr
├── libdm.z.so / libwm.z.so         # 窗口管理
├── librender_service_*.z.so         # 图形渲染
├── libbegetutil.z.so               # init 工具
└── libimage_native.z.so            # 图片处理
```

**启动命令**:
```bash
# 交互模式
./libsmartperf_daemon.z.so

# 参数模式
./libsmartperf_daemon.z.so -c "start:::cpu,fps,ram"
```

### SmartPerf HAP

**类型**: Harmony Ability Package (`.hap`)

**构建命令**:
```bash
# 构建 HAP
hb build -f --parts smartperf_host --js-stacktrace
```

**产物路径**:
```
out/[device_type]/packages/phone/products/
└── com.ohos.gameperceptio/
    └── assets/
        └── entry/
            └── default/
                └── [hash].hap     # SmartPerf HAP
```

**模块配置** (`module.json`):
```json
{
  "module": {
    "name": "entry",
    "type": "entry",
    "abilities": [
      {
        "name": "MainAbility",
        "type": "page"
      }
    ],
    "permissions": [
      "ohos.permission.INTERNET",
      "ohos.permission.SYSTEM_FLOAT_WINDOW"
    ]
  }
}
```

### Inner Kit 头文件

**类型**: C++ 头文件

**头文件路径**:
```
smartperf_device/device_command/interface/
├── GameServicePlugin.h
├── GameEventCallback.h
└── GpuCounterCallback.h
```

**使用方式**:
```cpp
#include "GameServicePlugin.h"
#include "GpuCounterCallback.h"
```

## Host 端产物

### trace_streamer

**类型**: 可执行文件 (ELF)

**构建命令**:
```bash
# 独立编译
cd smartperf_host/trace_streamer
gn gen out/[config]
ninja -C out/[config]

# WASM 编译
gn gen out/wasm --args="use_wasm=true"
ninja -C out/wasm
```

**产物路径**:
```
out/[config]/
└── trace_streamer              # 可执行文件
```

**产物大小**: ~10MB (包含 SQLite)

**运行时依赖**:
```
trace_streamer
├── libsqlite3.z.so              # SQLite3
└── [proto buffers]              # Protobuf 库
```

### trace_streamer.wasm

**类型**: WebAssembly 模块

**构建命令**:
```bash
# WASM 构建
gn gen out/wasm --args="use_wasm=true"
ninja -C out/wasm

# 提取 WASM 文件
```

**产物路径**:
```
out/wasm/
├── libtrace_streamer_builtin.wasm    # WASM 模块
└── trace_streamer_builtin.js         # JS 胶水代码
```

**WASM 产物大小**: ~5-8MB

**浏览器加载示例**:
```typescript
const wasmModule = await loadWasmModule('trace_streamer_builtin.wasm');
wasmModule._Initialize(callbackPtr);
```

### trace_streamer SDK

**类型**: C/C++ SDK 头文件和库

**产物路径**:
```
smartperf_host/trace_streamer/sdk/demo_sdk/
├── include/
│   ├── ts_sdk_api.h
│   └── wasm_func.h
├── lib/
│   ├── libtrace_streamer.a
│   └── libtrace_streamer_builtin_wasm.a
└── wasm/
    └── trace_streamer_builtin.wasm
```

**使用示例**:
```cpp
#include "ts_sdk_api.h"

// 初始化 SDK
SDKSetTableName("my_table");
SDKAppendCounterObject("cpu", 100);
SDKAppendCounter(100, 1000);
```

## 测试产物

### 单元测试

**类型**: 可执行文件

**构建命令**:
```bash
gn gen out/test --args="is_test=true"
ninja -C out/test
```

**产物路径**:
```
out/test/
└── smartperf_host/
    └── smartperf_device/
        └── sp_daemon_ut            # 单元测试可执行文件
```

**运行测试**:
```bash
./out/test/smartperf_host/smartperf_device/sp_daemon_ut
```

### 模糊测试

**类型**: 模糊测试可执行文件

**构建命令**:
```bash
gn gen out/fuzz --args="is_fuzz=true"
ninja -C out/fuzz
```

**产物路径**:
```
out/fuzz/
└── hiprofiler/
    └── hiprofiler
        └── SpDaemonFuzzTest       # 模糊测试可执行文件
```

**模糊测试配置**: `smartperf_device/device_command/test/fuzztest/spdaemon_fuzzer/`

## 产物清单汇总

| 产物名称 | 类型 | 路径模式 | 用途 |
|----------|------|----------|------|
| `libsmartperf_daemon.z.so` | 可执行文件 | `out/*/libs/` | SP_daemon 命令行工具 |
| `SmartPerf.hap` | HAP 包 | `out/*/packages/*/` | device_ui 悬浮窗 |
| `trace_streamer` | 可执行文件 | `out/*/` | Host 端 trace 解析 |
| `libtrace_streamer_builtin.wasm` | WASM 模块 | `out/wasm/` | 浏览器端 trace 解析 |
| `sp_daemon_ut` | 可执行文件 | `out/test/` | 单元测试 |
| `SpDaemonFuzzTest` | 可执行文件 | `out/fuzz/` | 模糊测试 |

## 产物安装路径

### 设备端

| 产物 | 安装路径 | 权限 |
|------|----------|------|
| SP_daemon | `/system/bin/sp_daemon` | 可执行 |
| SmartPerf HAP | `/data/app/` | 用户应用 |
| 配置文件 | `/etc/smartperf/` | 可读写 |

### Host 端

| 产物 | 安装路径 | 用途 |
|------|----------|------|
| trace_streamer | `$OHOS_SDK/toolchain/` | SDK 工具 |
| WASM | `$OHOS_SDK/wasm/` | IDE Web 集成 |

## 运行时加载关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                      SP_daemon 运行时依赖                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SP_daemon (libsmartperf_daemon.z.so)                           │
│       │                                                          │
│       ├──┬── libhilog.z.so (日志)                               │
│       │                                                          │
│       ├──┬── libipc_core.z.so (IPC)                             │
│       │                                                          │
│       ├──┬── libsamgr_proxy.z.so (SAMgr)                        │
│       │                                                          │
│       ├──┬── libdm.z.so / libwm.z.so (窗口管理)                  │
│       │                                                          │
│       ├──┬── librender_service_*.z.so (图形渲染)                 │
│       │                                                          │
│       ├──┬── libhisysevent.z.so (HiSysEvent)                    │
│       │                                                          │
│       ├──┬── libbegetutil.z.so (init)                          │
│       │                                                          │
│       ├──┬── libimage_native.z.so (图片)                        │
│       │                                                          │
│       └──┴── libsec_static.z.so (安全库)                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    trace_streamer 运行时依赖                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  trace_streamer                                                  │
│       │                                                          │
│       ├──┬── libsqlite3.z.so (SQLite)                          │
│       │                                                          │
│       ├──┬── libproto*.z.so (Protobuf)                          │
│       │                                                          │
│       └──┴── libsec_static.z.so (安全库)                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    trace_streamer.wasm 运行时依赖                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  浏览器                                                           │
│       │                                                          │
│       ├──┬── trace_streamer_builtin.wasm (WASM 模块)            │
│       │                                                          │
│       ├──┬── trace_streamer_builtin.js (JS 胶水)                 │
│       │                                                          │
│       └──┴── SQLite WASM 库                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 相关文档

- [项目概览](00_Overview.md)
- [系统架构](01_Architecture.md)
- [GN 构建配置](04_GNBuild.md)
- [安全风险评审](06_Security.md)
