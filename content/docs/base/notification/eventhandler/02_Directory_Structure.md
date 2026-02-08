# 目录结构与模块职责

## 目的

说明 EventHandler 项目的代码组织结构，明确各目录和模块的职责边界。

## 适用范围

- 路径：`base/notification/eventhandler`
- 排除：test/、tests/、unittest/、unit_test/、fuzz/、fuzztest/ 等测试目录

## 目录树（非测试）

```
base/notification/eventhandler
├── interfaces/                    # 接口定义目录
│   ├── inner_api/            # 内部 C++ 接口（stable API）
│   │   ├── event_handler.h
│   │   ├── event_runner.h
│   │   ├── inner_event.h
│   │   ├── event_queue.h
│   │   ├── file_descriptor_listener.h
│   │   ├── lock_base.h
│   │   ├── native_implement_eventhandler.h
│   │   ├── event_handler_errors.h
│   │   ├── event_logger.h
│   │   └── event_inner_logger.h
│   └── kits/                 # 外部接口目录
│       └── native/          # Native C 接口
│           └── native_interface_eventhandler.h
├── frameworks/                    # 框架实现目录
│   ├── eventhandler/           # 核心事件处理实现
│   │   ├── include/            # 内部头文件
│   │   │   ├── io_waiter.h
│   │   │   ├── epoll_io_waiter.h
│   │   │   ├── deamon_io_waiter.h
│   │   │   ├── none_io_waiter.h
│   │   │   ├── priority_inheritance_lock.h
│   │   │   ├── ffrt_descriptor_listener.h
│   │   │   ├── event_inner_runner.h
│   │   │   ├── thread_local_data.h
│   │   │   ├── frame_report_sched.h
│   │   │   ├── async_stack_adapter.h
│   │   │   └── event_handler_utils.h
│   │   ├── src/               # 核心实现源文件
│   │   │   ├── event_runner.cpp
│   │   │   ├── event_queue.cpp
│   │   │   ├── event_queue_base.cpp
│   │   │   ├── event_handler.cpp
│   │   │   ├── inner_event.cpp
│   │   │   ├── epoll_io_waiter.cpp
│   │   │   ├── deamon_io_waiter.cpp
│   │   │   ├── none_io_waiter.cpp
│   │   │   ├── ffrt_descriptor_listener.cpp
│   │   │   ├── file_descriptor_listener.cpp
│   │   │   ├── frame_report_sched.cpp
│   │   │   ├── async_stack_adapter.cpp
│   │   │   ├── native_implement_eventhandler.cpp
│   │   │   └── event_queue_ffrt.cpp (条件编译)
│   │   ├── BUILD.gn            # libeventhandler 构建目标
│   │   ├── inner_api_sources.gni # 源文件列表
│   │   └── test/               # 单元测试
│   ├── napi/                   # N-API (Node.js API) 实现
│   │   ├── include/
│   │   │   ├── events_emitter.h
│   │   │   └── interops.h
│   │   ├── src/
│   │   │   ├── init.cpp        # 模块注册
│   │   │   ├── events_emitter.cpp  # N-API 实现
│   │   │   └── interops.cpp   # 增强 API 注册
│   │   ├── libemitter.map      # 符号版本控制
│   │   └── BUILD.gn
│   ├── emitter/                 # Emitter 模块（跨运行时）
│   │   ├── base/              # 基础组件
│   │   │   ├── include/
│   │   │   │   ├── serialize.h
│   │   │   │   ├── composite_event.h
│   │   │   │   ├── async_callback_manager.h
│   │   │   │   ├── napi_async_callback_manager.h
│   │   │   │   ├── ani_deserialize.h
│   │   │   │   └── ani_async_callback_manager.h
│   │   │   └── src/
│   │   │       ├── async_callback_manager.cpp
│   │   │       ├── napi_async_callback_manager.cpp
│   │   │       ├── ani_deserialize.cpp
│   │   │       └── ani_async_callback_manager.cpp
│   │   ├── napi/              # N-API 实现
│   │   │   ├── include/
│   │   │   │   ├── napi_emitter.h
│   │   │   │   └── napi_serialize.h
│   │   │   └── src/
│   │   │       ├── napi_emitter.cpp
│   │   │       └── napi_serialize.cpp
│   │   ├── ani/               # ANI (Ark Native Interface) 实现
│   │   │   ├── include/
│   │   │   │   ├── ani_emitter.h
│   │   │   │   └── ani_serialize.h
│   │   │   └── src/
│   │   │       ├── ani_emitter.cpp
│   │   │       └── ani_serialize.cpp
│   │   ├── ani/ets/           # ArkTS 类型定义
│   │   │   └── @ohos.events.emitter.ets
│   │   └── BUILD.gn
│   ├── native/                  # Native C++ 接口实现
│   │   └── src/
│   │       └── native_interface_eventhandler.cpp
│   │   └── BUILD.gn
│   ├── cj/                      # CJ (Cangjie) FFI 接口
│   │   ├── include/
│   │   │   └── (空)
│   │   ├── src/
│   │   │   ├── emitter.h
│   │   │   ├── emitter.cpp
│   │   │   ├── emitter_ffi.h
│   │   │   ├── emitter_ffi.cpp
│   │   │   ├── event_handler_impl.h
│   │   │   ├── event_handler_impl.cpp
│   │   │   ├── emitter_common.h
│   │   │   └── emitter_mock.cpp
│   │   └── BUILD.gn
│   └── BUILD.gn                # 总构建入口
├── eventhandler.gni           # 全局 GN 配置
├── bundle.json                # 部件定义
├── OAT.xml                  # 开源协议文件
├── figures/                  # 架构图
└── wiki/                     # 文档目录
```

## 模块职责

### interfaces/inner_api/ - 内部 C++ 接口

**职责**：定义 EventHandler 内部使用的稳定 C++ API 供其他模块调用。

**主要文件**：

| 文件 | 职责 |
|------|------|
| `event_handler.h` | EventHandler 类定义，发送和处理事件的核心接口 |
| `event_runner.h` | EventRunner 类定义，消息队列循环分发器接口 |
| `inner_event.h` | InnerEvent 类定义，事件实体，支持优先级和延迟 |
| `event_queue.h` | EventQueue 类定义，线程消息队列接口 |
| `file_descriptor_listener.h` | 文件描述符监听器接口 |
| `lock_base.h` | 锁基类定义 |
| `event_handler_errors.h` | 错误码定义 |
| `event_logger.h` | 日志接口定义 |

**代码证据**：
- 目录路径：`interfaces/inner_api/`
- 文件数量：11 个头文件

### interfaces/kits/native/ - 外部 Native 接口

**职责**：定义对外暴露的 Native C 接口，供非 ArkTS 环境使用。

**主要文件**：
- `native_interface_eventhandler.h` - EventRunner Native 接口函数

**代码证据**：
- 文件路径：`interfaces/kits/native/native_interface_eventhandler.h:37`
- 导出函数：`GetEventRunnerNativeObjForThread()`, `CreateEventRunnerNativeObj()`, `EventRunnerRun()`, `EventRunnerStop()`, `EventRunnerAddFileDescriptorListener()`, `EventRunnerRemoveFileDescriptorListener()`

### frameworks/eventhandler/ - 核心实现

**职责**：实现 EventHandler 核心功能，包括事件循环、队列管理、I/O 等待等。

**子模块**：

1. **事件循环管理**
   - `event_runner.cpp` - EventRunner 实现，线程创建和事件分发
   - `event_queue.cpp` - EventQueue 实现，优先级队列
   - `event_queue_base.cpp` - 队列基类实现

2. **I/O 等待**
   - `epoll_io_waiter.cpp` - 基于 Linux epoll 的 I/O 等待
   - `deamon_io_waiter.cpp` - 后台 I/O 等待器
   - `none_io_waiter.cpp` - 空实现

3. **FFRT 集成**
   - `event_queue_ffrt.cpp` - FFRT 队列适配（条件编译）
   - `ffrt_descriptor_listener.cpp` - FFRT 文件描述符监听

4. **文件描述符监听**
   - `file_descriptor_listener.cpp` - 文件描述符监听器实现

5. **其他支持**
   - `event_handler.cpp` - EventHandler 实现
   - `inner_event.cpp` - InnerEvent 实现
   - `async_stack_adapter.cpp` - 异步栈适配器
   - `frame_report_sched.cpp` - 帧报告调度

**代码证据**：
- 源文件：`frameworks/eventhandler/src/` 目录下 12 个 .cpp 文件（不含测试）
- 头文件：`frameworks/eventhandler/include/` 目录下 12 个 .h 文件

### frameworks/napi/ - N-API 实现

**职责**：为 Node.js 环境提供 Emitter N-API 绑定。

**主要文件**：

| 文件 | 职责 |
|------|------|
| `init.cpp` | N-API 模块注册，定义 `events.emitter` 模块 |
| `events_emitter.cpp` | N-API 方法实现（on/once/off/emit/getListenerCount） |
| `interops.cpp` | 增强功能注册（跨运行时支持） |

**代码证据**：
- 模块名：`"events.emitter"` （frameworks/napi/src/init.cpp:34）
- 注册函数：`napi_module_register(&_module)` （frameworks/napi/src/init.cpp:41）

### frameworks/emitter/ - Emitter 模块

**职责**：实现跨运行时的 Emitter 功能，支持 N-API 和 ANI（Ark Native Interface）。

**子模块**：

1. **base/** - 基础组件
   - 序列化/反序列化支持
   - 异步回调管理器（N-API 和 ANI 版本）

2. **napi/** - N-API 实现
   - `napi_emitter.cpp` - N-API 版本的 Emitter 方法
   - `napi_serialize.cpp` - N-API 序列化

3. **ani/** - ANI 实现
   - `ani_emitter.cpp` - ANI 版本的 Emitter 方法
   - `ani_serialize.cpp` - ANI 序列化
   - `ani/ets/@ohos.events.emitter.ets` - ArkTS 类型定义

**代码证据**：
- ANI 源文件：`@ohos.events.emitter.ets`
- N-API 源文件：`napi_emitter.cpp`, `napi_serialize.cpp`

### frameworks/native/ - Native 接口实现

**职责**：实现对外 Native C 接口函数。

**主要文件**：
- `native_interface_eventhandler.cpp` - Native 接口函数实现

**代码证据**：
- 实现文件：`frameworks/native/src/native_interface_eventhandler.cpp`

### frameworks/cj/ - CJ (Cangjie) FFI 接口

**职责**：为 Cangjie 语言提供 FFI（Foreign Function Interface）绑定。

**主要文件**：

| 文件 | 职责 |
|------|------|
| `emitter.h`, `emitter.cpp` | CJ Emitter 核心类实现 |
| `emitter_ffi.h`, `emitter_ffi.cpp` | CJ FFI 导出函数 |
| `event_handler_impl.h`, `event_handler_impl.cpp` | CJ EventHandler 实现 |
| `emitter_common.h` | CJ 通用数据结构 |
| `emitter_mock.cpp` | SDK 构建时的 Mock 实现 |

**代码证据**：
- 库名称：`cj_emitter_ffi` （frameworks/cj/BUILD.gn:17）
- 条件编译：`!ohos_indep_compiler_enable && !build_ohos_sdk`

## 相关跳转

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [架构说明](03_Architecture.md) - 组件设计和数据流
- [N-API 接口](04_NAPI_API.md) - JS API 详细文档
