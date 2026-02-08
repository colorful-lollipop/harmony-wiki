# OAID Wiki 目录

## 📖 文档导航

本文档是 OpenHarmony OAID (Open Anonymous Device Identifier) 项目的智能 Wiki，为新人学习者和安全研究员提供全面的技术参考。

---

## 🎓 新人学习路线

**目标受众**: 刚接触 OAID 项目的开发者、需要集成 OAID 的应用开发者

### 快速入门（30 分钟）

1. **[项目概览](01_Overview.md)** ⭐ 必看
   - 了解 OAID 是什么、能做什么、怎么用
   - 快速开始代码示例
   - 权限申请指南

2. **[代码地图](03_CodeMap.md)** ⭐ 必看
   - 目录结构说明
   - 关键文件定位
   - 调用链导航

### 深入理解（1-2 小时）

3. **[架构分析](02_Architecture.md)**
   - 组件架构图
   - 数据流分析
   - 线程模型
   - 信任边界

4. **[接口文档](04_Interface.md)**
   - N-API 接口（JavaScript）
   - IPC 接口（C++）
   - 错误码参考
   - 配置说明

### 工程实践

5. **[构建配置](07_Build.md)**
   - GN 构建系统
   - 编译产物说明
   - 依赖关系

6. **[内部实现](08_Internals.md)**
   - 核心类职责
   - 资源生命周期
   - 关键算法

---

## 🔒 安全研究路线

**目标受众**: 安全研究员、代码审计人员、安全开发人员

### 快速评估（30 分钟）

1. **[攻击面分析](05_AttackSurface.md)** ⭐ 必看
   - 外部输入清单
   - 敏感操作清单
   - 信任边界图
   - 攻击路径分析

2. **[安全风险评估](06_SecurityReview.md)** ⭐ 必看
   - 11 项安全风险详细分析
   - 风险等级评估
   - 修复建议
   - 代码证据

### 深度审计

3. **[架构分析](02_Architecture.md)** → 安全视角
   - 信任边界跨越点
   - 权限检查流程

4. **[代码地图](03_CodeMap.md)** → 敏感函数定位
   - 权限检查函数位置
   - 输入处理点
   - 文件操作位置

5. **[接口文档](04_Interface.md)** → 接口安全
   - 权限要求分析
   - IPC 接口安全性

---

## 📚 完整文档索引

### 基础文档

| 文档 | 描述 | 受众 |
|------|------|------|
| [README.md](README.md) | Wiki 介绍、使用说明 | 所有读者 |
| [ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 | 维护者 |
| [NOTES.md](_work/NOTES.md) | 代码证据汇总 | 维护者 |
| [PLAN.md](_work/PLAN.md) | 任务进度追踪 | 维护者 |

### 核心内容

| 文档 | 描述 | 页数估算 |
|------|------|---------|
| [01_Overview.md](01_Overview.md) | 项目概览、快速开始 | ~200 行 |
| [02_Architecture.md](02_Architecture.md) | 架构与数据流 | ~400 行 |
| [03_CodeMap.md](03_CodeMap.md) | 目录结构与代码地图 | ~500 行 |
| [04_Interface.md](04_Interface.md) | 对外接口文档 | ~400 行 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 | ~400 行 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 | ~600 行 |
| [07_Build.md](07_Build.md) | 构建与产物 | ~300 行 |
| [08_Internals.md](08_Internals.md) | 内部实现细节 | ~400 行 |

---

## 🗺️ 场景导航

### 场景 1：我要集成 OAID 到我的应用

**路径**: [01_Overview](01_Overview.md) → [04_Interface](04_Interface.md)

**步骤**:
1. 阅读概览了解基本概念
2. 查看快速开始代码示例
3. 查看接口文档获取详细 API 说明
4. 参考权限申请流程

### 场景 2：我要审计 OAID 的安全性

**路径**: [05_AttackSurface](05_AttackSurface.md) → [06_SecurityReview](06_SecurityReview.md) → [03_CodeMap](03_CodeMap.md)

**步骤**:
1. 从攻击面分析了解输入点和敏感操作
2. 查看安全风险评估的详细风险分析
3. 使用代码地图定位具体代码位置
4. 交叉验证代码证据

### 场景 3：我要修改 OAID 代码

**路径**: [02_Architecture](02_Architecture.md) → [03_CodeMap](03_CodeMap.md) → [08_Internals](08_Internals.md) → [07_Build](07_Build.md)

**步骤**:
1. 理解整体架构
2. 定位需要修改的代码位置
3. 了解内部实现细节
4. 查看构建配置确保正确编译

### 场景 4：我要了解 OAID 工作原理

**路径**: [01_Overview](01_Overview.md) → [02_Architecture](02_Architecture.md) → [08_Internals](08_Internals.md)

**步骤**:
1. 了解项目定位和用途
2. 查看架构图和数据流
3. 深入了解内部实现细节

---

## 🔍 关键词索引

### 按功能

| 关键词 | 相关文档 |
|--------|---------|
| `getOAID` | [01_Overview](01_Overview.md), [04_Interface](04_Interface.md) |
| `resetOAID` | [01_Overview](01_Overview.md), [04_Interface](04_Interface.md) |
| `APP_TRACKING_CONSENT` | [01_Overview](01_Overview.md), [04_Interface](04_Interface.md) |
| `CheckPermission` | [05_AttackSurface](05_AttackSurface.md), [06_SecurityReview](06_SecurityReview.md) |
| `CheckSystemApp` | [04_Interface](04_Interface.md), [05_AttackSurface](05_AttackSurface.md) |
| `KVStore` | [02_Architecture](02_Architecture.md), [08_Internals](08_Internals.md) |
| `UUID` | [08_Internals](08_Internals.md) |

### 按安全风险

| 风险 | 相关文档 |
|------|---------|
| Mutex 死锁 | [06_SecurityReview.md#r1](06_SecurityReview.md#r1) |
| JSON 注入 | [06_SecurityReview.md#r2](06_SecurityReview.md#r2) |
| 路径遍历 | [06_SecurityReview.md#r3](06_SecurityReview.md#r3) |
| TOCTOU | [06_SecurityReview.md#r4](06_SecurityReview.md#r4) |

---

## 📞 相关资源

### 官方文档

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [OAID API 参考](https://gitee.com/openharmony/docs/blob/master/en/application-dev/reference/apis/js-apis-advertising.md)

### 参考实现

- [华为 HMS 广告示例](https://github.com/HMS-Core/hms-ads-demo-harmonyos)
- [华为开发者联盟](https://developer.huawei.com/consumer/cn/hms/huawei-adskit/)

### 内部参考

- [代码证据汇总](_work/NOTES.md)
- [项目评估报告](_work/ASSESSMENT.md)

---

## 📝 文档更新记录

| 日期 | 更新内容 | 版本 |
|------|---------|------|
| 2026-02-07 | 初始版本，完整 Wiki 文档生成 | v1.0 |

---

## 🤝 贡献指南

本文档由智能 Wiki 生成 Agent 自动生成，基于 OAID 项目源代码分析。

如需更新文档：
1. 修改源代码后重新运行分析
2. 更新 `_work/NOTES.md` 中的代码证据
3. 同步更新相关章节

---

**快速跳转**: [概览](01_Overview.md) | [架构](02_Architecture.md) | [攻击面](05_AttackSurface.md) | [安全风险](06_SecurityReview.md)
