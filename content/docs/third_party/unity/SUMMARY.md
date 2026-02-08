# 阅读指南

## 文档概述

本文档旨在为 OpenHarmony 开发者提供 Unity 测试框架的全面参考指南。Unity 是专为 C 语言设计的轻量级单元测试框架，在 OpenHarmony 生态系统中承担 XTS 测试体系的执行引擎角色。

## 阅读路线

### 路线一：快速了解（5 分钟）

如果你是第一次接触 Unity 框架，建议按照以下顺序快速浏览：

1. **[README.md](./README.md)** - 1 分钟
   - 了解 Unity 在 OpenHarmony 中的定位
   - 查看快速开始示例
   - 浏览文档导航

2. **[01_Overview.md](./01_Overview.md)** - 4 分钟
   - 理解 Unity 的核心设计理念
   - 了解框架的核心特性
   - 明确 Unity 在 OH 系统中的作用

### 路线二：深入集成（15 分钟）

如果你需要将 Unity 集成到你的项目中，建议按照以下顺序深入学习：

1. **[README.md](./README.md)** - 基础概览
2. **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建适配
   - CMake 集成详解
   - Meson 集成详解
   - GN 构建配置（参考）
3. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 使用方法
   - 依赖关系分析
   - 典型使用场景
   - 最佳实践

### 路线三：维护升级（10 分钟）

如果你负责 Unity 库的维护和升级工作，建议重点关注：

1. **[02_Patches.md](./02_Patches.md)** - Patch 分析
   - 无 Patch 的原因分析
   - 上游同步策略
   - 版本升级注意事项

### 路线四：完整阅读（25 分钟）

按照文档顺序完整阅读所有内容：

| 文档 | 预计时间 | 主要内容 |
|------|----------|----------|
| README.md | 5 分钟 | 概览、导航、快速开始 |
| 01_Overview.md | 5 分钟 | 原始库介绍、OH 定位 |
| 02_Patches.md | 5 分钟 | Patch 分析、维护建议 |
| 03_Build_Integration.md | 5 分钟 | 构建配置详解 |
| 04_Usage_in_OH.md | 5 分钟 | 依赖关系、使用场景 |

## 关键信息速查

### 核心要点

| 主题 | 关键信息 |
|------|----------|
| **库类型** | C 语言单元测试框架 |
| **上游版本** | v2.6.0 |
| **OH 打包版本** | 3.1 |
| **许可证** | MIT |
| **Patch 数量** | 0 |
| **主要使用者** | XTS / HCTest |
| **适配系统** | mini、small |

### 快速引用

**CMake 集成**：

```cmake
add_subdirectory(third_party/unity)
target_link_libraries(my_test PRIVATE unity)
```

**断言示例**：

```c
#include "unity.h"

void test_Example(void) {
  int result = my_function(5);
  TEST_ASSERT_EQUAL_INT(10, result);
}
```

**构建引用**：

```gn
sources = [
  "//third_party/unity/src/unity.c",
]

include_dirs = [
  "//third_party/unity/src",
]
```

## 常见问题

### Q1：Unity 和其他 C 测试框架（如 CMock、CMockery）有什么区别？

Unity 是轻量级的测试框架，专注于断言和执行。CMock 是 Unity 的配套工具，用于生成 Mock 函数。CMockery 是另一个轻量级框架，但维护不如 Unity 活跃。Unity 在嵌入式领域更受欢迎，社区更活跃。

### Q2：为什么 Unity 没有 BUILD.gn 文件？

Unity 最初为 CMake 和 Meson 设计，未提供 GN 构建配置。当前 OH 项目通过 CMake 方式引用 Unity，或者需要开发者自行添加 BUILD.gn 配置。本 Wiki 提供了参考配置。

### Q3：如何在 OpenHarmony 中运行 Unity 测试？

在 OH 中运行 Unity 测试需要：
1. 编译包含 Unity 的测试模块
2. 将测试程序部署到目标设备
3. 执行测试程序并查看输出

### Q4：Unity 支持测试覆盖率统计吗？

Unity 框架本身不直接支持覆盖率统计。通常配合 gcov（GCC）或 lcov 工具进行覆盖率分析。

## 相关资源

### 内部资源

| 资源 | 路径 |
|------|------|
| Unity 源码 | `//third_party/unity/src/` |
| XTS 测试体系 | `//test/xts/` |
| HCTest 框架 | `//test/xts/tools/lite/hctest/` |
| OH 测试文档 | `//docs/test/` |

### 外部资源

| 资源 | 地址 |
|------|------|
| Unity 官方 GitHub | https://github.com/ThrowTheSwitch/Unity |
| Unity 官方文档 | https://throwtheswitch.org/Unity-Framework |
| Unity 入门指南 | `docs/UnityGettingStartedGuide.md` |
| Unity 断言参考 | `docs/UnityAssertionsReference.md` |

## 文档更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| v1.0 | 2026-02-08 | 初始版本，完成全部文档编写 |

## 反馈与贡献

如果您发现文档中的错误或有改进建议，请通过以下方式反馈：

- 在 Gitee 提交 Issue
- 联系维护者：chenliangxing@huawei.com
