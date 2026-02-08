# GoogleTest 文档导航

> OpenHarmony third_party/googletest

---

## 文档概述

本文档库旨在说明 **GoogleTest 在 OpenHarmony 中的集成与适配**，重点介绍 OH 对该库的 Patch（hwext 扩展）、特殊适配以及被系统使用的方式。

**适用读者**：
- OpenHarmony 系统开发者
- 测试工程师
- 测试框架维护者
- 第三方库集成开发者

---

## 快速导航

### 核心文档

| 文档 | 内容 | 推荐读者 |
|------|------|----------|
| [01_Overview.md](./01_Overview.md) | GoogleTest 原始库简介、版本信息、OH 定位 | 所有人 |
| [02_Patches.md](./02_Patches.md) | hwext 扩展详细分析、Patch 清单、升级建议 | 测试框架维护者 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建适配、目标定义、编译选项 | 构建系统开发者 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用方式、最佳实践 | 测试工程师 |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估结果、关键发现 |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录（TODO） |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度（TODO） |

---

## 阅读路线建议

### 路线 1：快速了解（5 分钟）
1. [01_Overview.md](./01_Overview.md) - 了解 GoogleTest 原始库和 OH 定位
2. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 查看关键发现和总结

**适合**：想快速了解 OH 对 GoogleTest 的修改和使用的读者。

---

### 路线 2：测试工程师（15 分钟）
1. [01_Overview.md](./01_Overview.md) - 了解 GoogleTest 基础功能
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 学习 OH 中的使用方式和最佳实践
3. [02_Patches.md](./02_Patches.md) - 了解 hwext 扩展（测试标记、过滤等）

**适合**：需要在 OH 中编写或维护测试的工程师。

---

### 路线 3：测试框架维护者（30 分钟）
1. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整的项目评估
2. [02_Patches.md](./02_Patches.md) - 深入理解 hwext 扩展的实现和维护
3. [03_Build_Integration.md](./03_Build_Integration.md) - 掌握构建系统适配细节
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解依赖关系和影响范围

**适合**：需要升级上游版本、维护 hwext 扩展的工程师。

---

### 路线 4：构建系统开发者（20 分钟）
1. [01_Overview.md](./01_Overview.md) - 了解库的基本信息
2. [03_Build_Integration.md](./03_Build_Integration.md) - 掌握 BUILD.gn 配置和构建选项
3. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 了解与上游 CMake 的差异

**适合**：需要集成 GoogleTest 到 OH 构建系统的开发者。

---

## 关键概念

### OH 特有术语

| 术语 | 解释 | 相关文档 |
|------|------|----------|
| **hwext** | Huawei Extension，华为对 GoogleTest 的扩展集合 | [02_Patches.md](./02_Patches.md) |
| **TEST_FLAGS** | 测试标记，用于测试分类和过滤 | [02_Patches.md](./02_Patches.md) |
| **HWTEST** | OH 扩展的测试宏，支持 test_flags | [02_Patches.md](./02_Patches.md) |
| **测试级别** | Level0-4，按重要性分级的测试 | [02_Patches.md](./02_Patches.md) |
| **测试类型** | Function, Performance, Power 等 | [02_Patches.md](./02_Patches.md) |
| **测试大小** | SmallTest, MediumTest, LargeTest | [02_Patches.md](./02_Patches.md) |

### 构建系统术语

| 术语 | 解释 | 相关文档 |
|------|------|----------|
| **GN** | Generate Ninja，OH 使用的构建系统 | [03_Build_Integration.md](./03_Build_Integration.md) |
| **inner_kits** | bundle.json 中定义的对外接口 | [01_Overview.md](./01_Overview.md) |
| **static_library** | 静态库，OH 中 GoogleTest 的唯一构建类型 | [03_Build_Integration.md](./03_Build_Integration.md) |
| **testonly** | 标记目标仅用于测试 | [03_Build_Integration.md](./03_Build_Integration.md) |

---

## 核心发现

### 1. 扩展策略
- **无传统 Patch 文件**：采用源代码扩展方式
- **hwext 扩展体系**：5 个头文件 + 5 个实现文件
- **代码隔离清晰**：hwext 与上游代码分离

### 2. OH 特有价值
1. **测试过滤系统**：按级别、类型、大小、排名多维度过滤
2. **测试标记系统**：HWTEST 宏支持 test_flags
3. **多线程测试**：支持并发测试执行（待确认）
4. **测试分类**：Function, Performance, Power, Security 等 11 种类型

### 3. 使用模式
- **99.5% 用于测试代码**：389 个 BUILD.gn 引用中仅 2 个非测试
- **驱动层为主**：HDF 和 peripheral 驱动是主要使用者
- **静态链接**：所有目标为 `static_library`

### 4. 构建适配
- **7 个构建目标**：gtest/gtest_rtti/gtest_main/gtest_rtti_main + gmock/gmock_rtti/gmock_main
- **C++17 标准**：显式使用 C++17（上游要求 C++14+）
- **GN 构建**：替代上游 CMake，适配 OH 构建系统

---

## 常见问题

### Q1：OH 为什么不使用传统的 patch 文件？
**A**：OH 采用源代码扩展方式，新增 `hwext/` 目录与上游代码隔离。这种方式比 patch 更易维护，升级上游版本时冲突更少。

详见：[02_Patches.md](./02_Patches.md)

---

### Q2：hwext 扩展是否会向上游贡献？
**A**：
- **测试过滤系统**：部分功能（如标签系统）可考虑贡献，但需简化
- **多线程测试**：上游有 `gtest-parallel` 工具，但未集成，集成可能性较高
- **扩展测试宏**：OH 特有需求，不适合贡献

详见：[02_Patches.md](./02_Patches.md) - 维护建议

---

### Q3：如何在 OH 中使用 GoogleTest？
**A**：
```gn
# BUILD.gn
ohos_unittest("my_test") {
  sources = [ "my_test.cc" ]
  deps = [
    "//third_party/googletest:gtest_main",  # 使用 gtest_main
  ]
}
```

详见：[04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用方式

---

### Q4：如何使用 hwext 的测试过滤功能？
**A**：
```cpp
HWTEST(MyTest, MyFunction,
       testing::ext::TestType::Function |
       testing::ext::TestSize::Small) {
  // 测试代码
}
```

运行时：
```bash
./test --type=Function --size=SmallTest
```

详见：[02_Patches.md](./02_Patches.md) - 测试过滤系统

---

### Q5：升级上游 GoogleTest 版本需要注意什么？
**A**：
1. 必须保留 `hwext/` 目录
2. 验证 `TestDefManager::findDefFor()` 与上游 `TestInfo` 的兼容性
3. 检查测试过滤逻辑是否需要调整
4. 验证多线程实现（如果有）与上游测试框架的交互

详见：[02_Patches.md](./02_Patches.md) - 升级建议

---

## 相关资源

### 外部资源
- **GoogleTest 官方文档**: https://google.github.io/googletest/
- **GoogleTest GitHub**: https://github.com/google/googletest
- **OpenHarmony 文档**: https://docs.openharmony.cn/

### OH 内部资源
- **测试框架文档**: [01_Overview.md](./01_Overview.md)
- **构建系统文档**: [03_Build_Integration.md](./03_Build_Integration.md)
- **使用最佳实践**: [04_Usage_in_OH.md](./04_Usage_in_OH.md)

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0 | 2026-02-08 | 初始版本，完成核心文档 |

---

## 贡献与反馈

如果您发现文档错误或有改进建议，请：
1. 检查 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 中的 TODO 列表
2. 提交 Issue 或 Pull Request 到 OpenHarmony 代码仓库

---

## 许可证

本文档遵循与 GoogleTest 相同的 BSD 3-Clause License。
