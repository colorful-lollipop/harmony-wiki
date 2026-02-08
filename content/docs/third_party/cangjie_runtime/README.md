# Cangjie Runtime Wiki

## 仓颉运行时与标准库 OpenHarmony 集成文档

本文档详细介绍 Cangjie Runtime（仓颉运行时）在 OpenHarmony 中的集成方式、适配细节和使用方法。

---

## 快速导航

### 核心文档

| 文档 | 内容概述 |
|-----|---------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、功能概述、OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本项目采用预编译模式，无传统 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 结构、构建适配、OH 特殊配置 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、依赖图 |
| [05_API_Differences.md](./05_API_Differences.md) | OH 平台 API 差异、不支持的功能 |
| [06_Security.md](./06_Security.md) | 安全风险分析、CVE 状态、升级建议 |

### 工作文档

- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估报告（信息收集阶段产出）
- [_work/NOTES.md](./_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](./_work/PLAN.md) - 任务进度

---

## 库概览

### 基本信息

| 属性 | 值 |
|-----|-----|
| **名称** | cangjie_runtime |
| **OH 组件** | @ohos/cangjie_runtime |
| **版本** | 1.1.0-alpha.69 (OH 6.1) |
| **许可证** | Apache-2.0 with Runtime Library Exceptions |
| **子系统** | thirdparty |

### 功能定位

仓颉运行时是**仓颉编程语言 Native 后端（CJNative）的核心组件**，为仓颉程序提供运行时支撑：

- **垃圾回收（GC）**：全并发、低延迟内存管理
- **线程管理（CJThread）**：轻量级线程调度
- **异常处理**：Exception 和 Error 分级机制
- **模块加载**：包级代码加载与反射
- **FFI**：与 C/ArkTS 互操作
- **标准库**：31+ 个标准库模块

### OpenHarmony 适配特点

本项目采用**预编译库（Prebuilt）模式**，与传统第三方库的直接源码编译不同：

```
传统模式：源码 → Patch → 在 OH 中编译 → 产物
cangjie_runtime：外部构建 → 预编译二进制 → OH 分发
```

**OH 特有适配**：
- 使用 Hilog 日志系统替代标准输出
- 适配 OH 内存管理策略（堆大小限制）
- 启用 OH 安全特性（CFI、PAC-RET）
- 遵循 OH 命名空间隔离机制
- 支持 ARM64、x86_64、ARM32（计划中）

---

## 阅读建议

### 如果您是...

**系统开发者**：
1. 阅读 [01_Overview.md](./01_Overview.md) 了解架构
2. 阅读 [03_Build_Integration.md](./03_Build_Integration.md) 理解构建流程
3. 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 查看依赖关系

**安全工程师**：
1. 直接阅读 [06_Security.md](./06_Security.md)
2. 参考 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 了解组件依赖

**应用开发者**：
1. 阅读 [01_Overview.md](./01_Overview.md) 了解功能
2. 阅读 [05_API_Differences.md](./05_API_Differences.md) 了解平台限制
3. 参考仓颉官方文档：https://cangjie-lang.cn/

**维护者**：
1. 完整阅读所有文档
2. 重点关注 [02_Patches.md](./02_Patches.md) 中的升级注意事项
3. 参考 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 中的证据清单

---

## 相关链接

- **上游项目**：https://cangjie-lang.cn/
- **相关仓库**：
  - [cangjie_compiler](https://gitcode.com/openharmony-sig/third_party_cangjie_compiler)
  - [cangjie_tools](https://gitcode.com/openharmony-sig/third_party_cangjie_tools)
  - [cangjie_stdx](https://gitcode.com/openharmony-sig/third_party_cangjie_stdx)
- **构建指南**：https://gitcode.com/Cangjie/cangjie_build
- **官方文档**：https://cangjie-lang.cn/docs

---

*本文档由 OpenHarmony Wiki Agent 自动生成*
*最后更新：2025年2月*
