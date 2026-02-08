# 文档导航与阅读路线

本文档为 OpenHarmony XTS ACTS 仓库的完整工程 Wiki，提供从项目概述到安全评估的全方位指南。建议读者根据自身角色选择不同的阅读路线。

## 新人入门路线

### 必读内容（建议顺序）

1. **[README.md](./README.md)** - 了解 Wiki 的覆盖范围、更新方式与快速索引
2. **[01_Overview.md](./01_Overview.md)** - 理解 XTS 子系统定位、ACTS 核心能力与 OpenHarmony 三种系统类型的差异
3. **[02_Directory_Structure.md](./02_Directory_Structure.md)** - 掌握仓库目录结构、各模块职责与测试用例组织规范
4. **[03_Test_Frameworks.md](./03_Test_Frameworks.md)** - 学习三种测试框架（HCTest/HCPPTest/HJSUnit）的使用方法
5. **[05_Build_Artifacts.md](./05_Build_Artifacts.md)** - 了解编译产物、输出路径与运行时加载关系

### 进阶内容

完成上述内容后，可根据需要深入以下专题：

## 测试用例开发者路线

| 优先级 | 文档 | 说明 |
|--------|------|------|
| P0 | [03_Test_Frameworks.md](./03_Test_Frameworks.md) | 测试框架语法、用例编写规范 |
| P0 | [02_Directory_Structure.md](./02_Directory_Structure.md) | 正确放置测试用例位置 |
| P1 | [04_GN_Build.md](./04_GN_Build.md) | 理解 BUILD.gn 配置 |
| P2 | [07_Troubleshooting.md](./07_Troubleshooting.md) | 常见构建/执行问题排查 |

## 构建工程师路线

| 优先级 | 文档 | 说明 |
|--------|------|------|
| P0 | [04_GN_Build.md](./04_GN_Build.md) | GN target 详解、依赖关系 |
| P0 | [05_Build_Artifacts.md](./05_Build_Artifacts.md) | 编译产物清单、安装路径 |
| P1 | [02_Directory_Structure.md](./02_Directory_Structure.md) | 模块边界与职责 |

## 安全工程师路线

| 优先级 | 文档 | 说明 |
|--------|------|------|
| P0 | [06_Security_Review.md](./06_Security_Review.md) | 攻击面分析、风险点与修复建议 |
| P1 | [01_Overview.md](./01_Overview.md) | 理解系统边界与信任模型 |

## 完整文档列表

### 核心文档

| 文件名 | 标题 | 最后更新 |
|--------|------|----------|
| README.md | Wiki 使用指南 | 2026-02-06 |
| SUMMARY.md | 文档导航与阅读路线 | 2026-02-06 |
| [01_Overview.md](./01_Overview.md) | 项目概述与系统类型 | 2026-02-06 |
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构与模块职责 | 2026-02-06 |
| [03_Test_Frameworks.md](./03_Test_Frameworks.md) | 测试框架详解 | 2026-02-06 |
| [04_GN_Build.md](./04_GN_Build.md) | GN 构建系统 | 2026-02-06 |
| [05_Build_Artifacts.md](./05_Build_Artifacts.md) | 编译产物指南 | 2026-02-06 |
| [06_Security_Review.md](./06_Security_Review.md) | 安全风险评估 | 2026-02-06 |
| [07_Troubleshooting.md](./07_Troubleshooting.md) | 故障排查指南 | 2026-02-06 |

### 附录

| 文件名 | 标题 | 说明 |
|--------|------|------|
| [appendix/Audit_Log.md](./appendix/Audit_Log.md) | 审计日志 | 记录文档变更历史 |

## 快捷跳转

### 按主题跳转

- **项目入门**: [01_Overview.md](./01_Overview.md) → [02_Directory_Structure.md](./02_Directory_Structure.md)
- **开发测试**: [03_Test_Frameworks.md](./03_Test_Frameworks.md) → [07_Troubleshooting.md](./07_Troubleshooting.md)
- **构建部署**: [04_GN_Build.md](./04_GN_Build.md) → [05_Build_Artifacts.md](./05_Build_Artifacts.md)
- **安全评估**: [06_Security_Review.md](./06_Security_Review.md)

### 按角色跳转

- 新人: 从 [01_Overview.md](./01_Overview.md) 开始
- 测试开发: 从 [03_Test_Frameworks.md](./03_Test_Frameworks.md) 开始
- 构建工程师: 从 [04_GN_Build.md](./04_GN_Build.md) 开始
- 安全审计: 从 [06_Security_Review.md](./06_Security_Review.md) 开始

## 版本兼容性

| Wiki 版本 | 兼容 ACTS 版本 | 备注 |
|-----------|----------------|------|
| 1.0 | 3.1 | 初始版本，基于 bundle.json 3.1 |
