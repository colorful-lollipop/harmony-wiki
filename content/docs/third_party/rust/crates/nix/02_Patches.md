# 02 Patch 分析

> 本文档记录 Nix 库在 OpenHarmony 中的 Patch 情况

## Patch 概览

**结论：该库在 OpenHarmony 中没有使用任何 Patch 文件。**

### Patch 清单

| Patch 文件 | 修改文件 | 修改目的 | 状态 |
|-----------|---------|---------|------|
| 无 | 无 | 无 | ✅ 无需 Patch |

---

## 无 Patch 的原因分析

### 1. 上游原生支持 OpenHarmony

Nix 上游代码已将 OpenHarmony (`target_os = "ohos"`) 作为 Tier 2 支持平台，在官方 README 中明确列出：

```
Tier 2 支持平台：
✅ aarch64-unknown-linux-ohos
✅ armv7-unknown-linux-ohos
✅ x86_64-unknown-linux-ohos
```

### 2. 条件编译机制

Nix 使用 `cfg-if` 和条件编译处理不同 Unix 变体的差异：

```rust
// src/lib.rs 中的条件编译示例
#[cfg(any(target_os = "linux", target_os = "android", target_os = "ohos"))]
pub mod inotify;

// OpenHarmony 与 Linux 共用相同的实现
#[cfg(target_os = "ohos")]
type ::std::os::unix::io::RawFd = libc::c_int;
```

### 3. POSIX 兼容性

OpenHarmony 的 POSIX 兼容层与标准 Linux 具有良好的 API 兼容性，Nix 封装的系统调用无需修改即可工作。

---

## 可能的 Patch 需求（未实现）

### 如果需要 OH 特有功能

虽然当前无需 Patch，但以下场景可能需要 OH 特有 Patch：

| 潜在需求 | Patch 目的 | 当前状态 |
|---------|-----------|---------|
| OH 特有的系统调用 | 添加 OpenHarmony 特有的系统 API | 未实现 |
| OH 安全机制集成 | 集成 OH 权限管理系统 | 未实现 |
| OH 设备交互 | 添加设备特有的控制接口 | 未实现 |

---

## Patch 管理建议

### 上游版本升级时的注意事项

1. **验证 OH 支持状态**
   - 检查上游 README 是否仍列出 OpenHarmony 支持
   - 验证 OH 目标平台的 CI 构建是否通过

2. **Features 配置检查**
   - 确认已启用的 features 在新版本中仍然可用
   - 检查是否有新增/废弃的 features

3. **API 兼容性**
   - 检查是否有破坏性变更 (breaking changes)
   - 验证 API 返回类型和错误处理方式的一致性

### 建议的 Patch 策略

| 场景 | 建议 |
|------|------|
| **上游 Bug 修复** | 优先向上游提交修复，而非本地 Patch |
| **OH 特有功能** | 评估是否可作为 conditional feature 添加 |
| **性能优化** | 评估上游是否接受优化建议 |

---

## 替代方案：无 Patch 的集成方式

### 条件编译使用

对于 OH 特有的功能需求，可以通过条件编译实现：

```rust
#[cfg(target_os = "ohos")]
mod ohos_specific {
    pub fn oh_permission_check() -> bool {
        // OH 权限检查逻辑
    }
}

#[cfg(not(target_os = "ohos"))]
mod ohos_specific {
    pub fn oh_permission_check() -> bool {
        true // 非 OH 平台直接放行
    }
}
```

### Feature 驱动扩展

如果需要大量 OH 特有功能，建议：

```toml
# Cargo.toml
[features]
default = ["standard_features"]

standard_features = [
    "process",
    "signal", 
    "socket",
    # ... 标准 features
]

ohos_extended = [
    "standard_features",
    # OH 特有 features
]
```

---

## 总结

| 评估项 | 结论 |
|--------|------|
| **Patch 数量** | 0 |
| **Patch 必要性** | 无需 Patch |
| **上游 OH 支持** | Tier 2 级别 |
| **维护复杂度** | 低 |
| **升级风险** | 低 |

由于上游已原生支持 OpenHarmony，该库的 OH 集成非常简洁，无需维护任何 Patch 文件。

---

**上一节**: [01_Overview.md](01_Overview.md)  
**下一节**: [03_Build_Integration.md](03_Build_Integration.md)
