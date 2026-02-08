# OpenHarmony Linux Kernel 5.10 Wiki 导航

> 本文档提供全站导航和新人阅读顺序建议

---

## 📖 阅读路线

### 🚀 快速入门（30分钟）
适合：首次接触该项目的开发者

1. **[首页](./index.md)** - 项目定位、核心能力、运行环境
2. **[项目定位](./01_Overview.md)** - 项目边界、关键概念
3. **[目录结构](./02_Architecture.md)** - 代码组织、模块职责

### 🔧 深度理解（2小时）
适合：需要修改或扩展内核的开发者

4. **[架构图](./03_ArchDiagrams.md)** - 组件图、数据流、线程模型
5. **[对外接口](./04_PublicAPI.md)** - 系统调用、proc/sysfs、ioctl
6. **[内部API](./05_InnerAPI.md)** - 模块接口、依赖方向

### ⚙️ 构建与部署
适合：需要编译和部署内核的开发者

7. **[构建系统](./06_BuildSystem.md)** - Kconfig、Makefile、配置选项
8. **[编译产物](./07_BuildOutputs.md)** - 产物类型、安装路径

### 🔒 安全审计
适合：进行安全评审的开发者

9. **[安全评审](./08_SecurityReview.md)** - 攻击面、信任边界、风险点
10. **[问题定位](./09_Troubleshooting.md)** - 常见问题与调试方法

### 📚 附录
进阶参考

- **[调用链分析](./appendix/Callgraphs.md)** - 关键调用链
- **[配置选项](./appendix/ConfigFlags.md)** - 关键宏与 Feature Flags

---

## 📂 文档清单

| 序号 | 文档 | 状态 | 说明 |
|------|------|------|------|
| 1 | [首页](./index.md) | ✅ | 项目概览 |
| 2 | [项目定位](./01_Overview.md) | ✅ | 边界与概念 |
| 3 | [目录结构](./02_Architecture.md) | ✅ | 模块职责 |
| 4 | [架构图](./03_ArchDiagrams.md) | ✅ | 可视化架构 |
| 5 | [对外接口](./04_PublicAPI.md) | ✅ | 用户空间接口 |
| 6 | [内部API](./05_InnerAPI.md) | ✅ | 内部模块接口 |
| 7 | [构建系统](./06_BuildSystem.md) | ✅ | Kconfig/Makefile |
| 8 | [编译产物](./07_BuildOutputs.md) | ✅ | 编译输出 |
| 9 | [安全评审](./08_SecurityReview.md) | ✅ | 安全风险分析 |
| 10 | [问题定位](./09_Troubleshooting.md) | ✅ | 调试指南 |
| 11 | [附录-调用链](./appendix/Callgraphs.md) | ✅ | 调用链分析 |
| 12 | [附录-配置选项](./appendix/ConfigFlags.md) | ✅ | Feature Flags |

---

## 🔍 按主题查找

### 想了解架构？
- [首页](./index.md) - 总体架构描述
- [架构图](./03_ArchDiagrams.md) - Mermaid 可视化架构
- [目录结构](./02_Architecture.md) - 代码组织

### 想了解接口？
- [对外接口](./04_PublicAPI.md) - 用户空间可见的所有接口
- [内部API](./05_InnerAPI.md) - 内部模块间接口
- [附录-调用链](./appendix/Callgraphs.md) - 接口调用链

### 想了解构建？
- [构建系统](./06_BuildSystem.md) - Kconfig 和 Makefile
- [编译产物](./07_BuildOutputs.md) - 编译输出详解
- [附录-配置选项](./appendix/ConfigFlags.md) - 关键配置选项

### 想了解安全？
- [安全评审](./08_SecurityReview.md) - 完整安全风险分析
- [问题定位](./09_Troubleshooting.md) - 安全相关调试

---

## 📝 术语表

| 术语 | 说明 |
|------|------|
| Kconfig | 内核配置系统 |
| Makefile | 构建规则定义 |
| defconfig | 默认配置文件 |
| LSM | Linux Security Modules |
| Syscall | 系统调用 |
| Device Tree | 设备树描述硬件 |

---

## 🔗 外部链接

- [OpenHarmony 官网](https://www.openharmony.cn)
- [内核邮件列表](https://lists.openatom.io/postorius/lists/kernel.openharmony.io/)
- [上游 Linux 文档](https://www.kernel.org/doc/html/latest/)
- [DCO 签署](https://dco.openharmony.io/sign-dco)

---

*最后更新: 2026-02-06*
