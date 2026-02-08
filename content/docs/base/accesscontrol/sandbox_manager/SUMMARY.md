# 文档导航 (SUMMARY)

> 建议根据您的角色选择合适的阅读路径

---

## 新人学习路线 (Getting Started)

> **目标**：30 分钟内理解项目定位、架构和使用方法

| 顺序 | 文档 | 预计时间 | 核心内容 |
|-----|------|---------|---------|
| 1 | [README](README.md) | 2 分钟 | 项目概览、文档导航 |
| 2 | [01_Overview](01_Overview.md) | 5 分钟 | 什么是 Sandbox Manager、能做什么 |
| 3 | [02_Architecture](02_Architecture.md) | 10 分钟 | 三层架构、数据流、初始化流程 |
| 4 | [04_Interface](04_Interface.md) | 10 分钟 | API 快速参考、代码示例 |
| 5 | [03_CodeMap](03_CodeMap.md) | 3 分钟 | 关键文件定位 |

**学习检查点**：
- [ ] 能解释 Sandbox Manager 的用途
- [ ] 能描述三层架构的职责划分
- [ ] 能找到 API 入口和核心实现
- [ ] 能理解策略类型和操作模式

---

## 安全研究路线 (Security Research)

> **目标**：快速识别攻击面、评估安全风险

| 顺序 | 文档 | 预计时间 | 核心内容 |
|-----|------|---------|---------|
| 1 | [README](README.md) | 1 分钟 | 项目定位 |
| 2 | [05_AttackSurface](05_AttackSurface.md) | 15 分钟 | 外部输入清单、信任边界图 |
| 3 | [06_SecurityReview](06_SecurityReview.md) | 20 分钟 | 5+ 类风险分析、漏洞利用路径 |
| 4 | [02_Architecture](02_Architecture.md) | 10 分钟 | 架构层面的安全边界 |
| 5 | [04_Interface](04_Interface.md) | 5 分钟 | API 级别的攻击面 |

**研究检查点**：
- [ ] 能列出所有外部输入入口
- [ ] 能绘制信任边界跨越点
- [ ] 能识别关键权限检查点
- [ ] 能评估每类风险的严重程度

---

## 开发者参考路线 (Developer Reference)

> **目标**：快速定位代码、理解实现细节

| 顺序 | 文档 | 预计时间 | 核心内容 |
|-----|------|---------|---------|
| 1 | [README](README.md) | 1 分钟 | 项目概览 |
| 2 | [04_Interface](04_Interface.md) | 15 分钟 | API 完整清单、使用示例 |
| 3 | [08_Internals](08_Internals.md) | 20 分钟 | 核心类实现、生命周期 |
| 4 | [03_CodeMap](03_CodeMap.md) | 5 分钟 | 文件结构、代码导航 |
| 5 | [07_Build](07_Build.md) | 5 分钟 | 构建配置、GN targets |

---

## 快速索引

### API 参考

| API 类别 | 说明 | 文档位置 |
|---------|------|---------|
| 持久化策略 | Persist/UnPersistPolicy | 04_Interface.md |
| 临时策略 | Set/UnSetPolicy | 04_Interface.md |
| 策略检查 | CheckPolicy | 04_Interface.md |
| 策略激活 | Start/StopAccessingPolicy | 04_Interface.md |

### 安全相关

| 主题 | 说明 | 文档位置 |
|-----|------|---------|
| 权限检查 | CheckPermission | 05_AttackSurface.md |
| 路径验证 | CheckPathIsBlocked | 06_SecurityReview.md |
| 信任边界 | 边界跨越点 | 05_AttackSurface.md |
| 风险清单 | 已知风险 | 06_SecurityReview.md |

### 文件定位

| 层级 | 关键文件 | 文档位置 |
|-----|---------|---------|
| Interface | sandbox_manager_kit.h | 03_CodeMap.md |
| Framework | sandbox_manager_client.cpp | 03_CodeMap.md |
| Service | sandbox_manager_service.cpp | 03_CodeMap.md |
| Database | sandbox_manager_rdb.cpp | 03_CodeMap.md |
| MAC | mac_adapter.cpp | 03_CodeMap.md |

---

## 版本信息

| 项目 | 值 |
|-----|-----|
| 当前版本 | 1.0.0 |
| 文档版本 | 2025-02-07 |
| OpenHarmony 版本 | 5.0+ |

---

## 贡献指南

### 文档改进

如果您发现文档错误或需要补充：

1. 在文档对应章节添加 `[TODO]` 标记
2. 记录问题到 `wiki/_work/NOTES.md`
3. 提交 Issue 或 MR

### 代码改进

如果您发现代码需要改进：

1. 遵循项目编码规范
2. 添加必要的注释
3. 更新相关文档

---

*导航更新时间: 2025-02-07*
