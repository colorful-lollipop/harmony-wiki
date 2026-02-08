# 02 - 目录结构与模块职责

## 顶层目录结构

```
developtools_ace_js2bundle/
├── BUILD.gn                    # GN 构建入口
├── bundle.json                 # OHOS 组件配置
├── LICENSE                     # Apache 2.0 许可证
├── README.md / README_zh.md    # 项目说明
├── OAT.xml                     # 开源合规配置
├── build_ace_loader_library.py # 构建脚本
├── ace-loader/                 # 主代码目录
│   ├── src/                    # 编译框架源码
│   ├── plugin/                 # 插件目录
│   ├── sample/                 # 工程样例
│   ├── test/                   # 单元测试（忽略）
│   ├── third_party/            # 第三方依赖
│   ├── index.js                # 入口
│   ├── main.product.js         # 主产品逻辑
│   ├── webpack.rich.config.js  # 富设备配置
│   ├── webpack.lite.config.js  # 瘦设备配置
│   ├── package.json            # npm 配置
│   └── ...
└── wiki/                       # 本文档
```

## 模块详细说明

### 1. 根目录配置

| 文件 | 职责 | 关键内容 |
|------|------|----------|
| `BUILD.gn` | GN 构建定义 | 定义 4 个 build target |
| `bundle.json` | OHOS 组件配置 | 组件元数据、依赖、构建脚本 |
| `build_ace_loader_library.py` | 构建脚本 | 调用 Babel 编译源码 |

**代码证据**:
- `BUILD.gn:1-131`
- `bundle.json:1-35`

### 2. ace-loader/src/ - 核心源码

#### 2.1 Webpack Loaders

| 文件 | 职责 | 关键函数/类 |
|------|------|------------|
| `index.js` | 主入口 | 导出 loader |
| `loader-gen.js` | Loader 生成器 | `codegenHmlAndCss()`, `generateOutput()` |
| `card-loader.js` | 卡片 Loader | `loader()`, `findStyleFile()` |
| `manifest-loader.js` | Manifest 加载 | `manifestLoader()` |
| `module-script.js` | 模块脚本处理 | - |

**代码证据**: `ace-loader/src/loader-gen.js:172-209`

#### 2.2 Webpack Plugins

| 文件 | 职责 | 关键类 |
|------|------|--------|
| `resource-plugin.js` | 资源处理 | `ResourcePlugin` |
| `compile-plugin.js` | 编译结果管理 | `ResultStates` |
| `genAbc-plugin.js` | ABC 生成 | `GenAbcPlugin` |
| `genBin-plugin.js` | BIN 生成 | `GenBinPlugin` |
| `cardJson-plugin.js` | 卡片 JSON 处理 | `AfterEmitPlugin` |
| `read-json-plugin.js` | JSON 读取 | `ReadJsonPlugin` |
| `manifest-plugin.js` | Manifest 处理 | - |

**代码证据**:
- `ace-loader/src/resource-plugin.js:118-179`
- `ace-loader/src/compile-plugin.js:51-181`
- `ace-loader/src/genAbc-plugin.js:71-130`

#### 2.3 瘦设备专用 (lite/)

| 文件 | 职责 |
|------|------|
| `lite-transform-template.js` | 模板转换 |
| `lite-transform-style.js` | 样式转换 |
| `lite-customize.js` | 自定义处理 |
| `lite-image2bin.js` | 图片转二进制 |
| `lite-image-coverter-plugin.js` | 图片转换插件 |
| `lite-return-exports-plugin.js` | 导出处理 |
| `lite-snapshot-plugin.js` | 快照插件 |
| `lite-enum.js` | 枚举定义 |
| `lite-utils.js` | 工具函数 |

**代码证据**: `ace-loader/src/lite/lite-transform-template.js:138-147`

#### 2.4 工具与配置

| 文件 | 职责 |
|------|------|
| `extgen.js` | 扩展生成 |
| `gen-abc.js` | ABC 生成工具 |
| `manage-bundle-workers.js` | Worker 管理 |
| `resource-reference-script.js` | 资源引用脚本 |

### 3. ace-loader/plugin/ - 插件目录

```
plugin/
├── codegen/          # 代码生成
├── theme/            # 主题处理
│   ├── customThemeStyles.js
│   └── ohosStyles.js
└── templater/        # 模板处理
    ├── index.js
    ├── bind.js
    ├── content.js
    ├── data.js
    ├── rich_component_map.js
    ├── lite_component_map.js
    └── card_component_map.js
```

### 4. ace-loader/sample/ - 工程样例

```
sample/
├── rich/             # 富设备样例
│   ├── pages/
│   ├── i18n/
│   ├── app.js
│   └── manifest.json
├── lite/             # 瘦设备样例
│   ├── pages/
│   ├── app.js
│   └── manifest.json
├── card/             # 卡片样例
│   ├── pages/
│   ├── i18n/
│   └── manifest.json
├── TestRunner/       # 测试运行器样例
├── DataAbility/      # 数据能力样例
└── ServiceAbility/   # 服务能力样例
```

### 5. ace-loader/third_party/ - 第三方依赖

| 目录 | 来源 | 用途 |
|------|------|------|
| `parse5/` | npm | HML/HTML 解析器 |
| `weex-loader/` | npm | CSS/JS 解析器 |

**代码证据**: `bundle.json:20-24`

### 6. 配置文件

| 文件 | 职责 | 关键配置 |
|------|------|----------|
| `webpack.rich.config.js` | 富设备 webpack 配置 | module rules, plugins, optimization |
| `webpack.lite.config.js` | 瘦设备 webpack 配置 | - |
| `babel.config.js` | Babel 配置 | presets, plugins |
| `package.json` | npm 配置 | dependencies, scripts |

**代码证据**: `ace-loader/webpack.rich.config.js:1-439`

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                      模块依赖关系图                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                                           │
│  │  webpack.*   │─────────────────────────────────────┐     │
│  │  .config.js  │                                     │     │
│  └──────┬───────┘                                     │     │
│         │                                             │     │
│         ▼                                             │     │
│  ┌──────────────┐    ┌──────────────┐                │     │
│  │  main.product│◄───│  resource-   │                │     │
│  │    .js       │    │  plugin.js   │                │     │
│  └──────┬───────┘    └──────────────┘                │     │
│         │                                             │     │
│         ▼                                             ▼     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  loader-gen  │───→│  compile-    │───→│  genAbc-     │  │
│  │    .js       │    │  plugin.js   │    │  plugin.js   │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                      │                          │
│         ▼                      ▼                          │
│  ┌──────────────┐    ┌──────────────┐                    │
│  │  lite/*      │    │  genBin-     │                    │
│  │              │    │  plugin.js   │                    │
│  └──────────────┘    └──────────────┘                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 职责边界

### 编译流程职责

| 阶段 | 负责模块 | 输出 |
|------|----------|------|
| 资源扫描 | `resource-plugin.js` | 文件列表 |
| HML 解析 | `parse5` (第三方) | AST |
| 模板转换 | `loader-gen.js` / `lite-transform-template.js` | JS 渲染函数 |
| 样式转换 | `style.js` / `lite-transform-style.js` | 样式对象 |
| 脚本转换 | `babel-loader` | ES5/CommonJS |
| 代码分割 | `webpack` | chunks |
| ABC 生成 | `genAbc-plugin.js` | .abc 文件 |
| BIN 生成 | `genBin-plugin.js` | .bin 文件 |

### 设备适配职责

| 设备 | 配置 | 特殊处理 |
|------|------|----------|
| Rich | `webpack.rich.config.js` | 完整功能 |
| Lite | `webpack.lite.config.js` | 精简功能，特殊转换 |
| Card | `card-loader.js` | 静态 JSON 输出 |

## 相关文档

- [架构说明](./03_Architecture.md)
- [内部 API](./05_Internal_API.md)
- [构建系统](./06_Build_System.md)
