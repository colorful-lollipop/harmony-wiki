# OH 构建适配

> env_logger 在 OpenHarmony 构建系统中的集成配置

---

## 概述

env_logger 通过 GN 构建系统集成到 OpenHarmony 中。构建配置遵循 OH 的 Rust crates 集成标准。

**集成特点**：
- ✅ 使用 `ohos_cargo_crate` 模板
- ✅ 输出为 Rust 静态库（`.rlib`）
- ✅ 启用了常用 features（auto-color、humantime、regex）
- ❌ 无 OH 特定编译选项或宏定义

---

## BUILD.gn 完整配置

### 配置文件

**路径**：`/Volumes/lexar/code/d/work/oh/third_party/rust/crates/env_logger/BUILD.gn`

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "env_logger"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2021"
    cargo_pkg_version = "0.10.2"
    cargo_pkg_name = "env_logger"
    deps = [
        "//third_party/rust/crates/is-terminal:lib",
        "//third_party/rust/crates/humantime:lib",
        "//third_party/rust/crates/log:lib",
        "//third_party/rust/crates/regex:lib",
        "//third_party/rust/crates/termcolor:lib",
    ]
    features = [
        "auto-color",
        "humantime",
        "regex",
    ]
    module_output_extension = ".rlib"
    part_name = "rust_env_logger"
    subsystem_name = "thirdparty"
}
```

---

## 配置详解

### 1. Crate 基本信息

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | `env_logger` | Crate 名称 |
| `crate_type` | `rlib` | Rust 静态库 |
| `crate_root` | `src/lib.rs` | 入口文件 |
| `edition` | `2021` | Rust 2021 Edition |
| `cargo_pkg_version` | `0.10.2` | Cargo.toml 版本号 |
| `cargo_pkg_name` | `env_logger` | Cargo.toml 包名 |
| `sources` | `["src/lib.rs"]` | 源文件列表 |

**关键点**：
- `crate_type = "rlib"`：编译为 Rust 静态库，不包含主函数
- `edition = "2021"`：使用 Rust 2021 Edition

### 2. 依赖配置

```gn
deps = [
    "//third_party/rust/crates/is-terminal:lib",
    "//third_party/rust/crates/humantime:lib",
    "//third_party/rust/crates/log:lib",
    "//third_party/rust/crates/regex:lib",
    "//third_party/rust/crates/termcolor:lib",
]
```

| 依赖 | 版本 | 类型 | 用途 |
|------|------|------|------|
| `log` | 0.4.8+ | 必需 | Rust 日志门面（facade） |
| `is-terminal` | 0.4.0+ | 可选 | 终端检测（auto-color 功能） |
| `humantime` | 2.0.0+ | 可选 | 人类可读时间格式 |
| `regex` | 1.0.3+ | 可选 | 正则表达式日志过滤 |
| `termcolor` | 1.1.1+ | 可选 | 终端颜色输出 |

**依赖树**：

```
env_logger
├── log (必需)
├── is-terminal ─┐
│                ├──
│                └── termcolor (间接)
├── humantime
└── regex
```

### 3. Features 配置

```gn
features = [
    "auto-color",
    "humantime",
    "regex",
]
```

#### Feature 说明

| Feature | 默认 | OH 启用 | 说明 |
|---------|------|---------|------|
| `auto-color` | ✅ | ✅ | 自动检测终端并启用彩色输出 |
| `humantime` | ✅ | ✅ | 使用人类可读时间格式 |
| `regex` | ✅ | ✅ | 启用正则表达式日志过滤 |
| `color` | ❌ | ❌ | 手动颜色控制（被 auto-color 替代） |

#### 未启用的 Features

| Feature | 原因 |
|---------|------|
| `color` | 已通过 `auto-color` 间接启用 |
| 无其他 feature | env_logger 无其他可选功能 |

#### Feature 影响

**`auto-color`**：
- 自动检测是否为终端环境
- 终端环境：启用彩色输出
- 非终端环境（如日志文件）：禁用彩色输出

**`humantime`**：
- 时间戳格式示例：
  - 有 humantime：`3s ago`、`2min ago`
  - 无 humantime：`2024-01-18T10:30:00Z`

**`regex`**：
- 支持正则表达式过滤日志
- 示例：`RUST_LOG=my_crate/.*error.*/info`

### 4. 构建输出配置

```gn
module_output_extension = ".rlib"
```

- 输出文件扩展名：`.rlib`（Rust 库文件）
- 完整输出路径示例：
  ```
  out/rust/armeabi-v7a/obj/third_party/rust/crates/env_logger/libenv_logger-<hash>.rlib
  ```

### 5. OH 系统配置

```gn
part_name = "rust_env_logger"
subsystem_name = "thirdparty"
```

- `part_name`：部件名称（用于组件管理）
- `subsystem_name`：子系统名称（归为 thirdparty）

---

## bundle.json 配置

### 配置文件

**路径**：`/Volumes/lexar/code/d/work/oh/third_party/rust/crates/env_logger/bundle.json`

```json
{
  "name": "@ohos/rust_env_logger",
  "description": "A library that generates Rust code based on compile-time configuration options",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/env_logger"
  },
  "dirs": {},
  "scripts": {},
  "readmePath": {
    "en": "README.md"
  },
  "component": {
    "name": "rust_env_logger",
    "subsystem": "thirdparty",
    "adapted_system_type": [
      "standard"
    ],
    "deps": {
      "components": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/env_logger:lib"
        }
      ],
      "test": []
    }
  }
}
```

### 配置说明

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | `@ohos/rust_env_logger` | OH 包名 |
| `description` | "A library that generates Rust code..." | ⚠️ 描述不准确 |
| `version` | `6.1` | OH 组件版本 |
| `license` | "Apache License 2.0" | 许可证 |
| `publishAs` | `code-segment` | 发布方式 |
| `segment.destPath` | `third_party/rust/crates/env_logger` | 代码路径 |

### 内部 kits

```json
"inner_kits": [
  {
    "name": "//third_party/rust/crates/env_logger:lib"
  }
]
```

- 暴露的库目标：`//third_party/rust/crates/env_logger:lib`
- 其他模块可以通过此路径依赖 env_logger

---

## 构建系统差异

### 与上游构建系统的差异

| 维度 | 上游（Cargo） | OH（GN） | 说明 |
|------|-------------|----------|------|
| 构建工具 | `cargo build` | `gn gen && ninja` | 使用 GN 编译 |
| 配置文件 | `Cargo.toml` | `BUILD.gn` | BUILD.gn 为 OH 适配 |
| 输出格式 | `.rlib`、`.dylib` | `.rlib` | 仅输出 rlib |
| 依赖管理 | Cargo 自动解析 | GN 手动指定 | GN 需明确列出 deps |
| Features | 通过命令行启用 | 在 BUILD.gn 中声明 | 功能配置方式不同 |

### Cargo.toml vs BUILD.gn 对应关系

#### Cargo.toml

```toml
[package]
name = "env_logger"
version = "0.10.2"
edition = "2021"

[features]
default = ["auto-color", "humantime", "regex"]
color = ["dep:termcolor"]
auto-color = ["dep:is-terminal", "color"]
humantime = ["dep:humantime"]
regex = ["dep:regex"]

[dependencies]
log = { version = "0.4.8", features = ["std"] }
regex = { version = "1.0.3", optional = true }
termcolor = { version = "1.1.1", optional = true }
humantime = { version = "2.0.0", optional = true }
is-terminal = { version = "0.4.0", optional = true }
```

#### BUILD.gn

```gn
ohos_cargo_crate("lib") {
    crate_name = "env_logger"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    edition = "2021"
    cargo_pkg_version = "0.10.2"
    cargo_pkg_name = "env_logger"

    deps = [
        "//third_party/rust/crates/log:lib",
        "//third_party/rust/crates/regex:lib",
        "//third_party/rust/crates/termcolor:lib",
        "//third_party/rust/crates/humantime:lib",
        "//third_party/rust/crates/is-terminal:lib",
    ]

    features = [
        "auto-color",
        "humantime",
        "regex",
    ]
}
```

**映射关系**：

| Cargo.toml | BUILD.gn | 说明 |
|------------|----------|------|
| `[package]` 基本信息 | `crate_name`、`crate_type`、`edition`、`cargo_pkg_version` | 基本信息对应 |
| `[dependencies]` | `deps` | 依赖映射 |
| `[features]` | `features` | 功能开关映射 |

---

## 特殊处理

### OH 特定配置

env_logger 在 OH 构建中**无特殊处理**：

| 配置项 | 是否有 OH 特定配置 | 说明 |
|--------|------------------|------|
| Defines | ❌ 无 | 无 OH 特定宏定义 |
| Configs | ❌ 无 | 无额外配置选项 |
| Flags | ❌ 无 | 无编译标志 |
| 源文件修改 | ❌ 无 | 源代码未修改 |
| 测试配置 | ❌ 无 | 测试配置与上游一致 |

### 与上游的一致性

✅ **完全一致**：
- 源代码：完全同步上游
- Features：启用相同的默认 features
- 依赖：版本与上游一致

⚠️ **构建工具不同**：
- 构建系统：GN vs Cargo
- 依赖管理：手动 vs 自动

---

## 使用指南

### 在 BUILD.gn 中添加依赖

#### 方法一：直接依赖

```gn
ohos_rust_executable("my_app") {
    sources = ["src/main.rs"]
    edition = "2021"

    deps = [
        "//third_party/rust/crates/env_logger:lib",
        "//third_party/rust/crates/log:lib",  # 同时依赖 log
    ]

    features = []
}
```

#### 方法二：通过 inner_kits

如果 env_logger 已在 bundle.json 的 inner_kits 中声明：
```gn
# 在 bundle.json 中
"inner_kits": [
  {
    "name": "//third_party/rust/crates/env_logger:lib"
  }
]

# 在依赖模块的 bundle.json 中
"deps": {
  "components": [
    "rust_env_logger"
  ]
}
```

### 代码示例

```rust
use log::{info, debug, error};

fn main() {
    // 初始化 logger（从环境变量读取配置）
    env_logger::init();

    info!("应用程序启动");
    debug!("调试信息");
    error!("发生错误！");
}
```

### 环境变量配置

在运行时通过 `RUST_LOG` 环境变量控制日志：

```bash
# 启用 info 级别日志
export RUST_LOG=info

# 启用特定模块的 debug 日志
export RUST_LOG=my_crate=debug

# 组合配置
export RUST_LOG=error,my_crate=debug
```

---

## 编译选项详解

### 默认编译选项

`ohos_cargo_crate` 模板提供的默认选项：

| 选项 | 值 | 说明 |
|------|-----|------|
| `strip` | true | 符号表剥离（减小二进制大小） |
| `opt_level` | 2 | 优化级别 |
| `debug` | false | 不包含调试信息 |

### 自定义编译选项

如需自定义编译选项，可覆盖默认值：

```gn
ohos_cargo_crate("lib") {
    # ... 其他配置

    # 覆盖默认选项
    strip = false  # 保留符号表
    opt_level = 3  # 最高优化级别
    debug = true   # 包含调试信息
}
```

### 编译输出

#### 编译产物

```
out/
└── rust/
    └── <arch>/
        └── obj/
            └── third_party/
                └── rust/
                    └── crates/
                        └── env_logger/
                            └── libenv_logger-<hash>.rlib
```

#### 依赖分析

查看编译依赖：
```bash
ninja -C out/rust/<arch> -t deps third_party/rust/crates/env_logger:lib
```

---

## 故障排查

### 常见问题

#### 1. 找不到 crate

**错误**：
```
error: failed to resolve: could not find `env_logger` in `https://crates.io/...`
```

**原因**：依赖路径不正确

**解决**：
- 检查 `deps` 中路径是否正确
- 确认 env_logger 的 BUILD.gn 配置正确

#### 2. Feature 未生效

**错误**：正则表达式过滤不工作

**原因**：`regex` feature 未启用

**解决**：
```gn
features = [
    "regex",  # 确保启用 regex feature
]
```

#### 3. 日志输出没有颜色

**现象**：日志输出没有彩色

**原因**：`auto-color` feature 未启用，或输出到非终端

**解决**：
```gn
features = [
    "auto-color",  # 确保启用 auto-color feature
]
```

### 调试建议

#### 查看编译命令

```bash
gn gen out/rust/<arch> --args="rust_toolchain=\"<path>\""
ninja -C out/rust/<arch> -v third_party/rust/crates/env_logger:lib
```

#### 查看 rlib 内容

```bash
ar -t out/rust/<arch>/obj/third_party/rust/crates/env_logger/libenv_logger-*.rlib
```

#### 验证 features

查看编译后的 crate 元数据：
```bash
rustc --print crate-name --crate-type rlib src/lib.rs
```

---

## 总结

### 关键配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| Crate 类型 | `rlib` | Rust 静态库 |
| Edition | `2021` | Rust 2021 |
| Features | `auto-color`, `humantime`, `regex` | 启用常用功能 |
| 输出 | `.rlib` | Rust 库文件 |

### 使用建议

1. **默认配置即可**：OH 的 BUILD.gn 配置已足够用于大多数场景
2. **启用所有 features**：建议保持默认的 features 配置
3. **同步版本**：升级时注意依赖 crates 的版本兼容性
4. **测试验证**：升级后测试日志输出是否正常

### 维护要点

| 任务 | 频率 | 说明 |
|------|------|------|
| 跟随上游更新 | 每次发版 | 检查新版本 |
| 更新版本号 | 每次升级 | 修改 `cargo_pkg_version` |
| 验证功能 | 每次升级 | 测试日志输出 |
| 审计依赖 | 定期 | 检查依赖 crates 的安全性 |

---

**文档版本**：1.0
**更新时间**：2026-02-08
**评估版本**：env_logger v0.10.2
