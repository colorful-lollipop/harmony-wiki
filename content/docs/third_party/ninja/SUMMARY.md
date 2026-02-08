# Ninja Wiki 文档导航

## 文档概览

本文档描述了 **Ninja** 在 OpenHarmony `third_party` 目录中的集成情况。Ninja 是一个专注于速度的构建系统，OpenHarmony 将其作为构建工具使用。

## 阅读路线建议

| 角色 | 推荐阅读顺序 | 重点关注 |
|------|-------------|---------|
| **构建系统开发者** | README → 03 → 04 | Ninja 如何被 OH 构建系统调用 |
| **版本维护人员** | README → 02 → 03 | Patch 情况和升级注意事项 |
| **安全管理人员** | README → 06 | 安全公告和风险评估 |
| **新贡献者** | 01 → 04 → 03 | 了解 Ninja 基础和 OH 使用方式 |

## 文档索引

### 核心文档

| 文档 | 说明 | 必读 |
|------|------|------|
| **[README.md](./README.md)** | Ninja 库概览和 OpenHarmony 适配概述 | ✅ 是 |
| **[01_Overview.md](./01_Overview.md)** | Ninja 原始库的功能介绍和上游信息 | ✅ 是 |
| **[02_Patches.md](./02_Patches.md)** | OpenHarmony 对 Ninja 的 Patch 分析 | ⚠️ 简要 |
| **[03_Build_Integration.md](./03_Build_Integration.md)** | OH 构建系统对 Ninja 的集成方式 | ✅ 是 |
| **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** | Ninja 在 OpenHarmony 中的使用场景 | ✅ 是 |

### 工作文档

| 文档 | 说明 |
|------|------|
| **[\_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** | 库评估原始记录（供维护者参考） |

## 快速定位

### 我想了解...

- **Ninja 是什么？** → [01_Overview.md](./01_Overview.md)
- **OH 对 Ninja 做了什么修改？** → [02_Patches.md](./02_Patches.md)（答案是：没有修改）
- **Ninja 如何被 OH 构建？** → [03_Build_Integration.md](./03_Build_Integration.md)
- **OH 哪个模块使用了 Ninja？** → [04_Usage_in_OH.md](./04_Usage_in_OH.md)
- **如何升级 Ninja 版本？** → [02_Patches.md](./02_Patches.md) 和 [03_Build_Integration.md](./03_Build_Integration.md)

## 版本信息

| 项目 | 值 |
|------|-----|
| **Ninja 版本** | v1.12.0 |
| **OH 组件版本** | 3.1 |
| **文档更新日期** | 2024 |
| **最后验证** | [待填写] |

## 相关链接

- **上游项目**：https://github.com/ninja-build/ninja
- **官方文档**：https://ninja-build.org/manual.html
- **OpenHarmony 构建系统**：build/hb/
- **OH third_party 根目录**：/third_party/
