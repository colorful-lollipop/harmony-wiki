# 构建系统

## 5.1 概述

Developer Test Framework 使用 **GN (Generate Ninja)** 作为构建系统，与 OpenHarmony 整体构建体系保持一致。

### 构建入口

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根构建入口 |
| `bundle.json` | 组件配置 |

## 5.2 GN 构建文件

### 5.2.1 根 BUILD.gn

```gn
group("make_temp_test") {
  testonly = true
  deps = []
}
```

**证据**: `BUILD.gn:14-17`

此文件为空占位，实际编译由 `bundle.json` 中的 `sub_component` 和 `test` 配置驱动。

### 5.2.2 示例构建 (examples/calculator/BUILD.gn)

```gn
import("//build/ohos.gni")

config("calculator_config") {
  visibility = [ ":*" ]
  include_dirs = [ "include" ]
}

ohos_shared_library("calculator") {
  sources = [
    "include/calculator.h",
    "src/calculator.cpp",
  ]
  public_configs = [ ":calculator_config" ]
  subsystem_name = "testfwk"
}

ohos_static_library("calculator_static") {
  sources = [
    "include/calculator.h",
    "src/calculator.cpp",
  ]
  public_configs = [ ":calculator_config" ]
}
```

**证据**: `examples/calculator/BUILD.gn:14-38`

### 5.2.3 静态库构建 (aw/cxx/distributed/BUILD.gn)

```gn
import("//build/ohos.gni")

config("distributedtest_config") {
  include_dirs = [
    ".",
    "utils/",
  ]
}

ohos_static_library("distributedtest_lib") {
  testonly = true
  sources = [
    "distributed_agent.cpp",
    "distributed_cfg.cpp",
    "distributed_major.cpp",
  ]
  external_deps = [
    "c_utils:utils",
    "googletest:gtest",
    "hilog:libhilog",
  ]
  public_configs = [ ":distributedtest_config" ]
}
```

**证据**: `aw/cxx/distributed/BUILD.gn:16-44`

## 5.3 组件配置 (bundle.json)

### 5.3.1 基本信息

```json
{
  "name": "@openharmony/developer_test",
  "version": "3.1.0",
  "component": {
    "name": "developer_test",
    "subsystem": "testfwk",
    "adapted_system_type": ["mini", "small", "standard"]
  }
}
```

**证据**: `bundle.json:1-18`

### 5.3.2 子组件 (sub_component)

| 目标 | 路径 |
|------|------|
| `app_info` | `//test/testfwk/developer_test/examples/app_info:app_info` |
| `detector` | `//test/testfwk/developer_test/examples/detector:detector` |
| `calculator` | `//test/testfwk/developer_test/examples/calculator:calculator` |
| `calculator_static` | `//test/testfwk/developer_test/examples/calculator:calculator_static` |

**证据**: `bundle.json:23-28`

### 5.3.3 内部 Kit (inner_kits)

#### distributedtest_lib

| 属性 | 值 |
|------|-----|
| Header Base | `//test/testfwk/developer_test/aw/cxx/distributed/utils`<br>`//test/testfwk/developer_test/aw/cxx/distributed` |
| Header Files | `csv_transform_xml.h`<br>`distributed.h`<br>`distributed_agent.h`<br>`distributed_cfg.h`<br>`distributed_major.h` |

**证据**: `bundle.json:31-44`

#### performance_test_static

| 属性 | 值 |
|------|-----|
| Header Base | `//test/testfwk/developer_test/aw/cxx/hwext` |
| Header Files | `perf.h` |

**证据**: `bundle.json:46-52`

### 5.3.4 测试配置 (test)

| 测试目标 | 路径 | 类型 |
|---------|------|------|
| `app_info` | `//test/testfwk/developer_test/examples/app_info/test:unittest` | unittest |
| `calculator` | `//test/testfwk/developer_test/examples/calculator/test:unittest` | unittest |
| `calculator` | `//test/testfwk/developer_test/examples/calculator/test:fuzztest` | fuzztest |
| `calculator` | `//test/testfwk/developer_test/examples/calculator/test:benchmarktest` | benchmarktest |
| `detector` | `//test/testfwk/developer_test/examples/detector/test:unittest` | unittest |
| `sleep` | `//test/testfwk/developer_test/examples/sleep/test:performance` | performance |
| `distributedb` | `//test/testfwk/developer_test/examples/distributedb/test:distributedtest` | distributedtest |
| `stagetest` | `//test/testfwk/developer_test/examples/stagetest/actsbundlemanagerstagetest:unittest` | unittest |

**证据**: `bundle.json:54-63`

## 5.4 构建目标类型

### 5.4.1 目标类型说明

| 类型 | GN 模板 | 输出 | 用途 |
|------|---------|------|------|
| `ohos_shared_library` | ohos.gni | `.so` | 动态库 |
| `ohos_static_library` | ohos.gni | `.a` | 静态库 |
| `ohos_unittest` | test.gni | 可执行文件 | 单元测试 |
| `ohos_js_unittest` | test.gni | `.hap` | JS 测试 |
| `ohos_fuzztest` | test.gni | 可执行文件 | Fuzz 测试 |
| `ohos_performancetest` | test.gni | 可执行文件 | 性能测试 |

### 5.4.2 目标依赖

```gn
# 内部依赖
deps = [":target_name"]

# 外部依赖
external_deps = [
  "c_utils:utils",
  "googletest:gtest",
  "hilog:libhilog",
]

# 公共配置
public_configs = [":config_name"]
```

## 5.5 构建流程

### 5.5.1 Python 构建入口

```python
# src/core/build/build_manager.py:83-92
@classmethod
def _compile_test_cases_by_target(cls, project_root_path, product_form,
                                  build_target):
    if BuildTestcases(project_root_path).build_testcases(product_form,
                                                         build_target):
        LOG.info("Test case compilation successed.")
        build_result = True
    else:
        LOG.info("Test case compilation failed, please modify.")
        build_result = False
    return build_result
```

**证据**: `src/core/build/build_manager.py:83-92`

### 5.5.2 构建命令

```bash
# 编译全部测试用例
./build.sh --product-name {product_name} --build-target make_test

# 编译特定用例
./build.sh --product-name {product_name} --build-target {target_name}
```

## 5.6 编译产物

### 5.6.1 产物类型

| 产物类型 | 格式 | 说明 |
|---------|------|------|
| 动态库 | `.so` | 共享库 |
| 静态库 | `.a` | 静态库 |
| 可执行文件 | 无扩展名 | ELF 格式 |
| HAP 包 | `.hap` | JS/ArkTS 测试包 |

### 5.6.2 产物路径

```
out/{product}/tests/
├── {subsystem}/
│   ├── {part}/
│   │   ├── unittest/
│   │   │   └── {test_suite}
│   │   ├── fuzztest/
│   │   │   └── {fuzzer}
│   │   └── performance/
│   │       └── {perf_test}
│   └── ...
└── ...
```

## 5.7 测试构建配置

### 5.7.1 C++ 单元测试 (examples/calculator/test/unittest/common/BUILD.gn)

```gn
import("//build/test.gni")

module_output_path = "developer_test/calculator"

config("module_private_config") {
  visibility = [ ":*" ]
  include_dirs = [ "../../../include" ]
}

ohos_unittest("CalculatorSubTest") {
  module_out_path = module_output_path
  sources = [
    "../../../include/calculator.h",
    "../../../src/calculator.cpp",
  ]
  sources += [ "calculator_sub_test.cpp" ]
  configs = [ ":module_private_config" ]
  deps = [ "//third_party/googletest:gtest_main" ]
}

group("unittest") {
  testonly = true
  deps = [":CalculatorSubTest"]
}
```

### 5.7.2 JS 测试 (来自 `src/core/constants.py:81-122`)

```gn
ohos_js_unittest("%(suite_name)s") {
  module_out_path = module_output_path
  hap_profile = "./src/main/config.json"
  deps = [
    ":%(suite_name)s_js_assets",
    ":%(suite_name)s_resources",
  ]
  certificate_profile = "//test/testfwk/developer_test/signature/openharmony_sx.p7b"
  hap_name = "%(suite_name)s"
}
```

## 5.8 常见构建问题

| 问题 | 原因 | 解决方法 |
|------|------|---------|
| GN 找不到 | 环境变量未配置 | source build.sh |
| 依赖缺失 | external_deps 配置错误 | 检查依赖声明 |
| 编译超时 | 用例过多 | 分批编译 |
| 产物路径错误 | module_out_path 错误 | 检查路径配置 |

## 5.9 相关文档

- [02_Architecture.md](02_Architecture.md) - 系统架构
- [04_Configuration.md](04_Configuration.md) - 配置说明
- [06_Usage_Guide.md](06_Usage_Guide.md) - 使用指南
- [07_Examples.md](07_Examples.md) - 示例说明
