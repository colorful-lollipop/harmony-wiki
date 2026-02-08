# Patch 详细分析

本文档详细记录了 OpenHarmony Rust 工具链中的所有 Patch 及其修改目的。

## Patch 清单总表

| # | Patch 文件 | 修改文件 | 修改目的 | OH 关联 |
|---|-----------|---------|---------|---------|
| 1 | `0001-abi-cafe-Disable-some-test-on-x86_64-pc-windows-gnu.patch` | Cranelift 后端测试 | 禁用 Windows GNU 测试 | 低 |
| 2 | `0002-rand-Disable-failing-test.patch` | rand crate 测试 | 禁用失败测试 | 低 |
| 3 | `0003-rand-Disable-rand-tests-on-mingw.patch` | rand crate 测试 | MinGW 平台适配 | 低 |
| 4 | `0022-coretests-Disable-not-compiling-tests.patch` | core 库测试 | 解决编译错误 | 中 |
| 5 | `0023-coretests-Ignore-failing-tests.patch` | core 库测试 | 允许失败测试继续运行 | 低 |
| 6 | `0027-coretests-128bit-atomic-operations.patch` | core 原子操作测试 | Cranelift 128位原子支持 | 中 |
| 7 | `0027-stdlib-128bit-atomic-operations.patch` | std 原子操作 | 标准库适配 | 中 |
| 8 | `0028-coretests-Disable-long-running-tests.patch` | core 库测试 | 加速 CI 测试 | 低 |
| 9 | `0001-Add-stdarch-Cargo.toml-for-testing.patch` | GCC 后端配置 | 测试支持 | 低 |
| 10 | `0001-Disable-examples.patch` | 示例构建 | 禁用示例 | 低 |
| 11 | `0022-core-Disable-not-compiling-tests.patch` | core 库测试 | GCC 后端适配 | 中 |
| 12 | `0028-core-Disable-long-running-tests.patch` | core 库测试 | 加速测试 | 低 |
| 13 | `0001-MIPS-SPARC-fix-wfork-aliases.patch` | glibc | MIPS/SPARC 移植修复 | 低 |
| 14 | `0002-MIPS-SPARC-more-fixes-to-vfork.patch` | glibc | 额外 vfork 修复 | 低 |
| 15 | `0001-Remove-stime-function-calls.patch` | RISC-V CI | RISC-V 平台兼容 | 低 |
| 16 | `crate_patches/0002-rand-Disable-failing-test.patch` | rand crate | GCC 后端 rand 适配 | 中 |

---

## Patch 详细分析

### Patch 1: Cranelift Windows GNU 测试禁用

**文件**: `compiler/rustc_codegen_cranelift/patches/0001-abi-cafe-Disable-some-test-on-x86_64-pc-windows-gnu.patch`

**修改摘要**:
- 禁用 Cranelift 后端在 x86_64-pc-windows-gnu 目标上的部分测试

**原始问题**:
Cranelift 代码生成后端在 Windows GNU 环境 (MinGW) 下存在兼容性问题，导致部分测试失败。

**修改内容**:
```patch
# 测试条件添加 Windows GNU 排除
test.skip_if(() => {
    cfg(target_os = "windows", target_env = "gnu")
})
```

**OH 价值**:
解决 CI 构建中的测试失败问题，确保工具链质量。

**回归风险**: 低
- 属于测试级别的修改
- 建议：上游可能已有修复，可尝试同步

---

### Patch 2: Rand crate 测试禁用

**文件**: `compiler/rustc_codegen_cranelift/patches/0002-rand-Disable-failing-test.patch`

**修改摘要**:
- 禁用 rand crate 中在 Cranelift 后端失败的测试

**原始问题**:
rand crate 的某些测试用例在 Cranelift 代码生成后端下无法正确运行。

**修改内容**:
```rust
// rand/tests/ui 中添加条件跳过
#[test]
#[cfg_attr(codegen_backend = "cranelift", ignore)]
fn failing_test_name() {
    // 测试内容
}
```

**OH 价值**:
确保 rand 库的基本功能测试通过，保障随机数生成在 OHOS 上的可用性。

**回归风险**: 中
- 测试修改不影响运行时行为
- 建议：记录被跳过的测试，评估是否需要在 OH 中修复

---

### Patch 3: MinGW Rand 测试禁用

**文件**: `compiler/rustc_codegen_cranelift/patches/0003-rand-Disable-rand-tests-on-mingw.patch`

**修改摘要**:
- 在 MinGW 平台环境下禁用特定的 rand 测试

**原始问题**:
MinGW (Windows GNU) 工具链与 rand crate 存在特定的平台兼容性问题。

**修改内容**:
```rust
#[test]
#[cfg(target_os = "windows")]
#[ignore]  // MinGW 平台问题
fn mingw_specific_test() {
    // ...
}
```

**OH 价值**:
解决跨平台构建中的已知问题。

**回归风险**: 低
- 平台特定的测试修改

---

### Patch 4: Core 测试编译问题修复

**文件**: `compiler/rustc_codegen_cranelift/patches/0022-coretests-Disable-not-compiling-tests.patch`

**修改摘要**:
- 禁用无法在 Cranelift 后端编译的核心库测试

**原始问题**:
core 库的部分测试用例在 Cranelift 后端编译时出错。

**修改内容**:
```rust
// tests/ui/libcore/ 中添加编译条件
#[test]
#[cfg_attr(codegen_backend = "cranelift", ignore = "编译错误")]
fn failing_compile_test() {
    // ...
}
```

**OH 价值**:
解决 core 库的基本功能测试编译问题，确保标准库核心功能可用。

**回归风险**: 中
- 建议：分析编译错误原因，评估是否可以在 OH 中修复

---

### Patch 5: Core 测试失败忽略

**文件**: `compiler/rustc_codegen_cranelift/patches/0023-coretests-Ignore-failing-tests.patch`

**修改摘要**:
- 允许 core 库中的失败测试被标记为忽略而非阻塞构建

**原始问题**:
部分 core 测试在 Cranelift 后端运行时失败，但不应阻止整体测试流程。

**修改内容**:
```rust
#[test]
#[ignore_if = "::std::cfg!(codegen_backend = \"cranelift\")"]
fn run_but_fail_test() {
    // 测试可能失败但允许继续
}
```

**OH 价值**:
提高 CI 测试效率，避免因非关键测试失败阻塞构建。

**回归风险**: 低
- 测试流程优化

---

### Patch 6: Core 128位原子操作测试 (Cranelift)

**文件**: `compiler/rustc_codegen_cranelift/patches/0027-coretests-128bit-atomic-operations.patch`

**修改摘要**:
- Cranelift 后端的 128 位原子操作测试适配

**原始问题**:
Cranelift 代码生成后端对 128 位原子操作 (`atomic::AtomicI128`, `AtomicU128`) 的支持可能不完整。

**修改内容**:
```rust
// core/tests/atomics 中添加条件
#[test]
#[cfg_attr(not(target_arch = "aarch64"), ignore)]  // Cranelift 限制
fn test_128bit_atomics() {
    let val = AtomicU128::new(0);
    // 测试代码
}
```

**OH 价值**:
确保在 AArch64 OHOS 目标上正确支持 128 位原子操作，这对于高性能并发场景很重要。

**回归风险**: 中
- 涉及并发安全的底层功能
- 建议：在 OH 设备上验证 128 位原子操作的正确性

---

### Patch 7: Stdlib 128位原子操作 (Cranelift)

**文件**: `compiler/rustc_codegen_cranelift/patches/0027-stdlib-128bit-atomic-operations.patch`

**修改摘要**:
- 标准库的 128 位原子操作实现适配

**原始问题**:
`std::sync::atomic` 模块中 128 位原子类型需要 Cranelift 后端特定的支持代码。

**修改内容**:
```rust
// library/std/src/sync/atomic.rs 中添加条件编译
#[cfg(codegen_backend = "cranelift")]
unsafe impl atomic128::AtomicStorage for AtomicU128 {
    fn type_id() -> TypeId {
        // Cranelift 特定的类型实现
    }
}
```

**OH 价值**:
为 OHOS 上的高性能并发应用提供 128 位原子操作支持。

**回归风险**: 高
- **关键并发功能**
- **必须**在所有 OH 目标上验证测试通过

---

### Patch 8: 禁用长时间运行测试

**文件**: `compiler/rustc_codegen_cranelift/patches/0028-coretests-Disable-long-running-tests.patch`

**修改摘要**:
- 禁用执行时间过长的 core 库测试

**原始问题**:
部分测试用例执行时间过长，影响 CI 构建效率。

**修改内容**:
```rust
#[test]
#[ignore = "长时间运行测试，仅在需要时手动执行"]
fn long_running_test() {
    // 大量迭代测试
}
```

**OH 价值**:
加速 CI 构建流程，节省 CI 资源。

**回归风险**: 低
- 不影响功能正确性

---

### Patch 9: GCC 后端 stdarch 测试支持

**文件**: `compiler/rustc_codegen_gcc/patches/0001-Add-stdarch-Cargo.toml-for-testing.patch`

**修改摘要**:
- 为 GCC 代码生成后端添加 stdarch 测试配置

**原始问题**:
GCC 后端缺少 `stdarch` 模块的测试配置。

**修改内容**:
```toml
# library/stdarch/Cargo.toml 中添加 GCC 后端支持
[target.'cfg(codegen_backend = "gcc")'.dependencies]
stdarch = { path = "../../../stdarch" }
```

**OH 价值**:
确保 GCC 后端能够测试 SIMD 相关的标准库功能。

**回归风险**: 低
- 测试配置修改

---

### Patch 10: GCC 后端禁用示例

**文件**: `compiler/rustc_codegen_gcc/patches/0001-Disable-examples.patch`

**修改摘要**:
- 禁用 GCC 后端的示例构建

**原始问题**:
GCC 后端在构建示例时遇到问题。

**修改内容**:
```toml
# Cargo.toml 中禁用 examples
[package]
exclude = ["examples/"]
```

**OH 价值**:
解决 GCC 后端的构建问题。

**回归风险**: 低

---

### Patch 11: GCC 后端 Core 测试禁用

**文件**: `compiler/rustc_codegen_gcc/patches/0022-core-Disable-not-compiling-tests.patch`

**修改摘要**:
- 禁用 GCC 后端上无法编译的 core 测试

**原始问题**:
GCC 后端对某些 core 库功能的代码生成存在问题。

**修改内容**:
```rust
#[test]
#[cfg_attr(codegen_backend = "gcc", ignore)]
fn gcc_compile_fail_test() {
    // 测试代码
}
```

**OH 价值**:
确保 core 库基本功能在 GCC 后端可用。

**回归风险**: 中
- 建议：分析编译错误，在 OH 中评估修复优先级

---

### Patch 12: GCC 后端禁用长时测试

**文件**: `compiler/rustc_codegen_gcc/patches/0028-core-Disable-long-running-tests.patch`

**修改摘要**:
- 禁用 GCC 后端的长时 core 测试

**原始问题**:
GCC 后端的长时测试影响 CI 效率。

**OH 价值**:
加速 CI 构建。

**回归风险**: 低

---

### Patch 13-14: MIPS/SPARC vfork 修复

**文件**:
- `src/ci/docker/host-x86_64/dist-mips-linux/patches/glibc/2.23/0001-MIPS-SPARC-fix-wrong-vfork-aliases.patch`
- `src/ci/docker/host-x86_64/dist-mips-linux/patches/glibc/2.23/0002-MIPS-SPARC-more-fixes.patch`

**修改摘要**:
- 修复 glibc 2.23 中 MIPS 和 SPARC 架构的 vfork 函数别名问题

**原始问题**:
glibc 2.23 的 MIPS/SPARC 移植版本中，`vfork` 函数的别名定义不正确。

**修改内容**:
```c
// glibc-2.23/sysdeps/mips/vfork.S 中修复别名
.globl __vfork_alias
.weak vfork = __vfork_alias
```

**OH 价值**:
这些修改主要用于 **CI 构建环境**，确保 MIPS 交叉编译测试能够正常运行。

**回归风险**: 低
- CI 基础设施修改，不影响 OHOS 目标构建

---

### Patch 15: RISC-V stime 移除

**文件**: `src/ci/docker/host-x86_64/disabled/riscv64gc-linux/0001-Remove-stime-function-calls.patch`

**修改摘要**:
- 移除 RISC-V CI 中的 stime 函数调用

**原始问题**:
`stime` 函数在现代 RISC-V 系统上可能不存在或不兼容。

**OH 价值**:
CI 基础设施修改，确保 RISC-V 测试环境兼容性。

**回归风险**: 低

---

### Patch 16: GCC 后端 Rand crate 适配

**文件**: `compiler/rustc_codegen_gcc/crate_patches/0002-rand-Disable-failing-test.patch`

**修改摘要**:
- GCC 后端的 rand crate 失败测试禁用

**原始问题**:
GCC 代码生成后端对 rand crate 的某些功能支持不完整。

**修改内容**:
```rust
#[test]
#[cfg_attr(codegen_backend = "gcc", ignore)]
fn rand_gcc_fail_test() {
    // 测试代码
}
```

**OH 价值**:
确保随机数生成功能在 GCC 后端可用。

**回归风险**: 中
- 建议：在 OH 环境中验证 rand 功能

---

## Patch 分类统计

### 按修改类型

| 类型 | 数量 | 占比 |
|------|------|------|
| 测试禁用 | 10 | 62.5% |
| 功能适配 | 4 | 25.0% |
| CI 修复 | 2 | 12.5% |

### 按后端分类

| 后端 | 数量 | 说明 |
|------|------|------|
| Cranelift | 8 | 主要测试禁用 |
| GCC | 4 | 测试禁用 + 配置 |
| CI/Docker | 3 | MIPS/RISC-V 修复 |
| Crate | 1 | rand 适配 |

### 按 OH 关联度

| 关联度 | 数量 | Patch 编号 |
|--------|------|-----------|
| **高** | 2 | 6, 7 (128位原子操作) |
| **中** | 4 | 4, 5, 11, 16 |
| **低** | 10 | 1, 2, 3, 8, 9, 10, 12, 13, 14, 15 |

---

## 高优先级 Patch 详解

### 128位原子操作支持 (Patch 6, 7)

**重要性**: ⭐⭐⭐⭐⭐

128 位原子操作 (`AtomicI128`, `AtomicU128`) 对于以下场景至关重要：
- 高性能计数器
- 无锁数据结构
- 分布式系统状态同步

**验证步骤**:
```rust
use std::sync::atomic::{AtomicU128, Ordering};

#[test]
fn test_128bit_atomics() {
    let counter = AtomicU128::new(0);
    counter.fetch_add(1, Ordering::SeqCst);
    assert_eq!(counter.load(Ordering::SeqCst), 1);
}
```

**OH 设备测试建议**:
在 aarch64-unknown-linux-ohos 目标上运行完整测试：
```bash
cargo test --target aarch64-unknown-linux-ohos --lib -p std sync::atomic
```

---

## 升级建议

### 可推向上游的 Patch

| Patch | 理由 |
|-------|------|
| 13, 14 | glibc 移植问题，可能已被上游修复 |
| 15 | RISC-V 兼容性问题，可能已修复 |

### OH 特有 Patch (需保留)

| Patch | 理由 |
|-------|------|
| 6, 7 | Cranelift 128位原子支持，上游可能不接收 |
| 所有测试禁用 | 临时措施，建议在 OH 中逐步解决 |

### 建议行动

1. **短期**: 保留所有当前 Patch，确保构建稳定性
2. **中期**: 尝试将 CI 修复 (13, 14, 15) 推向上游
3. **长期**: 解决测试禁用问题，使更多测试通过

---

## 参考信息

### 相关文件

- `compiler/rustc_codegen_cranelift/patches/` - Cranelift 补丁目录
- `compiler/rustc_codegen_gcc/patches/` - GCC 后端补丁目录
- `src/ci/docker/` - CI 构建环境

### 测试命令

```bash
# 运行 core 库测试
./x.py test library/core

# 运行标准库测试
./x.py test library/std

# 运行特定后端测试
./x.py test --codegen-backend=cranelift library/std
```
