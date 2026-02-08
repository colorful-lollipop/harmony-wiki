# SUMMARY

## Neural Network Runtime Wiki

### 入门指南

- [简介](README.md)
- [首页/概览](index.md)

### 核心文档

- [01. 项目概览](01_Overview.md)
- [02. 架构说明](02_Architecture.md)
- [03. 目录结构](03_Directory_Structure.md)
- [04. 对外 Native API](04_Native_API.md)
- [05. 内部 API](05_Inner_API.md)
- [06. GN 构建目标](06_GN_Targets.md)
- [07. 编译产物](07_Build_Artifacts.md)
- [08. 安全风险评审](08_Security_Review.md)
- [09. 常见问题与调试](09_Troubleshooting.md)

### 附录

- [A. 关键调用链](appendix/Callgraphs.md)
- [B. 关键配置项](appendix/Config_Flags.md)
- [C. API 速查](appendix/API_Quick_Reference.md)
- [D. 错误码对照](appendix/Error_Codes.md)
- [E. 算子列表](appendix/Operators.md)

---

## 双路线阅读指南

### 路线 A: 新人学习路线 🔰

适合：AI 框架开发者、系统开发者、初次接触 NNRt 的工程师

**目标**: 快速理解项目定位、掌握 API 使用、熟悉代码结构

```
README.md → index.md → 01_Overview.md → 02_Architecture.md → 03_Directory_Structure.md
                                                              ↓
                                                    04_Native_API.md (API 使用)
                                                              ↓
                                                    06_GN_Targets.md → 07_Build_Artifacts.md
                                                              ↓
                                                    09_Troubleshooting.md
```

**预计时间**:
- 5 分钟: 理解项目定位 (01_Overview.md)
- 15 分钟: 熟悉架构和数据流 (02_Architecture.md)
- 30 分钟: 找到核心代码位置 (03_Directory_Structure.md)

**必读文档**:
1. [01. 项目概览](01_Overview.md) - 项目定位和能力边界
2. [02. 架构说明](02_Architecture.md) - 系统架构和数据流
3. [03. 目录结构](03_Directory_Structure.md) - 代码组织和文件导航
4. [04. 对外 Native API](04_Native_API.md) - API 使用方法

---

### 路线 B: 安全研究路线 🔒

适合：安全工程师、漏洞研究员、代码审计人员

**目标**: 识别攻击面、定位信任边界、发现可被利用点

```
README.md → index.md → 01_Overview.md → 02_Architecture.md ─┬──► 05_Inner_API.md (内部接口)
                                                              │        ↓
                                                              └──► 08_Security_Review.md ◄──┐
                                                                       ↓                    │
                                                              攻击面分析 → 风险评估 → 修复建议
```

**预计时间**:
- 10 分钟: 理解架构和信任边界 (02_Architecture.md)
- 20 分钟: 识别所有外部输入入口 (08_Security_Review.md 攻击面章节)
- 40 分钟: 深入安全风险分析 (08_Security_Review.md 风险详情)

**必读文档**:
1. [02. 架构说明](02_Architecture.md) - 信任边界和 IPC 机制
2. [05. 内部 API](05_Inner_API.md) - 内部接口和攻击面
3. [08. 安全风险评审](08_Security_Review.md) - 完整的安全分析
4. [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链

**重点关注**:
- 外部输入入口: 模型数据、张量数据、缓存路径、文件描述符
- 敏感操作: 内存分配、文件读写、IPC 调用、设备访问
- 信任边界: 应用进程 ↔ 驱动进程的数据流

---

## 快速导航

| 主题 | 文档 |
|------|------|
| 了解项目定位 | [01_Overview.md](01_Overview.md) |
| 理解系统架构 | [02_Architecture.md](02_Architecture.md) |
| 查看 API 接口 | [04_Native_API.md](04_Native_API.md) |
| 了解构建系统 | [06_GN_Targets.md](06_GN_Targets.md) |
| 查看安全风险 | [08_Security_Review.md](08_Security_Review.md) |
| 排查问题 | [09_Troubleshooting.md](09_Troubleshooting.md) |
| API 速查 | [appendix/API_Quick_Reference.md](appendix/API_Quick_Reference.md) |
| 错误码对照 | [appendix/Error_Codes.md](appendix/Error_Codes.md) |
