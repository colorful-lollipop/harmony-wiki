# ace_js2bundle - 项目首页

## 一句话描述

**ace_js2bundle** 是 OpenHarmony 的类 Web 范式编译构建工具，将 HML/CSS/JS 源码转换为可在 ArkUI 框架运行的 JS Bundle 或 Ark 字节码。

## 项目定位

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 应用开发流程                   │
├─────────────────────────────────────────────────────────────┤
│  开发者源码                    编译转换                      运行时 │
│  ┌─────────┐    ┌──────────┐    ┌─────────┐    ┌─────────┐  │
│  │  .hml   │───→│          │───→│  .js    │───→│  ArkUI  │  │
│  │  .css   │───→│ ace_js2  │───→│  .abc   │───→│  框架    │  │
│  │  .js    │───→│ bundle   │───→│  .bin   │───→│         │  │
│  └─────────┘    └──────────┘    └─────────┘    └─────────┘  │
│                      本项目负责此阶段                          │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力

| 能力 | 说明 | 代码证据 |
|------|------|----------|
| **语法编译转换** | HML → JS 渲染函数，CSS → 样式对象 | `ace-loader/src/loader-gen.js:172-209` |
| **语法验证** | 标签合法性、属性校验、事件绑定检查 | `ace-loader/src/card-loader.js:54-105` |
| **多设备支持** | 富设备(Rich)、瘦设备(Lite)、卡片(Card) | `ace-loader/webpack.rich.config.js:351-356` |
| **代码优化** | 代码分割、压缩、Tree Shaking | `ace-loader/webpack.rich.config.js:358-417` |
| **Ark 字节码生成** | JS → ABC (Ark Bytecode) | `ace-loader/src/genAbc-plugin.js:71-130` |

## 快速开始

### 环境准备

```bash
# Node.js 版本要求
npm -v    # 6.14.8
node -v   # v12.18.3
```

### 安装依赖

```bash
cd ace-loader
npm install
```

### 编译示例工程

```bash
# 编译富设备示例
npm run rich

# 编译瘦设备示例
npm run lite

# 编译卡片示例
npm run card
```

### 编译自定义工程

```bash
# Linux/Mac
export aceModuleRoot=/path/to/your/project
export aceModuleBuild=/path/to/output
node ./node_modules/webpack/bin/webpack.js --config webpack.rich.config.js

# Windows
set aceModuleRoot=C:\path\to\your\project
set aceModuleBuild=C:\path\to\output
node .\node_modules\webpack\bin\webpack.js --config webpack.rich.config.js
```

## 项目结构概览

```
developtools_ace_js2bundle/
├── BUILD.gn                    # GN 构建入口
├── bundle.json                 # OHOS 组件配置
├── ace-loader/                 # 主代码目录
│   ├── src/                    # 编译框架源码
│   │   ├── *.js                # webpack 插件和 loader
│   │   └── lite/               # 瘦设备专用代码
│   ├── plugin/                 # 插件目录
│   ├── sample/                 # 工程样例
│   ├── third_party/            # 第三方依赖
│   ├── index.js                # 入口
│   ├── main.product.js         # 主产品逻辑
│   ├── webpack.rich.config.js  # 富设备配置
│   └── webpack.lite.config.js  # 瘦设备配置
└── wiki/                       # 本文档
```

## 关键特性

### 1. 多范式支持
- **HML** (HarmonyOS Markup Language): 类 HTML 的声明式 UI
- **CSS**: 样式定义，支持 LESS/SCSS/SASS
- **JS**: 逻辑代码，支持 ES6+

### 2. 多设备适配
- **富设备 (Rich)**: 手机、平板，完整功能
- **瘦设备 (Lite)**: 手表、IoT，精简功能
- **卡片 (Card/Form)**: 服务卡片，静态展示

### 3. 构建优化
- 代码分割 (Code Splitting)
- 公共代码提取 (Common Chunks)
- 压缩混淆 (Minification)
- 增量编译 (Incremental Build)

### 4. 输出格式
- **JS Bundle**: 标准 JavaScript 文件
- **ABC**: Ark Bytecode（Ark 运行时字节码）
- **BIN**: 二进制格式（瘦设备）

## 相关链接

- [详细架构说明](./03_Architecture.md)
- [目录结构详解](./02_Directory_Structure.md)
- [构建系统说明](./06_Build_System.md)
- [安全风险评估](./07_Security.md)

## 许可证

Apache License 2.0
