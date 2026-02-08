# 03 OpenHarmony 构建适配

> Nix 库在 OpenHarmony 构建系统中的配置说明

## 构建配置概览

| 配置项 | 值 |
|--------|-----|
| **构建模板** | `ohos_cargo_crate` |
| **输出类型** | rlib (Rust 静态库) |
| **Rust Edition** | 2021 |
| **OH 版本** | 5.0 |
| **Part Name** | rust_nix |
| **Subsystem** | thirdparty |

---

## BUILD.gn 配置详解

### 完整配置

```gn
import("//build/templates/rust/ohos_cargo_crate.gni")

ohos_cargo_crate("lib") {
    # 基本信息
    crate_name = "nix"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    edition = "2021"
    
    # Cargo 包元数据
    cargo_pkg_version = "0.30.1"
    cargo_pkg_authors = "The nix-rust Project Developers"
    cargo_pkg_name = "nix"
    cargo_pkg_description = "Rust friendly bindings to *nix APIs"
    
    # 依赖配置
    deps = [
        "//third_party/rust/crates/bitflags:lib",
        "//third_party/rust/crates/cfg-if:lib",
        "//third_party/rust/crates/libc:lib",
        "//third_party/rust/crates/memoffset:lib",
        "//third_party/rust/crates/pin-utils:lib",
    ]
    
    # 启用的 Features
    features = [
        "acct",
        "aio",
        "default",
        "dir",
        "env",
        "event",
        "fanotify",
        "feature",
        "fs",
        "hostname",
        "inotify",
        "ioctl",
        "kmod",
        "memoffset",
        "mman",
        "mount",
        "mqueue",
        "net",
        "personality",
        "poll",
        "process",
        "pthread",
        "ptrace",
        "quota",
        "reboot",
        "resource",
        "sched",
        "signal",
        "socket",
        "syslog",
        "term",
        "time",
        "ucontext",
        "uio",
        "user",
        "zerocopy",
    ]
    
    # 输出配置
    module_output_extension = ".rlib"
    
    # OH 组件配置
    part_name = "rust_nix"
    subsystem_name = "thirdparty"
}
```

### 配置项解析

#### 构建模板：`ohos_cargo_crate`

该模板是 OpenHarmony 为 Rust crates 封装的标准化构建模板，负责：

1. **Cargo.toml 解析**：读取 crate 的 Cargo.toml 配置
2. **依赖解析**：处理 OH 内部的 Rust crate 依赖
3. **条件编译**：根据 `target_os` 配置条件编译
4. **输出打包**：生成 rlib 静态库文件

#### 依赖配置

```gn
deps = [
    "//third_party/rust/crates/bitflags:lib",
    "//third_party/rust/crates/cfg-if:lib",
    "//third_party/rust/crates/libc:lib",
    "//third_party/rust/crates/memoffset:lib",
    "//third_party/rust/crates/pin-utils:lib",
]
```

| 依赖 | 版本 | 用途 |
|------|------|------|
| `libc` | OH 版本 | C 标准库的系统调用绑定 |
| `bitflags` | 2.3.3 | 位标志的类型安全封装 |
| `cfg-if` | 1.0 | 条件编译支持 |
| `memoffset` | 0.9 | 结构体内存偏移量计算 |
| `pin-utils` | 0.1.0 | Pin 类型辅助工具 |

#### Features 配置

启用了 **31 个 features**，涵盖大部分核心功能：

| Feature 分类 | 启用的 Features |
|-------------|----------------|
| **进程/线程** | `process`, `pthread`, `sched`, `signal` |
| **文件系统** | `fs`, `dir`, `mount`, `mqueue`, `uio` |
| **网络** | `socket`, `net`, `syslog` |
| **系统调用** | `mman`, `inotify`, `ioctl`, `kmod`, `ptrace` |
| **工具** | `poll`, `event`, `env`, `hostname`, `term` |
| **扩展** | `acct`, `aio`, `fanotify`, `personality`, `quota`, `reboot`, `resource`, `time`, `ucontext`, `user`, `zerocopy` |

**注意**：`default` feature 未显式启用（nix 的 default 是空特征集）

---

## 与上游构建系统的差异

### 1. 依赖管理

| 维度 | 上游 | OH |
|------|------|-----|
| 依赖来源 | crates.io | OH 内部 third_party 目录 |
| 版本锁定 | Cargo.lock | OH 统一版本管理 |
| 条件依赖 | 按 platform cfg | 全部展开为具体依赖 |

### 2. 构建输出

| 维度 | 上游 | OH |
|------|------|-----|
| 输出格式 | cargo build 生成 | ohos_cargo_crate 生成 rlib |
| 输出路径 | target/ | out/ 目录 |
| 命名规则 | libnix.rlib | libnix.rlib (同) |

### 3. 条件编译

**上游**：
```toml
[target.'cfg(target_os = "linux")'.dependencies]
libc = "0.2"
```

**OH**：
```gn
# BUILD.gn 中直接指定 OH 内部依赖
deps = [
    "//third_party/rust/crates/libc:lib",  # 使用 OH 版本的 libc
]
```

---

## 特殊处理说明

### 1. 模块输出配置

```gn
module_output_extension = ".rlib"
```

指定输出 Rust 静态库文件。

### 2. 组件配置

```gn
part_name = "rust_nix"
subsystem_name = "thirdparty"
```

注册为 OpenHarmony 的：
- **Part**: rust_nix（可被其他模块依赖）
- **Subsystem**: thirdparty（第三方组件子系统）

### 3. 条件编译适配

nix 源码中通过以下方式处理 OpenHarmony：

```rust
// 条件编译示例
#[cfg(any(target_os = "linux", target_os = "android", target_os = "ohos"))]
pub mod inotify;

// OH 使用与 Linux 相同的实现
cfg_if::cfg_if! {
    if #[cfg(target_os = "ohos")] {
        // OH 特定配置
        type RawFd = libc::c_int;
    }
}
```

---

## 构建验证

### 验证命令

```bash
# 完整构建
hb build -p rust_nix

# 单独编译
cargo build --target aarch64-unknown-linux-ohos
```

### 构建输出

```
out/.../obj/third_party/rust/crates/nix/libnix.rlib
```

### 依赖关系验证

```bash
# 查看 nix 的依赖链
# nix -> libc, bitflags, cfg-if, memoffset, pin-utils
```

---

## 常见构建问题

### 问题 1：Features 配置冲突

**症状**：编译错误，提示缺少某些依赖

**解决方案**：确保 BUILD.gn 中的 features 与 Cargo.toml 中的定义一致

### 问题 2：条件编译缺失

**症状**：某些 API 在 OH 上不可用

**解决方案**：添加对应的 features 到 BUILD.gn

### 问题 3：依赖版本不兼容

**症状**：libc 版本冲突

**解决方案**：检查 OH 统一版本配置，确保兼容性

---

## 维护建议

### 版本升级流程

1. **更新版本号**
   ```gn
   cargo_pkg_version = "x.x.x"  // 新版本号
   ```

2. **同步 Cargo.toml**
   ```bash
   # 从上游复制新版本 Cargo.toml
   ```

3. **验证 Features**
   ```bash
   # 检查新版本 features 是否变化
   cargo build --features --help
   ```

4. **测试构建**
   ```bash
   hb build -p rust_nix
   ```

### Features 优化建议

| 场景 | 建议 |
|------|------|
| **最小化构建** | 只启用必要 features |
| **完整功能** | 启用所有 features |
| **特定场景** | 按需启用相关 features |

---

## 总结

| 配置项 | 状态 | 说明 |
|--------|------|------|
| **构建模板** | ✅ 标准 | 使用 ohos_cargo_crate |
| **依赖管理** | ✅ 标准 | OH 内部 crate 依赖 |
| **Features** | ✅ 完整 | 启用 31 个核心 features |
| **条件编译** | ✅ 兼容 | OH 与 Linux 共用实现 |
| **Patch** | ✅ 无需 | 上游原生支持 |

---

**上一节**: [02_Patches.md](02_Patches.md)  
**下一节**: [04_Usage_in_OH.md](04_Usage_in_OH.md)
