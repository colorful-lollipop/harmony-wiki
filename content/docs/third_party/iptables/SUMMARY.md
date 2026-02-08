# iptables Wiki 阅读指南

## 根据你的角色选择阅读路线

### 👨‍💻 开发者 - 快速了解

如果你需要快速了解 iptables 在 OH 中的情况：

1. [README.md](./README.md) - 概览和快速导航
2. [01_Overview.md](./01_Overview.md) - 了解 OH 中的作用
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看依赖关系

**预计阅读时间**: 15 分钟

### 🔧 维护者 - 深度维护

如果你需要维护或升级 iptables：

1. [02_Patches.md](./02_Patches.md) - **必读**: Patch 详细分析
2. [03_Build_Integration.md](./03_Build_Integration.md) - 理解构建系统
3. [06_Security.md](./06_Security.md) - 安全风险和升级建议
4. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整评估报告

**预计阅读时间**: 45 分钟

### 🏢 架构师 - 评估集成

如果你正在评估 iptables 的使用或替代方案：

1. [01_Overview.md](./01_Overview.md) - 功能定位
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系和影响范围
3. [05_API_Differences.md](./05_API_Differences.md) - 接口兼容性
4. [06_Security.md](./06_Security.md) - 安全风险评估

**预计阅读时间**: 30 分钟

### 🐛 调试者 - 问题排查

如果你遇到了 iptables 相关的问题：

1. [02_Patches.md](./02_Patches.md) - 检查 Patch 相关影响
2. [03_Build_Integration.md](./03_Build_Integration.md) - 构建配置检查
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 理解调用链
4. [_work/NOTES.md](./_work/NOTES.md) - 分析过程记录

**预计阅读时间**: 20 分钟

---

## 文档依赖关系

```
README.md (入口)
    │
    ├── 01_Overview.md (基础)
    │       │
    │       └── 04_Usage_in_OH.md (依赖)
    │
    ├── 02_Patches.md (核心)
    │       │
    │       └── 03_Build_Integration.md (构建)
    │
    ├── 05_API_Differences.md (接口)
    │
    └── 06_Security.md (安全)

_work/
    ├── ASSESSMENT.md (评估报告)
    ├── NOTES.md (分析记录)
    └── PLAN.md (任务进度)
```

---

## 按主题分类

### 📝 基础信息
- [库基本信息](./01_Overview.md#库基本信息)
- [OH 中的作用](./01_Overview.md#iptables-在-openharmony-中的作用)
- [依赖关系总览](./04_Usage_in_OH.md#依赖关系总览)

### 🔧 技术细节
- [Patch 分析](./02_Patches.md)
- [构建配置](./03_Build_Integration.md)
- [代码生成](./03_Build_Integration.md#1-扩展模块代码生成-geninitpy)
- [静态链接](./03_Build_Integration.md#1-静态链接模式--dno_shared_libs1)

### 🔒 安全相关
- [CVE 历史](./06_Security.md#cve-历史分析)
- [运行时风险](./06_Security.md#运行时安全风险)
- [加固建议](./06_Security.md#安全加固建议)

### 📊 使用统计
- [直接依赖模块](./04_Usage_in_OH.md#直接依赖模块)
- [NetManager 使用详情](./04_Usage_in_OH.md#1-netmanager-base---核心网络管理)
- [EDM 使用详情](./04_Usage_in_OH.md#2-enterprise-device-management-edm---企业设备管理)

---

## 关键决策点

### 是否需要升级 iptables？

阅读：
1. [02_Patches.md#升级建议](./02_Patches.md#升级建议)
2. [06_Security.md#当前安全状态](./06_Security.md#当前安全状态)

### Patch 是否可向上游提交？

阅读：
1. [02_Patches.md#上游提交建议](./02_Patches.md#上游提交建议)

### 如何添加新扩展模块？

阅读：
1. [03_Build_Integration.md#常见问题](./03_Build_Integration.md#常见问题)
2. [extensions/BUILD.gn](../extensions/BUILD.gn) - 查看现有扩展列表

### 应用如何调用 iptables？

阅读：
1. [04_Usage_in_OH.md#典型使用场景](./04_Usage_in_OH.md#典型使用场景)
2. [05_API_Differences.md#oh-特有的间接接口](./05_API_Differences.md#oh-特有的间接接口)

---

## 外部参考

### 上游文档
- [netfilter 官网](https://netfilter.org/)
- [iptables 手册页](https://ipset.netfilter.org/iptables.man.html)
- [Git 仓库](https://git.netfilter.org/iptables/)

### OpenHarmony 相关
- [NetManager 开发指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/network)
- [EDM 开发指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/customization)

---

*本 Wiki 文档按照 [OpenHarmony 第三方库文档规范](../_work/ASSESSMENT.md) 编写*
