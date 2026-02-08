# GoogleTest 原始库简介

> 版本: 1.16.0 | 许可证: BSD 3-Clause License

---

## 库概述

**GoogleTest** 是 Google 开发的 C++ 测试框架，广泛用于单元测试和集成测试。它提供了丰富的断言宏、测试发现机制、参数化测试等功能，是 C++ 社区中最流行的测试框架之一。

**在 OpenHarmony 中的作用**：作为 OH 项目的标准测试框架，为各模块（特别是驱动层）提供统一的单元测试能力。OH 对 GoogleTest 进行了扩展（hwext），增加了测试过滤、标记、多线程测试等功能，以满足大规模测试的管理和执行需求。

---

## 原始库基本信息

| 项目 | 值 |
|------|-----|
| **库名称** | GoogleTest (googletest) |
| **版本** | 1.16.0 |
| **许可证** | BSD 3-Clause License |
| **上游地址** | https://github.com/google/googletest/releases/tag/v1.16.0 |
| **C++ 标准** | C++14+ (OH 配置为 C++17) |

---

## 核心功能

### 1. xUnit 测试框架
基于 xUnit 架构设计，支持标准的测试用例结构：
- `TEST(TestCase, TestName)` - 基础测试宏
- `TEST_F(TestCase, TestName)` - 夹具测试宏
- `TYPED_TEST` - 类型参数化测试
- `TEST_P` - 值参数化测试

### 2. 断言系统
提供丰富的断言宏，分为 fatal 和 non-fatal 两类：
- `ASSERT_EQ`, `ASSERT_NE`, `ASSERT_TRUE`, `ASSERT_FALSE` - Fatal 断言（失败则终止）
- `EXPECT_EQ`, `EXPECT_NE`, `EXPECT_TRUE`, `EXPECT_FALSE` - Non-fatal 断言（继续执行）
- 支持字符串、浮点数、容器等多种类型的比较

### 3. 测试发现
自动发现和运行测试，无需手动注册测试用例。

### 4. 参数化测试
- **值参数化测试** (`TEST_P`)：使用不同输入值运行同一测试
- **类型参数化测试** (`TYPED_TEST`)：用不同数据类型运行同一测试

### 5. GoogleMock（gmock）
提供强大的模拟对象框架，用于测试代码间的交互：
- `MOCK_METHOD` - 模拟方法
- `EXPECT_CALL` - 设置期望调用
- `WithArgs`, `Times` 等匹配器

### 6. 死亡测试（Death Tests）
验证代码以特定方式退出，用于测试错误处理代码。

### 7. 多种测试运行选项
- 运行单个测试
- 按指定顺序运行测试
- 并行运行测试
- 重复运行测试

---

## 在 OpenHarmony 中的定位

### 主要使用者
OpenHarmony 中 **99.5%** 的使用场景为测试代码，主要集中在：
1. **驱动层测试**：HDF（硬件驱动框架）和 peripheral 驱动
2. **系统组件测试**：内核、子系统单元测试
3. **接口测试**：HDI（硬件驱动接口）测试

### OH 特点
- **静态链接**：所有目标为 `static_library`
- **仅测试用途**：所有目标标记 `testonly = true`
- **hwext 扩展**：华为特有的测试过滤、标记系统
- **C++17 标准**：比上游要求的 C++14 更严格

### 使用统计
| 统计项 | 数值 |
|--------|------|
| 总引用文件数 | 391 个 BUILD.gn |
| 测试代码引用 | 389 个 (99.5%) |
| 非测试代码引用 | 2 个 (0.5%，skia 构建辅助) |

---

## 相关资源

### 官方文档
- **GoogleTest 主页**: https://google.github.io/googletest/
- **入门指南**: https://google.github.io/googletest/primer.html
- **高级指南**: https://google.github.io/googletest/advanced.md
- **API 参考**: https://google.github.io/googletest/reference/assertions.html

### 上游代码
- **GitHub 仓库**: https://github.com/google/googletest
- **发布版本**: https://github.com/google/googletest/releases

### 在 OpenHarmony 中的文档
- **hwext 扩展文档**: `02_Patches.md`
- **构建集成**: `03_Build_Integration.md`
- **使用方式**: `04_Usage_in_OH.md`

---

## 下一步阅读

- [02_Patches.md](./02_Patches.md) - OH hwext 扩展详细分析
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用方式
