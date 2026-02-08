# heck Wiki 阅读路线

本文档提供 heck Wiki 的阅读建议，帮助不同需求的读者快速找到所需信息。

## 阅读路线建议

### 路线一：快速了解（5 分钟）
适合：想快速了解该库在 OH 中的作用的开发者

1. [README.md](./README.md) - 查看快速概览
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解依赖关系和使用场景

### 路线二：技术评估（15 分钟）
适合：需要评估该库维护成本和安全性的开发者

1. [01_Overview.md](./01_Overview.md) - 了解库的功能和定位
2. [02_Patches.md](./02_Patches.md) - 确认无 Patch，理解无需修改的原因
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看依赖者和使用场景

### 路线三：构建维护（20 分钟）
适合：需要维护 BUILD.gn 或升级版本的开发者

1. [03_Build_Integration.md](./03_Build_Integration.md) - 详细了解构建配置
2. [02_Patches.md](./02_Patches.md) - 确认升级时无需处理 Patch
3. [01_Overview.md](./01_Overview.md) - 查看版本和特性信息

### 路线四：完整阅读（30 分钟）
适合：需要全面了解的开发者

按顺序阅读所有文档：
1. [README.md](./README.md)
2. [01_Overview.md](./01_Overview.md)
3. [02_Patches.md](./02_Patches.md)
4. [03_Build_Integration.md](./03_Build_Integration.md)
5. [04_Usage_in_OH.md](./04_Usage_in_OH.md)

## 关键结论速查

| 问题 | 答案 |
|------|------|
| heck 是什么？ | Rust 字符串大小写转换库 |
| 需要 Patch 吗？ | **不需要**，无任何 Patch |
| 谁在用它？ | clap_derive（命令行解析宏） |
| 升级要注意什么？ | v0.4.0 有破坏性变更，trait 名称变了 |
| 安全吗？ | 完全安全，禁止 unsafe 代码 |
| Unicode 支持？ | 当前未启用 unicode 特性 |

## 相关链接

- **上游仓库**: https://github.com/withoutboats/heck
- **文档**: https://docs.rs/heck
- **Cargo 页面**: https://crates.io/crates/heck
- **Clap 文档**: https://docs.rs/clap (主要使用者)
