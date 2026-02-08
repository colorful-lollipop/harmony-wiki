# 概览

> JSVM 项目定位、边界、核心能力、运行环境、关键概念

## 目的与适用范围

### 目的
本文档为新人提供 JSVM 项目的高层理解，包括：
- 项目定位与边界
- 核心能力与特性
- 运行环境与依赖
- 关键概念与术语

### 适用范围
- 刚接触 JSVM 代码的开发者
- 需要集成 JSVM 的系统架构师
- 性能优化与安全评审人员

---

## 项目定位

### 什么是 JSVM？

**JSVM（JavaScript Virtual Machine）** 是 OpenHarmony 提供的 JavaScript 虚拟机，基于标准 JS 引擎（V8）封装，提供稳定、完整的 JS 引擎能力。

**证据位置**:
- 项目根目录: `README.md:3-8`
- 公共头文件: `interface/kits/jsvm.h:19-30`

### 核心定位
JSVM 的核心定位是：
1. **独立**: 可作为独立模块集成到应用
2. **标准**: 遵循标准的 JavaScript 引擎能力
3. **完整**: 提供引擎生命周期管理、代码编译执行、JS/C++ 互操作等能力
4. **稳定**: 提供稳定的 C 语言接口（遵循 C99 标准）

**证据位置**: `interface/kits/jsvm.h:19-30`

### 与 N-API 的关系

JSVM-API 与 N-API 的关系：
- **相似性**: 都提供 JS/C++ 互操作的 C 语言接口
- **独立性**: JSVM 是独立的 VM 实现，不依赖 N-API
- **能力**: JSVM 提供更底层的 VM 控制能力（VM 生命周期、快照等）
- **使用场景**: N-API 用于 ArkTS/Node.js 生态，JSVM 用于独立的 JS 引擎集成

---

## 项目边界

### 包含的能力

JSVM 提供以下核心能力：

#### 1. VM 生命周期管理
- VM 初始化与销毁
- Environment 创建与销毁
- Scope 管理（VM Scope、Env Scope、Handle Scope）
- 快照（Snapshot）创建与恢复

**证据位置**:
- `interface/kits/jsvm.h:323-353` - VM 管理 API
- `interface/kits/jsvm.h:433-476` - Environment 管理 API

#### 2. 代码编译与执行
- JavaScript 代码编译
- 代码缓存（Code Cache）
- 编译优化选项
- 脚本执行

**证据位置**:
- `interface/kits/jsvm.h:489-581` - 编译与执行 API

#### 3. JS/C++ 互操作
- 值类型转换（C/C++ ↔ JavaScript）
- 函数注册与调用
- 属性操作
- 类定义

**证据位置**:
- `interface/kits/jsvm.h:814-2500+` - 互操作 API

#### 4. 调试与性能分析
- Inspector 支持
- CPU Profiler
- Heap Snapshot
- Trace 事件

**证据位置**:
- `interface/kits/jsvm.h:1682-1774` - 调试 API

#### 5. 内存管理
- Heap Statistics
- Memory Pressure 通知
- GC 回调

**证据位置**:
- `interface/kits/jsvm.h:444-471` - Heap Statistics 定义
- `interface/kits/jsvm.h:1650-1681` - 内存管理 API

### 不包含的能力

JSVM **不提供**以下能力（需要依赖其他组件）：
- 文件 I/O（需要依赖 libuv 或平台层）
- 网络 I/O（需要依赖 llhttp 或平台层）
- 线程管理（需要依赖平台层）
- 定时器（需要依赖平台层）
- IPC 通信（需要依赖 OpenHarmony IPC 框架）

---

## 核心能力

### 1. VM 生命周期管理

JSVM 支持完整的 VM 生命周期管理：

#### 初始化
```c
JSVM_Status OH_JSVM_Init(const JSVM_InitOptions* options);
JSVM_Status OH_JSVM_CreateVM(const JSVM_CreateVMOptions* options, JSVM_VM* result);
```

**证据位置**: `interface/kits/jsvm.h:323,333`

#### 销毁
```c
JSVM_Status OH_JSVM_DestroyVM(JSVM_VM vm);
JSVM_Status OH_JSVM_DestroyEnv(JSVM_Env env);
```

**证据位置**: `interface/kits/jsvm.h:353,456`

### 2. 代码编译与执行

#### 编译选项
JSVM 支持多种编译模式：
- 默认编译模式
- 代码缓存消费模式
- 急切编译模式
- 编译 profile 生成/消费模式

**证据位置**: `interface/kits/jsvm_types.h:407-419` - JSVM_CompileMode

#### 代码缓存
```c
JSVM_Status OH_JSVM_CreateCodeCache(JSVM_Env env, JSVM_Script script,
                                   const uint8_t** data, size_t* length);
```

**证据位置**: `interface/kits/jsvm.h:566`

### 3. JS/C++ 互操作

JSVM 提供完整的 JS/C++ 互操作能力：

#### 类型转换
- JavaScript 原始类型 ↔ C/C++ 类型
- JavaScript 对象 ↔ C++ 对象
- TypedArray ↔ C/C++ 缓冲区

**证据位置**: `interface/kits/jsvm.h:1174-2500+` - 类型转换 API

#### 函数注册
```c
JSVM_Status OH_JSVM_CreateFunction(JSVM_Env env,
                                const char* utf8name,
                                size_t length,
                                JSVM_Callback callback,
                                void* data,
                                JSVM_Value* result);
```

**证据位置**: `interface/kits/jsvm.h:1832`

#### 类定义
```c
JSVM_Status OH_JSVM_DefineClass(JSVM_Env env,
                                const char* utf8name,
                                size_t length,
                                JSVM_Callback constructor,
                                void* data,
                                size_t propertyCount,
                                const JSVM_PropertyDescriptor* properties,
                                JSVM_Value* result);
```

**证据位置**: `interface/kits/jsvm.h:1906`

### 4. 调试与性能分析

#### Inspector
```c
JSVM_Status OH_JSVM_OpenInspector(JSVM_Env env, const char* host, uint16_t port);
JSVM_Status OH_JSVM_CloseInspector(JSVM_Env env);
JSVM_Status OH_JSVM_WaitForDebugger(JSVM_Env env, bool breakNextLine);
```

**证据位置**: `interface/kits/jsvm.h:1719,1736,1747`

#### CPU Profiler
```c
JSVM_Status OH_JSVM_StartCpuProfiler(JSVM_VM vm, JSVM_CpuProfiler* result);
JSVM_Status OH_JSVM_StopCpuProfiler(JSVM_VM vm, JSVM_CpuProfiler profiler);
```

**证据位置**: `interface/kits/jsvm.h:1682,1693`

---

## 运行环境

### 系统要求

**证据位置**: `bundle.json:19-21` - adapted_system_type

- **系统类型**: Standard（标准系统）
- **最小 ROM**: 5120 KB
- **最小 RAM**: 10240 KB
- **API Level**: 11+（OpenHarmony API 版本）

### 依赖组件

JSVM 依赖以下 OpenHarmony 组件：

**证据位置**: `bundle.json:24-38` - deps

| 组件 | 用途 |
|------|------|
| bounds_checking_function | 边界检查（安全） |
| hilog | 日志系统 |
| hisysevent | 系统事件 |
| hitrace | 性能追踪 |
| hiview | 系统视图 |
| icu | 国际化支持 |
| init | 初始化框架 |
| libuv | 异步 I/O（可选） |
| nghttp2 | HTTP/2 支持 |
| openssl | 加密库 |
| resource_schedule_service | 资源调度 |
| zlib | 压缩库 |

### 第三方依赖

JSVM 集成了以下第三方库：

**证据位置**: `BUILD.gn:28-95`

| 库 | 版本 | 用途 |
|----|------|------|
| V8 | [TODO: 需确认] | JavaScript 引擎核心 |
| llhttp | [TODO: 需确认] | HTTP 解析器 |

---

## 关键概念

### VM（Virtual Machine）

**定义**: JavaScript 虚拟机实例，代表一个独立的 JS 执行环境。

**类型**: `JSVM_VM` (`interface/kits/jsvm_types.h:66`)

**生命周期**:
1. 通过 `OH_JSVM_Init()` 初始化全局状态
2. 通过 `OH_JSVM_CreateVM()` 创建 VM 实例
3. 通过 `OH_JSVM_DestroyVM()` 销毁 VM

**证据位置**:
- `interface/kits/jsvm_types.h:66` - 类型定义
- `interface/kits/jsvm.h:323,333,353` - 生命周期管理 API

### Environment（执行环境）

**定义**: 在 VM 内的执行上下文，代表一个隔离的 JS 作用域。

**类型**: `JSVM_Env` (`interface/kits/jsvm_types.h:94`)

**生命周期**:
1. 通过 `OH_JSVM_CreateEnv()` 创建
2. 通过 `OH_JSVM_DestroyEnv()` 销毁
3. 可以通过 `OH_JSVM_CreateEnvFromSnapshot()` 从快照创建

**证据位置**:
- `interface/kits/jsvm_types.h:94` - 类型定义
- `interface/kits/jsvm.h:433,447,456` - 生命周期管理 API

### Scope（作用域）

**定义**: 用于管理资源生命周期的 RAII（Resource Acquisition Is Initialization）机制。

**类型**:
- `JSVM_VMScope` - VM 作用域
- `JSVM_EnvScope` - Environment 作用域
- `JSVM_HandleScope` - 句柄作用域
- `JSVM_EscapableHandleScope` - 可逃逸句柄作用域

**证据位置**: `interface/kits/jsvm_types.h:73,80,129,136`

### Value（值）

**定义**: JavaScript 值的句柄，代表任何 JavaScript 类型的值。

**类型**: `JSVM_Value` (`interface/kits/jsvm_types.h:108`)

**类型检查**:
- `OH_JSVM_Typeof()` - 获取类型
- `OH_JSVM_IsUndefined()`, `OH_JSVM_IsNull()`, `OH_JSVM_IsBoolean()` 等 - 类型检查

**证据位置**:
- `interface/kits/jsvm_types.h:108` - 类型定义
- `interface/kits/jsvm_types.h:219-240` - ValueType 枚举

### Reference（引用）

**定义**: 对 JSVM_Value 的强引用，延长对象的生命周期。

**类型**: `JSVM_Ref` (`interface/kits/jsvm_types.h:122`)

**生命周期**:
1. 通过 `OH_JSVM_CreateReference()` 创建
2. 通过 `OH_JSVM_ReferenceRef()` 增加引用计数
3. 通过 `OH_JSVM_ReferenceUnref()` 减少引用计数
4. 通过 `OH_JSVM_DeleteReference()` 删除引用

**证据位置**:
- `interface/kits/jsvm_types.h:122` - 类型定义
- `interface/kits/jsvm.h:823,853,865,877` - 引用管理 API

### Callback（回调）

**定义**: 从 C/C++ 导出到 JavaScript 的函数。

**类型**: `JSVM_Callback` (`interface/kits/jsvm_types.h:167`)

**结构**:
```c
typedef struct {
    JSVM_Value(JSVM_CDECL* callback)(JSVM_Env env, JSVM_CallbackInfo info);
    void* data;
} JSVM_CallbackStruct;
```

**证据位置**: `interface/kits/jsvm_types.h:157-167`

---

## 相关链接

- [目录结构与模块职责](./02_Directory_Structure.md) - 了解代码组织
- [对外 API 总览](./03_Public_API_Overview.md) - 掌握 API 面貌
- [架构说明](./04_Architecture.md) - 理解系统架构
