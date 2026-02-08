# Unity 测试框架

## 库概览

Unity 是专为 C 语言设计的轻量级单元测试框架，由 ThrowTheSwitch 社区维护并开源。该框架以简洁高效著称，核心实现仅包含一个 C 源文件和两个头文件，不依赖任何外部库或操作系统 API，非常适合嵌入式系统和资源受限环境的测试需求。

Unity 在 OpenHarmony 系统中的定位是**轻量级测试基础设施**，主要用于 XTS（Xtended Test Suite）兼容性测试套件的测试用例执行。框架的断言体系完善，支持整数、浮点数、字符串、数组、位操作等多种数据类型的测试验证，为 OpenHarmony 的内核模块、系统服务以及应用框架提供可靠的单元测试支持。

## OpenHarmony 适配概述

Unity 库在 OpenHarmony 中的集成采用了**最小化适配**策略。与其他需要大量 Patch 的第三方库不同，Unity 由于其嵌入式友好的设计理念，未被应用任何代码层面的修改。框架完全基于宏定义实现跨平台兼容，不调用任何操作系统服务，因此天然支持 OpenHarmony 的 mini 和 small 系统配置。

在构建系统层面，Unity 通过 CMake 和 Meson 原生支持进行集成，OpenHarmony 通过 `bundle.json` 组件声明完成适配。主要依赖者包括 XTS 工具链和 HCTest 测试框架，这些模块直接引用 Unity 源码或头文件来构建测试用例执行环境。

## 文档导航

| 文档 | 内容说明 | 适合人群 |
|------|----------|----------|
| [01_Overview.md](./01_Overview.md) | 原始库功能简介、OH 定位、核心特性 | 所有使用者 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch）、适配说明 | 维护者、升级负责人 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建配置详解、编译选项说明 | 开发者、构建工程师 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系图、使用场景、集成方式 | 集成工程师、测试工程师 |
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议、快捷导航 | 新使用者 |

## 快速开始

### 基本使用

在 OpenHarmony 项目中使用 Unity 进行单元测试，需要在 `BUILD.gn` 文件中添加以下依赖配置：

```gn
unity_test_sources = [
  "//third_party/unity/src/unity.c",
  "your_test_file.c",
]

ohos_unittest("your_test") {
  ...
  sources = unity_test_sources
  include_dirs = [
    "//third_party/unity/src",
  ]
}
```

### 断言示例

Unity 提供了丰富的断言宏，以下是常用断言的示例代码：

```c
#include "unity.h"

void setUp(void) {
  // 测试前置设置
}

void tearDown(void) {
  // 测试后置清理
}

void test_ExampleFunction(void) {
  int result = your_function(5);
  TEST_ASSERT_EQUAL_INT(10, result);
  
  float value = 3.14f;
  TEST_ASSERT_FLOAT_WITHIN(0.01f, 3.14f, value);
  
  const char* str = "hello";
  TEST_ASSERT_EQUAL_STRING("hello", str);
}
```

## 版本信息

| 项目 | 版本 |
|------|------|
| 上游版本 | v2.6.0 |
| OpenHarmony 打包版本 | 3.1 |
| 许可证 | MIT License |
| 上游地址 | https://github.com/ThrowTheSwitch/Unity/tree/v2.6.0 |

## 相关资源

- [Unity 官方文档](docs/)
- [Unity 官方入门指南](docs/UnityGettingStartedGuide.md)
- [Unity 断言参考](docs/UnityAssertionsReference.md)
- [OpenHarmony 测试体系](../docs/test/)
- [XTS 兼容性测试](../test/xts/)

## 维护者信息

| 项目 | 内容 |
|------|------|
| 维护者 | chenliangxing@huawei.com |
| 子系统 | thirdparty |
| 适配系统 | mini、small |
