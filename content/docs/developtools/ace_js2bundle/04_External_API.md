# 04 - 对外 API (N-API)

## 说明

**本项目为纯 Node.js 构建工具，不提供 N-API 对外接口。**

ace_js2bundle 是一个编译时工具（build-time tool），而非运行时库（runtime library）。它在开发机器上运行，将 HML/CSS/JS 源码编译为可在 OpenHarmony 设备上运行的 JS Bundle 或 Ark 字节码。

## 项目类型判定

| 特征 | 本项目 | N-API 项目 |
|------|--------|------------|
| 代码类型 | 纯 JavaScript | C/C++ + JS |
| 运行时机 | 编译时 (Build-time) | 运行时 (Runtime) |
| 输出产物 | JS Bundle / ABC | `.so` 动态库 |
| N-API 注册 | 无 | `napi_module_register` |
| 调用方式 | 命令行 / webpack | JS 代码调用 |

## 代码证据

### 1. 无 N-API 注册代码

搜索 `napi_*` 相关模式：

```bash
$ grep -r "napi_" --include="*.c" --include="*.cc" --include="*.cpp" --include="*.h" .
# 无结果
```

搜索 `NAPI_MODULE`：

```bash
$ grep -r "NAPI_MODULE" --include="*.c" --include="*.cc" --include="*.cpp" --include="*.h" .
# 无结果
```

### 2. 纯 JavaScript 项目

项目结构证据：

```
ace-loader/
├── src/
│   └── *.js          # 纯 JS 文件
├── package.json      # npm 配置
├── webpack.*.config.js
└── index.js          # JS 入口
```

**代码证据**: `ace-loader/package.json:1-76`

### 3. 构建工具特征

package.json 中的 scripts：

```json
{
  "scripts": {
    "build": "./node_modules/.bin/babel ...",
    "rich": "cd sample/rich && webpack --config ../../webpack.rich.config.js",
    "lite": "cd sample/lite && webpack --config ../../webpack.lite.config.js",
    "card": "cd sample/card && webpack --config ../../webpack.rich.config.js"
  }
}
```

**代码证据**: `ace-loader/package.json:14-26`

## 对外接口（命令行）

虽然本项目没有 N-API，但提供以下命令行接口：

### 1. npm scripts

| 命令 | 说明 | 代码位置 |
|------|------|----------|
| `npm run build` | 编译 ace-loader 本身 | `package.json:16` |
| `npm run rich` | 编译富设备示例 | `package.json:17` |
| `npm run lite` | 编译瘦设备示例 | `package.json:18` |
| `npm run card` | 编译卡片示例 | `package.json:19` |

### 2. webpack 配置导出

```javascript
// 富设备配置
module.exports = (env) => {
  setConfigs(env);
  // ...
  return config;
}
```

**代码证据**: `ace-loader/webpack.rich.config.js:290-438`

### 3. Node.js API

```javascript
// 主入口
module.exports = require('./lib/loader')

// main.product.js 导出
module.exports = {
  deleteFolderRecursive,
  readManifest,
  loadEntryObj,
  compileCardModule,
  hashProjectPath,
  checkMultiResourceBuild,
  multiResourceBuild,
  readWorkerFile,
  compareCache,
  systemModules,
  parseAbilityName
}
```

**代码证据**:
- `ace-loader/index.js:16`
- `ace-loader/main.product.js:299-311`

## 与 N-API 项目的关系

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 应用开发                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  开发阶段              编译阶段                运行阶段       │
│  ─────────            ─────────              ─────────       │
│                                                              │
│  ┌─────────┐         ┌──────────┐          ┌─────────┐      │
│  │ 开发者   │────────→│ ace_js2  │─────────→│ ArkUI   │      │
│  │ 编写源码 │         │ bundle   │          │ 运行时   │      │
│  │         │         │ (本项目) │          │         │      │
│  └─────────┘         └──────────┘          └────┬────┘      │
│                                                  │           │
│                                                  ▼           │
│                                           ┌─────────┐       │
│                                           │ N-API   │       │
│                                           │ 模块    │       │
│                                           │ (其他库)│       │
│                                           └─────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 相关项目（提供 N-API）

如果需要在 OpenHarmony 中使用 N-API，请参考以下项目：

| 项目 | 说明 |
|------|------|
| `arkui/ace_engine` | ArkUI 框架，提供 UI 相关的 N-API |
| `napi` | N-API 运行时支持 |
| 各子系统模块 | 如 `multimedia/audio` 等，提供各自领域的 N-API |

## 总结

- **本项目无 N-API 接口**
- 本项目是**编译时工具**，不是**运行时库**
- 对外接口为**命令行**和**Node.js 模块导出**
- N-API 接口由 OpenHarmony 运行时和其他子系统提供
