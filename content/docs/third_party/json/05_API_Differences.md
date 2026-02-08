# API 差异分析

## 概述

**本库在 OpenHarmony 中没有进行任何 API 修改。**

由于 nlohmann/json 是 header-only 库，且原生支持多种平台，OH 中使用的版本与上游版本完全一致。

## API 对比

### 完整 API 列表

以下 API 在 OH 中均可正常使用：

| 类别 | API | 状态 | 说明 |
|------|-----|------|------|
| **解析** | `json::parse()` | ✅ 可用 | 从字符串解析 |
| | `json::accept()` | ✅ 可用 | 不抛异常的解析 |
| **序列化** | `json::dump()` | ✅ 可用 | 序列化到字符串 |
| | `json::dump(n)` | ✅ 可用 | 格式化输出 |
| **访问** | `operator[]` | ✅ 可用 | 下标访问 |
| | `at()` | ✅ 可用 | 带检查的访问 |
| | `find()` | ✅ 可用 | 查找键 |
| | `contains()` | ✅ 可用 | 检查键是否存在 |
| **迭代** | `begin()`, `end()` | ✅ 可用 | 迭代器 |
| | `items()` | ✅ 可用 | 键值对迭代 |
| **转换** | `get<T>()` | ✅ 可用 | 类型转换 |
| | `get_to<T>()` | ✅ 可用 | 原地转换 |
| **检查** | `is_null()` | ✅ 可用 | 检查 null |
| | `is_object()` | ✅ 可用 | 检查对象 |
| | `is_array()` | ✅ 可用 | 检查数组 |
| **JSON Pointer** | `operator/()` | ✅ 可用 | JSON Pointer |
| | `patch()` | ✅ 可用 | 应用 Patch |
| | `diff()` | ✅ 可用 | 计算差异 |
| **二进制格式** | `to_bson()` | ✅ 可用 | 序列化为 BSON |
| | `to_cbor()` | ✅ 可用 | 序列化为 CBOR |
| | `to_msgpack()` | ✅ 可用 | 序列化为 MessagePack |
| | `to_ubjson()` | ✅ 可用 | 序列化为 UBJSON |
| | `from_bson()` | ✅ 可用 | 从 BSON 解析 |
| | `from_cbor()` | ✅ 可用 | 从 CBOR 解析 |
| | `from_msgpack()` | ✅ 可用 | 从 MessagePack 解析 |
| | `from_ubjson()` | ✅ 可用 | 从 UBJSON 解析 |

## 无差异说明

### 为什么不需要修改 API？

1. **Header-Only**
   - 整个库在头文件中实现
   - 无需修改源码即可使用

2. **标准 C++**
   - 使用标准 C++ 容器和类型
   - 不依赖平台特定 API

3. **完善的错误处理**
   - 异常机制处理错误
   - 可选的错误码模式

4. **跨平台设计**
   - 已在多种环境验证
   - 不含操作系统相关代码

## 预处理器宏

### 可用宏

以下宏在 OH 中均可正常使用：

```cpp
// 异常控制
#define JSON_NOEXCEPTION        // 禁用异常

// 转换控制
#define JSON_USE_IMPLICIT_CONVERSIONS  // 启用隐式转换

// 诊断控制
#define JSON_DIAGNOSTICS        // 启用详细错误信息

// 其他
#define NDEBUG                   // 禁用断言
#define JSON_ASSERT(x)           // 自定义断言
```

### 使用示例

```cpp
// 禁用异常处理
#define JSON_NOEXCEPTION
#include <nlohmann/json.hpp>

// 使用错误码
auto result = json::accept(str);
if (!result) {
    // 处理错误
}
```

## 命名空间

### 标准命名空间

```cpp
// 主命名空间
nlohmann::json

// 字面量命名空间
nlohmann::literals

// 详情命名空间 (谨慎使用)
nlohmann::detail
```

### OH 中的使用

```cpp
#include <nlohmann/json.hpp>

// 完整路径
nlohmann::json j;

// 别名
using json = nlohmann::json;
json j2;

// 字面量
using namespace nlohmann::literals;
json j3 = R"({"key": "value"})"_json;
```

## 类型差异

### OH 类型支持

| 类型 | 别名 | 状态 | 说明 |
|------|-----|------|------|
| `json` | `nlohmann::json` | ✅ | 主类型 |
| `json::string_t` | `std::string` | ✅ | 字符串类型 |
| `json::number_integer_t` | `int64_t` | ✅ | 整数类型 |
| `json::number_unsigned_t` | `uint64_t` | ✅ | 无符号整数 |
| `json::number_float_t` | `double` | ✅ | 浮点类型 |
| `json::array_t` | `std::vector<json>` | ✅ | 数组类型 |
| `json::object_t` | `std::map<std::string, json>` | ✅ | 对象类型 |
| `json::binary_t` | `std::vector<uint8_t>` | ✅ | 二进制类型 |
| `json::value_t` | `enum` | ✅ | 值类型枚举 |

### 自定义字符串类型

```cpp
// 使用自定义字符串类型
using json = nlohmann::basic_json<
    std::map,
    std::vector,
    std::string,
    bool,
    int64_t,
    uint64_t,
    double,
    std::allocator,
    nlohmann::adl_serializer,
    std::u8string
>;
```

## 扩展功能

### OH 可用的扩展

| 功能 | 状态 | 说明 |
|------|------|------|
| `ordered_json` | ✅ 可用 | 保持插入顺序 |
| `json_pointer` | ✅ 可用 | JSON Pointer 支持 |
| `json_patch` | ✅ 可用 | JSON Patch 支持 |
| `json_diff` | ✅ 可用 | JSON 差异计算 |
| 二进制格式 | ✅ 可用 | BSON, CBOR, MessagePack, UBJSON |

### 使用示例

```cpp
#include <nlohmann/json.hpp>
using json = nlohmann::json;

// 有序 JSON
#include <nlohmann/json/ordered_json.hpp>
using ordered_json = nlohmann::ordered_json;

// 使用
ordered_json oj;
oj["b"] = 1;
oj["a"] = 2;
// 保持插入顺序: {"b": 1, "a": 2}
```

## 编译器兼容性

### OH 环境

| 编译器 | 支持状态 | 说明 |
|--------|---------|------|
| **GCC** | ✅ | 4.8.5+ |
| **Clang** | ✅ | 3.4+ |
| **OH 编译器** | ✅ | 基于 LLVM |

### C++ 标准支持

| 标准 | 支持状态 | 特性使用 |
|------|---------|---------|
| **C++11** | ✅ | 最小要求 |
| **C++14** | ✅ | 完全支持 |
| **C++17** | ✅ | 推荐使用 |
| **C++20** | ✅ | 实验性支持 |

## 不兼容情况

### 已知不兼容场景

| 场景 | 原因 | 解决方案 |
|------|------|---------|
| 禁用 RTTI | 库内部使用 typeid | 保持 RTTI 启用 |
| 禁用异常 | 库默认使用异常 | 定义 JSON_NOEXCEPTION |
| 非 UTF-8 环境 | 只支持 UTF-8 | 确保输入为 UTF-8 |

### 替代方案

#### 禁用异常

```cpp
#define JSON_NOEXCEPTION
#include <nlohmann/json.hpp>

// 使用 accept() 而非 parse()
auto result = json::accept(str);
if (result) {
    json j = *result;
}
```

#### 禁用 RTTI

**注意**: 不支持，需要保持 RTTI 启用。

## 版本升级影响

### 升级到新版本

由于没有 API 修改，升级上游版本时：

| 步骤 | 描述 |
|------|------|
| 1. 下载新版本 | 从上游获取最新头文件 |
| 2. 替换头文件 | 更新 single_include 目录 |
| 3. 更新版本号 | 修改 bundle.json |
| 4. 测试验证 | 验证兼容性 |

### 潜在的破坏性变更

需要关注上游的以下变更：

- API 签名变化
- 行为变更
- 宏定义变化
- 依赖变化

## 总结

### 无差异原因

1. **Header-Only 设计**: 无需修改源码
2. **跨平台兼容**: 原生支持多种环境
3. **标准 C++**: 不依赖平台特定功能
4. **完善的配置**: 通过宏适应不同环境

### OH 使用建议

1. **直接使用**: 按上游文档使用所有 API
2. **关注上游**: 跟进上游的更新和变更
3. **测试验证**: 升级时充分测试
