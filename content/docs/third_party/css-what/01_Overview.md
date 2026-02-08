# 01\_概述

## 1.1 原始库简介

### 基本信息

| 项目         | 内容                             |
| ------------ | -------------------------------- |
| **库名称**   | css-what                         |
| **当前版本** | v7.0.0 (上游) / 3.1 (OH)         |
| **许可证**   | BSD-2-Clause                     |
| **上游地址** | https://github.com/fb55/css-what |
| **作者**     | Felix Böhm (fb55)                |
| **类型**     | TypeScript 库                    |

### 功能描述

css-what 是一个**CSS 选择器解析器**（CSS Selector Parser），用于将 CSS 选择器字符串解析为结构化的抽象语法树（AST）。它遵循 CSS3 选择器规范，能够处理各种复杂的选择器表达式。

### 核心能力

```
输入: CSS 选择器字符串
       ↓
  css-what.parse()
       ↓
输出: 结构化 AST
```

**示例**：

```typescript
import { parse } from "css-what";

const selector = "div.container > p:not(.highlight)";
const ast = parse(selector);

// 输出:
// [
//   [
//     { type: 'tag', name: 'div' },
//     { type: 'tag', name: 'container', action: 'equals', ... },
//     ...
//   ]
// ]
```

### 支持的选择器类型

| 类型                  | 示例                      | 说明           |
| --------------------- | ------------------------- | -------------- | ----- | -------- |
| **tag**               | `div`, `span`             | 标签选择器     |
| **universal**         | `*`                       | 通配符选择器   |
| **attribute**         | `[attr]`, `[attr=value]`  | 属性选择器     |
| **pseudo**            | `:hover`, `:nth-child(2)` | 伪类选择器     |
| **pseudo-element**    | `::before`, `::after`     | 伪元素选择器   |
| **child**             | `parent > child`          | 子元素选择器   |
| **parent**            | `child < parent`          | 父元素选择器   |
| **sibling**           | `prev ~ sibling`          | 兄弟选择器     |
| **adjacent**          | `prev + adjacent`         | 相邻兄弟选择器 |
| **descendant**        | `ancestor descendant`     | 后代选择器     |
| **column-combinator** | `col                      |                | cell` | 列选择器 |

### API 概览

#### 主要函数

```typescript
// 解析 CSS 选择器字符串
parse(selector: string): Selector[][]

// 将 AST 还原为字符串
stringify(selector: Selector[][]): string
```

#### 类型定义

```typescript
interface AttributeToken {
    type: "attribute";
    name: string;
    action: "equals" | "not" | "start" | "end" | "any" | "exists" | "hyphen";
    value: string;
    ignoreCase: boolean | null;
}

interface PseudoToken {
    type: "pseudo" | "pseudo-element";
    name: string;
    data: string | null;
}

interface TagToken {
    type: "tag";
    name: string;
}

interface CombinatorToken {
    type:
        | "child"
        | "parent"
        | "sibling"
        | "adjacent"
        | "descendant"
        | "column-combinator";
}
```

---

## 1.2 在 OpenHarmony 中的定位

### 集成方式

css-what 以**预编译源文件**的形式集成到 OpenHarmony 中：

```
third_party/css-what/
├── src/                    # TypeScript 源文件
│   ├── index.ts
│   ├── parse.ts
│   ├── stringify.ts
│   ├── types.ts
│   └── ...
├── BUILD.gn               # OH 构建适配
├── bundle.json            # OH 组件配置
└── package.json           # npm 包配置
```

### 在 OH 架构中的位置

```
┌─────────────────────────────────────────┐
│           ArkUI Application             │
├─────────────────────────────────────────┤
│           ace_engine                    │
│    (ArkUI 引擎, JS 运行时)               │
├─────────────────────────────────────────┤
│           jsframework                   │
│      (JS 框架, CSS 解析)                 │
├─────────────────────────────────────────┤
│           css-what                      │
│      (CSS 选择器解析器)                   │
└─────────────────────────────────────────┘
```

### 核心用途

css-what 在 OpenHarmony 中主要用于以下场景：

1. **CSS 样式解析**：解析组件的样式选择器，支持复杂的选择器表达式
2. **样式选择匹配**：在运行时匹配元素与 CSS 选择器
3. **组件选择器支持**：支持 ArkUI 组件的样式选择功能

### 依赖链

```
应用层
  ↓
AceEngine (foundation/arkui/ace_engine)
  ↓
JSFramework (third_party/jsframework)
  ↓
css-what (third_party/css-what)
```

### 为什么选择 css-what

1. **纯 TypeScript 实现**：无需原生代码，跨平台兼容
2. **标准兼容**：完整支持 CSS3 选择器规范
3. **轻量级**：无外部依赖，体积小
4. **活跃维护**：上游社区活跃，持续更新
5. **类型安全**：完整的 TypeScript 类型定义

---

## 1.3 版本对照

### 上游与 OH 版本

| OH 版本 | 上游版本 | 集成时间 | 差异说明                 |
| ------- | -------- | -------- | ------------------------ |
| 3.1     | v7.0.0   | OH 3.1   | 初始集成，版本号策略不同 |

### 版本说明

- **上游版本 (v7.0.0)**：遵循 SemVer 语义化版本
- **OH 版本 (3.1)**：遵循 OH 第三方库版本规范，可能与上游版本号不一致

### 更新策略

由于该库：

- ✅ 无 Patch（纯源码集成）
- ✅ 无平台适配代码
- ✅ 功能稳定

**建议**：跟随上游版本更新，更新时只需：

1. 同步 `package.json` 版本
2. 验证构建正常
3. 运行测试验证兼容性

---

## 1.4 与同类库的对比

### 为什么选择 css-what 而非其他方案

| 方案                    | 优点                  | 缺点             | OH 选择理由     |
| ----------------------- | --------------------- | ---------------- | --------------- |
| **css-what**            | 纯 TS、无依赖、体积小 | 功能相对基础     | ✅ 轻量、够用   |
| postcss-selector-parser | 功能丰富、生态好      | 依赖较多、体积大 | ❌ 过重         |
| css-select              | 兼容性好              | 需要适配器       | ❌ 额外适配成本 |
| 自研                    | 完全可控              | 维护成本高       | ❌ 不必要       |

### 结论

css-what 的**轻量级**和**零依赖**特性使其成为 OpenHarmony 的理想选择，能够满足 CSS 选择器解析的核心需求，同时保持最小的集成复杂度。

---

## 1.5 小结

css-what 是一个轻量级的 CSS 选择器解析库，集成到 OpenHarmony 中用于支持 ArkUI 框架的 CSS 样式选择功能。该库**无 Patch**，直接使用上游源码，通过 BUILD.gn 进行简单的构建适配即可集成到 OH 系统中。

---

_文档版本：1.0_
_最后更新：2026-02-08_
