# OH 构建系统适配

> 本文档详细说明 Abseil-CPP 如何集成到 OpenHarmony 的 GN 构建系统。

---

## 概述

### 构建系统

| 系统 | 上游 | OpenHarmony |
|------|------|-----------|
| **主要** | Bazel | **GN** |
| **次要** | CMake | - |
| **OH 特有** | - | BUILD.gn, configure_copts.gni |

### 构建目标统计

- **共享库**: 25 个（`ohos_shared_library`）
- **静态库**: 1 个（`ohos_static_library`）
- **总计**: 26 个目标

---

## BUILD.gn 结构

### 导入文件

```gn
import("//build/ohos.gni")
import("./configure_copts.gni")
```

- `//build/ohos.gni`: OH 原生构建规则
- `./configure_copts.gni`: 编译器标志和路径变量

### 配置变量

| 变量 | 值 | 来源 |
|------|-----|------|
| `ABSEIL_DIR` | `//third_party/abseil-cpp/` | configure_copts.gni |
| `THIRDPARTY_ABSEIL_SUBSYS_NAME` | `"thirdparty"` | configure_copts.gni |
| `THIRDPARTY_ABSEIL_PART_NAME` | `"abseil-cpp"` | configure_copts.gni |

### 公共配置

```gn
config("absl_public_config") {
  include_dirs = [ "${ABSEIL_DIR}/" ]
}
```

- 所有库目标引用此配置
- 提供统一的头文件路径

---

## 库目标详解

### 1. 核心库

#### absl_base

**类型**: `ohos_shared_library`

**源文件** (11 个):
- `cycleclock.cc`
- `low_level_alloc.cc`
- `raw_logging.cc`
- `spinlock.cc`
- `strerror.cc`
- `sysinfo.cc`
- `thread_identity.cc`
- `throw_delegate.cc`
- `tracing.cc`
- `unscaledcycleclock.cc`

**依赖**:
- `:absl_log_severity`
- `:absl_raw_logging_internal`
- `:absl_spinlock_wait`

**特点**:
- 所有其他 abseil 库的基础
- 提供初始化和原子操作
- **install_enable = true**

#### absl_base_static

**类型**: `ohos_static_library`

**源文件** (194 个):
- 包含所有 abseil 组件的源文件
- 单体静态库，适合静态链接场景

**特点**:
- OH 特有的整合方式
- 上游没有对应的单体静态库
- 减少链接时目标数量

---

### 2. 日志系统

#### absl_log_severity

**类型**: `ohos_shared_library`

**源文件**:
- `log_severity.cc`

**特点**:
- 定义日志严重级别
- 最小化的日志基础设施
- **innerapi_tags = ["platformsdk_indirect"]**

#### absl_raw_logging_internal

**类型**: `ohos_shared_library`

**源文件**:
- `raw_logging.cc`

**安全加固**:
```gn
branch_protector_ret = "pac_ret"
```

**特点**:
- 内部原始日志实现
- PAC-RET 分支保护
- **innerapi_tags = ["platformsdk_indirect"]**

#### absl_log

**类型**: `ohos_shared_library`

**源文件** (17 个):
- 日志框架核心实现
- 内部格式化和条件处理

**依赖**:
- `:absl_base`
- `:absl_hash`
- `:absl_raw_logging_internal`
- `:absl_spinlock_wait`
- `:absl_stacktrace`
- `:absl_str_format_internal`
- `:absl_strings`
- `:absl_sync`
- `:absl_time`
- `:absl_time_zone`

**安全加固**:
```gn
branch_protector_ret = "pac_ret"
```

**特点**:
- 完整的日志框架（LOG、CHECK 宏）
- PAC-RET 分支保护
- **innerapi_tags = ["platformsdk_indirect"]**

---

### 3. 字符串处理

#### absl_strings

**类型**: `ohos_shared_library`

**源文件** (15 个):
- ASCII 转换
- 字符串连接和分割
- 数字转换
- 匹配和替换

**依赖**:
- `:absl_int128`
- `:absl_raw_logging_internal`
- `:absl_strings_internal`

**配置**:
```gn
configs = [ ":cflags_config" ]
```

**安全加固**:
```gn
branch_protector_ret = "pac_ret"
```

**特点**:
- 字符串操作（常用攻击面）
- PAC-RET 分支保护
- **innerapi_tags = ["platformsdk_indirect"]**

#### absl_strings_internal

**类型**: `ohos_shared_library`

**源文件**:
- 内部字符串辅助
- UTF-8 处理

**依赖**:
- `:absl_raw_logging_internal`
- `:absl_throw_delegate`

**安全加固**:
```gn
branch_protector_ret = "pac_ret"
```

**特点**:
- 字符串内部实现
- PAC-RET 分支保护
- **innerapi_tags = ["platformsdk_indirect"]**

#### absl_cord

**类型**: `ohos_shared_library`

**源文件** (16 个):
- Cord 增量字符串实现
- CRC 校验
- Cord 分析和统计

**依赖**:
- `:absl_base`
- `:absl_raw_logging_internal`
- `:absl_spinlock_wait`
- `:absl_stacktrace`
- `:absl_strings`
- `:absl_symbolize`
- `:absl_sync`
- `:absl_throw_delegate`
- `:absl_time`

**特点**:
- 高效的增量字符串类型
- **install_enable = true**

#### absl_str_format_internal

**类型**: `ohos_shared_library`

**源文件** (6 个):
- 格式化字符串实现
- printf 风格格式化

**依赖**:
- `:absl_int128`
- `:absl_strings`

**特点**:
- 内部格式化实现
- **install_enable = true**

---

### 4. 同步和并发

#### absl_spinlock_wait

**类型**: `ohos_shared_library`

**源文件** (5 个):
- spinlock 实现的汇编内联
- 支持多平台

**特点**:
- 平台特定实现
- **install_enable = true**

#### absl_sync

**类型**: `ohos_shared_library`

**源文件** (14 个):
- Mutex 实现
- Barrier、BlockingCounter、Notification
- 线程身份创建
- 内部 waiter 实现（futex、pthread、stdcpp 等）

**依赖**:
- `:absl_base`
- `:absl_raw_logging_internal`
- `:absl_spinlock_wait`
- `:absl_stacktrace`
- `:absl_symbolize`
- `:absl_time`

**特点**:
- 并发原语
- 替代 `std::mutex`
- **install_enable = true**

---

### 5. 时间处理

#### absl_civil_time

**类型**: `ohos_shared_library`

**源文件**:
- `civil_time_detail.cc`

**特点**:
- 民用时间表示
- 独立于时区

#### absl_time_zone

**类型**: `ohos_shared_library`

**源文件** (11 个):
- 时区实现
- 时区格式化
- 时区查找（POSIX、libc、POSIX、info）

**依赖**:
- `:absl_civil_time`

**特点**:
- 完整的时区支持
- **install_enable = true**

#### absl_time

**类型**: `ohos_shared_library`

**源文件** (6 个):
- 时长、绝对时间
- 时间格式化

**依赖**:
- `:absl_base`
- `:absl_civil_time`
- `:absl_int128`
- `:absl_raw_logging_internal`
- `:absl_strings`
- `:absl_time_zone`

**特点**:
- 时间和时长处理
- **install_enable = true**

---

### 6. 其他重要库

#### absl_status / absl_statusor

**类型**: `ohos_shared_library`

**源文件**:
- Status 和 StatusOr 实现
- 错误处理和传播

**特点**:
- 错误处理框架
- **install_enable = true**

#### absl_container

**类型**: `ohos_shared_library`

**源文件**:
- Swiss table 实现
- 高效哈希表

**依赖**:
- `:absl_base`
- `:absl_hash`

**特点**:
- 高效容器
- **install_enable = true**

#### absl_hash

**类型**: `ohos_shared_library`

**源文件** (5 个):
- 哈希框架
- City hash 和 low_level_hash

**依赖**:
- `:absl_base`
- `:absl_stacktrace`
- `:absl_symbolize`
- `:absl_time`

**特点**:
- 哈希框架
- **install_enable = true**

---

## 编译器配置

### ABSL_DEFAULT_COPTS

**文件**: `configure_copts.gni`

**内容**: 77 行的编译标志列表

#### 启用的警告

```gn
-Wall
-Wextra
-Weverything
-Wbitfield-enum-conversion
-Wbool-conversion
-Wconstant-conversion
-Wenum-conversion
-Wint-conversion
-Wliteral-conversion
-Wstring-conversion
```

#### 禁用的警告

```gn
-Wno-c++98-compat-pedantic
-Wno-conversion
-Wno-covered-switch-default
-Wno-deprecated
-Wno-disabled-macro-expansion
-Wno-double-promotion
-Wno-comma
-Wno-extra-semi
-Wno-extra-semi-stmt
-Wno-packed
-Wno-padded
-Wno-sign-compare
-Wno-float-conversion
-Wno-float-equal
-Wno-format-nonliteral
-Wno-gcc-compat
-Wno-global-constructors
-Wno-exit-time-destructors
-Wno-non-modular-include-in-module
-Wno-old-style-cast
-Wno-range-loop-analysis
-Wno-reserved-id-macro
-Wno-shorten-64-to-32
-Wno-switch-enum
-Wno-thread-safety-negative
-Wno-unknown-warning-option
-Wno-unreachable-code
-Wno-unused-macros
-Wno-weak-vtables
-Wno-zero-as-null-pointer-constant
-Wno-reserved-identifier
-Wno-unused-template
-Wno-unknown-pragmas
-Wno-c++17-attribute-extensions
-Wno-cast-function-type
-Wno-atomic-implicit-seq-cst
-Wno-used-but-marked-unused
-Wno-shadow
```

#### 其他标志

```gn
-DNOMINMAX
-Wno-shadow-field-in-constructor
-Wno-unreachable-code-break
-Wno-missing-noreturn
-Wno-tautological-type-limit-compare
-Wno-unreachable-code-return
-Wno-error=unknown-pragmas
-Wno-undef
```

### cflags_config

**使用于**: `absl_strings`, `absl_strings_internal`, `absl_cord`, `absl_str_format_internal`

#### 特殊定义

```gn
config("cflags_config") {
  cflags = [
    # ... warning flags ...
    "NDEBUG"  # ← 关键！强制 Release 模式
  ]
}
```

#### 技术债务

```gn
# Adapating DEBUG version, FIX ME
# https://gitee.com/openharmony/build/pulls/1206/files
defines = [ "NDEBUG" ]
```

**问题**:
- 强制定义 `NDEBUG`，即使 OH 为 Debug 构建
- 禁用所有 `assert` 和调试检查

**影响**:
- Debug 构建失去调试能力
- 无法捕获内部断言失败

**建议**:
- 调查 PR #1206
- 确定是否可以移除此硬编码

---

## 安全加固

### PAC-RET 分支保护

| 目标 | PAC-RET | 原因 |
|------|---------|------|
| `absl_raw_logging_internal` | ✅ | 关键日志路径 |
| `absl_log` | ✅ | 日志基础设施 |
| `absl_strings` | ✅ | 字符串操作（攻击面） |
| `absl_strings_internal` | ✅ | 字符串内部实现 |

**PAC-RET 说明**:
- Pointer Authentication Code for Return addresses
- ARM 安全特性
- 防止 ROP（Return-Oriented Programming）攻击

### Inner API 标签

设置 `innerapi_tags = ["platformsdk_indirect"]` 的目标：

- `absl_raw_logging_internal`
- `absl_log`
- `absl_strings`
- `absl_strings_internal`
- `absl_int128`
- `absl_throw_delegate`

**说明**:
- 可用给平台开发者
- 不作为直接公共 API 暴露
- 控制库的可见性

---

## OpenHarmony 特定设置

### 子系统/部件集成

每个目标都包含：

```gn
subsystem_name = "${THIRDPARTY_ABSEIL_SUBSYS_NAME}"  # = "thirdparty"
part_name = "${THIRDPARTY_ABSEIL_PART_NAME}"         # = "abseil-cpp"
```

**说明**:
- 集成到 OH 的部件模型
- 支持按子系统组织代码
- 方便系统镜像构建

### 安装设置

```gn
install_enable = true
```

**应用于**: 24/26 个目标（所有共享库）

**说明**:
- 将库包含到系统镜像
- 支持系统级应用链接
- 默认分发到 OH 设备

---

## 与上游差异

| 方面 | 上游（Bazel/CMake） | OpenHarmony（GN） |
|------|---------------------|------------------|
| **构建系统** | Bazel（主要）、CMake | **GN** |
| **库类型** | 默认静态 | **大部分共享**（ohos_shared_library） |
| **NDEBUG** | 可配置 | **硬编码**在 cflags_config（技术债务） |
| **分支保护** | 通常不配置 | **4 个关键目标启用 `pac_ret`** |
| **安装/分发** | CMake install 目标 | **`install_enable = true`** |
| **子系统模型** | N/A | **集成到 OH 部件/子系统** |
| **API 可见性** | 标准 C++ 可见性 | **`innerapi_tags` 控制** |
| **静态库整合** | N/A | **`absl_base_static` 组合 194 个源文件** |

---

## 如何在自己的模块中依赖 Abseil-CPP

### 方法 1：使用 external_deps

```gn
import("//build/ohos.gni")

ohos_shared_library("my_library") {
  sources = [ "my_lib.cc" ]

  external_deps = [
    "//third_party/abseil-cpp:absl_strings",
    "//third_party/abseil-cpp:absl_sync",
  ]

  include_dirs = [ "include" ]
}
```

### 方法 2：使用公共配置

```gn
ohos_shared_library("my_library") {
  sources = [ "my_lib.cc" ]

  deps = [
    "//third_party/abseil-cpp:absl_strings",
  ]

  public_configs = [
    "//third_party/abseil-cpp:absl_public_config"
  ]

  include_dirs = [ "include" ]
}
```

### 常用依赖

| 用途 | 推荐的 abseil 库 |
|------|-------------------|
| 字符串处理 | `absl_strings` |
| 并发/同步 | `absl_sync` |
| 时间/时长 | `absl_time`, `absl_time_zone` |
| 错误处理 | `absl_status`, `absl_statusor` |
| 容器 | `absl_container`, `absl_hash` |
| 日志 | `absl_log`, `absl_raw_logging_internal` |
| 128 位整数 | `absl_int128` |

### 静态链接

如果需要静态链接：

```gn
ohos_static_library("my_static_lib") {
  sources = [ "my_lib.cc" ]

  deps = [
    "//third_party/abseil-cpp:absl_base_static"
  ]
}
```

**注意**:
- `absl_base_static` 包含所有 abseil 组件
- 单体静态库，但可能增加二进制大小

---

## 构建验证

### 检查构建命令

```bash
# 构建所有 abseil-cpp 目标
./build.sh --product-name <product> --build-target ohos_shared_library --build-name abseil-cpp

# 构建特定目标
./build.sh --product-name <product> --build-target ohos_shared_library --build-name absl_strings
```

### 验证输出

```bash
# 检查生成的库
find out -name "libabsl*.so"

# 检查静态库
find out -name "libabsl*.a"
```

---

**最后更新**: 2026-02-07
