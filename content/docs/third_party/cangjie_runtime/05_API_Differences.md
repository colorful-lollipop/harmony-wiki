# 05 - API/接口差异

## 5.1 概述

仓颉标准库在 OpenHarmony 平台上存在部分 API 不支持或行为差异的情况。本文档详细列出这些差异及其原因。

### 差异类型

| 类型 | 说明 | 数量 |
|-----|------|------|
| **不支持** | 使用 `@When[env != "ohos"]` 注解排除 | ~50 个 API |
| **行为差异** | 功能可用但行为与 Linux 不同 | 少量 |
| **平台特定** | OH 特有的扩展 | 少量 |

## 5.2 不支持的 API 清单

### 5.2.1 core 包中的限制

以下 API 在 OpenHarmony 平台**不支持**：

| API | 类型 | 说明 |
|-----|------|------|
| `AtomicInt32.fetchAdd` (非原子版本) | 函数 | 不支持非原子 fetch 操作 |
| `AtomicInt64.fetchAdd` (非原子版本) | 函数 | 不支持非原子 fetch 操作 |
| `WorkerThread` 类 | 类 | 不支持工作线程 |
| `WorkerThread.run` | 方法 | - |
| `WorkerThread.join` | 方法 | - |
| `HeapTracker` 类 | 类 | 不支持堆跟踪 |
| `RawMutex` 类 | 类 | 不支持原始互斥锁 |
| `RawMutex.lock` | 方法 | - |
| `RawMutex.unlock` | 方法 | - |

**原因**: 
- OH 平台暂不支持某些底层同步原语
- 线程模型差异导致 WorkerThread 不可用

### 5.2.2 collection 包中的限制

以下并行流/并发 API 在 OpenHarmony 平台**不支持**：

#### ArrayList

| API | 说明 |
|-----|------|
| `streamPar()` | 并行流 |
| `parallelStream()` | 并行流（别名） |
| `filterPar(predicate)` | 并行过滤 |
| `mapPar(transform)` | 并行映射 |
| `flatMapPar(transform)` | 并行扁平映射 |
| `reducePar(initial, operation)` | 并行归约 |
| `forEachPar(action)` | 并行遍历 |
| `anyMatchPar(predicate)` | 并行任意匹配 |
| `allMatchPar(predicate)` | 并行全匹配 |
| `noneMatchPar(predicate)` | 并行无匹配 |
| `findFirstPar()` | 并行查找首个 |
| `findAnyPar()` | 并行查找任意 |
| `sortedPar()` | 并行排序 |
| `distinctPar()` | 并行去重 |
| `limitPar(maxSize)` | 并行限制 |
| `skipPar(n)` | 并行跳过 |
| `collectPar(supplier, accumulator, combiner)` | 并行收集 |
| `toArrayPar()` | 并行转数组 |
| `minPar(comparator)` | 并行最小值 |
| `maxPar(comparator)` | 并行最大值 |
| `countPar()` | 并行计数 |

#### LinkedList

同上 ArrayList，包含所有并行流相关 API。

#### HashMap

| API | 说明 |
|-----|------|
| `computePar(key, remappingFunction)` | 并行计算 |
| `computeIfAbsentPar(key, mappingFunction)` | 并行条件计算 |
| `computeIfPresentPar(key, remappingFunction)` | 并行条件计算 |
| `mergePar(key, value, remappingFunction)` | 并行合并 |
| `forEachPar(action)` | 并行遍历 |
| `replaceAllPar(function)` | 并行替换 |

#### HashSet

| API | 说明 |
|-----|------|
| `streamPar()` | 并行流 |
| `parallelStream()` | 并行流（别名） |
| `filterPar(predicate)` | 并行过滤 |
| `mapPar(transform)` | 并行映射 |
| `forEachPar(action)` | 并行遍历 |
| `reducePar(initial, operation)` | 并行归约 |
| `collectPar(supplier, accumulator, combiner)` | 并行收集 |
| `toArrayPar()` | 并行转数组 |
| `minPar(comparator)` | 并行最小值 |
| `maxPar(comparator)` | 并行最大值 |
| `countPar()` | 并行计数 |

#### TreeMap

同 HashMap，包含所有并行计算相关 API。

#### TreeSet

同 HashSet，包含所有并行流相关 API。

**原因**:
- 并行流依赖 Fork/Join 框架
- OH 平台暂不支持 Fork/Join 线程池
- 部分底层并发原语在 OH 上有实现限制

### 5.2.3 代码示例

**文件**: `stdlib/libs/std/collection/array_list.cj`

```cangjie
// 并行流 API 仅在非 OHOS 平台可用
@When[env != "ohos"]
public func streamPar(): Stream<T> { ... }

@When[env != "ohos"]
public func filterPar(predicate: (T) -> Bool): Stream<T> { ... }

@When[env != "ohos"]
public func mapPar<R>(transform: (T) -> R): Stream<R> { ... }

// ... 其他并行流 API
```

**使用限制说明**:
```cangjie
// 在 OHOS 平台上，以下代码无法编译：
let list = ArrayList<Int>()
list.append(1)
list.append(2)
list.append(3)

// ❌ 错误：streamPar() 在 OHOS 不可用
let sum = list.streamPar().map({ it * 2 }).reduce(0, { a, b -> a + b })

// ✅ 正确：使用串行流
let sum = list.stream().map({ it * 2 }).reduce(0, { a, b -> a + b })
```

## 5.3 行为差异的 API

### 5.3.1 时区处理

| API | Linux 行为 | OHOS 行为 |
|-----|-----------|-----------|
| `TimeZone.local()` | 读取 `/etc/localtime` | 读取 `/system/etc/zoneinfo` |
| `TimeZone.of(id)` | 从 `/usr/share/zoneinfo` 加载 | 从 `/system/etc/zoneinfo` 加载 |

**适配代码**:

**文件**: `stdlib/libs/std/time/timezone_platform.cj`

```cangjie
// try get ohos timezone first
// OHOS 时区路径: /system/etc/zoneinfo
// Linux 时区路径: /usr/share/zoneinfo
```

### 5.3.2 内存统计

| API | Linux 行为 | OHOS 行为 |
|-----|-----------|-----------|
| `Runtime.getUsedHeapSize()` | 获取仓颉堆物理内存 | 获取仓颉堆物理内存（相同） |
| `Runtime.getTotalMemory()` | 获取进程总内存 | 获取进程总内存（可能受限） |

**文件**: `stdlib/doc/libs/std/runtime/runtime_package_api/runtime_package_funcs.md`

```
功能：在 Linux、macOS、OpenHarmony、HarmonyOS、iOS、Android 
平台下获取仓颉堆实际占用的物理内存大小，单位为 byte。
```

### 5.3.3 进程管理

| API | Linux 行为 | OHOS 行为 |
|-----|-----------|-----------|
| `Process.spawn()` | 使用 `posix_spawn` | 使用 `posix_spawn`（相同） |
| `Process.exec()` | 使用 `execvp` | 使用 `execvp`（相同，但沙箱限制） |

**说明**: OHOS 的沙箱机制可能限制某些进程操作，即使 API 可用。

## 5.4 OH 特有扩展

### 5.4.1 日志系统

虽然这不是公开的 API，但运行时内部使用 OH 特有的日志系统：

```cpp
// runtime/src/Base/Log.cpp
#if defined(__OHOS__) && (__OHOS__ == 1)
    #include "hilog/log.h"
    // 使用 HILOG_INFO/HILOG_ERROR 等
    #define CJ_LOG_DOMAIN 0xD003F00
#else
    // 使用标准输出
#endif
```

### 5.4.2 内存限制

运行时内部针对 OH 设备设置了内存限制：

```cpp
// runtime/src/CangjieRuntimeApi.cpp
#if defined(__OHOS__)
    // use limited heap size in OHOS devices
    size_t maxHeapSize = getLimitedHeapSize();  // 设备相关
#else
    size_t maxHeapSize = getDefaultHeapSize();  // 桌面默认值
#endif
```

## 5.5 替代方案

### 5.5.1 并行流的替代

| 不支持的 API | 替代方案 |
|-------------|---------|
| `streamPar()` | `stream()` |
| `filterPar()` | `filter()` |
| `mapPar()` | `map()` |
| `forEachPar()` | `forEach()` |
| `reducePar()` | `reduce()` |

**性能提示**:
- 串行流在单核或数据量小的情况下性能相近
- 大数据集考虑手动分片 + 多线程处理

### 5.5.2 并发的替代

使用 `std.sync` 中的并发原语：

```cangjie
import std.sync.*
import std.collection.*

// 使用并发集合替代并行流
let queue = ConcurrentQueue<Int>()

// 使用线程池手动实现并行处理
let pool = ThreadPool(4)
// ... 提交任务到线程池
```

## 5.6 API 差异汇总表

### 完全不支持的 API（编译错误）

| 包 | API/类 | 替代方案 |
|---|-------|---------|
| std.core | `WorkerThread` | 使用 `std.sync.Thread` |
| std.core | `HeapTracker` | 使用运行时日志 |
| std.core | `RawMutex` | 使用 `std.sync.Mutex` |
| std.collection | `*Par` 方法（并行流） | 使用串行流方法 |

### 行为有差异的 API

| 包 | API | 差异说明 |
|---|-----|---------|
| std.time | `TimeZone` | 时区数据库路径不同 |
| std.process | `Process` | 沙箱限制可能影响行为 |

### 平台特定实现

| 组件 | 说明 |
|-----|------|
| 日志 | 内部使用 Hilog 替代 stdout |
| 内存 | 堆大小限制更严格 |
| 加载器 | 使用命名空间隔离 |

## 5.7 条件编译注解说明

### @When 注解

仓颉语言使用 `@When` 注解进行条件编译：

```cangjie
// 仅在非 OHOS 平台编译
@When[env != "ohos"]
public func someAPI(): Unit { ... }

// 仅在 OHOS 平台编译
@When[env == "ohos"]
public func ohosSpecificAPI(): Unit { ... }

// 多条件
@When[env == "linux" || env == "macos"]
public func desktopOnlyAPI(): Unit { ... }
```

### 平台标识

| 平台 | env 值 |
|-----|--------|
| Linux | `"linux"` |
| macOS | `"macos"` |
| Windows | `"windows"` |
| OpenHarmony | `"ohos"` |
| Android | `"android"` |
| iOS | `"ios"` |

## 5.8 开发建议

### 跨平台兼容性

编写跨平台仓颉代码时：

```cangjie
// 推荐：使用条件编译为 OHOS 提供降级方案
@When[env != "ohos"]
public func processInParallel(data: ArrayList<T>): Unit {
    // 使用并行流（高性能）
    data.streamPar().forEach({ ... })
}

@When[env == "ohos"]
public func processInParallel(data: ArrayList<T>): Unit {
    // OHOS 降级方案（串行处理）
    data.stream().forEach({ ... })
}
```

### 平台检测

```cangjie
// 运行时平台检测（如需要）
public func getPlatform(): String {
    // 返回当前平台标识
    return "ohos"  // 或 "linux", "macos" 等
}
```

### 文档查阅

开发时参考官方文档的平台支持说明：

```
标准库 API 文档中标记：
> 不支持平台：OpenHarmony

表示该 API 在 OHOS 上不可用。
```

## 5.9 未来计划

根据官方路线图，以下功能计划在 OH 平台支持：

| 功能 | 计划时间 | 状态 |
|-----|---------|------|
| ARM32 (arm) 架构 | 2025 Q4 | 计划中 |
| 完整反射支持 | 2025 Q4 | 计划中 |
| 动态加载 | 2025 Q4 | 计划中 |
| 部分编译器优化 | 2025 Q4 | 计划中 |

---

*本文档基于 cangjie_runtime 1.1.0-alpha.69 版本编写*
*部分 API 限制可能随版本更新而变化，请参考最新官方文档*
