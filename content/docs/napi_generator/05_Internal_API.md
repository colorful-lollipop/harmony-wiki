# 内部 API 与模块接口

> 模块接口、依赖方向、稳定性标注

## 模块概览

```
napi-generator 模块划分
│
├── 工具层 (src/cli/)
│   ├── dts2cpp/          # TS → N-API (核心)
│   ├── h2sa/             # SA 服务生成
│   ├── h2dtscpp/         # C++ → TS + NAPI + 测试
│   ├── h2dts/            # C++ → TS
│   ├── cmake2gn/         # CMake → GN
│   └── h2hdf/            # HDF 驱动
│
├── 示例层 (examples/)
│   ├── napitutorials/    # N-API 教程
│   ├── akitutorials/     # AKI 示例
│   └── p7zipTest/        # 完整应用
│
└── 插件层 (src/*_plugin/)
    ├── vscode_plugin/
    └── intellij_plugin/
```

---

## dts2cpp 模块接口

### 1. 公共接口文件

| 文件路径 | 导出接口 | 职责 |
|----------|----------|------|
| `src/cli/dts2cpp/src/gen/main.js` | `doGenerate()` | 主生成流程 |
| `src/cli/dts2cpp/src/gen/analyze.js` | `doAnalyze()` | TS 解析 |
| `src/cli/dts2cpp/src/gen/generate.js` | `doGenerate()` | 代码生成 |

### 2. 内部模块接口

```
analyze/
├── namespace.js
│   └── parseNamespace() ───▶ 返回 { namespace, body }
├── interface.js
│   └── parseInterface() ──▶ 返回 InterfaceInfo[]
├── function.js
│   └── parseFunction() ───▶ 返回 FunctionInfo[]
├── params.js
│   └── parseParams() ──────▶ 返回 ParamInfo[]
├── return.js
│   └── parseReturn() ─────▶ 返回 ReturnInfo
├── enum.js
│   └── parseEnum() ──────▶ 返回 EnumInfo[]
└── type.js
    └── parseType() ──────▶ 返回 TypeInfo[]
```

### 3. 生成模块接口

```
generate/
├── namespace.js
│   └── generateNamespace() ──▶ 生成命名空间代码
├── interface.js
│   └── generateInterface() ──▶ 生成接口桥接
├── class.js
│   └── generateClass() ─────▶ 生成类桥接
├── enum.js
│   └── generateEnum() ──────▶ 生成枚举代码
├── function_sync.js
│   └── generateFunctionSync() ──▶ 同步回调函数
├── function_async.js
│   └── generateFunctionAsync() ──▶ 异步/Promise 函数
├── function_direct.js
│   └── generateFunctionDirect() ─▶ 直接调用函数
├── function_onoff.js
│   └── generateFunctionOnOff() ──▶ on/off 事件函数
├── function_threadsafe.js
│   └── generateThreadsafeFunc() ─▶ 线程安全函数
├── param_generate.js
│   └── generateParams() ────▶ 参数代码生成
└── return_generate.js
    └── generateReturn() ───▶ 返回值代码生成
```

### 4. XNapiTool 工具类

**证据**: `src/cli/dts2cpp/src/gen/extend/tool_utility.js`

| 方法类别 | 方法名 | 功能 | 稳定性 |
|---------|--------|------|--------|
| **JS→C 转换** | `SwapJs2CBool()` | 布尔 JS→C | ✅ Stable |
| | `SwapJs2CInt32()` | Int32 JS→C | ✅ Stable |
| | `SwapJs2CUint32()` | Uint32 JS→C | ✅ Stable |
| | `SwapJs2CInt64()` | Int64 JS→C | ✅ Stable |
| | `SwapJs2CDouble()` | Double JS→C | ✅ Stable |
| | `SwapJs2CUtf8()` | String JS→C | ✅ Stable |
| **C→JS 转换** | `SwapC2JsBool()` | 布尔 C→JS | ✅ Stable |
| | `SwapC2JsInt32()` | Int32 C→JS | ✅ Stable |
| | `SwapC2JsUint32()` | Uint32 C→JS | ✅ Stable |
| | `SwapC2JsInt64()` | Int64 C→JS | ✅ Stable |
| | `SwapC2JsDouble()` | Double C→JS | ✅ Stable |
| | `SwapC2JsUtf8()` | String C→JS | ✅ Stable |
| **数组操作** | `CreateArray()` | 创建 JS 数组 | ✅ Stable |
| | `GetArrayLength()` | 获取数组长度 | ✅ Stable |
| | `GetArrayElement()` | 获取数组元素 | ✅ Stable |
| | `SetArrayElement()` | 设置数组元素 | ✅ Stable |
| **Map 操作** | `GetMapLength()` | 获取 Map 大小 | ✅ Stable |
| | `GetMapElementName()` | 获取键名 | ✅ Stable |
| | `GetMapElementValue()` | 获取值 | ✅ Stable |
| | `SetMapElement()` | 设置键值对 | ✅ Stable |
| **异步支持** | `StartAsync()` | 启动异步任务 | ✅ Stable |
| | `FinishAsync()` | 完成异步任务 | ✅ Stable |
| **回调支持** | `SyncCallBack()` | 同步回调 | ✅ Stable |
| **导出** | `DefineFunction()` | 导出函数 | ✅ Stable |
| | `DefineClass()` | 导出类 | ✅ Stable |
| | `CreateEnumObject()` | 创建枚举对象 | ✅ Stable |

**稳定性标注说明**:
- ✅ Stable: 稳定接口，已广泛使用
- ⚠️ Unstable: 不稳定接口，可能变化
- 🔧 Internal: 内部接口，不建议外部使用

---

## h2sa 模块接口

### 1. 公共接口文件

| 文件路径 | 导出接口 | 职责 |
|----------|----------|------|
| `src/cli/h2sa/src/gen/main.js` | `doGenerate()` | 主生成流程 |
| `src/cli/h2sa/src/gen/analyze.js` | `doAnalyze()` | .h 解析 |
| `src/cli/h2sa/src/gen/generate.js` | `doGenerate()` | 框架生成 |

### 2. MessageParcel 类型映射

**证据**: `src/cli/h2sa/src/tools/common.js`

```javascript
const DATA_W_MAP = {
    'bool': 'WriteBoolUnaligned',
    'int8_t': 'WriteInt8',
    'uint8_t': 'WriteUint8',
    'int16_t': 'WriteInt16',
    'uint16_t': 'WriteUint16',
    'int32_t': 'WriteInt32',
    'int': 'WriteInt32',
    'uint32_t': 'WriteUint32',
    'int64_t': 'WriteInt64',
    'long': 'WriteInt64',
    'uint64_t': 'WriteUint64',
    'float': 'WriteFloat',
    'double': 'WriteDouble',
    'std::string': 'WriteString',
    'char*': 'WriteCString'
};

const DATA_R_MAP = {
    'bool': 'ReadBoolUnaligned',
    'int8_t': 'ReadInt8',
    'uint8_t': 'ReadUint8',
    // ... 类似映射
};
```

### 3. 代码模板接口

**证据**: `src/cli/h2sa/src/gen/file_template.js`

| 模板变量 | 说明 | 示例值 |
|----------|------|--------|
| `{CLASS_NAME}` | 类名 | `TestService` |
| `{SERVICE_ID}` | SA ID | `19000` |
| `{NAMESPACE}` | 命名空间 | `OHOS::Example` |
| `{METHODS}` | 方法列表 | `testFuncInner()` |
| `{PROXY_CODE}` | Proxy 代码块 | IPC 调用代码 |
| `{STUB_CODE}` | Stub 代码块 | 请求分发代码 |

---

## h2dtscpp 模块接口

### 1. 公共接口文件

| 文件路径 | 导出接口 | 职责 |
|----------|----------|------|
| `src/cli/h2dtscpp/src/src/main.js` | `doGenerate()` | 主流程 |
| `src/cli/h2dtscpp/src/src/tsGen/tsMain.js` | `doGenerate()` | TS 生成 |
| `src/cli/h2dtscpp/src/src/napiGen/functionDirect.js` | `generateDirectFunction()` | N-API 生成 |

### 2. 类型映射表

**证据**: `src/cli/h2dtscpp/src/src/tools/common.js`

| C++ 类型 | N-API 获取 | N-API 创建 |
|---------|-----------|------------|
| `int32_t` | `napi_get_value_int32` | `napi_create_int32` |
| `uint32_t` | `napi_get_value_uint32` | `napi_create_uint32` |
| `int64_t` | `napi_get_value_int64` | `napi_create_int64` |
| `double` | `napi_get_value_double` | `napi_create_double` |
| `bool` | `napi_get_value_bool` | `napi_create_boolean` |
| `std::string` | `napi_get_value_string_utf8` | `napi_create_string_utf8` |

### 3. 测试用例生成

| 函数 | 职责 |
|------|------|
| `generateFuncTestCase()` | 生成单个测试用例 |
| `genInitTestfunc()` | 初始化测试函数 |
| `getTestType()` | 生成测试值 |

---

## 模块依赖方向

### 依赖关系图

```
                    ┌─────────────────────────────────────────┐
                    │            examples/                    │
                    │   (依赖 cli 生成的框架代码)              │
                    └─────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              src/cli/                                       │
│                                                                             │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                 │
│   │  dts2cpp    │────▶│   h2sa      │     │  h2dtscpp   │                 │
│   │             │     │             │     │             │                 │
│   │ 依赖:       │     │  依赖:      │     │  依赖:      │                 │
│   │ - analyze/  │     │  - tools/   │     │  - tsGen/   │                 │
│   │ - generate/ │     │  - template │     │  - napiGen/ │                 │
│   │ - extend/   │     │             │     │  - json/    │                 │
│   └─────────────┘     └─────────────┘     └─────────────┘                 │
│         │                   │                   │                         │
│         └────────────────────┴───────────────────┘                         │
│                                  │                                          │
└──────────────────────────────────┼──────────────────────────────────────────┘
                                   │
                                   ▼
                        ┌─────────────────────┐
                        │      tools/         │
                        │  (公共工具模块)     │
                        └─────────────────────┘
```

### 依赖规则

| 依赖方向 | 是否允许 | 示例 |
|----------|----------|------|
| cli → tools | ✅ 允许 | `analyze.js` → `tools/common.js` |
| cli → docs | ❌ 禁止 | 不应依赖文档 |
| cli → examples | ❌ 禁止 | 工具不应依赖示例 |
| examples → cli | ⚠️ 有限 | 示例可引用生成产物 |
| 子模块 → 父模块 | ⚠️ 有限 | 需通过导出接口 |

---

## 稳定性标注

### 稳定性等级

| 等级 | 标注 | 含义 |
|------|------|------|
| **Stable** | ✅ | 稳定接口，版本兼容 |
| **Unstable** | ⚠️ | 可能变化的接口 |
| **Internal** | 🔧 | 内部实现，不保证兼容性 |
| **Deprecated** | ❌ | 已废弃，不建议使用 |

### 公共 API 稳定性

| 模块 | API | 稳定性 |
|------|-----|--------|
| dts2cpp | `main.doGenerate()` | ✅ Stable |
| dts2cpp | `XNapiTool` 方法 | ✅ Stable |
| dts2cpp | `generate/function_*.js` | ✅ Stable |
| h2sa | `main.doGenerate()` | ✅ Stable |
| h2sa | `tools/common.js` 类型映射 | ✅ Stable |
| h2dtscpp | `main.doGenerate()` | ✅ Stable |
| h2dtscpp | `napiGen/functionDirect.js` | ⚠️ Unstable |

---

## 接口契约

### dts2cpp 主接口

```javascript
// src/cli/dts2cpp/src/gen/main.js
/**
 * 主生成函数
 * @param {string} fileName - 输入 .d.ts 文件路径
 * @param {string} outDir - 输出目录
 * @param {boolean} imports - 是否支持导入
 * @param {string} numberType - number 类型的 C++ 映射
 * @param {Object} jsonConfig - 业务代码配置
 * @returns {Object} { success: boolean, message: string }
 */
function doGenerate(fileName, outDir, imports, numberType, jsonConfig)
```

### h2sa 主接口

```javascript
// src/cli/h2sa/src/gen/main.js
/**
 * 主生成函数
 * @param {string} filePath - 输入 .h 文件路径
 * @param {string} outDir - 输出目录
 * @param {number} serviceId - SA ID
 * @param {string} versionTag - 版本标签
 * @param {number} logLevel - 日志级别
 * @returns {Object} { success: boolean, message: string }
 */
function doGenerate(filePath, outDir, serviceId, versionTag, logLevel)
```

---

## 相关章节

- 架构说明: [03_Architecture.md](03_Architecture.md)
- API 参考: [04_NAPI_Reference.md](04_NAPI_Reference.md)
- 构建系统: [06_Build_System.md](06_Build_System.md)

---

[返回 SUMMARY.md](SUMMARY.md)
