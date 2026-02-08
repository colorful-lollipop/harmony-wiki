# JSON for Modern C++ 库概述

## 简介

**JSON for Modern C++** 是由 Niels Lohmann 开发的一个现代 C++ JSON 库。该库设计目标包括：

- **直观的语法**: 类似 Python 的 JSON 操作体验
- **简单的集成**: 单个头文件，无需复杂构建系统
- **内存效率**: 使用标准 C++ 容器，内存占用可控

## 核心功能

### JSON 解析与序列化

```cpp
#include <nlohmann/json.hpp>
using json = nlohmann::json;

// 从字符串解析
json j = json::parse("{\"name\": \"OpenHarmony\"}");

// 序列化
std::string str = j.dump();           // {"name":"OpenHarmony"}
std::string pretty = j.dump(4);       // 格式化输出
```

### JSON 对象操作

```cpp
// 创建对象
json obj;
obj["name"] = "OpenHarmony";
obj["version"] = 3.1;
obj["features"] = {"分布式", "原生支持"};

// 访问元素
auto name = obj["name"];
auto version = obj.at("version");  // 带检查的访问

// 遍历
for (auto& [key, value] : obj.items()) {
    // 处理 key-value 对
}
```

### JSON 数组操作

```cpp
// 创建数组
json arr = {1, 2, 3, 4, 5};

// 添加元素
arr.push_back(6);
arr.emplace_back(7);

// 遍历
for (auto& element : arr) {
    // 处理元素
}
```

### 类型转换

```cpp
// 从 C++ 类型转换到 JSON
json j;
j["string"] = std::string("hello");
j["number"] = 42;
j["float"] = 3.14;
j["boolean"] = true;
j["null"] = nullptr;

// 从 JSON 转换到 C++
std::string str = j["string"].get<std::string>();
int num = j["number"].get<int>();
```

### STL 容器支持

```cpp
// 向 STL 容器转换
std::vector<int> vec = {1, 2, 3};
json j_vec = vec;  // [1, 2, 3]

std::map<std::string, int> map = {{"a", 1}, {"b", 2}};
json j_map = map;  // {"a": 1, "b": 2}

// 从 JSON 转换
auto vec2 = j_vec.get<std::vector<int>>();
auto map2 = j_map.get<std::map<std::string, int>>();
```

### 用户自定义类型

```cpp
struct Person {
    std::string name;
    int age;
    std::vector<std::string> hobbies;
};

// 定义转换函数
namespace nlohmann {
    void to_json(json& j, const Person& p) {
        j = json{{"name", p.name}, {"age", p.age}, {"hobbies", p.hobbies}};
    }
    
    void from_json(const json& j, Person& p) {
        j.at("name").get_to(p.name);
        j.at("age").get_to(p.age);
        j.at("hobbies").get_to(p.hobbies);
    }
}

// 使用
Person p{"Alice", 30, {"reading", "coding"}};
json j = p;
Person p2 = j.get<Person>();
```

### JSON Pointer

```cpp
json j = {
    {"user", {
        {"name", "Alice"},
        {"address", {
            {"city", "Beijing"},
            {"street", "Main St"}
        }}
    }}
};

// 使用 JSON Pointer 访问
auto name = j["/user/name"_json_pointer];  // "Alice"
auto city = j["/user/address/city"_json_pointer];  // "Beijing"
```

### JSON Patch (RFC 6902)

```cpp
json original = R"({
    "name": "OpenHarmony",
    "version": 3.0
})"_json;

json patch = R"([
    {"op": "replace", "path": "/name", "value": "OH"},
    {"op": "add", "path": "/features", "value": ["分布式"]},
    {"op": "remove", "path": "/version"}
])"_json;

// 应用 Patch
json result = original.patch(patch);
```

### 二进制格式支持

```cpp
json j = {"compact", true, "schema", 0};

// 序列化为各种二进制格式
auto bson = json::to_bson(j);
auto cbor = json::to_cbor(j);
auto msgpack = json::to_msgpack(j);
auto ubjson = json::to_ubjson(j);

// 从二进制格式解析
json j_bson = json::from_bson(bson);
json j_cbor = json::from_cbor(cbor);
```

## 技术规格

### 支持的编译器

- GCC 4.8.5 - 13.0
- Clang 3.4 - 16.0
- Apple Clang 9.1 - 14.0
- MSVC 2015 - 2022
- Intel C++ Compiler 17.0+

### C++ 标准支持

- C++11 (最低要求)
- C++17 (推荐)
- C++20 (实验性支持)

### 依赖

**无外部依赖** - 这是该库的核心优势之一。

## OH 中的定位

在 OpenHarmony 生态系统中，该库主要用于：

1. **配置解析**: 解析应用和系统配置文件
2. **数据交换**: 模块间的 JSON 格式数据交换
3. **序列化支持**: 持久化和网络传输的 JSON 序列化
4. **测试支持**: 测试框架的 JSON 断言和数据生成

## 上游资源

- **GitHub**: https://github.com/nlohmann/json
- **文档**: https://json.nlohmann.me/
- **API 参考**: https://json.nlohmann.me/api/basic_json/
- **FAQ**: https://json.nlohmann.me/home/faq/
