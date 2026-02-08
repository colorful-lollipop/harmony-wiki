# OpenHarmony Cangjie SDK 文档

## 项目概述

本 Wiki 是 OpenHarmony Cangjie SDK（`@interface/sdk_cangjie`）的工程文档，旨在帮助开发者快速理解项目结构、API 架构、构建系统和安全机制。

- **仓库位置**: `interface/sdk_cangjie`
- **版本**: 6.0
- **许可证**: Apache License 2.0
- **文档生成时间**: 2025-02-06

## 文档覆盖范围

### 核心文档

| 模块 | 状态 | 说明 |
|------|------|------|
| [项目概览](01_Overview.md) | ✅ 已完成 | 项目定位、核心能力、运行环境 |
| [目录结构](02_Directory_Structure.md) | ✅ 已完成 | 模块职责划分（不含测试） |
| [系统架构](03_Architecture.md) | 🔄 进行中 | 组件图、数据流、线程模型 |
| [Kit API 参考](04_Kit_API.md) | 🔄 进行中 | N-API 接口清单、参数校验 |
| [攻击面分析](05_AttackSurface.md) | ✅ 已完成 | 构建脚本攻击面、外部输入、敏感操作 |
| [内部 API](05_Inner_API.md) | ⏳ 待完成 | 模块接口、依赖方向 |
| [GN 构建系统](06_GN_Build.md) | ✅ 已完成 | Targets 列表、产物映射 |
| [编译产物](07_Build_Artifacts.md) | ⏳ 待完成 | .so/.cjo 文件、安装路径 |
| [安全风险评审](08_Security_Review.md) | 🔄 进行中 | API声明安全风险、信任边界、风险点 |
| [常见问题](09_FAQ.md) | ✅ 已完成 | 构建/运行/调试问题 |

## 项目定位

Cangjie SDK 是 OpenHarmony 的原生应用开发 SDK，支持使用 Cangjie 语言开发 OpenHarmony 应用。

### 核心能力

1. **跨平台开发**: 支持 Windows/Linux/Mac-x64/Mac-arm64 平台交叉编译
2. **原生性能**: Cangjie 编译为原生代码，运行效率高
3. **完整 API**: 覆盖 25+ Kit，提供与 ArkTS 等价的 API 能力
4. **互操作**: 支持与 ArkTS、C/C++ 代码互操作

### 运行环境

- **目标设备**: OpenHarmony 标准设备（standard）
- **开发平台**: Windows/Linux/Mac-x64/Mac-arm64
- **不支持**: 在 ohos 平台上直接构建应用

## 文档维护

### 更新方式

当代码仓库发生变化时，需要同步更新 Wiki：

1. **API 变更**: 修改 `.cj.d` 文件后，更新 `04_Kit_API.md`
2. **构建变更**: 修改 `BUILD.gn` 或 `.gni` 后，更新 `06_GN_Build.md` 和 `07_Build_Artifacts.md`
3. **新增 Kit**: 在 `kits/` 添加声明后，更新 `SUMMARY.md` 和相关 Kit 文档

### 贡献指南

欢迎贡献文档改进，请参考 [OpenHarmony 代码贡献指南](https://gitcode.com/openharmony/docs/blob/master/en/contribute/code-contribution.md)。

## 相关链接

- [Cangjie SDK 集成构建指南](../docs/cangjie_sdk_build_guide.md)
- [Cangjie cjo 序列化指南](../docs/cangjie_cjo_serialization_and_deserialization_guide.md)
- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Cangjie 语言文档](https://gitcode.com/Cangjie/cangjie_docs)
