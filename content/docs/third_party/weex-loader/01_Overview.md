# 01 - 原始库简介

本文档介绍 weex-loader 原始库的基本信息，以及它在 OpenHarmony 中的作用和定位。

---

## 1.1 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | weex-loader |
| **版本** | v0.7.12 (上游) / 3.1 (OH) |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/apache/weex-loader.git |
| **原始描述** | a webpack loader for weex |

## 1.2 原始功能

### 1.2.1 一句话描述

weex-loader 是一个 **webpack loader**，用于将 Weex 框架的 `.we` 文件（包含 template、style、script）编译为 JavaScript 模块。

### 1.2.2 详细功能

weex-loader 的主要功能包括：

1. **模板编译**: 将 Weex 模板编译为 JavaScript 渲染函数
2. **样式处理**: 解析 CSS 样式并转换为 JSON 格式
3. **脚本处理**: 处理 JavaScript 代码，支持 ES6+ 语法
4. **资源管理**: 管理依赖关系和资源引用

### 1.2.3 输入输出

**输入**: `.we` 文件 (Weex 单文件组件)

```html
<template>
  <div class="container">
    <text class="title">{{title}}</text>
  </div>
</template>

<style>
  .container { flex: 1; }
  .title { font-size: 48px; }
</style>

<script>
  module.exports = {
    data: {
      title: 'Hello Weex'
    }
  }
</script>
```

**输出**: JavaScript 模块 (可在 Weex 运行时中执行)

```javascript
// 编译后的 JavaScript 代码
$app_define$('@app-component/index', [], function($app_require$, $app_exports$, $app_module$) {
  $app_module$.exports = {
    template: function() { /* 渲染函数 */ },
    style: { /* 样式对象 */ }
  }
})
```

## 1.3 上游技术栈

| 技术 | 用途 |
|------|------|
| webpack | 构建工具，loader 运行环境 |
| Babel | ES6+ 代码转译 |
| parse5 | HTML 解析 |
| weex-styler | CSS 解析 |
| weex-scripter | JS 代码处理 |

## 1.4 在 OpenHarmony 中的作用

### 1.4.1 定位

在 OpenHarmony 中，weex-loader 作为 **ace_js2bundle 的核心依赖**，用于：

1. **编译 HML 文件**: 将 HML (Harmony Markup Language) 编译为 JavaScript
2. **JS FA 应用构建**: 支持 JS Feature Ability 应用的编译
3. **CSS 样式解析**: 为轻量级设备提供 CSS 解析能力

### 1.4.2 使用场景

```
开发者 HML 源码
      ↓
ace_js2bundle (构建工具)
      ↓
  weex-loader (本库)
      ├── template.js → HML 转 JS 渲染函数
      ├── style.js → CSS 解析
      └── script.js → JS 处理
      ↓
编译后的 JS Bundle
      ↓
在 ArkUI 或 Lite 引擎中运行
```

### 1.4.3 支持的设备类型

| 设备级别 | 描述 | weex-loader 的作用 |
|----------|------|-------------------|
| **Rich** | 富设备 (手机、平板等) | 完整功能，标准编译 |
| **Lite** | 轻量设备 (IoT 设备等) | 简化编译，优化输出 |
| **Card** | 卡片应用 | 特殊处理，资源优化 |

## 1.5 OH 版本与上游版本的差异

### 1.5.1 版本号体系

| 版本类型 | 版本号 | 说明 |
|----------|--------|------|
| 上游版本 | v0.7.12 | Apache Weex 官方版本 |
| OH 版本 | 3.1 | OpenHarmony 集成版本 |

**注意**: 两个版本号采用不同的版本体系，OH 版本号表示在 OpenHarmony 中的迭代版本。

### 1.5.2 主要差异

| 方面 | 上游版本 | OH 版本 |
|------|----------|---------|
| **输入格式** | `.we` 文件 | `.hml` 文件 |
| **构建系统** | npm + webpack | GN + Python |
| **设备支持** | 通用 | Rich/Lite/Card 分级 |
| **模块系统** | 标准 npm | OH 特有模块 (`@ohos`, `@system`) |
| **Patch 方式** | - | 直接修改源代码 |

### 1.5.3 文件结构对比

**上游版本** (v0.7.12):
```
weex-loader/
├── src/              # 源代码
├── lib/              # 编译输出
├── package.json      # npm 配置
└── test/             # 测试
```

**OH 版本** (3.1):
```
weex-loader/
├── src/                    # 源代码 (含 OH 修改)
├── deps/                   # 依赖库 (weex-scripter, weex-styler)
├── test/                   # 测试
├── BUILD.gn               # GN 构建配置 (新增)
├── build_weex_loader_library.py  # 构建脚本 (新增)
├── babel.config.js        # Babel 配置 (OH 定制)
├── module-source.js       # 模块复制脚本 (新增)
└── uglify-source.js       # 代码压缩脚本 (新增)
```

## 1.6 关键技术点

### 1.6.1 HML 模板编译

将 HML (Harmony Markup Language) 编译为 JavaScript 渲染函数：

```javascript
// HML 输入
<div>
  <text>{{message}}</text>
</div>

// 编译输出
function() {
  return {
    type: 'div',
    children: [{
      type: 'text',
      attr: { value: this.message }
    }]
  }
}
```

### 1.6.2 设备分级支持

weex-loader 在 OH 中最重要的扩展是支持**设备分级**：

```javascript
// 根据 DEVICE_LEVEL 输出不同代码
if (process.env.DEVICE_LEVEL === 'rich') {
  // 富设备: 完整功能
} else if (process.env.DEVICE_LEVEL === 'lite') {
  // 轻量设备: 简化功能
} else if (process.env.DEVICE_LEVEL === 'card') {
  // 卡片: 特殊处理
}
```

### 1.6.3 模块导入适配

适配 OpenHarmony 的模块系统：

```javascript
// 输入
import router from '@system.router';
import prompt from '@ohos.prompt';

// 输出 (处理后)
const router = requireModule('@system.router');
const prompt = requireNapi('prompt');
```

## 1.7 许可证

```
Copyright 2019 Apache Incubator-Weex

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## 1.8 参考链接

- **上游仓库**: https://github.com/apache/weex-loader
- **Apache Weex**: https://weex.apache.org/
- **OpenHarmony**: https://www.openharmony.cn/

---

**下一步**: 了解 OH 适配详情 → [02_Patches.md](./02_Patches.md)
