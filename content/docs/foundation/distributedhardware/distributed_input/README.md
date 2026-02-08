# Distributed Input Wiki

本文档为 OpenHarmony distributed_input 模块的完整技术文档。

## 文档导航

本 Wiki 提供以下主题的完整技术文档：

- [概览](00_Overview.md) - 项目介绍、核心概念、架构概览
- [项目定位](01_Project_Positioning.md) - 功能边界、核心能力、运行环境、关键概念
- [目录结构](02_Directory_Structure.md) - 模块组织、各目录职责
- [架构设计](03_Architecture.md) - 组件关系、数据流、线程模型、时序图
- [公共 API](04_Public_API.md) - C++ Inner SDK 接口、方法列表、参数说明
- [攻击面分析](05_Attack_Surface.md) - 外部输入、敏感操作、信任边界
- [内部 API](05_Inner_API.md) - 模块间接口、依赖关系、稳定性
- [GN 目标](06_GN_Targets.md) - 编译目标列表、依赖关系、产物映射
- [编译产物](07_Build_Artifacts.md) - 库文件、安装路径、运行时加载
- [安全评审](08_Security_Review.md) - 攻击面、信任边界、风险评估、修复建议

## 阅读建议

### 新人快速入门路径

1. 阅读 [概览](00_Overview.md) - 了解整体架构
2. 阅读 [项目定位](01_Project_Positioning.md) - 理解功能边界
3. 阅读 [目录结构](02_Directory_Structure.md) - 熟悉代码组织
4. 阅读 [架构设计](03_Architecture.md) - 理解组件关系和数据流
5. 阅读 [公共 API](04_Public_API.md) - 了解如何使用本模块
6. 阅读 [安全评审](08_Security_Review.md) - 理解安全机制

### 开发者路径

1. 阅读 [公共 API](04_Public_API.md) - 了解可用的公共 API
2. 阅读 [内部 API](05_Inner_API.md) - 了解模块间接口（如需扩展功能）
3. 阅读 [GN 目标](06_GN_Targets.md) - 了解编译依赖关系
4. 阅读 [安全评审](08_Security_Review.md) - 理解权限和安全要求

### 安全审计员路径

1. 阅读 [安全评审](08_Security_Review.md) - 了解攻击面和威胁模型
2. 阅读 [架构设计](03_Architecture.md) - 理解数据流和信任边界
3. 阅读 [公共 API](04_Public_API.md) - 了解接口的权限要求
4. 检查代码实现（参考本文档中的证据路径）

---

## 重要说明

### N-API 说明

**本项目不提供 N-API/JavaScript 接口。**

根据 [README_zh.md:6](../README_zh.md:6)：
> "分布式输入不提供北向接口，由多模输入子系统提供分布式输入业务接口供开发者调用分布式输入的能力。"

JavaScript/N-API 接口位于独立的 `multimodalinput_input` 模块中，而非本模块。

### 接口类型

本模块使用 OpenHarmony 的 **IPC（进程间通信）** 和 **SA（System Ability）** 机制，而非 N-API：

- **SA 4809**: Source 侧服务（libdinput_source.z.so）
- **SA 4810**: Sink 侧服务（libdinput_sink.z.so）
- **C++ Inner SDK**: `libdinput_sdk.so` - 供多模输入模块调用的内部接口

### 证据要求

本文档中的所有关键结论都基于代码证据：
- 文件路径（必要时含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述
- 无法确认的内容标注 `TODO(需确认)`

---

## 更新记录

- **生成时间**: 2026-02-06 15:08:55
- **模块版本**: 3.2
- **代码仓库**: [distributed_input](https://gitee.com/openharmony/distributedhardware_distributed_input)

## 如何更新本文档

当代码库发生变更时，按以下流程更新文档：

1. **Phase 0**: 更新 `wiki/_work/NOTES.md` 记录新发现
2. **Phase 1**: 运行代码扫描，更新结构/API/构建信息
3. **Phase 2-7**: 按照本文档的章节结构更新对应内容
4. **Phase 7**: 运行一致性校验
5. **最后**: 更新本文件中的生成时间和变更说明

## 覆盖范围

### 已覆盖
- ✅ 目录结构和模块职责
- ✅ 架构设计和组件关系
- ✅ 公共 C++ Inner SDK API
- ✅ 内部模块间接口
- ✅ GN 构建目标和依赖
- ✅ 编译产物和安装路径
- ✅ IPC 和 SA 接口
- ✅ 权限和安全机制
- ✅ 安全风险评审
- ✅ 常见问题和调试建议

### 未覆盖
- ❌ JavaScript/N-API 层（位于 multimodalinput_input 模块）
- ❌ 测试相关内容（按要求排除）
- ❌ 详细的 API 用法示例代码（仅在公共 API 章节提供接口说明）

---

## 快速链接

- [OpenHarmony 官方文档](https://www.openharmony.cn/mainPlay)
- [分布式硬件子系统](https://gitee.com/openharmony/distributedhardware_distributed_hardware_fwk)
- [多模输入子系统](https://gitee.com/openharmony/multimodalinput_input)
- [设备管理](https://gitee.com/openharmony/distributedhardware_device_manager)
