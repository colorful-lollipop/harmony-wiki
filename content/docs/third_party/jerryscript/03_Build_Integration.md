# JerryScript OH 构建集成

## 1. 构建系统概览

OpenHarmony 对 JerryScript 进行了完整的构建系统适配，将原始的 **CMake** 构建系统替换为 **GN/Ninja**。

### 1.1 构建文件结构

```
third_party/jerryscript/
├── BUILD.gn                    # 主构建文件（双模式）
├── bundle.json                 # OH 组件定义
├── engine.gni                  # 全局变量定义
├── OAT.xml                     # 开源合规配置
├── jerry-core/
│   └── BUILD.gn              # 核心引擎构建
├── jerry-ext/
│   └── BUILD.gn              # 扩展模块构建
├── jerry-port/
│   └── default/
│       └── BUILD.gn          # 默认端口层构建
├── jerry-libm/
│   └── BUILD.gn              # 数学库构建
└── tests/
    ├── jerry/
    │   └── BUILD.gn          # Jerry 测试
    └── unit-core/
        └── BUILD.gn          # 单元测试
```

### 1.2 双模式构建架构

| 构建模式 | 触发条件 | 构建系统 | 构建目标类型 |
|----------|----------|----------|--------------|
| **Lite 模式** | `defined(ohos_lite)` | GN + lite_component.gni | `lite_component` / `lite_library` |
| **标准模式** | 非 lite 环境 | GN + ohos.gni | `ohos_executable` / `ohos_static_library` / `ohos_shared_library` |

---

## 2. 主构建文件 (BUILD.gn)

### 2.1 条件编译结构

```gn
if (defined(ohos_lite)) {
  # Lite 模式：IP Camera 等轻量设备
  import("//build/lite/config/component/lite_component.gni")

  lite_component("jerry_engine") {
    features = [
      "jerry-core",
      "jerry-ext",
      "jerry-port/default:jerry-port-default",
    ]
    if (ohos_kernel_type != "liteos_m") {
      features += [ "jerry_libm" ]
    }
  }
} else {
  # 标准模式：Linux 等标准系统
  import("//build/ohos.gni")
  import("//third_party/jerryscript/engine.gni")

  # ... 标准模式构建目标（见 2.2 节）
}
```

### 2.2 标准模式构建目标

#### 2.2.1 可执行文件

| 目标名 | 说明 | 源文件 |
|--------|------|--------|
| `jerry` | 完整命令行工具 | jerry-core + jerry-ext + jerry-libm + jerry-port-default + jerry-main |
| `jerry-snapshot` | 快照工具 | jerry-core + jerry-libm + jerry-port-default + jerry-main (snapshot) |

#### 2.2.2 库文件

| 目标名 | 类型 | 说明 |
|--------|------|------|
| `libjerryscript` | `ohos_static_library` | 模拟器静态库 |
| `jerryscript_static_not_lite` | `ohos_static_library` | 标准静态库 |
| `jerryscript_shared_not_lite` | `ohos_shared_library` | 标准动态库（带 part_name） |

### 2.3 编译配置

#### 2.3.1 jerryscript_config（基础配置）

```gn
config("jerryscript_config") {
    defines = [
      "JERRY_COMMIT_HASH=\"ignored\"",
      "JERRY_NDEBUG",
      "JERRY_HEAPDUMP",           # 堆转储支持
      "JERRY_REF_TRACKER",        # 引用追踪
    ]
    # ES2015 功能开关
    defines += [
      "JERRY_SNAPSHOT_SAVE=${jerryscript_jerry_snapshot_save}",
      "JERRY_ES2015=${jerryscript_jerry_es2015}",
      "JERRY_ES2015_BUILTIN_TYPEDARRAY=${jerryscript_jerry_es2015_builtin_typedarray}",
      # ... 更多 ES2015 特性
    ]
    cflags = [
      "-Wno-unused-function",
      "-Wno-sign-compare",
      "-Wno-implicit-fallthrough",
    ]
    include_dirs = [ "." ]
}
```

#### 2.3.2 jerryscript_simulator（模拟器配置）

```gn
config("jerryscript_simulator") {
    defines = [
      "JERRY_FUNCTION_BACKTRACE",  # 函数回溯
      "JERRY_FUNCTION_NAME",      # 函数名
      "JERRY_HEAPDUMP",
      "JERRY_NDEBUG",
      "JERRY_REF_TRACKER",
    ]
    # GC 和 VM 配置
    defines += [
      "JERRY_CPOINTER_32_BIT=${jerryscript_jerry_cpointer_32_bit}",
      "JERRY_DEBUGGER=${jerryscript_jerry_debugger}",
      "JERRY_GC_LIMIT=${jerryscript_jerry_gc_limit}",
      "JERRY_LINE_INFO=${jerryscript_jerry_line_info}",
      "JERRY_MEM_GC_BEFORE_EACH_ALLOC=${jerryscript_jerry_mem_gc_before_each_alloc}",
      "JERRY_PARSER=${jerryscript_jerry_parser}",
      "JERRY_PARSER_DUMP_BYTE_CODE=${jerryscript_jerry_parser_dump_byte_code}",
      "JERRY_REGEXP_DUMP_BYTE_CODE=${jerryscript_jerry_regexp_dump_byte_code}",
      "JERRY_REGEXP_STRICT_MODE=${jerryscript_jerry_regexp_strict_mode}",
      "JERRY_STACK_LIMIT=${jerryscript_jerry_stack_limit}",
      "JERRY_SYSTEM_ALLOCATOR=${jerryscript_jerry_system_allocator}",
      "JERRY_VALGRIND=${jerryscript_jerry_valgrind}",
      "JERRY_VM_EXEC_STOP=${jerryscript_jerry_vm_exec_stop}",
      # ... ES2015 特性（同 jerryscript_config）
    ]
    if (jerryscript_enable_external_context == true) {
      defines += [ "JERRY_EXTERNAL_CONTEXT=1" ]
    }
    # OH 特定缓冲区大小
    defines += [ "INPUTJS_BUFFER_SIZE=${jerryscript_inputjs_buffer_size}" ]
    defines += [ "SNAPSHOT_BUFFER_SIZE=${jerryscript_snapshot_buffer_size}" ]
    defines += [ "BMS_TASK_HEAP_SIZE=${jerryscript_bms_task_heap_size}" ]
    defines += [ "JS_TASK_HEAP_SIZE=${jerryscript_js_task_heap_size}" ]

    cflags = [
      "-Wno-unused-function",
      "-Wno-sign-compare",
      "-Wno-error",  # jerry add
      "-Wno-implicit-fallthrough",
    ]
    include_dirs = [ "." ]
}
```

---

## 3. 全局变量定义 (engine.gni)

### 3.1 核心变量

```gn
declare_args() {
  # 外部上下文支持
  jerryscript_enable_external_context = true

  # 缓冲区大小（单位：KB）
  jerryscript_inputjs_buffer_size = 32768      # 默认 32KB
  jerryscript_snapshot_buffer_size = 24576     # 默认 24KB

  # 任务堆大小（单位：KB）
  jerryscript_bms_task_heap_size = 64          # BMS 任务堆
  jerryscript_js_task_heap_size = 64           # JS 任务堆

  # 内存和 GC 配置
  jerryscript_jerry_cpointer_32_bit = 0        # 32 位压缩指针
  jerryscript_jerry_gc_limit = 0               # GC 触发阈值（0 = 自动）
  jerryscript_jerry_stack_limit = 0             # 栈深度限制（0 = 不限制）
  jerryscript_jerry_system_allocator = 0        # 使用系统分配器
  jerryscript_jerry_mem_gc_before_each_alloc = 0 # 每次分配前 GC
```

### 3.2 ES2015 特性开关

```gn
  # ES2015 基础支持
  jerryscript_jerry_es2015 = 1                  # 启用 ES2015

  # ES2015 内置对象（默认启用）
  jerryscript_jerry_es2015_builtin_typedarray = 1
  jerryscript_jerry_es2015_builtin_set = 1
  jerryscript_jerry_es2015_builtin_promise = 1
  jerryscript_jerry_es2015_builtin_proxy = 1
  jerryscript_jerry_es2015_module_system = 1
  jerryscript_jerry_es2015_builtin_map = 1

  # ES2015 内置对象（默认禁用，内存考虑）
  jerryscript_jerry_es2015_builtin_weakmap = 0   # WeakMap
  jerryscript_jerry_es2015_builtin_weakset = 0   # WeakSet
  jerryscript_jerry_es2015_builtin_dataview = 0   # DataView
  jerryscript_jerry_es2015_builtin_reflect = 0   # Reflect
```

### 3.3 功能开关

```gn
  # 调试和诊断
  jerryscript_jerry_debugger = 1                 # 调试器支持
  jerryscript_jerry_line_info = 1               # 行号信息
  jerryscript_jerry_parser_dump_byte_code = 0    # 字节码转储
  jerryscript_jerry_regexp_dump_byte_code = 0    # RegExp 字节码转储
  jerryscript_jerry_valgrind = 0                # Valgrind 支持
  jerryscript_jerry_vm_exec_stop = 0            # VM 执行停止

  # 错误处理和日志
  jerryscript_jerry_error_messages = 1           # 错误信息
  jerryscript_jerry_logging = 0                  # 日志输出

  # 快照支持
  jerryscript_jerry_snapshot_exec = 1            # 执行快照
  jerryscript_jerry_snapshot_save = 1            # 保存快照
}
```

---

## 4. 子模块构建

### 4.1 jerry-core/BUILD.gn

**核心引擎构建配置**

#### 条件编译

```gn
# LiteOS-M 使用静态库，其他使用共享库
if (ohos_kernel_type == "liteos_m") {
  lite_library("jerry-core_static") {
    sources = jerry_core_sources
    include_dirs = jerry_core_include_dirs
  }
} else {
  lite_library("jerry-core_shared") {
    sources = jerry_core_sources
    include_dirs = jerry_core_include_dirs
  }
}
```

#### IAR 工具链适配

```gn
if (board_toolchain_type == "iccarm") {
  # 额外源文件
  sources += [
    "api/external-context-helpers.c",
    "api/generate-bytecode.c",
    "api/jerryscript_adapter.c",
  ]

  # IAR 特定宏
  defines += [
    "JERRY_IAR_JUPITER",
    "JERRY_FOR_IAR_CONFIG",
  ]

  # 抑制警告
  cflags = [
    "--diag_suppress",
    "Pa089,Pe111,Pe188,Pe191,Pe546,Pe940,Pe128",
  ]

  # 额外 include 路径
  include_dirs += [
    "//commonlibrary/utils_lite/memory/include",
    "//commonlibrary/utils_lite/include",
  ]
}
```

### 4.2 jerry-ext/BUILD.gn

**扩展模块构建配置**

```gn
# 条件编译（同 jerry-core）
if (ohos_kernel_type == "liteos_m") {
  lite_library("jerry-ext_static") { ... }
} else {
  lite_library("jerry-ext_shared") { ... }
}

# IAR 工具链适配
if (board_toolchain_type == "iccarm") {
  cflags = [
    "--diag_suppress",
    "Pe111,Pe940",
  ]
}
```

**主要源文件**:
- `arg/*.c` - 参数验证工具
- `debugger/*.c` - 调试器支持
- `handle-scope/*.c` - 句柄作用域
- `module/*.c` - 模块系统

### 4.3 jerry-port/default/BUILD.gn

**默认端口层构建配置**

```gn
# 条件编译（同 jerry-core）
if (ohos_kernel_type == "liteos_m") {
  lite_library("jerry-port-default_static") { ... }
} else {
  lite_library("jerry-port-default_shared") { ... }
}

# IAR 工具链适配
if (board_toolchain_type == "iccarm") {
  cflags = [
    "--diag_suppress",
    "Pe111",
  ]
}
```

**主要源文件**:
- `default-date.c` - 日期函数
- `default-debugger.c` - 调试器 I/O
- `default-external-context.c` - 外部上下文
- `default-fatal.c` - 致命错误处理
- `default-io.c` - 文件 I/O
- `default-module.c` - 模块加载

### 4.4 jerry-libm/BUILD.gn

**数学库构建配置**

```gn
# 条件编译（同 jerry-core）
if (ohos_kernel_type == "liteos_m") {
  # LiteOS-M 不构建 libm（使用系统 math 库）
  # 无构建目标
} else {
  lite_library("jerry-libm_static") { ... }
  lite_library("jerry-libm_shared") { ... }
}

# IAR 工具链适配
if (board_toolchain_type == "iccarm") {
  cflags = [
    "--diag_suppress",
    "Pe039,Pa089,Pe222",
  ]
}
```

**说明**:
- JerryScript 自包含数学库，用于不支持标准 libm 的系统
- LiteOS-M 系统使用系统 math 库，不构建 jerry-libm

---

## 5. OH 特定配置

### 5.1 组件定义 (bundle.json)

```json
{
  "name": "@ohos/jerryscript",
  "description": "JerryScript is the lightweight JavaScript engine...",
  "version": "3.1",
  "license": "Apache V2",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/jerryscript"
  },
  "component": {
    "name": "jerryscript",
    "subsystem": "thirdparty",
    "syscap": [],
    "features": [
      "jerryscript_enable_external_context",
      "jerryscript_inputjs_buffer_size",
      "jerryscript_snapshot_buffer_size",
      "jerryscript_bms_task_heap_size",
      "jerryscript_js_task_heap_size",
      // ... 50+ 特性开关
    ],
    "adapted_system_type": ["mini","small"],
    "deps": {
      "components": [],
      "third_party": []
    },
    "build": {
      "inner_kits": [
        {
          "name": "//third_party/jerryscript:jerryscript_static_not_lite"
        },
        {
          "name": "//third_party/jerryscript:jerryscript_shared_not_lite"
        },
        {
          "name": "//third_party/jerryscript:libjerryscript"
        }
      ]
    }
  }
}
```

**说明**:
- 定义了 50+ 个可配置特性
- 支持轻量级系统（mini, small）
- 提供 3 个内部构建产物供其他模块依赖

### 5.2 OAT.xml（开源合规）

```xml
<OAT>
  <Project Name="jerryscript" Version="v2.3.0" License="Apache-2.0">
    <UpstreamUrl>https://github.com/jerryscript-project/jerryscript.git</UpstreamUrl>
    <Description>JavaScript engine for the Internet of Things</Description>
  </Project>
  <!-- OpenHarmony OSS Audit Tool 配置 -->
</OAT>
```

### 5.3 头文件配置

#### jerryscript.h 条件包含

```c
// jerry-core/include/jerryscript.h
#ifdef JERRY_IAR_JUPITER
#include "ohos_mem_pool.h"  // OH 内存池接口
#endif
```

#### config-jupiter.h OH 类型引用

```c
// jerry-port/config-jupiter.h
#ifdef JERRY_IAR_JUPITER
#include "ohos_types.h"  // OH 类型定义
#endif
```

---

## 6. 与上游构建系统的差异

| 方面 | 上游（CMake） | OH（GN） | 说明 |
|------|---------------|----------|------|
| **构建系统** | CMake | GN/Ninja | OH 统一构建体系 |
| **平台支持** | 自定义工具链 | Lite/标准双模式 | OH 平台体系适配 |
| **组件化** | 可执行文件 + 库 | OH 组件（bundle.json） | OH 组件管理 |
| **配置方式** | CMakeLists.txt | BUILD.gn + engine.gni | GN 声明式配置 |
| **特性开关** | CMake 变量 | GN declare_args | 构建参数管理 |
| **条件编译** | `if(CMAKE_SYSTEM_NAME MATCHES "Linux")` | `if (defined(ohos_lite))` | OH 特定判断 |
| **库类型** | 静态/共享库 | lite_library/ohos_static_library/ohos_shared_library | 多种库类型适配 |
| **工具链** | GCC/Clang | GCC/Clang/IAR | IAR 工具链支持 |

---

## 7. 构建示例

### 7.1 轻量系统（LiteOS-M）

```bash
# 构建命令（简化示例）
hb build -f -t liteos_m --build-type debug
```

**生成产物**:
- `jerry-core_static.a`
- `jerry-ext_static.a`
- `jerry-port-default_static.a`
- `jerry_engine.lite_component`

### 7.2 标准系统（Linux）

```bash
# 构建命令（简化示例）
./build.sh --product-name rk3568 --ccache
```

**生成产物**:
- `libjerryscript.a`（静态库）
- `libjerryscript.so`（动态库）
- `jerry`（可执行文件）
- `jerry-snapshot`（快照工具）

### 7.3 模拟器构建

```bash
# 构建命令（简化示例）
./build.sh --product-name ohos-sdk --ccache
```

**生成产物**:
- `libjerryscript.a`（带额外调试信息的静态库）

---

## 8. 构建优化建议

### 8.1 内存优化

**降低堆大小**（针对极低内存设备）:
```gn
jerryscript_inputjs_buffer_size = 16384    # 16KB（默认 32KB）
jerryscript_snapshot_buffer_size = 12288   # 12KB（默认 24KB）
jerryscript_js_task_heap_size = 32         # 32KB（默认 64KB）
```

**禁用 ES2015 特性**（减少代码体积）:
```gn
jerryscript_jerry_es2015_builtin_typedarray = 0  # 减少 ~20KB
jerryscript_jerry_es2015_builtin_promise = 0     # 减少 ~15KB
jerryscript_jerry_es2015_builtin_proxy = 0       # 减少 ~10KB
```

### 8.2 性能优化

**启用 GC 每次分配前执行**（减少内存碎片）:
```gn
jerryscript_jerry_mem_gc_before_each_alloc = 1
```

**使用系统分配器**（提升分配速度）:
```gn
jerryscript_jerry_system_allocator = 1
```

**增大堆大小**（减少 GC 频率）:
```gn
jerryscript_js_task_heap_size = 128  # 128KB（默认 64KB）
```

### 8.3 调试优化

**启用详细错误信息**:
```gn
jerryscript_jerry_error_messages = 1
jerryscript_jerry_line_info = 1
jerryscript_jerry_debugger = 1
```

**启用字节码转储**（用于性能分析）:
```gn
jerryscript_jerry_parser_dump_byte_code = 1
jerryscript_jerry_regexp_dump_byte_code = 1
```

---

## 9. 常见问题

### Q1: 如何修改默认配置？

在项目的 `args.gn` 或 `BUILD.gn` 中覆盖 engine.gni 定义的变量：

```gn
# 项目的 BUILD.gn
import("//third_party/jerryscript/engine.gni")

jerryscript_js_task_heap_size = 128  # 覆盖默认值
```

### Q2: LiteOS-M 和标准系统有什么区别？

| 特性 | LiteOS-M | 标准系统 |
|------|----------|----------|
| **库类型** | 静态库（lite_library） | 共享库（ohos_shared_library） |
| **数学库** | 使用系统 math 库 | 使用 jerry-libm |
| **工具链** | 通常 IAR | GCC/Clang |
| **内存限制** | 极低（<64KB） | 较宽松（>64KB） |

### Q3: IAR 工具链适配是否必要？

如果目标平台使用 IAR 编译器（如 JUPITER 芯片），则需要启用 IAR 适配：

```gn
board_toolchain_type = "iccarm"  # 在产品配置中设置
```

这会自动添加：
- 额外源文件（external-context-helpers.c 等）
- IAR 特定宏（JERRY_IAR_JUPITER）
- 抑制警告标志

### Q4: 如何检查实际编译选项？

在编译日志中搜索 `JERRY_` 前缀的宏定义：

```bash
grep -r "JERRY_" out/ 2>/dev/null | head
```

或者查看编译命令：

```bash
ninja -C out/rk3568 -v 2>&1 | grep "jerry" | head
```

---

## 10. 升级构建系统

### 10.1 上游 CMake 到 GN 的迁移原则

1. **保持功能等价**: 确保 GN 构建产物与 CMake 一致
2. **保留 OH 特性**: 保留 lite/标准模式双架构、IAR 适配等
3. **遵循 OH 规范**: 使用 OH 组件定义、子系统分类等

### 10.2 升级检查清单

- [ ] 所有源文件都已包含在 BUILD.gn 中
- [ ] 所有编译选项（defines, cflags）都已映射
- [ ] 头文件路径（include_dirs）正确配置
- [ ] 库依赖关系正确声明
- [ ] IAR 工具链适配已更新
- [ ] lite/标准模式条件编译正确
- [ ] bundle.json 特性列表已更新
- [ ] 编译产物已验证（静态库/动态库/可执行文件）

---

**相关文档**:
- [01_Overview.md](01_Overview.md) - 原始库简介
- [02_Patches.md](02_Patches.md) - Patch 详细分析
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用场景
