# 02 - Patch 详细分析

## 2.1 Patch 清单总览

### 结论

**本项目无任何传统意义上的 Patch 文件。**

| Patch 类型 | 数量 | 说明 |
|-----------|------|------|
| `.patch` 文件 | 0 | 未发现任何 Patch 文件 |
| `patches/` 目录 | 0 | 未发现 Patch 目录 |
| `.diff` 文件 | 0 | 未发现 Diff 文件 |

### 为什么无 Patch？

cangjie_runtime 采用**预编译库（Prebuilt）集成模式**，与传统第三方库的直接源码编译模式不同：

| 模式 | 代表库 | 集成方式 | Patch 位置 |
|-----|-------|---------|-----------|
| **源码编译模式** | curl、openssl、ffmpeg | 源码 + Patch → OH 构建系统编译 | OH 仓库中的 `.patch` 文件 |
| **预编译模式** | cangjie_runtime | 外部构建 → 预编译二进制 → OH 分发 | 上游仓库源码中 |

在预编译模式下：
1. 源码修改发生在**上游仓库**（cangjie-lang.cn 官方仓库）
2. OH 仓库仅包含**预编译的二进制产物**（`.so` 文件）
3. 通过 `BUILD.gn` 引用预编译产物，而非编译源码

## 2.2 源码级 OH 适配分析

虽然无 Patch 文件，但运行时和标准库源码中大量使用了 `__OHOS__` 宏进行条件编译，实现 OH 平台适配。

### OH 条件编译统计

| 宏定义 | 出现次数 | 主要用途 |
|-------|---------|---------|
| `__OHOS__` | 100+ | 运行时核心适配 |
| `__ohos__` | 10+ | 标准库适配 |
| `OHOS_FLAG` | 50+ | 构建系统配置 |
| `is_ohos` / `current_os == "ohos"` | 20+ | GN 构建条件 |

### 关键适配点详解

#### 1. 运行时 API 适配

**文件**: `runtime/src/CangjieRuntimeApi.cpp`

```cpp
// OH 平台堆大小限制
#if defined(__OHOS__) && (__OHOS__ == 1)
// use limited heap size in OHOS devices
size_t maxHeapSize = ... // OH 设备内存受限
#endif

// OH 平台默认栈大小
#if defined(__OHOS__) || defined(__ANDROID__)
size_t defaultStackSize = 1024; // default 1MB in OHOS, measured in KB
#endif
```

**适配目的**: 
- OH 设备通常内存受限，需要限制堆大小
- 与 Android 共用部分资源限制策略

---

#### 2. 异常处理策略差异

**文件**: `runtime/src/ExceptionManager.cpp`

```cpp
#if defined(__OHOS__) && (__OHOS__ == 1) || (__APPLE__)
// OHOS 和 macOS 使用 setjmp/longjmp 实现异常跳转
// ...
#endif

#if defined(__OHOS__) && (__OHOS__ == 1)
// In OHOS, C calling Cangjie: uncaught exception allow C-side execution 
// to continue without proactive exit,
// which is different from Linux.
// This is because OHOS has stricter sandbox mechanism.
#endif
```

**适配目的**:
- OH 的沙箱机制与 Linux 不同
- 未捕获异常时允许 C 端继续执行（不主动退出）

---

#### 3. 加载器命名空间隔离

**文件**: `runtime/src/LoaderManager.cpp`

```cpp
#ifdef __OHOS__
// Due to the namespace isolation mechanism of ohos, the runtime has no
// direct access to application symbols. CreateEnvMethods is used to bridge
// this gap.
const char* createEnvFuncMangledName = "_ZN4OHOS13CJEnvironment16CreateEnvMethodsEv";
#endif
```

**适配目的**:
- OH 使用命名空间隔离机制增强安全性
- 运行时无法直接访问应用符号，需要通过 `CJEnvironment` 桥接

---

#### 4. 日志系统适配（Hilog）

**文件**: `runtime/src/Base/Log.cpp`

```cpp
#if defined(__OHOS__) && (__OHOS__ == 1)
// 使用 OHOS 的 Hilog 日志系统
#include "hilog/log.h"
// 日志输出到 Hilog 而非标准输出
#endif

#if defined(__OHOS__) && (__OHOS__ == 1)
// OHOS 平台使用 Domian ID 管理日志
#define CJ_LOG_DOMAIN 0xD003F00
#endif
```

**文件**: `runtime/src/Base/Print.h`

```cpp
#if (defined(__OHOS__) && (__OHOS__ == 1))
// OHOS 下输出到日志系统而非 stdout/stderr
#endif
```

**适配目的**:
- OH 应用需要统一使用 Hilog 日志系统
- 便于日志收集、过滤和分析

---

#### 5. 内存统计适配

**文件**: `runtime/src/Base/MemUtils.cpp`

```cpp
#if defined(__OHOS__)
// OHOS 内存统计使用 /proc/pid/status 中的特定字段
// 或 OHOS 特有的内存查询接口
#endif
```

**适配目的**:
- OH 的内存统计接口与标准 Linux 有差异

---

#### 6. 信号处理适配

**文件**: `runtime/src/Signal/SignalStack.cpp`

```cpp
#if defined(__OHOS__) || defined(__ANDROID__)
// OHOS 和 Android 使用相同的信号栈处理方式
// ...
#endif
```

**文件**: `runtime/src/SignalManager.cpp`

```cpp
#if !defined(__OHOS__) && !defined(__ANDROID__)
// Linux 特有的信号处理（OHOS 和 Android 不启用）
#endif
```

**适配目的**:
- OH 的信号机制与 Android 更接近，与桌面 Linux 不同

---

#### 7. 栈管理适配

**文件**: `runtime/src/StackManager.cpp`

```cpp
#elif defined(__OHOS__) || defined(__ANDROID__) // OHOS, ANDROID
// OHOS 和 Android 栈管理使用相同的阈值
static constexpr size_t STACK_WARN_THRESHOLD = ...
#endif
```

---

#### 8. 对象模型适配

**文件**: `runtime/src/ObjectModel/MObject.cpp`

```cpp
#if defined(__OHOS__) && (__OHOS__ == 1)
// OHOS 平台特定的对象初始化
#endif
```

**文件**: `runtime/src/ObjectModel/MArray.inline.h`

```cpp
#if defined(__OHOS__) && (__OHOS__ == 1)
// OHOS 数组操作优化
#endif
```

---

#### 9. Panic 处理适配

**文件**: `runtime/src/Base/Panic.h`

```cpp
#if (defined(__OHOS__) && (__OHOS__ == 1))
// OHOS 下 panic 输出到日志系统
#endif
```

---

#### 10. 编译器调用适配

**文件**: `runtime/src/CompilerCalls.h`

```cpp
#ifdef __OHOS__
// OHOS 平台特定的编译器调用路径
#endif
```

---

### 标准库中的 OH 适配

#### 1. 时区处理

**文件**: `stdlib/libs/std/time/timezone_platform.cj`

```cangjie
// try get ohos timezone first
// OHOS 时区路径前缀: /system/etc/zoneinfo/
// Linux 时区路径: /usr/share/zoneinfo/
```

**文件**: `stdlib/libs/std/time/native/timezone.c`

```c
#ifdef __ohos__
// OHOS 使用 /system/etc/zoneinfo 作为时区数据库路径
#else
// Linux 使用 /usr/share/zoneinfo
#endif
```

**适配目的**:
- OH 的时区数据库存放路径与标准 Linux 不同

---

#### 2. 时间获取

**文件**: `stdlib/libs/std/time/native/time_common.c`

```c
#ifdef __ohos__
// OHOS 使用 clock_gettime(CLOCK_REALTIME, ...)
#endif
```

---

#### 3. 进程管理

**文件**: `stdlib/libs/std/process/native/process_ffi_unix.c`

```c
#if defined(__ANDROID__) || defined(__OHOS__)
// Android 和 OHOS 使用相同的进程管理接口
// 如 execvp、posix_spawn 等行为差异
#endif
```

**适配目的**:
- OH 的进程管理与 Android 更相似，与桌面 Linux 有差异

---

#### 4. POSIX 适配

**文件**: `stdlib/libs/std/posix/native/file_dirc.c`

```c
#if defined(__linux__) || defined(__ohos__) || defined(__APPLE__)
// 支持的平台
#endif
```

**文件**: `stdlib/libs/std/posix/native/native.c`

```c
#if defined(__linux__) || defined(__APPLE__) || defined(__ohos__)
// 支持的平台
#endif
```

---

#### 5. 条件编译注解

**文件**: `stdlib/libs/std/collection/array_list.cj`

```cangjie
@When[env != "ohos"]
// 非 OHOS 平台可用的 API
// 主要是并行流相关 API
public func streamPar(...)
```

**涉及文件**:
- `array_list.cj` - ArrayList 并行流 API
- `linked_list.cj` - LinkedList 并行流 API
- `hash_map.cj` - HashMap 并行计算 API
- `hash_set.cj` - HashSet 并行计算 API
- `tree_map.cj` - TreeMap 并行计算 API
- `tree_set.cj` - TreeSet 并行流 API

**适配目的**:
- OH 平台暂不支持完整的并行流和某些底层同步原语
- 使用 `@When[env != "ohos"]` 注解排除这些 API

---

#### 6. 反射与 AST

**文件**: `stdlib/libs/std/ast/native/BUILD.gn`

```gn
if (is_ohos) {
  defines += [ "__OHOS__" ]
}
```

**适配目的**:
- AST 模块在 OH 下需要特殊定义

---

## 2.3 构建系统适配

### CMake 工具链

三个 OH 专用工具链文件：

| 文件 | 目标平台 |
|-----|---------|
| `stdlib/cmake/ohos_aarch64_clang_toolchain.cmake` | OHOS ARM64 |
| `stdlib/cmake/ohos_arm_clang_toolchain.cmake` | OHOS ARM32 |
| `stdlib/cmake/ohos_x86_64_clang_toolchain.cmake` | OHOS x86_64 |

**关键配置**:
```cmake
set(TRIPLE aarch64-linux-ohos)  # 或 arm-linux-ohos、x86_64-linux-ohos
set(OHOS ON)
add_compile_definitions(__ohos__)
```

### GN 构建规则

**文件**: `platform.gni`

```gn
if (current_os == "ohos") {
  cj_config.dyn_extension = "so"
  if (target_cpu == "arm64") {
    cj_config.platform_target = "linux_ohos_aarch64_cjnative"
  } else if (target_cpu == "x86_64") {
    cj_config.platform_target = "linux_ohos_x86_64_cjnative"
  } else if (target_cpu == "arm") {
    cj_config.platform_target = "linux_ohos_arm_cjnative"
  }
}
```

**文件**: `runtime/runtime_config.gni`

```gn
if(current_os == "ohos") {
  if (target_cpu == "arm64") {
    OHOS_FLAG = 1
    TARGET_ARCH = "aarch64"
  } else if (target_cpu == "x86_64") {
    OHOS_FLAG = 2
    TARGET_ARCH = "x86_64"
  }
}
```

---

## 2.4 升级注意事项

### 对于维护者

由于本项目采用预编译模式，升级流程与传统 Patch 模式不同：

#### 升级流程

```
1. 上游仓库发布新版本
        ↓
2. 同步源码到内部构建系统
        ↓
3. 验证所有 __OHOS__ 适配点是否仍然有效
        ↓
4. 重新交叉编译所有 OH 平台（aarch64、x86_64、arm）
        ↓
5. 更新预编译 SDK 仓库
        ↓
6. 更新 OH 仓库 bundle.json 版本号
        ↓
7. 在 OH 设备上验证
```

#### 需要验证的 OH 适配点

升级时需重点检查以下宏定义处：

| 文件 | 检查点 |
|-----|-------|
| `CangjieRuntimeApi.cpp` | 堆大小限制、栈大小 |
| `ExceptionManager.cpp` | 异常处理策略 |
| `LoaderManager.cpp` | 命名空间隔离 |
| `Log.cpp` | Hilog 集成 |
| `Signal*.cpp` | 信号处理 |
| `StackManager.cpp` | 栈管理 |
| `timezone_platform.cj` | 时区路径 |
| `process_ffi_unix.c` | 进程管理 |
| `array_list.cj` 等 | @When 注解 |

#### 风险评估

| 风险等级 | 场景 |
|---------|------|
| **高** | 上游修改了条件编译相关的代码结构（如重命名了 OH 判断的函数） |
| **中** | 上游新增平台相关代码，需要补充 OH 适配 |
| **低** | 纯功能更新，未涉及平台相关代码 |

---

## 2.5 Patch/适配汇总表

| 类型 | 位置 | 数量 | 说明 |
|-----|------|------|------|
| **__OHOS__ 宏** | runtime/src/*.cpp | ~50处 | 运行时核心适配 |
| **__ohos__ 宏** | stdlib/**/*.c | ~10处 | 标准库底层适配 |
| **@When[env != "ohos"]** | stdlib/**/*.cj | ~50处 | 标准库 API 条件编译 |
| **GN 条件** | *.gni, BUILD.gn | ~20处 | 构建系统条件 |
| **CMake 工具链** | stdlib/cmake/*.cmake | 3个 | 交叉编译工具链 |

**总计**：约 130+ 处 OH 特定适配代码

---

## 2.6 结论

### 与传统 Patch 模式的对比

| 特性 | 传统 Patch 模式 | cangjie_runtime 模式 |
|-----|----------------|---------------------|
| Patch 文件 | 有（.patch） | 无 |
| 源码位置 | OH 仓库 | 上游仓库 |
| 修改形式 | Patch 文件打补丁 | 源码中使用条件编译 |
| 升级方式 | 重新打 Patch | 重新编译预编译库 |
| 维护复杂度 | 中（需管理 Patch 冲突） | 低（条件编译更清晰） |
| 源码可读性 | 一般（需看 Patch） | 好（源码中直接可见） |

### 维护建议

1. **升级时**：关注上游 Release Note 中平台相关变更
2. **开发时**：在源码中继续遵循 `__OHOS__` 宏约定
3. **测试时**：确保覆盖所有 OH 架构（aarch64、x86_64、arm）
4. **文档时**：维护 OH 适配点清单，便于升级检查

---

*本文档分析基于 cangjie_runtime 1.1.0-alpha.69 版本*
