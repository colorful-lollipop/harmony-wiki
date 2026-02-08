# 全站导航

> **阅读时间**: 3 分钟 | **目标**: 快速找到所需文档

---

## 双路线导航

### 📚 新人学习路线

> **目标**: 从零开始学习 TEE SDK 开发

```
开始 ──▶ 5分钟 ──▶ 15分钟 ──▶ 30分钟 ──▶ 60分钟
  │        │          │           │          │
  ▼        ▼          ▼           ▼          ▼
README  Overview   Architecture  TA Guide   Examples
```

| 阶段 | 阅读时间 | 文档 | 目标 |
|------|---------|------|------|
| **1. 入门** | 5 分钟 | [README.md](README.md) | 了解 Wiki 结构和文档范围 |
| **2. 理解** | 5 分钟 | [00_Overview.md](00_Overview.md) | 理解项目定位和核心能力 |
| **3. 架构** | 15 分钟 | [01_Architecture.md](01_Architecture.md) | 理解 CA ↔ TEE 通信架构 |
| **4. 开发** | 30 分钟 | [02_TA_Development_Guide.md](02_TA_Development_Guide.md) | 开发第一个 TA |
| **5. 示例** | 20 分钟 | [05_Examples.md](05_Examples.md) | 参考示例代码学习 |
| **6. 构建** | 20 分钟 | [03_Build_System.md](03_Build_System.md) | 理解构建系统 |
| **7. 安全** | 15 分钟 | [04_Security_Review.md](04_Security_Review.md) | 了解安全要点 |
| **8. 排错** | 按需 | [06_Troubleshooting.md](06_Troubleshooting.md) | 遇到问题时查阅 |

### 🔐 安全研究路线

> **目标**: 快速定位安全风险和攻击面

```
开始 ──▶ 5分钟 ──▶ 15分钟 ──▶ 30分钟 ──▶ 深入
  │        │          │           │
  ▼        ▼          ▼           ▼
README  Overview   Attack      Security
                   Surface     Review
```

| 阶段 | 阅读时间 | 文档 | 目标 |
|------|---------|------|------|
| **1. 定位** | 5 分钟 | [README.md](README.md) | 了解项目边界和架构 |
| **2. 入口** | 5 分钟 | [00_Overview.md](00_Overview.md) | 理解对外暴露面 |
| **3. 攻击面** | 15 分钟 | [05_Attack_Surface.md](05_Attack_Surface.md) | 识别所有输入点和敏感操作 |
| **4. 深度** | 30 分钟 | [04_Security_Review.md](04_Security_Review.md) | 详细风险分析和修复建议 |
| **5. 构建** | 20 分钟 | [03_Build_System.md](03_Build_System.md) | 理解签名工具链 |
| **6. 架构** | 15 分钟 | [01_Architecture.md](01_Architecture.md) | 理解信任边界 |
| **7. 示例** | 按需 | [05_Examples.md](05_Examples.md) | 审计示例代码 |

---

## 文档索引

| # | 文档 | 说明 | 关键内容 | 受众 |
|---|------|------|---------|------|
| 00 | [README.md](README.md) | Wiki 说明 | 覆盖范围、更新方式、导航 | 所有人 |
| 01 | [00_Overview.md](00_Overview.md) | 项目概览 | 项目定位、目录结构、关键概念 | 新人 |
| 02 | [01_Architecture.md](01_Architecture.md) | 架构说明 | 组件关系、数据流、时序图 | 所有人 |
| 03 | [02_TA_Development_Guide.md](02_TA_Development_Guide.md) | TA 开发 | 配置、编译、签名、调试 | 开发者 |
| 04 | [03_Build_System.md](03_Build_System.md) | 构建系统 | CMake/Make/GN 详解 | 开发者 |
| 05 | [04_Security_Review.md](04_Security_Review.md) | 安全评审 | 风险点、攻击面、修复建议 | 安全研究员 |
| 06 | [05_Attack_Surface.md](05_Attack_Surface.md) | 攻击面分析 | 输入清单、信任边界、审计清单 | 安全研究员 |
| 07 | [05_Examples.md](05_Examples.md) | 示例说明 | TA/CA 示例代码分析 | 开发者 |
| 08 | [06_Troubleshooting.md](06_Troubleshooting.md) | 常见问题 | 问题定位、解决方案 | 所有人 |

---

## 推荐阅读路径

### 🎯 快速上手（30 分钟）

1. [README.md](README.md)
2. [00_Overview.md](00_Overview.md)
3. [02_TA_Development_Guide.md](02_TA_Development_Guide.md)
4. [05_Examples.md](05_Examples.md)

### 🔍 安全审计（60 分钟）

1. [00_Overview.md](00_Overview.md)
2. [05_Attack_Surface.md](05_Attack_Surface.md)
3. [04_Security_Review.md](04_Security_Review.md)
4. [01_Architecture.md](01_Architecture.md)

### 🛠️ 深度开发（2 小时）

1. [00_Overview.md](00_Overview.md)
2. [01_Architecture.md](01_Architecture.md)
3. [02_TA_Development_Guide.md](02_TA_Development_Guide.md)
4. [05_Examples.md](05_Examples.md)
5. [03_Build_System.md](03_Build_System.md)
6. [04_Security_Review.md](04_Security_Review.md)
7. [06_Troubleshooting.md](06_Troubleshooting.md)

---

## 相关链接

### 官方资源

| 资源 | 说明 |
|------|------|
| [OpenHarmony TEE 官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-tEE.md) | OpenHarmony TEE 子系统文档 |
| [GP TEE 标准规范](https://globalplatform.org/specifications/) | GlobalPlatform TEE 标准 |
| [OpenHarmony build 仓库](https://gitee.com/openharmony/build) | OpenHarmony 构建系统 |
| [TEE SDK 开发工具包](https://gitee.com/openharmony/tee_dev_kit) | 本项目仓库 |

### 项目依赖

| 依赖 | 说明 |
|------|------|
| [third_party_musl](https://gitee.com/openharmony/third_party_musl) | Musl C 标准库 |
| [third_party_bounds_checking_function](https://gitee.com/openharmony/third_party_bounds_checking_function) | 安全函数库 |

---

## 文档统计

| 指标 | 数值 |
|------|------|
| 核心文档 | 8 篇 |
| 工作文档 | 2 篇（ASSESSMENT.md, PLAN.md） |
| 代码证据 | 100+ 处引用 |
| Mermaid 图表 | 10+ 张 |

---

*最后更新: 2026-02-07*
