# OpenHarmony TEE OS Kernel - Wiki 文档中心

> **生成时间**: 2026-02-06
> **项目**: OpenHarmony tee_os_kernel (OpenTrustee TEE OS Kernel)
> **架构**: 基于 ChCore 的微内核 TEE 操作系统
> **许可证**: Mulan PSL v2

---

## 文档导航

本 Wiki 提供 OpenHarmony TEE OS Kernel（tee_os_kernel 仓库）的完整技术文档，包括：
- 项目概览与架构
- 目录结构与模块职责
- 内核与用户态 API
- 构建系统与编译产物
- 安全机制与风险评审
- 常见问题与调试指南

**新人阅读顺序**（推荐）：
1. [项目概览](00_Overview.md) - 了解项目定位、核心能力、关键概念
2. [目录结构](01_Directory_Structure.md) - 熟悉代码组织与模块划分
3. [架构设计](02_Architecture.md) - 理解微内核架构、组件关系、数据流
4. [系统调用接口](03_Syscall_Interfaces.md) - 掌握内核暴露的 API（**替代 N-API**）
5. [内部 API](04_Internal_APIs.md) - 了解模块间接口与依赖关系
6. [GN Targets](05_GN_Targets.md) - 理解构建系统与产物
7. [编译产物](06_Build_Artifacts.md) - 了解最终输出与安装路径
8. [攻击面分析](05_AttackSurface.md) - **安全研究员必读**：外部输入入口、敏感操作、信任边界
9. [安全评审](07_Security_Review.md) - 安全机制、可被利用点、修复建议
10. [常见问题](08_FAQ.md) - 构建、运行、调试指南

**双路线导航**：
- 📚 **新人学习路线**: 00 → 01 → 02 → 03 → 04 → 05_GN → 06 → 08
- 🔒 **安全研究路线**: 00 → 05_AttackSurface → 07 → 03 → 02

---

## 文档清单

| 文档 | 状态 | 关键内容 |
|------|------|----------|
| [Wiki 导航](SUMMARY.md) | 📚 | **推荐阅读顺序、快速参考、模块化学习路径** |
| [项目概览](00_Overview.md) | ✅ | 项目定位、边界、核心能力、运行环境、关键概念 |
| [目录结构](01_Directory_Structure.md) | ✅ | 目录树、模块职责、文件分布 |
| [架构设计](02_Architecture.md) | ✅ | 组件图、数据流、线程模型、关键时序 |
| [系统调用接口](03_Syscall_Interfaces.md) | ✅ | **256 个系统调用**（包括 TEE 专用调用）、参数校验、错误码 |
| [内部 API](04_Internal_APIs.md) | ✅ | 模块接口、依赖方向、稳定性标注 |
| [攻击面分析](05_AttackSurface.md) | 🆕 | **安全专项**：外部输入、敏感操作、信任边界、攻击向量 |
| [GN Targets](05_GN_Targets.md) | ✅ | GN 目标梳理、类型、依赖、产物、开关 |
| [编译产物](06_Build_Artifacts.md) | ✅ | .so/.a/.hap/可执行文件、安装路径、加载关系 |
| [安全评审](07_Security_Review.md) | ✅ | 攻击面、信任边界、可被利用点、修复建议 |
| [常见问题](08_FAQ.md) | ✅ | 构建/运行/调试问题与定位路径 |
| **项目评估** | 🆕 | `_work/ASSESSMENT.md` - 项目画像与文档策略 |
| **代码证据** | 🆕 | `_work/NOTES.md` - 代码证据汇总与安全分析 |
| [内部 API](04_Internal_APIs.md) | - | 模块接口、依赖方向、稳定性标注 |
| [GN Targets](05_GN_Targets.md) | - | GN 目标梳理、类型、依赖、产物、开关 |
| [编译产物](06_Build_Artifacts.md) | - | .so/.a/.hap/可执行文件、安装路径、加载关系 |
| [安全评审](07_Security_Review.md) | - | 攻击面、信任边界、可被利用点、修复建议 |
| [常见问题](08_FAQ.md) | - | 构建/运行/调试问题与定位路径 |

---

## 重要说明

### ⚠️ N-API 声明

**本项目（tee_os_kernel）是 TEE 微内核，不存在 N-API（Node-API）接口。**

对外 API 通过 **系统调用（Syscall）** 和 **IPC 消息传递** 提供，而非 JavaScript/N-API 绑定。

- **N-API 相关内容**: 本仓库无 N-API 代码
- **实际 API 层**: 256 个系统调用 + IPC 通道 + TEE 专用调用
- **JS 交互层**: 由 `tee_os_framework` 仓库提供（不在本仓库范围内）

### 📚 相关仓库

- [tee_os_framework](https://gitcode.com/openharmony/tee_tee_os_framework) - TEE 框架层（包含可能的 JS/N-API 接口）

### 🔒 安全说明

本文档基于代码证据进行安全分析，所有结论均标注文件路径和代码位置。安全风险评审覆盖：
- 输入校验
- 权限/能力系统
- 内存安全
- TrustZone/SMC 机制
- TEE 特有安全机制

### 📝 证据原则

本文档遵循"先读代码再写文档"原则：
- ✅ 有关键结论必有文件路径（必要时含行号）
- ✅ 有关键结论必有符号名（函数/类/宏/target）
- ❌ 无证据结论标注 `TODO(需确认)`

---

## 文档覆盖范围

### ✅ 已覆盖

- 微内核架构（ChCore）
- ARM64 平台支持（RK3568, RK3399）
- TrustZone 集成（OP-TEE Dispatcher）
- 能力（Capability）系统
- 系统调用接口（256 个）
- IPC 通信机制（Connection + Channel）
- 文件系统（VFS + tmpfs）
- 进程管理（procmgr）
- 内存管理（PMO + VMSpace）
- 线程调度（PBRR/RR）
- 构建系统（Make + GN）

### ❌ 未覆盖

- **测试相关内容**（已排除：test/, unittest/, fuzz/, *_test.*）
- **tee_os_framework**（独立仓库）
- **N-API/JS 接口**（不在本仓库）
- **上层应用框架**（由 framework 提供）

---

## 如何更新文档

当代码变更后，建议：

1. **小改动**: 直接更新对应章节的 Markdown 文件
2. **架构变更**: 更新 `02_Architecture.md` 和 `01_Directory_Structure.md`
3. **新增 API**: 更新 `03_Syscall_Interfaces.md`（syscall_num.h）和 `04_Internal_APIs.md`
4. **安全修复**: 更新 `07_Security_Review.md`
5. **构建变更**: 更新 `05_GN_Targets.md` 和 `06_Build_Artifacts.md`

更新后请在 `wiki/_work/NOTES.md` 中记录变更摘要。

---

## 许可证

本文档遵循 [Mulan PSL v2](../LICENSE) 许可证。

**文档维护**: OpenHarmony TEE OS Kernel 开发团队
**最后更新**: 2025-02-07
