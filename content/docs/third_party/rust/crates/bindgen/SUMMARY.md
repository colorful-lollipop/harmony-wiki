# bindgen - 阅读路线建议

## 根据角色的阅读指南

### 1. 构建系统开发者 / 集成工程师

**目标**: 理解如何在 BUILD.gn 中使用 bindgen

**阅读顺序**:
1. [README.md](README.md) - 了解整体概况
2. [03_Build_Integration.md](03_Build_Integration.md) - 深入理解 rust_bindgen 模板
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看实际使用案例

### 2. Rust 组件开发者

**目标**: 为自己的 Rust 组件添加 C/C++ 绑定

**阅读顺序**:
1. [README.md](README.md) - 快速入门
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) 中的 "使用示例" 章节
3. [03_Build_Integration.md](03_Build_Integration.md) 中的 "rust_bindgen 模板参数"

### 3. 安全审计人员

**目标**: 评估 bindgen 的安全风险

**阅读顺序**:
1. [06_Security.md](06_Security.md) - 安全风险分析
2. [README.md](README.md) - 了解组件定位
3. [03_Build_Integration.md](03_Build_Integration.md) - 理解构建时行为

### 4. 维护者 / 升级负责人

**目标**: 升级 bindgen 版本或排查问题

**阅读顺序**:
1. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 评估报告
2. [03_Build_Integration.md](03_Build_Integration.md) - 理解 OH 适配层
3. [02_Patches.md](02_Patches.md) - 确认无 Patch 需要迁移
4. [06_Security.md](06_Security.md) - 升级后安全验证

### 5. 新接触 OH 第三方库的管理者

**目标**: 理解 bindgen 在 OH 中的位置和作用

**阅读顺序**:
1. [README.md](README.md) - 概览
2. [01_Overview.md](01_Overview.md) - 原始库简介
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系图

## 文档速查表

| 想了解 | 阅读 |
|-------|-----|
| bindgen 是什么 | [01_Overview.md](01_Overview.md) |
| 有没有 Patch | [02_Patches.md](02_Patches.md) |
| BUILD.gn 怎么写 | [03_Build_Integration.md](03_Build_Integration.md) |
| 谁在用它 | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| API 有没有变化 | [05_API_Differences.md](05_API_Differences.md) |
| 安全不安全 | [06_Security.md](06_Security.md) |
| 项目整体评估 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) |

## 关键信息摘要

- **无 Patch**: 本库是原生集成，未修改上游代码
- **版本不一致**: BUILD.gn 版本号 (0.64.0) ≠ 实际版本 (0.70.1)，维护时注意同步
- **工具性质**: bindgen 是构建时工具，不打包到最终系统镜像
- **ANI 支撑**: 主要使用场景是为 ANI (Ark Native Interface) 生成绑定
