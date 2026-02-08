# 目录结构与代码地图

## 3.1 顶层目录概览

### 项目根目录结构

```
hisysevent/
├── adapter/                    # 适配层（平台适配、IPC 通信）
│   └── native/idl/            # IDL 接口定义与 IPC 适配实现
├── frameworks/                 # 框架层（核心业务逻辑）
│   └── native/               # C++ 框架代码
├── interfaces/                # 接口层（对外编程接口）
│   ├── native/innerkits/     # Native C++ API（InnerKits）
│   ├── js/kits/napi/         # JavaScript N-API 接口
│   ├── rust/innerkits/       # Rust FFI 接口
│   └── ets/ani/              # ArkTS ANI 接口
├── test/                      # 测试代码（单元测试、模糊测试）
├── wiki/                      # 文档目录
├── bundle.json                # 组件配置文件
├── Cargo.toml                 # Rust 工作区配置
└── README.md                  # 项目说明文档
```

### 目录职责说明

| 目录 | 职责 | 文件数量 | 核心组件 |
|------|------|----------|----------|
| **adapter/** | 平台适配层，处理 IPC 通信和系统服务交互 | ~20 | IDL 代理/存根、SysEventService |
| **frameworks/** | 框架核心层，实现事件处理核心逻辑 | ~25 | HiSysEventTool、JSON Decorator |
| **interfaces/** | 接口层，提供多语言编程接口 | ~150 | N-API、Rust FFI、ANI、C++ API |
| **test/** | 测试代码，包含单元测试和模糊测试 | ~50 | UnitTest、FuzzTest |

---

## 3.2 适配层详解

### adapter/native/idl 目录结构

```
adapter/native/idl/
├── ISysEventService.idl              # 系统事件服务 IPC 接口定义
├── ISysEventCallback.idl              # 回调接口定义
├── BUILD.gn                           # 构建配置
├── include/                           # 头文件目录
│   ├── iquery_sys_event_callback.h   # 查询回调接口头文件
│   ├── query_sys_event_callback_stub.h # 回调存根头文件
│   ├── hisysevent_delegate.h         # 适配器头文件
│   ├── hisysevent_listener_proxy.h    # 监听代理头文件
│   ├── hisysevent_query_proxy.h       # 查询代理头文件
│   ├── ret_code.h                     # 返回码定义
│   ├── sys_event_rule.h              # 事件规则定义
│   ├── query_argument.h               # 查询参数定义
│   ├── ash_mem_utils.h                # 共享内存工具
│   ├── file_util.h                   # 文件工具
│   └── parcelable_vector_rw.h         # 可序列化向量读写
└── src/                              # 实现文件目录
    ├── query_sys_event_callback_stub.cpp
    ├── hisysevent_delegate.cpp
    ├── hisysevent_listener_proxy.cpp
    ├── hisysevent_query_proxy.cpp
    ├── query_argument.cpp
    ├── sys_event_rule.cpp
    ├── sys_event_query_rule.cpp
    ├── ash_mem_utils.cpp
    └── file_util.cpp
```

### 核心文件定位

| 文件 | 功能 | 关键符号 |
|------|------|----------|
| `ISysEventService.idl` | IPC 服务接口定义 | `AddListener`, `Query`, `RemoveListener` |
| `hisysevent_delegate.cpp` | IPC 适配器实现 | `HisyseventDelegate` |
| `hisysevent_query_proxy.cpp` | 查询代理实现 | `HisyseventQueryProxy` |
| `ret_code.h` | 错误码定义 | `ERR_*` 常量 |

---

## 3.3 框架层详解

### frameworks/native 目录结构

```
frameworks/native/
├── BUILD.gn                           # 构建配置
├── main.cpp                           # CLI 工具入口
├── hisysevent_tool.cpp                # 核心工具实现
├── hisysevent_tool_listener.cpp       # 监听器实现
├── hisysevent_tool_query.cpp          # 查询工具实现
├── hisysevent_json_decorator.cpp      # JSON 装饰器实现
├── json_flatten_parser.cpp             # JSON 扁平化解析器
├── c_wrapper/                         # C Wrapper（C 与 Rust 桥接）
│   ├── BUILD.gn
│   ├── include/
│   │   ├── hisysevent_c_wrapper.h     # C Wrapper 头文件
│   │   ├── hisysevent_rust_manager.h  # Rust 管理器桥接头文件
│   │   ├── hisysevent_rust_querier.h  # Rust 查询器桥接头文件
│   │   └── hisysevent_rust_listener.h # Rust 监听器桥接头文件
│   └── source/
│       ├── hisysevent_c_wrapper.cpp
│       ├── hisysevent_rust_manager.cpp
│       ├── hisysevent_rust_querier.cpp
│       └── hisysevent_rust_listener.cpp
├── include/                           # 头文件目录
│   ├── hisysevent_tool.h
│   ├── hisysevent_tool_listener.h
│   ├── hisysevent_tool_query.h
│   ├── hisysevent_json_decorator.h
│   └── json_flatten_parser.h
├── util/                              # 工具类
│   ├── BUILD.gn
│   ├── string_util.cpp
│   └── string_util.h
└── test/                              # 框架测试
    └── unittest/
        └── common/
            ├── BUILD.gn
            ├── hisysevent_c_wrapper_test.cpp
            └── hisysevent_tool_unit_test.cpp
```

### 核心文件定位

| 文件 | 功能 | 关键符号 |
|------|------|----------|
| `hisysevent_tool.cpp` | 核心事件工具类 | `HiSysEventTool::Write()`, `HiSysEventTool::Query()` |
| `hisysevent_json_decorator.cpp` | JSON 格式转换 | `HiSysEventJsonDecorator::Decorate()` |
| `c_wrapper/hisysevent_c_wrapper.cpp` | C Wrapper 入口 | `OH_HiSysEvent_*` |
| `hisysevent_tool_query.cpp` | 事件查询实现 | `HiSysEventToolQuery::Query()` |

---

## 3.4 接口层详解

### interfaces/native/innerkits/hisysevent 目录

```
interfaces/native/innerkits/hisysevent/
├── BUILD.gn                           # 构建配置
├── hisysevent.cpp                     # 核心实现
├── hisysevent_c.cpp                  # C API 实现
├── event_socket_factory.cpp           # Socket 工厂
├── transport.cpp                      # 传输实现
├── write_controller.cpp               # 写入控制
├── encoded_param.cpp                  # 参数编码
├── raw_data.cpp                       # 原始数据处理
├── raw_data_base_def.cpp              # 原始数据基类
├── raw_data_encoder.cpp              # 原始数据编码
├── stringfilter.cpp                   # 字符串过滤
├── libhisysevent.map                  # 导出符号映射
└── include/                           # 头文件
    ├── hisysevent.h                  # 主 API 头文件
    ├── hisysevent_c.h                # C API 头文件
    ├── def.h                          # 常量定义
    ├── encoded_param.h                # 编码参数头文件
    ├── event_socket_factory.h        # Socket 工厂头文件
    ├── raw_data.h                    # 原始数据头文件
    ├── raw_data_base_def.h           # 基类头文件
    ├── raw_data_encoder.h            # 编码器头文件
    ├── stringfilter.h                # 过滤头文件
    └── transport.h                   # 传输头文件
```

### interfaces/native/innerkits/hisysevent_easy 目录

```
interfaces/native/innerkits/hisysevent_easy/
├── BUILD.gn
├── hisysevent_easy.c                 # Easy API 实现
├── easy_event_builder.c               # 事件构建器
├── easy_event_encoder.c               # 事件编码器
├── easy_socket_writer.c               # Socket 写入器
├── easy_util.c                        # 工具函数
└── include/
    ├── hisysevent_easy.h             # Easy API 头文件
    ├── easy_event_builder.h
    ├── easy_event_encoder.h
    ├── easy_socket_writer.h
    └── easy_util.h
```

### interfaces/native/innerkits/hisysevent_manager 目录

```
interfaces/native/innerkits/hisysevent_manager/
├── BUILD.gn
├── hisysevent_manager.cpp            # 管理器实现
├── hisysevent_base_manager.cpp       # 基础管理器
├── hisysevent_listener_c.cpp         # C 监听器
├── hisysevent_query_callback_c.cpp   # C 查询回调
├── hisysevent_record.cpp             # 事件记录
├── hisysevent_record_c.cpp           # C 事件记录
├── hisysevent_record_convertor.cpp   # 记录转换器
├── hisysevent_manager_c.cpp          # C 管理器
├── libhisyseventmanager.map          # 导出符号映射
└── include/                           # 头文件目录
    ├── hisysevent_manager.h         # 主管理器头文件
    ├── hisysevent_base_listener.h
    ├── hisysevent_base_manager.h
    ├── hisysevent_base_query_callback.h
    ├── hisysevent_listener.h
    ├── hisysevent_listener_c.h
    ├── hisysevent_listenning_operate.h
    ├── hisysevent_manager_c.h
    ├── hisysevent_query_callback.h
    ├── hisysevent_query_callback_c.h
    ├── hisysevent_record.h
    ├── hisysevent_record_c.h
    ├── hisysevent_record_convertor.h
    ├── hisysevent_rules.h
    ├── hisysevent_value.h
    └── rule_type.h
```

### interfaces/js/kits/napi 目录

```
interfaces/js/kits/napi/
├── BUILD.gn
├── include/                           # 头文件目录
│   ├── napi_hisysevent_adapter.h     # N-API 适配器
│   ├── napi_hisysevent_init.h        # 初始化
│   ├── napi_hisysevent_listener.h   # 监听器
│   ├── napi_hisysevent_querier.h    # 查询器
│   ├── napi_hisysevent_util.h        # 工具函数
│   ├── js_callback_manager.h         # JS 回调管理
│   ├── napi_callback_context.h        # 回调上下文
│   └── ret_def.h                     # 返回定义
└── src/                              # 实现文件目录
    ├── napi_hisysevent_js.cpp        # N-API 主实现
    ├── napi_hisysevent_adapter.cpp
    ├── napi_hisysevent_init.cpp
    ├── napi_hisysevent_listener.cpp
    ├── napi_hisysevent_querier.cpp
    ├── napi_hisysevent_util.cpp
    └── js_callback_manager.cpp
```

### interfaces/rust/innerkits 目录

```
interfaces/rust/innerkits/
├── Cargo.toml                        # Rust 配置
├── BUILD.gn                          # 构建配置
└── src/                              # Rust 源码
    ├── lib.rs                        # 库入口
    ├── macros.rs                     # 宏定义
    ├── sys_event.rs                  # 事件接口
    ├── sys_event_manager.rs          # 事件管理器
    └── utils.rs                      # 工具函数
```

### interfaces/ets/ani 目录

```
interfaces/ets/ani/
├── BUILD.gn
└── hisysevent/                       # ANI 接口
    ├── BUILD.gn
    ├── ets/
    │   └── @ohos.hiSysEvent.ets     # ArkTS 类型定义
    ├── include/                      # 头文件目录
    │   ├── hisysevent_ani.h
    │   ├── ani_hisysevent_listener.h
    │   ├── ani_hisysevent_querier.h
    │   ├── ani_callback_manager.h
    │   ├── ani_callback_context.h
    │   ├── hisysevent_ani_util.h
    │   └── ret_def.h
    └── src/                         # 实现文件目录
        ├── hisysevent_ani.cpp
        ├── ani_hisysevent_listener.cpp
        ├── ani_hisysevent_querier.cpp
        ├── ani_callback_manager.cpp
        └── hisysevent_ani_util.cpp
```

---

## 3.5 代码导航图

### 功能到文件映射

| 功能 | 对应文件 | 代码行数 |
|------|----------|----------|
| **事件写入** | `hisysevent.cpp` | ~500 |
| **C 事件写入** | `hisysevent_c.cpp` | ~100 |
| **Easy C 写入** | `hisysevent_easy.c` | ~150 |
| **事件传输** | `transport.cpp` | ~300 |
| **速率控制** | `write_controller.cpp` | ~150 |
| **参数编码** | `encoded_param.cpp` | ~200 |
| **N-API 实现** | `napi_hisysevent_js.cpp` | ~450 |
| **Rust 事件** | `sys_event.rs` | ~200 |
| **Rust 管理器** | `sys_event_manager.rs` | ~500 |
| **事件查询** | `hisysevent_tool_query.cpp` | ~200 |
| **JSON 装饰** | `hisysevent_json_decorator.cpp` | ~350 |
| **IPC 适配** | `hisysevent_delegate.cpp` | ~300 |

### 符号查找索引

| 符号名 | 类型 | 定义文件 | 行号 |
|--------|------|----------|------|
| `HiSysEvent::Write()` | 函数 | `hisysevent.h` | 78 |
| `HiSysEvent::Create()` | 函数 | `hisysevent.h` | - |
| `EventType` | 枚举 | `hisysevent.h` | - |
| `HiSysEvent::Domain` | 类 | `hisysevent.h` | 79 |
| `WriteController` | 类 | `write_controller.h` | - |
| `EventSocketFactory` | 类 | `event_socket_factory.h` | - |
| `HiSysEventTool` | 类 | `hisysevent_tool.h` | - |
| `HisyseventDelegate` | 类 | `hisysevent_delegate.h` | - |

### API 调用链

#### 事件写入调用链

```
用户代码
    │
    ▼
HiSysEvent::Write() [hisysevent.cpp:78]
    │
    ├── 权限检查 [hisysevent.cpp:120]
    ├── 参数校验 [hisysevent.cpp:89-118]
    ├── 事件编码 [encoded_param.cpp:45]
    ├── 速率控制 [write_controller.cpp:45]
    └── 发送数据 [transport.cpp:89]
            │
            ▼
        Socket 发送
```

#### N-API 事件写入调用链

```
JS/ArkTS: hiSysEvent.write()
    │
    ▼
napi_hisysevent_js.cpp: HiSysEventWriteNapi()
    │
    ├── 参数解析 [napi_hisysevent_js.cpp:78-120]
    ├── 类型转换 [napi_hisysevent_util.cpp]
    └── 调用 Native API
            │
            ▼
        HiSysEvent::Write()
```

#### 事件查询调用链

```
用户代码: HiSysEventManager::Query()
    │
    ▼
hisysevent_manager.cpp: Query()
    │
    ├── IPC 调用 [hisysevent_query_proxy.cpp]
    │       │
    │       └── Binder IPC → SysEventImpl SA
    │
    └── 回调处理 [hisysevent_query_callback.cpp]
```

---

## 3.6 测试目录说明

### test/ 目录结构

```
test/
├── unittest/                         # 单元测试
│   ├── cpp/                          # C++ 单元测试
│   └── rust/                         # Rust 单元测试
├── moduletest/                       # 模块测试
├── fuzztest/                         # 模糊测试
└── benchmark/                        # 性能测试
```

| 目录 | 用途 | 排除原因 |
|------|------|----------|
| **test/** | 所有测试代码 | 属于测试代码，不属于功能实现 |

---

## 3.7 快速文件定位

### 按任务类型

| 任务类型 | 推荐查看文件 |
|----------|--------------|
| **添加事件写入** | `hisysevent.h`, `hisysevent.cpp` |
| **添加 N-API 接口** | `napi_hisysevent_js.cpp`, `napi_hisysevent_util.cpp` |
| **添加 Rust 接口** | `sys_event.rs`, `sys_event_manager.rs` |
| **修改 IPC 协议** | `ISysEventService.idl`, `hisysevent_delegate.cpp` |
| **修改构建配置** | `BUILD.gn`, `bundle.json` |
| **调试传输问题** | `transport.cpp`, `event_socket_factory.cpp` |
| **性能优化** | `write_controller.cpp`, `encoded_param.cpp` |
| **安全审计** | `hisysevent.cpp`, `hisysevent_delegate.cpp` |

### 按错误类型

| 错误类型 | 排查文件 |
|----------|----------|
| **权限错误** | `hisysevent.cpp` 权限检查相关代码 |
| **参数错误** | `hisysevent.cpp` 参数校验相关代码 |
| **IPC 错误** | `hisysevent_delegate.cpp`, `hisysevent_query_proxy.cpp` |
| **Socket 错误** | `transport.cpp`, `event_socket_factory.cpp` |
| **编码错误** | `encoded_param.cpp`, `raw_data_encoder.cpp` |

---

## 3.8 关键文件摘要

### 核心文件行数统计

| 文件 | 代码行数 | 说明 |
|------|----------|------|
| `sys_event_manager.rs` | ~650 | Rust 事件管理器 |
| `hisysevent_ani_util.cpp` | ~600 | ANI 工具实现 |
| `napi_hisysevent_util.cpp` | ~550 | N-API 工具实现 |
| `hisysevent_ani.cpp` | ~500 | ANI 主实现 |
| `hisysevent.cpp` | ~500 | C++ 主实现 |
| `napi_hisysevent_js.cpp` | ~450 | N-API 主实现 |
| `hisysevent_json_decorator.cpp` | ~350 | JSON 处理 |
| `hisysevent_delegate.cpp` | ~300 | IPC 适配器 |
| `transport.cpp` | ~300 | Socket 传输 |

---

*文档版本：1.0*
*创建时间：2026-02-07*
*最后更新：2026-02-07*
