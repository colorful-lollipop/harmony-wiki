# Patch 详细分析

## 概述

Protobuf 库在 OpenHarmony 中的源码修改非常少，仅有 **1 个 Patch 文件**，且该 Patch 与 OpenHarmony 特有功能无关，是针对 Bazel 构建系统的通用修复。

## Patch 清单

| 序号 | Patch 文件 | 修改文件 | 修改目的 | 类型 | OH 关联 |
|------|------------|----------|----------|------|---------|
| 1 | `third_party/rules_fuzzing.patch` | `fuzzing/private/binary.bzl` | 修复 getattr 默认值兼容 | Buildfix | 低 |

## Patch 详细分析

### Patch 1: rules_fuzzing.patch

#### 基本信息

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `third_party/rules_fuzzing.patch` |
| **修改文件** | `fuzzing/private/binary.bzl` |
| **Patch 类型** | Bazel 构建规则修复 |
| **上游状态** | 可推向上游 |

#### 原始问题

Bazel 构建系统在不同版本中，`DefaultInfo.files_to_run` 对象的 `repo_mapping_manifest` 属性存在性不一致。较新版本的 Bazel 移除了该属性或返回 `None`，导致直接调用 `getattr()` 时抛出 `AttributeError`。

```python
# 原始代码问题
binary_repo_mapping_manifest = getattr(default_info.files_to_run, "repo_mapping_manifest")
# 当属性不存在时抛出: AttributeError: '... object' has no attribute 'repo_mapping_manifest'
```

#### 修改内容

```diff
diff --git a/fuzzing/private/binary.bzl b/fuzzing/private/binary.bzl
index 4c85aed..8ff9723 100644
--- a/fuzzing/private/binary.bzl
+++ b/fuzzing/private/binary.bzl
@@ -114,7 +114,7 @@ def _fuzzing_binary_impl(ctx):
      else:
          default_info = ctx.attr.binary[DefaultInfo]
      binary_runfiles = default_info.default_runfiles
-    binary_repo_mapping_manifest = getattr(default_info.files_to_run, "repo_mapping_manifest")
+    binary_repo_mapping_manifest = getattr(default_info.files_to_run, "repo_mapping_manifest", None)
      other_runfiles = []
      if ctx.file.corpus:
          other_runfiles.append(ctx.file.corpus)
```

#### OH 需求关联

**此 Patch 与 OpenHarmony 特有功能无直接关联**，属于：

1. **构建工具兼容性修复**：确保 protobuf 的 Bazel 模糊测试规则与不同版本 Bazel 兼容
2. **上游修复下沉**：可能是从上游 cherry-pick 的修复
3. **通用问题**：影响所有使用 Bazel 构建 protobuf 的项目

#### 升级建议

| 建议项 | 说明 |
|--------|------|
| **上游状态** | 建议检查上游 protobuf 29.x 版本是否已包含此修复 |
| **推向上游** | 可尝试将此 Patch 推向上游，属于通用构建修复 |
| **版本验证** | 升级到上游 29.4+ 时需验证是否需要此 Patch |
| **风险评估** | **低风险**，仅影响 Bazel 模糊测试构建路径 |

## OH 特有修改分析

### 源码修改统计

| 类型 | 数量 | 说明 |
|------|------|------|
| **#ifdef OHOS 宏** | 0 | 无源码层面的 OH 定制 |
| **OH 新增文件** | 0 | 无 OH 特有源文件 |
| **OH 定制函数** | 0 | 无 OH 特有功能函数 |

### 构建配置修改

所有 OpenHarmony 适配均通过 **BUILD.gn** 配置完成，包括：

| 配置项 | 值 | 用途 |
|--------|-----|------|
| `HAVE_HILOG` | 定义在 lite_static | OH 日志系统集成 |
| `HAVE_PTHREAD` | 多处定义 | POSIX 线程支持 |
| `branch_protector_ret` | "pac_ret" (lite) | ARM PAC-RET 安全加固 |

## Patch 维护策略

### 当前状态

| 维度 | 状态 |
|------|------|
| **Patch 总数** | 1 |
| **OH 特有 Patch** | 0 |
| **可推向上游 Patch** | 1 |
| **需要持续维护 Patch** | 0 |

### 升级注意事项

1. **检查上游合并状态**：确认 rules_fuzzing.patch 是否已合并到上游
2. **Bazel 版本兼容性**：模糊测试功能在 OH 中使用较少，风险可控
3. **回归测试**：升级后验证 Bazel 模糊测试构建路径

## 结论

Protobuf 库在 OpenHarmony 中保持**高度原生**，未对源码进行 OH 特有修改。所有适配均通过构建系统配置完成，这种方式：

✅ **优点**：
- 上游升级简单（无源码冲突）
- 维护成本低
- 符合上游最佳实践

⚠️ **注意事项**：
- ABI 兼容性需持续关注
- 大版本升级需全面测试
