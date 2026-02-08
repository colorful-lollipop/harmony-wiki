# 目录结构

## 顶层目录概览

```
foundation/arkui/napi
├── interfaces/          # 接口定义
├── module_manager/      # 模块管理
├── native_engine/       # NativeEngine 实现
├── scope_manager/       # 作用域管理
├── reference_manager/   # 引用管理
├── callback_scope_manager/  # 回调作用域管理
├── utils/              # 工具类
├── sample/             # 示例代码
└── test/               # 测试代码（不作为业务证据）
```

## 目录职责详解

### interfaces/ - 接口定义

**位置**：`foundation/arkui/napi/interfaces/`

**职责**：定义 N-API 的公共接口和内部 API

```
interfaces/
├── kits/
│   └── napi/           # 对外 N-API 头文件
│       ├── native_api.h     # 主 N-API 接口（NAPI v8）
│       └── common.h         # 公共类型定义
└── inner_api/
    ├── napi/               # 内部 N-API
    │   ├── native_common.h     # 宏定义（NAPI_CALL, DECLARE_NAPI_FUNCTION 等）
    │   ├── native_node_api.h    # 扩展 Node API
    │   └── native_node_hybrid_api.h  # 混合模式 API
    └── cjffi/             # CJFFI（Cangjie FFI）接口
        ├── native/            # 原生 FFI 接口
        │   ├── ffi_remote_data.h
        │   ├── cj_fn_invoker.h
        │   └── cj_lambda.h
        ├── cj_ffi/            # 数据 FFI
        │   ├── cj_common_ffi.h
        │   └── cj_data_ffi.h
        └── ark_interop/       # ArkTS 互操作
            ├── ark_interop_napi.h     # 低级互操作 API
            ├── ark_interop_macro.h    # 导出宏
            ├── ark_interop_scope.h    # 作用域管理
            └── ark_interop_*.h        # 其他互操作接口
```

**关键文件**：

| 文件 | 用途 |
|------|------|
| `kits/napi/native_api.h` | 对外 N-API 接口，NAPI_VERSION=8 |
| `inner_api/napi/native_common.h` | 宏定义：错误处理、属性声明 |
| `inner_api/napi/native_node_api.h` | 扩展 API：sendable、TSFN、序列化 |
| `inner_api/cjffi/ark_interop/ark_interop_napi.h` | ArkTS 互操作 API |

### module_manager/ - 模块管理

**位置**：`foundation/arkui/napi/module_manager/`

**职责**：管理 native 模块的加载、缓存和卸载

**关键文件**：

| 文件 | 用途 |
|------|------|
| `native_module_manager.h` | 模块管理器单例 |
| `module_load_checker.cpp` | 模块加载检查 |
| `module_checker_delegate.cpp` | 模块验证委托 |

**核心数据结构**：

```cpp
// 模块结构体（证据：native_module_manager.h:57-75）
struct NativeModule {
    const char* name = nullptr;           // 模块名
    const char* moduleName = nullptr;      // 模块名（带路径）
    const char* fileName = nullptr;        // 文件名
    const char* systemFilePath = nullptr;  // 系统路径
    RegisterCallback registerCallback = nullptr;  // 注册回调
    GetJSCodeCallback getABCCode = nullptr; // ABC 代码回调
    GetJSCodeCallback getJSCode = nullptr; // JS 代码回调
    int32_t version = 0;
    uint32_t refCount = 0;
    NativeModule* next = nullptr;
    bool moduleLoaded = false;
    bool isAppModule = false;
};
```

### native_engine/ - NativeEngine 实现

**位置**：`foundation/arkui/napi/native_engine/`

**职责**：JS 引擎抽象层，核心运行时实现

**目录结构**：

```
native_engine/
├── native_engine.h          # 抽象基类定义
├── native_engine.cpp        # 通用实现
├── native_api.cpp           # N-API 接口实现
├── native_async_work.cpp    # 异步工作实现
├── native_deferred.h        # Promise deferred
├── native_reference.h       # 引用接口
├── native_value.h           # 值类型定义
├── native_sendable.h        # 可发送对象
├── native_event.h           # 事件接口
├── worker_manager.h         # Worker 管理
├── impl/
│   └── ark/                 # Ark 引擎实现
│       ├── ark_native_engine.h/cpp    # Ark NativeEngine
│       ├── ark_native_reference.h/cpp # 引用实现
│       ├── ark_native_deferred.h/cpp   # Deferred 实现
│       ├── ark_native_timer.h/cpp      # 定时器
│       ├── ark_idle_monitor.h/cpp      # 空闲监控
│       ├── ark_hybrid_native_reference.h/cpp  # 混合引用
│       ├── ark_sendable_native_reference.h/cpp # 可发送引用
│       └── cj_support.h/cpp            # CJ 支持
```

**关键类**：

| 类 | 职责 | 文件 |
|---|------|------|
| `NativeEngine` | 抽象基类，定义 N-API 接口 | `native_engine.h` |
| `ArkNativeEngine` | Ark 引擎实现 | `ark_native_engine.h` |
| `NativeAsyncWork` | 异步工作任务 | `native_async_work.h` |
| `NativeReference` | JS 值引用 | `native_reference.h` |
| `ArkNativeReference` | Ark 引用实现 | `ark_native_reference.h` |

### scope_manager/ - 作用域管理

**位置**：`foundation/arkui/napi/scope_manager/`

**职责**：管理 NativeValue 的生命周期（已标记为废弃）

**注意**：此目录代码标记为 `// To be delete`

### reference_manager/ - 引用管理

**位置**：`foundation/arkui/napi/reference_manager/`

**职责**：管理 NativeReference 的生命周期

**关键文件**：

| 文件 | 用途 |
|------|------|
| `native_reference_manager.h` | 引用管理器 |
| `native_reference_manager.cpp` | 引用管理实现 |

### callback_scope_manager/ - 回调作用域管理

**位置**：`foundation/arkui/napi/callback_scope_manager/`

**职责**：管理异步回调和钩子

**关键文件**：

| 文件 | 用途 |
|------|------|
| `native_callback_scope_manager.h` | 回调作用域管理器 |

### utils/ - 工具类

**位置**：`foundation/arkui/napi/utils/`

**职责**：通用工具函数

**关键文件**：

| 文件 | 用途 |
|------|------|
| `data_protector.cpp/h` | 数据保护（arm64 OHOS） |
| `log.cpp/h` | 日志工具 |
| `macros.h` | 宏定义 |
| `assert.h` | 断言 |
| `file.h/cpp` | 文件操作 |
| `platform/` | 平台相关代码 |

### sample/ - 示例代码

**位置**：`foundation/arkui/napi/sample/`

**职责**：提供 native 模块开发示例

| 示例目录 | 功能 |
|----------|------|
| `native_module_demo/` | 基本模块示例 |
| `native_module_calc/` | 计算器模块 |
| `native_module_callback/` | 回调示例 |
| `native_module_netserver/` | 网络服务器示例 |
| `native_module_storage/` | 存储示例 |
| `native_module_systemtest/` | 系统测试 |

## 忽略的目录

根据约束，以下目录不作为业务证据来源：
- `test/` - 测试代码
- 测试相关的 BUILD.gn 文件

## 模块依赖关系

```
interfaces/kits/napi/
    │
    ▼
native_engine/     ◄──►  module_manager/
    │                     │
    ▼                     ▼
reference_manager/  ◄──►  callback_scope_manager/
    │                     │
    ▼                     ▼
utils/ ◄──────────────► ◄─┘
```
