# 目录结构与模块职责

## 顶层目录结构

```
arkcompiler/runtime_core/
├── BUILD.gn                    # 根构建文件
├── bundle.json                 # 组件描述文件
├── README.md / README_zh.md    # 项目说明
├── ark_config.gni              # GN 配置
├── static_vm_config.gni        # 静态 VM 配置
│
├── abc2program/                # ABC 到程序转换工具
├── ark_guard/                  # Ark 混淆保护工具
├── arkplatform/                # Ark 平台适配层
├── assembler/                  # 汇编器（PA → ABC）
├── bytecode_optimizer/         # 字节码优化器
├── cmake/                      # CMake 构建脚本
├── common_interfaces/          # 公共接口定义
├── common_runtime/             # 公共运行时
├── compiler/                   # JIT/AOT 编译器（旧版）
├── disassembler/               # 反汇编器（ABC → PA）
├── docs/                       # 设计文档
├── gn/                         # GN 模板
├── isa/                        # 指令集架构定义
├── ldscripts/                  # 链接器脚本
├── libabckit/                  # ABC Kit 库
├── libark_defect_scan_aux/     # 漏洞扫描辅助库
├── libpandabase/               # 基础库（旧版）
├── libpandafile/               # 字节码文件库（旧版）
├── libziparchive/              # Zip 归档库
├── panda/                      # CLI 工具
├── pandastdlib/                # Panda 标准库
├── platforms/                  # 平台抽象层
├── plugins/                    # 语言插件（旧版）
├── scripts/                    # CI 脚本
├── static_core/                # 静态核心（新版主要代码）
├── taihe/                      # 泰合（ANI 运行时）
├── templates/                  # Ruby 模板
├── test/ tests/                # 测试（本文档忽略）
└── verifier/                   # 字节码验证器
```

## 核心模块详解

### 1. static_core/ - 静态核心（最重要）

这是 Runtime Core 的主要代码目录，包含完整的运行时实现。

```
static_core/
├── BUILD.gn                    # 静态核心构建入口
├── ark_config.gni              # 静态核心配置
├── README.md                   # 开发指南
│
├── abc2program/                # ABC 到程序转换
├── assembler/                  # 汇编器
├── bytecode_optimizer/         # 字节码优化器
├── compiler/                   # JIT/AOT 编译器
│   ├── optimizer/              # 优化器（IR、Pass）
│   ├── aot/                    # AOT 编译
│   └── tools/                  # 编译器工具
│       ├── paoc/               # Panda AOT 编译器
│       └── aotdump/            # AOT 文件 dump 工具
├── disassembler/               # 反汇编器
├── dprof/                      # 性能分析数据收集
├── isa/                        # 指令集定义
├── libarkbase/                 # 基础库（新版）
│   ├── mem/                    # 内存管理
│   ├── os/                     # 操作系统抽象
│   ├── taskmanager/            # 任务管理器
│   └── utils/                  # 工具类
├── libarkfile/                 # 字节码文件库（新版）
│   ├── include/                # 公共头文件
│   └── src/                    # 实现
├── libziparchive/              # Zip 支持
├── pandastdlib/                # 标准库
├── platforms/                  # 平台抽象
│   ├── common/                 # 通用平台代码
│   ├── unix/                   # Unix/Linux 实现
│   ├── ohos/                   # OpenHarmony 实现
│   └── windows/                # Windows 实现
├── plugins/                    # 语言插件
│   └── ets/                    # ETS (ArkTS) 插件
│       ├── runtime/            # ETS 运行时
│       │   ├── ani/            # ANI 接口实现
│       │   ├── interop_js/     # JS 互操作
│       │   ├── interpreter/    # 解释器
│       │   ├── intrinsics/     # 内建函数
│       │   ├── mem/            # ETS 内存管理
│       │   └── types/          # ETS 类型系统
│       ├── compiler/           # ETS 编译器支持
│       ├── stdlib/             # ETS 标准库
│       └── tools/              # ETS 工具
├── runtime/                    # 核心运行时
│   ├── include/                # 公共头文件
│   ├── mem/                    # 内存管理
│   │   ├── gc/                 # 垃圾回收器
│   │   │   ├── g1/             # G1 GC
│   │   │   ├── epsilon/        # Epsilon GC
│   │   │   └── workers/        # GC 工作线程
│   │   └── allocator/          # 分配器
│   ├── interpreter/            # 解释器
│   ├── tooling/                # 工具支持
│   │   ├── inspector/          # 调试器
│   │   ├── sampler/            # 采样器
│   │   └── backtrace/          # 堆栈回溯
│   └── coroutines/             # 协程支持
├── static_linker/              # 静态链接器
├── tools/                      # 工具集
└── verification/               # 字节码验证器
```

**关键文件**:
- `static_core/runtime/include/runtime.h` - 运行时主类
- `static_core/runtime/include/thread.h` - 线程定义
- `static_core/runtime/include/class.h` - 类定义
- `static_core/runtime/mem/gc/gc.h` - GC 基类

### 2. libpandabase/ - 基础库（旧版）

向后兼容的基础库目录，新代码应使用 `static_core/libarkbase/`。

```
libpandabase/
├── include/
│   └── libpandabase/
│       ├── macros.h            # 常用宏
│       ├── mem/                # 内存管理头文件
│       ├── os/                 # OS 抽象头文件
│       └── utils/              # 工具类头文件
├── mem/                        # 内存管理实现
├── os/                         # OS 抽象实现
├── utils/                      # 工具类实现
└── platforms/                  # 平台相关代码
```

### 3. libpandafile/ - 字节码文件（旧版）

向后兼容的字节码文件库。

```
libpandafile/
├── include/
│   └── file.h                  # 主头文件
├── file.cpp                    # 文件实现
├── class_data_accessor.cpp     # 类数据访问器
└── method_data_accessor.cpp    # 方法数据访问器
```

### 4. assembler/ - 汇编器

将 Panda Assembly（.pa）编译为 Ark Bytecode（.abc）。

```
assembler/
├── assembler.cpp               # 主实现
├── assembly_parser.cpp         # 汇编语法解析器
├── assembly_program.cpp        # 程序表示
└── extensions/                 # 扩展支持
```

**关键符号**:
- `pandasm::Parser` - 汇编解析器
- `pandasm::Program` - 汇编程序表示

### 5. disassembler/ - 反汇编器

将 ABC 反汇编为可读的 PA 格式。

```
disassembler/
├── disassembler.cpp            # 主实现
├── disasm.cpp                  # 命令行工具
└── templates/                  # 代码生成模板
```

**产物**: `ark_disasm` 可执行文件

### 6. bytecode_optimizer/ - 字节码优化器

对 ABC 进行优化，生成更高效的 ABC。

```
bytecode_optimizer/
├── bytecode_optimizer.cpp      # 主实现
├── constant_propagation/       # 常量传播
└── ir_interface.h              # IR 接口
```

### 7. compiler/ - 编译器（旧版）

JIT/AOT 编译器的老版本目录，新代码在 `static_core/compiler/`。

### 8. verifier/ - 字节码验证器

验证 ABC 文件的安全性和正确性。

```
verifier/
├── verify.cpp                  # 验证主逻辑
├── verifier.cpp                # 验证器实现
└── job_queue/                  # 验证任务队列
```

**产物**: `ark_verifier` 可执行文件

### 9. libabckit/ - ABC Kit

ABC 文件操作工具包，支持读取、修改、生成 ABC。

```
libabckit/
├── include/                    # 公共头文件
├── src/                        # 实现
│   ├── adapter_static/         # 静态适配器
│   ├── adapter_dynamic/        # 动态适配器
│   ├── irbuilder_dynamic/      # IR 构建器
│   └── codegen/                # 代码生成
└── abckit/                     # 工具入口
```

### 10. arkplatform/ - Ark 平台层

OpenHarmony 平台适配层，提供与系统服务的集成。

```
arkplatform/
├── hybrid/                     # 混合运行时支持
│   ├── vm_interface.h          # VM 接口
│   ├── ecma_vm_interface.h     # JS VM 接口
│   └── sts_vm_interface.h      # ETS VM 接口
└── BUILD.gn
```

### 11. taihe/ - 泰合

ANI 运行时支持库。

```
taihe/
├── runtime/
│   └── include/
│       └── taihe/              # Taihe 头文件
│           ├── runtime.hpp     # 运行时支持
│           ├── array.hpp       # 数组支持
│           └── platform/ani.hpp # ANI 平台支持
└── compiler/                   # Taihe 编译器
```

## 模块依赖关系

```
依赖关系图（简化）:

                    ┌─────────────┐
                    │  应用代码    │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │   ANI API   │ │   N-API     │ │  ETS Runtime │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │               │
           └───────────────┼───────────────┘
                           │
                    ┌──────▼──────┐
                    │ Runtime Core │
                    │  ┌───────┐  │
                    │  │  GC   │  │
                    │  ├───────┤  │
                    │  │Thread │  │
                    │  ├───────┤  │
                    │  │ File  │  │
                    │  ├───────┤  │
                    │  │ Base  │  │
                    │  └───────┘  │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  OS / 平台   │
                    └─────────────┘
```

## 代码组织原则

### 1. 头文件组织

```
include/ 目录结构:
├── public/                     # 对外暴露的 API
│   ├── ani.h                   # ANI 主头文件
│   └── runtime.h               # 运行时公共头文件
├── internal/                   # 内部使用
│   └── ...
└── generated/                  # 生成的头文件
    └── ...
```

### 2. 源文件组织

- 每个模块有独立的目录
- 实现文件（.cpp）与头文件（.h）对应
- 平台相关代码放在 `platforms/` 下

### 3. 测试代码

按约束**不包含**在本文档中，测试目录包括：
- `test/`, `tests/` - 单元测试
- `fuzztest/` - Fuzz 测试
- `*_test.cpp` - 测试文件

## 关键路径速查

| 要找什么 | 去哪里找 |
|----------|----------|
| ANI 接口定义 | `static_core/plugins/ets/runtime/ani/ani.h` |
| N-API 实现 | `static_core/plugins/ets/runtime/interop_js/` |
| GC 实现 | `static_core/runtime/mem/gc/` |
| 线程管理 | `static_core/runtime/include/thread.h` |
| 类加载 | `static_core/runtime/include/class_linker.h` |
| 解释器 | `static_core/runtime/interpreter/` |
| 编译器 IR | `static_core/compiler/optimizer/ir/` |
| 字节码文件 | `static_core/libarkfile/` |
| 基础工具 | `static_core/libarkbase/` |
| 构建配置 | `BUILD.gn`, `ark_config.gni` |

## 下一步

- 深入理解 [架构说明](03_Architecture.md)
- 查看 [对外 API](04_Public_API.md) 进行开发
- 了解 [GN 构建目标](06_GN_Targets.md)
