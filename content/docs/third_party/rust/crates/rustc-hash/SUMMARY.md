# 阅读指南

本文档旨在全面介绍 rustc-hash 库在 OpenHarmony 生态系统中的集成与适配情况。

## 文档结构

```
wiki/
├── README.md              # 快速入门和文档导航
├── SUMMARY.md             # 本文档，阅读指南
├── 01_Overview.md         # 库概述和 OH 定位
├── 02_Patches.md          # Patch 分析（本库无 Patch）
├── 03_Build_Integration.md # OH 构建适配详情
├── 04_Usage_in_OH.md      # 依赖关系和使用场景
└── _work/
    ├── ASSESSMENT.md      # 完整项目评估报告
    ├── NOTES.md           # 分析过程记录
    └── PLAN.md            # 任务进度追踪
```

## 推荐阅读路线

### 快速了解（5 分钟）
1. 阅读 [README.md](./README.md) 了解库的基本信息
2. 阅读 [01_Overview.md](./01_Overview.md) 理解该库在 OH 中的定位

### 深入了解（15 分钟）
完成上述快速了解后，继续阅读：
3. [03_Build_Integration.md](./03_Build_Integration.md) 了解构建适配细节
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) 了解使用场景和依赖关系

### 详细参考（30 分钟）
如需完整的技术细节，请阅读：
5. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 完整的项目评估报告
6. [06_Security.md](./06_Security.md) 安全风险分析

## 关键信息速查

| 问题 | 答案 |
|------|------|
| 该库是否有 OH 定制 Patch？ | **否**，该库完全使用上游代码 |
| 主要依赖者是谁？ | bindgen（Rust FFI 绑定生成工具） |
| 是否需要特殊配置？ | **否**，构建配置与上游基本一致 |
| 安全风险如何？ | **低**，无已知 CVE，未引入新攻击面 |

## 相关链接

- **上游仓库**: https://github.com/rust-lang/rustc-hash
- ** crates.io**: https://crates.io/crates/rustc-hash
- **官方文档**: https://docs.rs/rustc-hash/latest/rustc_hash/
- **OH 组件配置**: [bundle.json](../bundle.json)
- **构建配置**: [BUILD.gn](../BUILD.gn)
