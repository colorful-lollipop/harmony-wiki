# 文档导航

> 本文档提供 OpenHarmony Hisilicon 板卡仓库的完整导航和新人阅读路径

---

## 📖 新人阅读顺序（推荐路径）

### 第一阶段：快速了解（30 分钟）
1. **[00_Overview.md](00_Overview.md)** - 项目概览、板卡介绍、核心能力
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构、模块职责、文件组织

### 第二阶段：深入理解（1-2 小时）
3. **[02_Architecture.md](02_Architecture.md)** - 架构设计、组件关系、数据流
4. **[04_GN_Targets.md](04_GN_Targets.md)** - 构建系统、目标依赖、编译流程

### 第三阶段：实践操作（按需）
5. **[05_Build_Artifacts.md](05_Build_Artifacts.md)** - 编译产物、安装路径、运行时加载
6. **[07_FAQ.md](07_FAQ.md)** - 常见问题、调试技巧、问题定位

### 第四阶段：安全审查（安全工程师）
7. **[06_Security_Review.md](06_Security_Review.md)** - 攻击面分析、安全风险、修复建议

---

## 📚 文档结构

### 核心文档

| 文档 | 用途 | 适用对象 | 阅读时间 |
|------|------|---------|---------|
| [00_Overview.md](00_Overview.md) | 项目定位、边界、核心能力 | 所有人 | 10 min |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构、模块职责、文件组织 | 开发者 | 15 min |
| [02_Architecture.md](02_Architecture.md) | 架构设计、组件关系、数据流 | 架构师、高级开发者 | 30 min |
| [03_Inner_API.md](03_Inner_API.md) | 内部 API、模块接口、稳定性 | 驱动开发者 | 20 min |
| [04_GN_Targets.md](04_GN_Targets.md) | 构建系统、目标依赖、编译流程 | 构建工程师 | 25 min |
| [05_Build_Artifacts.md](05_Build_Artifacts.md) | 编译产物、安装路径、运行时 | 运维、测试工程师 | 15 min |
| [06_Security_Review.md](06_Security_Review.md) | 攻击面、安全风险、修复建议 | 安全工程师 | 30 min |
| [07_FAQ.md](07_FAQ.md) | 常见问题、调试技巧 | 所有人 | 按需 |

### 附录文档

| 文档 | 用途 | 适用对象 |
|------|------|---------|
| [appendix/Build_Config_Flags.md](appendix/Build_Config_Flags.md) | 关键编译选项、Feature flags | 构建工程师 |

---

## 🔍 按角色查找文档

### 新手入门
- [00_Overview.md](00_Overview.md) - 了解这是什么仓库
- [01_Directory_Structure.md](01_Directory_Structure.md) - 熟悉代码组织
- [07_FAQ.md](07_FAQ.md) - 解决常见问题

### 驱动开发者
- [02_Architecture.md](02_Architecture.md) - 理解系统架构
- [03_Inner_API.md](03_Inner_API.md) - 查看内部接口
- [04_GN_Targets.md](04_GN_Targets.md) - 了解如何编译驱动

### 系统工程师
- [02_Architecture.md](02_Architecture.md) - 整体架构
- [05_Build_Artifacts.md](05_Build_Artifacts.md) - 编译产物和运行时
- [04_GN_Targets.md](04_GN_Targets.md) - 构建流程

### 安全工程师
- [06_Security_Review.md](06_Security_Review.md) - 安全风险分析
- [02_Architecture.md](02_Architecture.md) - 理解系统边界

### 构建工程师
- [04_GN_Targets.md](04_GN_Targets.md) - GN 构建系统
- [05_Build_Artifacts.md](05_Build_Artifacts.md) - 输出产物
- [appendix/Build_Config_Flags.md](appendix/Build_Config_Flags.md) - 编译选项

---

## 🏷️ 按主题查找文档

### 项目概览
- [00_Overview.md](00_Overview.md) - 项目定位、核心能力

### 代码结构
- [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构、模块职责

### 架构设计
- [02_Architecture.md](02_Architecture.md) - 组件图、数据流、时序

### 接口与 API
- [03_Inner_API.md](03_Inner_API.md) - 内部 API、依赖关系

### 构建系统
- [04_GN_Targets.md](04_GN_Targets.md) - GN 目标、依赖、产物
- [appendix/Build_Config_Flags.md](appendix/Build_Config_Flags.md) - 编译选项

### 运行时
- [05_Build_Artifacts.md](05_Build_Artifacts.md) - 镜像、库、加载关系

### 安全
- [06_Security_Review.md](06_Security_Review.md) - 攻击面、风险分析

### 实践
- [07_FAQ.md](07_FAQ.md) - 常见问题、调试技巧

---

## ⚠️ 重要说明

### 不在本文档范围内的内容

以下内容不在本 Wiki 的覆盖范围内，请参考其他仓库:

1. **N-API (Node-API) 模块**
   - 本仓库为板卡级，不包含应用层 API
   - 参考: `//foundation/arkui/napi` 和各子系统 N-API 实现

2. **IPC / System Ability**
   - 本仓库不包含系统服务
   - 参考: `//foundation/communication/ipc` 和各 SA 服务

3. **应用层权限管理**
   - 权限管理位于框架层
   - 参考: `//base/security/permission`

4. **业务功能实现**
   - 如相机应用、音频应用等
   - 参考: `//applications` 和各应用子系统

---

## 📝 文档符号说明

- ✅ **已验证**: 结论基于代码证据，有明确文件路径和行号
- ⚠️ **部分确认**: 有证据但不完整，需要进一步验证
- ❌ **未实现**: 功能尚未实现或文档未找到
- 🔧 **TODO**: 需要补充的内容

---

## 🔗 相关资源

- [主 README](README.md) - Wiki 使用说明
- [工作笔记](_work/NOTES.md) - 代码证据和事实记录
- [工作计划](_work/PLAN.md) - 文档生成进度
