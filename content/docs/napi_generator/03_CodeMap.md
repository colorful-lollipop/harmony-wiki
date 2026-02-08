# 代码导航图

> 关键功能到文件路径的快速导航指南

## 目录

1. [核心工具入口](#1-核心工具入口)
2. [N-API 生成逻辑](#2-n-api-生成逻辑)
3. [SA 服务生成](#3-sa-服务生成)
4. [类型映射与转换](#4-类型映射与转换)
5. [工具类与辅助函数](#5-工具类与辅助函数)
6. [模板系统](#6-模板系统)
7. [文件操作](#7-文件操作)

---

## 1. 核心工具入口

### CLI 工具入口点

| 工具 | 入口文件 | 行号 | 职责 |
|------|----------|------|------|
| **dts2cpp** | `src/cli/dts2cpp/src/gen/cmd_gen.js` | 1-144 | 命令行参数解析、文件遍历 |
| **dts2cpp** | `src/cli/dts2cpp/src/gen/main.js` | - | 主生成流程控制 |
| **h2dts** | `src/cli/h2dts/src/tsGen/tsMain.js` | 1-600+ | TS 生成主逻辑 |
| **h2dtscpp** | `src/cli/h2dtscpp/src/src/main.js` | 1-100+ | CLI 入口、目录创建 |
| **h2sa** | `src/cli/h2sa/src/gen/main.js` | 1-100+ | SA 生成主入口 |
| **cmake2gn** | `src/cli/cmake2gn/src/main.js` | - | CMake 解析入口 |

### VSCode 插件入口

| 组件 | 文件路径 | 职责 |
|------|----------|------|
| **激活函数** | `src/vscode_plugin/src/extension.ts` | 747 行，注册 7 个命令 |
| **dts2cpp 命令** | `src/vscode_plugin/src/gen/gencpp.ts` | 生成 N-API C++ |
| **h2dts 命令** | `src/vscode_plugin/src/gen/gendts.ts` | 生成 TS 声明 |
| **h2sa 命令** | `src/vscode_plugin/src/gen/gensa.ts` | 生成 SA 框架 |

---

## 2. N-API 生成逻辑

### 2.1 主生成器

| 功能 | 文件路径 | 关键函数/行号 |
|------|----------|---------------|
| **主生成流程** | `src/cli/dts2cpp/src/gen/generate.js` | `doGenerate()` |
| **命名空间生成** | `src/cli/dts2cpp/src/gen/generate/namespace.js` | `generateNamespace()` |
| **接口生成** | `src/cli/dts2cpp/src/gen/generate/interface.js` | `generateInterface()` |
| **类生成** | `src/cli/dts2cpp/src/gen/generate/class.js` | `generateClass()` |
| **枚举生成** | `src/cli/dts2cpp/src/gen/generate/enum.js` | `generateEnum()` |

### 2.2 函数类型生成

| 函数类型 | 文件路径 | 关键函数 |
|----------|----------|----------|
| **直接调用** | `src/cli/dts2cpp/src/gen/generate/function_direct.js` | `generateFunctionDirect()` |
| **同步回调** | `src/cli/dts2cpp/src/gen/generate/function_sync.js` | `generateFunctionSync()` |
| **异步/Promise** | `src/cli/dts2cpp/src/gen/generate/function_async.js` | `generateFunctionAsync()` |
| **on/off 事件** | `src/cli/dts2cpp/src/gen/generate/function_onoff.js` | `generateFunctionOnOff()` |
| **线程安全** | `src/cli/dts2cpp/src/gen/generate/function_threadsafe.js` | `generateThreadsafeFunc()` |

### 2.3 参数与返回值

| 功能 | 文件路径 | 关键函数 |
|------|----------|----------|
| **参数生成** | `src/cli/dts2cpp/src/gen/generate/param_generate.js` | `paramGenerate()`, `jsToC()` |
| **返回值生成** | `src/cli/dts2cpp/src/gen/generate/return_generate.js` | `returnGenerate()`, `cToJs()` |

### 2.4 模块注册

**位置**: `src/cli/dts2cpp/src/gen/generate.js:118-131`

```cpp
// 生成的 N-API 模块注册代码模板
static napi_module g_[implName]_Module = {
    .nm_version = 1,
    .nm_register_func = init,
    .nm_modname = "[modulename]",
};
```

---

## 3. SA 服务生成

### 3.1 主生成器

| 功能 | 文件路径 | 关键函数 |
|------|----------|----------|
| **主生成流程** | `src/cli/h2sa/src/gen/generate.js` | `doGenerate()` |
| **头文件解析** | `src/cli/h2sa/src/gen/analyze.js` | `doAnalyze()` |
| **代码模板** | `src/cli/h2sa/src/gen/file_template.js` | 模板定义 |

### 3.2 生成产物

| 产物 | 模板位置 | 说明 |
|------|----------|------|
| **接口定义** | `file_template.js` | `I{className}Service` |
| **Proxy** | `file_template.js` | `{className}Proxy` |
| **Stub** | `file_template.js` | `{className}Stub` |
| **Service** | `file_template.js` | `{className}Service` |

### 3.3 MessageParcel 映射

**位置**: `src/cli/h2sa/src/tools/common.js`

```javascript
const DATA_W_MAP = {
    'int32_t': 'WriteInt32',
    'std::string': 'WriteString',
    // ...
};
```

---

## 4. 类型映射与转换

### 4.1 TypeScript → C++ 映射

| TS 类型 | C++ 类型 | 映射位置 |
|---------|----------|----------|
| `number` | `uint32_t/int32_t/int64_t/double` | `function_direct.js:118` |
| `string` | `std::string` | `tool_utility.js` |
| `boolean` | `bool` | `tool_utility.js:322` |
| `Array<T>` | `std::vector<T>` | `param_generate.js` |
| `Map<K,V>` | `std::map<K,V>` | `return_generate.js` |

### 4.2 XNapiTool 工具类

**位置**: `src/cli/dts2cpp/src/gen/extend/tool_utility.js`

| 方法类别 | 方法名 | 行号 |
|----------|--------|------|
| **JS→C 转换** | `SwapJs2CInt32()` | 1301 |
| | `SwapJs2CUint32()` | 1310 |
| | `SwapJs2CInt64()` | 1319 |
| | `SwapJs2CDouble()` | 1328 |
| | `SwapJs2CUtf8()` | 1338 |
| **C→JS 转换** | `SwapC2JsInt32()` | 1356 |
| | `SwapC2JsUint32()` | 1364 |
| | `SwapC2JsInt64()` | 1372 |
| | `SwapC2JsDouble()` | 1380 |
| | `SwapC2JsUtf8()` | 1388 |
| **异步支持** | `StartAsync()` | 1442-1468 |
| | `FinishAsync()` | - |

---

## 5. 工具类与辅助函数

### 5.1 文件操作

| 功能 | 文件路径 | 关键函数 |
|------|----------|----------|
| **文件读取** | `src/cli/dts2cpp/src/gen/tools/FileRW.js:128` | `readFile()` |
| **文件写入** | `src/cli/dts2cpp/src/gen/tools/FileRW.js:134` | `writeFile()` |
| **路径处理** | `src/cli/dts2cpp/src/gen/tools/re.js:71` | `pathJoin()` |

### 5.2 日志记录

| 工具 | 文件路径 | 关键函数 |
|------|----------|----------|
| **NapiLog** | `src/cli/dts2cpp/src/gen/tools/NapiLog.js` | `logInfo()`, `logError()` |

### 5.3 通用工具

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **正则工具** | `src/cli/dts2cpp/src/gen/tools/re.js` | 正则表达式封装 |
| **通用函数** | `src/cli/dts2cpp/src/gen/tools/common.js` | 常用辅助函数 |

---

## 6. 模板系统

### 6.1 dts2cpp 模板

| 模板 | 文件路径 | 说明 |
|------|----------|------|
| **工具类头文件** | `tool_utility.js:1783` | XNapiTool.h 模板 |
| **工具类实现** | `tool_utility.js:1784` | XNapiTool.cpp 模板 |
| **BUILD.gn** | `build_gn.js:61` | GN 构建模板 |
| **binding.gyp** | `binding_gyp.js:45` | node-gyp 模板 |

### 6.2 h2dtscpp 模板

**位置**: `src/cli/h2dtscpp/src/src/json/directFunction/`

| 模板 | 路径 | 说明 |
|------|------|------|
| **函数体** | `cppTempleteDetails/funcBody/` | C++ 函数体模板 |
| **参数输入** | `funcBody/funcParamIn/` | 参数获取模板 |
| **返回值** | `funcBody/funcReturnOut/` | 返回值处理模板 |
| **初始化** | `initTempleteDetails/` | 模块初始化模板 |

### 6.3 VSCode 插件模板

**位置**: `src/vscode_plugin/src/template/`

| 模板 | 文件路径 | 说明 |
|------|----------|------|
| **函数模板** | `func_template.ts` | N-API 函数模板 |
| **dts2cpp 模板** | `dtscpp/` | dts2cpp 专用模板 |
| **SA 模板** | `sa/` | SA 框架模板 |

---

## 7. 文件操作

### 7.1 输入文件处理

| 工具 | 输入类型 | 处理文件 |
|------|----------|----------|
| **dts2cpp** | .d.ts | `analyze.js` |
| **h2dts** | .h | `tsGen/tsMain.js` |
| **h2dtscpp** | .h | `tsGen/tsMain.js` |
| **h2sa** | .h | `gen/analyze.js` |
| **cmake2gn** | CMakeLists.txt | `analyze_cmake.js` |

### 7.2 输出文件生成

| 产物 | 生成位置 | 生成函数 |
|------|----------|----------|
| `{name}_middle.h` | `destDir/` | `generate.js:349` |
| `{name}_middle.cpp` | `destDir/` | `generate.js:315` |
| `{name}.h` | `destDir/` | `generate.js:304` |
| `{name}.cpp` | `destDir/` | `generate.js:282` |
| `tool_utility.h/cpp` | `destDir/` | `tool_utility.js:1783-1784` |
| `BUILD.gn` | `destDir/` | `build_gn.js:61` |

---

## 8. 快速索引

### 8.1 按功能查找

| 你想找 | 去这里 |
|--------|--------|
| CLI 参数解析 | `cmd_gen.js` |
| TS 解析逻辑 | `analyze.js` |
| N-API 代码生成 | `generate.js` + `generate/*.js` |
| 类型转换 | `extend/tool_utility.js` |
| 参数处理 | `generate/param_generate.js` |
| 返回值处理 | `generate/return_generate.js` |
| 文件读写 | `tools/FileRW.js` |
| 日志记录 | `tools/NapiLog.js` |
| GN 生成 | `extend/build_gn.js` |
| SA 生成 | `h2sa/src/gen/generate.js` |

### 8.2 按问题查找

| 问题 | 相关文件 |
|------|----------|
| 函数类型如何映射？ | `function_direct.js:118` |
| 异步函数怎么生成？ | `function_async.js` |
| 数组类型怎么处理？ | `param_generate.js` |
| 对象类型怎么转换？ | `return_generate.js` |
| 模块注册代码在哪？ | `generate.js:130` |
| XNapiTool 有啥方法？ | `tool_utility.js` |

---

## 9. 调用链示例

### 9.1 dts2cpp 完整调用链

```
cmd_gen.js (入口)
    │
    ▼
main.doGenerate()
    │
    ├── analyze.js (解析 .d.ts)
    │   ├── parseNamespace()
    │   ├── parseInterface()
    │   └── parseFunction()
    │
    ▼
generate.js (生成代码)
    │
    ├── generateMiddleH() ──▶ {name}_middle.h
    ├── generateMiddleCpp() ──▶ {name}_middle.cpp
    │   ├── function_direct.js
    │   ├── function_async.js
    │   └── ...
    ├── generateImplH() ──▶ {name}.h
    ├── generateImplCpp() ──▶ {name}.cpp
    └── extend/build_gn.js ──▶ BUILD.gn
```

### 9.2 N-API 函数生成调用链

```
generateFunction() (分发器)
    │
    ├── 直接调用 ──▶ function_direct.js
    ├── 同步回调 ──▶ function_sync.js
    ├── 异步/Promise ──▶ function_async.js
    ├── on/off 事件 ──▶ function_onoff.js
    └── 线程安全 ──▶ function_threadsafe.js
```

---

## 相关章节

- 目录结构: [02_Directory_Structure.md](02_Directory_Structure.md)
- 架构说明: [03_Architecture.md](03_Architecture.md)
- 内部 API: [05_Internal_API.md](05_Internal_API.md)

---

[返回 SUMMARY.md](SUMMARY.md)
