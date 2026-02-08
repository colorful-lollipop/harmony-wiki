# API/接口差异

本文档分析 glob 库在 OpenHarmony 中的 API 差异、行为变更以及功能限制。

---

## 1. 总体结论

**✅ 无 API 差异**

glob 库在 OpenHarmony 中**完全使用上游版本**，未进行任何 API 修改或行为变更。

**证据**:
- ❌ 无 Patch 文件
- ❌ 源代码零修改（1434 行，无任何 OH 特定代码）
- ❌ 无条件编译代码（`#[cfg(...)]`）
- ❌ 无 OHOS 宏定义

---

## 2. OH 新增的 API

**❌ 无新增 API**

OpenHarmony **未添加**任何新的公共 API。

**原因**:
- glob 的功能已满足 OH 的需求
- 无需扩展 glob 的功能
- 保持与上游版本同步，便于升级

---

## 3. 行为变更的 API

**❌ 无行为变更**

所有 API 的行为与上游版本**完全一致**。

**验证**:
- 源代码无修改
- 无平台特定条件分支
- 使用默认的 Rust 标准库

**对比表**:

| API | OH 行为 | 上游行为 | 差异 |
|-----|---------|---------|------|
| `glob()` | Unix shell 风格匹配 | Unix shell 风格匹配 | ❌ 无 |
| `glob_with()` | 带选项的匹配 | 带选项的匹配 | ❌ 无 |
| `Pattern::matches()` | 单路径匹配 | 单路径匹配 | ❌ 无 |
| `MatchOptions` | 匹配选项配置 | 匹配选项配置 | ❌ 无 |

---

## 4. 废弃或禁用的功能

**❌ 无废弃或禁用的功能**

glob 的所有功能在 OpenHarmony 中**完全可用**。

**可用功能**:
- ✅ Unix shell 通配符（`*`, `?`, `[...]`, `**` 等）
- ✅ 递归目录遍历
- ✅ 匹配选项（大小写敏感、分隔符要求等）
- ✅ 错误处理

---

## 5. 平台兼容性

### 5.1 OpenHarmony vs 其他平台

glob 在 OpenHarmony 中的行为与其他平台**完全一致**：

| 平台 | 路径分隔符 | glob 行为 | 差异 |
|------|-----------|----------|------|
| **OpenHarmony** | `/` | Unix 风格 | - |
| **Linux** | `/` | Unix 风格 | ❌ 无 |
| **macOS** | `/` | Unix 风格 | ❌ 无 |
| **Windows** | `\` | 自动转换 | ⚠️ OH 不涉及 |

**说明**:
- OpenHarmony 使用 Unix 风格路径（`/`）
- glob 对此提供完全支持
- 无需任何路径转换逻辑

### 5.2 Rust 标准库兼容性

glob 依赖的 Rust 标准库 API 在 OpenHarmony 中**完全可用**：

```rust
// glob 使用的标准库模块
use std::cmp;           // ✅ 可用
use std::error::Error;  // ✅ 可用
use std::fmt;           // ✅ 可用
use std::fs;           // ✅ 可用
use std::io;           // ✅ 可用
use std::path;         // ✅ 可用（Unix 路径）
```

---

## 6. 性能特性

### 6.1 性能对比

glob 在 OpenHarmony 中的性能与其他平台**无差异**：

| 性能指标 | OH | Linux | 差异 |
|---------|----|-------|------|
| **模式匹配速度** | 相同 | 相同 | ❌ 无 |
| **文件遍历效率** | 相同 | 相同 | ❌ 无 |
| **内存占用** | 相同 | 相同 | ❌ 无 |
| **编译产物大小** | ~50-100 KB | ~50-100 KB | ❌ 无 |

**原因**:
- 同一代码
- 相同编译选项
- 相同优化级别

### 6.2 编译优化

glob 在 OpenHarmony 中使用**默认编译选项**：

```gn
# BUILD.gn
ohos_cargo_crate("lib") {
    # 无特殊优化选项
    # 继承 OH 系统默认
}
```

**优化级别**:
- **Debug**: 无优化，快速编译
- **Release**: 基础优化（`-O2` 级别）

**未启用的优化**:
- ❌ LTO（Link Time Optimization）
- ❌ PGO（Profile Guided Optimization）
- ❌ Aggressive inlining

---

## 7. 兼容性测试

### 7.1 上游测试

glob 包含完整的上游测试套件：

```rust
// tests/glob-std.rs
// 标准库兼容性测试

// 源代码中的文档测试
//! # Examples
//! ```rust
//! use glob::glob;
//! for entry in glob("*.rs")? {
//!     println!("{:?}", entry?);
//! }
//! ```
```

**测试覆盖**:
- ✅ 各种 glob 模式
- ✅ 错误处理
- ✅ 边界情况
- ✅ 性能基准

### 7.2 OH 特定测试

**❌ 无 OH 特定测试**

glob **未添加** OH 特定的测试用例。

**建议**:
- 考虑添加 OH 平台测试
- 验证在 OH 设备上的行为
- 监控性能指标

---

## 8. 使用限制

### 8.1 已知限制

这些限制来自 glob 本身的设计，非 OH 特定：

| 限制 | 说明 | 影响 |
|------|------|------|
| **不支持正则表达式** | 仅支持 Unix shell 风格模式 | ⚠️ 需要复杂匹配时不适用 |
| **无并发支持** | 迭代器不支持并发遍历 | ⚠️ 大规模遍历时性能受限 |
| **路径长度限制** | 受文件系统限制 | ⚠️ 超长路径可能失败 |
| **符号链接处理** | 可能跟随符号链接 | ⚠️ 可能有性能影响 |

### 8.2 OH 特定限制

**❌ 无 OH 特定限制**

glob 在 OpenHarmony 中的使用**无额外限制**。

---

## 9. 迁移建议

### 9.1 从其他平台迁移到 OH

**✅ 无需任何修改**

如果您的代码在其他平台使用 glob，在 OpenHarmony 中**完全兼容**：

```rust
// 这些代码在 OH 中完全可用
use glob::glob;

for entry in glob("src/**/*.rs")? {
    println!("{:?}", entry?);
}
```

**无需修改**:
- ❌ 路径分隔符（OH 使用 `/`）
- ❌ API 调用方式
- ❌ 匹配模式语法
- ❌ 错误处理逻辑

### 9.2 从旧版本 glob 升级

如果从 glob 0.3.0 升级到 0.3.1（OH 当前版本）：

**✅ 完全向后兼容**

版本 0.3.1 是 bug 修复版本，无 API 破坏性变更。

---

## 10. 常见问题

### Q1: glob 是否支持 Windows 风格路径（`\`）?

**A**: glob 主要支持 Unix 风格路径。在 Windows 上，glob 会自动转换路径分隔符，但 OpenHarmony 使用 Unix 风格路径（`/`），因此无需转换。

### Q2: glob 是否支持扩展模式（如 `**/*.rs`）?

**A**: ✅ 支持。glob 支持 Unix shell 的扩展 glob 模式，包括递归通配符 `**`。

### Q3: glob 在 OH 中的性能如何?

**A**: 与其他 Unix-like 平台性能相同。glob 使用高效的算法，文件遍历性能优秀。

### Q4: glob 是否支持中文文件名?

**A**: ✅ 支持。glob 使用 Rust 标准库的路径处理，完全支持 Unicode 文件名。

### Q5: glob 是否支持绝对路径和相对路径?

**A**: ✅ 都支持。
```rust
// 绝对路径
glob("/usr/lib/*.so")

// 相对路径
glob("src/**/*.rs")
```

---

## 11. API 完整性检查

### 11.1 公共 API 列表

glob 的所有公共 API 在 OH 中**完全可用**：

| 模块 | API | 可用性 |
|------|-----|--------|
| **核心函数** | `glob()` | ✅ |
| **核心函数** | `glob_with()` | ✅ |
| **Pattern** | `Pattern::new()` | ✅ |
| **Pattern** | `Pattern::matches()` | ✅ |
| **Pattern** | `Pattern::matches_path()` | ✅ |
| **MatchOptions** | `case_sensitive` | ✅ |
| **MatchOptions** | `require_literal_separator` | ✅ |
| **MatchOptions** | `require_literal_leading_dot` | ✅ |
| **Paths** | 迭代器实现 | ✅ |
| **GlobError** | 错误类型 | ✅ |
| **PatternError** | 错误类型 | ✅ |

### 11.2 文档完整性

glob 的 API 文档在 OH 中**与上游完全一致**：

- ✅ 完整的文档注释
- ✅ 使用示例
- ✅ 错误说明
- ✅ 行为描述

参考：https://docs.rs/glob/0.3.1

---

## 12. 总结

glob 库在 OpenHarmony 中的 API 和行为**与上游版本完全一致**：

**核心结论**:
- ✅ 无 API 差异
- ✅ 无行为变更
- ✅ 无废弃或禁用的功能
- ✅ 平台兼容性良好
- ✅ 性能无差异

**维护优势**:
- 零 Patch 集成
- 完全同步上游
- 升级风险低
- 使用简单

**使用建议**:
- 直接使用 glob API，无需考虑 OH 特殊情况
- 参考上游文档：https://docs.rs/glob/0.3.1
- 监控上游更新，及时同步

---

**最后更新**: 2026-02-08
