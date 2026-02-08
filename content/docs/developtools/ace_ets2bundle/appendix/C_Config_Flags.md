# 附录 C：配置开关

## 环境变量

### 编译器环境变量

| 变量名 | 类型 | 默认值 | 说明 |
|-------|------|--------|------|
| `aceBuildJson` | string | 无 | 构建配置 JSON |
| `xtsMode` | boolean | false | XTS 测试模式 |
| `isPreview` | boolean | false | 预览模式 |
| `compileMode` | string | 'jsbundle' | 编译模式 |
| `runtimeOS` | string | 'default' | 运行时系统 |
| `checkEntry` | boolean | 无 | 入口检查 |
| `obfuscate` | boolean | 无 | 代码混淆 |
| `isDebug` | boolean | 无 | 调试模式 |

### 构建环境变量

| 变量名 | 类型 | 默认值 | 说明 |
|-------|------|--------|------|
| `NODE_OPTIONS` | string | 无 | Node.js 选项 |
| `MAX_WORKERS` | number | CPU 核数 | 最大 Worker 数 |
| `MAX_MEMORY` | number | 4096MB | 最大内存限制 |
| `ETS_FRONTEND_HOME` | string | 无 | ETS 前端路径 |

### 调试环境变量

| 变量名 | 说明 |
|-------|------|
| `DEBUG=ets_checker` | ArkTS 检查器日志 |
| `DEBUG=fast_build` | Fast Build 日志 |
| `DEBUG=arkui-plugins` | ArkUI 插件日志 |
| `DEBUG=koala-wrapper` | Koala 包装器日志 |

## package.json 脚本

### 核心脚本

| 脚本 | 命令 | 说明 |
|------|------|------|
| `build` | `npm run clean && rollup -c` | 构建编译器 |
| `compile` | `node ./compile.js` | 执行编译 |
| `watch` | `rollup -c -w` | 监视模式 |
| `lint` | `eslint .` | 代码检查 |
| `test` | `jest` | 运行测试 |

### 构建选项

```bash
# 产物目录
npm run build -- --output ./dist

# 源目录
npm run build -- --input ./src

# 配置文件
npm run build -- --config rollup.config.js
```

## Rollup 配置

### rollup.config.js

```javascript
export default {
  input: './src/interop/main.js',
  output: {
    file: './dist/lib/main.js',
    format: 'cjs',
    sourcemap: true,
  },
  plugins: [
    resolve(),
    commonjs(),
    typescript(),
  ],
};
```

### 配置选项

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `input` | string | 无 | 入口文件 |
| `output.file` | string | 无 | 输出文件 |
| `output.format` | string | 'es' | 输出格式 |
| `external` | string[] | 无 | 外部依赖 |
| `plugins` | Plugin[] | 无 | 插件列表 |

## TypeScript 配置

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2019",
    "module": "ESNext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "test"]
}
```

### 编译选项

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `target` | string | 'ES2019' | 目标版本 |
| `module` | string | 'ESNext' | 模块系统 |
| `strict` | boolean | true | 严格模式 |
| `declaration` | boolean | true | 生成声明 |

## 项目配置

### module.json

```json
{
  "module": {
    "name": "MyApp",
    "bundleName": "com.example.myapp",
    "versionCode": 1,
    "versionName": "1.0.0",
    "deviceType": ["default"],
    "distributionFilter": {},
    "pages": [
      "pages/index",
      "pages/detail"
    ],
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets"
      }
    ]
  }
}
```

### 配置字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 模块名 |
| `bundleName` | string | 是 | 包名 |
| `versionCode` | number | 是 | 版本码 |
| `pages` | string[] | 是 | 页面列表 |
| `abilities` | object[] | 否 | 能力列表 |

## GN 构建配置

### 条件编译

```gn
# ArkUI X 模式
if (defined(is_arkui_x) && is_arkui_x) {
  deps += [
    "//interface/sdk-js:bundle_arkts",
    "//interface/sdk-js:bundle_kits",
  ]
}

# 标准系统模式
if (is_standard_system) {
  _ace_config_dir = "compiler"
}
```

### 产品配置

```gn
# SDK 构建
sdk_build_arkts = true
sdk_build_public = true
```

## Babel 配置

### babel.config.js

```javascript
module.exports = {
  presets: [
    ['@babel/preset-env', {
      targets: {
        node: 'current'
      }
    }]
  ],
  plugins: [
    '@babel/plugin-proposal-class-properties',
    '@babel/plugin-proposal-object-rest-spread'
  ]
};
```

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [GN 构建目标](07_GN_Targets.md)
