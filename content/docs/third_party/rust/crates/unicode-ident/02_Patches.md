# Patch 分析

## 概述

**unicode-ident 在 OpenHarmony 中无任何 Patch 文件。**

这是一个**纯粹的上游库**，所有代码均来自上游版本，未做任何修改。

---

## Patch 文件清单

### 搜索结果

```bash
# 在库根目录执行
$ find . -name "*.patch" -o -name "patches" -type d 2>/dev/null
# 结果: (无输出)

# 检查 .git 目录外
$ find . -path ./.git -prune -o -name "*.patch" -print 2>/dev/null
# 结果: (无输出)
```

**确认**: 本库没有任何 Patch 文件。

---

## 为何无需 Patch

### 1. 纯数据表实现

```rust
// src/lib.rs - 核心实现仅包含两个公共函数
#![no_std]

pub fn is_xid_start(ch: char) -> bool {
    if ch.is_ascii() {
        return ASCII_START.0[ch as usize];
    }
    let chunk = *TRIE_START.0.get(ch as usize / 8 / CHUNK).unwrap_or(&0);
    let offset = chunk as usize * CHUNK / 2 + ch as usize / 8 % CHUNK;
    unsafe { LEAF.0.get_unchecked(offset) }.wrapping_shr(ch as u32 % 8) & 1 != 0
}

pub fn is_xid_continue(ch: char) -> bool {
    // 类似实现...
}
```

所有数据存储在 `src/tables.rs` 的静态数组中：
- `ASCII_START` / `ASCII_CONTINUE`: ASCII 字符查找表
- `TRIE_START` / `TRIE_CONTINUE`: 非 ASCII 字符的 trie 索引
- `LEAF`: 共享的叶节点数据块

### 2. 无平台相关代码

**代码中无任何平台条件编译**:

```rust
// 检查所有 #[cfg] 用法
$ grep -r '#\[cfg' src/
# 结果: 无匹配

// 检查所有条件编译
$ grep -r '#!\[cfg' src/
# 结果: 无匹配
```

### 3. no_std 设计

```rust
// src/lib.rs 第一行
#![no_std]
```

- 不依赖标准库 (`std`)
- 不使用操作系统 API
- 纯计算逻辑，无系统调用

### 4. Unicode 标准直接实现

遵循 **Unicode Standard Annex #31**，这是跨平台的国际标准：
- 不依赖特定操作系统的 Unicode 支持
- 数据表直接从 Unicode Character Database (UCD) 生成
- 实现方式与平台无关

---

## 与典型带 Patch 库的对比

| 特性 | unicode-ident (无 Patch) | 典型带 Patch 库 (如 curl/openssl) |
|------|-------------------------|----------------------------------|
| **代码性质** | 纯数据表 | 系统调用/API 封装 |
| **平台依赖** | 无 | 高（网络、文件系统等） |
| **OH 适配需求** | 无需适配 | 需要适配 OH 特定 API |
| **Patch 数量** | 0 | 通常 5-50+ |
| **维护复杂度** | 极低 | 高 |

**其他无 Patch 的 Rust crates 示例**:
- `quote`: 纯代码生成
- `bitflags`: 纯宏实现
- `unicode-ident`: 纯数据表

---

## 潜在 Patch 场景分析

### 理论上可能需要 Patch 的情况

| 场景 | 可能性 | 说明 |
|------|--------|------|
| **Unicode 版本回退** | 极低 | 如需支持旧 Unicode 版本，但会违反标准 |
| **特定字符集白名单** | 极低 | 限制允许的标识符字符，需业务层实现 |
| **内存优化** | 低 | 如需要更小的二进制体积，需重新生成数据表 |

### 实际评估

以上场景在 OpenHarmony 中均不适用：
- ✅ 使用标准 Unicode 16.0.0
- ✅ 无需字符集限制
- ✅ 10.4 KB 静态存储可接受

---

## 数据表生成流程

虽然不需要 Patch，但理解数据表来源有助于维护：

```bash
# 数据表重新生成步骤（通常不需要执行）
$ curl -LO https://www.unicode.org/Public/zipped/16.0.0/UCD.zip
$ unzip UCD.zip -d UCD
$ cargo run --manifest-path generate/Cargo.toml
```

**生成工具**:
- 位于 `generate/` 目录
- 从 Unicode Character Database 提取数据
- 输出压缩的 trie 结构到 `src/tables.rs`

**注意**: 重新生成会改变 `src/tables.rs`，但这不是 Patch，而是版本升级。

---

## 升级建议

### 上游版本升级流程

1. **获取新版本**:
   ```bash
   # 从上游 GitHub 下载新版本
   wget https://github.com/dtolnay/unicode-ident/archive/refs/tags/1.0.x.tar.gz
   ```

2. **验证兼容性**:
   ```bash
   # 检查 API 变更
   diff src/lib.rs  # 确保函数签名未变
   ```

3. **测试下游依赖**:
   ```bash
   # 测试 proc-macro2
   cd third_party/rust/crates/proc-macro2 && cargo test
   
   # 测试 syn
   cd third_party/rust/crates/syn && cargo test
   ```

4. **更新 BUILD.gn**:
   ```gn
   # 更新版本号
   cargo_pkg_version = "1.0.x"
   ```

### 风险等级

- **低风险**: 1.0.x 系列内升级（仅 Unicode 数据更新）
- **中风险**: 2.0.x 升级（API 可能变更，但概率低）

---

## 总结

| 项目 | 状态 |
|------|------|
| **Patch 数量** | 0 |
| **OH 特有修改** | 无 |
| **维护复杂度** | 极低 |
| **升级方式** | 直接替换上游版本 |

**unicode-ident 是 OpenHarmony 中最简单类型的第三方库**: 纯数据表、无平台依赖、无特殊适配。所有变更均通过上游版本更新实现，无需维护任何 Patch。

---

*下一章: [BUILD 集成](03_Build_Integration.md)*
