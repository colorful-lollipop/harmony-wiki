# which-rs Wiki 阅读指南

本文档提供 which-rs 在 OpenHarmony 中的 Wiki 阅读路线建议。

## 阅读路线图

### 路径一：快速了解（5 分钟）

适合想快速了解该库的读者：

1. **[README.md](./README.md)** - 库概览和文档导航
2. **[01_Overview.md](./01_Overview.md)** - 功能简介和 OH 定位
   - 了解 which-rs 是什么
   - 在 OH 中的作用
   - 核心功能模块

### 路径二：开发使用（10 分钟）

适合需要在项目中使用该库的开发者：

1. **[01_Overview.md](./01_Overview.md)** - 基础了解
2. **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建配置
   - BUILD.gn 详解
   - 如何在自己的项目中引用
3. **[05_API_Differences.md](./05_API_Differences.md)** - API 参考
   - 可用 API 列表
   - 与上游的差异
   - 代码示例

### 路径三：维护分析（15 分钟）

适合维护人员或需要深度了解该库的读者：

1. **[README.md](./README.md)** - 概览
2. **[02_Patches.md](./02_Patches.md)** - Patch 分析
   - 为何无 Patch
   - 升级建议
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建系统
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 依赖关系
   - 谁在依赖该库
   - 使用场景分析
5. **[06_Security.md](./06_Security.md)** - 安全分析
   - 风险评估
   - 安全使用建议

### 路径四：完整审计（30 分钟）

适合进行完整技术审计：

按顺序阅读所有文档：
1. [README.md](./README.md)
2. [01_Overview.md](./01_Overview.md)
3. [02_Patches.md](./02_Patches.md)
4. [03_Build_Integration.md](./03_Build_Integration.md)
5. [04_Usage_in_OH.md](./04_Usage_in_OH.md)
6. [05_API_Differences.md](./05_API_Differences.md)
7. [06_Security.md](./06_Security.md)
8. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 原始评估报告
9. [_work/NOTES.md](./_work/NOTES.md) - 分析过程

## 文档依赖关系

```
README.md
    ├── 01_Overview.md（基础）
    │       └── 03_Build_Integration.md（构建）
    │       └── 05_API_Differences.md（API）
    ├── 02_Patches.md（Patch 分析）
    ├── 04_Usage_in_OH.md（依赖关系）
    └── 06_Security.md（安全）
```

## 关键信息速查

### 基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | which-rs |
| **上游版本** | 4.4.0 |
| **OH 组件名** | rust_which_rs |
| **Patch 数量** | **0** |

### 快速链接

- [上游仓库](https://github.com/harryfei/which-rs)
- [crates.io](https://crates.io/crates/which)
- [API 文档](https://docs.rs/which/)

### 常用 API

```rust
// 基础查找
which("rustc")?;

// 查找所有匹配
which_all("python")?;

// 在指定路径查找
which_in("tool", Some("/opt/bin:/usr/bin"), ".")?;

// 获取规范化路径（推荐，更安全）
CanonicalPath::new("tool")?;
```

## 问题排查

### which-rs 找不到命令？

1. 检查 PATH 环境变量：
   ```rust
   println!("PATH: {:?}", std::env::var("PATH"));
   ```

2. 检查文件是否真的可执行（Unix 权限）

3. 使用绝对路径验证：
   ```rust
   which("/absolute/path/to/binary")?;
   ```

### 在 OH 中无法使用 regex 功能？

OH 构建未启用 regex feature。如需正则匹配，请：
1. 使用标准库或 regex crate 手动过滤结果
2. 或修改 BUILD.gn 启用 regex feature（需引入 regex crate）

### bindgen 依赖 which-rs 但实际未使用？

正确。bindgen 的 BUILD.gn 声明了 which-rs 依赖，但实际代码未调用。
这是遗留依赖，建议在未来升级 bindgen 时清理。

---

**建议**: 根据您的角色和目的选择合适的阅读路径，不必一次性阅读所有文档。
