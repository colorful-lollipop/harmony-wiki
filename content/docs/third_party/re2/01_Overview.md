# 原始库简介

## 1.1 基本信息

| 属性 | 内容 |
|------|------|
| **原始库名称** | RE2 (Google RE2 Regular Expressions) |
| **版本** | 2024-07-02 |
| **许可证** | BSD 3-Clause |
| **上游仓库** | https://github.com/google/re2 |
| **编程语言** | C++ |
| **依赖** | Abseil C++ |

## 1.2 功能概述

RE2 是 Google 开发的正则表达式库，设计目标是提供**快速、安全、线程友好**的正则表达式匹配功能。

### 核心设计原则

1. **线性时间复杂度保证**
   - 匹配时间与输入长度成线性关系：O(n)
   - 避免回溯式正则引擎的灾难性回溯问题
   - 适用于处理不受信任的输入

2. **线程安全**
   - RE2 对象可安全地在多线程间共享
   - 无需额外的同步机制

3. **C++ 现代 API**
   - 提供直观的 C++ 接口
   - 支持 StringPiece 等高效字符串视图
   - 兼容标准库风格

### 与 PCRE 的差异

| 特性 | RE2 | PCRE |
|------|-----|------|
| **时间复杂度** | O(n) 保证 | 可能指数级回溯 |
| **线程安全** | 原生支持 | 需额外配置 |
| **递归** | 无递归，无栈溢出风险 | 使用递归 |
| **功能集** | 标准正则功能 | 更丰富的扩展功能 |
| **回溯引用** | 有限支持 | 完整支持 |
| **前瞻断言** | 有限支持 | 完整支持 |

**RE2 不支持的特性**:
- 反向引用（Backreferences）如 `\1`
- 复杂的先行/后行断言
- 递归模式
- 条件子模式

## 1.3 主要组件

```
re2/
├── re2.h              # 主 API 头文件
├── re2.cc             # 主实现
├── regexp.h/cc        # 正则表达式内部表示
├── compile.cc         # 编译器：将正则编译为程序
├── prog.h/cc          # 正则程序表示
├── dfa.cc             # DFA 引擎实现
├── nfa.cc             # NFA 引擎实现
├── bitstate.cc        # 位状态回溯引擎
├── onepass.cc         # 单次遍历引擎
├── parse.cc           # 正则表达式解析器
├── filtered_re2.cc    # 多模式批量匹配
├── set.cc             # 正则集合
└── ...
```

### 关键类

| 类 | 说明 |
|----|------|
| `RE2` | 主正则表达式类，类似 `std::regex` |
| `StringPiece` | 轻量级字符串视图（类似 `std::string_view`） |
| `Options` | 正则选项（大小写敏感、编码等） |
| `FilteredRE2` | 高效的多模式匹配 |
| `Set` | 正则集合，用于批量匹配 |

## 1.4 在 OpenHarmony 中的作用和定位

### 在 OH 中的角色

RE2 在 OpenHarmony 中是**基础设施级**的正则表达式库，为上层模块提供安全、高效的正则匹配能力。

### 主要使用场景

| 使用模块 | 使用场景 | 重要性 |
|----------|----------|--------|
| **gRPC** | URI 模板匹配、路由解析 | 核心功能 |
| **Protobuf** | Profile 分析工具 | 工具支持 |
| **Abseil** | 依赖统计 | 基础依赖 |

### 选型理由

OpenHarmony 选择 RE2 而非 PCRE 的主要原因：

1. **安全性**: 线性时间保证，避免 ReDoS（正则表达式拒绝服务攻击）
2. **稳定性**: Google 维护，质量有保障
3. **性能**: 在大多数场景下性能优异
4. **兼容性**: 已被 gRPC 等核心组件使用，生态兼容性好

### 架构位置

```
应用层
   │
中间件层 (gRPC, Protobuf)
   │
基础库层 (RE2, Abseil)
   │
系统层
```

## 1.5 API 示例

### 基本匹配

```cpp
#include "re2/re2.h"

// 简单匹配
bool matched = RE2::FullMatch("hello", "h.*o");

// 提取捕获组
std::string str;
if (RE2::Extract("foo bar baz", "foo (\\w+) (\\w+)", "\\2", &str)) {
    // str == "baz"
}

// 查找所有匹配
std::string s = "foo=123;bar=456";
RE2::GlobalReplace(&s, "(\\w+)=(\\d+)", "\\1: \\2");
// s == "foo: 123;bar: 456"
```

### 使用 Options

```cpp
RE2::Options options;
options.set_case_sensitive(false);  // 忽略大小写
options.set_log_errors(false);      // 不输出错误日志
options.set_encoding(RE2::Options::EncodingUTF8);  // UTF-8 编码

RE2 re("pattern", options);
```

## 1.6 相关资源

- **官方文档**: https://github.com/google/re2/wiki
- **API 参考**: `re2/re2.h` 头文件
- **Issue 跟踪**: https://github.com/google/re2/issues
- **邮件列表**: re2-dev@googlegroups.com

---

**上一篇**: [SUMMARY.md](./SUMMARY.md) | **下一篇**: [02_Patches.md](./02_Patches.md)
