# AI Engine 构建系统

## 构建系统概述

AI Engine 使用 OpenHarmony 的 **build_lite** 构建系统：

| 构建系统 | 用途 |
|----------|------|
| **GN (Generate Ninja)** | 主要构建系统，用于正式编译 |
| **CMake** | 辅助构建，用于 IDE 开发支持 |

**证据**：`services/BUILD.gn` 根构建定义

---

## 构建文件清单

### BUILD.gn 文件（39 个）

| 层级 | 文件 | 目标类型 | 说明 |
|------|------|----------|------|
| 根 | `services/BUILD.gn` | lite_component | 根组件 "ai" |
| 服务端 | `services/server/BUILD.gn` | executable | ai_server 可执行文件 |
| 服务端 | `services/server/server_executor/BUILD.gn` | source_set | 执行器 |
| 服务端 | `services/server/plugin_manager/BUILD.gn` | source_set + action | 插件管理器 |
| 服务端 | `services/server/communication_adapter/BUILD.gn` | static_library | 通信适配 |
| 服务端 | `services/server/plugin/BUILD.gn` | lite_component | 插件组件 |
| 服务端 | `services/server/plugin/asr/keyword_spotting/BUILD.gn` | lite_library | KWS 插件 |
| 服务端 | `services/server/plugin/cv/image_classification/BUILD.gn` | lite_library | IC 插件 |
| 客户端 | `services/client/BUILD.gn` | shared_library | libai_client.so |
| 客户端 | `services/client/client_executor/BUILD.gn` | source_set | 客户端执行器 |
| 客户端 | `services/client/communication_adapter/BUILD.gn` | source_set | 通信适配 |
| 客户端 | `services/client/algorithm_sdk/BUILD.gn` | lite_component | SDK 组件 |
| 公共 | `services/common/platform/*/BUILD.gn` | source_set | 平台模块 |
| 公共 | `services/common/utils/*/BUILD.gn` | source_set | 工具模块 |

**证据**：`services/BUILD.gn` 根组件定义

### CMakeLists.txt 文件（5 个）

| 文件 | 用途 |
|------|------|
| `CMakeLists.txt` | 根构建入口，定义 engine 可执行目标 |
| `services/server/CMakeLists.txt` | 服务端源文件 |
| `services/client/CMakeLists.txt` | 客户端源文件 |
| `services/common/CMakeLists.txt` | 公共模块源文件 |
| `test/CMakeLists.txt` | 测试源文件 |

---

## 关键 Targets

### 根组件

```gn
# services/BUILD.gn
lite_component("ai") {
  features = [
    "client:client",
    "server:server",
  ]
}
```

**目标**：`ai` (lite_component)

### 服务端可执行文件

```gn
# services/server/BUILD.gn
lite_component("ai_server") {
  target_type = "executable"
  features = [
    "communication_adapter:ai_communication_adapter",
    "plugin_manager",
    "server_executor",
  ]
  cflags = [ "-fPIC" ]
  ldflags = [
    "-Wl,-Map=server.map",
    "-lstdc++",
    "-Wl,--whole-archive",
    "libs/libai_communication_adapter.a",
    "-Wl,--no-whole-archive",
    "-ldl",
    "-pthread",
  ]
}
```

**目标**：`ai_server` (executable)

### 客户端共享库

```gn
# services/client/BUILD.gn
shared_library("ai_client") {
  features = [
    ":client",
    ":ai_communication_adapter",
    ":algorithm_sdk",
  ]
}
```

**目标**：`ai_client` (shared_library) → `libai_client.so`

### 插件

```gn
# services/server/plugin/asr/keyword_spotting/BUILD.gn
lite_library("asr_keyword_spotting") {
  sources = [ "kws_plugin.cpp" ]
  defines = [ "USE_NNIE" ]
  deps = [
    "//foundation/ai/ai_engine/services/common/platform/os_wrapper/ipc:aie_ipc",
    "//vendor/hispark_taurus/hardware/nnie_adapter:nnie_adapter",
  ]
}
```

**目标**：`asr_keyword_spotting` (shared_library) → `libasr_keyword_spotting.so`

---

## 依赖配置

### 系统依赖

| 组件 | 用途 | 证据 |
|------|------|------|
| hilog_lite | 日志输出 | `services/server/BUILD.gn:33` |
| samgr_lite | 系统能力管理 | `services/server/BUILD.gn:40` |
| ipc | 进程间通信 | bundle.json |
| utils_base | 基础工具 | bundle.json |

### 第三方依赖

| 组件 | 用途 | 证据 |
|------|------|------|
| bounds_checking_function | 安全函数 | bundle.json |

### 内部依赖

```
ai_server
├── ai_communication_adapter
│   └── samgr
├── plugin_manager
├── server_executor
│   └── data_channel
├── plugin (lite_component)
│   ├── cv_image_classification (USE_NNIE)
│   └── asr_keyword_spotting (USE_NNIE)
└── platform 模块
    ├── dlOperation
    ├── event
    ├── lock
    ├── aie_ipc
    ├── semaphore
    └── threadpool
```

**证据**：`services/server/BUILD.gn:32-41`

---

## 配置宏

### 编译选项

| 宏 | 位置 | 用途 |
|---|------|------|
| `USE_NNIE` | CV/ASR 插件 BUILD.gn | 启用海思 NNIE 推理引擎 |
| `-fPIC` | 22+ BUILD.gn | 位置无关代码 |
| `-fexceptions` | test BUILD.gn | 启用 C++ 异常 |

### GNI 配置

| 变量 | 文件 | 说明 |
|------|------|------|
| `activate_plugin_list` | `ai_plugin_config.gni` | 激活的插件列表 |

### 链接选项

| 选项 | 目标 | 用途 |
|------|------|------|
| `-lstdc++` | ai_server | C++ 标准库 |
| `-ldl` | ai_server | 动态加载 |
| `-lpthread` | ai_server | POSIX 线程 |
| `-lnnie` | ASR 插件 | NNIE 库 |
| `-lnnie_adapter` | CV/ASR 插件 | NNIE 适配层 |

**证据**：`services/server/BUILD.gn:23-31`

---

## 板级配置

构建系统使用 `board_name` 变量进行条件编译：

| 开发板 | 启用特性 |
|--------|----------|
| `hispark_taurus` | CV 图像分类、ASR 关键词检测、NNIE 支持 |
| 其他 | 仅基础框架（插件禁用） |

---

## 编译命令

### HB 编译（推荐）

```bash
# 编译整个项目
hb build -f

# 仅编译 AI Engine
hb build ai_engine
```

### GN 编译

```bash
# 设置编译路径
hb set -root <project_root>

# 设置产品
hb set -p

# 编译
hb build -f
```

**证据**：`README.md:56-75`

---

## 构建产物

| 组件 | 产物 | 类型 |
|------|------|------|
| 服务端 | `ai_server` | 可执行文件 |
| 客户端 | `libai_client.so` | 共享库 |
| CV 插件 | `libcv_image_classification.so` | 共享库 |
| ASR 插件 | `libasr_keyword_spotting.so` | 共享库 |
| 配置文件 | `ai_engine_plugin.ini` | 配置文件 |

**证据**：`services/server/BUILD.gn:15-51`
