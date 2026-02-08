# API/接口差异

> tex-hyphen 在 OpenHarmony 中的 API 差异分析

---

## 概述

**重要说明**: tex-hyphen **不提供任何面向应用的 API**。

它是一个**资源文件包**，提供断词模式数据。实际的 API 由 Skia 的 `Hyphenator` 类提供，封装了 tex-hyphen 的数据。

```
应用层
    ↓ [不直接使用 tex-hyphen]
Skia Hyphenator (API 封装层)
    ↓ [读取 .hpb 文件]
tex-hyphen (资源文件)
```

---

## API 差异说明

### 原始库 API

**tex-hyphen 原始库提供以下接口**:

#### Ruby API（上游工具）

```ruby
# lib/tex/hyphen/language.rb
module Hyphen
  class Language
    # 加载语言模式
    def self.load(name)

    # 断词
    def hyphenate(word)

    # 获取语言信息
    def info
  end
end
```

**使用方式**:
```ruby
require 'hyphen'

# 加载英语
language = Hyphen::Language.load('en-us')

# 断词
result = language.hyphenate('extraordinary')
# 输出: "ex-traor-di-nary"
```

**适用场景**:
- TeX 排版系统
- Ruby 应用
- 文本处理脚本

### OH API（Skia 封装）

**Skia 提供以下接口**:

#### Hyphenator 类

```cpp
// third_party/skia/m133/modules/skparagraph/include/Hyphenator.h

class Hyphenator {
public:
    // 单例模式
    static Hyphenator& getInstance();

    // 获取断词数据（内部使用）
    const std::vector<uint8_t>& getHyphenatorData(const std::string& locale);

    // 查找断词位置（主要 API）
    std::vector<uint8_t> findBreakPositions(const SkString& locale,
                                         const SkString& text,
                                         size_t startPos,
                                         size_t endPos);
};
```

**使用方式**:
```cpp
#include "include/Hyphenator.h"

// 获取单例
auto& hyphenator = skia::textlayout::Hyphenator::getInstance();

// 查找断词位置
SkString text = "extraordinary";
std::vector<uint8_t> breaks = hyphenator.findBreakPositions(
    "en-US",      // locale
    text,          // text
    0,             // startPos
    text.size()    // endPos
);

// 处理断词结果
for (size_t i = 0; i < breaks.size(); i++) {
    if (breaks[i] > 0) {
        // 可以在位置 i 处断词
    }
}
```

**适用场景**:
- Skia 文本排版
- Graphic 2D 文本渲染
- OpenHarmony 应用

---

## 与原始库的差异

### 1. API 层级差异

| 维度 | 原始库 | OH (Skia) |
|-----|---------|-----------|
| **API 语言** | Ruby | C++ |
| **API 层级** | 应用层 | 框架层 |
| **调用方式** | 直接调用 | 通过 Skia 封装 |
| **可见性** | 公开 API | 内部 API |

**对比**:
```
原始库:
  应用 → Ruby API → tex-hyphen 数据

OH:
  应用 → Skia Paragraph → Hyphenator → tex-hyphen 数据
```

### 2. 数据格式差异

| 维度 | 原始库 | OH |
|-----|---------|-----|
| **输入格式** | .tex 文件 | .hpb 二进制文件 |
| **数据加载** | Ruby 解析 | C++ 内存映射 |
| **性能** | 较慢 | 快速 |
| **可读性** | 高（文本） | 低（二进制） |

### 3. 集成方式差异

| 维度 | 原始库 | OH |
|-----|---------|-----|
| **集成方式** | Ruby gem | 资源文件 |
| **构建工具** | make/Rakefile | GN |
| **运行时** | Ruby 虚拟机 | 原生代码 |
| **依赖管理** | RubyGems | GN deps |

---

## OH 新增的 API

### 1. Hyphenator 单例模式

**新增功能**: 提供全局单例访问

**实现**:
```cpp
class Hyphenator {
public:
    static Hyphenator& getInstance() {
        static Hyphenator instance;
        std::call_once(initFlag, []() { instance.initTrieTree(); });
        return instance;
    }
};
```

**优势**:
- 全局共享
- 懒加载
- 线程安全

### 2. 语言代码提取

**新增功能**: 从 locale 提取语言代码

**实现**:
```cpp
std::string getLanguageCode(const std::string& locale, int num) {
    // 提取语言代码
    // 例如: "en-US" → "en"
}
```

**示例**:
```cpp
// 优先尝试二级语言代码
std::string langCode = getLanguageCode("en-US", 2);  // "en"

// 回退到一级语言代码
if (langCode.empty()) {
    langCode = getLanguageCode("zh-CN-pinyin", 1);  // "zh"
}
```

### 3. 懒加载机制

**新增功能**: 按需加载语言模式

**实现**:
```cpp
const std::vector<uint8_t>& Hyphenator::getHyphenatorData(
    const std::string& locale) {

    // 检查缓存
    auto search = fHyphenMap.find(langCode);
    if (search != fHyphenMap.end()) {
        return search->second;
    }

    // 首次加载
    return loadPatternFile(langCode);
}
```

**优势**:
- 减少启动时间
- 节省内存
- 按需加载

### 4. 静默降级

**新增功能**: 加载失败时返回空结果

**实现**:
```cpp
const std::vector<uint8_t>& Hyphenator::loadPatternFile(
    const std::string& langCode) {

    std::ifstream file(filename, std::ios::binary);
    if (!file.is_open()) {
        // 静默降级，返回空结果
        return fEmptyResult;
    }

    // 加载文件...
}
```

**优势**:
- 不中断应用
- 自动降级
- 用户体验平滑

---

## 行为变更的 API

### 无行为变更

**说明**: Skia 的 Hyphenator 是全新实现，基于相同的数据，行为一致。

**对比**:
- **输入**: 相同的断词模式（.tex / .hpb）
- **输出**: 相同的断词结果
- **算法**: 相同的断词算法（Knuth 优化）

---

## 废弃或禁用的功能

### 1. Ruby API

**废弃原因**: OH 不使用 Ruby 运行时

**影响**:
- Ruby 应用无法直接使用
- 需要使用 Skia API

**替代方案**:
```ruby
# ❌ 不再支持
require 'hyphen'
lang = Hyphen::Language.load('en-us')
result = lang.hyphenate('word')

# ✅ 使用 NDK 调用 Skia
# 通过 ArkUI / Graphic 2D 使用
```

### 2. TeX 宏

**废弃原因**: OH 不使用 TeX 引擎

**影响**:
- TeX 宏无法直接调用
- 需要通过 Skia API

**替代方案**:
```tex
% ❌ 不再支持
\usepackage{hyph-utf8}
\hypenation{extraordinary}{ex-traor-di-nary}

% ✅ 使用 Skia 文本引擎
// 通过 ArkUI 文本组件使用
```

### 3. Ruby 工具链

**废弃原因**: OH 使用 C++17 构建工具

**影响**:
- Ruby 构建脚本无法使用
- 需要使用 GN 构建系统

**替代方案**:
```bash
# ❌ 不再支持
rake hyphen:generate

# ✅ 使用 GN 构建
./build.sh
```

---

## API 兼容性

### Skia API 层面

**兼容性**: 完全兼容原始断词算法

**证据**:
```cpp
// Skia 使用相同的断词逻辑
// third_party/skia/m133/modules/skparagraph/src/Hyphenator.cpp

// 基于 Knuth 的断词算法
// 与原始 TeX 算法一致
```

**验证方式**:
```bash
# 使用测试工具对比
./reader hyph-en-us.hpb extraordinary
# 输出: ex-traor-di-nary

# 对比原始工具
ruby -e "require 'hyphen'; puts Hyphen::Language.load('en-us').hyphenate('extraordinary')"
# 输出: ex-traor-di-nary
```

### 数据格式层面

**兼容性**: .hpb 格式包含完整的断词模式

**证据**:
```cpp
// .hpb 文件包含
// - 所有断词模式
// - Trie 树结构
// - 代码点映射

// 与原始 .tex 文件包含相同的数据
```

---

## API 使用指南

### 应用开发者视角

**如何使用断词功能**:

#### 方法 1: 通过 ArkUI 文本组件（推荐）

```typescript
// ArkUI (TypeScript)
// 文本组件自动使用断词

Text("This is an extraordinarily long word")
  .fontSize(16)
  .width(200)  // 触发断词
  .textAlign(TextAlign.Start)
```

**特点**:
- 自动使用断词
- 无需手动调用 API
- 根据系统语言自动选择

#### 方法 2: 通过 NDK 调用 Skia（高级）

```cpp
// C++ NDK
#include "modules/skparagraph/include/Paragraph.h"

using namespace skia::textlayout;

// 创建 Paragraph
ParagraphStyle style;
ParagraphBuilder builder(style, fontCollection);

// 添加文本
builder.addText("This is an extraordinarily long word");

// 构建段落
std::unique_ptr<Paragraph> paragraph = builder.Build();

// 布局（自动应用断词）
paragraph->layout(width);

// 渲染
paragraph->paint(canvas, x, y);
```

**特点**:
- 完全控制
- 可自定义断词策略
- 需要了解 Skia API

### 框架开发者视角

**如何添加新语言**:

#### 步骤 1: 更新 tex-hyphen 配置

```gni
// tex-hyphen.gni
tex_source_config = [
  // ... 现有语言
  {
    language = "hyph-xx"
    file_path = "${hyphen_tex_root}/hyph-xx.tex"
  },
]
```

#### 步骤 2: 更新 Skia 语言映射

```cpp
// third_party/skia/m133/modules/skparagraph/src/Hyphenator.cpp

const std::map<std::string, std::string> HPB_FILE_NAMES = {
    // ... 现有映射
    {"xx", "hyph-xx.hpb"},  // 新语言
};
```

#### 步骤 3: 重新构建

```bash
./build.sh
```

---

## 总结

### API 差异总结

| 项目 | 原始库 | OH |
|-----|---------|-----|
| **API 层级** | Ruby 应用层 | C++ 框架层 |
| **调用方式** | 直接调用 | Skia 封装 |
| **数据格式** | .tex 文本 | .hpb 二进制 |
| **集成方式** | Ruby gem | 资源文件 |
| **行为一致性** | - | 完全兼容 |

### 关键差异点

1. **API 语言**: Ruby vs C++
2. **集成层级**: 应用层 vs 框架层
3. **调用方式**: 直接 vs 封装
4. **数据格式**: 文本 vs 二进制

### 最佳实践

对于应用开发者:
- 优先使用 ArkUI 文本组件（自动断词）
- 需要高级功能时使用 NDK 调用 Skia

对于框架开发者:
- 通过 Skia API 使用断词功能
- 遵循现有的语言配置流程

---

**相关文档**:
- [01_Overview.md](./01_Overview.md) - 原始库简介
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 详细评估报告
