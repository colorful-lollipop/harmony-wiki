# libfuse OpenHarmony 文档

> [libfuse](https://github.com/libfuse/libfuse) 是 FUSE (Filesystem in Userspace) 框架的用户态程序接口实现，在 OpenHarmony 中作为用户态文件系统基础设施使用。

---

## 文档导航

本文档重点说明 libfuse 在 **OpenHarmony (OH)** 中的 Patch、特殊适配、以及被系统使用的方式。libfuse 的原始功能仅作简要介绍。

### 快速开始

| 文档 | 说明 |
|------|------|
| **[SUMMARY.md](SUMMARY.md)** | 📖 阅读路线建议 |
| **[01_Overview.md](01_Overview.md)** | 📚 libfuse 库概览及在 OH 中的作用 |
| **[02_Patches.md](02_Patches.md)** | 🔧 OH 特有 Patch 详细分析（核心文档） |
| **[03_Build_Integration.md](03_Build_Integration.md)** | 🏗️ OH 构建系统集成 |
| **[04_Usage_in_OH.md](04_Usage_in_OH.md)** | 🔗 OH 中的依赖关系与使用方式 |
| **[06_Security.md](06_Security.md)** | 🔒 安全风险分析与升级策略 |

### 工作文档

| 文档 | 说明 |
|------|------|
| **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** | 🔍 项目评估结果 |

---

## 核心信息

| 属性 | 值 |
|------|-----|
| **库名称** | libfuse |
| **OH 版本** | 3.17.3 |
| **许可证** | LGPL-2.1 (OH 仅使用 LGPL 部分) |
| **上游地址** | https://github.com/libfuse/libfuse |
| **OH 子系统** | thirdparty |
| **OH 部件** | libfuse |

---

## 关键特性

### OpenHarmony 特有适配

1. **HMFS 支持**: 添加 HMFS（HarmonyOS 分布式文件系统）白名单，允许在 HMFS 上挂载 FUSE 文件系统
2. **符号链接优化**: 跳过 `/etc/mtab` 符号链接更新，适配 OH 系统挂载表管理
3. **构建系统集成**: 完整的 GN 构建系统适配，包含 OH 特定的编译选项
4. **pthread 兼容**: 禁用 pthread 取消操作，解决 OH 平台兼容性问题

### OH 中的主要使用场景

| 场景 | 模块 | 说明 |
|------|------|------|
| **云端文件系统** | cloudfiledaemon | 云盘本地挂载和访问 |
| **数据防泄漏** | libdlp_fuse | 透明加解密文件系统 |
| **媒体库虚拟文件系统** | medialibrary_data_extension | 媒体文件统一访问接口 |
| **MTP 设备挂载** | mtpfs | 通过 FUSE 访问 MTP 设备 |

---

## 快速链接

- [官方文档](https://libfuse.github.io/doxygen/index.html)
- [GitHub 仓库](https://github.com/libfuse/libfuse)
- [OpenHarmony 组件说明](README_OpenHarmony.md)

---

## 更新日志

- **2026-02-08**: 初始文档创建，完成 Phase 0 评估
