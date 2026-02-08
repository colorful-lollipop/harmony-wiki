# 06 - GN 构建系统与编译产物

## GN 构建概述

本项目使用 GN (Generate Ninja) 作为构建系统，通过 `BUILD.gn` 定义构建目标。

## 构建目标 (Targets)

### 根 BUILD.gn

**文件**: `BUILD.gn:1-131`

定义了 4 个主要构建目标：

| Target | 类型 | 说明 | 代码位置 |
|--------|------|------|----------|
| `build_ace_loader_library` | `action` | 构建 ace-loader 库 | `BUILD.gn:21-66` |
| `ace_loader` | `ohos_copy` | 复制 ace-loader 源码 | `BUILD.gn:82-87` |
| `ace_loader_library` | `ohos_copy` | 复制编译后的库 | `BUILD.gn:89-95` |
| `previewer_copy` | `ohos_copy` | 复制预览器 | `BUILD.gn:97-111` |
| `ace_loader_ark` | `ohos_copy` | 复制 Ark 配置 | `BUILD.gn:115-119` |
| `ace_loader_ark_hap` | `ohos_copy` | 复制 Ark HAP 配置 | `BUILD.gn:121-130` |

## Target 详细说明

### 1. build_ace_loader_library

**类型**: `action`

**职责**: 调用 Python 脚本编译 ace-loader 源码

**配置**:
```gn
action("build_ace_loader_library") {
  script = "build_ace_loader_library.py"
  outputs = [ ace_loader_lib_dir ]
  
  inputs = [
    _babel_config_js,
    _babel_js,
    _module_source_js,
    _uglify_source_js,
  ]
  
  args = [
    "--depfile",
    "--node",
    "--babel-js",
    "--ace-loader-src-dir",
    "--babel-config-js",
    "--module-source-js",
    "--uglify-source-js",
    "--output-dir",
  ]
}
```

**代码证据**: `BUILD.gn:21-66`

**依赖**:
- `//prebuilts/build-tools/common/nodejs/current/bin/node`
- `ace-loader/node_modules/@babel/cli/bin/babel.js`
- `ace-loader/babel.config.js`
- `ace-loader/module-source.js`
- `ace-loader/uglify-source.js`

### 2. ace_loader

**类型**: `ohos_copy`

**职责**: 复制 ace-loader 源码到输出目录

**源文件列表**:
```gn
ace_loader_sources = [
  "ace-loader/.npmignore",
  "ace-loader/babel.config.js",
  "ace-loader/index.js",
  "ace-loader/main.product.js",
  "ace-loader/node_modules",
  "ace-loader/npm-install.js",
  "ace-loader/package-lock.json",
  "ace-loader/package.json",
  "ace-loader/sample",
  "ace-loader/webpack.lite.config.js",
  "ace-loader/webpack.rich.config.js",
]
```

**代码证据**: `BUILD.gn:68-87`

### 3. ace_loader_library

**类型**: `ohos_copy`

**职责**: 复制编译后的 ace-loader 库

**依赖**: `:build_ace_loader_library`

**代码证据**: `BUILD.gn:89-95`

### 4. previewer_copy

**类型**: `ohos_copy`

**职责**: 复制预览器到输出目录

**平台适配**:
```gn
if (host_os == "mac") {
  sources = [ "//prebuilts/previewer/darwin/previewer" ]
} else if ("${current_os}_${current_cpu}" == "mingw_x86_64") {
  sources = [ "//prebuilts/previewer/windows/previewer" ]
} else {
  sources = [ "//prebuilts/previewer/linux/previewer" ]
}
```

**代码证据**: `BUILD.gn:97-111`

### 5. ace_loader_ark_hap

**类型**: `ohos_copy`

**职责**: 复制 Ark HAP 配置

**依赖**:
- `:ace_loader`
- `:ace_loader_ark`
- `:build_ace_loader_library`
- `//developtools/ace_ets2bundle:ets_loader_ark_hap`

**代码证据**: `BUILD.gn:121-130`

## 构建脚本

### build_ace_loader_library.py

**文件**: `build_ace_loader_library.py`

**职责**: 
- 调用 Babel 编译 ES6+ 源码到 ES5
- 处理模块依赖
- 生成 depfile

**关键参数**:
```python
parser.add_argument('--depfile')
parser.add_argument('--node')
parser.add_argument('--babel-js')
parser.add_argument('--ace-loader-src-dir')
parser.add_argument('--babel-config-js')
parser.add_argument('--module-source-js')
parser.add_argument('--uglify-source-js')
parser.add_argument('--output-dir')
```

## 第三方依赖构建

### parse5

**文件**: `ace-loader/third_party/parse5/BUILD.gn`

**类型**: `ohos_copy`

**职责**: 复制 parse5 库文件

### weex-loader

**文件**: `ace-loader/third_party/weex-loader/BUILD.gn`

**类型**: `ohos_copy`

**职责**: 复制 weex-loader 库文件

## 编译产物

### 1. 输出目录结构

```
out/
└── developtools/
    └── ace_js2bundle/
        ├── ace_loader/           # 复制的源码
        │   ├── index.js
        │   ├── main.product.js
        │   ├── webpack.rich.config.js
        │   └── ...
        ├── ace_loader_library/   # 编译后的库
        │   └── lib/
        │       ├── index.js
        │       ├── main.product.js
        │       └── ...
        └── previewer/            # 预览器
            └── previewer
```

### 2. 产物类型

| 产物 | 类型 | 说明 | 生成方式 |
|------|------|------|----------|
| `lib/*.js` | JS | 编译后的 loader | Babel |
| `ace_loader/*` | 文件 | 源码复制 | ohos_copy |
| `previewer/*` | 二进制 | 预览器 | ohos_copy |

### 3. Target → 产物映射

| Target | 产物路径 | 产物类型 |
|--------|----------|----------|
| `build_ace_loader_library` | `lib/` | JS 文件 |
| `ace_loader` | `ace_loader/` | 源码目录 |
| `ace_loader_library` | `ace_loader_library/` | 库目录 |
| `previewer_copy` | `previewer/` | 二进制目录 |
| `ace_loader_ark_hap` | `ace_loader_ark/` | 配置目录 |

## 依赖关系

### 构建依赖图

```
ace_loader_ark_hap
    ├── ace_loader
    ├── ace_loader_ark
    ├── build_ace_loader_library
    │       └── ace_loader_src/*.js
    └── ets_loader_ark_hap (external)

ace_loader_library
    └── build_ace_loader_library
```

### 运行时依赖

```
ace-loader (运行时)
    ├── parse5 (HML 解析)
    ├── weex-loader (CSS/JS 解析)
    ├── webpack (构建框架)
    ├── babel (语法转换)
    └── node_modules/*
```

## 关键配置

### 1. bundle.json 构建配置

**文件**: `bundle.json:25-32`

```json
{
  "build": {
    "sub_component": [
      "//developtools/ace_js2bundle:ace_loader",
      "//developtools/ace_js2bundle:ace_loader_library",
      "//developtools/ace_js2bundle:previewer_copy",
      "//developtools/ace_js2bundle:ace_loader_ark_hap"
    ]
  }
}
```

### 2. 第三方依赖

**文件**: `bundle.json:20-24`

```json
{
  "deps": {
    "third_party": [
      "parse5",
      "weex-loader"
    ]
  }
}
```

## 构建命令

### 完整构建

```bash
# 在 OpenHarmony 源码根目录执行
./build.sh --product {product_name} --ccache
```

### 单独构建

```bash
# 构建 ace_loader
ninja -C out/{product_name} developtools/ace_js2bundle:ace_loader

# 构建 ace_loader_library
ninja -C out/{product_name} developtools/ace_js2bundle:ace_loader_library

# 构建 previewer_copy
ninja -C out/{product_name} developtools/ace_js2bundle:previewer_copy
```

### 本地开发构建

```bash
cd ace-loader
npm install
npm run build
```

## 安装路径

### 系统安装路径

```
/system/
└── developtools/
    └── ace_js2bundle/
        ├── ace_loader/
        ├── ace_loader_library/
        └── previewer/
```

### SDK 安装路径

```
{sdk_path}/
└── developtools/
    └── ace_js2bundle/
        ├── ace_loader/
        └── previewer/
```

## 相关文档

- [目录结构](./02_Directory_Structure.md)
- [内部 API](./05_Internal_API.md)
