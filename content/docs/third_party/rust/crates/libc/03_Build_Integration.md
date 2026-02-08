# 03_Build_Integration - OH 构建适配

> **文档状态**: ✅ 完成
> **最后更新**: 2026-02-08

---

## 3.1 构建系统概述

### 构建方式

libc 在 OpenHarmony 中使用 **GN (Generate Ninja)** 构建系统，通过 `ohos_cargo_crate` 模板集成。

**关键特点**：
- 使用 GN 作为顶层构建系统
- 通过 `ohos_cargo_crate` 封装 Cargo 构建过程
- 支持 Rust 2015 edition
- 输出为 Rust 静态库（`.rlib`）

### 构建文件位置

```
third_party/rust/crates/libc/
├── BUILD.gn              # GN 构建配置
├── Cargo.toml            # Cargo 包配置
├── build.rs              # Cargo 构建脚本
└── src/
    └── lib.rs            # Rust crate 根文件
```

---

## 3.2 BUILD.gn 详细说明

### 完整配置

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
  crate_name = "libc"
  crate_type = "rlib"
  crate_root = "src/lib.rs"
  output_name = "liblibc"

  sources = [ "src/lib.rs" ]
  edition = "2015"
  cargo_pkg_version = "0.2.153"
  cargo_pkg_authors = "The Rust Project Developers"
  cargo_pkg_name = "libc"
  features = [
    "default",
    "extra_traits",
    "std",
  ]
  build_root = "build.rs"
  build_sources = [ "build.rs" ]
  module_output_extension = ".rlib"
  part_name = "rust_libc"
  subsystem_name = "thirdparty"
}
```

### 配置项详解

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **crate_name** | `"libc"` | Rust crate 的名称 |
| **crate_type** | `"rlib"` | 输出类型：Rust 静态库（不产生独立二进制）|
| **crate_root** | `"src/lib.rs"` | Rust crate 的根文件 |
| **output_name** | `"liblibc"` | 输出文件名 |
| **edition** | `"2015"` | Rust edition（Rust 2015）|
| **cargo_pkg_version** | `"0.2.153"` | Cargo 包版本 |
| **features** | `["default", "extra_traits", "std"]` | 启用的特性（见下文详细说明）|
| **build_root** | `"build.rs"` | Cargo 构建脚本 |
| **module_output_extension** | `".rlib"` | 输出文件扩展名 |
| **part_name** | `"rust_libc"` | OHOS 部件名称 |
| **subsystem_name** | `"thirdparty"` | OHOS 子系统名称 |

### 特性（Features）说明

#### default
```toml
default = ["std"]
```
- **作用**: 启用默认特性
- **默认包含**: `std` 特性（链接 Rust 标准库）
- **禁用场景**: 在 `#![no_std]` 环境中需要禁用此特性

#### std
```toml
std = []
```
- **作用**: 链接 Rust 标准库
- **影响**: 允许使用 `std` crate 中的类型和函数
- **禁用效果**: 仅使用 `core` 和 `alloc` crate，适合嵌入式或 `no_std` 环境

#### extra_traits
```toml
extra_traits = []
```
- **作用**: 为所有 `struct` 派生额外的 trait（`Debug`, `Eq`, `Hash`, `PartialEq`）
- **默认行为**: 所有 `struct` 实现了 `Copy` 和 `Clone`
- **启用效果**: 增加以下 trait：
  - `Debug`: 支持格式化输出（`println!("{:?}", struct)`）
  - `Eq`: 支持等值比较（`==`）
  - `PartialEq`: 支持部分等值比较
  - `Hash`: 支持作为哈希表键值
- **权衡**: 增加编译时间和代码体积，但提供更好的调试和开发体验

#### OHOS 启用的特性组合

```gn
features = [
    "default",        # 包含 std
    "extra_traits",  # 额外的调试和比较 trait
    "std",           # 显式启用 std
]
```

**选择理由**：
- `std`: OHOS 是完整的操作系统，不需要 `no_std` 限制
- `extra_traits`: 提供更好的调试能力，便于开发和问题定位
- `default`: 保持与上游默认配置一致

---

## 3.3 build.rs 功能说明

### 主要功能

`build.rs` 是 Cargo 的构建脚本，在编译 crate 之前运行，用于：
1. 检测编译器和平台特性
2. 设置条件编译标志（`cfg`）
3. 生成编译配置

### 关键功能模块

#### A. Rust 版本检测

```rust
fn rustc_minor_nightly() -> (u32, bool) {
    // 检测 rustc 的版本号和是否为 nightly 版本
    // 返回 (minor_version, is_nightly)
}
```

**检测用途**：
- 根据 Rust 版本启用或禁用某些特性
- 支持 Rust 1.13.0 及以上版本
- 为不同版本的 Rust 提供向后兼容性

#### B. 版本特性开关

| Rust 版本 | 设置的 cfg | 启用的特性 |
|-----------|------------|------------|
| >= 1.15 | `libc_priv_mod_use` | 私有模块 `use` 支持 |
| >= 1.19 | `libc_union` | `union` 类型支持 |
| >= 1.24 | `libc_const_size_of` | `const mem::size_of` |
| >= 1.25 | `libc_align` | `#[repr(align)]` 支持 |
| >= 1.26 | `libc_int128` | `i128` / `u128` 类型支持 |
| >= 1.30 | `libc_core_cvoid` | `core::ffi::c_void` |
| >= 1.33 | `libc_packedN`, `libc_cfg_target_vendor` | `#[repr(packed(N))]` 和 `cfg(target_vendor)` |
| >= 1.40 | `libc_non_exhaustive` | `#[non_exhaustive]` |
| >= 1.47 | `libc_long_array` | 长数组支持 |
| >= 1.62 | `libc_const_extern_fn` | `const extern fn` |

#### C. 平台特定检测

```rust
// FreeBSD 版本检测
fn which_freebsd() -> Option<i32> {
    // 检测 FreeBSD 版本（10, 11, 12, 13, 14, 15）
    // 返回版本号或 None
}

// Emscripten 版本检测
fn emcc_version_code() -> Option<u64> {
    // 检测 Emscripten 版本
    // 返回版本码或 None
}
```

**检测结果**：
- FreeBSD 版本：设置 `freebsd10` / `freebsd11` / ... / `freebsd15`
- Emscripten 版本 >= 3.1.42：设置 `emscripten_new_stat_abi`

#### D. OpenHarmony 支持

```rust
const CHECK_CFG_EXTRA: &'static [(&'static str, &'static [&'static str])] = &[
    ("target_os", &["switch", "aix", "hurd", "ohos"]),      // OHOS 作为合法的 target_os
    ("target_env", &["illumos", "wasi", "aix", "ohos"]),    // OHOS 作为合法的 target_env
    ("target_arch", &["loongarch64", "mips32r6", "mips64r6", "csky"]),
];
```

**说明**：
- `target_os = "ohos"` 和 `target_env = "ohos"` 被列为合法的编译目标
- 这意味着 libc crate **原生支持** OpenHarmony 平台
- 在编译 OHOS target 时，这些配置会被激活

#### E. CI 支持

```rust
// CI 环境禁止警告
if libc_ci {
    set_cfg("libc_deny_warnings");
}
```

**触发条件**：设置环境变量 `LIBC_CI=1`

**效果**：将所有编译警告视为错误，确保代码质量

---

## 3.4 与上游构建系统的差异

### Cargo.toml vs BUILD.gn

| 方面 | Cargo.toml（上游）| BUILD.gn（OHOS）|
|------|------------------|----------------|
| **构建工具** | Cargo | GN + Cargo（通过 ohos_cargo_crate）|
| **版本** | 0.2.153 | 0.2.153 |
| **edition** | 2015 | 2015（一致）|
| **特性** | 默认（std）| std + extra_traits（增强）|
| **目标平台** | 多平台 | OpenHarmony 特定 |

### OHOS 特定差异

#### 1. 特性增强
```toml
# 上游（Cargo.toml）
[features]
default = ["std"]
std = []
extra_traits = []

# OHOS（BUILD.gn）
features = [
    "default",        # 包含 std
    "extra_traits",  # 额外的 trait（上游可选，OHOS 启用）
    "std",           # 显式启用 std
]
```

**影响**：
- OHOS 版本额外启用了 `extra_traits`，提供更好的调试能力
- 其他特性与上游保持一致

#### 2. 构建系统封装
```gn
ohos_cargo_crate("lib") {
    # ... GN 配置 ...
}
```

**说明**：
- OHOS 使用 `ohos_cargo_crate` 模板封装 Cargo 构建过程
- 这允许在 GN 构建系统中集成 Rust crates
- 保持与 Cargo 生态系统的兼容性

#### 3. 元数据
```json
// bundle.json
{
  "name": "@ohos/rust_libc",
  "version": "5.0",
  "subsystem": "thirdparty",
  "component": {
    "name": "rust_libc"
  }
}
```

**说明**：
- OHOS 组件使用 `@ohos/rust_libc` 命名
- 版本号使用 OHOS 系统版本（5.0），而非 Cargo 版本
- 隶属于 `thirdparty` 子系统

---

## 3.5 特殊处理

### 1. 禁用的功能

OpenHarmony 不支持以下 POSIX 功能（通过条件编译排除）：

| 功能类别 | 排除的函数 | 说明 |
|----------|------------|------|
| **影子密码** | `getspnam_r` | 获取影子密码记录 |
| **共享内存** | `shm_open`, `shm_unlink` | POSIX 共享内存 |
| **消息队列** | `mq_open`, `mq_close`, `mq_send`, `mq_receive` 等 | POSIX 消息队列 |
| **Robust Mutex** | `pthread_mutex_consistent`, `pthread_mutex_setprioceiling` 等 | 强健互斥锁 |
| **线程取消** | `pthread_cancel` | 线程取消功能 |

**排除位置**：`src/unix/linux_like/linux/mod.rs` 第 4911-4977 行

```rust
cfg_if! {
    if #[cfg(not(target_env = "ohos"))] {
        extern "C" {
            // 这些函数仅在非 OHOS 环境下声明
            pub fn getspnam_r(...) -> ::c_int;
            pub fn shm_open(...) -> ::c_int;
            // ...
        }
    }
}
```

### 2. 类型对齐规则

OpenHarmony 与 musl 共享相同的类型对齐规则：

| 类型 | 对齐方式 | 说明 |
|------|----------|------|
| `pthread_rwlockattr_t` | `c_int` 对齐（而非 `c_long`）| 读写锁属性 |
| `pthread_cond_t` | 指针大小对齐 | 条件变量 |
| `pthread_mutexattr_t` | `c_int` 对齐（而非 `c_long`）| 互斥锁属性 |
| 零大小数组 | `*const c_void` 对齐（而非 `c_longlong`）| 灵活数组成员 |

**相关文件**：
- `src/unix/linux_like/linux/align.rs`
- `src/unix/linux_like/linux/no_align.rs`

### 3. Socket 选项常量

OpenHarmony 不支持较新的 socket 选项常量：

```rust
// 排除的常量
if #[cfg(all(any(target_arch = "x86", ...),
             not(any(target_env = "musl", target_env = "ohos"))))] {
    pub const SO_TIMESTAMP_NEW: ::c_int = 63;
    pub const SO_TIMESTAMPNS_NEW: ::c_int = 64;
    // ...
}
```

**说明**：
- `SO_*_NEW` 系列常量在 OHOS 和 musl 中不可用
- 这是 Linux 内核较新的特性，OHOS 尚未支持

---

## 3.6 编译配置详解

### Rust 编译器要求

| 最低版本 | 推荐版本 | 理由 |
|----------|----------|------|
| 1.13.0 | 1.62.0+ | 支持 `const extern fn` 特性 |

### 目标平台配置

OpenHarmony 使用以下目标配置：

```rust
target_os = "linux"           // Linux 类系统
target_env = "ohos"           // OpenHarmony 特定环境
target_arch = {x86_64, aarch64, riscv64, ...}  // 支持的架构
```

### 条件编译逻辑

```rust
// 示例：utmpx 结构体
pub struct utmpx {
    // ...
    #[cfg(target_env = "musl")]
    pub ut_session: ::c_long,

    #[cfg(target_env = "ohos")]
    #[cfg(target_endian = "little")]
    pub ut_session: ::c_int,      // OHOS 使用 c_int 而非 c_long
    #[cfg(target_env = "ohos")]
    #[cfg(target_endian = "little")]
    __ut_pad2: ::c_int,           // 填充字段保持对齐
}
```

**说明**：
- OHOS 使用 `target_env = "ohos"` 进行条件编译
- 与 musl 共享部分代码路径，但有特定差异
- 需要区分大小端（`target_endian`）

---

## 3.7 依赖关系

### 构建依赖

| 依赖类型 | 名称 | 版本 | 用途 |
|----------|------|------|------|
| **构建依赖** | rustc | >= 1.13.0 | Rust 编译器 |
| **构建依赖** | Cargo | 任意 | 包管理器 |
| **构建依赖** | GN | OHOS 版本 | 构建系统 |
| **构建依赖** | ohos_cargo_crate | OHOS 版本 | GN-Cargo 集成 |

### 运行时依赖

| 依赖类型 | 名称 | 用途 |
|----------|------|------|
| **系统库** | musl libc | C 标准库实现 |
| **系统库** | Linux 内核 | 系统调用 |
| **标准库** | Rust std | Rust 标准库（通过 `std` 特性）|

---

## 3.8 构建产物

### 输出文件

```
out/ohos/{target}/obj/third_party/rust/crates/libc/liblibc.rlib
```

**文件格式**：Rust 静态库（`.rlib`）

**内容**：
- 编译后的 Rust 中间代码（`.rmeta`）
- 静态链接的 Rust 代码（`.o` 或 `.a`）
- 元数据（版本、依赖等）

### 使用方式

依赖 libc 的其他 Rust crates 通过以下方式链接：

```gn
ohos_cargo_crate("my_crate") {
  deps = [ "//third_party/rust/crates/libc:lib" ]
  # ...
}
```

或者在 Rust 代码中：

```rust
extern crate libc;

use libc::{open, O_RDONLY, c_int};
```

---

## 3.9 构建调试

### 常见构建问题

#### 1. 编译错误：类型冲突

**错误信息**：
```
error: type mismatch
    --> src/unix/linux_like/linux/musl/mod.rs:377:5
     |
377  |     pub ut_session: ::c_int,
     |                      ^^^^^^ expected `c_long`, found `c_int`
```

**原因**：
- utmpx 结构体的字段类型与 OHOS C 库不匹配

**解决方案**：
- 检查 `src/unix/linux_like/linux/musl/mod.rs` 中的 `utmpx` 定义
- 确保使用正确的条件编译（`target_env = "ohos"`）
- 验证与 OHOS C 库的头部文件一致

#### 2. 链接错误：未定义符号

**错误信息**：
```
error: linking with `cc` failed: exit code: 1
note: Undefined symbols for architecture x86_64:
  "_mq_open", referenced from...
```

**原因**：
- 尝试使用 OHOS 不支持的 POSIX 函数（如 `mq_open`）

**解决方案**：
- 检查是否意外使用了被排除的函数
- 查看 `src/unix/linux_like/linux/mod.rs` 第 4911-4977 行的排除列表
- 使用替代方案或移除相关代码

#### 3. 特性冲突

**错误信息**：
```
error: cannot find macro `s!` in this scope
    --> src/unix/mod.rs:100:5
     |
100  | s! {
     | ^ not found in this scope
```

**原因**：
- `s!` 宏未正确定义

**解决方案**：
- 确保 `src/macros.rs` 中的 `cfg_if!` 宏正确配置
- 检查 build.rs 是否正确设置了 `libc_union` 等标志

### 调试工具

| 工具 | 用途 | 使用方式 |
|------|------|----------|
| **rustc --version** | 检查 Rust 版本 | `rustc --verbose --version` |
| **cargo tree** | 查看依赖树 | `cargo tree` |
| **cargo build --verbose** | 详细编译输出 | `cargo build --verbose` |
| **GN 配置文件** | 查看 GN 配置 | 查看 `out/ohos/toolchain.ninja` |

---

## 3.10 性能优化

### 编译优化

| 优化项 | 说明 | 配置方式 |
|--------|------|----------|
| **Release 模式** | 启用优化编译 | `build_type = "release"` |
| **LTO** | 链接时优化 | `lto = true` |
| **codegen-units** | 减少代码生成单元 | `codegen-units = 1` |
| **opt-level** | 优化级别 | `opt-level = 3` |

### 运行时优化

| 优化项 | 说明 | 影响 |
|--------|------|------|
| **内联函数** | 减少函数调用开销 | 减少调用开销，增加代码体积 |
| **const fn** | 编译时计算 | 提升运行时性能 |
| **零成本抽象** | Rust 特性 | 不影响性能 |

---

## 3.11 参考资源

### 外部参考
- [Cargo Book - Build Scripts](https://doc.rust-lang.org/cargo/reference/build-scripts.html)
- [GN Reference](https://gn.googlesource.com/gn/+/main/docs/reference.md)
- [Rust Edition Guide](https://doc.rust-lang.org/edition-guide/rust-2015/index.html)
- [musl libc](https://musl.libc.org/)

### 内部资源
- [BUILD.gn](../../BUILD.gn) - GN 构建配置
- [Cargo.toml](../../Cargo.toml) - Cargo 包配置
- [build.rs](../../build.rs) - Cargo 构建脚本
- [bundle.json](../../bundle.json) - OHOS 组件配置

---

**文档版本**: 1.0
**作者**: Sisyphus (OpenHarmony Third-Party Wiki Agent)
**最后审核**: 待审核
