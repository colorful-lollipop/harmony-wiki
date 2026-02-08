# API 参考

## 命令行 API

### es2abc

#### 使用方式

```bash
es2abc [options] input_file
```

#### 选项

| 参数 | 类型 | 描述 |
|------|------|------|
| `--extension <type>` | string | 输入文件类型 (js/ts/as) |
| `--module` | flag | 按模块编译 |
| `--output <path>` | string | 输出文件路径 |
| `--opt-level <0-2>` | number | 优化级别 |
| `--thread <count>` | number | 编译线程数 |
| `--dump-ast` | flag | 输出 AST |
| `--dump-assembly` | flag | 输出字节码汇编 |
| `--dump-size-stat` | flag | 显示大小统计 |
| `--debug-info` | flag | 输出调试信息 |
| `--help` | flag | 显示帮助 |

## N-API 接口

### 模块注册

```cpp
NAPI_MODULE(modname, InitModule)
```

### 类型转换

#### JavaScript → C++

| 函数 | 参数 | 返回值 |
|------|------|--------|
| `GetBoolean(env, value)` | napi_env, napi_value | bool |
| `GetInt32(env, value)` | napi_env, napi_value | int32_t |
| `GetUInt32(env, value)` | napi_env, napi_value | uint32_t |
| `GetFloat32(env, value)` | napi_env, napi_value | float |
| `GetFloat64(env, value)` | napi_env, napi_value | double |
| `GetString(env, value)` | napi_env, napi_value | string |
| `GetPointer(env, value)` | napi_env, napi_value | void* |
| `GetInt64(env, value)` | napi_env, napi_value | int64_t |

#### C++ → JavaScript

| 函数 | 参数 | 返回值 |
|------|------|--------|
| `MakeBoolean(env, value)` | napi_env, bool | napi_value |
| `MakeInt32(env, value)` | napi_env, int32_t | napi_value |
| `MakeUInt32(env, value)` | napi_env, uint32_t | napi_value |
| `MakeFloat32(env, value)` | napi_env, float | napi_value |
| `MakeFloat64(env, value)` | napi_env, double | napi_value |
| `MakeString(env, value)` | napi_env, string | napi_value |
| `MakePointer(env, value)` | napi_env, void* | napi_value |
| `MakeObject(env, value)` | napi_env, - | napi_value |
| `MakeVoid(env)` | napi_env | napi_value |

### Promise 支持

| 函数 | 描述 |
|------|------|
| `napi_create_promise` | 创建 Promise |
| `napi_resolve_deferred` | 解决 Promise |
| `napi_reject_deferred` | 拒绝 Promise |
| `napi_create_threadsafe_function` | 创建线程安全函数 |

## GN Target API

### 构建 Target

| Target | 类型 | 描述 |
|--------|------|------|
| `ets_frontend_build` | group | 主构建组 |
| `es2panda` | executable | JS/TS 编译器 |
| `merge_proto_abc_build` | executable | ABC 合并工具 |
| `libes2panda_public` | shared_library | 公共库 |

## 内部 API

### 编译器核心类

| 类 | 模块 | 描述 |
|----|------|------|
| `Lexer` | lexer | 词法分析器 |
| `Parser` | parser | 语法解析器 |
| `Binder` | binder | 符号绑定器 |
| `IRBuilder` | ir | IR 构建器 |
| `BytecodeBuilder` | ir | 字节码构建器 |
| `TSBinder` | varbinder | TS 符号绑定器 |
| `Checker` | checker | 类型检查器 |

---

## 错误码

### 编译错误

| 错误码 | 含义 |
|--------|------|
| 0 | 成功 |
| 1 | 编译错误 |
| 2 | 无效参数 |
| 3 | 文件不存在 |
| 4 | 系统错误 |

### N-API 错误

| 错误码 | 含义 |
|--------|------|
| `napi_ok` | 成功 |
| `napi_invalid_arg` | 无效参数 |
| `napi_pending_exception` | 挂起异常 |
| `napi_invalid_handle` | 无效句柄 |
