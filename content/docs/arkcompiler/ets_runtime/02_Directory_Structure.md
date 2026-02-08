# 目录结构

本文档描述 ArkCompiler ETS Runtime 的目录结构和模块职责划分。

## 顶层目录

```
arkcompiler/ets_runtime/
├─ ecmascript/             # 核心运行时实现（主要目录）
├─ common_components/        # 通用组件（跨模块复用）
├─ compiler_service/        # 编译器服务
├─ test/                    # 模块测试代码（不含单元测试）
├─ BUILD.gn                 # 根构建配置
└─ js_runtime_config.gni     # 运行时配置
```

## ecmascript 目录详解

```
ecmascript/
├─ base/                     # 基础辅助类
│   ├─ array_helper.cpp/h    # 数组操作辅助函数
│   ├─ atomic_helper.cpp/h   # 原子操作辅助函数
│   ├─ json_parser.cpp/h    # JSON 解析器
│   ├─ path_helper.cpp/h    # 路径处理辅助函数
│   └─ ...
├─ builtins/                 # ECMAScript 标准内置对象
│   ├─ builtins_array.cpp   # Array 对象
│   ├─ builtins_string.cpp  # String 对象
│   ├─ builtins_map.cpp     # Map 对象
│   ├─ builtins_promise.cpp # Promise 对象
│   ├─ builtins_object.cpp  # Object 对象
│   └─ ... (50+ 内置对象文件)
├─ checkpoint/               # VM 安全点（GC 协调）
├─ compiler/                # AOT 编译器
│   ├─ codegen/             # 代码生成后端
│   │   ├─ llvm/            # LLVM 代码生成
│   │   └─ maple/           # Maple 代码生成
│   ├─ aot_file/            # AOT 文件格式处理
│   ├─ pgo_pgo/             # PGO 优化支持
│   └─ ...
├─ containers/              # 非 ECMA 标准容器（N-API 导出）
│   ├─ containers_vector.cpp/h
│   ├─ containers_list.cpp/h
│   ├─ containers_map.cpp/h
│   ├─ containers_queue.cpp/h
│   └─ ...
├─ daemon/                  # 共享 GC 守护线程
├─ debugger/                 # 调试器支持
├─ deoptimizer/              # 编译去优化支持
├─ dfx/                      # 调试与性能分析
│   ├─ cpu_profiler/        # CPU 性能分析
│   ├─ hprof/               # 堆内存分析
│   └─ stackinfo/           # 栈信息处理
├─ extractortool/            # SourceMap 解析工具
├─ ic/                       # 内联缓存（类型优化）
├─ interpreter/              # 字节码解释器
├─ intl/                     # 国际化支持
├─ jit/                      # JIT 编译器
├─ jobs/                     # 微任务队列
├─ js_api/                   # 非标准 JS API（@ohos.*）
│   ├─ js_api_arraylist.cpp/h
│   ├─ js_api_hashmap.cpp/h
│   ├─ js_api_buffer.cpp/h
│   └─ ...
├─ js_type_metadata/         # JS 类型元数据
├─ js_vm/                    # 命令行工具
├─ jspandafile/              # ABC 文件管理
├─ mem/                      # 内存管理
│   ├─ space/               # 内存空间管理
│   ├─ collector/           # 垃圾回收器
│   ├─ heap/                # 堆内存管理
│   └─ ...
├─ module/                   # ES Module 支持
├─ napi/                     # N-API 接口（核心对外 API）
│   ├─ jsnapi.cpp           # N-API 实现
│   ├─ jsnapi_expo.cpp      # N-API 导出表
│   ├─ dfx_jsnapi.cpp       # DFX 相关 N-API
│   └─ include/
│       ├─ jsnapi.h         # N-API 头文件
│       └─ jsnapi_expo.h    # N-API 导出头文件
├─ ohos/                     # OpenHarmony 系统集成
│   ├─ adapter/              # 系统适配层
│   ├─ code_decrypt.cpp/h   # 代码解密
│   └─ ...
├─ patch/                    # 热修复支持
├─ pgo_profiler/             # PGO 性能分析器
├─ platform/                 # 跨平台适配
├─ quick_fix/                # 快速修复工具
├─ regexp/                   # 正则表达式引擎
├─ require/                  # CommonJS 支持
├─ serializer/               # 序列化支持
├─ shared_mm/                # 共享内存管理
├─ shared_objects/           # 共享对象实现
├─ snapshot/                 # 快照支持
├─ stackmap/                 # 栈映射信息
├─ stubs/                    # Runtime 存根函数
└─ taskpool/                 # 任务池
```

## common_components 目录详解

```
common_components/
├─ base/                     # 基础工具类
├─ common/                   # 通用工具
├─ common_runtime/            # 运行时通用组件
├─ heap/                     # 堆内存管理组件
│   ├─ allocator/            # 内存分配器
│   ├─ collector/            # 垃圾回收器组件
│   ├─ space/                # 内存空间
│   └─ barrier/              # 内存屏障
├─ log/                      # 日志组件
├─ mutator/                  # Mutator 锁
├─ platform/                 # 平台适配
├─ taskpool/                 # 任务池组件
├─ thread/                   # 线程管理
└─ serialize/                # 序列化组件
```

## 模块职责总结

| 模块 | 职责 | 稳定性 |
|------|------|--------|
| `ecmascript/builtins/` | ECMAScript 标准库 | 稳定 |
| `ecmascript/js_api/` | @ohos.* 扩展 API | 稳定 |
| `ecmascript/containers/` | N-API 容器 API | 稳定 |
| `ecmascript/napi/` | N-API 编程接口 | 稳定 |
| `ecmascript/interpreter/` | 字节码解释器 | 稳定 |
| `ecmascript/compiler/` | AOT 编译器 | 稳定 |
| `ecmascript/jit/` | JIT 编译器 | 实验性 |
| `ecmascript/mem/` | 内存管理 | 稳定 |
| `ecmascript/ohos/` | 系统集成 | 稳定 |
| `ecmascript/debugger/` | 调试支持 | 稳定 |

## 相关文档

- [项目概览](01_Project_Overview.md)
- [架构说明](03_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
