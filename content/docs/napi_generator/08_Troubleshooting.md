# 常见问题与定位

> 构建、运行、调试问题与解决方案

## 目录

1. [dts2cpp 问题](#1-dts2cpp-问题)
2. [h2sa 问题](#2-h2sa-问题)
3. [h2dtscpp 问题](#3-h2dtscpp-问题)
4. [构建问题](#4-构建问题)
5. [运行时问题](#5-运行时问题)
6. [调试方法](#6-调试方法)

---

## 1. dts2cpp 问题

### Q1: 生成代码编译失败 - 找不到头文件

**症状**:
```
fatal error: 'napi/native_api.h' file not found
```

**原因**: N-API 头文件路径未配置

**解决**:

```bash
# 1. 检查 OpenHarmony SDK 是否安装
ls $OHOS_SDK/native/sysroot/usr/include/napi/

# 2. 设置环境变量
export OHOS_SDK=/path/to/ohos-sdk

# 3. 修改 BUILD.gn 中的 include_dirs
include_dirs = [
    "$OHOS_SDK/native/sysroot/usr/include/napi",
]
```

**证据**: `src/cli/dts2cpp/src/gen/extend/build_gn.js`

---

### Q2: 生成的 number 类型映射错误

**症状**:
```typescript
// .d.ts 中
function getValue(): number;  // 期望返回精确整数

// C++ 生成
int32_t getValue() { return 9007199254740993; }  // 精度丢失!
```

**原因**: TypeScript number 默认映射为 `uint32_t`

**解决**:

```bash
# 使用 -n 参数指定类型
node cmd_gen.js -f test.d.ts -n int64_t
```

**或在 .d.ts 中明确类型**:
```typescript
function getLargeValue(): bigint;  // 使用 bigint
```

**证据**: `src/cli/dts2cpp/src/gen/cmd_gen.js`

```javascript
ops = stdio.getopt({
    'numbertype': { key: 'n', default: 'uint32_t' }
});
```

---

### Q3: 生成的函数无法在 JS 中调用

**症状**:
```javascript
// JS 调用
import test from 'libentry.so';
test.myFunction();  // TypeError: test.myFunction is not a function
```

**排查步骤**:

1. **检查模块是否加载**:
```javascript
// 添加调试
console.log('Module loaded:', typeof test);
console.log('Available functions:', Object.keys(test));
```

2. **检查 N-API 注册代码**:
```cpp
// 在 init 函数中查看导出
static napi_value init(napi_env env, napi_value exports)
{
    // 确保包含导出
    napi_property_descriptor desc[] = {
        {"myFunction", nullptr, MyFunction, nullptr, nullptr, nullptr, napi_default, nullptr}
    };
    napi_define_properties(env, exports, 1, desc);
    return exports;
}
```

3. **检查函数名大小写**:
```typescript
// .d.ts 中使用 camelCase
function myFunction(): void;

// C++ 生成函数名
napi_value MyFunction(napi_env env, napi_callback_info info);
```

---

### Q4: 异步函数不返回 Promise

**症状**:
```typescript
// 声明
function asyncProcess(data: string): Promise<string>;

// JS 调用
const result = await asyncProcess('test');  // 不工作
```

**原因**: 异步函数需要特殊处理

**检查**: 函数类型识别

```javascript
// 确认 .d.ts 声明正确
declare function asyncProcess(data: string): Promise<string>;

// 或使用 AsyncCallback
declare function asyncProcess(data: string, callback: (err: string, data: string) => void): void;
```

**证据**: `src/cli/dts2cpp/src/gen/generate/function_async.js`

---

## 2. h2sa 问题

### Q5: SA 服务启动失败

**症状**:
```bash
# 日志
[FW] Failed to publish service: MyService
```

**排查步骤**:

1. **检查 SA ID 冲突**:
```bash
# 列出已注册的 SA
hdc shell
sa_manager list
```

2. **检查配置文件路径**:
```json
// bundle.json
{
    "module": {
        "deviceConfig": {
            "default": {
                "service": [
                    {
                        "name": "myservice",
                        "path": ["/system/bin/sa_main", "12345"]
                    }
                ]
            }
        }
    }
}
```

3. **检查权限**:
```xml
<!-- sa_profile/12345.xml -->
<info>
    <permission>
        ohos.permission.XXX
    </permission>
</info>
```

---

### Q6: IPC 调用返回错误码

**症状**:
```cpp
// 返回
retCode = Remote()->SendRequest(code, data, reply, option);
// retCode = 1 (错误)
```

**常见错误码**:

| 错误码 | 含义 | 排查方向 |
|--------|------|----------|
| `ERR_NONE` (0) | 成功 | - |
| `ERR_INVALID_VALUE` (1) | 参数无效 | 检查输入参数 |
| `OBJECT_NULL` (2) | 对象为空 | 检查对象初始化 |
| `STUB_OBJECT_NULL` | Stub 为空 | 检查服务是否发布 |
| `PROXY_OBJECT_NULL` | Proxy 为空 | 检查获取 Proxy 方式 |

**调试代码**:
```cpp
// Stub 端添加日志
int MyServiceStub::OnRemoteRequest(uint32_t code, MessageParcel& data,
    MessageParcel& reply, MessageOption &option)
{
    NAPI_LOG_INFO("OnRemoteRequest code=%d", code);
    
    // 校验接口令牌
    std::u16string descriptor = GetDescriptor();
    std::u16string remoteDescriptor = data.ReadInterfaceToken();
    if (descriptor != remoteDescriptor) {
        NAPI_LOG_ERROR("Descriptor mismatch!");
        return OBJECT_NULL;
    }
    
    // ... 正常处理
}
```

---

### Q7: MessageParcel 序列化失败

**症状**:
```cpp
// 写入
data.WriteInt32(value);  // 崩溃或异常

// 读取
int32_t value = data.ReadInt32();  // 返回错误值
```

**原因**: 

1. **读写不匹配**:
```cpp
// 错误示例 - 写和读类型不同
data.WriteInt32(42);     // 写入 4 字节
float v = data.ReadFloat();  // 读取 4 字节，但位置可能不对

// 正确示例 - 类型匹配
data.WriteInt32(42);
int32_t v = data.ReadInt32();
```

2. **数据越界**:
```cpp
// 错误 - 读取超出数据范围
data.WriteInt32(1);  // 只写了 4 字节
int64_t v = data.ReadInt64();  // 试图读 8 字节 → 越界!
```

**解决**: 确保 `Write*` 和 `Read*` 类型匹配

**证据**: `src/cli/h2sa/src/tools/common.js`

---

## 3. h2dtscpp 问题

### Q8: header_parser.exe 找不到

**症状**:
```
Error: header_parser.exe not found
```

**解决**:

```bash
# 1. 下载 header_parser.exe
# 从 https://gitee.com/openharmony/napi_generator/releases 下载

# 2. 放到正确位置
cp header_parser.exe src/cli/h2dtscpp/src/src/tsGen/
```

---

### Q9: 生成的测试用例编译失败

**症状**:
```bash
# ArkTS 编译错误
Cannot find name 'testNapi'
```

**解决**:

1. **检查模块导出**:
```typescript
// 确保 libentry.so 被正确导出
import testNapi from 'libentry.so';
```

2. **检查测试文件格式**:
```typescript
// 正确格式
import { describe, it, expect } from '@ohos/hypium';

export default function myAbilityTest() {
    describe('MyTest', () => {
        it('testName', 0, () => {
            // 测试代码
        })
    })
}
```

---

## 4. 构建问题

### Q10: GN 构建找不到依赖

**症状**:
```
ninja: Entering directory `out/default'
ninja: error: dependency //foundation/ability/ability_runtime:libnapi not found
```

**解决**:

```bash
# 1. 检查子系统配置
hb set

# 2. 检查依赖是否在 subsystems 中
# config.json 或 .gn 文件

# 3. 手动同步依赖
hb env
```

---

### Q11: 编译时内存不足

**症状**:
```
c++: internal compiler error: killed (program cc1plus)
```

**解决**:

```bash
# 增加编译内存
ninja -C out/default -j4  # 减少并行数

# 或
export MAKEFLAGS="-j2"
```

---

### Q12: 产物文件命名冲突

**症状**:
```
error: multiple definition of `NAPI_MODULE'
```

**原因**: 多个源文件定义了相同的 `NAPI_MODULE`

**解决**:

```cpp
// 正确: 每个模块只有一个入口
// file1.cpp - 不包含 NAPI_MODULE 定义
napi_value Function1(napi_env env, napi_callback_info info) {
    return nullptr;
}

// file2.cpp - 包含 NAPI_MODULE 定义
static napi_module MyModule = {
    .nm_register_func = Init,
    .nm_modname = "mymodule",
};

extern "C" void RegisterMyModule() {
    napi_module_register(&MyModule);
}
```

---

## 5. 运行时问题

### Q13: N-API 返回值类型错误

**症状**:
```javascript
// JS 期望 string，返回 number
const result = test.getString();
console.log(typeof result);  // "number"
```

**排查**:

1. **检查 N-API 返回函数**:
```cpp
// 错误 - 返回了 C 类型
napi_value GetString(napi_env env, napi_callback_info info) {
    std::string str = "hello";
    return str;  // 错误! 应该转换为 napi_value
}

// 正确 - 使用 napi_create_string_utf8
napi_value GetString(napi_env env, napi_callback_info info) {
    std::string str = "hello";
    napi_value result;
    napi_create_string_utf8(env, str.c_str(), str.length(), &result);
    return result;
}
```

2. **检查类型映射配置**:
```javascript
// dts2cpp 中确认类型映射
// functionDirect.js
function mapReturnType(returnType) {
    if (returnType === 'string') {
        return 'napi_create_string_utf8';
    }
    // ...
}
```

---

### Q14: 异步回调不执行

**症状**:
```javascript
// Promise pending 永不 resolve
const result = await asyncOperation();
```

**排查**:

1. **检查异步工作队列**:
```cpp
// 确保 napi_queue_async_work 被调用
napi_value AsyncOperation(napi_env env, napi_callback_info info) {
    napi_value promise;
    napi_create_promise(env, &promise, &deferred);
    
    napi_async_work work;
    napi_create_async_work(env, nullptr, Execute, Complete,
        nullptr, &work);
    napi_queue_async_work(env, work);  // 确保调用!
    
    return promise;
}
```

2. **检查 Complete 回调**:
```cpp
void Complete(napi_env env, napi_status status, void* data) {
    napi_deferred deferred = (napi_deferred)data;
    
    if (status == napi_ok) {
        // 确保 resolve
        napi_value result;
        napi_create_string_utf8(env, "done", &result);
        napi_resolve_deferred(env, deferred, result);
    } else {
        napi_value error;
        napi_create_error(env, nullptr, &error);
        napi_reject_deferred(env, deferred, error);
    }
}
```

---

## 6. 调试方法

### 6.1 启用日志

**dts2cpp 日志**:
```bash
# 设置日志级别 0-3
node cmd_gen.js -f test.d.ts -l 3
```

**N-API 日志**:
```cpp
// C++ 代码中
#define NAPI_DEBUG 1

#ifdef NAPI_DEBUG
#define NAPI_LOG(fmt, ...) printf(fmt, ##__VA_ARGS__)
#else
#define NAPI_LOG(...)
#endif
```

### 6.2 打印函数调用链

```cpp
// 入口函数打印
napi_value MyFunction(napi_env env, napi_callback_info info) {
    NAPI_LOG("MyFunction called");
    
    size_t argc;
    napi_get_cb_info(env, info, &argc, nullptr, nullptr, nullptr);
    NAPI_LOG("  argc=%zu", argc);
    
    // ... 业务逻辑
}
```

### 6.3 使用 hdc 调试 SA

```bash
# 列出 SA
hdc shell
sa_manager list

# 查看 SA 日志
hdc shell
hilog | grep MyService

# 获取 SA 状态
hdc shell
dump -a MyService
```

### 6.4 常见调试命令

| 命令 | 用途 |
|------|------|
| `hdc shell hilog` | 查看日志 |
| `hdc shell sa_manager list` | 列出 SA |
| `hdc shell ps -A | grep sa` | 查看 SA 进程 |
| `hdc file recv <remote> <local>` | 拉取文件 |
| `hdc shell` | 进入设备 shell |

---

## 性能优化建议

### 1. 减少 IPC 调用

```cpp
// 批量操作替代多次 IPC
// 错误 - 多次调用
proxy->SetValue(1);
proxy->SetName("test");
proxy->Save();

// 正确 - 批量调用
proxy->SetValues(1, "test");
proxy->Save();
```

### 2. 使用异步替代同步

```cpp
// 同步 - 阻塞调用线程
int result = proxy->HeavyOperation();  // 阻塞

// 异步 - 不阻塞
proxy->HeavyOperationAsync([](int result) {
    // 回调处理
});
```

---

## 相关章节

- 项目概览: [01_Overview.md](01_Overview.md)
- 架构说明: [03_Architecture.md](03_Architecture.md)
- API 参考: [04_NAPI_Reference.md](04_NAPI_Reference.md)

---

[返回 SUMMARY.md](SUMMARY.md)
