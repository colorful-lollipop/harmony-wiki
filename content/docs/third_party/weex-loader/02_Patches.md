# 02 - Patch 详细分析

本文档详细分析 weex-loader 在 OpenHarmony 中的所有修改，包括 Patch 文件分析和源代码修改分析。

---

## 2.1 Patch 清单总览

### 2.1.1 Patch 文件搜索结果

通过全面搜索，**未发现任何 `.patch` 文件**或 `patches` 目录：

```bash
find . -name "*.patch" -o -name "patches" -type d
# 结果: 未找到匹配项
```

### 2.1.2 修改方式说明

该库在 OH 中**未使用传统 Patch 方式**进行适配，而是采用**直接修改源代码**的方式。

**原因分析**:
1. 修改范围广，涉及多个核心文件
2. 需要新增大量 OH 特有文件
3. 与上游代码差异较大，Patch 维护成本高

### 2.1.3 修改统计

| 修改类型 | 数量 | 文件列表 |
|----------|------|----------|
| **新增文件** | 5 | BUILD.gn, build_weex_loader_library.py, babel.config.js, module-source.js, uglify-source.js |
| **修改源代码** | 5+ | src/loader.js, src/template.js, src/style.js, src/script.js, src/util.js |
| **Patch 文件** | 0 | 无 |

---

## 2.2 新增文件分析

### 2.2.1 BUILD.gn

**文件路径**: `BUILD.gn`

**修改目的**: 为 OpenHarmony 的 GN 构建系统提供构建配置

**关键内容**:

```gn
# 定义 weex-loader 输出文件列表
weex_loader_files_set = [
  weex_loader_lib_dir + "/element.js",
  weex_loader_lib_dir + "/json.js",
  weex_loader_lib_dir + "/legacy.js",
  weex_loader_lib_dir + "/loader.js",
  weex_loader_lib_dir + "/parser.js",
  weex_loader_lib_dir + "/script.js",
  weex_loader_lib_dir + "/style.js",
  weex_loader_lib_dir + "/template.js",
  weex_loader_lib_dir + "/util.js",
]

# 构建动作: 调用 Python 脚本执行转译
action("build_weex_loader_library") {
  script = "build_weex_loader_library.py"
  # ... 输入输出配置
}

# 复制目标
ohos_copy("weex_loader") { ... }
ohos_copy("scripter") { ... }
ohos_copy("styler") { ... }
```

**OH 价值**:
- 集成到 OH 的 GN 构建系统
- 支持 ace_js2bundle 的依赖引用
- 提供 Ark HAP 打包支持

**升级建议**: 此文件为 OH 特有，升级上游版本时保留

---

### 2.2.2 build_weex_loader_library.py

**文件路径**: `build_weex_loader_library.py`

**修改目的**: 构建脚本，协调 Babel 转译、模块复制和代码压缩

**关键内容**:

```python
def main():
    options = parse_args()
    
    # 1. Babel 转译
    build_cmd = [
        options.node, options.babel_js, options.weex_loader_src_dir,
        '--out-dir', options.output_dir,
        '--config-file', options.babel_config_js
    ]
    
    # 2. 复制模块
    copy_cmd = [options.node, options.module_source_js, options.output_dir]
    
    # 3. 代码压缩
    uglify_cmd = [options.node, options.uglify_source_js, options.output_dir]
    
    # 执行构建
    build_utils.call_and_write_depfile_if_stale(...)
```

**OH 价值**:
- 协调多步构建流程
- 生成 depfile 支持增量构建
- 集成到 OH 的 build_utils 工具

**升级建议**: 此文件为 OH 特有，升级上游版本时保留

---

### 2.2.3 babel.config.js

**文件路径**: `babel.config.js`

**修改目的**: 配置 Babel 转译，添加 Lite 设备支持

**关键内容**:

```javascript
module.exports = function(api) {
  api.cache(true);

  const presets = ['@babel/preset-env'];
  const plugins = [
    ['@babel/plugin-transform-modules-commonjs', {'allowTopLevelThis': true}],
    '@babel/plugin-proposal-class-properties',
    '@babel/plugin-transform-runtime'
  ];
  
  // OH 特有: Lite 设备额外添加箭头函数转换
  if (process.env.DEVICE_LEVEL === 'lite') {
    plugins.push([
      '@babel/plugin-transform-arrow-functions',
      {spec: true}
    ]);
  }
  
  return {
    presets,
    plugins,
    comments: false
  };
};
```

**OH 价值**:
- 根据设备级别调整转译策略
- Lite 设备使用更保守的转译，确保兼容性

**升级建议**: 注意保持 Lite 设备的特殊插件配置

---

### 2.2.4 module-source.js

**文件路径**: `module-source.js`

**修改目的**: 复制 weex-scripter 和 weex-styler 到输出目录

**关键内容**:

```javascript
const TARGET_POSITION = 2;
copyResource(
  path.resolve(__dirname, './deps/weex-scripter'), 
  process.argv[TARGET_POSITION] + '/scripter'
);
copyResource(
  path.resolve(__dirname, './deps/weex-styler'), 
  process.argv[TARGET_POSITION] + '/styler'
);
```

**OH 价值**:
- 确保依赖库被正确复制到输出目录
- 支持 ace_js2bundle 引用这些库

**升级建议**: 此文件为 OH 特有，升级上游版本时保留

---

### 2.2.5 uglify-source.js

**文件路径**: `uglify-source.js`

**修改目的**: 使用 UglifyJS 压缩编译后的代码

**关键内容**:

```javascript
const uglifyJS = require('uglify-js');

function uglifyCode(code, outPath) {
  const uglifyCode = uglifyJS.minify(code).code;
  fs.writeFileSync(outPath, uglifyCode);
}
```

**OH 价值**:
- 减小输出文件体积
- 提高运行时加载性能

**升级建议**: 此文件为 OH 特有，升级上游版本时保留

---

## 2.3 源代码修改分析

### 2.3.1 src/loader.js

**修改文件**: `src/loader.js`

**修改摘要**:
- 引入 Lite 设备枚举: `const { DEVICE_LEVEL } = require('./lite/lite-enum')`
- 添加设备级别判断逻辑
- 支持 Rich/Lite/Card 三种输出格式

**关键代码变更**:

```javascript
// 原始: 单一输出格式
function loadApp(...) {
  // ... 生成 Rich 格式代码
}

// OH 修改: 分级输出
function loadApp(...) {
  if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.RICH || 
      process.env.DEVICE_LEVEL === 'card') {
    // Rich/Card 格式
    output += `
    $app_define$('@app-application/${name}', [], function(...) {
      $app_script$(...)
      ...
    })
    `;
  }
  if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
    // Lite 格式
    output += `var options=$app_script$...module.exports=new ViewModel(options);`;
  }
}
```

**OH 需求**: 支持不同设备级别的代码输出

**升级建议**: 升级时需保留 DEVICE_LEVEL 判断逻辑

---

### 2.3.2 src/template.js

**修改文件**: `src/template.js`

**修改摘要**:
- 引入 Lite 模板转换器: `const compiler = require('./lite/lite-transform-template')`
- 添加 Lite 设备模板处理逻辑
- 支持 element 标签处理

**关键代码变更**:

```javascript
// OH 特有: Lite 设备转换
if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
  if (hasError) {
    parsed = `function () { return {} }`;
  } else {
    parsed = compiler.transformTemplate(parsed);
  }
}
```

**OH 需求**: Lite 设备需要简化模板格式

**升级建议**: 注意保持 Lite 设备转换逻辑

---

### 2.3.3 src/style.js

**修改文件**: `src/style.js`

**修改摘要**:
- 引入 Lite 样式转换器: `const compileStyle = require('./lite/lite-transform-style')`
- 添加 Lite 设备样式处理逻辑

**关键代码变更**:

```javascript
// OH 特有: Lite 设备样式转换
if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
  parsed = compileStyle.transformStyle(parsed);
}
```

**OH 需求**: Lite 设备需要转换样式格式

**升级建议**: 注意保持 Lite 设备样式转换

---

### 2.3.4 src/script.js

**修改文件**: `src/script.js`

**修改摘要**:
- 引入 DEVICE_LEVEL 枚举
- 添加 Rich 设备的 VM 对象校验逻辑
- 添加 Lite 设备的直接输出逻辑

**关键代码变更**:

```javascript
// OH 特有: Rich 设备 VM 对象校验
if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.RICH || 
    process.env.DEVICE_LEVEL === 'card') {
  // VM 对象属性和访问器校验
  parsed += `\nvar moduleOwn = exports.default || module.exports;
  var accessors = ['public', 'protected', 'private'];
  if (moduleOwn.data && accessors.some(...)) {
    throw new Error('For VM objects...');
  }
  ...`;
  let result = `module.exports = function(module, exports, $app_require$){${parsed}}`;
}

// Lite 设备直接输出
if (process.env.DEVICE_LEVEL === DEVICE_LEVEL.LITE) {
  callback(null, parsed, map);
}
```

**OH 需求**: Rich 设备需要 VM 对象校验，Lite 设备需要简化输出

**升级建议**: 注意保持 VM 校验逻辑和 Lite 输出逻辑

---

### 2.3.5 src/util.js

**修改文件**: `src/util.js`

**修改摘要**:
- 添加 OH 模块导入解析: `parseRequireModule` 函数
- 支持 `@ohos.xxx` 和 `@system.xxx` 模块
- 支持 `libxxx.so` NAPI 模块

**关键代码变更**:

```javascript
// OH 特有: OH 模块导入方法
const methodForOthers = `
function requireModule(moduleName) {
  const systemList = ['system.router', 'system.app', ...];
  var target = '';
  if (systemList.includes(moduleName.replace('@', ''))) {
    target = $app_require$('@app-module/' + moduleName.substring(1));
    return target;
  }
  var shortName = moduleName.replace(/@[^.]+\.([^.]+)/, '$1');
  target = requireNapi(shortName);
  ...
  if (typeof ohosplugin !== 'undefined' && /@ohos/.test(moduleName)) {
    target = ohosplugin;
    for (let key of shortName.split('.')) {
      target = target[key];
      if(!target) break;
    }
    ...
  }
  return target;
}
`;

// 模块校验函数 (OH 特有)
export function checkModuleIsVaild(requireStatementExec, resourcePath) {
  // Lite 设备模块校验逻辑
  if (process.env.DEVICE_LEVEL !== 'lite' || ...) return;
  // 校验模块是否在 oh-package.json5 的 dependencies 中
  ...
}
```

**OH 需求**: 支持 OpenHarmony 特有的模块系统

**升级建议**: 这是核心 OH 特有功能，升级时必须保留

---

## 2.4 修改汇总表

### 2.4.1 新增文件清单

| 文件 | 修改目的 | OH 特有 | 升级注意 |
|------|----------|---------|----------|
| `BUILD.gn` | GN 构建配置 | ✅ | 保留 |
| `build_weex_loader_library.py` | 构建脚本 | ✅ | 保留 |
| `babel.config.js` | Babel 配置 | ⚠️ | 保留 Lite 逻辑 |
| `module-source.js` | 模块复制 | ✅ | 保留 |
| `uglify-source.js` | 代码压缩 | ✅ | 保留 |

### 2.4.2 源代码修改清单

| 文件 | 修改函数/模块 | 修改目的 | 关联 OH 需求 |
|------|---------------|----------|-------------|
| `src/loader.js` | `loadApp`, `loadPage` | 设备分级输出 | Rich/Lite/Card 支持 |
| `src/template.js` | 模块级代码 | Lite 模板转换 | Lite 设备支持 |
| `src/style.js` | 模块级代码 | Lite 样式转换 | Lite 设备支持 |
| `src/script.js` | VM 校验, Lite 输出 | 分级脚本处理 | Rich/Lite 支持 |
| `src/util.js` | `parseRequireModule` | OH 模块导入 | 模块系统适配 |

### 2.4.3 依赖的 OH 环境变量

| 变量名 | 用途 | 影响文件 |
|--------|------|----------|
| `DEVICE_LEVEL` | 设备级别 (rich/lite/card) | loader.js, template.js, style.js, script.js |
| `abilityType` | 应用类型 (page/app) | loader.js, script.js |
| `projectPath` | 项目路径 | loader.js, util.js |
| `aceManifestPath` | manifest 路径 | loader.js, script.js |
| `logLevel` | 日志级别 | util.js |

---

## 2.5 升级建议

### 2.5.1 上游版本升级流程

由于该库**没有 Patch 文件**，升级上游版本需要：

1. **准备工作**:
   ```bash
   # 1. 备份当前 OH 版本
   cp -r weex-loader weex-loader-backup
   
   # 2. 获取上游新版本
   git clone https://github.com/apache/weex-loader.git weex-loader-upstream
   ```

2. **对比分析**:
   ```bash
   # 对比 src/ 目录差异
   diff -ru weex-loader-upstream/src/ weex-loader/src/
   
   # 特别关注 OH 修改的文件
   # - loader.js
   # - template.js
   # - style.js
   # - script.js
   # - util.js
   ```

3. **合并修改**:
   - 合并上游新功能到 OH 版本
   - 重新应用 OH 特有修改 (DEVICE_LEVEL, OH 模块导入等)
   - 保留 OH 新增文件 (BUILD.gn, *.py, *.js)

4. **验证测试**:
   - 运行 test/ 目录下的测试用例
   - 验证 ace_js2bundle 的编译功能
   - 在 Rich/Lite/Card 设备上验证

### 2.5.2 回归风险

| 风险项 | 等级 | 说明 |
|--------|------|------|
| DEVICE_LEVEL 逻辑 | 高 | 影响所有设备级别的代码生成 |
| OH 模块导入 | 高 | 影响 `@ohos` 和 `@system` 模块的使用 |
| Lite 转换 | 中 | 影响 Lite 设备的模板和样式 |
| VM 校验 | 中 | 影响 Rich 设备的运行时行为 |

### 2.5.3 建议的自动化测试

升级后必须执行的测试：

1. **单元测试**: `npm test` (在 test/ 目录)
2. **集成测试**: ace_js2bundle 的编译流程
3. **设备测试**: Rich/Lite/Card 三级别的真机验证

---

## 2.6 TODO(需确认)

### TODO 1: src/lite/ 目录位置

**问题**: 代码中引用了 `./lite/lite-enum`, `./lite/lite-transform-template`, `./lite/lite-transform-style`，但未找到这些文件。

**影响**: 影响 Lite 设备支持的完整分析

**建议**: 需要确认这些文件的实际位置或实现方式

### TODO 2: 上游版本对比

**问题**: 未执行与上游 Apache weex-loader v0.7.12 的详细对比

**影响**: 可能遗漏部分修改点

**建议**: 建议通过 `git diff` 对比上游版本和 OH 版本

### TODO 3: test/ 目录覆盖

**问题**: 不确定 test/ 目录的测试用例是否覆盖 OH 特有功能

**影响**: 升级时可能遗漏回归测试

**建议**: 检查 test/spec 目录，确认是否有 DEVICE_LEVEL 相关的测试

---

**下一步**: 了解构建系统详情 → [03_Build_Integration.md](./03_Build_Integration.md)
