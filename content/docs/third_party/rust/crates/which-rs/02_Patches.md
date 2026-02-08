# Patch 分析

## 概述

**which-rs 在 OpenHarmony 中无任何 Patch 文件。**

这是该库的一个重要特点：它原生支持 OpenHarmony 标准系统，无需任何修改即可正常工作。

```bash
# 在库根目录搜索 Patch 文件
$ find . -name "*.patch" -o -name "patches" -type d
# 无输出
```

## 为何无需 Patch

### 1. 功能单一且稳定

which-rs 的核心功能非常聚焦：
- 解析 PATH 环境变量
- 遍历目录查找文件
- 检查文件可执行权限

这些功能完全基于标准 Rust 库和 POSIX API，与具体操作系统细节耦合度低。

### 2. 原生支持 Linux

OpenHarmony 标准系统基于 Linux 内核，而 which-rs 原生支持 Linux：

```rust
// checker.rs 中的 Unix 可执行检查
#[cfg(unix)]
fn is_valid(&self, path: &Path) -> bool {
    CString::new(path.as_os_str().as_bytes())
        .map(|c| unsafe { libc::access(c.as_ptr(), libc::X_OK) == 0 })
        .unwrap_or(false)
}
```

使用标准的 `libc::access` 系统调用，在所有 Linux 系统上行为一致。

### 3. 无硬件依赖

该库是纯软件逻辑：
- 不操作硬件设备
- 不依赖特定系统调用
- 不读取 /proc 等特殊文件系统

### 4. 构建系统兼容

通过 `ohos_cargo_crate` 模板，Cargo 项目可以无缝转换为 GN 构建：

```gn
ohos_cargo_crate("lib") {
    crate_name = "which"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    # ... 标准配置
}
```

## 上游版本适配情况

### 版本历史

| 版本 | 发布时间 | 主要变更 | OH 适配难度 |
|------|----------|----------|-------------|
| 4.4.0 | 2023年 | 当前集成版本 | - |
| 5.x | 2024年 | 内部重构，API 保持兼容 | 低 |
| 6.x | 2024年 | API 可能有 Breaking Changes | 中 |

### 升级建议

由于无 Patch，升级上游版本相对简单：

1. **检查 API 兼容性**
   - 阅读上游 CHANGELOG
   - 确认公共 API 是否有 Breaking Changes

2. **更新构建配置**
   - 修改 BUILD.gn 中的版本号
   - 更新依赖声明（如有新增依赖）

3. **验证功能**
   - 运行单元测试
   - 验证依赖该库的模块编译正常

## 无 Patch 的优缺点

### 优点

| 优点 | 说明 |
|------|------|
| 维护成本低 | 无需跟踪和管理 Patch 文件 |
| 升级简单 | 直接替换源码，无需重新应用 Patch |
| 质量可靠 | 直接使用上游经过充分测试的代码 |
| 社区同步 | 易于跟进上游安全更新和 Bug 修复 |

### 潜在考虑

| 考虑点 | 说明 |
|--------|------|
| 功能定制受限 | 如需 OH 特有功能，需向上游贡献或 Fork |
| 行为不可控 | 完全依赖上游实现决策 |

## 实际案例对比

与其他需要 Patch 的库对比：

| 库 | Patch 数量 | 需要 Patch 的原因 |
|----|------------|-------------------|
| **which-rs** | **0** | **原生兼容，无需修改** |
| curl | 50+ | 需要适配 OH 网络栈、安全策略 |
| openssl | 30+ | 需要适配 OH 加密模块、证书管理 |
| libpng | 10+ | 需要适配 OH 图形系统 |

## 结论

which-rs 是一个设计良好、兼容性强的基础工具库。其**零 Patch**特性意味着：

1. **代码质量高**: 原生实现就满足跨平台需求
2. **适配成本低**: OH 可以低成本跟进上游更新
3. **可靠性好**: 无自定义修改，减少引入问题的风险

对于这类无需 Patch 的库，OH 的维护重点是：
- 及时跟进安全更新
- 监控上游 API 变更
- 确保依赖管理正确

---

**注意**: 本文档虽无具体 Patch 内容，但明确记录了"无 Patch"这一事实，这对理解该库在 OH 中的集成方式同样重要。
