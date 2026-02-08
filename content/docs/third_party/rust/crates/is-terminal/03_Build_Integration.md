# OH 构建适配

> is-terminal 使用 OpenHarmony 的 GN 构建系统进行编译，通过 `ohos_cargo_crate()` 模板将 Cargo 项目无缝集成。

---

## BUILD.gn 结构说明

### 文件位置
```
third_party/rust/crates/is-terminal/BUILD.gn
```

### 完整内容

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
    crate_name = "is_terminal"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.4.3"
    cargo_pkg_authors = "softprops <d.tangren@gmail.com>,  Dan Gohman <dev@sunfishcode.online>"
    cargo_pkg_name = "is-terminal"
    cargo_pkg_description = "Test whether a given stream is a terminal"
    deps = [
        "//third_party/rust/crates/io-lifetimes:lib",
        "//third_party/rust/crates/rustix:lib",
    ]
    module_output_extension = ".rlib"
    part_name = "rust_is_terminal"
    subsystem_name = "thirdparty"
}
```

### 配置字段说明

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | `"is_terminal"` | GN 目标名称（使用下划线） |
| `crate_type` | `"rlib"` | Rust 静态库（rlib）类型 |
| `crate_root` | `"src/lib.rs"` | 库入口文件 |
| `edition` | `"2018"` | Rust Edition |
| `cargo_pkg_version` | `"0.4.3"` | Cargo.toml 中的版本号 |
| `cargo_pkg_authors` | 原始作者 | 保留上游作者信息 |
| `cargo_pkg_name` | `"is-terminal"` | Cargo.toml 中的包名 |
| `cargo_pkg_description` | 功能描述 | 包描述信息 |
| `module_output_extension` | `".rlib"` | 输出文件扩展名 |
| `part_name` | `"rust_is_terminal"` | OH 部件名称 |
| `subsystem_name` | `"thirdparty"` | OH 子系统名称 |

---

## 关键编译选项

### 依赖配置

```gn
deps = [
    "//third_party/rust/crates/io-lifetimes:lib",
    "//third_party/rust/crates/rustix:lib",
]
```

**依赖映射说明**:

| 上游依赖 (Cargo.toml) | OH 映射 (BUILD.gn) | 用途 |
|----------------------|-------------------|------|
| `io-lifetimes = "1.0.0"` | `//third_party/rust/crates/io-lifetimes:lib` | I/O 资源抽象 |
| `rustix = "0.36.4"` | `//third_party/rust/crates/rustix:lib` | 系统调用封装 |

**平台条件依赖**: BUILD.gn 中的 `deps` 包含了所有平台的依赖，平台选择由 Rust 的 `#[cfg()]` 属性在编译时自动处理。

### 条件编译

is-terminal 使用 Rust 标准的条件编译机制，BUILD.gn 无需特殊配置：

```rust
// Unix 平台
#[cfg(any(unix, target_os = "wasi"))]
rustix::termios::isatty(self)

// Windows 平台
#[cfg(windows)]
_is_terminal(self.as_filelike())

// HermitOS
#[cfg(target_os = "hermit")]
hermit_abi::isatty(self.as_filelike().as_fd())
```

GN 构建系统会根据目标平台自动选择正确的实现分支。

---

## 与上游构建系统的差异

### 构建系统对比

| 维度 | 上游 (Cargo) | OpenHarmony (GN) |
|------|--------------|------------------|
| **构建工具** | `cargo build` | `ohos_cargo_crate()` |
| **配置文件** | `Cargo.toml` | `BUILD.gn` |
| **目标类型** | `lib` (rlib) | `rlib` |
| **依赖解析** | 自动（Cargo） | 手动映射 deps |
| **条件编译** | `#[cfg()]` | `#[cfg()]` (相同) |
| **构建脚本** | `build.rs` (is-terminal 无) | N/A |
| **Feature flags** | `[features]` | N/A (未使用) |
| **测试** | `cargo test` | `ohos_rust_test()` (未配置) |

### 关键差异说明

1. **依赖映射**: BUILD.gn 需要手动列出所有依赖，而 Cargo 自动解析依赖树
2. **版本管理**: BUILD.gn 通过 `cargo_pkg_version` 声明版本，与 Cargo.toml 同步
3. **无 Feature Flags**: is-terminal 未使用 Cargo features，GN 无需额外配置
4. **无构建脚本**: is-terminal 没有 `build.rs`，GN 配置更简单

### 构建输出

| 环境 | 命令 | 输出路径 |
|------|------|---------|
| **上游** | `cargo build --release` | `target/release/libis_terminal-*.rlib` |
| **OH** | `hb build -f` | `out/ohos-arm64/.../libis_terminal.rlib` |

---

## 特殊处理

### 无特殊特性禁用

is-terminal **没有禁用任何上游特性**，所有功能都保留：

- ✅ Unix `isatty()` 支持
- ✅ Windows 终端检测
- ✅ WASI 支持
- ✅ HermitOS 支持
- ✅ `target_os = "unknown"` 简化实现

### 无 OH 特定源文件

OH 没有添加任何自定义源文件或适配层：

- ❌ 没有 `.ohos` 目录
- ❌ 没有 `ohos_*.rs` 文件
- ❌ 没有 OH 特定宏定义

### Apache 2.0 版权头部

BUILD.gn 使用 OH 标准的 Apache 2.0 版权声明：

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0
```

这符合 OpenHarmony 项目的代码规范，与上游的 MIT 许可证并存。

---

## bundle.json 组件配置

### 文件位置
```
third_party/rust/crates/is-terminal/bundle.json
```

### 完整内容

```json
{
  "name": "@ohos/rust_is_terminal",
  "description": "A Rust library that provides support for detecting whether a terminal is present.",
  "version": "6.1",
  "license": "MIT License",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/is-terminal"
  },
  "dirs": {},
  "scripts": {},
  "readmePath": {
    "en": "README.md"
  },
  "component": {
    "name": "rust_is_terminal",
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
          "name": "//third_party/rust/crates/is-terminal:lib"
        }
      ],
      "test": []
    }
  }
}
```

### 关键字段说明

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | `@ohos/rust_is_terminal` | OH 组件全名 |
| `version` | `6.1` | OH 组件版本（独立于库版本） |
| `publishAs` | `code-segment` | 代码段类型 |
| `component.name` | `rust_is_terminal` | 部件名称 |
| `component.subsystem` | `thirdparty` | 所属子系统 |
| `component.adapted_system_type` | `["standard"]` | 适配标准系统 |
| `component.inner_kits` | `//...:lib` | 内部 API 提供者 |

---

## 构建调试

### 常见构建命令

```bash
# 构建 is-terminal
hb build -f //third_party/rust/crates/is-terminal:lib

# 查看依赖
hb analyze //third_party/rust/crates/is-terminal:lib

# 清理构建
hb clean
```

### 构建问题排查

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| `cargo_pkg_version` 不匹配 | Cargo.toml 更新但 BUILD.gn 未同步 | 同步 BUILD.gn 中的版本号 |
| 依赖路径错误 | OH 内部 crate 路径变更 | 检查 deps 中的路径是否正确 |
| 条件编译不生效 | 目标平台配置错误 | 检查 `target_os` 配置 |

---

## 参考资源

- **GN 模板文档**: `//build/ohos.gni` 中的 `ohos_cargo_crate()` 定义
- **BUILD.gn 文件**: `third_party/rust/crates/is-terminal/BUILD.gn`
- **bundle.json 文件**: `third_party/rust/crates/is-terminal/bundle.json`
- **上游 Cargo.toml**: 项目根目录 `Cargo.toml`
