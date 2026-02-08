# GoogleTest 在 OpenHarmony 中的构建集成

> BUILD.gn 构建系统适配分析

---

## 概述

OpenHarmony 使用 GN (Generate Ninja) 构建系统替代上游的 CMake。`BUILD.gn` 文件定义了 GoogleTest 的构建配置，包括 7 个目标、8 个配置块，以及华为 hwext 扩展的集成。

**关键特点**：
- 所有目标为静态库 (`static_library`)
- 所有目标标记为仅测试用途 (`testonly = true`)
- 显式使用 C++17 标准
- 包含华为 hwext 扩展源文件

---

## BUILD.gn 结构总览

```
BUILD.gn (220 行)
│
├── Config 定义 (8 个配置块)
│   ├── gtest_private_config         # gtest 私有配置（C++17）
│   ├── gtest_private_config_rtti    # gtest + RTTI 私有配置
│   ├── gtest_private_config_main    # gtest main 私有配置
│   ├── gtest_private_config_rtti_main  # gtest + RTTI + main 私有配置
│   ├── gtest_config                # gtest 公共配置（警告抑制）
│   ├── gmock_private_config        # gmock 私有配置
│   ├── gmock_private_config_rtti   # gmock + RTTI 私有配置
│   └── gmock_config               # gmock 公共配置（override 警告抑制）
│
├── 源文件列表 (2 个变量)
│   ├── sources_files              # gtest 源文件（包含 hwext）
│   └── gmock_sources_files       # gmock 源文件
│
└── Static Library 目标定义 (7 个目标)
    ├── gtest                     # 标准 gtest 库
    ├── gtest_rtti                # 启用 RTTI 的 gtest
    ├── gtest_rtti_main           # gtest + RTTI + main
    ├── gtest_main                # gtest + main
    ├── gmock                     # 标准 gmock 库
    ├── gmock_rtti                # 启用 RTTI 的 gmock
    └── gmock_main                # gmock + main
```

---

## 目标定义详解

### GTest 目标（4个）

#### 1. gtest（标准库）
```gn
static_library("gtest") {
  testonly = true
  public = [
    "googletest/include/gtest/gtest-spi.h",
    "googletest/include/gtest/gtest.h",
  ]
  sources = sources_files
  sources -= [ "googletest/src/gtest-all.cc" ]  # 排除汇总文件
  public_configs = [ ":gtest_config" ]
  configs += [ ":gtest_private_config" ]
  configs -= [ "//build/config/coverage:default_coverage" ]
}
```

**特点**：
- 标准 gtest 库，不带 RTTI 和 main 函数
- 移除默认覆盖率配置（OH 构建系统有自己的覆盖率方案）
- 包含 hwext 扩展源文件

**适用场景**：需要自定义 main 函数的测试项目。

---

#### 2. gtest_rtti（启用 RTTI）
```gn
static_library("gtest_rtti") {
  testonly = true
  public = [
    "googletest/include/gtest/gtest-spi.h",
    "googletest/include/gtest/gtest.h",
  ]
  sources = sources_files
  sources -= [ "googletest/src/gtest-all.cc" ]
  public_configs = [ ":gtest_config" ]
  configs += [ ":gtest_private_config_rtti" ]  # +RTTI
  configs -= [ "//build/config/coverage:default_coverage" ]
}
```

**特点**：
- 启用 RTTI（运行时类型信息）
- 其他与 `gtest` 相同

**适用场景**：测试代码需要使用 RTTI 功能（如 `dynamic_cast`）。

---

#### 3. gtest_main（带 main 函数）
```gn
static_library("gtest_main") {
  testonly = true
  sources = [ "googletest/src/gtest_main.cc" ]
  public_configs = [ ":gtest_config" ]
  deps = [ ":gtest" ]
  configs += [ ":gtest_private_config_main" ]
  configs -= [ "//build/config/coverage:default_coverage" ]
}
```

**特点**：
- 包含 `main()` 函数，可直接运行测试
- 依赖 `gtest` 库

**适用场景**：快速编写测试，无需自定义 main 函数。

---

#### 4. gtest_rtti_main（RTTI + main）
```gn
static_library("gtest_rtti_main") {
  testonly = true
  sources = [ "googletest/src/gtest_main.cc" ]
  public_configs = [ ":gtest_config" ]
  deps = [ ":gtest_rtti" ]
  configs += [ ":gtest_private_config_rtti_main" ]
  configs -= [ "//build/config/coverage:default_coverage" ]
}
```

**特点**：
- 同时启用 RTTI 和提供 main 函数
- 依赖 `gtest_rtti` 库

**适用场景**：需要 RTTI 且希望使用默认 main 函数的测试项目。

---

### GMock 目标（3个）

#### 1. gmock（标准库）
```gn
static_library("gmock") {
  testonly = true
  public = [ "googlemock/include/gmock/gmock.h" ]
  sources = gmock_sources_files
  sources -= [ "googlemock/src/gmock-all.cc" ]
  public_configs = [ ":gmock_config" ]
  configs += [ ":gmock_private_config" ]
  configs -= [ "//build/config/coverage:default_coverage" ]
  deps = [ ":gtest" ]  # 依赖 gtest
}
```

**特点**：
- 标准 gmock 库
- 依赖 gtest（gmock 依赖 gtest 的基础功能）
- 抑制 `override` 警告（MOCK_METHOD 宏不使用 `override` 关键字）

**适用场景**：需要模拟对象的测试项目。

---

#### 2. gmock_rtti（启用 RTTI）
```gn
static_library("gmock_rtti") {
  testonly = true
  public = [ "googlemock/include/gmock/gmock.h" ]
  sources = gmock_sources_files
  sources -= [ "googlemock/src/gmock-all.cc" ]
  public_configs = [ ":gmock_config" ]
  configs += [ ":gmock_private_config_rtti" ]
  configs -= [ "//build/config/coverage:default_coverage" ]
  deps = [ ":gtest_rtti" ]  # 依赖 gtest_rtti
}
```

**特点**：
- 启用 RTTI 的 gmock
- 依赖 `gtest_rtti`

**适用场景**：需要 RTTI 和模拟对象的测试项目。

---

#### 3. gmock_main（带 main 函数）
```gn
static_library("gmock_main") {
  testonly = true
  sources = [ "googlemock/src/gmock_main.cc" ]
  public_configs = [ ":gmock_config", ":gtest_config" ]
  deps = [
    ":gmock",
    ":gtest",
  ]
  configs += [ ":gmock_private_config_main" ]
  configs -= [ "//build/config/coverage:default_coverage" ]
}
```

**特点**：
- 提供 main 函数
- 同时依赖 gmock 和 gtest

**适用场景**：需要模拟对象且希望使用默认 main 函数的测试项目。

---

## 配置块详解

### GTest 配置

#### gtest_config（公共配置）
```gn
config("gtest_config") {
  include_dirs = [ "googletest/include" ]
  cflags_cc = [
    "-Wno-float-equal",           # 抑制浮点数直接比较警告
    "-Wno-sign-compare",          # 抑制有符号/无符号比较警告
    "-Wno-reorder-init-list",     # 抑制初始化列表重排序警告
  ]
  if (is_mingw) {
    cflags_cc += [
      "-Wno-unused-const-variable",
      "-Wno-unused-private-field",
    ]
  }
}
```

**作用**：
- 设置公共头文件搜索路径
- 抑制 gtest 代码中的常见警告
- MinGW 平台额外处理

**警告抑制说明**：
- `float-equal`: gtest 中某些测试场景需要直接比较浮点数
- `sign-compare`: gtest 内部代码存在有符号/无符号比较
- `reorder-init-list`: 初始化列表顺序不影响功能

---

#### gtest_private_config（私有配置）
```gn
config("gtest_private_config") {
  visibility = [ ":*" ]  # 仅内部可见
  include_dirs = [ "googletest" ]
  cflags_cc = [ "-std=c++17" ]
}
```

**作用**：
- 设置内部头文件搜索路径
- 强制使用 C++17 标准

---

#### gtest_private_config_rtti（RTTI 私有配置）
```gn
config("gtest_private_config_rtti") {
  visibility = [ ":*" ]
  include_dirs = [ "googletest" ]
  cflags = [ "-frtti" ]
  cflags_objcc = [ "-frtti" ]
  cflags_cc = [
    "-std=c++17",
    "-frtti",
  ]
}
```

**作用**：
- 启用 RTTI（`-frtti`）
- 同时应用于 C++、Objective-C++ 代码

---

#### gtest_private_config_main（main 私有配置）
```gn
config("gtest_private_config_main") {
  cflags_cc = [ "-std=c++17" ]
}
```

**作用**：
- 为 main 函数源文件设置 C++17 标准

---

#### gtest_private_config_rtti_main（RTTI + main 私有配置）
```gn
config("gtest_private_config_rtti_main") {
  cflags_cc = [ "-std=c++17" ]
}
```

**作用**：
- 为 RTTI + main 函数源文件设置 C++17 标准

---

### GMock 配置

#### gmock_config（公共配置）
```gn
config("gmock_config") {
  include_dirs = [ "googlemock/include" ]
  cflags_cc = [
    "-Wno-inconsistent-missing-override",  # 抑制 override 警告
  ]
}
```

**作用**：
- 设置 gmock 公共头文件搜索路径
- 抑制 `MOCK_METHOD` 宏的 override 警告

**警告抑制说明**：
- `inconsistent-missing-override`: gmock 的 `MOCK_METHOD` 宏生成的代码不使用 `override` 关键字，这会导致用户代码中的 override 检查报警。上游 issue: https://github.com/google/googletest/issues/533

---

#### gmock_private_config（私有配置）
```gn
config("gmock_private_config") {
  visibility = [ ":*" ]
  include_dirs = [ "googlemock" ]
}
```

**作用**：
- 设置 gmock 内部头文件搜索路径

---

#### gmock_private_config_rtti（RTTI 私有配置）
```gn
config("gmock_private_config_rtti") {
  visibility = [ ":*" ]
  include_dirs = [ "googlemock/include" ]
  cflags = [ "-frtti" ]
  cflags_objcc = [ "-frtti" ]
  cflags_cc = [ "-frtti" ]
}
```

**作用**：
- 启用 RTTI

---

#### gmock_private_config_main（main 私有配置）
```gn
config("gmock_private_config_main") {
  cflags_cc = [ "-std=c++17" ]
}
```

**作用**：
- 为 gmock main 函数源文件设置 C++17 标准

---

## 华为 hwext 扩展集成

### 源文件中的 hwext 引用

BUILD.gn 在 `sources_files` 变量中包含了所有 hwext 扩展源文件：

```gn
sources_files = [
  # 标准 gtest 头文件
  "googletest/include/gtest/gtest-death-test.h",
  "googletest/include/gtest/gtest-matchers.h",
  # ... 其他标准头文件 ...

  # hwext 扩展头文件（5个）
  "googletest/include/gtest/hwext/gtest-ext.h",
  "googletest/include/gtest/hwext/gtest-filter.h",
  "googletest/include/gtest/hwext/gtest-multithread.h",
  "googletest/include/gtest/hwext/gtest-tag.h",
  "googletest/include/gtest/hwext/utils.h",

  # 标准 gtest 源文件
  "googletest/src/gtest-all.cc",  # 被排除，使用单独编译
  "googletest/src/gtest-assertion-result.cc",
  "googletest/src/gtest-death-test.cc",
  # ... 其他标准源文件 ...

  # hwext 扩展源文件（5个）
  "googletest/src/hwext/gtest-ext.cc",
  "googletest/src/hwext/gtest-filter.cc",
  "googletest/src/hwext/gtest-multithread.cpp",
  "googletest/src/hwext/gtest-tag.cc",
  "googletest/src/hwext/gtest-utils.cc",
]
```

### 集成方式
- **与上游代码同编译**：hwext 源文件与 gtest 上游源文件一起编译
- **无额外依赖**：hwext 扩展不依赖额外的第三方库
- **命名空间隔离**：使用 `testing::ext` 命名空间与上游分离

---

## 与上游 CMake 的差异对比

| 特性 | OH BUILD.gn | 上游 CMake |
|------|-------------|------------|
| **构建系统** | GN (Generate Ninja) | CMake |
| **库类型** | 仅静态库 | 静态/动态可选（`BUILD_SHARED_LIBS`） |
| **目标数量** | 7 个精简目标 | 更多变体（含测试、样本等） |
| **C++ 标准** | 显式 C++17 | 依赖环境，默认 C++14+ |
| **RTTI 支持** | 独立 rtti/rtti_main 目标 | 通过选项控制（`cxx_no_rtti`） |
| **安装支持** | 无 install 规则 | 完整 install 规则（`install_project`） |
| **测试构建** | 不包含自测试 | 可选自测试（`gtest_build_tests`） |
| **样本程序** | 不包含 | 可选构建（`gtest_build_samples`） |
| **覆盖率** | 显式移除默认覆盖率 | 支持覆盖率配置 |
| **扩展代码** | 包含 hwext 扩展 | 无 |
| **Hermetic Build** | 不支持 | 支持（`hermetic_build.cmake`） |
| **Abseil 集成** | 无 | 可选（`GTEST_HAS_ABSL`） |
| **目标可见性** | 使用 `visibility` 限制 | 无此概念 |
| **配置粒度** | 多个精细 `config()` 块 | 集中配置（`config_compiler_and_linker()`） |

---

## 构建选项详解

### C++ 标准
OH 强制使用 C++17 标准：
```gn
cflags_cc = [ "-std=c++17" ]
```

**原因**：
- 上游要求的 C++14 是最低要求
- OH 整体代码基使用 C++17
- 确保测试代码与生产代码的一致性

### 警告抑制
| 警告选项 | 作用 | 原因 |
|----------|------|------|
| `-Wno-float-equal` | 抑制浮点数直接比较警告 | gtest 中某些测试场景需要直接比较 |
| `-Wno-sign-compare` | 抑制有符号/无符号比较警告 | gtest 内部代码存在此模式 |
| `-Wno-reorder-init-list` | 抑制初始化列表重排序警告 | 功能不受影响，避免噪声 |
| `-Wno-inconsistent-missing-override` | 抑制 override 警告 | MOCK_METHOD 宏不使用 override |

### RTTI 控制
- **默认关闭**：RTTI 会增加二进制大小和运行时开销
- **独立目标**：需要 RTTI 的测试使用 `gtest_rtti` 等目标
- **编译选项**：`-frtti` 启用 RTTI

### 覆盖率
移除默认覆盖率配置：
```gn
configs -= [ "//build/config/coverage:default_coverage" ]
```

**原因**：
- OH 构建系统有自己的覆盖率收集方案
- 避免与 OH 默认配置冲突

### 条件编译
```gn
if (is_mingw) {
  cflags_cc += [
    "-Wno-unused-const-variable",
    "-Wno-unused-private-field",
  ]
}
```

**MinGW 平台特殊处理**：抑制 Windows 平台特有的警告。

---

## 使用方式

### 在其他 BUILD.gn 中引用

#### 基础引用
```gn
import("//build/ohos.gni")

ohos_unittest("my_test") {
  sources = [ "my_test.cc" ]
  deps = [
    "//third_party/googletest:gtest",
  ]
}
```

#### 使用 main 函数
```gn
ohos_unittest("my_test") {
  sources = [ "my_test.cc" ]
  deps = [
    "//third_party/googletest:gtest_main",  # 自动提供 main()
  ]
}
```

#### 使用 gmock
```gn
ohos_unittest("my_test") {
  sources = [ "my_test.cc" ]
  deps = [
    "//third_party/googletest:gmock",
    "//third_party/googletest:gmock_main",
  ]
}
```

#### 启用 RTTI
```gn
ohos_unittest("my_test") {
  sources = [ "my_test.cc" ]
  deps = [
    "//third_party/googletest:gtest_rtti",  # 启用 RTTI
  ]
}
```

### 典型使用场景

#### 1. 驱动层测试
```gn
# drivers/hdf_core/adapter/uhdf/test/unittest/osal/BUILD.gn
ohos_unittest("osal_unittest") {
  subsystem_name = "hdf"
  part_name = "hdf_adapter"
  module_out_path = "test/unittest/osal"
  sources = [ "osal_test.cpp" ]
  deps = [
    "//third_party/googletest:gtest",
    "//third_party/googletest:gmock",
  ]
}
```

#### 2. 组件单元测试
```gn
# foundation/xxx/xxx/test/unittest/BUILD.gn
ohos_unittest("component_unittest") {
  module_out_path = "test/unittest"
  sources = [
    "test1.cpp",
    "test2.cpp",
  ]
  deps = [
    ":component",
    "//third_party/googletest:gtest_main",
  ]
}
```

---

## 性能与资源影响

### 静态库特点
| 特性 | 影响 |
|------|------|
| **二进制大小** | 每个测试可执行文件包含 gtest 代码，可能增加总大小 |
| **编译时间** | 每个测试目标需要链接 gtest，可能增加编译时间 |
| **运行时性能** | 静态链接避免了动态加载开销 |
| **部署简便性** | 无需处理动态库依赖，便于部署 |

### C++17 影响
- **现代 C++ 特性**：可使用结构化绑定、if constexpr 等特性
- **代码简洁性**：提高测试代码的可读性
- **编译兼容性**：确保测试代码与生产代码的一致性

---

## 维护建议

### 1. 升级上游版本时的注意事项

#### 必须保留的配置
- ✅ 所有 hwext 源文件的引用
- ✅ 7 个静态库目标的定义
- ✅ C++17 标准的强制使用
- ✅ 警告抑制配置

#### 需要验证的兼容性
- ⚠️ 上游新增的源文件是否需要添加到 `sources_files`
- ⚠️ 上游 CMake 选项是否需要在 GN 中映射
- ⚠️ hwext 扩展是否需要调整以适配上游变化

#### 可能需要调整的代码
- 🔧 警告抑制选项（上游修复后可移除）
- 🔧 RTTI 配置（上游 RTTI 支持变化时）
- 🔧 MinGW 特定处理（MinGW 支持变化时）

### 2. 最佳实践

#### 选择合适的目标
| 需求 | 推荐目标 |
|------|----------|
| 标准单元测试 | `:gtest` + 自定义 main |
| 快速原型 | `:gtest_main` |
| 需要 RTTI | `:gtest_rtti` |
| 需要 RTTI + 快速原型 | `:gtest_rtti_main` |
| 需要模拟对象 | `:gmock` |
| 需要模拟对象 + 快速原型 | `:gmock_main` |

#### 避免反模式
- ❌ 在生产代码中依赖 gtest（所有目标都是 `testonly = true`）
- ❌ 同时链接 `:gtest` 和 `:gtest_main`（会导致 main 函数冲突）
- ❌ 滥用 RTTI（仅在必要时启用）

### 3. 性能优化建议

#### 减少编译时间
- 使用预编译头（如果 OH 构建系统支持）
- 考虑将 gtest 编译为静态库并缓存

#### 减少运行时开销
- 禁用 RTTI（除非必要）
- 使用 `:gtest` 而非 `:gtest_rtti`
- 避免过度使用 gmock（Mock 对象有一定开销）

---

## 相关文档

- [01_Overview.md](./01_Overview.md) - GoogleTest 原始库简介
- [02_Patches.md](./02_Patches.md) - hwext 扩展详细分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用方式
