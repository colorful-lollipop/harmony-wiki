# Bundle Framework Wiki - 导航目录

本文档提供全站导航，帮助开发者快速定位所需内容。

## 快速开始

| 场景 | 推荐章节 |
|------|----------|
| 新人入门 | [01_Overview](01_Overview.md) → [01_Directory_Structure](01_Directory_Structure.md) → [02_Architecture](02_Architecture.md) |
| 使用 JS API | [04_Interface](04_Interface.md) 或 [03_N-API_Reference](03_N-API_Reference.md) |
| 使用 Native API | [04_Interface](04_Interface.md) 或 [04_Inner_API](04_Inner_API.md) |
| 理解代码结构 | [03_CodeMap](03_CodeMap.md) - 功能→文件映射 |
| 安全评估 | [05_AttackSurface](05_AttackSurface.md) → [06_SecurityReview](06_SecurityReview.md) |
| 构建/编译 | [07_Build](07_Build.md) |
| 内部实现 | [08_Internals](08_Internals.md) |
| 问题排查 | [08_FAQ](08_FAQ.md) |

---

## 完整目录

### 基础信息

- [README](README.md) - 文档说明、覆盖范围、更新指南
- [01_Overview](01_Overview.md) - 项目定位、核心能力、关键概念、运行环境
- [SUMMARY](SUMMARY.md) - 全站导航（本文档）

### 结构与架构

- [01_Directory_Structure](01_Directory_Structure.md) - 目录结构、模块职责
- [02_Architecture](02_Architecture.md) - 组件图、数据流、线程模型、关键时序
- [03_CodeMap](03_CodeMap.md) - 代码导航图、功能→文件映射

### 接口参考

- [03_N-API_Reference](03_N-API_Reference.md) - JS/N-API 接口、API 清单表
- [04_Inner_API](04_Inner_API.md) - Inner API、模块接口
- [04_Interface](04_Interface.md) - 接口汇总（N-API + IPC + 配置）

### 构建与部署

- [07_Build](07_Build.md) - GN Targets、构建配置、编译产物、Feature 开关

### 内部实现

- [08_Internals](08_Internals.md) - 核心类职责、API 契约、资源生命周期

### 安全与运维

- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析、外部输入、敏感操作、信任边界
- [06_SecurityReview](06_SecurityReview.md) - 安全风险评估、漏洞详情、修复建议
- [07_Security_Review](07_Security_Review.md) - 安全概览、检查清单
- [08_FAQ](08_FAQ.md) - 常见问题、构建问题、运行时问题、调试方法

---

## 模块快速索引

### N-API 模块 (JS 接口)

| 模块 | 命名空间 | 主要功能 | 章节 |
|------|----------|----------|------|
| bundleManager | bundleManager | 包管理器主接口 | 03_N-API_Reference |
| bundle | bundle | 包信息查询 | 03_N-API_Reference |
| installer | bundleInstaller | 安装器 | 03_N-API_Reference |
| appControl | appControl | 应用控制 | 03_N-API_Reference |
| overlay | overlay | 叠加包管理 | 03_N-API_Reference |
| shortcutManager | shortcutManager | 快捷方式管理 | 03_N-API_Reference |
| bundleMonitor | bundleMonitor | 包状态监控 | 03_N-API_Reference |
| bundleResource | bundleResource | 包资源管理 | 03_N-API_Reference |
| launcherBundleManager | launcherBundleManager | 启动器包管理 | 03_N-API_Reference |
| launcher | launcher | 启动器管理 | 03_N-API_Reference |
| freeInstall | freeInstall | 自由安装 | 03_N-API_Reference |
| defaultApp | defaultApp | 默认应用管理 | 03_N-API_Reference |
| package | package | 包工具 | 03_N-API_Reference |
| zlib | zlib | ZIP 文件操作 | 03_N-API_Reference |

### 服务模块

| 模块 | 路径 | 职责 | 章节 |
|------|------|------|------|
| BundleMgrService | services/bundlemgr/ | 包管理主服务 | 02_Architecture, 04_Inner_API, 08_Internals |
| BundleInstaller | services/bundlemgr/ | 安装逻辑 | 02_Architecture, 08_Internals |
| BundleDataMgr | services/bundlemgr/ | 包数据管理 | 02_Architecture, 08_Internals |
| Verify | services/bundlemgr/verify/ | 签名校验 | 02_Architecture, 06_SecurityReview |
| Installd | services/bundlemgr/installd/ | 文件操作 | 02_Architecture, 05_AttackSurface |

---

## 系统能力 (SA)

| SA ID | 进程 | 库 | 职责 | 章节 |
|-------|------|-----|------|------|
| 401 | foundation | libbms.z.so | BundleMgrService | 02_Architecture, 04_Inner_API, 08_Internals |
| 511 | installs | libinstalls.z.so | InstalldService | 02_Architecture, 05_AttackSurface |

---

## 文档变更日志

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 1.0.0 | 2026-02-06 | 初始版本 |
| 1.1.0 | 2026-02-07 | 新增 03_CodeMap, 05_AttackSurface, 06_SecurityReview, 07_Build, 08_Internals |
|      |          | 拆分 07_Security_Review 为攻击面和风险评估两个独立文档 |

---

## 反馈与贡献

- **Issue**: 在代码仓库创建 Issue
- **文档改进**: 提交 PR 修改 wiki/ 目录
- **联系方式**: 参见代码仓库 README
