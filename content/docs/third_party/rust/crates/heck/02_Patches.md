# heck Patch 分析

## 分析结论

**heck 库在 OpenHarmony 中无任何 Patch 文件。**

这是 heck 作为第三方库在 OH 集成中的一个显著特点——完全不需要定制化修改。

## 1. Patch 文件清单

### 搜索结果

```bash
# 在 heck 根目录搜索
find . -name "*.patch" -o -name "patches" -type d

# 结果：无任何匹配
```

### 结论

| 项目 | 结果 |
|------|------|
| **Patch 文件数量** | 0 |
| **patches 目录** | 不存在 |
| **OH 特有修改** | 无 |

## 2. 为何无需 Patch？

heck 库无需 Patch 的原因可从以下维度分析：

### 2.1 代码特性维度

| 特性 | heck 的实现 | 为何无需 Patch |
|------|-------------|----------------|
| **unsafe 代码** | `#![forbid(unsafe_code)]` | 全 Safe Rust，无需平台适配 |
| **系统调用** | 无 | 纯算法，不依赖 OS API |
| **I/O 操作** | 无 | 不读写文件、网络 |
| **FFI 调用** | 无 | 无 C 库依赖 |
| **平台相关代码** | 无 | 纯 Rust 标准库实现 |

### 2.2 功能维度

heck 是纯**字符串处理算法库**：

```rust
// 核心逻辑示例：状态机驱动的字符遍历
fn transform<F, G>(
    s: &str,
    mut with_word: F,      // 处理每个词
    mut boundary: G,        // 处理词边界
    f: &mut fmt::Formatter,
) -> fmt::Result
where
    F: FnMut(&str, &mut fmt::Formatter) -> fmt::Result,
    G: FnMut(&mut fmt::Formatter) -> fmt::Result,
{
    // 遍历字符，根据大小写规则判断词边界
    // 完全基于 Unicode/ASCII 字符属性
}
```

### 2.3 依赖维度

- **上游依赖**: 0（默认配置）
- **OH 构建**: 通过 `ohos_cargo_crate` 模板直接构建
- **无版本冲突**: 独立库，无复杂依赖树

## 3. OH 与上游的差异

尽管无 Patch，但 OH 的构建配置与上游 Cargo.toml 存在细微差异：

### 3.1 特性差异

| 项目 | 上游 Cargo.toml | OH BUILD.gn | 差异说明 |
|------|----------------|-------------|----------|
| `unicode` 特性 | 可选启用 | **未启用** | OH 使用 ASCII 模式处理 |

**影响分析**：

```rust
// src/lib.rs 中的特性控制代码
#[cfg(feature = "unicode")]
fn get_iterator(s: &str) -> unicode_segmentation::UnicodeWords {
    use unicode_segmentation::UnicodeSegmentation;
    s.unicode_words()
}

#[cfg(not(feature = "unicode"))]
fn get_iterator(s: &str) -> impl Iterator<Item = &str> {
    s.split(|letter: char| !letter.is_ascii_alphanumeric())
}
```

- **启用 unicode**: 使用 `unicode-segmentation` crate，正确处理 Unicode 词边界
- **未启用 unicode**: 使用 ASCII 字符分割（`is_ascii_alphanumeric`）

**结论**：OH 当前配置下，非 ASCII 字符（如中文、日文）的处理可能与上游有差异，但这不影响 clap_derive 的使用场景（命令行参数通常为 ASCII）。

### 3.2 配置等价性

```toml
# 上游 Cargo.toml (简化)
[package]
name = "heck"
version = "0.4.1"
edition = "2018"
```

```gn
# OH BUILD.gn (完整)
ohos_cargo_crate("lib") {
    crate_name = "heck"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    sources = ["src/lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.4.1"
    cargo_pkg_authors = "Without Boats <woboats@gmail.com>"
    cargo_pkg_name = "heck"
    cargo_pkg_description = "heck is a case conversion library."
    module_output_extension = ".rlib"
    part_name = "rust_heck"
    subsystem_name = "thirdparty"
}
```

**差异**: 仅元数据差异，功能完全一致。

## 4. 升级建议

### 4.1 当前版本状态

- **OH 版本**: 0.4.1
- **上游最新**: 0.4.1（截至评估时）
- **状态**: 已是最新稳定版

### 4.2 未来升级注意事项

如需升级上游版本，需关注：

| 风险项 | 评估 | 建议 |
|--------|------|------|
| API 变更 | 低 | heck API 稳定，v0.4.x 无破坏性变更 |
| 特性变更 | 中 | 检查是否新增默认特性 |
| 依赖变更 | 低 | 当前零依赖，变更可能性低 |
| clap_derive 兼容性 | 需验证 | 升级后需验证 clap_derive 构建 |

### 4.3 推荐的 Patch 策略

由于 heck **当前无需 Patch**，建议：

1. **保持零 Patch**：继续直接使用上游源码
2. **升级流程**：更新 BUILD.gn 中的 `cargo_pkg_version` → 验证构建 → 提交
3. **问题处理**：若发现 bug，优先尝试推向上游修复

## 5. 与其他库的对比

| 库 | Patch 数量 | 原因 |
|----|-----------|------|
| **heck** | **0** | 纯算法，无平台依赖 |
| curl | 大量 | 网络、平台适配、OH 定制需求 |
| openssl | 大量 | 安全加固、平台适配 |
| syn | 少量 | 修复、兼容性调整 |

heck 属于 **"零 Patch 友好"** 类型的库，维护成本低。

## 6. 总结

| 问题 | 答案 |
|------|------|
| 有 Patch 吗？ | **无** |
| 需要 Patch 吗？ | **不需要** |
| 未来需要 Patch 吗？ | 可能性极低 |
| 升级复杂吗？ | **简单**，直接替换源码 |

**维护建议**: heck 是 OH 中维护最简单的第三方库之一，保持零 Patch 策略，按需升级即可。
