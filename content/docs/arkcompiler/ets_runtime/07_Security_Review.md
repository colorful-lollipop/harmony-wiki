# 安全风险评审

本文档对 ArkCompiler ETS Runtime 进行安全风险分析，包括攻击面识别、信任边界分析和可被利用点（附修复建议）。

## 评审范围

本次评审覆盖以下代码区域：

- `ecmascript/napi/` - N-API 接口层
- `ecmascript/js_api/` - JS API 实现
- `ecmascript/containers/` - 容器 API
- `ecmascript/ohos/` - 系统集成模块
- `ecmascript/jspandafile/` - 文件加载模块
- `ecmascript/serializer/` - 序列化模块

**未覆盖范围**：
- 测试代码（`test/`、`*_test.*`）
- 调试器模块（debugger/）
- 第三方依赖（libuv、ICU 等）

## 攻击面清单

### 1. N-API 入口点

**文件位置**：`ecmascript/napi/jsnapi.cpp`

| 入口函数 | 攻击面类型 | 说明 |
|----------|----------|------|
| `napi_create_runtime` | 参数校验 | 传入配置对象的校验 |
| `napi_load_module` | 文件路径 | ABC 文件路径处理 |
| `napi_call_function` | 参数传递 | 函数调用参数传递 |
| `napi_create_async_work` | 回调注册 | 异步回调函数注册 |

### 2. 文件加载入口

**文件位置**：`ecmascript/jspandafile/js_pandafile.cpp`

| 函数 | 攻击面类型 | 说明 |
|------|----------|------|
| `JsPandafile::Load` | 文件解析 | ABC 字节码解析 |
| `JsPandafileManager::GetInstance` | 资源管理 | 模块缓存管理 |

### 3. 系统集成接口

**文件位置**：`ecmascript/ohos/`

| 函数 | 攻击面类型 | 说明 |
|------|----------|------|
| `code_decrypt.cpp` | 代码解密 | 运行时解密处理 |
| `module_pkg_parser.cpp` | 模块解析 | 模块包解析 |

### 4. 序列化接口

**文件位置**：`ecmascript/serializer/`

| 函数 | 攻击面类型 | 说明 |
|------|----------|------|
| `ValueSerializer::Serialize` | 数据序列化 | 跨进程数据序列化 |
| `ValueDeserializer::Deserialize` | 数据反序列化 | 跨进程数据反序列化 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界模型                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              可信区域（Trust Zone）                      │    │
│  │  ┌─────────────────────────────────────────────────┐   │    │
│  │  │  Runtime 核心代码                               │   │    │
│  │  │  - 解释器                                        │   │    │
│  │  │  - 编译器                                        │   │    │
│  │  │  - GC                                           │   │    │
│  │  └─────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ▲                                   │
│                              │ 边界                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              边界区域（Boundary Zone）                   │    │
│  │  ┌─────────────────────────────────────────────────┐   │    │
│  │  │  N-API 接口层                                    │   │    │
│  │  │  - 参数校验                                      │   │    │
│  │  │  - 类型转换                                      │   │    │
│  │  │  - 错误处理                                      │   │    │
│  │  └─────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ▲                                   │
│                              │ 边界                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              不可信区域（Untrusted Zone）              │    │
│  │  ┌─────────────────────────────────────────────────┐   │    │
│  │  │  外部输入                                        │   │    │
│  │  │  - ABC 字节码文件                                │   │    │
│  │  │  - 用户输入数据                                  │   │    │
│  │  │  - 网络数据                                      │   │    │
│  │  │  - 文件系统                                      │   │    │
│  │  └─────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 可被利用点与修复建议

### 1. 路径遍历漏洞

**风险级别**：高

**证据位置**：
- `ecmascript/base/path_helper.cpp` - 路径处理逻辑
- `ecmascript/jspandafile/js_pandafile.cpp:150+` - 文件加载逻辑

**问题描述**：
在加载 ABC 文件时，路径校验不严格可能导致路径遍历攻击。攻击者可能通过构造特殊路径访问受限目录。

**触发场景**：
```cpp
// 潜在漏洞代码模式
std::string LoadModule(const std::string& path) {
    // 未校验 path 中的 "../" 或绝对路径
    return LoadFile(path);
}
```

**影响**：
- 读取任意系统文件
- 绕过沙箱限制

**修复建议**：
1. 对所有文件路径进行规范化处理
2. 拒绝包含 ".." 的路径
3. 限制可访问的根目录范围
4. 白名单校验文件扩展名

**修复代码示例**：
```cpp
bool IsValidModulePath(const std::string& path) {
    // 1. 检查绝对路径
    if (path[0] == '/') {
        return false;  // 拒绝绝对路径
    }
    // 2. 检查路径遍历
    if (path.find("..") != std::string::npos) {
        return false;  // 拒绝路径遍历
    }
    // 3. 检查扩展名
    if (GetExtension(path) != ".abc") {
        return false;  // 只允许 .abc 文件
    }
    return true;
}
```

### 2. 整数溢出漏洞

**风险级别**：高

**证据位置**：
- `ecmascript/base/array_helper.cpp` - 数组操作
- `ecmascript/containers/containers_*.cpp` - 容器实现

**问题描述**：
在处理数组索引、缓冲区大小等数值时，未进行溢出校验，可能导致缓冲区溢出或拒绝服务。

**触发场景**：
```cpp
// 潜在漏洞代码模式
void ProcessData(uint32_t size) {
    char* buffer = new char[size - 1];  // size=0 时为 size_t 最大值
    // ...
}
```

**影响**：
- 堆溢出
- 拒绝服务（DoS）
- 代码执行

**修复建议**：
1. 在算术运算前进行边界检查
2. 使用安全的整数运算库
3. 验证所有外部输入的数值范围

**修复代码示例**：
```cpp
void ProcessData(uint32_t size) {
    // 1. 范围检查
    if (size == 0 || size > MAX_BUFFER_SIZE) {
        return;  // 拒绝非法大小
    }
    // 2. 使用安全乘法
    size_t total = SafeMul(size, sizeof(Element));
    if (total > MAX_TOTAL_SIZE) {
        return;  // 拒绝溢出
    }
    char* buffer = new (std::nothrow) char[total];
    // ...
}
```

### 3. 空指针解引用

**风险级别**：中

**证据位置**：
- `ecmascript/js_api/js_api_*.cpp` - JS API 实现
- `ecmascript/containers/containers_*.cpp` - 容器实现

**问题描述**：
在处理可选参数或空值时，未进行空指针检查，可能导致运行时崩溃。

**触发场景**：
```cpp
// 潜在漏洞代码模式
void ProcessObject(JsApiObject* obj) {
    obj->SetData(data);  // obj 可能为 nullptr
}
```

**影响**：
- 进程崩溃（DoS）
- 拒绝服务

**修复建议**：
1. 对所有外部输入进行空指针检查
2. 使用断言进行内部合约校验
3. 返回明确的错误码

**修复代码示例**：
```cpp
napi_status ProcessObject(napi_env env, napi_value obj) {
    // 1. 空指针检查
    if (obj == nullptr) {
        return napi_invalid_arg;
    }
    // 2. 转换为内部对象
    JsApiObject* internalObj = nullptr;
    napi_status status = napi_unwrap(env, obj, reinterpret_cast<void**>(&internalObj));
    if (status != napi_ok || internalObj == nullptr) {
        return napi_invalid_arg;
    }
    // 3. 安全使用
    internalObj->SetData(data);
    return napi_ok;
}
```

### 4. 缓冲区溢出

**风险级别**：高

**证据位置**：
- `ecmascript/js_api/js_api_buffer.cpp` - Buffer 实现
- `ecmascript/base/string_helper.cpp` - 字符串操作

**问题描述**：
在字符串拼接、缓冲区复制等操作中，未校验目标缓冲区大小，可能导致栈或堆溢出。

**触发场景**：
```cpp
// 潜在漏洞代码模式
void CopyData(char* dest, const char* src) {
    strcpy(dest, src);  // 未检查 dest 缓冲区大小
}
```

**影响**：
- 堆溢出
- 栈溢出
- 代码执行

**修复建议**：
1. 使用安全的字符串函数（strncpy_s、memcpy_s）
2. 始终校验目标缓冲区大小
3. 使用现代 C++ 安全字符串库

**修复代码示例**：
```cpp
napi_status CopyData(napi_env env, char* dest, size_t destSize,
                     const char* src, size_t srcSize) {
    // 1. 大小校验
    if (destSize < srcSize + 1) {
        return napi_string_expected;  // 缓冲区不足
    }
    // 2. 安全复制
    errno_t err = strncpy_s(dest, destSize, src, srcSize);
    if (err != EOK) {
        return napi_generic_failure;
    }
    dest[srcSize] = '\0';
    return napi_ok;
}
```

### 5. 类型混淆漏洞

**风险级别**：中

**证据位置**：
- `ecmascript/napi/jsnapi.cpp` - N-API 类型转换
- `ecmascript/js_api/js_api_*.cpp` - JS API 类型处理

**问题描述**：
在 N-API 参数类型转换过程中，未严格校验实际类型，可能导致类型混淆攻击。

**触发场景**：
```cpp
// 潜在漏洞代码模式
JSValue GetProperty(napi_value obj, const char* key) {
    return obj->GetProperty(key);  // 未验证 obj 是否为 Object 类型
}
```

**影响**：
- 类型混淆
- 崩溃
- 潜在的代码执行

**修复建议**：
1. 对每个参数进行类型校验
2. 使用断言进行内部类型检查
3. 明确错误码返回

**修复代码示例**：
```cpp
napi_status GetProperty(napi_env env, napi_value obj,
                        napi_value key, napi_value* result) {
    // 1. 参数校验
    if (env == nullptr || obj == nullptr || key == nullptr || result == nullptr) {
        return napi_invalid_arg;
    }
    // 2. 类型校验
    bool isObject = false;
    napi_status status = napi_is_object(env, obj, &isObject);
    if (status != napi_ok || !isObject) {
        return napi_object_expected;
    }
    status = napi_is_string(env, key, &isObject);
    if (status != napi_ok || !isObject) {
        return napi_string_expected;
    }
    // 3. 安全获取属性
    return napi_get_property(env, obj, key, result);
}
```

### 6. 资源泄漏

**风险级别**：低

**证据位置**：
- `ecmascript/js_api/js_api_buffer.cpp` - Buffer 资源管理
- `ecmascript/serializer/*.cpp` - 序列化资源

**问题描述**：
在错误处理路径中，可能存在句柄、文件描述符等资源未释放的问题。

**触发场景**：
```cpp
// 潜在漏洞代码模式
napi_status ProcessFile(const char* path) {
    FILE* fp = fopen(path, "rb");
    // ... 处理逻辑
    if (error_condition) {
        return napi_generic_failure;  // fclose 未调用
    }
    fclose(fp);
    return napi_ok;
}
```

**影响**：
- 资源耗尽
- 拒绝服务（DoS）

**修复建议**：
1. 使用 RAII 模式管理资源
2. 使用智能指针
3. 统一的错误处理路径

**修复代码示例**：
```cpp
class FileGuard {
public:
    FileGuard(FILE* fp) : fp_(fp) {}
    ~FileGuard() {
        if (fp_ != nullptr) {
            fclose(fp_);
        }
    }
    // 禁止拷贝
    FileGuard(const FileGuard&) = delete;
    FileGuard& operator=(const FileGuard&) = delete;
private:
    FILE* fp_;
};

napi_status ProcessFile(const char* path) {
    FILE* fp = fopen(path, "rb");
    if (fp == nullptr) {
        return napi_invalid_arg;
    }
    FileGuard guard(fp);
    // ... 处理逻辑
    if (error_condition) {
        return napi_generic_failure;  // 自动调用 fclose
    }
    return napi_ok;
}
```

### 7. 序列化反序列化漏洞

**风险级别**：高

**证据位置**：
- `ecmascript/serializer/value_serializer.cpp` - 值序列化
- `ecmascript/serializer/value_deserializer.cpp` - 值反序列化

**问题描述**：
在跨进程序列化/反序列化过程中，恶意构造的数据可能导致类型混淆或内存破坏。

**触发场景**：
```cpp
// 潜在漏洞代码模式
void Deserialize(DataInputStream& input) {
    uint32_t type = input.ReadUInt32();
    void* data = input.ReadBytes(size);  // size 由输入决定
    // 根据 type 解析 data
}
```

**影响**：
- 内存破坏
- 代码执行

**修复建议**：
1. 严格校验序列化数据的格式和大小
2. 白名单校验类型字段
3. 深度校验嵌套数据结构

**修复代码示例**：
```cpp
napi_status SafeDeserialize(napi_env env, const uint8_t* data, size_t size) {
    DataInputStream stream(data, size);

    // 1. 校验魔数
    uint32_t magic = stream.ReadUInt32();
    if (magic != EXPECTED_MAGIC) {
        return napi_invalid_arg;
    }

    // 2. 校验类型白名单
    uint32_t type = stream.ReadUInt32();
    if (!IsAllowedType(type)) {
        return napi_invalid_arg;
    }

    // 3. 递归深度限制
    if (stream.GetDepth() > MAX_DEPTH) {
        return napi_invalid_arg;
    }

    // 4. 严格校验大小
    size_t dataSize = stream.ReadSize();
    if (dataSize > MAX_DATA_SIZE) {
        return napi_invalid_arg;
    }

    return napi_ok;
}
```

## 安全最佳实践

### 输入验证

1. **所有外部输入必须校验**
   - N-API 参数
   - 文件路径
   - 网络数据
   - 用户配置

2. **验证内容而非仅类型**
   - 数值范围
   - 字符串长度
   - 路径规范性

### 内存安全

1. **优先使用现代 C++ 安全特性**
   - std::string 替代 char*
   - std::vector 替代裸数组
   - unique_ptr/shared_ptr 替代 raw pointer

2. **边界检查**
   - 所有数组访问
   - 所有指针运算

### 错误处理

1. **永不忽略错误码**
2. **统一的错误码定义**
3. **详细的错误日志**

## 相关文档

- [项目概览](01_Project_Overview.md)
- [架构说明](03_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
- [内部 API](05_Inner_API.md)
- [构建系统](06_Build_System.md)
