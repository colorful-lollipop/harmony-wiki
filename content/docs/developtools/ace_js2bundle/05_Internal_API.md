# 05 - 内部 API 与模块接口

## 模块概览

```
┌─────────────────────────────────────────────────────────────────┐
│                      内部模块结构                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    核心模块 (Core)                        │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │  │
│  │  │ index.js    │ │ main.product│ │ loader-gen  │        │  │
│  │  │ (入口)       │ │    .js      │ │    .js      │        │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   插件模块 (Plugins)                      │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │  │
│  │  │ resource │ │ compile  │ │ genAbc   │ │ genBin   │    │  │
│  │  │ -plugin  │ │ -plugin  │ │ -plugin  │ │ -plugin  │    │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   转换模块 (Transformers)                 │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │  │
│  │  │ template │ │ style    │ │ script   │ │ card     │    │  │
│  │  │ .js      │ │ .js      │ │ .js      │ │ -loader  │    │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   瘦设备模块 (Lite)                       │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │  │
│  │  │ lite-    │ │ lite-    │ │ lite-    │ │ lite-    │    │  │
│  │  │ transform│ │ transform│ │ customize│ │ utils    │    │  │
│  │  │ -template│ │ -style   │ │ .js      │ │ .js      │    │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 核心模块 API

### 1. index.js

**文件**: `ace-loader/index.js:16`

**职责**: 模块入口，导出主 loader

**接口**:
```javascript
module.exports = require('./lib/loader')
```

**稳定性**: 稳定

### 2. main.product.js

**文件**: `ace-loader/main.product.js:299-311`

**职责**: 主产品逻辑，提供构建工具函数

**导出接口**:

| 函数 | 说明 | 参数 | 返回值 |
|------|------|------|--------|
| `deleteFolderRecursive(url)` | 递归删除文件夹 | `string` url | void |
| `readManifest(manifestFilePath)` | 读取 manifest | `string` path | `object` manifest |
| `loadEntryObj(projectPath, device_level, abilityType, manifestFilePath)` | 加载入口配置 | ... | `object` entryObj |
| `compileCardModule(env)` | 编译卡片模块 | `object` env | void |
| `hashProjectPath(projectPath)` | 项目路径哈希 | `string` path | void |
| `checkMultiResourceBuild(aceBuildJson)` | 检查多资源构建 | `string` path | void |
| `readWorkerFile()` | 读取 Worker 配置 | - | `object\|null` |
| `compareCache(cachePath)` | 比较缓存 | `string` path | void |
| `parseAbilityName(abilityType, projectPath)` | 解析 Ability 名称 | ... | `string` |

**导出常量**:
```javascript
{
  multiResourceBuild,  // 多资源构建配置
  systemModules        // 系统模块列表
}
```

**稳定性**: 稳定

**依赖方向**:
```
main.product.js
    ├── fs (Node.js 内置)
    ├── path (Node.js 内置)
    ├── crypto (Node.js 内置)
    └── shelljs (第三方)
```

## 插件模块 API

### 3. ResourcePlugin

**文件**: `ace-loader/src/resource-plugin.js:118-179`

**职责**: 资源处理插件

**类定义**:
```javascript
class ResourcePlugin {
  constructor(input_, output_, manifestFilePath_, watchCSSFiles_, workerFile_ = null)
  apply(compiler)
}
```

**关键方法**:

| 方法 | 说明 | 代码位置 |
|------|------|----------|
| `copyFile(input, output)` | 复制文件 | `resource-plugin.js:38-73` |
| `circularFile(inputPath, outputPath, ext)` | 循环复制目录 | `resource-plugin.js:75-105` |
| `addPageEntryObj()` | 添加页面入口 | `resource-plugin.js:204-241` |
| `loadWorker(entryObj)` | 加载 Worker | `resource-plugin.js:376-393` |
| `readManifest(manifestFilePath)` | 读取配置 | `resource-plugin.js:293-312` |
| `themeFileBuild(customThemePath, customThemeBuild)` | 构建主题 | `resource-plugin.js:426-449` |

**稳定性**: 稳定

**依赖方向**:
```
ResourcePlugin
    ├── fs
    ├── path
    ├── webpack/lib/SingleEntryPlugin
    ├── ./theme/customThemeStyles
    └── ./theme/ohosStyles
```

### 4. ResultStates (compile-plugin)

**文件**: `ace-loader/src/compile-plugin.js:51-181`

**职责**: 编译结果管理

**类定义**:
```javascript
class ResultStates {
  constructor(options)
  apply(compiler)
}
```

**关键方法**:

| 方法 | 说明 | 代码位置 |
|------|------|----------|
| `addCacheFiles(entryFile, cachePath, entryPaths)` | 添加缓存文件 | `compile-plugin.js:190-204` |
| `printResult(buildPath)` | 打印编译结果 | `compile-plugin.js:231-268` |
| `printWarning()` | 打印警告 | `compile-plugin.js:309-331` |
| `printError(buildPath)` | 打印错误 | `compile-plugin.js:339-370` |

**稳定性**: 稳定

### 5. GenAbcPlugin

**文件**: `ace-loader/src/genAbc-plugin.js:71-130`

**职责**: Ark 字节码生成

**类定义**:
```javascript
class GenAbcPlugin {
  constructor(output_, arkDir_, nodeJs_, workerFile_, isDebug_)
  apply(compiler)
}
```

**关键方法**:

| 方法 | 说明 | 代码位置 |
|------|------|----------|
| `processMultiThreadEntry()` | 多线程入口 | `genAbc-plugin.js:132-140` |
| `invokeWorkerToGenAbc()` | 调用 Worker | `genAbc-plugin.js:261-286` |
| `filterIntermediateJsBundleByHashJson(buildPath, inputPaths)` | 哈希过滤 | `genAbc-plugin.js:296-338` |
| `writeHashJson()` | 写入哈希 | `genAbc-plugin.js:340-359` |
| `splitJsBundlesBySize(bundleArray, groupNumber)` | 按大小分组 | `genAbc-plugin.js:232-259` |
| `generateAbcByEs2AbcOfBundleMode(inputPaths)` | ES2ABC 生成 | `genAbc-plugin.js:575-626` |

**导出函数**:
```javascript
{
  GenAbcPlugin,
  checkWorksFile,
  isWindows,
  isLinux,
  isMacOs,
  isOpenHarmony,
  maxFilePathLength,
  validateFilePathLength,
  isEs2Abc,
  isTs2Abc
}
```

**稳定性**: 稳定

**依赖方向**:
```
GenAbcPlugin
    ├── fs
    ├── path
    ├── cluster
    ├── crypto
    ├── os
    └── child_process
```

### 6. GenBinPlugin

**文件**: `ace-loader/src/genBin-plugin.js:44-70`

**职责**: 二进制文件生成（瘦设备）

**类定义**:
```javascript
class GenBinPlugin {
  constructor(output_, webpackPath_, workerFile_)
  apply(compiler)
}
```

**关键方法**:

| 方法 | 说明 | 代码位置 |
|------|------|----------|
| `writeFileSync(inputString, output, jsBundleFile)` | 写入文件 | `genBin-plugin.js:72-83` |
| `qjscFirst(inputPath, outputPath)` | 第一阶段编译 | `genBin-plugin.js:93-106` |
| `qjscSecond(filePath)` | 第二阶段编译 | `genBin-plugin.js:108-122` |

**稳定性**: 稳定

## 转换模块 API

### 7. loader-gen.js

**文件**: `ace-loader/src/loader-gen.js`

**职责**: Loader 生成器

**关键函数**:

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `codegenHmlAndCss()` | 生成 HML/CSS 代码 | `loader-gen.js:172-209` |
| `generateOutput(that, type, jsFileName, isElement, customLang)` | 生成输出 | `loader-gen.js:212-261` |
| `getLoaderString(type, config)` | 获取 Loader 字符串 | `loader-gen.js:41-61` |

**稳定性**: 稳定

### 8. card-loader.js

**文件**: `ace-loader/src/card-loader.js:33-109`

**职责**: 卡片 Loader

**关键函数**:

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `loader(source)` | 主 loader | `card-loader.js:33-109` |
| `findStyleFile(fileName)` | 查找样式文件 | `card-loader.js:121-150` |
| `addJson(_this, output, fileName, query, elementLastName)` | 添加 JSON | `card-loader.js:163-180` |

**稳定性**: 稳定

### 9. lite-transform-template.js

**文件**: `ace-loader/src/lite/lite-transform-template.js`

**职责**: 瘦设备模板转换

**关键函数**:

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `transformTemplate(value)` | 转换模板 | `lite-transform-template.js:138-147` |
| `transformNode(node)` | 转换节点 | `lite-transform-template.js:154-162` |
| `transformNodeDetail(node)` | 转换节点详情 | `lite-transform-template.js:169-175` |
| `transformOptions(node)` | 转换选项 | `lite-transform-template.js:182-201` |
| `transformFor(node)` | 转换 for 指令 | `lite-transform-template.js:364-378` |
| `transformIf(node)` | 转换 if 指令 | `lite-transform-template.js:385-389` |
| `transformChildren(node)` | 转换子节点 | `lite-transform-template.js:544-555` |

**导出**:
```javascript
exports.transformTemplate = transformTemplate;
```

**稳定性**: 稳定

## 模块依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                        模块依赖关系                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  webpack.rich.config.js                                         │
│       │                                                          │
│       ├──► main.product.js ──► fs, path, crypto, shelljs        │
│       │                                                          │
│       ├──► ResourcePlugin ──► fs, path, SingleEntryPlugin       │
│       │                          ├─► customThemeStyles            │
│       │                          └─► ohosStyles                   │
│       │                                                          │
│       ├──► ResultStates ──► fs, path, Compilation               │
│       │                        JavascriptModulesPlugin            │
│       │                        CachedSource, ConcatSource         │
│       │                                                          │
│       ├──► GenAbcPlugin ──► fs, path, cluster                   │
│       │                        crypto, os, child_process          │
│       │                                                          │
│       ├──► GenBinPlugin ──► fs, path, child_process             │
│       │                                                          │
│       └──► loader-gen.js ──► loader-utils, path                 │
│                               util                                │
│                                                                  │
│  webpack.lite.config.js                                         │
│       │                                                          │
│       └──► lite-transform-template.js ──► lite-utils            │
│                                          lite-enum                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 稳定性标注

| 模块 | 稳定性 | 说明 |
|------|--------|------|
| `index.js` | 稳定 | 入口模块，向后兼容 |
| `main.product.js` | 稳定 | 核心工具函数 |
| `ResourcePlugin` | 稳定 | 资源处理核心 |
| `ResultStates` | 稳定 | 编译结果管理 |
| `GenAbcPlugin` | 稳定 | ABC 生成核心 |
| `GenBinPlugin` | 稳定 | BIN 生成 |
| `loader-gen.js` | 稳定 | Loader 生成 |
| `card-loader.js` | 稳定 | 卡片处理 |
| `lite-transform-template.js` | 稳定 | 瘦设备模板转换 |

## 可替换点

| 组件 | 替换方式 | 说明 |
|------|----------|------|
| `parse5` | 修改 `loader.js` | HML 解析器 |
| `weex-loader` | 修改 `style.js` | CSS 解析器 |
| `babel-loader` | 修改 webpack 配置 | JS 转换器 |
| `GenAbcPlugin` | 替换插件 | ABC 生成器 |
| `ResourcePlugin` | 替换插件 | 资源处理器 |

## 相关文档

- [架构说明](./03_Architecture.md)
- [构建系统](./06_Build_System.md)
