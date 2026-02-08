# DeviceManager Wiki 导航

## 快速入门

- [README](README.md) - Wiki 使用说明
- [01_Overview](01_Overview.md) - 项目概览

---

## 双路线阅读导航

### 👨‍💻 新人学习路线

适合快速理解项目、上手开发的开发者：

```
入门理解 (5分钟)
├── [README](README.md) - Wiki结构和使用方法
└── [01_Overview](01_Overview.md) - 项目定位、核心能力

架构理解 (15分钟)
├── [02_Architecture](02_Architecture.md) - 组件关系、数据流
└── [03_CodeMap](03_CodeMap.md) - 目录结构、代码地图

API 学习 (20分钟)
├── [03_N_API_Reference](03_N_API_Reference.md) - JS接口清单
└── [04_Inner_API](04_Inner_API.md) - 内部模块接口

工程实践 (15分钟)
├── [05_GN_Build](05_GN_Build.md) - 构建配置
└── [06_Artifacts](06_Artifacts.md) - 编译产物

安全了解 (10分钟)
└── [05_AttackSurface](05_AttackSurface.md) - 攻击面概览
```

### 🔒 安全研究路线

适合进行安全审计、漏洞挖掘的安全研究员：

```
快速定位 (10分钟)
├── [05_AttackSurface](05_AttackSurface.md) - 攻击面地图
│   ├── N-API入口点
│   ├── IPC接口
│   └── 网络攻击面
└── [02_Architecture](02_Architecture.md) - 信任边界

深度分析 (30分钟)
├── [06_SecurityReview](06_SecurityReview.md) - 风险详细分析
│   ├── 输入验证缺陷
│   ├── 权限与鉴权
│   ├── 内存安全
│   └── 逻辑漏洞
└── [07_Security_Review](07_Security_Review.md) - 原始安全评审

代码定位 (20分钟)
├── [03_CodeMap](03_CodeMap.md) - 代码地图
└── [04_Inner_API](04_Inner_API.md) - 接口实现

验证测试
├── [05_GN_Build](05_GN_Build.md) - 构建用于测试
└── [appendix/Callgraphs](appendix/Callgraphs.md) - 调用链图
```

---

## 完整文档索引

### 基础理解
- [01_Overview](01_Overview.md) - 项目定位、能力边界、快速开始
- [02_Architecture](02_Architecture.md) - 架构设计、数据流、线程模型
- [03_CodeMap](03_CodeMap.md) - 目录结构、核心文件、代码导航

### API 参考
- [03_N_API_Reference](03_N_API_Reference.md) - N-API 接口清单（JS接口）
- [04_Inner_API](04_Inner_API.md) - 内部模块接口、依赖关系

### 安全分析
- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析（N-API/IPC/网络）
- [06_SecurityReview](06_SecurityReview.md) - 安全风险评估（详细分析）
- [07_Security_Review](07_Security_Review.md) - 安全风险评审（原始版本）

### 工程实现
- [05_GN_Build](05_GN_Build.md) - GN 构建配置、Targets
- [06_Artifacts](06_Artifacts.md) - 编译产物、安装路径
- [08_Internals](08_Internals.md) - 内部实现细节（待补充）

### 附录
- [appendix/Callgraphs](appendix/Callgraphs.md) - 关键调用链图
- [appendix/Config_Flags](appendix/Config_Flags.md) - 关键配置开关

---

## 按角色推荐

| 角色 | 推荐阅读 | 预计时间 |
|------|---------|----------|
| **应用开发者** | README → 01_Overview → 03_N_API_Reference | 30分钟 |
| **系统开发者** | README → 01_Overview → 02_Architecture → 04_Inner_API → 05_GN_Build | 1小时 |
| **安全研究员** | 05_AttackSurface → 06_SecurityReview → 02_Architecture → 03_CodeMap | 1-2小时 |
| **架构师** | 01_Overview → 02_Architecture → 04_Inner_API → 05_GN_Build | 45分钟 |

---

## 文档状态

| 文档 | 状态 | 更新时间 |
|------|------|----------|
| 01_Overview | ✅ 完成 | 2025-02-06 |
| 02_Architecture | ✅ 完成 | 2025-02-06 |
| 03_N_API_Reference | ✅ 完成 | 2025-02-06 |
| 03_CodeMap | ✅ 完成 | 2025-02-06 |
| 04_Inner_API | ✅ 完成 | 2025-02-06 |
| 05_AttackSurface | ✅ 新增 | 2026-02-07 |
| 05_GN_Build | ✅ 完成 | 2025-02-06 |
| 06_Artifacts | ✅ 完成 | 2025-02-06 |
| 06_SecurityReview | ✅ 新增 | 2026-02-07 |
| 07_Security_Review | ✅ 完成 | 2025-02-06 |
| appendix/Callgraphs | ✅ 完成 | 2025-02-06 |
| appendix/Config_Flags | ✅ 完成 | 2025-02-06 |
