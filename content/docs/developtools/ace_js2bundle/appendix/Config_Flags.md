# 附录 B - 关键配置与 Feature Flags

## 环境变量

### 核心环境变量

| 变量名 | 说明 | 默认值 | 代码位置 |
|--------|------|--------|----------|
| `aceModuleRoot` | 项目根目录 | `process.cwd()` | `webpack.rich.config.js:210` |
| `aceModuleBuild` | 构建输出目录 | `projectPath/build` | `webpack.rich.config.js:212` |
| `aceManifestPath` | manifest.json 路径 | `projectPath/manifest.json` | `webpack.rich.config.js:215` |
| `aceModuleJsonPath` | module.json 路径 | - | `webpack.rich.config.js:218` |
| `aceProfilePath` | profile 路径 | - | `webpack.rich.config.js:219` |
| `aceBuildJson` | build.json 路径 | - | `webpack.rich.config.js:237` |
| `aceConfigPath` | config.json 路径 | - | `webpack.rich.config.js:340` |
| `cachePath` | 缓存目录 | `node_modules/.cache` | `webpack.rich.config.js:214` |
| `DEVICE_LEVEL` | 设备级别 | `rich` | `webpack.rich.config.js:217` |
| `abilityType` | Ability 类型 | `page` | `webpack.rich.config.js:216` |
| `PLATFORM_VERSION` | 平台版本 | `VERSION6` | `webpack.rich.config.js:228-235` |

### 调试环境变量

| 变量名 | 说明 | 默认值 | 代码位置 |
|--------|------|--------|----------|
| `error` | 显示错误 | `true` | `webpack.rich.config.js:204` |
| `warning` | 显示警告 | `true` | `webpack.rich.config.js:205` |
| `note` | 显示提示 | `true` | `webpack.rich.config.js:206` |
| `buildMode` | 构建模式 | `debug` | `webpack.rich.config.js:207` |
| `logLevel` | 日志级别 | `1` | `webpack.rich.config.js:208` |
| `isPreview` | 预览模式 | `false` | `webpack.rich.config.js:209` |
| `watchMode` | 监听模式 | `false` | `webpack.rich.config.js:31` |
| `tddMode` | TDD 模式 | `false` | `resource-plugin.js:460` |

### Ark 编译环境变量

| 变量名 | 说明 | 默认值 | 代码位置 |
|--------|------|--------|----------|
| `panda` | 编译器类型 | `ES2ABC` | `genAbc-plugin.js:498` |
| `minPlatformVersion` | 最小平台版本 | - | `resource-plugin.js:306` |
| `abcCompileSuccess` | ABC 编译成功标志 | `true` | `compile-plugin.js:275` |
| `hashProjectPath` | 项目路径哈希 | - | `main.product.js:189` |
| `projectRootPath` | 项目根路径 | - | `main.product.js:204` |

## Webpack 配置

### 模式配置

```javascript
// webpack.rich.config.js:192
mode: 'development'  // 或 'production'

// 生产模式配置 (buildMode === 'release')
mode: 'production'
optimization.minimize: true
```

### 缓存配置

```javascript
// webpack.rich.config.js:167-169
cache: {
  type: 'filesystem',
  cacheDirectory: path.resolve(process.env.cachePath, '.rich_cache', ...)
}
```

### Source Map 配置

```javascript
// webpack.rich.config.js:191
devtool: 'nosources-source-map'  // 开发模式
// 或
devtool: false  // release 模式 (env.sourceMap === 'none')
```

### 代码分割配置

```javascript
// webpack.rich.config.js:358-378
optimization.splitChunks: {
  chunks(chunk) {
    return !excludeWorker(workerFile, chunk.name) && 
           !/^\.\/TestAbility/.test(chunk.name);
  },
  minSize: 0,
  cacheGroups: {
    vendors: {
      test: /[\\/](node|oh)_modules[\\/]/,
      priority: 20,
      name: "vendors"
    },
    commons: {
      test: /\.js|css|hml$/,
      name: 'commons',
      priority: 10,
      minChunks: 2
    }
  }
}
```

## Feature Flags

### 设备级别判断

```javascript
// webpack.rich.config.js:351-356
if (process.env.DEVICE_LEVEL === 'card') {
  config.module = cardModule;
  config.plugins.push(new AfterEmitPlugin());
} else {
  // rich 设备配置
}
```

### 编译模式判断

```javascript
// webpack.rich.config.js:294-303
if (process.env.compileMode === 'moduleJson') {
  process.env.DEVICE_LEVEL = 'card';
  config.entry = {};
} else {
  deleteFolderRecursive(process.env.buildPath);
  config.entry = loadEntryObj(...);
}
```

### Ark 编译判断

```javascript
// webpack.rich.config.js:242-267
if (env.isPreview === "true" || env.compilerType === 'ark') {
  config.plugins.push(new GenAbcPlugin(...));
} else {
  config.plugins.push(new GenBinPlugin(...));
}
```

### Worker 判断

```javascript
// genAbc-plugin.js:152-169
function checkWorksFile(assetPath, workerFile) {
  if (workerFile === null) {
    return assetPath.search("./workers/") !== 0;
  }
  // ...
}
```

## Babel 配置

### 预设 (Presets)

```javascript
// babel.config.js
presets: ['@babel/preset-env']

// webpack 配置中的 babel-loader
presets: [util.loadBabelModule('@babel/preset-env')]
targets: 'node 8'
```

### 插件 (Plugins)

```javascript
// babel.config.js
plugins: [
  '@babel/plugin-transform-modules-commonjs',
  '@babel/plugin-proposal-class-properties',
  ['@babel/plugin-transform-arrow-functions', { spec: true }]
]

// webpack 配置
plugins: [
  util.loadBabelModule('@babel/plugin-transform-modules-commonjs'),
  util.loadBabelModule('@babel/plugin-proposal-class-properties')
]
```

## 主题配置

### 自定义主题映射

```javascript
// resource-plugin.js:426-449
const CUSTOM_THEME_PROP_GROUPS = require('./theme/customThemeStyles');
const OHOS_THEME_PROP_GROUPS = require('./theme/ohosStyles');

function themeFileBuild(customThemePath, customThemeBuild) {
  const themeContent = JSON.parse(fs.readFileSync(customThemePath));
  Object.keys(styleContent).forEach(function(key) {
    const customKey = CUSTOM_THEME_PROP_GROUPS[key];
    const ohosKey = OHOS_THEME_PROP_GROUPS[key];
    if (ohosKey) {
      newContent['style'][ohosKey] = styleContent[key];
    } else if (customKey) {
      newContent['style'][customKey] = styleContent[key];
    }
  });
}
```

## 文件扩展名配置

### 资源文件白名单

```javascript
// resource-plugin.js:25
const FILE_EXT_NAME = [
  '.js', '.css', '.jsx', '.less', '.sass', 
  '.scss', '.md', '.DS_Store', '.hml', '.json'
];
```

### Loader 测试规则

```javascript
// webpack.rich.config.js:43-110
const richModule = {
  rules: [
    { test: /\.visual$/, use: [...] },
    { test: /(\.hml)(\?[^?]+)?$/, use: [...] },
    { test: /\.png$/, use: [...] },
    { test: /\.css$/, use: [...] },
    { test: /\.less$/, use: [...] },
    { test: /\.(scss|sass)$/, use: [...] },
    { test: /\.jsx?$/, use: [...] }
  ]
};
```

## 路径配置

### 系统模块路径

```javascript
// main.product.js:26-31
const systemModulesPath = path.resolve(__dirname,'../../api');
if (fs.existsSync(systemModulesPath)) {
  systemModules.push(...fs.readdirSync(systemModulesPath));
}
```

### 主题路径

```javascript
// resource-plugin.js:124-127
shareThemePath = path.join(input_, '../share/resources/styles');
internalThemePath = path.join(input_, 'resources/styles');
resourcesPath = path.resolve(input_, 'resources');
sharePath = path.resolve(input_, '../share/resources');
```

## 编译器配置

### Ark 编译器路径

```javascript
// genAbc-plugin.js:441-518
if (process.env.panda === TS2ABC) {
  js2abc = path.join(arkDir, 'build', 'src', 'index.js');
  // Windows: build-win, Mac: build-mac
} else if (process.env.panda === ES2ABC) {
  es2abc = path.join(arkDir, 'build', 'bin', 'es2abc');
  // Windows: es2abc.exe
}
```

### QJSC 路径

```javascript
// genBin-plugin.js:19
const qjsc = path.join(__dirname, '..', 'bin', 'qjsc');
```

## 性能配置

### Worker 数量

```javascript
// genAbc-plugin.js:265
let maxWorkerNumber = isEs2Abc() ? os.cpus().length : 3;

// ES2ABC bundle 模式
const fileThreads = os.cpus().length < 16 ? os.cpus().length : 16;
```

### 文件路径长度限制

```javascript
// genAbc-plugin.js:536-546
function maxFilePathLength() {
  if (isWindows()) return 32766;
  else if (isLinux() || isOpenHarmony()) return 4095;
  else if (isMacOs()) return 1016;
  else return -1;
}
```

## 相关文档

- [构建系统](../06_Build_System.md)
- [内部 API](../05_Internal_API.md)
