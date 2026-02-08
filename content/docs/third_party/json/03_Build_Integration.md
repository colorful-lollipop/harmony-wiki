# OH 构建适配

## 构建配置概览

### BUILD.gn 结构

```gn
import("//build/ohos.gni")

config("nlohmann_json_config") {
  include_dirs = [
    "single_include/",
    "single_include/nlohmann",
  ]
  
  cflags_cc = [ "-fexceptions" ]
  cflags_objcc = cflags_cc
}

ohos_static_library("nlohmann_json_static") {
  public_configs = [ ":nlohmann_json_config" ]
  part_name = "json"
  subsystem_name = "thirdparty"
}
```

## 配置详解

### 头文件配置

#### include_dirs

```gn
include_dirs = [
  "single_include/",         # 合并后的主头文件目录
  "single_include/nlohmann", # 原始头文件目录
]
```

**说明**:

- `single_include/`: 包含合并后的 `json.hpp` 文件，可以直接 `#include <nlohmann/json.hpp>`
- `single_include/nlohmann/`: 包含按模块分割的原始头文件，支持更细粒度的包含

**推荐使用方式**:

```cpp
// 推荐：使用合并后的头文件
#include <nlohmann/json.hpp>

// 备选：使用原始头文件
#include <nlohmann/json/json.hpp>
#include <nlohmann/json/ordered_json.hpp>
```

### 编译器标志配置

#### cflags_cc

```gn
cflags_cc = [ "-fexceptions" ]
```

**作用**: 启用 C++ 异常支持

**原因**:
- nlohmann/json 广泛使用 C++ 异常进行错误处理
- 解析错误、类型错误等都会抛出 `nlohmann::json::exception` 及其子类
- 禁用异常会导致未定义行为

**替代方案** (如果需要禁用异常):

```cpp
// 在包含头文件前定义
#define JSON_NOEXCEPTION

// 使用错误码代替异常
json j;
auto result = json::accept(str);  // 不抛异常
if (!result) {
    // 处理错误
}
```

### 静态库目标

#### ohos_static_library

```gn
ohos_static_library("nlohmann_json_static") {
  public_configs = [ ":nlohmann_json_config" ]
  part_name = "json"
  subsystem_name = "thirdparty"
}
```

**配置说明**:

| 属性 | 值 | 说明 |
|------|-----|------|
| **目标类型** | ohos_static_library | OH 静态库目标 |
| **part_name** | json | 组件名称 |
| **subsystem_name** | thirdparty | 所属子系统 |
| **public_configs** | nlohmann_json_config | 导出配置供依赖者使用 |

## bundle.json 配置

```json
{
    "name": "@ohos/json",
    "version": "3.1",
    "component": {
        "name": "json",
        "subsystem": "thirdparty",
        "adapted_system_type": ["standard"],
        "build": {
            "inner_kits": [
                {
                    "header": {
                        "header_base": "//third_party/json/single_include",
                        "header_files": []
                    },
                    "name": "//third_party/json:nlohmann_json_static"
                }
            ]
        }
    }
}
```

### 配置说明

#### inner_kits

```json
"inner_kits": [
    {
        "header": {
            "header_base": "//third_party/json/single_include",
            "header_files": []
        },
        "name": "//third_party/json:nlohmann_json_static"
    }
]
```

**作用**: 声明该组件提供的头文件和库

| 属性 | 值 | 说明 |
|------|-----|------|
| **header_base** | //third_party/json/single_include | 头文件根目录 |
| **header_files** | [] | 导出的头文件列表，空数组表示全部导出 |
| **name** | //third_party/json:nlohmann_json_static | 静态库目标名称 |

## 依赖声明方式

### 在其他模块中使用

```gn
// 方式1: 通过 inner_kits 依赖
ohos_static_library("my_module") {
    deps = [
        "//third_party/json:nlohmann_json_static"
    ]
}

// 方式2: 直接引用头文件
ohos_static_library("my_module") {
    include_dirs = [
        "//third_party/json/single_include",
    ]
}
```

### 使用头文件

```cpp
#include <nlohmann/json.hpp>

using json = nlohmann::json;

json j = {{"key", "value"}};
```

## 与上游构建系统的差异

### 上游构建系统

上游支持多种构建系统：

| 构建系统 | 用途 |
|---------|------|
| **单头文件** | 直接包含 json.hpp |
| **CMake** | 完整的构建和测试配置 |
| **Meson** | Meson 构建系统支持 |
| **Bazel** | Bazel 构建系统支持 |

### OH 构建适配

| 方面 | 上游 | OH |
|------|------|-----|
| **构建产物** | 可选 (header-only) | 静态库 |
| **编译选项** | 用户配置 | 预定义 (-fexceptions) |
| **头文件** | 单个或多个 | single_include 目录 |

### 差异说明

1. **静态库目标**: OH 创建了静态库目标，便于依赖管理和链接控制
2. **预配置编译选项**: OH 预配置了异常支持，简化用户使用
3. **头文件组织**: 使用 single_include 目录，简化包含路径

## 头文件目录结构

```
third_party/json/
├── single_include/
│   ├── nlohmann/
│   │   ├── json.hpp              # 主头文件 (合并)
│   │   ├── json_fwd.hpp          # 前向声明
│   │   ├── detail/                # 实现细节
│   │   │   ├── input/             # 输入解析
│   │   │   ├── output/            # 输出序列化
│   │   │   ├── iterators/         # 迭代器
│   │   │   └── macros/            # 宏定义
│   │   └── ordered_json.hpp       # 有序 JSON
│   └── LICENSE.MIT                # MIT 许可证
├── include/                       # 原始头文件 (未使用)
│   └── nlohmann/
├── CMakeLists.txt                # CMake 配置 (上游)
├── BUILD.gn                       # OH 构建配置
└── bundle.json                    # OH 组件配置
```

## 编译注意事项

### C++ 标准要求

- **最低要求**: C++11
- **推荐**: C++17

### 编译器支持

```cpp
// GCC
g++ -std=c++11 -fexceptions ...

// Clang
clang++ -std=c++11 -fexceptions ...

// OH 编译器
# 使用预定义的编译选项
```

### 常见问题

#### 问题1: 禁用异常后编译失败

**原因**: 库默认使用异常处理错误

**解决方案**:

```cpp
#define JSON_NOEXCEPTION
#include <nlohmann/json.hpp>

// 使用错误码 API
auto result = json::accept(str);
if (!result) {
    // 处理错误
}
```

#### 问题2: 头文件路径错误

**原因**: 包含路径未正确配置

**解决方案**:

```gn
# 确保在 BUILD.gn 中配置正确的 include_dirs
include_dirs = [
    "//third_party/json/single_include",
]
```

## 最佳实践

### 1. 使用合并头文件

```cpp
// 推荐
#include <nlohmann/json.hpp>

// 不推荐 (如果不需要特定模块)
#include <nlohmann/json/detail/input/input_adapters.hpp>
```

### 2. 启用异常处理

```gn
# 在 BUILD.gn 中确保启用异常
cflags_cc = [ "-fexceptions" ]
```

### 3. 选择合适的 C++ 标准

```cpp
// 如果需要最新特性，使用 C++17
json j = {
    {"name", "OpenHarmony"},
    {"version", 3.1}
};

// C++17 结构化绑定
for (auto& [key, value] : j.items()) {
    // ...
}
```

## 版本升级流程

### 1. 下载新版本

```bash
# 从上游下载新版本
wget https://github.com/nlohmann/json/releases/download/v{新版本}/json.hpp

# 或使用 amalgamation 工具生成
python amalgamate.py
```

### 2. 更新文件

```bash
# 替换 single_include 目录
cp json.hpp third_party/json/single_include/nlohmann/json.hpp
```

### 3. 更新配置

```json
// bundle.json
"version": "{新版本}"
```

### 4. 测试验证

```bash
# 编译测试
hb build ...

# 运行单元测试
# ...
```

## 总结

### 配置要点

1. **头文件路径**: 使用 single_include 目录
2. **异常支持**: 必须启用 (-fexceptions)
3. **静态库**: 便于依赖管理
4. **C++ 标准**: 至少 C++11

### 优势

1. **简单适配**: 无需复杂的 OH 特定修改
2. **原生兼容**: 库本身支持 OH 环境
3. **维护简单**: 可直接同步上游版本
