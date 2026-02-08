# Patch 分析

## Patch 清单

**本库在 OpenHarmony 中没有任何 Patch 文件**。

## 为什么不需要 Patch

proc-macro2 是一个纯 Rust 实现的基础设施库，不涉及平台特定代码，因此不需要进行 OH 特定修改。

### 原因分析

| 原因 | 说明 |
|------|------|
| **纯 Rust 实现** | 不依赖 C/C++ 代码，无平台特定逻辑 |
| **API 稳定** | 与 Rust 编译器版本解耦，不受编译器变更影响 |
| **核心功能** | OH 使用的功能是库的标准功能，无需修改 |
| **无外部依赖** | 唯一的运行时依赖是 unicode-ident（纯 Rust） |

### OH 使用的功能

OH 中启用的 features:

```toml
[features]
proc-macro = []      # 启用过程宏 API 支持（默认开启）
span-locations = []  # 启用位置信息（用于代码诊断）
```

这些 features 都是上游标准配置，无需 Patch。

## 如果需要添加 Patch

如果未来需要添加 OH 特定功能，应遵循以下流程:

### 1. Patch 文件命名规范

```
0001-功能描述.patch
```

### 2. Patch 内容结构

```diff
--- a/src/lib.rs
+++ b/src/lib.rs
@@ -1,3 +1,7 @@
+// OH-specific: 添加 xxx 功能
+#[cfg(OHOS)]
+mod ohos_adapter;
+
 use std::process::ExitCode;
```

### 3. 提交要求

- 每个 Patch 必须有清晰的 commit message
- 包含 OH 需求关联（如 JIRA 任务号）
- 说明修改目的和预期效果

## Patch 管理建议

### 与上游同步策略

由于本库无 Patch，与上游同步时:

1. **直接替换**: 将 OH BUILD.gn 配置应用到新版本
2. **版本升级**: 更新 `cargo_pkg_version` 即可
3. **回归测试**: 运行 OH Rust crates 测试确保兼容性

### 监控上游变更

建议监控上游的以下变更:

- API 签名变更（可能影响 syn/quote 等下游库）
- Cargo.toml 依赖变更
- Rust edition 要求变更

## 总结

| 项目 | 状态 |
|------|------|
| **Patch 文件数** | 0 |
| **OH 特有修改** | 无 |
| **可升级性** | 高（直接使用上游） |
| **维护成本** | 低 |
