# OH 构建适配

## 3.1 BUILD.gn 配置详解

### 构建配置概述

OpenHarmony 通过 `BUILD.gn` 文件将 rustix 集成到构建系统中。以下是完整的构建配置：

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

if (host_os != "linux" || host_cpu != "arm64") {
  ohos_cargo_crate("lib") {
    crate_name = "rustix"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = [ "src/lib.rs" ]
    edition = "2018"
    cargo_pkg_version = "0.36.16"
    cargo_pkg_authors = "Dan Gohman <dev@sunfishcode.online>,  Jakub Konka <kubkon@jakubkonka.com>"
    cargo_pkg_name = "rustix"
    cargo_pkg_description =
        "Safe Rust bindings to POSIX/Unix/Linux/Winsock2-like syscalls"
    deps = [
      "//third_party/rust/crates/bitflags:lib",
      "//third_party/rust/crates/io-lifetimes:lib",
      "//third_party/rust/crates/libc:lib",
      "//third_party/rust/crates/linux-raw-sys:lib",
    ]
    features = [
      "io-lifetimes",
      "libc",
      "std",
      "use-libc-auxv",
      "termios",
    ]
    rustenv = [ string_join("", [ "CARGO_CFG_TARGET_OS=linux" ]) ]
    rustenv += [ string_join("", [ "CARGO_CFG_TARGET_ARCH=x86_64" ]) ]
    rustenv += [ string_join("", [ "CARGO_CFG_TARGET_POINTER_WIDTH=64" ]) ]
    rustenv += [ string_join("", [ "CARGO_CFG_TARGET_ENDIAN=little" ]) ]
    build_root = "build.rs"
    build_sources = [ "build.rs" ]
    build_script_outputs = [ "librust_out.rmeta" ]
    module_output_extension = ".rlib"
    part_name = "rust_rustix"
    subsystem_name = "thirdparty"
  }
}
```

### 配置字段详解

| 字段 | 值 | 说明 |
|------|-----|------|
| `crate_name` | "rustix" | Cargo 包名称 |
| `crate_type` | "rlib" | 静态库类型 |
| `crate_root` | "src/lib.rs" | 包入口文件 |
| `edition` | "2018" | Rust Edition |
| `part_name` | "rust_rustix" | OH 组件名 |
| `subsystem_name` | "thirdparty" | 所属子系统 |

### 条件构建逻辑

```gn
if (host_os != "linux" || host_cpu != "arm64") {
  # 仅在此条件下构建
}
```

此条件表明 rustix 的 OH 构建仅在非 Linux arm64 环境下执行，可能用于：

- **交叉编译场景**：在 x86_64 Linux 上为 OH arm64 编译
- **开发环境**：开发机的编译验证
- **CI 构建**：持续集成环境中的构建验证

## 3.2 编译选项详解

### 依赖配置

```gn
deps = [
  "//third_party/rust/crates/bitflags:lib",
  "//third_party/rust/crates/io-lifetimes:lib",
  "//third_party/rust/crates/libc:lib",
  "//third_party/rust/crates/linux-raw-sys:lib",
]
```

| 依赖 | OH 组件 | 用途 |
|------|---------|------|
| bitflags | rust_bitflags | 位标志的安全类型封装 |
| io-lifetimes | rust_io_lifetimes | 文件描述符生命周期管理 |
| libc | rust_libc | C 标准库绑定 |
| linux-raw-sys | rust_linux_raw_sys | Linux 内核 ABI 定义 |

### Feature 配置

```gn
features = [
  "io-lifetimes",
  "libc",
  "std",
  "use-libc-auxv",
  "termios",
]
```

| Feature | 状态 | 用途 |
|---------|------|------|
| `io-lifetimes` | 启用 | 文件描述符生命周期管理 |
| `libc` | 启用 | **强制使用 libc 后端** |
| `std` | 启用 | 标准库支持 |
| `use-libc-auxv` | 启用 | 使用 libc 读取 auxv |
| `termios` | 启用 | 终端 I/O 操作 |

### 环境变量配置

```gn
rustenv = [
  "CARGO_CFG_TARGET_OS=linux",
  "CARGO_CFG_TARGET_ARCH=x86_64",
  "CARGO_CFG_TARGET_POINTER_WIDTH=64",
  "CARGO_CFG_TARGET_ENDIAN=little",
]
```

| 环境变量 | 值 | 说明 |
|---------|-----|------|
| `CARGO_CFG_TARGET_OS` | linux | **伪装为 Linux** |
| `CARGO_CFG_TARGET_ARCH` | x86_64 | 架构设置 |
| `CARGO_CFG_TARGET_POINTER_WIDTH` | 64 | 指针宽度 |
| `CARGO_CFG_TARGET_ENDIAN` | little | 字节序 |

## 3.3 与上游构建的差异

### 构建系统差异

| 维度 | 上游 | OpenHarmony |
|------|------|-------------|
| **构建系统** | Cargo | GN + Cargo |
| **包管理器** | crates.io | OH thirdparty |
| **构建配置** | Cargo.toml | BUILD.gn |
| **条件编译** | Cargo features | GN 条件 + features |

### 功能配置差异

| Feature | 上游默认 | OH 配置 | 差异说明 |
|---------|---------|---------|----------|
| `std` | ✓ 默认 | ✓ 启用 | 一致 |
| `use-libc` | ✗ 可选 | ✓ 强制 | **OH 强制使用 libc** |
| `use-libc-auxv` | ✓ 默认 | ✓ 启用 | 一致 |
| `fs` | ✗ 可选 | ✗ 未启用 | OH 未启用 |
| `net` | ✗ 可选 | ✗ 未启用 | OH 未启用 |
| `process` | ✗ 可选 | ✗ 未启用 | OH 未启用 |
| `io_uring` | ✗ 可选 | ✗ 未启用 | OH 未启用 |

### 平台条件差异

**上游 Cargo.toml**:
```toml
[target.'cfg(all(not(rustix_use_libc), target_os = "linux", any(target_arch = "x86", ...)))'.dependencies]
linux-raw-sys = "0.1.2"
```

**OH BUILD.gn**:
```gn
rustenv = [
  "CARGO_CFG_TARGET_OS=linux",
  "CARGO_CFG_TARGET_ARCH=x86_64",
]
features = ["libc", ...]  # 强制 libc 后端
```

**差异说明**：
- 上游会根据目标平台自动选择后端
- OH 通过强制设置环境和启用 `libc` feature，确保始终使用 libc 后端

## 3.4 特殊构建处理

### 构建脚本

```gn
build_root = "build.rs"
build_sources = [ "build.rs" ]
build_script_outputs = [ "librust_out.rmeta" ]
```

`build.rs` 是 Rust 的构建脚本，用于：

- 检测目标平台特性
- 生成代码或配置文件
- 执行平台相关的构建逻辑

### 输出配置

```gn
module_output_extension = ".rlib"
build_script_outputs = [ "librust_out.rmeta" ]
```

- **输出类型**：`.rlib` 静态库
- **元数据文件**：`.rmeta` 包含库的元信息

## 3.5 OH 构建优化建议

### 当前状态

| 维度 | 评估 |
|------|------|
| **后端选择** | 安全但性能次优 |
| **功能启用** | 基础功能，完整功能未启用 |
| **条件构建** | 有限制的构建条件 |

### 潜在优化

#### 1. 根据需要启用更多 Features

```gn
features = [
  "io-lifetimes",
  "libc",
  "std",
  "use-libc-auxv",
  "termios",
  # 可选启用：
  # "fs",        # 文件系统操作
  # "net",       # 网络操作
  # "process",   # 进程操作
  # "thread",    # 线程操作
]
```

#### 2. 平台感知构建

当前构建条件较为简单，可以考虑：

```gn
# 根据实际目标平台调整
if (ohos_target_cpu == "arm64") {
  rustenv += [ "CARGO_CFG_TARGET_ARCH=aarch64" ]
} else if (ohos_target_cpu == "x86_64") {
  rustenv += [ "CARGO_CFG_TARGET_ARCH=x86_64" ]
}
```

#### 3. 测试和基准

建议添加：

- 构建正确性验证
- 性能基准测试
- 与上游的行为对比测试

## 3.6 版本管理

### 版本映射

| 维度 | 版本 |
|------|------|
| **上游 rustix** | 0.36.16 |
| **OH bundle.json** | 6.1 |
| **上游 Cargo.toml** | 0.36.16 |

### 版本升级流程

1. **检查上游更新**：访问 https://github.com/bytecodealliance/rustix/releases
2. **验证兼容性**：确认新版本与 OH 依赖（libc、io-lifetimes）兼容
3. **更新配置**：修改 bundle.json 版本号
4. **测试验证**：运行相关测试确保功能正常
5. **Patch 评估**：检查 Patch 是否仍适用或需要新增

---

## 3.7 故障排查

### 常见问题

#### 1. 构建失败：找不到依赖

**症状**：
```
error: failed to resolve dependencies for package rustix
```

**解决**：
```bash
# 确保依赖的 OH crates 已构建
hb set
hb build //third_party/rust/crates/bitflags:lib
hb build //third_party/rust/crates/io-lifetimes:lib
hb build //third_party/rust/crates/libc:lib
```

#### 2. 条件编译问题

**症状**：
```
error[E0433]: use of undeclared type or module `xyz`
```

**解决**：检查 `features` 配置是否包含所需的模块

#### 3. 架构不匹配

**症状**：
```
error: this crate is being compiled for an unsupported architecture
```

**解决**：检查 `rustenv` 中的架构设置
