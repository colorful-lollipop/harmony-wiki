# OpenHarmony Updater Wiki

## 项目信息

| 属性 | 值 |
|------|-----|
| **组件名称** | @ohos/updater |
| **版本** | 3.2 |
| **子系统** | 升级子系统 (Update Subsystem) |
| **License** | Apache License 2.0 |
| **源码路径** | `//base/update/updater` |

## Wiki 覆盖范围

本文档集为 OpenHarmony Updater 子系统提供完整的工程分析，覆盖以下内容：

- **项目定位与核心能力**
- **目录结构与模块职责**
- **系统架构与数据流**
- **对外 API 接口（N-API/C++）**
- **内部模块与接口**
- **GN 构建系统与编译产物**
- **安全风险评审**
- **常见问题与调试**

## 未覆盖范围

- 测试代码（`test/` 目录下的单元测试、Fuzz 测试、基准测试）
- 详细实现代码解读（仅提供接口和调用链分析）
- 特定芯片平台的适配细节

## 文档生成时间

生成时间: 2026-02-06

## 更新方式

本文档基于代码证据生成，当以下文件变更时需同步更新：
- `bundle.json` - 组件配置变更
- `BUILD.gn` / `.gni` - 构建配置变更
- `interfaces/kits/` - 对外接口变更
- `services/include/` - 内部接口变更

## 阅读指南

### 新人阅读顺序

1. [00_Overview.md](./00_Overview.md) - 项目概览
2. [01_Architecture.md](./01_Architecture.md) - 架构说明
3. [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
4. [03_Public_API.md](./03_Public_API.md) - 对外 API
5. [05_GN_Build.md](./05_GN_Build.md) - 构建系统

### 深度阅读

- [04_Inner_API.md](./04_Inner_API.md) - 内部接口
- [06_Security_Analysis.md](./06_Security_Analysis.md) - 安全分析
- [07_Troubleshooting.md](./07_Troubleshooting.md) - 问题排查
- [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 调用链附录
- [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 配置标志附录

## 术语表

| 术语 | 说明 |
|------|------|
| OTA | Over-The-Air 升级 |
| HOTA | Huawei OTA 升级方式 |
| AB 分区 | Android Bootloader 支持的双分区升级机制 |
| VAB | Virtual AB，Android 虚拟 AB 分区方案 |
| Misc 分区 | 存储升级相关元数据的分区 |
| Updater 分区 | 独立的升级模式分区 |
| Flashd | 刷机模式，支持格式化/擦除/刷写 |
| HDI | Hardware Device Interface，硬件设备接口 |
| GN | Generate Ninja，构建系统 |
| SA | System Ability，系统能力 |

---

**注意**: 本文档所有结论均基于代码证据，引用格式为 `文件路径:行号`。
