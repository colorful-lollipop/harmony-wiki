# foreign-types OpenHarmony 适配文档

本文档记录 OpenHarmony 对 `foreign-types` Rust crate 的集成与适配情况。

> **foreign-types** 是一个用于 Rust FFI 编程的框架，专门为 C API 创建安全的 Rust 包装器。

---

## 📋 文档导航

### 核心文档
| 文档 | 说明 | 优先级 |
|------|------|--------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 中的定位 | 中 |
| [02_Patches.md](./02_Patches.md) | **Patch 详细分析（核心）** | **高** |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配 | **高** |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | 中 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异 | 低 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | 中 |

### 工作文档
| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估结果（Phase 0 输出） |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度跟踪 |

---

## 🎯 快速开始

### 对于 OH 开发者
如果你需要了解：
- **OH 如何使用这个库** → 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md)
- **OH 有哪些定制修改** → 阅读 [02_Patches.md](./02_Patches.md)
- **如何在 OH 中构建** → 阅读 [03_Build_Integration.md](./03_Build_Integration.md)

### 对于维护者
如果你需要：
- **升级上游版本** → 阅读 [02_Patches.md](./02_Patches.md) 和 [03_Build_Integration.md](./03_Build_Integration.md)
- **了解依赖关系** → 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md)
- **评估升级风险** → 阅读 [06_Security.md](./06_Security.md)

---

## 📊 库信息概览

| 属性 | 值 |
|------|-----|
| **上游仓库** | https://github.com/sfackler/foreign-types |
| **上游版本** | 0.3.2 |
| **OH 组件版本** | 6.1 |
| **许可证** | Apache-2.0 / MIT (双许可) |
| **OH 子系统** | thirdparty |
| **组件名** | @ohos/rust_foreign_types |

---

## 🔧 OH 适配状态

| 维度 | 状态 | 说明 |
|------|------|------|
| **源代码修改** | 🟢 无 | 与上游完全一致 |
| **Patch 文件** | 🟢 无 | 仅通过 Git 新增构建文件 |
| **BUILD.gn 适配** | 🟢 完成 | 2 个 BUILD.gn 文件 |
| **使用复杂度** | 🟢 低 | 仅 rust-openssl 使用 |
| **升级风险** | 🟢 低 | 零侵入，仅需更新版本号 |

---

## 📝 阅读路线建议

### 第一次阅读（了解概况）
1. 📖 [01_Overview.md](./01_Overview.md) - 了解库的基本功能
2. 🔍 [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解 OH 中的使用场景

### 深入分析（维护者）
1. 📖 [01_Overview.md](./01_Overview.md)
2. 🔧 [02_Patches.md](./02_Patches.md) - **重点**
3. 🔧 [03_Build_Integration.md](./03_Build_Integration.md) - **重点**
4. 📊 [04_Usage_in_OH.md](./04_Usage_in_OH.md)
5. 🔒 [06_Security.md](./06_Security.md)

### 快速查找（解决问题）
- **构建失败** → [03_Build_Integration.md](./03_Build_Integration.md)
- **依赖冲突** → [04_Usage_in_OH.md](./04_Usage_in_OH.md)
- **安全问题** → [06_Security.md](./06_Security.md)
- **API 不匹配** → [05_API_Differences.md](./05_API_Differences.md)

---

## 📌 关键发现

### OH 适配特点
1. **零侵入修改**：所有 Rust 源文件与上游完全一致
2. **仅构建适配**：OH 仅添加了 GN 构建配置文件
3. **单一直接依赖**：主要被 `rust-openssl` 使用
4. **低维护成本**：升级上游版本仅需更新 BUILD.gn 版本号

### 主要风险
- 🔴 **待确认**：已知 CVE 和修复状态
- 🟡 **版本差异**：OH 使用 0.3.2，可能与上游最新版本有差异

---

## 📚 参考资源

| 资源 | 链接 |
|------|------|
| 上游 GitHub | https://github.com/sfackler/foreign-types |
| crates.io | https://crates.io/crates/foreign-types |
| API 文档 | https://docs.rs/foreign-types |
| OH 仓库 | third_party/rust/crates/foreign-types |

---

## 🤝 贡献与反馈

本文档由 OpenHarmony Wiki 生成 Agent 自动生成，如有问题或建议，请联系 OH 维护者。

**维护者**: xuelei3@huawei.com
