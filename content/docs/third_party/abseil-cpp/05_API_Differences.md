# API 差异（OpenHarmony 与上游对比）

> 本文档说明 Abseil-CPP 在 OpenHarmony 中与上游版本的 API 和行为差异。

---

## 概述

### 差异类型

| 类型 | 数量 | 说明 |
|------|------|------|
| **禁用的 API** | 0 | 没有完全禁用的公共 API |
| **行为变更** | 0 | 公共 API 行为与上游一致 |
| **平台限制** | 7 | 通过 `__OHOS__` 宏影响的功能 |

### 关键结论

- **公共 API 完全兼容**: 所有公共头文件和接口与上游一致
- **内部实现差异**: 调试和符号化功能受限
- **行为一致性**: 可用功能的行为与上游相同

---

## 禁用/受限的功能

### 1. 堆栈跟踪（Stack Trace）

**上游 API**:
```cpp
#include "absl/debugging/stacktrace.h"

// 获取当前堆栈跟踪
int stack_size = absl::GetStackTrace(stack, max_depth);
```

**OH 限制**:
```cpp
// 返回空堆栈（使用 unimplemented stub）
int stack_size = absl::GetStackTrace(stack, max_depth);
// stack_size == 0, 无实际堆栈信息
```

**影响范围**:
- `absl::GetStackTrace()` - 返回 0
- `absl::debugging_internal::GetStackFrames()` - 无实现

**代码位置**:
- `absl/debugging/internal/stacktrace_config.h` - 选择 `stacktrace_unimplemented-inl.inc`

**升级建议**:
- 考虑实现 OH 原生堆栈跟踪 API
- 使用 OH 调试工具或内核接口

---

### 2. 符号化（Symbolization）

**上游 API**:
```cpp
#include "absl/debugging/symbolize.h"

// 符号化地址
const char* symbol = absl::Symbolize(addr);
```

**OH 限制**:
```cpp
// 可能返回 demangled 或 undemangled 地址
// 如果 OH libc++ 不支持 abi::__cxa_demangle，返回简单地址
const char* symbol = absl::Symbolize(addr);
// 符号可能不是人类可读的函数名
```

**影响范围**:
- `absl::Symbolize()` - 返回简化格式
- `absl::SymbolizeWithDemangler()` - 不使用 `abi::__cxa_demangle`

**代码位置**:
- `absl/base/config.h` - `ABSL_INTERNAL_HAS_CXA_DEMANGLE 0`

**行为差异**:
- 上游: 返回格式化的函数名（如 `MyFunction`）
- OH: 可能返回十六进制地址或 mangled 名称

**升级建议**:
- 验证 OH libc++ 是否支持 `abi::__cxa_demangle`
- 如果支持，可移除 `#define ABSL_INTERNAL_HAS_CXA_DEMANGLE 0`

---

### 3. 信号处理（Signal Handling）

**上游 API**:
```cpp
#include "absl/debugging/failure_signal_handler.h"

// 安装失败信号处理程序
absl::InstallFailureSignalHandler();
```

**OH 限制**:
```cpp
// 基本功能可用，但内部实现受限
absl::InstallFailureSignalHandler();
// VMA 命名功能被禁用
```

**影响范围**:
- VMA 命名（`PR_SET_VMA_ANON_NAME`）- 不工作
- ucontext 检查 - 不工作

**代码位置**:
- `absl/debugging/failure_signal_handler.cc` - 排除 `sys/prctl.h`
- `absl/debugging/internal/examine_stack.cc` - 排除 `sys/ucontext.h`

**功能差异**:
- 上游: 可以在 `/proc/$PID/smaps` 中看到命名的内存区域
- OH: 内存区域未命名（VMA 功能禁用）

**升级建议**:
- 除非 OH 添加 `prctl` 支持，否则保持现状

---

### 4. CPU 特性检测（CPU Detection）

**上游 API**:
```cpp
#include "absl/crc/internal/cpu_detect.h"

// 检测 CPU 支持
bool has_crc32 = absl::HasCrc32Instruction();
bool has_aes = absl::HasAesInstruction();
```

**OH 限制**:
```cpp
// 返回保守默认值
bool has_crc32 = absl::HasCrc32Instruction();
// 可能返回 false（即使硬件支持）
```

**影响范围**:
- `absl::internal::CpuHasFeature()` - 基于保守假设
- 性能优化可能未完全利用

**代码位置**:
- `absl/crc/internal/cpu_detect.cc` - 禁用 `getauxval` 和 `/proc/cpuinfo`

**行为差异**:
- 上游: 检测实际 CPU 特性，使用优化的 SIMD 实现
- OH: 使用通用实现，性能可能较低

**升级建议**:
- 研究是否有 OH 特定的 CPU 特性检测 API
- 如果有，可添加 `__OHOS__` 分支使用原生方法

---

### 5. ELF 内存映像（ELF Memory Image）

**上游 API**:
```cpp
#include "absl/debugging/elf_mem_image.h"

// 访问 ELF 内存映像
const ElfMemImage& image = GetElfMemImage();
```

**OH 限制**:
```cpp
// ELF 内存映像功能不可用
// 可能返回空或默认实现
const ElfMemImage& image = GetElfMemImage();
// 功能受限
```

**影响范围**:
- `ABSL_HAVE_ELF_MEM_IMAGE` - 定义为 0
- 符号化功能受限

**代码位置**:
- `absl/debugging/internal/elf_mem_image.h` - 添加 `!defined(__OHOS__)`

**行为差异**:
- 上游: 可以解析 ELF 段和符号
- OH: 不支持 ELF 内存映像访问

**升级建议**:
- 除非 OH 使用 ELF，否则保持现状

---

## 完全可用的 API

### 基础设施（absl/base）

所有基础 API 完全可用：

| API | 功能 | OH 状态 |
|------|------|----------|
| `absl::Cleanup` | 作用域退出回调 | ✅ |
| `absl::GetFlag()` | 命令行标志 | ✅ |
| `absl::SetFlag()` | 命令行标志 | ✅ |
| `absl::Log()` | 日志宏 | ✅ |
| `absl::CHECK()` | 断言宏 | ✅ |

### 字符串处理（absl/strings）

所有字符串 API 完全可用：

| API | 功能 | OH 状态 |
|------|------|----------|
| `absl::string_view` | 字符串视图 | ✅ |
| `absl::StrCat()` | 字符串拼接 | ✅ |
| `absl::StrSplit()` | 字符串分割 | ✅ |
| `absl::StrFormat()` | 格式化字符串 | ✅ |
| `absl::Cord` | 增量字符串 | ✅ |

### 并发控制（absl/synchronization）

所有并发 API 完全可用：

| API | 功能 | OH 状态 |
|------|------|----------|
| `absl::Mutex` | 互斥锁 | ✅ |
| `absl::ReaderWriterLock` | 读写锁 | ✅ |
| `absl::CondVar` | 条件变量 | ✅ |
| `absl::Notification` | 线程通知 | ✅ |
| `absl::Barrier` | 屏障同步 | ✅ |

### 时间处理（absl/time）

所有时间 API 完全可用：

| API | 功能 | OH 状态 |
|------|------|----------|
| `absl::Now()` | 当前时间 | ✅ |
| `absl::Time` | 绝对时间点 | ✅ |
| `absl::Duration` | 时间段 | ✅ |
| `absl::TimeZone` | 时区 | ✅ |
| `absl::CivilTime` | 民用时间 | ✅ |

### 容器（absl/container）

所有容器 API 完全可用：

| API | 功能 | OH 状态 |
|------|------|----------|
| `absl::flat_hash_map` | 平面哈希表 | ✅ |
| `absl::flat_hash_set` | 平面哈希集合 | ✅ |
| `absl::node_hash_map` | 节点哈希表 | ✅ |
| `absl::node_hash_set` | 节点哈希集合 | ✅ |

### 错误处理（absl/status）

所有错误处理 API 完全可用：

| API | 功能 | OH 状态 |
|------|------|----------|
| `absl::Status` | 错误状态 | ✅ |
| `absl::StatusOr<T>` | 带值的错误 | ✅ |
| `absl::StatusCode` | 错误码 | ✅ |

---

## 行为差异总结

### 性能差异

| 功能 | 上游 | OH | 影响 |
|------|------|-----|------|
| CRC32 计算 | SIMD 优化（如果 CPU 支持） | 通用实现 | 可能较慢 |
| 字符串操作 | 相同 | 相同 | 无差异 |
| 容器操作 | Swiss table 优化 | Swiss table 优化 | 无差异 |

### 调试差异

| 功能 | 上游 | OH | 影响 |
|------|------|-----|------|
| 堆栈跟踪 | 完整实现 | 空实现（unimplemented） | 无法获取调用堆栈 |
| 符号化 | demangled 名称 | 简化格式（如果禁用 demangling） | 错误信息可读性降低 |
| VMA 命名 | 支持命名内存区域 | 不支持 | `/proc/$PID/smaps` 中无额外信息 |

### 内存差异

| 功能 | 上游 | OH | 影响 |
|------|------|-----|------|
| ELF 内存映像 | 支持解析 | 不支持 | 符号化功能受限 |
| prctl VMA 命名 | 支持命名 | 不支持 | 内存区域不可命名 |

---

## 兼容性矩阵

| API 类别 | 兼容性 | 说明 |
|---------|--------|------|
| **公共 API** | ✅ 完全兼容 | 所有公共头文件和接口可用 |
| **内部 API** | ⚠️ 部分受限 | 调试和符号化内部实现受限 |
| **宏定义** | ✅ 兼容 | LOG、CHECK 等宏正常工作 |
| **类型定义** | ✅ 兼容 | Status、string_view 等类型正常工作 |
| **模板** | ✅ 兼容 | 所有模板正常实例化 |

---

## 代码示例对比

### 堆栈跟踪

**上游代码**（Linux）:
```cpp
#include "absl/debugging/stacktrace.h"

void MyCrashHandler() {
    void* stack[32];
    int depth = absl::GetStackTrace(stack, 32);
    for (int i = 0; i < depth; ++i) {
        LOG(INFO) << "  " << i << ": " << stack[i];
    }
}
// 输出: 有意义的函数名和行号
```

**OH 代码**（受限）:
```cpp
#include "absl/debugging/stacktrace.h"

void MyCrashHandler() {
    void* stack[32];
    int depth = absl::GetStackTrace(stack, 32);
    if (depth == 0) {
        LOG(ERROR) << "Stack trace not available on this platform";
    }
}
// 输出: 无堆栈信息
```

### 符号化

**上游代码**（Linux）:
```cpp
#include "absl/debugging/symbolize.h"

void PrintAddress(void* addr) {
    const char* symbol = absl::Symbolize(addr);
    if (symbol != nullptr && symbol[0] != '\0') {
        LOG(INFO) << "Address: " << symbol;  // 例如: MyFunction
    }
}
```

**OH 代码**（可能）:
```cpp
#include "absl/debugging/symbolize.h"

void PrintAddress(void* addr) {
    const char* symbol = absl::Symbolize(addr);
    if (symbol != nullptr) {
        LOG(INFO) << "Address: " << symbol;  // 可能是 0x12345678 或 mangled 名称
    }
}
// 输出: 可能不是人类可读的函数名
```

---

## 升级建议

### 检查清单

升级 abseil-cpp 版本时，检查以下方面：

1. **公共 API 兼容性**
   - 所有公共头文件可用
   - 接口签名未改变
   - 行为保持一致

2. **`__OHOS__` 宏有效性**
   - 所有现有的 `__OHOS__` 适配仍然有效
   - 没有新的平台假设冲突

3. **新增 OH 支持**
   - 上游是否添加了 `__OHOS__` 原生支持？
   - 是否有新的 OH 特定实现？

4. **测试关键模块**
   - profiler/hiperf（堆栈跟踪禁用影响）
   - gRPC（符号化禁用影响）
   - protobuf（基础库兼容性）

### 潜在改进

| 改进 | 收益 | 优先级 |
|------|------|--------|
| 实现 OH 原生堆栈跟踪 | 显著提升调试能力 | 高 |
| 启用符号还原 | 改善错误信息 | 中 |
| 添加 CPU 特性检测 API | 恢复性能优化 | 中 |
| 实现 ELF 内存映像支持 | 增强符号化功能 | 低 |

---

**最后更新**: 2026-02-07
