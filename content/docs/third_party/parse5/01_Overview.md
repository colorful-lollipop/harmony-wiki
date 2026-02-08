# 原始库简介

> parse5 HTML 解析器/序列化器

---

## 基础信息

| 项目         | 内容                                   |
| ------------ | -------------------------------------- |
| **库名称**   | parse5                                 |
| **当前版本** | v7.2.1 (OH 使用版本)                   |
| **许可证**   | MIT                                    |
| **上游地址** | https://github.com/inikulin/parse5.git |
| **官方文档** | https://parse5.js.org/                 |
| **npm 包**   | https://www.npmjs.com/package/parse5   |

---

## 功能概述

parse5 是一个符合 **WHATWG HTML Living Standard (HTML5)** 规范的 HTML 解析和序列化工具集。

### 核心功能

#### 1. HTML 解析器

将 HTML 字符串解析为 DOM 树：

```javascript
const parse5 = require('parse5');

const document = parse5.parse('<div>Hello World</div>');
```

**特点**:

- 完全符合 HTML5 规范
- 与现代浏览器的解析行为一致
- 处理各种边界情况和错误输入

#### 2. HTML 序列化器

将 DOM 树序列化为 HTML 字符串：

```javascript
const parse5 = require('parse5');

const html = parse5.serialize(document);
```

#### 3. SAX 解析器

流式 HTML 解析，适合处理大型文档：

```javascript
const parse5 = require('parse5');
const SAXParser = require('parse5-sax-parser');

const parser = new SAXParser();
// 设置事件处理器...
parser.parse('<div>Content</div>');
```

#### 4. 树适配器

提供不同的 DOM 树表示方式：

- `default`: parse5 默认树结构
- `htmlparser2`: 兼容 htmlparser2 的树结构

---

## 技术特性

### 规范兼容性

- ✅ **WHATWG HTML Living Standard** 完全兼容
- ✅ 测试覆盖 95%+ 的 HTML5 规范用例
- ✅ 通过 [html5lib-tests](https://github.com/html5lib/html5lib-tests) 测试套件

### 性能

- 🚀 Node.js 上最快的规范兼容 HTML 解析器
- 🚀 经过性能优化的解析算法
- 🚀 内存使用效率高

### 可靠性

- ✅ 在 [jsdom](https://github.com/tmpvar/jsdom) 中使用
- ✅ 被 [Angular](https://angular.io) 采用
- ✅ 在 [Lit](https://lit.dev)、[Cheerio](https://github.com/cheeriojs/cheerio) 等项目中使用

---

## 工具集 (Toolset)

parse5 不仅仅是一个解析器，而是一个完整的工具集：

| 包名                                  | 功能                  |
| ------------------------------------- | --------------------- |
| `parse5`                              | 核心解析器和序列化器  |
| `parse5-sax-parser`                   | SAX 风格流式解析器    |
| `parse5-parser-stream`                | Node.js 流 API 解析器 |
| `parse5-html-rewriting-stream`        | HTML 重写和转换       |
| `parse5-plain-text-conversion-stream` | 纯文本转换            |
| `parse5-htmlparser2-tree-adapter`     | htmlparser2 兼容层    |

**注意**: OpenHarmony 主要使用核心的 `parse5` 包。

---

## API 示例

### 基础解析

```javascript
const parse5 = require('parse5');

// 解析完整文档
const document = parse5.parse('<!DOCTYPE html><html><body>Hi!</body></html>');

console.log(document.childNodes[1].tagName); // 'html'
```

### 片段解析

```javascript
// 解析 HTML 片段
const fragment = parse5.parseFragment('<div class="test">Content</div>');

console.log(fragment.childNodes[0].tagName); // 'div'
```

### 序列化

```javascript
const html = parse5.serialize(document);
console.log(html); // '<!DOCTYPE html><html><body>Hi!</body></html>'
```

### 使用 Tree Adapter

```javascript
const parse5 = require('parse5');
const htmlparser2 = require('parse5-htmlparser2-tree-adapter');

// 使用 htmlparser2 风格的树
const document = parse5.parse('<div>Content</div>', {
    treeAdapter: htmlparser2,
});
```

---

## 内部架构

### 源代码结构

```
packages/parse5/lib/
├── index.ts                    # 主入口
├── common/                     # 公共模块
│   ├── doctype.ts             # 文档类型定义
│   ├── error-codes.ts         # 错误码
│   ├── foreign-content.ts     # SVG/MathML 外部内容处理
│   ├── html.ts                # HTML 标签和属性定义
│   ├── token.ts               # Token 类型定义
│   └── unicode.ts             # Unicode 相关
├── parser/                     # HTML 解析器
│   ├── index.ts               # 主解析器
│   ├── open-element-stack.ts  # 元素栈管理
│   └── formatting-element-list.ts # 格式化元素列表
├── serializer/                 # HTML 序列化器
│   └── index.ts
├── tokenizer/                  # HTML 分词器
│   ├── index.ts               # 主分词器
│   └── preprocessor.ts        # 字符预处理器
└── tree-adapters/              # 树适配器
    ├── default.ts             # 默认适配器
    └── interface.ts           # 接口定义
```

### 解析流程

```
HTML 字符串
    ↓
Tokenizer (分词器)
    ↓
Tokens (标签、属性、文本等)
    ↓
Parser (解析器)
    ↓
DOM Tree (文档树)
```

---

## 依赖关系

### 构建依赖

| 依赖         | 版本   | 用途               |
| ------------ | ------ | ------------------ |
| `typescript` | ^4.9.5 | TypeScript 编译    |
| `uglify-js`  | 3.17.4 | 代码压缩           |
| `entities`   | ^4.5.0 | HTML 实体编码/解码 |

### 运行时依赖

**无** - parse5 是纯 JavaScript/TypeScript 实现，无运行时外部依赖。

---

## 版本历史

OpenHarmony 使用的版本：

| 版本   | OH 集成日期 | 主要变更                     |
| ------ | ----------- | ---------------------------- |
| v7.2.1 | 2025-05-16  | 当前版本，性能优化和错误修复 |
| v7.1.2 | 历史版本    | 前一个版本                   |

**说明**: OH 的 bundle.json 版本号 (4.0) 是 OH 组件的版本，不是原始库版本。

---

## 在 OpenHarmony 中的定位

### 主要用途

parse5 在 OH 中主要用于：

1. **Ace Engine (ArkUI)**: 提供 HTML 解析能力
2. **Web 组件**: 支持类似 Web 的 HTML 内容处理
3. **DOM 操作**: 支持 DOM 树构建和操作

### 与 upstream 的差异

| 项目         | Upstream | OpenHarmony     | 说明                         |
| ------------ | -------- | --------------- | ---------------------------- |
| **源代码**   | ✅ 使用  | ✅ 完全相同     | 无修改                       |
| **构建系统** | npm/tsc  | GN + 自定义脚本 | 详见 03_Build_Integration.md |
| **模块名称** | `parse5` | `parse`         | 构建产物重命名               |
| **代码压缩** | 可选     | ✅ 必须压缩     | 使用 uglify-js               |
| **类型声明** | ✅ 有    | ❌ 无           | 减小体积                     |

---

## 性能特点

### 解析速度

- 在标准基准测试中，parse5 是 Node.js 上最快的规范兼容 HTML 解析器
- 解析 1MB HTML 文档通常在数十毫秒级别

### 内存使用

- 流式解析模式支持处理超大文档
- 内存使用效率高，适合资源受限环境

---

## 安全性

### 已知风险

- **XSS 防护**: parse5 本身不提供 XSS 防护，需要上层应用过滤输出
- **DoS 攻击**: 恶意构造的 HTML 可能导致解析变慢（上游已知）

### CVE 信息

定期检查上游的 CVE 报告：https://github.com/inikulin/parse5/security/advisories

---

## 参考资料

- [parse5 官方文档](https://parse5.js.org/)
- [parse5 GitHub 仓库](https://github.com/inikulin/parse5)
- [WHATWG HTML 标准](https://html.spec.whatwg.org/)
- [parse5 在线 Playground](http://astexplorer.net/#/1CHlCXc4n4)

---

**下一步**: 阅读 [02_Patches.md](02_Patches.md) 了解 OH 中的 Patch 分析
