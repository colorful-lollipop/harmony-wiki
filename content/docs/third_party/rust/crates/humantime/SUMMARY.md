# 阅读路线建议

本 Wiki 按照以下路线组织，建议按顺序阅读：

## 快速了解（5分钟）

1. **[README.md](./README.md)** - 库概览与导航
2. **[01_Overview.md](./01_Overview.md)** - 快速了解原始库和 OH 定位

## 深度理解（15分钟）

3. **[02_Patches.md](./02_Patches.md)** - 理解 OH 对该库的修改（本库无 Patch）
4. **[03_Build_Integration.md](./03_Build_Integration.md)** - 了解 GN 构建适配

## 系统视角（10分钟）

5. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 了解谁在用它、怎么用
6. **[06_Security.md](./06_Security.md)** - 了解安全考量

## 按角色推荐阅读

### 如果你是...

| 角色 | 推荐阅读 | 原因 |
|------|----------|------|
| **Rust 开发者** | 01_Overview + 04_Usage_in_OH | 了解如何在 OH 中使用 |
| **构建系统维护者** | 03_Build_Integration + 02_Patches | 了解构建集成方式 |
| **安全工程师** | 06_Security + 02_Patches | 了解安全风险 |
| **新接触 OH 的开发者** | README + 01_Overview | 快速建立整体认知 |

## 关键结论速览

- ✅ **零 Patch**: 本库在 OH 中未做任何修改
- ✅ **构建简单**: 标准 GN 模板集成
- ✅ **用途单一**: 主要用于日志时间戳格式化
- ✅ **升级友好**: 可直接替换上游版本
