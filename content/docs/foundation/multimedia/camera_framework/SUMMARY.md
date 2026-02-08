# 文档导航 (SUMMARY)

> OpenHarmony Camera Framework Wiki - 为新人学习者和安全研究员提供的完整文档

---

## 快速选择您的阅读路线

### 📚 新人学习路线
适合刚接触项目的开发者，从全局到细节逐步深入：

1. [01_Overview.md](./01_Overview.md) - **项目概览** (5分钟)
   - 项目是什么、能做什么、不能做什么
   - 运行环境和依赖
   
2. [02_Architecture.md](./02_Architecture.md) - **架构与数据流** (15分钟)
   - 组件图、分层架构
   - 数据流完整路径
   - 线程模型

3. [03_CodeMap.md](./03_CodeMap.md) - **目录结构与代码地图** (15分钟)
   - 顶层目录职责
   - 核心文件定位
   - 代码导航图

4. [04_Interface.md](./04_Interface.md) - **对外接口文档** (20分钟)
   - N-API 接口清单
   - IPC 接口定义
   - 配置文件格式

5. [07_Build.md](./07_Build.md) - **构建与产物** (10分钟)
   - GN Targets
   - 编译产物说明
   - Feature 开关

### 🔒 安全研究路线
适合安全审计和漏洞研究人员，快速识别攻击面和风险点：

1. [05_AttackSurface.md](./05_AttackSurface.md) - **攻击面分析** (15分钟)
   - 外部输入入口清单
   - 敏感操作清单
   - 信任边界图

2. [06_SecurityReview.md](./06_SecurityReview.md) - **安全风险评估** (30分钟)
   - 输入验证缺陷
   - 内存安全问题
   - 权限与鉴权
   - 并发安全
   - 逻辑漏洞
   
3. [02_Architecture.md](./02_Architecture.md) - **架构与数据流** (10分钟)
   - 信任边界分析
   - 数据流安全关键点

4. [08_Internals.md](./08_Internals.md) - **内部实现细节** (15分钟)
   - 核心类职责
   - 资源生命周期

---

## 文档清单

### 核心文档 (必看)

| 文档 | 目标读者 | 内容 | 预计阅读时间 |
|------|----------|------|--------------|
| [01_Overview.md](./01_Overview.md) | 所有人 | 项目定位、能力边界、快速开始 | 5-10 min |
| [02_Architecture.md](./02_Architecture.md) | 所有人 | 架构图、数据流、线程模型 | 15-20 min |
| [03_CodeMap.md](./03_CodeMap.md) | 新人 | 目录结构、代码导航 | 15 min |
| [04_Interface.md](./04_Interface.md) | 开发者 | N-API/IPC/Native 接口 | 20-30 min |
| [05_AttackSurface.md](./05_AttackSurface.md) | 安全研究员 | 攻击面识别 | 15 min |
| [06_SecurityReview.md](./06_SecurityReview.md) | 安全研究员 | 风险分析、漏洞评估 | 30 min |
| [07_Build.md](./07_Build.md) | 开发者 | 构建系统、产物说明 | 10 min |
| [08_Internals.md](./08_Internals.md) | 深入研究者 | 实现细节、设计模式 | 20 min |

### 附录 (参考)

- [appendix/FAQ.md](./appendix/FAQ.md) - 常见问题
- [appendix/Changelog.md](./appendix/Changelog.md) - 变更历史

### 工作文件 (开发维护用)

- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估报告
- [_work/PLAN.md](./_work/PLAN.md) - 任务计划
- [_work/NOTES.md](./_work/NOTES.md) - 调研笔记与证据记录

---

## 文档阅读指南

### 代码引用格式说明

```markdown
文件：`path/to/file.cpp:123`

```cpp
// 关键代码片段
void Function() {
    // ...
}
```
```

### 风险描述格式

```markdown
### R1: 风险名称（风险等级）

**位置**：`path/to/file.cpp:line`

**证据**：
```cpp
// 有风险的代码
```

**触发路径**：
```
输入入口 → 调用链 → 漏洞点
```

**影响评估**：可利用性、权限提升可能

**修复建议**：具体代码建议
```

---

## 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2025-02-07 | v1.0 | 初始版本，完成 Phase 0-4 |

---

## 项目基本信息

| 属性 | 内容 |
|------|------|
| **项目名称** | @ohos/camera_framework |
| **子系统** | multimedia |
| **系统能力** | SystemCapability.Multimedia.Camera.Core |
| **版本** | 3.1 |
| **SAID** | 3008 (camera_service) |
| **License** | Apache 2.0 |

