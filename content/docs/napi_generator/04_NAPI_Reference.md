# N-API 参考文档

> 6 个代码生成工具的 API 清单、注册点与调用链

## 目录

1. [dts2cpp 工具](#1-dts2cpp-工具)
2. [h2sa 工具](#2-h2sa-工具)
3. [h2dtscpp 工具](#3-h2dtscpp-工具)
4. [h2dts 工具](#4-h2dts-工具)
5. [cmake2gn 工具](#5-cmake2gn-工具)
6. [scan 工具](#6-scan-工具)

---

## 1. dts2cpp 工具

> TypeScript 声明文件 (.d.ts) → N-API C++ 框架代码

### 1.1 工具信息

| 属性 | 值 |
|------|-----|
| **入口文件** | `src/cli/dts2cpp/src/gen/cmd_gen.js` |
| **依赖** | Node.js 14+, npm, typescript |
| **版本** | V1.4.1 |

### 1.2 使用方法

```bash
# 基本用法
node src/cli/dts2cpp/src/gen/cmd_gen.js \
    -f @ohos.mylib.d.ts \
    -o output_directory

# 完整参数
node cmd_gen.js \
    -f <file.d.ts>        # 输入文件
    -d <directory>        # 输入目录（批量）
    -o <out_dir>          # 输出目录 (默认 .)
    -i <true|false>       # 支持导入自定义文件 (默认 false)
    -n <cpp_type>         # number 类型映射 (默认 uint32_t)
    -l <0-3>              # 日志级别 (默认 1)
    -s <cfg.json>         # 业务代码配置文件
```

### 1.3 支持的 TypeScript 类型

| TypeScript 类型 | C++ 类型 | N-API 函数 |
|----------------|----------|-----------|
| `string` | `std::string` | `napi_create_string_utf8` / `napi_get_value_string_utf8` |
| `number` | `uint32_t` (默认) | `napi_create_uint32` / `napi_get_value_uint32` |
| | `int32_t` | `napi_create_int32` / `napi_get_value_int32` |
| | `int64_t` | `napi_create_int64` / `napi_get_value_int64` |
| | `double` | `napi_create_double` / `napi_get_value_double` |
| `boolean` | `bool` | `napi_create_boolean` / `napi_get_value_bool` |
| `Array<T>` | `std::vector<T>` | `napi_create_array` / `napi_*_element` |
| `Map<K,V>` | `std::map<K,V>` | `napi_create_object` / `napi_*_property` |

### 1.4 函数类型支持

| 类型 | 标识 | 说明 | 生成文件 |
|------|------|------|----------|
| **DIRECT** | 无回调 | 直接调用，同步返回 | `function_direct.js` |
| **SYNC** | Callback 参数 | 同步回调模式 | `function_sync.js` |
| **ASYNC** | AsyncCallback | 异步执行，Promise | `function_async.js` |
| **PROMISE** | 无回调，返回 Promise | 异步返回 Promise | `function_async.js` |
| **ON/OFF** | 函数名 on/off 开头 | 事件监听模式 | `function_onoff.js` |
| **THREADSAFE** | 线程安全 | 多线程安全调用 | `function_threadsafe.js` |

### 1.5 N-API 模块注册

**证据**: `src/cli/dts2cpp/src/gen/generate.js`

```cpp
// 标准 N-API 模块注册代码
static napi_module g_[implName]_Module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = init,
    .nm_modname = "[modulename]",
    .nm_priv = ((void *)0),
    .reserved = {(void *)0},
};

extern "C" __attribute__((constructor)) void Register_[implName]_Module(void)
{
    napi_module_register(&g_[implName]_Module);
}
```

### 1.6 生成的 API 清单示例

**输入** (`@ohos.mylib.d.ts`):
```typescript
declare namespace mylib {
    function add(a: number, b: number): number;
    function asyncProcess(data: string): Promise<string>;
}
```

**生成 C++ 接口** (`mylib_middle.h`):
```cpp
#include <napi/native_api.h>

namespace mylib {
napi_value Add(napi_env env, napi_callback_info info);
napi_value AsyncProcess(napi_env env, napi_callback_info info);
}  // namespace mylib
```

**生成 N-API 实现** (`mylib_middle.cpp`):
```cpp
#include "mylib_middle.h"
#include "tool_utility.h"

napi_value Add(napi_env env, napi_callback_info info) {
    std::shared_ptr<XNapiTool> pxt = std::make_shared<XNapiTool>(env, info);
    // 参数解析
    int32_t a = pxt->SwapJs2CInt32(0);
    int32_t b = pxt->SwapJs2CInt32(1);
    // 业务调用
    int32_t result = AddCpp(a, b);
    // 返回值
    return pxt->SwapC2JsInt32(result);
}
```

### 1.7 调用链

```
JS 调用
    │
    ▼
libentry.so (N-API 模块)
    │
    ├── napi_register_func (init 函数)
    │   │
    │   └── napi_define_properties (注册导出)
    │       │
    │       ├── Add (同步函数)
    │       └── AsyncProcess (异步函数)
    │
    ▼
*_middle.cpp (N-API 中间层)
    │
    ├── SwapJs2C* (JS → C 类型转换)
    │   └── napi_get_value_*(env, value, &cValue)
    │
    ├── 业务调用
    │   └── MyLib::MethodCpp()
    │
    └── SwapC2Js* (C → JS 类型转换)
        └── napi_create_*(env, cValue, &jsValue)
```

---

## 2. h2sa 工具

> C++ 头文件 → Service Ability 框架代码

### 2.1 工具信息

| 属性 | 值 |
|------|-----|
| **入口文件** | `src/cli/h2sa/src/gen/main.js` |
| **依赖** | Node.js |
| **版本** | V1.0.0 |

### 2.2 使用方法

```bash
node src/cli/h2sa/src/gen/main.js \
    -f <input.h>              # 输入 .h 文件
    -o <out_dir>              # 输出目录 (默认当前目录)
    -s <serviceId>            # SA ID (9000~16777214, 默认 19000)
    -v <4.1|3.2>              # 版本标签 (默认 3.2)
    -l <0-3>                  # 日志级别 (默认 1)
```

### 2.3 输入格式要求

**证据**: `src/cli/h2sa/README_ZH.md`

```cpp
// 输入 .h 文件必须包含 @ServiceClass 注解
namespace OHOS {
    namespace Example {
        /**
         * @brief service服务，提供IPC调用接口
         * @ServiceClass
         */
        class test {
        public:
            int testFunc(int v1, int v2, bool v3);
        };
    }
}
```

### 2.4 生成的类结构

| 类名 | 继承 | 职责 |
|------|------|------|
| `I{className}Service` | 接口 | 定义远程调用接口 |
| `{className}Proxy` | `IRemoteProxy<I{className}Service>` | 客户端代理，发送 IPC 请求 |
| `{className}Stub` | `IRemoteStub<I{className}Service>` | 服务端存根，接收 IPC 请求 |
| `{className}Service` | `SystemAbility` + `{className}Stub` | 服务实现 |

### 2.5 MessageParcel 类型映射

**证据**: `src/cli/h2sa/src/tools/common.js`

| C++ 类型 | 写方法 | 读方法 |
|---------|-------|-------|
| `bool` | `WriteBoolUnaligned` | `ReadBoolUnaligned` |
| `int8_t` / `uint8_t` | `WriteInt8` / `WriteUint8` | `ReadInt8` / `ReadUint8` |
| `int16_t` / `uint16_t` | `WriteInt16` / `WriteUint16` | `ReadInt16` / `ReadUint16` |
| `int32_t` / `int` | `WriteInt32` | `ReadInt32` |
| `uint32_t` | `WriteUint32` | `ReadUint32` |
| `int64_t` / `long` | `WriteInt64` | `ReadInt64` |
| `uint64_t` | `WriteUint64` | `ReadUint64` |
| `float` | `WriteFloat` | `ReadFloat` |
| `double` | `WriteDouble` | `ReadDouble` |
| `std::string` | `WriteString` | `ReadString` |
| `char*` | `WriteCString` | `ReadCString` |

### 2.6 生成产物清单

```
{serviceName}service/
├── include/
│   ├── i_{name}_service.h        # 接口定义
│   ├── {name}_service_proxy.h    # Proxy 头文件
│   ├── {name}_service_stub.h     # Stub 头文件
│   └── {name}_service.h          # Service 头文件
├── src/
│   ├── i_{name}_service.cpp       # 接口实现
│   ├── {name}_service_proxy.cpp  # Proxy 实现
│   ├── {name}_service_stub.cpp   # Stub 实现
│   ├── {name}_service.cpp        # Service 实现
│   └── {name}_client.cpp         # 客户端示例
├── interface/                      # 接口文件
├── sa_profile/
│   ├── BUILD.gn                    # 构建配置
│   └── {serviceId}.xml             # SA 配置
├── etc/
│   └── {name}_service.cfg          # 启动配置
├── BUILD.gn                         # 子系统构建
└── bundle.json                      # 子系统配置
```

### 2.7 IPC 调用链

```
客户端                               Binder                    服务端
  │                                   │                        │
  │ {Service}Proxy::method()          │                        │
  │──────────────────────────────────▶│                        │
  │                                   │                        │
  │ MessageParcel::WriteInterfaceToken()                    │
  │ MessageParcel::WriteInt32(param)  │                        │
  │──────────────────────────────────▶│                        │
  │                                   │                        │
  │ Remote()->SendRequest()           │───────────────────────▶│
  │                                   │                        │
  │                                   │                        │ Stub::OnRemoteRequest()
  │                                   │                        │◀─────────────────────
  │                                   │                        │
  │                                   │                        │ MessageParcel::ReadInt32()
  │                                   │                        │◀─────────────────────
  │                                   │                        │
  │                                   │                        │ Service::methodInner()
  │                                   │                        │◀─────────────────────
  │                                   │                        │
  │                                   │                        │ MessageParcel::WriteInt32()
  │                                   │                        │◀─────────────────────
  │◀──────────────────────────────────│◀───────────────────────│
  │                                   │                        │
  │ Reply.ReadInt32()                 │                        │
  ▼                                   ▼                        ▼
```

---

## 3. h2dtscpp 工具

> C++ 头文件 → TypeScript 声明 + N-API 实现 + 测试用例

### 3.1 工具信息

| 属性 | 值 |
|------|-----|
| **入口文件** | `src/cli/h2dtscpp/src/src/main.js` |
| **依赖** | Node.js, header_parser.exe |
| **版本** | V1.0.0 |

### 3.2 使用方法

```bash
node src/cli/h2dtscpp/src/src/main.js \
    -f <input.h>          # 输入 .h 文件
    -o <out_dir>          # 输出目录 (默认 .h 所在目录)
```

### 3.3 生成产物结构

```
{output_dir}/
├── tsout/
│   └── index.d.ts        # TypeScript 声明文件
├── cppout/
│   ├── {name}common.h     # 公共头文件
│   ├── {name}common.cpp   # 公共实现
│   ├── {function}.cpp     # 各函数 N-API 实现
│   ├── {name}init.cpp     # 模块初始化
│   └── {name}napi.h       # N-API 声明
└── testout/
    └── {name}.Ability.test.ets  # ArkTS 测试用例
```

### 3.4 N-API 生成模式

**证据**: `src/cli/h2dtscpp/src/src/napiGen/functionDirect.js`

```cpp
// 生成的 N-API 函数框架
napi_value {FunctionName}(napi_env env, napi_callback_info info) {
    // 1. 获取参数数量
    size_t argc = 1;
    napi_value argv[1];
    napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);

    // 2. 解析 JS 参数
    // 3. 调用 C++ 业务实现
    // 4. 返回结果
}
```

### 3.5 测试用例格式

**证据**: `src/cli/h2dtscpp/src/src/napiGen/functionDirectTest.js`

```typescript
// {name}Ability.test.ets
import testNapi from 'libentry.so';
import { describe, it, expect } from '@ohos/hypium';

export default function nameAbilityTest() {
  describe('ActsAbilityTest', () => {
    it('KH123_funcName', 0, () => {
      let param1 = 5
      let result = testNapi.KH123_funcName(param1)
      hilog.info(0x0000, "testTag", "Test NAPI KH123_funcName: ", JSON.stringify(result));
    })
  })
}
```

---

## 4. h2dts 工具

> C++ 头文件 → TypeScript 声明文件

### 4.1 工具信息

| 属性 | 值 |
|------|-----|
| **入口文件** | `src/cli/h2dts/src/` |
| **依赖** | VS Code 插件 |
| **版本** | V1.0.0 |

### 4.2 使用方式

- **VS Code 插件**: 右键 .h 文件 → "Generate TypeScript Definition"
- **命令行**: 通过 VS Code 插件调用

---

## 5. cmake2gn 工具

> CMakeLists.txt → BUILD.gn

### 5.1 工具信息

| 属性 | 值 |
|------|-----|
| **入口文件** | `src/cli/cmake2gn/src/` |
| **依赖** | CMake, Python |
| **版本** | V1.0.0 |

### 5.2 使用方法

```bash
# 需在 CMake 环境下运行
cd src/cli/cmake2gn/src/
python cmake2gn.py -i <CMakeLists.txt> -o <BUILD.gn>
```

---

## 6. scan 工具

> 三方库 API 依赖扫描

### 6.1 工具信息

| 属性 | 值 |
|------|-----|
| **入口文件** | `src/tool/api/src/` |
| **依赖** | Node.js |
| **版本** | V1.0.0 |

### 6.2 使用方法

```bash
cd src/tool/api/src/
npm install
node scan.js -i <input_dir> -o <result.xlsx>
```

### 6.3 输出格式

| 列名 | 说明 |
|------|------|
| 文件路径 | 包含非 OH API 的源文件 |
| API 名称 | 检测到的外部 API |
| 行号 | API 出现位置 |
| 建议 | 替代方案或注意事项 |

---

## 相关章节

- 架构说明: [03_Architecture.md](03_Architecture.md)
- 内部 API: [05_Internal_API.md](05_Internal_API.md)
- 构建系统: [06_Build_System.md](06_Build_System.md)

---

[返回 SUMMARY.md](SUMMARY.md)
