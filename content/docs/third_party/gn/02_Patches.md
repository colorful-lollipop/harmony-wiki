# 02 - Patch 详细分析

## Patch 清单

| Patch 文件 | 修改文件数 | 修改类型 | OH 特有 |
|-----------|-----------|----------|--------|
| `fd9f2036f26d83f9fcfe93042fb952e5a7fe2167.patch` | 3 个文件 | 功能简化 | 是 |

---

## Patch: fd9f2036f26d83f9fcfe93042fb952e5a7fe2167

### 基本信息

| 属性 | 值 |
|-----|---|
| **Commit Hash** | fd9f2036f26d83f9fcfe93042fb952e5a7fe2167 |
| **文件路径** | `patches/fd9f2036f26d83f9fcfe93042fb952e5a7fe2167.patch` |
| **修改范围** | `src/gn/filesystem_utils.cc`, `src/gn/ninja_c_binary_target_writer_unittest.cc`, `src/gn/substitution_writer_unittest.cc` |

### 修改文件列表

| 文件 | 修改行数 | 修改性质 |
|-----|---------|----------|
| `src/gn/filesystem_utils.cc` | -19 +5 | 核心逻辑修改 |
| `src/gn/ninja_c_binary_target_writer_unittest.cc` | -3 +3 | 单元测试更新 |
| `src/gn/substitution_writer_unittest.cc` | -6 +0 | 单元测试移除 |

---

### 详细分析

#### 1. src/gn/filesystem_utils.cc

**修改函数**: `GetSubBuildDirAsOutputFile()`

**原始逻辑**（已删除）:
```cpp
if (source_dir.is_source_absolute()) {
  std::string_view build_dir = context.build_settings->build_dir().value();
  std::string_view source_dir_path = source_dir.value();
  if (source_dir_path.substr(0, build_dir.size()) == build_dir) {
    // 源目录位于构建目录内（如生成的文件）
    // 使用 BUILD_DIR/ 前缀替代实际路径
    result.value().append("BUILD_DIR/");
    result.value().append(&source_dir_path[build_dir.size()],
                          source_dir_path.size() - build_dir.size());
  } else {
    // 源目录是源码绝对路径，去掉前两个斜杠
    result.value().append(&source_dir.value()[2],
                          source_dir.value().size() - 2);
  }
}
```

**新逻辑**:
```cpp
if (source_dir.is_source_absolute()) {
  // The source dir is source-absolute, so we trim off the two leading
  // slashes to append to the toolchain object directory.
  result.value().append(&source_dir.value()[2],
                        source_dir.value().size() - 2);
}
```

**关键变更**:
- **移除了 BUILD_DIR 占位符逻辑**：不再使用 `BUILD_DIR/` 作为构建目录内源文件的路径前缀
- **统一处理**：所有 source-absolute 路径统一去掉前两个斜杠（`//`）

**原始问题**: 为什么需要这个 Patch？

上游 GN 对位于构建目录内的生成文件（如 `//out/Debug/gen/foo.cc`）使用特殊的 `BUILD_DIR/` 前缀表示，这样做：
1. 使构建输出路径更短
2. 在不同构建目录配置间保持一致性

但在 OpenHarmony 中，这种表示方式可能带来问题：
1. **可读性**: `BUILD_DIR/` 是占位符，不如实际路径直观
2. **调试困难**: 实际路径更易于问题定位
3. **工具链兼容性**: 某些 OH 工具链可能不支持 BUILD_DIR 占位符

**OH 价值**:
- 统一使用实际路径（如 `out/Debug/`），提高构建输出的可读性
- 简化路径处理逻辑，减少潜在的错误场景
- 使 Ninja 文件中的路径与实际文件系统路径一致

#### 2. src/gn/ninja_c_binary_target_writer_unittest.cc

**变更内容**:

| 行 | 原始 | 修改后 |
|---|-----|-------|
| 408 | `build obj/BUILD_DIR/gen_obj.generated.o` | `build obj/out/Debug/gen_obj.generated.o` |
| 411 | `build obj/foo/gen_obj.stamp: stamp obj/BUILD_DIR/gen_obj.generated.o` | `build obj/foo/gen_obj.stamp: stamp obj/out/Debug/gen_obj.generated.o` |
| 445 | `build ./libgen_lib.so: solink obj/BUILD_DIR/gen_obj.generated.o` | `build ./libgen_lib.so: solink obj/out/Debug/gen_obj.generated.o` |

**说明**: 更新测试期望输出，将 `BUILD_DIR` 替换为实际的 `out/Debug`。

#### 3. src/gn/substitution_writer_unittest.cc

**变更内容**:

删除了以下测试用例：
```cpp
TEST(SubstitutionWriter, ApplyPatternToSource) {
  // ... 前面的测试 ...
  
  // 删除的测试用例:
  result = SubstitutionWriter::ApplyPatternToSource(
      nullptr, setup.settings(), pattern,
      SourceFile("//out/Debug/gen/generated_file.cc"));
  ASSERT_EQ("//out/Debug/gen/BUILD_DIR/gen/generated_file.tmp", result.value())
      << result.value();
}
```

**说明**: 该测试用例专门测试 BUILD_DIR 路径处理，由于 Patch 移除了该功能，对应测试也被删除。

---

### Patch 影响评估

#### 功能影响

| 方面 | 影响 |
|-----|------|
| **构建功能** | 无功能影响，仅路径表示方式变化 |
| **输出路径** | 构建输出文件路径从 `obj/BUILD_DIR/...` 变为 `obj/out/Debug/...` |
| **增量构建** | 首次应用 Patch 后需要重新生成 Ninja 文件 |

#### 回归风险

| 风险项 | 等级 | 说明 |
|-------|-----|------|
| **上游合并** | 中 | Patch 修改核心路径逻辑，升级上游版本时需重新适配 |
| **测试覆盖** | 低 | 删除了一个测试用例，可能降低该场景的测试覆盖 |
| **兼容性** | 低 | 仅影响构建输出路径表示，不影响运行时行为 |

#### 升级建议

1. **短期（维护当前版本）**:
   - 保留当前 Patch，文档化其目的
   - 在升级上游版本前进行充分测试

2. **中期（考虑上游化）**:
   - 评估是否将此变更推向上游
   - 或请求上游添加配置选项控制 BUILD_DIR 行为

3. **长期（版本升级）**:
   - 升级时检查 `filesystem_utils.cc` 的变更
   - 确保新的上游版本与 Patch 兼容
   - 如上游有重大变更，考虑重新实现 Patch

---

### 证据引用

Patch 文件完整内容：

```diff
diff --git a/src/gn/filesystem_utils.cc b/src/gn/filesystem_utils.cc
index 58d8cca..5f61443 100644
--- a/src/gn/filesystem_utils.cc
+++ b/src/gn/filesystem_utils.cc
@@ -1069,25 +1069,10 @@ OutputFile GetSubBuildDirAsOutputFile(const BuildDirContext& context,
   OutputFile result = GetBuildDirAsOutputFile(context, type);
 
   if (source_dir.is_source_absolute()) {
-    std::string_view build_dir = context.build_settings->build_dir().value();
-    std::string_view source_dir_path = source_dir.value();
-    if (source_dir_path.substr(0, build_dir.size()) == build_dir) {
-      // The source dir is source-absolute, but in the build directory
-      // (e.g. `//out/Debug/gen/src/foo.cc` or
-      // `//out/Debug/toolchain1/gen/foo.cc`), which happens for generated
-      // sources. In this case, remove the build directory prefix, and replace
-      // it with `BUILD_DIR`. This will create results like `obj/BUILD_DIR/gen`
-      // or `toolchain2/obj/BUILD_DIR/toolchain1/gen` which look surprising,
-      // but guarantee unicity.
-      result.value().append("BUILD_DIR/");
-      result.value().append(&source_dir_path[build_dir.size()],
-                            source_dir_path.size() - build_dir.size());
-    } else {
-      // The source dir is source-absolute, so we trim off the two leading
-      // slashes to append to the toolchain object directory.
-      result.value().append(&source_dir.value()[2],
-                            source_dir.value().size() - 2);
-    }
+    // The source dir is source-absolute, so we trim off the two leading
+    // slashes to append to the toolchain object directory.
+    result.value().append(&source_dir.value()[2],
+                          source_dir.value().size() - 2);
   } else {
     // System-absolute.
     AppendFixedAbsolutePathSuffix(context.build_settings, source_dir, &result);
```

完整 Patch 文件位于：`patches/fd9f2036f26d83f9fcfe93042fb952e5a7fe2167.patch`
