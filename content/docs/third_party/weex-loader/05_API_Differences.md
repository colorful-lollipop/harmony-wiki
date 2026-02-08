# 05 - API/接口差异

本文档介绍 weex-loader 在 OpenHarmony 中与上游版本相比的 API 差异，包括环境变量、新增功能和行为变更。

---

## 5.1 环境变量依赖

weex-loader 在 OH 中依赖以下环境变量来控制编译行为：

### 5.1.1 必需的环境变量

| 变量名 | 类型 | 取值范围 | 说明 | 影响文件 |
|--------|------|----------|------|----------|
| `DEVICE_LEVEL` | string | `rich` / `lite` / `card` | 设备级别 | 所有 src/*.js |
| `abilityType` | string | `page` / `app` / `testrunner` | 应用类型 | loader.js, script.js |
| `projectPath` | string | 绝对路径 | 项目根目录 | loader.js, util.js |
| `aceManifestPath` | string | 绝对路径 | manifest.json 路径 | loader.js, script.js |

### 5.1.2 可选的环境变量

| 变量名 | 类型 | 默认值 | 说明 | 影响文件 |
|--------|------|--------|------|----------|
| `logLevel` | number | `0` | 日志级别 | util.js |

**日志级别说明**:
- `0`: 不输出日志
- `1`: 输出 NOTE 级别日志
- `2`: 输出 NOTE + WARNING 级别日志
- `3`: 输出 NOTE + WARNING + ERROR 级别日志

### 5.1.3 环境变量使用示例

```javascript
// src/loader.js
if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.RICH) {
  // Rich 设备: 完整输出
  output += `$app_define$('@app-application/${name}', ...)`;
}

if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
  // Lite 设备: 简化输出
  output += `module.exports = new ViewModel(options);`;
}
```

```javascript
// src/script.js
if (process.env.abilityType === 'page' && 
    fs.existsSync(process.env.aceManifestPath)) {
  // 添加 manifest 处理
  loaders.push({ name: defaultLoaders.manifest, ... });
}
```

---

## 5.2 新增功能

### 5.2.1 设备分级支持

**功能描述**: 根据设备级别 (rich/lite/card) 输出不同的编译代码。

**上游版本**: 不支持，只输出一种格式

**OH 版本**: 支持三种格式

**实现文件**: `src/loader.js`, `src/template.js`, `src/style.js`, `src/script.js`

**代码示例**:

```javascript
// src/loader.js
const { DEVICE_LEVEL } = require('./lite/lite-enum');

function loadApp(_this, name, isEntry, customLang, source) {
  if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.RICH || 
      process.env.DEVICE_LEVEL === 'card') {
    // Rich/Card: 使用 $app_define$ 格式
    output += `$app_define$('@app-application/${name}', [], function(...) {
      ...
    })`;
  }
  
  if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
    // Lite: 使用 ViewModel 格式
    output += `var options = $app_script$
      if ($app_script$.__esModule) {
        options = $app_script$.default;
      }
      options.styleSheet = $app_style$
      module.exports = new ViewModel(options);`;
  }
}
```

### 5.2.2 OH 模块导入解析

**功能描述**: 支持 OpenHarmony 特有的模块系统，包括 `@system.xxx` 和 `@ohos.xxx` 模块。

**上游版本**: 不支持，只支持标准 npm 模块

**OH 版本**: 支持 OH 特有模块

**实现文件**: `src/util.js`

**代码示例**:

```javascript
// src/util.js

// Rich 设备的模块导入方法
const methodForOthers = `
function requireModule(moduleName) {
  // 系统模块列表
  const systemList = [
    'system.router', 'system.app', 'system.prompt', 
    'system.configuration', 'system.image', 'system.device',
    'system.mediaquery', 'ohos.animator', 'system.grid', 
    'system.resource'
  ];
  
  // 1. 系统模块: @system.xxx → $app_require$('@app-module/xxx')
  if (systemList.includes(moduleName.replace('@', ''))) {
    return $app_require$('@app-module/' + moduleName.substring(1));
  }
  
  // 2. OHOS 模块: @ohos.xxx → requireNapi('xxx')
  var shortName = moduleName.replace(/@[^.]+\.([^.]+)/, '$1');
  var target = requireNapi(shortName);
  if (typeof target !== 'undefined' && /@ohos/.test(moduleName)) {
    return target;
  }
  
  // 3. 插件模块: ohosplugin / systemplugin
  if (typeof ohosplugin !== 'undefined' && /@ohos/.test(moduleName)) {
    target = ohosplugin;
    for (let key of shortName.split('.')) {
      target = target[key];
      if (!target) break;
    }
    if (typeof target !== 'undefined') return target;
  }
  
  if (typeof systemplugin !== 'undefined') {
    target = systemplugin;
    for (let key of shortName.split('.')) {
      target = target[key];
      if (!target) break;
    }
    if (typeof target !== 'undefined') return target;
  }
  
  return target;
}
`;

// 转换 require('@system.router') → requireModule('@system.router')
export function parseRequireModule(source, resourcePath) {
  const requireMethod = process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE 
    ? methodForLite 
    : methodForOthers;
  
  source = `${source}\n${requireMethod}`;
  
  const requireReg = /require\(['"]([^()]+)['"]\)/g;
  const REG_SYSTEM = /@(system|ohos)\.(\S+)/g;
  
  let requireStatements = source.match(requireReg);
  if (requireStatements) {
    for (let requireStatement of requireStatements) {
      if (requireStatement.match(REG_SYSTEM)) {
        // 替换为 requireModule
        source = source.replace(requireStatement, 
          requireStatement.replace('require', 'requireModule'));
      }
    }
  }
  
  // 处理 libxxx.so → requireNapi
  source = source.replace(requireReg, (item, item1) => {
    if (libReg.test(item1)) {
      item = `requireNapi("${item1.replace(libReg, '$1')}", true)`;
    }
    return item;
  });
  
  return source;
}
```

### 5.2.3 Lite 设备模板转换

**功能描述**: 将标准模板转换为 Lite 设备支持的格式。

**上游版本**: 不支持

**OH 版本**: 支持

**实现文件**: `src/template.js`

**代码示例**:

```javascript
// src/template.js
const compiler = require('./lite/lite-transform-template');

module.exports = function (source) {
  parseTemplate(source, this.resourcePath)
    .then(({ parsed, log }) => {
      if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
        if (hasError) {
          parsed = `function () { return {} }`;
        } else {
          parsed = compiler.transformTemplate(parsed);
        }
      }
      callback(null, parsed);
    });
}
```

### 5.2.4 Lite 设备样式转换

**功能描述**: 将标准样式转换为 Lite 设备支持的格式。

**上游版本**: 不支持

**OH 版本**: 支持

**实现文件**: `src/style.js`

**代码示例**:

```javascript
// src/style.js
const compileStyle = require('./lite/lite-transform-style');

module.exports = function (source) {
  parseStyle(source, this.resourcePath)
    .then(({ parsed, log }) => {
      if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
        parsed = compileStyle.transformStyle(parsed);
      }
      callback(null, parsed);
    });
}
```

### 5.2.5 模块依赖校验 (Lite)

**功能描述**: Lite 设备需要校验模块是否在 `oh-package.json5` 的依赖列表中。

**上游版本**: 不支持

**OH 版本**: 支持

**实现文件**: `src/util.js`

**代码示例**:

```javascript
// src/util.js - checkModuleIsVaild
export function checkModuleIsVaild(requireStatementExec, resourcePath) {
  // 只在 Lite 设备启用
  if (process.env.DEVICE_LEVEL !== 'lite' || ...) return;
  
  // 读取 oh-package.json5
  const json5Path = path.join(process.env.projectPath, 
    '../../../../', 'oh-package.json5');
  
  if (fs.existsSync(json5Path)) {
    const json5Content = fs.readFileSync(json5Path, 'utf8');
    const content = JSON5.parse(json5Content);
    const dependencies = Object.keys(content.dependencies || {});
    
    // 校验模块是否在依赖列表中
    const moduleName = requireStatementExec[2];
    if (!dependencies.includes(moduleName)) {
      throw new Error(`Cannot find module ${moduleName} ...`);
    }
  }
}
```

### 5.2.6 VM 对象校验 (Rich)

**功能描述**: Rich 设备对 VM 对象的 data 属性和访问器进行校验。

**上游版本**: 不支持

**OH 版本**: 支持

**实现文件**: `src/script.js`

**代码示例**:

```javascript
// src/script.js
if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.RICH || 
    process.env.DEVICE_LEVEL === 'card') {
  // VM 对象校验
  parsed += `\nvar moduleOwn = exports.default || module.exports;
var accessors = ['public', 'protected', 'private'];

// 校验: data 不能与 public/protected/private 共存
if (moduleOwn.data && accessors.some(function (acc) {
  return moduleOwn[acc];
})) {
  throw new Error('For VM objects, attribute data must not coexist with ' +
    'public, protected, or private. Please replace data with public.');
} 

// 将访问器合并到 data
else if (!moduleOwn.data) {
  moduleOwn.data = {};
  moduleOwn._descriptor = {};
  accessors.forEach(function(acc) {
    var accType = typeof moduleOwn[acc];
    if (accType === 'object') {
      moduleOwn.data = Object.assign(moduleOwn.data, moduleOwn[acc]);
      for (var name in moduleOwn[acc]) {
        moduleOwn._descriptor[name] = {access : acc};
      }
    }
  });
}`;
}
```

---

## 5.3 行为变更

### 5.3.1 输入格式变更

| 方面 | 上游版本 | OH 版本 |
|------|----------|---------|
| **输入格式** | `.we` 文件 | `.hml` 文件 |
| **模板语言** | Weex 模板 | Harmony 模板 (HML) |
| **组件定义** | `<we-element>` | `<element>` |

**代码示例**:

```html
<!-- 上游: .we 文件 -->
<we-element name="my-component">
  <template>...</template>
</we-element>

<!-- OH: .hml 文件 -->
<element name="my-component">
  <template>...</template>
</element>
```

### 5.3.2 输出格式变更

| 设备级别 | 输出格式 | 说明 |
|----------|----------|------|
| **Rich** | `$app_define$` 格式 | 完整 VM 对象支持 |
| **Lite** | `ViewModel` 格式 | 简化对象，直接导出 |
| **Card** | `$app_define$` 格式 (简化) | 类似 Rich 但资源受限 |

**Rich 输出示例**:

```javascript
$app_define$('@app-component/index', [], function($app_require$, $app_exports$, $app_module$) {
  $app_script$($app_module$, $app_exports$, $app_require$)
  if ($app_exports$.__esModule && $app_exports$.default) {
    $app_module$.exports = $app_exports$.default
  }
  $app_module$.exports.template = $app_template$
  $app_module$.exports.style = $app_style$
})
```

**Lite 输出示例**:

```javascript
var options = $app_script$
if ($app_script$.__esModule) {
  options = $app_script$.default;
}
options.styleSheet = $app_style$
options.render = $app_template$
module.exports = new ViewModel(options)
```

### 5.3.3 模块系统变更

| 方面 | 上游版本 | OH 版本 |
|------|----------|---------|
| **标准模块** | `require('xxx')` | `require('xxx')` |
| **Weex 模块** | `require('@weex-module/xxx')` | 废弃 |
| **系统模块** | 不支持 | `requireModule('@system.xxx')` |
| **OHOS 模块** | 不支持 | `requireNapi('xxx')` |

### 5.3.4 错误处理变更

| 方面 | 上游版本 | OH 版本 |
|------|----------|---------|
| **错误输出** | 简单 console.error | 分级日志系统 (NOTE/WARNING/ERROR) |
| **编译失败** | 抛出异常 | 根据级别 emitWarning/emitError |
| **Lite 错误** | 同 Rich | 返回空对象 `{}` |

**OH 日志系统代码**:

```javascript
// src/util.js - logWarn
export function logWarn(loader, logs) {
  logs.forEach(log => {
    if (log.reason.startsWith('NOTE') && parseInt(process.env.logLevel) <= 1) {
      loader.emitWarning('noteStartNOTE File:' + ...);
    } else if (log.reason.startsWith('WARN') && parseInt(process.env.logLevel) <= 2) {
      loader.emitWarning('warnStartWARNING File:' + ...);
    } else if (log.reason.startsWith('ERROR') && parseInt(process.env.logLevel) <= 3) {
      loader.emitError('errorStartERROR File:' + ...);
    }
  });
}
```

---

## 5.4 API 使用示例

### 5.4.1 基本使用 (与上游相同)

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.hml$/,
        use: [
          {
            loader: 'weex-loader',
            options: {
              lang: {
                sass: ['sass-loader'],
                scss: ['sass-loader'],
                less: ['less-loader']
              }
            }
          }
        ]
      }
    ]
  }
};
```

### 5.4.2 OH 特有功能使用

```javascript
// 设置环境变量
process.env.DEVICE_LEVEL = 'lite';
process.env.abilityType = 'page';
process.env.projectPath = '/path/to/project';
process.env.aceManifestPath = '/path/to/project/manifest.json';
process.env.logLevel = '3';

// 使用 weex-loader
const loader = require('weex-loader');
```

---

## 5.5 兼容性说明

### 5.5.1 向前兼容性

- **上游代码迁移到 OH**: 需要修改输入格式 (.we → .hml)
- **模块导入**: 需要替换 `@weex-module` 为 `@system` 或 `@ohos`

### 5.5.2 向后兼容性

- **OH 代码无法直接在上游使用**: 依赖 OH 特有功能
- **需要 mock 环境变量**: DEVICE_LEVEL, abilityType 等

---

**下一步**: 了解安全风险分析 → [06_Security.md](./06_Security.md)
