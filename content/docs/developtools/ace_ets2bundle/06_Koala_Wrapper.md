# Koala 包装器

## 模块概述

Koala 包装器提供 TypeScript/JavaScript 与 Native C++ 代码的互操作桥梁，是 arkui-plugins 的底层支撑模块。

### 主要组件

| 组件 | 路径 | 职责 |
|------|------|------|
| Native 实现 | `koala-wrapper/native/src/` | C++ 核心功能 |
| N-API 绑定 | `koala-wrapper/koalaui/interop/src/cpp/napi/` | JS/Native 桥接 |
| 构建配置 | `koala-wrapper/native/BUILD.gn` | GN 构建脚本 |

## Native 模块架构

### 源码结构

```
koala-wrapper/native/src/
├── common.cc                 # 核心功能
│   ├── 上下文管理
│   ├── 库加载
│   └── 工具函数
├── bridges.cc                # 手动桥接函数
├── memoryTracker.cc          # 内存追踪
└── generated/
    └── bridges.cc           # 自动生成桥接
```

### 头文件

```
koala-wrapper/native/include/
├── common.h                  # 通用接口
└── memoryTracker.h           # 内存追踪接口
```

## N-API 绑定层

### 核心文件

| 文件 | 职责 |
|------|------|
| `convertors-napi.h` | N-API 类型转换头文件 |
| `convertors-napi.cc` | N-API 实现 + 模块注册 |
| `win-dynamic-node.cc` | Windows 动态加载 |

### 类型转换系统

**convertors-napi.h/cc** 实现了 JS 类型与 Native 类型的双向转换：

```cpp
// 字符串转换
template<>
struct InteropTypeConverter<KStringPtr> {
  static KStringPtr fromJs(napi_env env, napi_value jsValue) {
    // JS String -> Native String
  }

  static napi_value toJs(napi_env env, KStringPtr nativeValue) {
    // Native String -> JS String
  }
};

// 数值转换
template<>
struct InteropTypeConverter<KInt> {
  static KInt fromJs(napi_env env, napi_value jsValue);
  static napi_value toJs(napi_env env, KInt nativeValue);
};

// 缓冲区转换
template<>
struct InteropTypeConverter<KInteropBuffer> {
  static KInteropBuffer fromJs(napi_env env, napi_value jsValue);
  static napi_value toJs(napi_env env, KInteropBuffer nativeValue);
};
```

### 模块注册

**convertors-napi.cc:389** - 模块注册入口：

```cpp
// NAPI_MODULE 适配器
#define NAPI_MODULE(modname, regfunc) \
  napi_value __napi_##regfunc(napi_env env, napi_callback_info cbinfo) { \
      napi_value exports; \
      napi_get_named_property(env, cbinfo, "exports", &exports); \
      regfunc(env, exports); \
      return exports; \
  } \
  NODE_API_LINKED(modname)

// 模块注册
NAPI_MODULE(INTEROP_LIBRARY_NAME, InitModule)
```

**模块信息**：
- 模块名：`es2panda` (`INTEROP_LIBRARY_NAME`)
- 命名空间：`NativeModule` (`KOALA_INTEROP_MODULE`)

### 导出函数注册宏

```cpp
// 单参数函数
#define KOALA_INTEROP_1(name, Ret, P0) \
  napi_value Node_##name(napi_env env, napi_callback_info cbinfo) { \
      CallbackInfo info(env, cbinfo); \
      P0 p0 = getArgument<P0>(info, 0); \
      return makeResult<Ret>(info, impl_##name(p0)); \
  }

// 多参数函数
#define KOALA_INTEROP_2(name, Ret, P0, P1) ...
#define KOALA_INTEROP_3(name, Ret, P0, P1, P2) ...

// 调用示例
KOALA_INTEROP_2(CreateConfig, void, KInt, KStringArray)
KOALA_INTEROP_1(DestroyContext, void, KLong)
```

### JS 调用示例

```javascript
// 加载 Native 模块
const koala = require('./es2panda.node');
const NativeModule = koala.NativeModule;

// 创建配置
const config = NativeModule._CreateConfig(argc, argv);

// 创建上下文
const context = NativeModule._CreateContextFromFile(config, filePath);

// 执行编译
NativeModule._ProceedToState(context, targetState);

// 清理
NativeModule._DestroyContext(context);
```

## 核心功能接口

### 上下文管理

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `_CreateConfig` | argc, argv | Config | 创建编译配置 |
| `_DestroyContext` | contextPtr | void | 销毁上下文 |
| `_CreateContextFromFile` | config, file | Context | 从文件创建上下文 |
| `_ProceedToState` | context, state | void | 执行到指定状态 |

### AST 操作

| 函数 | 说明 |
|------|------|
| `_CreateCallExpression` | 创建调用表达式 |
| `_UpdateCallExpression` | 更新调用表达式 |
| `_AstNodeChildren` | 获取 AST 节点子节点 |
| `_GetAstNodeType` | 获取节点类型 |

### 诊断信息

| 函数 | 说明 |
|------|------|
| `_ContextErrorMessage` | 获取上下文错误消息 |
| `_GetAllErrorMessages` | 获取所有错误 |

### 内存管理

| 函数 | 说明 |
|------|------|
| `_MemInitialize` | 初始化内存管理 |
| `_MemFinalize` | 释放内存管理 |
| `_FreeCompilerPartMemory` | 释放编译器部分内存 |

## 内存追踪器

### memoryTracker.h/cc

提供编译过程中的内存使用追踪：

```cpp
class MemoryTracker {
public:
  void StartTracking(const std::string& phase);
  void StopTracking();
  size_t GetCurrentUsage();
  void LogStatistics();
};
```

## 跨平台支持

### Windows 动态加载

**win-dynamic-node.cc** 实现了 Windows 平台的动态 N-API 函数加载：

```cpp
// N-API 函数指针列表
struct NapiFunctionTable {
  const char* name;
  void* address;
};

static NapiFunctionTable napiFunctions[] = {
  {"napi_module_register", nullptr},
  {"napi_define_properties", nullptr},
  {"napi_create_function", nullptr},
  // ... 70+ 个函数
};

// 动态加载
HMODULE nodeModule = LoadLibraryA("node.dll");
for (auto& func : napiFunctions) {
  func.address = GetProcAddress(nodeModule, func.name);
}
```

### 平台标识

| 宏定义 | 平台 |
|-------|------|
| `KOALA_MACOS` | macOS |
| `KOALA_LINUX` | Linux |
| `KOALA_WINDOWS` | Windows |

## 构建配置

### koala-wrapper/native/BUILD.gn

```gn
shared_library("es2panda") {
  sources = [
    "src/common.cc",
    "src/bridges.cc",
    "src/memoryTracker.cc",
    "src/generated/bridges.cc",
  ]

  include_dirs = [
    "//third_party/node-addon-api",
    "//third_party/node-api-headers",
  ]

  external_deps = [
    "ets_frontend:libes2panda_public_headers",
  ]

  defines = [
    "KOALA_NAPI",
    "KOALA_USE_NODE_VM",
  ]
}
```

## 与 arkui-plugins 的集成

### 动态加载

**arkui-plugins/path.ts**：

```typescript
export function getInteropPath(): string {
  return path.join(
    findRootDir(),
    'koala-wrapper/koalaui/interop',
    './dist/lib/src/interop/index.js'
  );
}

export function getArktsPath(): string {
  return path.join(
    findRootDir(),
    'koala-wrapper',
    './build/lib/arkts-api/index.js'
  );
}
```

### 使用示例

```typescript
// 动态加载 Koala 模块
const interop = require(getInteropPath());

// 访问原生指针
const nullptr = interop.nullptr;

// 调用原生函数
const context = interop.NativeModule._CreateContextFromFile(
  config,
  filePath
);
```

## 相关文档

- [架构说明](02_Architecture.md)
- [目录结构](03_Directory_Structure.md)
- [ArkUI 插件系统](05_ArkUI_Plugins.md)
- [GN 构建目标](07_GN_Targets.md)
