# 概览

本文档描述 OpenHarmony **applications_launcher** 项目的工程 Wiki，涵盖项目架构、模块职责、API 接口、构建配置和安全分析。

> **项目定位**: OpenHarmony 系统桌面应用，作为系统人机交互的首要入口
> 
> **技术栈**: ArkTS + Stage 模型 + hvigor 构建

## 覆盖范围

本 Wiki 包含以下内容：

| 文档 | 说明 |
|------|------|
| [README](README.md) | 本文档，说明、生成信息 |
| [SUMMARY](SUMMARY.md) | 全站导航与阅读顺序 |
| [概览](00_Overview.md) | 项目定位、技术栈、核心能力 |
| [目录结构](01_Directory_Structure.md) | 模块划分与职责 |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型 |
| [对外 API](03_APIs.md) | ArkTS 模块导出清单（已验证） |
| [内部 API](04_Inner_API.md) | 模块接口与依赖方向 |
| [构建配置](05_Build.md) | hvigor Targets 与编译产物 |
| [安全评审](06_Security.md) | 风险分析与修复建议（已验证） |
| [调用链图谱](appendix/Callgraphs.md) | 关键调用链 |

## 关键发现

### N-API 状态
- **本项目不包含传统 N-API**（C/C++ 导出层）
- 采用纯 ArkTS + Stage 模型开发
- 所有模块导出通过 `index.ts` 实现

### 模块结构
- **12 个模块**: 10 个 HAR 模块 + 2 个 HAP 模块（phone/pad）
- **三层架构**: product → feature → common
- **构建系统**: hvigor（类 Gradle）

### 权限配置
- **11 项系统权限**（已验证）
- 包含高敏感权限：`INJECT_INPUT_EVENT`, `MANAGE_MISSIONS` 等

## 未覆盖范围

- **测试代码**：不包括 `test/`、`tests/` 等测试目录
- **运行时产物**：不包括实际 `.hap` 文件分析
- **外部依赖文档**：不包括 `@ohos/*` 系统 API 详细说明

## 文档更新

当项目代码发生变更时：

1. 扫描相关模块的 `index.ts` 导出 → 更新 [03_APIs.md](03_APIs.md)
2. 更新架构图 → 更新 [02_Architecture.md](02_Architecture.md)
3. 修改构建配置 → 更新 [05_Build.md](05_Build.md)
4. 新增安全风险 → 更新 [06_Security.md](06_Security.md)
5. 新增模块 → 更新 [01_Directory_Structure.md](01_Directory_Structure.md) 和 [04_Inner_API.md](04_Inner_API.md)

## 代码证据

本 Wiki 所有关键结论均基于代码验证：

| 验证项 | 证据位置 |
|--------|----------|
| MainAbility 继承 ServiceExtension | `product/phone/src/main/ets/MainAbility/MainAbility.ts:44` |
| 模块配置 (12 模块) | `build-profile.json5:26-171` |
| 权限配置 (11 项) | `product/phone/src/main/module.json5:46-80` |
| common 模块导出 | `common/index.ts:16-103` |
| feature 模块导出 | `feature/*/index.ts` |

## 版本信息

**最后更新**: 2026-02-06
**生成工具**: OpenHarmony Wiki Agent
**验证状态**: 已完成代码验证（见 `_work/NOTES.md`）
