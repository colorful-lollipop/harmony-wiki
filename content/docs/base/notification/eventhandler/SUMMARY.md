# EventHandler Wiki 导航

本文档提供了 EventHandler 部件的完整技术文档，适合新人快速理解项目架构，也适合安全研究员进行代码审计。

---

## 双路线阅读指南

### 路线 A: 新人学习路线

适合快速掌握项目架构和开发使用：

1. **[项目概览](01_Overview.md)** - 了解项目定位、核心能力和运行环境
2. **[目录结构](02_Directory_Structure.md)** - 熟悉代码组织和模块职责
3. **[架构说明](03_Architecture.md)** - 理解组件设计、数据流和线程模型
4. **[N-API 接口](04_NAPI_API.md)** - 学习对外 JS API 接口和使用方法
5. **[内部 API](05_Inner_API.md)** - 了解模块间接口和依赖关系
6. **[GN 构建目标](06_GN_Targets.md)** - 掌握构建系统和编译产物
7. **[编译产物](07_Build_Artifacts.md)** - 了解输出文件和运行时加载关系
8. **[常见问题](09_Troubleshooting.md)** - 遇到问题时的定位方法

### 路线 B: 安全研究路线

适合代码审计和安全评估：

1. **[项目概览](01_Overview.md)** - 快速了解项目边界和对外接口
2. **[攻击面分析](05_AttackSurface.md)** - 系统梳理所有外部输入入口
   - 信任边界图
   - N-API/ANI/FFI/C API 入口清单
   - 敏感操作清单
   - 输入验证状态
3. **[安全风险评审](08_Security_Review.md)** - 详细风险分析和修复建议
   - 输入验证缺陷
   - 内存安全问题
   - 权限与鉴权
   - 并发安全
   - 逻辑漏洞
4. **[架构说明](03_Architecture.md)** - 理解线程模型和数据流
   - 重点关注信任边界跨越点
5. **[N-API 接口](04_NAPI_API.md)** - 参数处理和校验逻辑
6. **[内部 API](05_Inner_API.md)** - 敏感接口调用链

---

## 文档目录

| 文档 | 描述 | 主要读者 |
|------|------|----------|
| [README.md](README.md) | Wiki 概览和更新指南 | 所有人 |
| [01_Overview.md](01_Overview.md) | 项目定位、边界、核心能力 | 新人、安全研究员 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录组织和模块职责 | 新人 |
| [03_Architecture.md](03_Architecture.md) | 组件图、数据流、线程模型 | 开发者、架构师、安全研究员 |
| [04_NAPI_API.md](04_NAPI_API.md) | JS API 面的完整说明 | 前端开发者 |
| [05_AttackSurface.md](05_AttackSurface.md) | **攻击面分析** - 外部输入入口、信任边界 | **安全研究员** |
| [05_Inner_API.md](05_Inner_API.md) | 模块接口、依赖方向 | 框架开发者 |
| [06_GN_Targets.md](06_GN_Targets.md) | targets、类型、依赖关系 | 构建工程师 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 输出文件、安装路径 | 构建工程师 |
| [08_Security_Review.md](08_Security_Review.md) | **安全风险评审** - 可被利用点、修复建议 | **安全研究员** |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 构建和调试问题定位 | 所有人 |

---

## 快速查找

| 查找内容 | 目标文档 |
|----------|----------|
| 项目是什么、能做什么 | [01_Overview.md](01_Overview.md) |
| 文件在哪里 | [02_Directory_Structure.md](02_Directory_Structure.md) |
| 架构如何设计 | [03_Architecture.md](03_Architecture.md) |
| JS/ArkTS 怎么调用 | [04_NAPI_API.md](04_NAPI_API.md) |
| C++ 怎么调用 | [05_Inner_API.md](05_Inner_API.md) |
| 安全入口在哪里 | [05_AttackSurface.md](05_AttackSurface.md) |
| 有什么安全风险 | [08_Security_Review.md](08_Security_Review.md) |
| 怎么编译 | [06_GN_Targets.md](06_GN_Targets.md) |
| 编译输出什么 | [07_Build_Artifacts.md](07_Build_Artifacts.md) |
| 遇到问题怎么办 | [09_Troubleshooting.md](09_Troubleshooting.md) |

---

## 附录

- **工作记录**: [_work/NOTES.md](_work/NOTES.md) - 代码证据汇总
- **项目评估**: [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目画像和文档策略
- **任务计划**: [_work/PLAN.md](_work/PLAN.md) - Wiki 生成计划

---

**更新记录**:
- 2026-02-07: 添加双路线导航，新增攻击面分析文档
- 2026-02-06: 初始版本创建

**维护说明**: 本文档随代码变更同步更新，详见 [README.md](README.md) 中的更新方式。
