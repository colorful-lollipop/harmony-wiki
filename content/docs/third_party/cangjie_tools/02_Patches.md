# Patch 详细分析

本文档说明 cangjie_tools 在 OpenHarmony 中的 Patch 文件情况。

---

## Patch 清单

| Patch 文件 | 修改文件 | 修改函数 | 修改目的 | 关联的 OH 需求 |
|-----------|----------|----------|----------|----------------|
| **无** | - | - | - | - |

---

## 结论：该仓库不使用 Patch 文件

### 维护策略

OpenHarmony 的 `third_party/cangjie_tools` 仓库**不使用 patch 文件**管理代码修改，而是采用 **直接同步上游完整代码** 的方式。

### Git 维护记录

根据 git 历史记录分析，该仓库的维护方式如下：

**最近提交历史**：
```
4c7b843d !103 merge auto-sync-cangjie_tools-20260205160047 into master
bca17419 feat(cangjie_tools): sync from upstream v1.1.0-alpha.69
1544ed23 !102 merge auto-sync-cangjie_tools-20260133170343 into master
2772ee9c feat(cangjie_tools): sync from upstream v1.1.0-alpha.68
...
```

**维护机制**：
- 使用 `auto-sync-cangjie_tools-YYYYMMDDhhmmss` 格式的合并提交
- 从上游 `cangjie_tools` 同步版本（如 v1.1.0-alpha.69）
- 通过 `!{number}` 格式的 MR 合并到 master 分支
- **版本号直接对应上游版本**，如 v1.1.0-alpha.69

### 代码变更处理方式

对于 OpenHarmony 特定的修改，该仓库采用以下方式处理：

1. **直接修改源文件**：如有 OH 特定需求，直接修改源文件（而非通过 patch）
2. **条件编译**：使用 `#ifdef __OHOS__` 或 `Feature.OS_OHOS` 等宏控制 OH 特定代码
3. **Git Commit 跟踪**：所有变更通过 git commit 跟踪记录

---

## 与上游的差异

### 差异来源

虽然不使用 patch 文件，但该仓库可能包含与上游的差异，主要来源：

1. **OH 特定适配**：
   - OpenHarmony 目标平台支持（`ohos-x86_64`, `ohos-aarch64`）
   - DevEco Studio 深度集成
   - ArkTS 互操作支持
   - 条件编译支持

2. **第三方依赖适配**：
   - 使用 OpenHarmony 官方版本的三方库（`OpenHarmony-v6.0-Release` 分支）

3. **构建系统适配**：
   - OH 特定的编译器和链接器选项
   - 安全编译选项

### 差异管理

如需查看与上游的差异，可使用以下方法：

```bash
# 1. 克隆上游仓库
git clone https://cangjie-lang.cn/your-upstream-repo.git

# 2. 对比差异
git diff --no-index /path/to/upstream /path/to/oh/third_party/cangjie_tools

# 或使用 git remote
git remote add upstream https://cangjie-lang.cn/your-upstream-repo.git
git fetch upstream
git diff upstream/main
```

---

## Patch 相关的搜索结果

### 搜索方式

| 搜索方式 | 命令 | 结果 |
|---------|-------|------|
| find patch 文件 | `find . -name "*.patch"` | 0 个文件 |
| find patches 目录 | `find . -name "patches" -type d` | 0 个目录 |
| find diff 文件 | `find . -name "*.diff"` | 0 个文件 |
| glob patch 文件 | `glob **/*.patch` | No files found |
| find 文件名含 patch | `find . -name "*patch*"` | 0 个文件 |

### 结论

所有搜索方式均未发现任何 patch 文件或 patches 目录。

---

## 升级建议

### 升级上游版本

如需升级到上游新版本，流程如下：

1. **同步上游代码**：
   ```bash
   # 通过自动同步脚本（推荐）
   # 或手动同步
   git fetch upstream
   git merge upstream/vX.Y.Z
   ```

2. **解决冲突**（如有）：
   - 检查 OH 特定修改是否冲突
   - 保留 OH 特定代码（如 `#ifdef __OHOS__` 块）
   - 更新第三方依赖版本（如需）

3. **测试验证**：
   - 运行各工具的测试用例
   - 验证 OH 特定功能（DevEco 集成、ArkTS 互操作等）
   - 确认编译通过

4. **提交合并**：
   ```bash
   git commit -m "feat(cangjie_tools): sync from upstream vX.Y.Z"
   ```

### 回退策略

如升级后发现严重问题，可快速回退：

```bash
# 回退到上一个稳定版本
git revert <commit-id>

# 或直接回退到特定提交
git reset --hard <stable-commit-id>
```

---

## 常见问题

### Q1: 为什么不使用 patch 文件？

**A**：cangjie_tools 是华为自研的 Cangjie 语言工具链，与 OpenHarmony 深度集成。采用直接同步上游代码的方式可以：

1. 简化维护流程，避免 patch 管理的复杂性
2. 快速同步上游更新和 bug 修复
3. 保持与上游版本的一致性
4. 通过条件编译（`#ifdef __OHOS__`）控制 OH 特定代码

### Q2: 如何了解 OH 特定的修改？

**A**：可以通过以下方式：

1. **搜索 OH 特定宏**：
   ```bash
   grep -r "__OHOS__\|OHOS\|os.ohos" --include="*.cpp" --include="*.cj" .
   ```

2. **查看 git log**：
   ```bash
   git log --oneline --all | grep -i "ohos\|ohos\|openharmony"
   ```

3. **对比上游差异**：
   ```bash
   git diff upstream/main
   ```

### Q3: 如需添加 OH 特定功能，如何处理？

**A**：推荐方式：

1. **直接修改源文件**：在源文件中添加 OH 特定代码
2. **使用条件编译**：使用 `#ifdef __OHOS__` 或 `@When[os == "ohos"]` 控制
3. **提交合并**：通过 git commit 提交，并注明 OH 特定需求
4. **推向上游**（如适用）：如功能通用，可推向上游仓库

### Q4: 如何验证 OH 特定代码的正确性？

**A**：验证方法：

1. **编译验证**：在 OH 目标平台（ohos-x86_64, ohos-aarch64）编译
2. **测试验证**：运行 OH 特定测试用例（如 `test/testChr/crossLanguageDefinition/ohos/`）
3. **DevEco 集成测试**：验证 LSP 在 DevEco Studio 中的功能
4. **互操作测试**：验证 HLE 生成的代码正确性

---

## 总结

### 核心要点

1. **无 Patch 文件**：该仓库不使用 patch 文件管理修改
2. **直接同步策略**：采用直接同步上游完整代码的方式
3. **条件编译控制**：通过 `__OHOS__` 等宏控制 OH 特定代码
4. **版本对应关系**：OH 版本号直接对应上游版本

### 维护建议

1. **定期同步上游**：保持与上游版本同步，获取最新功能和 bug 修复
2. **保留 OH 特定代码**：同步时注意保留 OH 特定代码块
3. **充分测试**：升级后充分测试 OH 特定功能
4. **文档记录**：记录 OH 特定修改和升级注意事项

---

## 参考文档

- [README.md](./README.md) - 库概览和 OH 适配概述
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估结果
- [../README.md](../README.md) - 项目主文档
