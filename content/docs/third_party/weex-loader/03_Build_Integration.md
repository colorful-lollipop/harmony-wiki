# 03 - OH 构建适配

本文档详细介绍 weex-loader 在 OpenHarmony 中的构建系统适配，包括 BUILD.gn 结构、关键编译选项和构建流程。

---

## 3.1 BUILD.gn 结构说明

### 3.1.1 文件位置

```
third_party/weex-loader/
└── BUILD.gn          # GN 构建配置文件
```

### 3.1.2 整体结构

```gn
# 1. 导入依赖的 GN 配置文件
import("//build/ohos.gni")
import("//build/ohos/ace/ace.gni")
import("//foundation/arkui/ace_engine/ace_config.gni")

# 2. 定义输出目录
weex_loader_lib_dir = get_label_info(...)

# 3. 定义输出文件列表
weex_loader_files_set = [...]

# 4. 定义构建目标
## 4.1 主要构建动作
action("build_weex_loader_library") { ... }

## 4.2 复制目标
ohos_copy("weex_loader") { ... }
ohos_copy("scripter") { ... }
ohos_copy("styler") { ... }

## 4.3 Ark HAP 相关目标
ohos_copy("weex_loader_ark_hap") { ... }
ohos_copy("weex_scripter_ark_hap") { ... }
ohos_copy("weex_styler_ark_hap") { ... }
```

### 3.1.3 导入的配置文件

| 导入路径 | 用途 |
|----------|------|
| `//build/ohos.gni` | OH 基础构建配置 |
| `//build/ohos/ace/ace.gni` | ACE (ArkUI) 相关配置 |
| `//foundation/arkui/ace_engine/ace_config.gni` | ACE 引擎配置 |

---

## 3.2 关键编译选项

### 3.2.1 环境变量

weex-loader 构建时依赖以下环境变量：

| 变量名 | 说明 | 取值 | 使用位置 |
|--------|------|------|----------|
| `DEVICE_LEVEL` | 设备级别 | `rich`/`lite`/`card` | babel.config.js, src/*.js |
| `abilityType` | 应用类型 | `page`/`app`/`testrunner` | src/loader.js, src/script.js |
| `projectPath` | 项目路径 | 绝对路径 | src/loader.js, src/util.js |
| `aceManifestPath` | manifest 路径 | 绝对路径 | src/loader.js, src/script.js |
| `logLevel` | 日志级别 | `0`/`1`/`2`/`3` | src/util.js |

### 3.2.2 Babel 配置选项

```javascript
// babel.config.js
module.exports = function(api) {
  const presets = ['@babel/preset-env'];
  const plugins = [
    ['@babel/plugin-transform-modules-commonjs', {'allowTopLevelThis': true}],
    '@babel/plugin-proposal-class-properties',
    '@babel/plugin-transform-runtime'
  ];
  
  // Lite 设备额外配置
  if (process.env.DEVICE_LEVEL === 'lite') {
    plugins.push([
      '@babel/plugin-transform-arrow-functions',
      {spec: true}
    ]);
  }
  
  return { presets, plugins, comments: false };
};
```

**关键选项说明**:
- `@babel/preset-env`: 根据目标环境自动确定转译插件
- `@babel/plugin-transform-modules-commonjs`: 将 ES Module 转为 CommonJS
- `@babel/plugin-proposal-class-properties`: 支持类属性语法
- `@babel/plugin-transform-runtime`: 复用 Babel 辅助代码
- `@babel/plugin-transform-arrow-functions` (Lite): 箭头函数转换

### 3.2.3 输出文件

构建完成后，输出以下文件：

```
out/xxx/weex_loader/lib/
├── element.js          # 元素处理
├── json.js            # JSON 处理
├── legacy.js          # 遗留兼容
├── loader.js          # 主 loader
├── parser.js          # 解析器
├── script.js          # 脚本处理
├── style.js           # 样式处理
├── template.js        # 模板处理
├── util.js            # 工具函数
├── scripter/          # weex-scripter 依赖
│   └── ...
└── styler/            # weex-styler 依赖
    └── ...
```

---

## 3.3 构建流程

### 3.3.1 构建时序图

```
┌─────────────────┐
│   触发构建      │
│ (ace_js2bundle  │
│  或独立构建)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ action()        │
│ build_weex_     │
│ loader_library  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Python 脚本     │
│ build_weex_     │
│ loader_library  │
│ .py             │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────────┐
│ Babel │ │ 复制模块  │
│ 转译  │ │ 文件      │
└───┬───┘ └─────┬─────┘
    │           │
    ▼           ▼
┌───────────┐ ┌───────────┐
│ Uglify    │ │ 输出到    │
│ 压缩      │ │ lib/ 目录 │
└─────┬─────┘ └─────┬─────┘
      │             │
      └──────┬──────┘
             ▼
    ┌─────────────────┐
    │ ohos_copy()     │
    │ 复制到最终      │
    │ 输出目录        │
    └─────────────────┘
```

### 3.3.2 详细构建步骤

#### 步骤 1: 触发构建

当构建依赖 weex-loader 的目标时，GN 会触发 `build_weex_loader_library` action：

```gn
# BUILD.gn
action("build_weex_loader_library") {
  script = "build_weex_loader_library.py"
  outputs = [ weex_loader_lib_dir, ... ]
  inputs = [
    "//third_party/weex-loader/node_modules/@babel/cli/bin/babel.js",
    "//third_party/weex-loader/babel.config.js",
    ...
  ]
  args = [
    "--node", rebase_path(nodejs_path, root_build_dir),
    "--babel-js", rebase_path(_babel_js, root_build_dir),
    "--weex-loader-src-dir", rebase_path("src", root_build_dir),
    "--output-dir", rebase_path(weex_loader_lib_dir, root_build_dir),
    ...
  ]
}
```

#### 步骤 2: Python 脚本执行

`build_weex_loader_library.py` 执行以下操作：

```python
def main():
    options = parse_args()
    
    # 构建命令 1: Babel 转译
    build_cmd = [
        options.node,
        options.babel_js,
        options.weex_loader_src_dir,
        '--out-dir', options.output_dir,
        '--config-file', options.babel_config_js
    ]
    
    # 构建命令 2: 复制模块
    copy_cmd = [
        options.node,
        options.module_source_js,
        options.output_dir
    ]
    
    # 构建命令 3: 代码压缩
    uglify_cmd = [
        options.node,
        options.uglify_source_js,
        options.output_dir
    ]
    
    # 执行构建
    do_build(build_cmd, copy_cmd, uglify_cmd)
```

#### 步骤 3: Babel 转译

使用 Babel 将 ES6+ 源代码转译为兼容代码：

```bash
node //third_party/weex-loader/node_modules/@babel/cli/bin/babel.js \
  src \
  --out-dir out/xxx/weex_loader/lib \
  --config-file //third_party/weex-loader/babel.config.js
```

**转译过程**:
1. 读取 `src/` 目录下的所有 `.js` 文件
2. 根据 `babel.config.js` 应用转译规则
3. 根据 `DEVICE_LEVEL` 环境变量决定是否添加 Lite 特有插件
4. 输出转译后的代码到 `lib/` 目录

#### 步骤 4: 复制依赖模块

`module-source.js` 复制 weex-scripter 和 weex-styler：

```javascript
// module-source.js
copyResource(
  path.resolve(__dirname, './deps/weex-scripter'),
  process.argv[2] + '/scripter'
);
copyResource(
  path.resolve(__dirname, './deps/weex-styler'),
  process.argv[2] + '/styler'
);
```

**复制的文件**:
- `deps/weex-scripter/` → `lib/scripter/`
- `deps/weex-styler/` → `lib/styler/`

#### 步骤 5: Uglify 压缩

`uglify-source.js` 使用 UglifyJS 压缩代码：

```javascript
// uglify-source.js
const uglifyJS = require('uglify-js');

function uglifyCode(code, outPath) {
  const uglifyCode = uglifyJS.minify(code).code;
  fs.writeFileSync(outPath, uglifyCode);
}
```

**压缩范围**: `lib/` 目录下的所有 `.js` 文件

#### 步骤 6: 复制到输出目录

`ohos_copy` 目标将构建结果复制到最终输出目录：

```gn
ohos_copy("weex_loader") {
  deps = [":build_weex_loader_library", ...]
  sources = weex_loader_files_set
  outputs = [target_out_dir + "/$target_name/{{source_file_part}}"]
  part_name = "weex-loader"
  subsystem_name = "thirdparty"
}
```

---

## 3.4 与上游构建系统的差异

### 3.4.1 上游构建方式

Apache weex-loader (v0.7.12) 使用 npm 构建：

```json
// package.json (上游)
{
  "scripts": {
    "build": "node node_modules/babel-cli/bin/babel.js src --out-dir lib",
    "test": "npm run test:build"
  }
}
```

**构建命令**:
```bash
npm install
npm run build
npm test
```

### 3.4.2 OH 构建方式

OpenHarmony 使用 GN + Python 构建：

```bash
# 在 OH 构建系统中
gn gen out
ninja -C out weex_loader
```

**关键差异**:

| 方面 | 上游 | OH |
|------|------|-----|
| 构建工具 | npm + Babel | GN + Python + Node.js |
| 依赖管理 | npm install | prebuilts 预置 |
| 输出目录 | `lib/` | `out/xxx/weex_loader/lib/` |
| 设备分级 | 不支持 | Rich/Lite/Card |
| 压缩 | 可选 | 必选项 (uglify-source.js) |
| 模块复制 | 手动 | 自动 (module-source.js) |

### 3.4.3 适配原因

为什么 OH 不使用 npm 构建：

1. **统一构建系统**: OH 使用 GN 作为统一构建系统，保持所有模块构建方式一致
2. **预置依赖**: OH 使用预置的 Node.js 和 npm 模块，避免运行时下载
3. **增量构建**: GN 支持增量构建，提高构建效率
4. **环境隔离**: 通过 Python 脚本隔离构建环境，确保可重复性

---

## 3.5 特殊处理

### 3.5.1 Ark HAP 打包支持

weex-loader 提供专门的 target 用于 Ark 应用打包：

```gn
# 获取 ace_loader_ark 目录
ace_loader_ark_dir = get_label_info("//developtools/ace_js2bundle:ace_loader",
                                    "target_out_dir") + "/ace_loader_ark"

# 复制 weex-loader 到 Ark 目录
ohos_copy("weex_loader_ark_hap") {
  deps = [
    ":build_weex_loader_library",
    ":weex_loader",
    "//developtools/ace_js2bundle:ace_loader_ark_hap",
  ]
  sources = weex_loader_files_set
  outputs = [ace_loader_ark_dir + "/lib/{{source_file_part}}"]
}
```

**用途**: 支持 ArkUI 应用的编译打包

### 3.5.2 Lite 设备特殊处理

在 `babel.config.js` 中，Lite 设备有特殊的 Babel 插件配置：

```javascript
if (process.env.DEVICE_LEVEL === 'lite') {
  const liteArray = [
    ['@babel/plugin-transform-arrow-functions', {spec: true}]
  ];
  plugins.push(...liteArray);
}
```

**原因**: Lite 设备运行时可能不支持箭头函数语法，需要显式转换

### 3.5.3 依赖文件追踪

Python 脚本生成 depfile 支持增量构建：

```python
build_utils.call_and_write_depfile_if_stale(
    lambda: do_build(build_cmd, copy_cmd, uglify_cmd),
    options,
    depfile_deps=depfile_deps,    # 依赖文件列表
    input_paths=depfile_deps,      # 输入路径
    output_paths=([options.output_dir])  # 输出路径
)
```

**depfile 内容示例**:
```
out/xxx/weex_loader/lib/loader.js: \
  third_party/weex-loader/src/loader.js \
  third_party/weex-loader/src/util.js \
  ...
```

---

## 3.6 构建命令参考

### 3.6.1 独立构建 weex-loader

```bash
# 进入 OH 代码根目录
cd /path/to/openharmony

# 生成构建配置
gn gen out --args='...'

# 构建 weex-loader
ninja -C out weex_loader

# 或构建 Ark HAP 版本
ninja -C out weex_loader_ark_hap
```

### 3.6.2 查看构建图

```bash
# 查看 weex-loader 的依赖关系
gn desc out //third_party/weex-loader:weex_loader deps --tree

# 查看所有 targets
gn ls out //third_party/weex-loader
```

### 3.6.3 调试构建

```bash
# 查看详细构建日志
ninja -C out weex_loader -v

# 仅执行 action (跳过其他依赖)
ninja -C out build_weex_loader_library
```

---

## 3.7 常见问题

### Q1: 构建失败，提示找不到 node

**原因**: `prebuilts/build-tools/common/nodejs/current/bin/node` 不存在

**解决**: 确保已下载预置工具链

```bash
./build/prebuilts_download.sh
```

### Q2: Babel 转译报错

**原因**: `node_modules` 不完整

**解决**: 检查 `third_party/weex-loader/node_modules` 是否存在

### Q3: 修改 src/ 后没有重新构建

**原因**: GN 的增量构建未正确检测变化

**解决**: 强制重新构建

```bash
ninja -C out -t clean weex_loader
ninja -C out weex_loader
```

---

**下一步**: 了解依赖关系和使用场景 → [04_Usage_in_OH.md](./04_Usage_in_OH.md)
