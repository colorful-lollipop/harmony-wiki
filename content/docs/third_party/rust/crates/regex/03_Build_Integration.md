# OH 构建适配

## 构建配置概述

regex 库通过 OpenHarmony 的 `ohos_cargo_crate` 模板集成到构建系统。该模板是专门为 Rust Cargo 包设计的构建适配层，将 Cargo 包转换为 GN 构建系统的目标。

## BUILD.gn 完整配置

**文件位置**: `//third_party/rust/crates/regex/BUILD.gn`

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "regex"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "1.7.1"
    cargo_pkg_authors = "The Rust Project Developers"
    cargo_pkg_name = "regex"
    deps = [
        "//third_party/rust/crates/aho-corasick:lib",
        "//third_party/rust/crates/memchr:lib",
        "//third_party/rust/crates/regex/regex-syntax:lib",
    ]
    features = [
        "aho-corasick",
        "memchr",
        "perf",
        "perf-cache",
        "perf-dfa",
        "perf-inline",
        "perf-literal",
        "std",
        "unicode",
        "unicode-age",
        "unicode-bool",
        "unicode-case",
        "unicode-gencat",
        "unicode-perl",
        "unicode-script",
        "unicode-segment",
    ]
    module_output_extension = ".rlib"
    part_name = "rust_regex"
    subsystem_name = "thirdparty"
}
```

## 配置详解

### ohos_cargo_crate 模板参数

| 参数 | 值 | 说明 |
|------|-----|------|
| **crate_name** | regex | 内部 crate 名称 |
| **crate_type** | rlib | Rust 静态库类型（.a 归档） |
| **crate_root** | src/lib.rs | crate 入口文件 |
| **edition** | 2018 | Rust Edition 版本 |
| **cargo_pkg_version** | 1.7.1 | 与 Cargo.toml 版本一致 |
| **cargo_pkg_name** | regex | 与 Cargo.toml name 一致 |

### 依赖配置（deps）

```gn
deps = [
    "//third_party/rust/crates/aho-corasick:lib",      # 多模式字符串匹配
    "//third_party/rust/crates/memchr:lib",            # 高性能字节搜索
    "//third_party/rust/crates/regex/regex-syntax:lib", # 正则语法解析
]
```

| 依赖项 | 用途 | 版本要求 |
|--------|------|---------|
| aho-corasick | 多模式字符串同时匹配算法 | 来自 OH 生态版本 |
| memchr | 高性能字节/字符搜索 | 来自 OH 生态版本 |
| regex-syntax | 正则表达式语法解析和 AST | regex 子 crate |

### Features 配置

#### 性能优化类

| Feature | 启用状态 | 功能说明 |
|---------|---------|---------|
| **perf** | ✅ | 综合性能优化开关 |
| **perf-cache** | ✅ | 匹配结果缓存，减少重复计算 |
| **perf-dfa** | ✅ | 确定性有限自动机引擎优化 |
| **perf-inline** | ✅ | 热点函数内联优化 |
| **perf-literal** | ✅ | 字面量字符串加速匹配 |
| **memchr** | ✅ | 使用 memchr crate 进行字节搜索 |
| **aho-corasick** | ✅ | 使用 aho-corasick 进行多模式匹配 |

#### Unicode 支持类

| Feature | 启用状态 | 功能说明 |
|---------|---------|---------|
| **unicode** | ✅ | Unicode 综合支持开关 |
| **unicode-age** | ✅ | 字符首次出现的 Unicode 版本 |
| **unicode-bool** | ✅ | 字符布尔属性（is_alphabetic 等） |
| **unicode-case** | ✅ | Unicode 大小写转换规则 |
| **unicode-gencat** | ✅ | 通用类别（Letter, Number 等） |
| **unicode-perl** | ✅ | Perl 兼容的 Unicode 规则 |
| **unicode-script** | ✅ | Unicode 脚本属性 |
| **unicode-segment** | ✅ | 词、句、行边界检测 |

#### 基础类

| Feature | 启用状态 | 功能说明 |
|---------|---------|---------|
| **std** | ✅ | 标准库支持（禁用则使用 core only） |

### 构建产物配置

```gn
module_output_extension = ".rlib"
part_name = "rust_regex"
subsystem_name = "thirdparty"
```

| 配置 | 值 | 说明 |
|------|-----|------|
| **module_output_extension** | .rlib | Rust 静态库文件扩展名 |
| **part_name** | rust_regex | 组件名，用于 hb 编译 |
| **subsystem_name** | thirdparty | 属于 thirdparty 子系统 |

## 与上游 Cargo.toml 的对比

### Cargo.toml 默认配置

```toml
[features]
default = ["std", "perf-literal", "unicode-perl"]
perf-literal = ["dep:memchr", "dep:regex-syntax/perf-literal"]
unicode-perl = ["dep:regex-syntax/unicode-perl"]
# ... 其他可选特性
```

### OH 配置差异

| 配置项 | 上游默认 | OH 配置 | 差异 |
|--------|---------|---------|------|
| **std** | 启用 | 启用 | ✅ 相同 |
| **perf-literal** | 启用 | 启用 | ✅ 相同 |
| **unicode-perl** | 启用 | 启用 | ✅ 相同 |
| **perf-cache** | 可选 | **启用** | OH 额外启用 |
| **perf-dfa** | 可选 | **启用** | OH 额外启用 |
| **perf-inline** | 可选 | **启用** | OH 额外启用 |
| **unicode-age** | 可选 | **启用** | OH 额外启用 |
| **unicode-bool** | 可选 | **启用** | OH 额外启用 |
| **unicode-case** | 可选 | **启用** | OH 额外启用 |
| **unicode-gencat** | 可选 | **启用** | OH 额外启用 |
| **unicode-script** | 可选 | **启用** | OH 额外启用 |
| **unicode-segment** | 可选 | **启用** | OH 额外启用 |

**结论**：OH 配置启用了上游所有可选的优化和 Unicode 特性，追求最佳性能和功能完整性。

## 构建产物

### 输出文件

```
out/.../obj/third_party/rust/crates/regex/libregex.rlib
```

### 库文件规格

| 属性 | 值 |
|------|-----|
| **类型** | Rust 静态库（rlib） |
| **链接方式** | 静态链接 |
| **依赖传递** | 依赖的 crates 会被静态链接到使用方 |

## 组件声明

### bundle.json 组件配置

```json
{
  "name": "@ohos/rust_regex",
  "description": "A Rust library that provides support for regular expressions.",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/regex"
  },
  "component": {
    "name": "rust_regex",
    "subsystem": "thirdparty",
    "adapted_system_type": ["standard"],
    "build": {
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/regex:lib"
        }
      ]
    }
  }
}
```

### Inner Kit 导出

| 导出项 | 路径 | 类型 |
|--------|------|------|
| lib | //third_party/rust/crates/regex:lib | code-segment |

## 编译注意事项

### 编译时间

regex 库由于包含大量 Unicode 数据表，编译时间较长。建议：

- 使用预编译产物
- 避免频繁重新编译

### 依赖链

```
regex
├── aho-corasick
│   └── memchr
└── regex-syntax
    └── unicode（大量数据）
```

### 磁盘占用

Unicode 数据表会使最终的二进制文件增大，但这是功能完整性的必要代价。

## 版本升级检查清单

升级 regex 库版本时，需验证：

- [ ] Cargo.toml 版本号更新
- [ ] BUILD.gn cargo_pkg_version 更新
- [ ] features 列表覆盖新版本特性
- [ ] 测试 bindgen 功能正常
- [ ] 测试 env_logger 功能正常
- [ ] 运行 regex 测试套件
- [ ] 检查是否有安全更新需要同步
