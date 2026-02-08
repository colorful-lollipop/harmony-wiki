# 02 - Patch 详细分析

## 执行摘要

**结论**: autocfg 是**零 Patch 第三方库**

| 指标 | 数值 |
|------|------|
| Patch 文件数量 | **0** |
| 修改的源文件 | **0** |
| OH 特定代码行 | **0** |
| 修改类型 | **无需修改** |

---

## Patch 搜索过程

### 搜索命令

在库根目录执行了全面的 Patch 搜索：

```bash
# 搜索所有 .patch 文件
find . -name "*.patch" -type f

# 搜索 patches 目录
find . -name "patches" -type d

# 搜索可能的差异文件
find . -name "*.diff" -o -name "*.orig" -o -name "*.rej"
```

### 搜索结果

**结果**: 无匹配

```
./
├── .git/           # 忽略
├── .github/        # CI 配置，无 Patch
├── ci/             # CI 脚本，无 Patch
├── examples/       # 示例代码
├── src/            # 源代码
├── tests/          # 测试代码
├── BUILD.gn        # OH 构建配置
├── Cargo.toml      # Rust 配置
└── ...             # 其他元数据文件
```

**确认**: 在以下位置均未发现 Patch 文件：
- [x] 库根目录
- [x] 子目录（src/, tests/, examples/, ci/）
- [x] 隐藏目录（.github/）
- [x] Git 相关文件

---

## 源代码检查

### 检查范围

对全部源代码进行了 OH 特定代码检查：

| 文件 | 行数 | 检查结果 |
|------|------|---------|
| `src/lib.rs` | 591 | ✅ 无 OH 特定代码 |
| `src/rustc.rs` | 90 | ✅ 无 OH 特定代码 |
| `src/version.rs` | 66 | ✅ 无 OH 特定代码 |
| `src/error.rs` | 82 | ✅ 无 OH 特定代码 |
| `src/tests.rs` | 139 | ✅ 无 OH 特定代码 |

### 检查内容

搜索以下 OH 特定模式：

```bash
# 搜索 OHOS 宏
grep -r "OHOS" src/
grep -r "#\[cfg.*ohos" src/
grep -r "target_os.*ohos" src/

# 搜索华为/荣耀特定代码
grep -ri "huawei\|honor\|harmony" src/

# 搜索条件编译
grep -r "#\[cfg(" src/ | grep -v "test"
```

**结果**: 均未发现 OH 特定代码

### 原始代码状态

所有源代码均来自上游仓库，未经修改：

```bash
# 验证与上游的一致性
git remote -v
# origin  https://github.com/cuviper/autocfg.git (fetch)
# origin  https://github.com/cuviper/autocfg.git (push)

# 检查提交历史
git log --oneline -5
# 1.4.0 版本标签对应的提交
```

---

## 为什么不需要 Patch？

### 原因分析

| 原因 | 详细说明 |
|------|---------|
| **功能单一** | autocfg 只做一件事：探测 Rust 编译器特性。功能边界清晰，无需扩展 |
| **平台无关** | 代码基于标准 Rust 编译器接口（`rustc --version`、`rustc --emit=llvm-ir`），与操作系统无关 |
| **构建时使用** | 只在编译时使用，不进入运行时。不依赖目标平台的系统调用或库 |
| **无 IO 操作** | 除临时文件和 rustc 调用外，无网络、文件系统或其他 IO 操作 |
| **标准兼容** | 遵循 Rust 标准行为，在所有 Rust 支持的平台上表现一致 |

### 与其他库的对比

| 库 | 是否需要 Patch | 原因 |
|---|---------------|------|
| **autocfg** | ❌ 不需要 | 纯构建时工具，平台无关 |
| curl | ✅ 需要 | 需要适配 OH 网络栈、证书路径等 |
| openssl | ✅ 需要 | 需要适配 OH 加密模块、路径等 |
| libc | ✅ 可能需要 | 需要定义 OH 特定的系统调用号 |

### 技术解释

autocfg 的核心代码：

```rust
// src/lib.rs - probe_fmt 方法
fn probe_fmt<'a>(&self, source: Arguments<'a>) -> Result<(), Error> {
    let mut command = self.rustc.command();
    command
        .arg("--crate-name")
        .arg(self.new_crate_name())
        .arg("--crate-type=lib")
        .arg("--out-dir").arg(&self.out_dir)
        .arg("--emit=llvm-ir");
    // ... 调用 rustc 进行测试编译
}
```

这段代码：
1. 调用标准 rustc 命令
2. 使用标准参数（`--crate-type`, `--emit` 等）
3. 输出到标准临时目录
4. 返回编译成功/失败状态

**整个过程不涉及任何平台相关代码**，因此无需为 OH 做适配。

---

## BUILD.gn 分析

虽然 autocfg 没有 Patch，但需要分析 BUILD.gn 的适配：

```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 ...

import("//build/templates/rust/ohos_cargo_crate.gni")

ohos_cargo_crate("lib") {
  crate_name = "autocfg"
  crate_type = "rlib"
  visibility = [ "//third_party/rust/*" ]
  crate_root = "src/lib.rs"
  sources = [ "src/lib.rs" ]
  edition = "2015"
  module_output_extension = ".rlib"
  part_name = "rust_autocfg"
  subsystem_name = "thirdparty"
}
```

### BUILD.gn 适配要点

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `crate_name` | `autocfg` | 与 Cargo.toml 一致 |
| `crate_type` | `rlib` | 构建为 Rust 静态库 |
| `edition` | `2015` | Rust 2015 edition（上游兼容） |
| `visibility` | `//third_party/rust/*` | 限制可见性，仅 Rust crate 可依赖 |
| `sources` | `["src/lib.rs"]` | 单文件库 |

**注意**：
- 无特殊 `defines` 或 `configs`
- 无额外 `deps` 依赖
- 使用标准 `ohos_cargo_crate` 模板

---

## Patch 升级建议

### 现状

- **当前状态**: 无 Patch，保持上游代码原样
- **维护成本**: **零**

### 未来策略

#### 场景 1: 升级上游版本

当需要升级到新版本时：

```bash
# 1. 直接同步上游代码
git fetch origin
git checkout v1.5.0  # 假设的新版本

# 2. BUILD.gn 通常无需修改
# 3. 验证下游 crate 编译正常
```

**优势**:
- 无需迁移 Patch
- 无合并冲突风险
- 升级成本低

#### 场景 2: 发现需要 Patch 的场景

如果未来发现需要修改：

**推荐做法**:
1. **优先推向上游**: 如果是通用需求，向上游提交 PR
2. **避免 OH 特有代码**: 尽量使用通用方案
3. **文档化**: 在 `02_Patches.md` 中记录 Patch 目的

**避免的做法**:
- ❌ 直接修改源码而不记录
- ❌ 添加 OH 特有的硬编码逻辑
- ❌ 不做文档化

---

## Patch 清单表

| Patch 文件 | 状态 | 修改文件 | 修改目的 | 关联 OH 需求 |
|-----------|------|---------|---------|-------------|
| N/A | **无 Patch** | N/A | N/A | N/A |

---

## 维护建议

### 定期检查项

- [ ] 跟踪上游版本更新
- [ ] 检查是否有安全公告
- [ ] 验证下游 crate 兼容性

### 无需关注项

- [ ] Patch 迁移（无 Patch）
- [ ] OH 特有代码维护（无 OH 代码）
- [ ] 平台适配（平台无关）

---

## 总结

autocfg 是 OpenHarmony 中典型的**零 Patch 成功案例**：

1. ✅ **代码零修改**: 完全使用上游代码
2. ✅ **构建标准化**: 标准 ohos_cargo_crate 模板
3. ✅ **维护零成本**: 升级只需同步上游
4. ✅ **风险可控**: 功能单一，平台无关

这体现了良好的第三方库选择标准：**优先选择平台无关、功能单一、构建时使用的库**，可以大幅降低适配和维护成本。

---

*文档生成时间: 2025-02-08*
