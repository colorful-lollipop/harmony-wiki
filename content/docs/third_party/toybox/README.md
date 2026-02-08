# Toybox OpenHarmony 适配文档

> Toybox 是 OpenHarmony 系统中的基础命令行工具集，提供 200+ 个常用 Linux 命令。

---

## 快速导航

### 核心文档

| 文档 | 说明 |
|-----|------|
| [SUMMARY.md](./SUMMARY.md) | 📖 阅读路线建议 - 从这里开始 |
| [01_Overview.md](./01_Overview.md) | 📚 Toybox 库简介 - 功能与定位 |
| [02_Patches.md](./02_Patches.md) | 🔧 OH Patch 详细分析 - 修改内容与目的 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 🔨 OH 构建适配 - BUILD.gn 与编译选项 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 🔗 依赖关系与使用 - 谁在使用 toybox |

### 工作文档

| 文档 | 说明 |
|-----|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 📊 项目评估结果 - Phase 0 信息收集 |
| [_work/NOTES.md](./_work/NOTES.md) | 📝 分析过程记录 |
| [_work/PLAN.md](./_work/PLAN.md) | 📋 任务进度跟踪 |

---

## 文档概览

### Toybox 是什么？

Toybox 是一个将常见 Linux 命令行工具组合到单个可执行文件中的工具集，由 Rob Landley 创建。它具有以下特点：

- **简单**：代码结构清晰，易于维护
- **小巧**：优化后的二进制文件仅约 73KB
- **快速**：经过性能优化的实现
- **符合标准**：遵循 POSIX 和 LSB 规范
- **多调用二进制**：单个可执行文件，通过符号链接提供多个命令

### 在 OpenHarmony 中的作用

Toybox 在 OH 中扮演**基础命令行工具集**的角色：

- ✅ 提供 200+ 个基础 shell 命令（ls, cat, ps, mount, ifconfig 等）
- ✅ 支持标准系统（standard）和轻量系统（small）
- ✅ 适配 LiteOS_A 内核
- ✅ 集成 SELinux 安全框架
- ✅ 提供独立的 su 命令（调试版）

### OH 定制化程度

| 指标 | 数值 |
|-----|------|
| **OH 特有源文件** | 1 个独立（su.c）+ 20+ 个适配修改 |
| **条件编译点** | 95 处 `TOYBOX_OH_ADAPT` 宏 |
| **修改命令数** | 19 个核心命令 |
| **Patch 文件** | 0（无传统 patch 文件） |

### 关键修改内容

#### 核心修复
- 🔒 **稳定性**：修复 closedir(NULL) 崩溃、top 终端乱码
- 🐛 **Bug 修复**：修复 mv -v 不打印详细日志
- 🎯 **用户体验**：ls 命令排序和显示优化

#### 平台适配
- 💻 **64 位系统**：类型适配（1ULL vs 1LL）
- 🔧 **LiteOS_A**：轻量级内核适配（20+ 命令）
- 🛡️ **SELinux**：安全框架集成
- 📦 **构建系统**：BUILD.gn 适配，安全加固标志

#### 安全加固
- 🔒 **PIE/RELRO/NX**：链接安全标志
- 👤 **su 限制**：仅 root (uid=0) 和 shell (uid=2000) 可切换用户

---

## 依赖关系

### toybox 依赖的组件
- **selinux**：安全策略支持（可选）
- **openssl**：加密功能支持（扩展命令需要）

### 依赖 toybox 的模块
- **samgr**（标准系统）：系统能力管理器
- **samgr_lite**（轻量系统）：轻量级系统能力管理器
- **liteos_a**（内核）：用户空间 shell 环境

详见 [04_Usage_in_OH.md](./04_Usage_in_OH.md)。

---

## 维护与升级

### 升级上游版本时的注意事项

⚠️ **高优先级**：
- 95 处 `TOYBOX_OH_ADAPT` 宏需要重新应用
- 19 个修改的命令需要逐个验证

⚠️ **中优先级**：
- `porting/liteos_a/` 目录需要重新适配
- BUILD.gn 配置需要同步更新

✅ **低风险**：
- `openharmony/su.c` 独立实现，不影响上游

### 测试重点

升级后必须测试的命令：
- 🔧 **文件操作**：ls, cp, mv, rm, find
- 🖥️ **进程管理**：ps, kill, top
- 📡 **网络工具**：ifconfig, netstat, ping
- 📂 **目录遍历**：所有遍历目录的命令

---

## 贡献与反馈

- **上游仓库**：https://github.com/landley/toybox
- **上游邮件列表**：http://lists.landley.net/listinfo.cgi/toybox-landley.net
- **OH 问题反馈**：[OpenHarmony Gitee](https://gitee.com/openharmony)

---

## 许可证

- **原始库**：BSD Zero Clause License (0BSD)
- **OH 特有代码（openharmony/su.c）**：Apache License 2.0

---

## 文档版本

- **生成时间**：2026-02-08
- **库版本**：toybox 0.8.12
- **OH 组件版本**：@ohos/toybox 3.1
