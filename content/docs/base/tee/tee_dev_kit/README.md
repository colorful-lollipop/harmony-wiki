# TEE SDK 开发工具包 - 工程 Wiki

## 文档说明

本 Wiki 提供 tee_dev_kit (TEE SDK Development Kit) 项目的完整工程文档，帮助开发者快速理解项目结构、构建系统和安全机制。

**项目定位**: OpenHarmony TEE SDK 开发工具包，用于开发运行在可信执行环境中的 Trusted Application (TA)。

### 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 已覆盖 | 项目定位、目录结构、核心能力 |
| 架构说明 | ✅ 已覆盖 | 组件关系、数据流、线程模型 |
| TA 开发指南 | ✅ 已覆盖 | 开发流程、配置、示例代码 |
| 构建系统 | ✅ 已覆盖 | CMake/Make/GN 配置详解 |
| 安全评审 | ✅ 已覆盖 | 风险分析、攻击面、修复建议 |
| 攻击面分析 | ✅ 已覆盖 | 输入清单、信任边界、审计清单 |

### 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 全站导航（双路线导航）
├── 00_Overview.md              # 项目概览
├── 01_Architecture.md          # 架构说明
├── 02_TA_Development_Guide.md  # TA 开发指南
├── 03_Build_System.md          # 构建系统详解
├── 04_Security_Review.md        # 安全风险评审
├── 05_Attack_Surface.md        # 攻击面分析（新增）
├── 05_Examples.md              # 示例代码说明
├── 06_Troubleshooting.md        # 常见问题
└── _work/                      # 工作区（不包含在 Wiki 中）
    ├── ASSESSMENT.md           # 项目评估报告
    ├── PLAN.md                 # 任务计划
    └── NOTES.md                # 事实记录
```

### 受众定位

| 受众 | 关注重点 | 推荐阅读路线 |
|------|---------|-------------|
| **新人学习者** | 项目定位、快速上手、API 使用 | 新人学习路线（30 分钟上手） |
| **安全研究员** | 攻击面、信任边界、漏洞利用 | 安全研究路线（60 分钟深入） |

### 更新方式

当代码库发生以下变更时，需要同步更新 Wiki：

1. **新增 GN target**：在 `03_Build_System.md` 中添加 target 说明
2. **修改签名流程**：更新 `04_Security_Review.md` 相关章节
3. **新增示例代码**：在 `05_Examples.md` 中添加说明
4. **修改配置文件**：更新对应的配置说明章节
5. **安全相关变更**：在 `05_Attack_Surface.md` 中更新攻击面

### 生成信息

- **生成时间**: 2026-02-07
- **代码版本**: 基于当前仓库状态
- **维护者**: OpenHarmony TEE 团队

---

## 快速导航

### 📚 新人学习路线

> **目标**: 30 分钟上手开发第一个 TA

```
1. 阅读 [项目概览](00_Overview.md) → 理解项目是什么
2. 阅读 [架构说明](01_Architecture.md) → 理解系统如何工作
3. 阅读 [TA 开发指南](02_TA_Development_Guide.md) → 学习开发流程
4. 参考 [示例代码](05_Examples.md) → 动手实践
5. 了解 [构建系统](03_Build_System.md) → 理解构建配置
```

### 🔐 安全研究路线

> **目标**: 60 分钟定位安全风险

```
1. 阅读 [项目概览](00_Overview.md) → 理解对外暴露面
2. 阅读 [攻击面分析](05_Attack_Surface.md) → 识别所有输入点
3. 阅读 [安全评审](04_Security_Review.md) → 详细风险分析
4. 了解 [构建系统](03_Build_System.md) → 理解签名工具链
```

### 🛠️ 开发者参考

| 参考类型 | 文档链接 |
|---------|---------|
| 构建配置 | [03_Build_System.md](03_Build_System.md) |
| 安全机制 | [04_Security_Review.md](04_Security_Review.md) |
| 攻击面 | [05_Attack_Surface.md](05_Attack_Surface.md) |
| 示例代码 | [05_Examples.md](05_Examples.md) |
| 故障排查 | [06_Troubleshooting.md](06_Troubleshooting.md) |

### 📖 完整导航

请参考 [SUMMARY.md](SUMMARY.md) 获取详细的双路线导航和文档索引。

---

## 相关链接

### 官方资源

- [OpenHarmony TEE 官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-tEE.md)
- [GP TEE 标准规范](https://globalplatform.org/specifications/)
- [OpenHarmony build 仓库](https://gitee.com/openharmony/build)
- [TEE SDK 开发工具包](https://gitee.com/openharmony/tee_dev_kit)

---

*最后更新: 2026-02-07*
