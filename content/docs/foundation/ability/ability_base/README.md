# ability_base Wiki 文档

## 概述

本文档档为 **ability_base**（元能力基础部件）提供完整的架构参考、API 说明、构建配置和安全分析，面向需要深入了解 OpenHarmony Ability 子系统基础组件的开发者。

**适用范围**:
- OpenHarmony 标准系统（standard system type）
- API 版本：15+
- 组件版本：3.1

**生成时间**: 2026-02-07 12:30

**文档语言**: 中文

---

## 覆盖范围

### ✅ 已覆盖内容
- 项目定位、核心能力、模块职责
- 目录结构与代码组织
- Native C++ API（Want、Configuration、URI、Base等）- 03_Native_CPP_API.md
- C NDK API（仅 Want 模块）- 04_C_NDK_API.md
- GN 构建系统（targets、依赖、编译产物）
- 内部架构（类关系、数据流、线程模型）
- 安全风险评审（输入验证、IPC、内存安全）
- 项目评估报告（ASSESSMENT.md）

### ❌ 未覆盖内容

#### N-API (JavaScript API)
**说明**: 此仓库不包含 N-API 绑定。根据 README.md:22，N-API 代码应在 `frameworks/js/napi/` 目录，但该目录在此仓库中不存在。

**实际位置**: N-API 绑定可能在以下位置：
- **ability_runtime** 组件：`/foundation/ability/ability_runtime/` - 提供 Ability 子系统的 JS API 层
- **独立的 NAPI 仓库**：可能存在单独的 `ability_base_napi` 仓库

**建议**: 查阅 ability_runtime 仓库的 N-API 绑定实现，或参考 OpenHarmony 官方文档：
- 开发指南：https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/application-models/Readme-CN.md

#### 运行时行为
- 组件启动流程（由 ability_runtime 实现）
- 权限验证实际执行（在 ability_runtime 或系统服务层）
- Session 生命周期管理（由 AbilityManager 等服务管理）

#### 测试相关内容
本文档根据约束**忽略测试相关内容**，不引用测试文件或测试代码作为业务证据。

---

## 如何随代码更新文档

### 文档更新时机
建议在以下情况更新文档：
1. **新增 API 或修改现有 API 签名**：更新 Native API、C NDK API 文档
2. **GN 构建配置变更**：新增/删除 targets、修改依赖关系
3. **架构重大调整**：新增模块、修改类关系、改变数据流
4. **安全相关代码变更**：修改验证逻辑、新增 IPC 接口、改变权限模型

### 更新步骤
1. **运行 Phase 1 扫描**：重新扫描目录结构、GN 配置、关键代码
2. **更新 NOTES.md**：记录新的发现和证据路径
3. **更新对应文档**：根据变更范围修改相关 MD 文件
4. **更新本文档**：同步更新"覆盖范围"和"生成时间"

### 证据验证原则
所有文档中的关键结论必须满足：
- ✅ **有代码证据**：提供文件路径（含行号）、函数名、类名
- ✅ **可直接验证**：读者可通过代码审查确认结论
- ❌ **禁止凭空猜测**：无法确认的内容必须标注 `TODO(需确认)`

---

## 文档结构

### 首页与导航
- **[SUMMARY.md](SUMMARY.md)**：全站导航 + 新人阅读顺序
- **[index.md](index.md)**：项目定位、核心能力、运行环境、关键概念

### 架构与设计
- **[01_Directory_Structure.md](01_Directory_Structure.md)**：目录结构、模块职责、文件组织
- **[02_Architecture.md](02_Architecture.md)**：架构说明、组件图、数据流、线程模型、关键时序

### API 文档
- **[03_Native_CPP_API.md](03_Native_CPP_API.md)**：Native C++ API、Want、Configuration、URI、Base 类型
- **[04_C_NDK_API.md](04_C_NDK_API.md)**：C API (NDK)、OH_AbilityBase_* 函数

### 构建与编译
- **[05_GN_Build.md](05_GN_Build.md)**：GN targets、依赖、编译产物、构建配置

### 安全与质量
- **[06_Security_Review.md](06_Security_Review.md)**：攻击面、信任边界、可被利用点、修复建议

### 附录
- **[appendix/Callgraphs.md](appendix/Callgraphs.md)**：关键调用链（入口→核心逻辑）
- **[appendix/Config_Flags.md](appendix/Config_Flags.md)**：关键宏/feature flags

---

## 新人阅读顺序

为快速理解 ability_base 组件，建议按以下顺序阅读：

1. **[index.md](index.md)** - 了解项目定位和核心概念（10 分钟）
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 熟悉代码组织（15 分钟）
3. **[02_Architecture.md](02_Architecture.md)** - 理解架构和数据流（30 分钟）
4. **[03_Native_CPP_API.md](03_Native_CPP_API.md)** - 学习 Want API（核心）（45 分钟）
5. **[05_GN_Build.md](05_GN_Build.md)** - 了解构建系统（20 分钟）
6. **[06_Security_Review.md](06_Security_Review.md)** - 掌握安全考虑（20 分钟）

**总时长**: 约 2.5 小时

---

## 术语表

| 术语 | 定义 |
|------|------|
| **Ability** | OpenHarmony 中的可执行单元，类似 Android 的 Activity/Service |
| **Want** | Ability 之间通信的消息容器，携带启动参数和数据 |
| **Operation** | Want 中的目标操作描述（action、entities、flags、URI） |
| **WantParams** | Want 中的类型安全键值存储 |
| **Configuration** | 系统环境配置信息（语言、主题、显示等） |
| **URI** | 统一资源标识符，用于跨设备资源访问 |
| **SessionInfo** | UI 会话元数据，管理窗口/会话状态 |
| **Parcelable** | OpenHarmony IPC 序列化接口 |
| **IRemoteObject** | 远程对象接口，用于 IPC |
| **Inner API** | 系统内部组件间接口（platformsdk、sasdk） |
| **Native Kit** | 原生 C++ SDK 接口（platformsdk） |
| **C NDK** | Native Development Kit 的 C 接口（ndk） |

---

## 相关资源

### 官方文档
- **Ability 子系统**: https://gitee.com/openharmony/ability_ability_base
- **开发指南**: https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/application-models/Readme-CN.md
- **API 参考**: https://gitee.com/openharmony/docs (搜索 Want、Configuration)

### 相关仓库
- **ability_runtime**: `https://gitee.com/openharmony/ability_ability_runtime` - Ability 运行时
- **dmsfwk**: `https://gitee.com/openharmony/ability_dmsfwk` - 分布式管理服务
- **form_fwk**: `https://gitee.com/openharmony/ability_form_fwk` - 卡片框架
- **idl_tool**: `https://gitee.com/openharmony/ability_idl_tool` - IDL 工具

---

## 工作区说明

本文档在生成过程中使用了 `wiki/_work/` 工作区：

- **[NOTES.md](_work/NOTES.md)**：事实记录、证据路径、发现、TODO
- **[PLAN.md](_work/PLAN.md)**：任务拆解、进度追踪、阻塞处理

这些工作区文件记录了文档生成的完整过程，可作为后续更新的参考。

---

## 许可证

本文档遵循 [Apache License 2.0](../LICENSE)，与 ability_base 组件许可证一致。

---

## 联系与反馈

如有问题或建议，请：
1. 在相关仓库提 Issue
2. 联系 OpenHarmony 社区
3. 查阅官方文档获取最新信息

---

**文档版本**: v1.0
**最后更新**: 2026-02-07 12:30
**维护者**: OpenHarmony Ability 子系统
