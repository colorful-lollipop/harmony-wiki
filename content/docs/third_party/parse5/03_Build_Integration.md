# OH 构建适配

> parse5 在 OpenHarmony 中的构建系统适配详解

---

## 概述

parse5 在 OpenHarmony 中采用 **构建系统适配** 方案，即通过 GN 构建配置和自定义脚本将上游 TypeScript 源码编译、打包为 OH 可用的 CommonJS 模块。

### 适配特点

- ✅ **零代码修改**: 源代码完全使用 upstream 版本
- ✅ **TypeScript → CommonJS**: 编译为 OH 模块系统支持的格式
- ✅ **代码压缩**: 使用 uglify-js 减小体积
- ✅ **模块重命名**: 输出名为 `parse` 而非 `parse5`
- ✅ **ARK HAP 专用目标**: 为 Ace Engine 提供专用构建产物

---

## 构建流程图

```
┌─────────────────────────────────────────────────────────────┐
│  parse5 源代码 (TypeScript)                                 │
│  packages/parse5/lib/**/*.ts                                │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  build_parse5.py 构建脚本                                    │
│  - 配置 TypeScript 编译参数                                   │
│  - 调用 uglify-source.js 压缩                                 │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
┌──────────────────┐  ┌────────────────────┐
│ TypeScript 编译器 │  │ uglify-js 压缩器   │
│ (tsc)            │  │                   │
│ - Module: CommonJS│  │ - 移除注释         │
│ - Target: ES6    │  │ - 变量混淆         │
│ - 去除类型声明   │  │ - 压缩空白字符     │
└────────┬─────────┘  └─────────┬──────────┘
         │                      │
         ▼                      │
┌──────────────────┐             │
│ 编译产物 (JS)     │             │
│ target_out_dir/  │             │
└────────┬─────────┘             │
         │                      │
         └──────────┬───────────┘
                    ▼
┌─────────────────────────────────────────────────────────────┐
│  最终产物                                                  │
│  - target_out_dir/parse (基础库)                            │
│  - ace_loader_ark_dir/lib/parse (ARK HAP 专用)               │
└─────────────────────────────────────────────────────────────┘
```

---

## BUILD.gn 配置详解

### 文件路径

```
third_party/parse5/BUILD.gn
```

### 完整配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.

import("//build/ohos.gni")
import("//build/ohos/ace/ace.gni")
import("//foundation/arkui/ace_engine/ace_config.gni")

# 定义输出目录
parse5_lib_dir =
    get_label_info(":build_parse5_library", "target_out_dir") + "/parse"
_parse5_project_dir = "//third_party/parse5/packages/parse5"

# ============================================
# 目标 1: 构建基础库
# ============================================
action("build_parse5_library") {
  script = "build_parse5.py"
  depfile = "$target_gen_dir/$target_name.d"
  outputs = [ parse5_lib_dir ]

  # 构建工具路径
  _tsc_js = _parse5_project_dir + "/node_modules/typescript/bin/tsc"
  _uglify_source_js = _parse5_project_dir + "/uglify-source.js"

  inputs = [
    _tsc_js,
    _uglify_source_js,
  ]

  nodejs_path = "//prebuilts/build-tools/common/nodejs/current/bin/node"

  args = [
    "--depfile",
    rebase_path(depfile, root_build_dir),
    "--node",
    rebase_path(nodejs_path, root_build_dir),
    "--tsc-js",
    rebase_path(_tsc_js, root_build_dir),
    "--parse5-project",
    rebase_path(_parse5_project_dir, root_build_dir),
    "--parse5-output-dir",
    rebase_path(parse5_lib_dir, root_build_dir),
    "--uglify-source-js",
    rebase_path(_uglify_source_js, root_build_dir),
  ]
}

# ============================================
# 目标 2: 复制到标准位置
# ============================================
ohos_copy("parse5") {
  deps = [ ":build_parse5_library" ]
  sources = [ parse5_lib_dir ]
  outputs = [ target_out_dir + "/$target_name" ]
  module_install_name = "parse"        # ⭐ 注意：输出名为 "parse"
  subsystem_name = "thirdparty"
  part_name = "parse5"
  license_file = "//third_party/parse5/LICENSE"
}

# ============================================
# 目标 3: ARK HAP 专用
# ============================================
ace_loader_ark_dir = get_label_info("//developtools/ace_js2bundle:ace_loader",
                                    "target_out_dir") + "/ace_loader_ark"

ohos_copy("parse5_ark_hap") {
  deps = [
    ":build_parse5_library",
    ":parse5",
    "//developtools/ace_js2bundle:ace_loader_ark_hap",
  ]
  sources = [ parse5_lib_dir ]
  outputs = [ ace_loader_ark_dir + "/lib/parse" ]
}
```

### 关键配置说明

#### 1. 导入 OH 构建配置

```gn
import("//build/ohos.gni")                           # OH 基础构建规则
import("//build/ohos/ace/ace.gni")                    # Ace Engine 规则
import("//foundation/arkui/ace_engine/ace_config.gni") # Ace 配置
```

**说明**: 通过这些导入，parse5 可以访问 OH 的构建基础设施。

#### 2. 输出目录配置

```gn
parse5_lib_dir =
    get_label_info(":build_parse5_library", "target_out_dir") + "/parse"
```

**说明**: 构建产物输出到 `target_out_dir/parse` 目录。

#### 3. 模块重命名

```gn
ohos_copy("parse5") {
    module_install_name = "parse"  # 输出名为 "parse"
}
```

**说明**:

- 构建目标名为 `parse5`
- 但安装后模块名为 `parse`
- 这样在代码中引用时使用 `require('parse')`

#### 4. ARK HAP 专用目标

```gn
ohos_copy("parse5_ark_hap") {
    outputs = [ ace_loader_ark_dir + "/lib/parse" ]
}
```

**说明**: 为 Ace Engine 的 ARK HAP 模块提供专用构建产物。

---

## build_parse5.py 脚本详解

### 脚本功能

`build_parse5.py` 是 parse5 在 OH 中的核心构建脚本，负责：

1. 调用 TypeScript 编译器
2. 配置编译参数
3. 调用代码压缩脚本
4. 生成 depfile 用于 GN 增量构建

### 关键代码分析

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Copyright (c) 2020 Huawei Device Co., Ltd.

import os
import sys
import subprocess
import argparse

# 支持标准系统和 small 系统
standard_system_build_dir = os.path.join(os.path.dirname(__file__), os.pardir,
    os.pardir, 'build', 'scripts', 'util')
build_dir = os.path.join(os.path.dirname(__file__), os.pardir, os.pardir,
    os.pardir, os.pardir, os.pardir, 'build', 'maple', 'java', 'util')

# 根据系统类型导入 build_utils
if os.path.exists(standard_system_build_dir):
    sys.path.append(
        os.path.join(standard_system_build_dir, os.pardir, os.pardir))
    from scripts.util import build_utils  # noqa: E402
if os.path.exists(build_dir):
    sys.path.append(os.path.join(build_dir, os.pardir, os.pardir, os.pardir))
    from maple.java.util import build_utils  # noqa: E402
```

**说明**:

- 支持标准系统和 small 系统
- 动态导入 `build_utils`（GN 的 Python 工具库）

```python
def parse_args():
    parser = argparse.ArgumentParser()
    build_utils.add_depfile_option(parser)

    parser.add_argument('--node', help='path to nodejs exetuable')
    parser.add_argument('--tsc-js', help='path to parse5 module tsc')
    parser.add_argument('--parse5-project', help='path to parse5 project')
    parser.add_argument('--parse5-output-dir', help='path to parse5 output')
    parser.add_argument('--uglify-source-js', help='path uglify-source.js')

    options = parser.parse_args()
    return options
```

**说明**: 定义构建所需的命令行参数。

```python
def main():
    options = parse_args()

    # ============================================
    # TypeScript 编译命令
    # ============================================
    build_cmd = [
        options.node, options.tsc_js
    ]
    build_cmd.extend(['--project', options.parse5_project])
    build_cmd.extend(['--outDir', options.parse5_output_dir])
    build_cmd.extend(['--module', 'CommonJS'])      # ⭐ 使用 CommonJS
    build_cmd.extend(['--target', 'ES6'])         # ⭐ 目标 ES6

    # 收集依赖
    depfile_deps = [options.node, options.tsc_js]
    depfile_deps.extend(build_utils.get_all_files(options.parse5_project))

    # ============================================
    # Uglify 压缩命令
    # ============================================
    uglify_cmd = [options.node, options.uglify_source_js, options.parse5_output_dir]
    depfile_deps.append(options.uglify_source_js)

    # 执行构建并写入 depfile
    build_utils.call_and_write_depfile_if_stale(
        lambda: do_build(build_cmd, uglify_cmd),
        options,
        depfile_deps=depfile_deps,
        input_paths=depfile_deps,
        output_paths=([options.parse5_output_dir]))
```

**说明**:

- TypeScript 编译为 CommonJS 模块
- 目标为 ES6
- 使用 depfile 实现增量构建

### 编译参数详解

| 参数        | 值              | 说明                 |
| ----------- | --------------- | -------------------- |
| `--project` | parse5 项目路径 | 指定 TypeScript 项目 |
| `--outDir`  | 输出目录        | 编译产物输出位置     |
| `--module`  | CommonJS        | 模块系统类型         |
| `--target`  | ES6             | JavaScript 目标版本  |

**为什么选择 CommonJS？**

- OpenHarmony 的模块系统基于 CommonJS
- Ace Engine 使用 CommonJS 格式的模块
- 与现有 OH 模块生态一致

**为什么目标 ES6？**

- OH 的 JavaScript 引擎支持 ES6
- ES6 提供更好的性能和语法特性
- 与现代 JavaScript 标准一致

---

## uglify-source.js 脚本详解

### 脚本功能

`uglify-source.js` 是华为编写的自定义代码压缩脚本，用于：

1. 递归遍历输出目录
2. 对每个 JavaScript 文件进行压缩
3. 使用 uglify-js 进行优化

### 完整代码

```javascript
/*
 * Copyright (c) 2021 Huawei Device Co., Ltd.
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
import fs from 'fs';
import path from 'path';
import uglifyJS from 'uglify-js';

const SOURCE_POSITION = 2;
readCode(process.argv[SOURCE_POSITION]);

function readCode(inputPath) {
    if (fs.existsSync(inputPath)) {
        const files = fs.readdirSync(inputPath);
        files.forEach(function (file) {
            const filePath = path.join(inputPath, file);
            if (fs.existsSync(filePath)) {
                const fileStat = fs.statSync(filePath);
                if (fileStat.isFile()) {
                    const code = fs.readFileSync(filePath, 'utf-8');
                    uglifyCode(code, filePath);
                }
                if (fileStat.isDirectory()) {
                    readCode(filePath);
                }
            }
        });
    }
}

function uglifyCode(code, outPath) {
    const uglifyCode = uglifyJS.minify(code).code;
    fs.writeFileSync(outPath, uglifyCode);
}
```

### 工作原理

```
1. readCode(inputPath)
   ↓
2. 读取目录中的所有文件
   ↓
3. 对于每个文件：
   ├─ 如果是文件 → 读取内容 → uglifyCode()
   └─ 如果是目录 → 递归调用 readCode()
```

### UglifyJS 压缩效果

**压缩前**:

```javascript
function parse(html) {
    // This is a comment
    const parser = new Parser();
    return parser.parse(html);
}
```

**压缩后**:

```javascript
function parse(e) {
    const r = new Parser();
    return r.parse(e);
}
```

**压缩效果**:

- 移除注释
- 移除空白字符
- 变量名混淆
- 通常可减少 30-50% 的代码体积

### 为什么要压缩？

| 原因         | 说明                    |
| ------------ | ----------------------- |
| **ROM 占用** | 减少 OH 系统的 ROM 占用 |
| **加载速度** | 减少模块加载时间        |
| **内存占用** | 减少运行时内存占用      |
| **安全**     | 混淆代码，增加逆向难度  |

---

## TypeScript 配置

### tsconfig.json

```json
{
    "extends": "../../tsconfig.json",
    "compilerOptions": {
        "rootDir": "lib",
        "outDir": "dist",
        "lib": ["es2020"],
        "declaration": false, // ⭐ 不生成类型声明
        "declarationMap": false, // ⭐ 不生成声明映射
        "sourceMap": false // ⭐ 不生成 sourceMap
    },
    "include": ["**/*.ts"],
    "exclude": ["**/*.test.ts", "dist"]
}
```

### 关键配置说明

#### 1. 不生成类型声明

```json
"declaration": false
```

**原因**:

- OH 模块不使用 TypeScript 类型检查
- 去除 `.d.ts` 文件可减少体积
- 类型声明文件大约占 10-20% 的体积

#### 2. 不生成 sourceMap

```json
"sourceMap": false
```

**原因**:

- production 环境不需要调试信息
- sourceMap 通常占 10-15% 的体积
- 保护源码逻辑

#### 3. 目标库

```json
"lib": ["es2020"]
```

**说明**: 使用 ES2020 标准，包含最新的 JavaScript 特性。

---

## 依赖管理

### 构建依赖

**packages/parse5/package.json**:

```json
{
    "dependencies": {
        "entities": "^4.5.0", // HTML 实体编码/解码（运行时）
        "typescript": "^4.9.5", // TypeScript 编译器（构建时）
        "uglify-js": "3.17.4" // 代码压缩（构建时）
    }
}
```

**说明**:

- `entities`: 运行时依赖，随 parse5 一起打包
- `typescript`: 构建工具，仅在构建时使用
- `uglify-js`: 构建工具，仅在构建时使用

### Node.js 依赖

```python
nodejs_path = "//prebuilts/build-tools/common/nodejs/current/bin/node"
```

**说明**: 使用 OH 预置的 Node.js 环境。

---

## 构建产物结构

### 输出目录

```
target_out_dir/
└── parse/                    # 压缩后的 CommonJS 模块
    ├── parser/
    ├── serializer/
    ├── tokenizer/
    ├── tree-adapters/
    ├── common/
    └── index.js             # 主入口
```

### ARK HAP 专用输出

```
developtools/ace_js2bundle/
└── ace_loader_ark_dir/
    └── lib/
        └── parse/           # Ace Engine 专用
            └── ...          # (与上面相同)
```

---

## 与上游构建的差异

| 项目          | Upstream 构建        | OH 构建                 |
| ------------- | -------------------- | ----------------------- |
| **构建系统**  | npm/tsc              | GN + 自定义脚本         |
| **输出格式**  | ES Module + CommonJS | 仅 CommonJS             |
| **代码压缩**  | 可选                 | 必须压缩                |
| **类型声明**  | 生成                 | 不生成                  |
| **sourceMap** | 生成                 | 不生成                  |
| **模块名称**  | `parse5`             | `parse`                 |
| **输出目录**  | `dist/`              | `target_out_dir/parse/` |

---

## 构建调试

### 启用调试输出

如果要调试构建过程，可以临时修改配置：

**临时启用 sourceMap**:

```json
{
    "compilerOptions": {
        "sourceMap": true // ⭐ 临时启用
    }
}
```

**临时跳过压缩**:

注释掉 `build_parse5.py` 中的 uglify 调用：

```python
# uglify_cmd = [...]  # ⭐ 临时注释
# depfile_deps.append(options.uglify_source_js)

# 修改 do_build
def do_build(build_cmd, uglify_cmd):
    build_utils.check_output(build_cmd)  # ⭐ 只运行 tsc
    # build_utils.check_output(uglify_cmd)  # 跳过压缩
```

### 查看构建日志

```bash
# GN 构建时查看详细日志
ninja -C out/ohos-arm64 -v

# 或者在 BUILD.gn 中添加 prints
```

---

## 常见问题

### Q1: 为什么不使用上游的构建脚本？

**A**: 上游使用 npm 构建，而 OH 使用 GN 构建系统。需要适配 GN 构建流程。

### Q2: 为什么不用 Webpack 等现代打包工具？

**A**:

- OH 的构建系统基于 GN
- 不希望引入额外的构建依赖
- 当前的方案已经满足需求

### Q3: 压缩后如何调试问题？

**A**:

- 开发时可以临时跳过压缩（参考"构建调试"章节）
- 使用 sourceMap（临时启用）
- 在上游代码中调试

### Q4: 如何修改构建配置？

**A**:

- 修改 `BUILD.gn`: 修改 GN 构建目标
- 修改 `build_parse5.py`: 修改编译流程
- 修改 `uglify-source.js`: 修改压缩策略
- 修改 `tsconfig.json`: 修改 TypeScript 配置

---

## 总结

### 构建适配要点

1. ✅ **TypeScript → CommonJS**: 编译为 OH 兼容的模块格式
2. ✅ **代码压缩**: 使用 uglify-js 减小体积
3. ✅ **模块重命名**: 输出为 `parse`
4. ✅ **多目标输出**: 支持基础库和 ARK HAP 两种输出
5. ✅ **增量构建**: 使用 depfile 实现 GN 增量构建

### 关键文件

| 文件                               | 用途            | 重要性 |
| ---------------------------------- | --------------- | ------ |
| `BUILD.gn`                         | GN 构建配置     | ⭐⭐⭐ |
| `build_parse5.py`                  | 构建脚本        | ⭐⭐⭐ |
| `packages/parse5/uglify-source.js` | 代码压缩        | ⭐⭐   |
| `packages/parse5/tsconfig.json`    | TypeScript 配置 | ⭐⭐   |

---

## 参考资料

- [02_Patches.md](02_Patches.md) - 为什么不需要代码 Patch
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 在 OH 中的使用方式
- [\_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估结果

---

**下一步**: 阅读 [04_Usage_in_OH.md](04_Usage_in_OH.md) 了解在 OH 中的使用方式
