# OpenHarmony 构建适配

## 3.1 构建系统概述

### 3.1.1 构建方式

TypeScript 在 OpenHarmony 中采用**预构建（prebuilt）方式**进行集成。不同于其他第三方库的直接源码编译，TypeScript 被编译为预构建产物（tgz 包），供其他模块引用。

**构建流程**：

```
TypeScript 源码 ──► compile_typescript.py ──► ohos-typescript-4.9.5-r4.tgz
                      │
                      ├── 收集所有源文件
                      ├── 打包为 tgz
                      └── 生成 notice 文件
```

### 3.1.2 构建脚本说明

**编译脚本**：`compile_typescript.py`

该脚本负责将 TypeScript 源文件收集并打包为 OH 格式的预构建产物。

```python
# 主要功能
def main(project_root: str, output_dir: str):
    # 1. 收集 TypeScript 源文件
    sources = collect_sources(project_root)

    # 2. 生成文件列表
    source_list = generate_source_list(sources)

    # 3. 打包为 tgz
    output_path = os.path.join(output_dir, "ohos-typescript-4.9.5-r4.tgz")
    package(sources, output_path)

    # 4. 返回产物路径
    return output_path
```

**输入**：项目根目录路径
**输出**：`ohos-typescript-4.9.5-r4.tgz`

## 3.2 BUILD.gn 配置详解

### 3.2.1 整体结构

```gn
# 导入 OH 构建系统
import("//build/ohos.gni")
import("//build/ohos/notice/notice.gni")

# 源文件列表
typescript_sources = [
    "lib/.gitattributes",
    "lib/README.md",
    # ... 更多文件
]

# 构建目标组
group("build_typescript") {
    deps = [
        "//third_party/typescript:build_typescript_pack",
        "//third_party/typescript:typescript_notice",
    ]
}

# 打包动作
action("build_typescript_pack") {
    sources = typescript_sources
    script = "compile_typescript.py"
    args = [
        rebase_path(get_path_info("./", "abspath")),
        rebase_path("${target_out_dir}"),
    ]
    outputs = [ "${target_out_dir}/ohos-typescript-4.9.5-r4.tgz" ]
}

# 预构建产物
ohos_prebuilt_etc("build_typescript_etc") {
    deps = [
        ":build_typescript",
        ":build_typescript_pack",
    ]
    source = target_outputs_[0]
    install_enable = false
    part_name = "typescript"
    subsystem_name = "thirdparty"
}

# 许可证收集
collect_notice("typescript_notice") {
    license_file = "LICENSE"
    module_source_dir = get_label_info(":${target_name}", "dir")
    outputs = [
        "${sdk_notice_dir}/ets/build-tools/ets-loader/node_modules/typescript.txt",
    ]
}
```

### 3.2.2 关键配置项

#### 源文件列表 typescript_sources

| 配置项 | 说明 |
|-------|------|
| `lib/lib.*.d.ts` | TypeScript 标准库类型定义（ES3 到 ES2022） |
| `lib/typescript.js` | TypeScript 编译器主文件 |
| `lib/tsc.js` | tsc 命令行工具 |
| `lib/tsserver.js` | 语言服务器 |
| `lib/diagnosticMessages.*.json` | 诊断消息本地化文件 |

**配置说明**：源文件列表包含所有 TypeScript 运行所需的 JavaScript 文件和类型定义文件。这些文件被打包到 tgz 中，供运行时使用。

#### 预构建产物配置 ohos_prebuilt_etc

```gn
ohos_prebuilt_etc("build_typescript_etc") {
    # 依赖声明
    deps = [
        ":build_typescript",      # 先完成打包
        ":build_typescript_pack", # 先完成打包
    ]

    # 产物来源
    source = target_outputs_[0]  # tgz 包路径

    # 安装控制
    install_enable = false        # 不安装到系统目录

    # 组件信息
    part_name = "typescript"      # 组件名
    subsystem_name = "thirdparty" # 子系统名
}
```

**配置说明**：
- `install_enable = false` 表示该产物不直接安装到系统目录，而是作为工具链的一部分
- 产物通过 `developtools/ace_ets2bundle` 等模块间接使用

### 3.2.3 许可证收集配置

```gn
collect_notice("typescript_notice") {
    license_file = "LICENSE"  # 许可证文件
    module_source_dir = get_label_info(":${target_name}", "dir")

    # 输出到 SDK 的 ets-loader 目录
    outputs = [
        "${sdk_notice_dir}/ets/build-tools/ets-loader/node_modules/typescript.txt",
    ]
}
```

**输出位置说明**：`typescript.txt` 会被复制到 SDK 的 `ets/build-tools/ets-loader/node_modules/` 目录，作为应用开发时可见的许可证信息。

## 3.3 编译选项说明

### 3.3.1 源文件包含策略

TypeScript 的源文件包含遵循以下策略：

**核心文件**：所有 `lib/` 目录下的 .js 和 .d.ts 文件都包含，这些是 TypeScript 运行时的最小集合。

**本地化文件**：包含所有语言的诊断消息文件（`diagnosticMessages.*.json`），支持多语言错误提示。

**配置文件**：包含 `.gitattributes` 等配置文件，但不包含测试文件。

### 3.3.2 构建产物命名规范

**产物名称**：`ohos-typescript-{version}-r{revision}.tgz`

| 部分 | 说明 |
|-----|------|
| `ohos` | OpenHarmony 前缀 |
| `typescript` | 库名称 |
| `4.9.5` | TypeScript 版本号 |
| `r4` | OH 修订版本号 |
| `.tgz` | tar.gz 压缩格式 |

**修订版本号说明**：`r4` 表示 OH 对此版本进行了第 4 次修订，可能包含配置更新、补丁更新等。

## 3.4 与上游构建系统的差异

### 3.4.1 构建工具差异

| 维度 | 上游 TypeScript | OH 适配版 |
|-----|----------------|---------|
| **构建工具** | npm, gulp | Python, gn |
| **输出产物** | npm 包（.tgz） | 预构建产物（.tgz） |
| **编译过程** | JavaScript 编译 | 源文件打包 |
| **配置方式** | package.json | BUILD.gn |

### 3.4.2 核心差异说明

**上游构建**：

```json
// package.json
{
    "scripts": {
        "build": "npm run generate:lights",
        "package": "node bin/tsc",
        "publish": "npm publish"
    },
    "dependencies": {
        "typescript": "4.9.5"
    }
}
```

上游 TypeScript 使用 npm 管理依赖，使用 gulp 进行构建自动化，最终发布为 npm 包。

**OH 构建**：

```gn
# BUILD.gn
action("build_typescript_pack") {
    sources = typescript_sources
    script = "compile_typescript.py"
    args = [...]
    outputs = ["${target_out_dir}/ohos-typescript-4.9.5-r4.tgz"]
}
```

OH 适配版使用 Python 脚本将源文件打包，不进行实际编译（因为是 JavaScript 源代码）。

### 3.4.3 适配原因

OH 选择预构建方式的原因：

**运行时依赖**：TypeScript 是 JavaScript 项目，不需要像 C/C++ 项目那样编译为机器码。OH 只需要将源代码打包分发即可。

**简化构建**：避免在 OH 构建系统中处理 npm、node_modules 等复杂依赖，直接使用源码包。

**一致性**：与其他 JavaScript 工具链（如 eslint、prettier）的集成方式保持一致。

## 3.5 产物分发机制

### 3.5.1 构建产物路径

```
out/                    # 构建输出根目录
└── ohos/
    └── ohos-typescript-4.9.5-r4.tgz
```

### 3.5.2 SDK 分发路径

TypeScript 及其许可证会被分发到 SDK：

| 文件 | SDK 路径 |
|-----|---------|
| TypeScript tgz | `ets/build-tools/ets-loader/node_modules/` |
| 许可证 | `ets/build-tools/ets-loader/node_modules/typescript.txt` |

### 3.5.3 版本管理

OH 对 TypeScript 的版本管理遵循以下规范：

**版本号格式**：`<typescript-version>-r<oh-revision>`

- `typescript-version`：跟随上游 TypeScript 版本
- `oh-revision`：OH 适配修订版本，从 r1 开始

**升级策略**：
- 上游小版本升级（patch）：可能只更新修订版本号
- 上游大版本升级（minor/major）：需要评估兼容性，可能修改 major 版本号

## 3.6 集成方式

### 3.6.1 静态链接方式

TypeScript 在 OH 中主要通过**静态引用**方式使用：

```gn
# 引用预构建产物
ohos_prebuilt_etc("my_tool") {
    source = "${typescript_dir}/ohos-typescript-4.9.5-r4.tgz"
    # 提取并使用
}
```

### 3.6.2 动态调用方式

部分工具通过 npm script 调用 tsc：

```json
// package.json (在 ace_ets2bundle 中)
{
    "scripts": {
        "compile": "tsc --project tsconfig.json"
    },
    "dependencies": {
        "typescript": "file:path/to/ohos-typescript-4.9.5-r4.tgz"
    }
}
```

### 3.6.3 依赖传递

```
┌─────────────────────────────────────────────────────────┐
│                    ace_ets2bundle                        │
│                    (顶层依赖者)                           │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
              ┌─────────────────────────────┐
              │   @ohos/typescript          │
              │   (预构建产物)               │
              └─────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
    ┌─────────────────┐         ┌─────────────────┐
    │   es2panda      │         │   IDE 工具链     │
    │   (类型检查)     │         │   (语言服务)      │
    └─────────────────┘         └─────────────────┘
```

## 3.7 常见问题

### Q1：如何更新 TypeScript 版本？

**步骤**：
1. 在 `bundle.json` 中更新版本号
2. 修改 `BUILD.gn` 中的输出文件名
3. 测试所有依赖模块的兼容性
4. 更新许可证收集配置（如果需要）

### Q2：为什么使用 Python 脚本而不是 npm？

**原因**：
- OH 构建系统使用 GN，不直接支持 npm
- 避免在构建环境中引入 Node.js 依赖
- 简化许可证收集流程

### Q3：产物为什么是 tgz 而不是直接使用源码目录？

**原因**：
- 符合 OH 预构建产物的标准分发格式
- 便于版本管理和依赖声明
- 支持 GN 的 `ohos_prebuilt_etc` 模板
