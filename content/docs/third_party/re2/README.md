# RE2 Wiki

OpenHarmony 第三方库 RE2 的适配与使用文档。

## 库概览

| 属性 | 值 |
|------|-----|
| **原始库名称** | RE2 (Google Regular Expressions) |
| **上游版本** | 2024-07-02 |
| **OH 组件版本** | 3.1 |
| **许可证** | BSD 3-Clause |
| **上游地址** | https://github.com/google/re2 |
| **OH 子系统** | thirdparty |

## RE2 是什么

RE2 是 Google 开发的快速、安全、线程友好的正则表达式库，作为 PCRE、Perl 和 Python 中回溯式正则引擎的替代方案。

**核心特性**:
- **线性时间复杂度**: 匹配时间与输入长度成线性关系，避免灾难性回溯
- **线程安全**: 支持并发使用
- **C++ 原生**: 提供现代 C++ API
- **无递归**: 避免栈溢出风险

## OpenHarmony 适配概述

### 适配状态

| 维度 | 状态 |
|------|------|
| **功能性 Patch** | 无（使用纯上游代码） |
| **构建适配** | BUILD.gn 配置 |
| **OH 特有修改** | 无 |
| **升级难度** | 低 |

### 关键特性

1. **零 Patch 策略**: RE2 在 OH 中未应用任何功能性 Patch，完全依赖上游代码
2. **构建系统适配**: 仅添加 BUILD.gn 以适配 OH 的 GN 构建系统
3. **符号版本控制**: 使用 libre2.map 管理共享库符号导出
4. **Abseil 依赖**: 依赖 abseil-cpp 提供基础功能支持

### 在 OH 中的作用

RE2 主要被以下核心模块使用：
- **gRPC**: URI 路由匹配、模板解析
- **Protobuf**: Profile 分析工具中的正则匹配

## 文档导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（重要：RE2 无功能性 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建适配详解 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | OH 中的依赖关系与使用场景 |

## 快速参考

### 构建目标

```gn
//third_party/re2:re2
```

### 依赖声明

```gn
external_deps = [
    "abseil-cpp:absl_base",
    "abseil-cpp:absl_container",
    "abseil-cpp:absl_cord",
    "abseil-cpp:absl_hash",
    "abseil-cpp:absl_log",
    "abseil-cpp:absl_raw_logging_internal",
    "abseil-cpp:absl_spinlock_wait",
    "abseil-cpp:absl_str_format_internal",
    "abseil-cpp:absl_strings",
]
```

### 头文件引用

```cpp
#include "re2/re2.h"
```

## 维护建议

1. **版本升级**: 可直接跟随上游升级，无 Patch 冲突风险
2. **ABI 兼容性**: 注意维护 libre2.map 中的符号版本
3. **依赖更新**: 升级时注意与 abseil-cpp 版本匹配
4. **测试覆盖**: 升级后需验证 gRPC 等依赖模块的功能

---

**文档版本**: 1.0  
**最后更新**: 2025-02-08
