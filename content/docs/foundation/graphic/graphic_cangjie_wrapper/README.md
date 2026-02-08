# graphic_cangjie_wrapper Wiki

## 文档概述

本 Wiki 面向 OpenHarmony 仓库 `graphic_cangjie_wrapper` 的工程文档，旨在帮助开发者快速理解项目定位、架构设计、API 接口、构建配置及安全风险。

## 覆盖范围

| 文档 | 内容 | 状态 |
|-----|-----|-----|
| [README](README.md) | 文档说明与导航 | ✅ 完成 |
| [SUMMARY](SUMMARY.md) | 全站导航与阅读顺序 | ✅ 完成 |
| [01_Overview](01_Overview.md) | 项目定位、边界、核心能力 | ✅ 完成 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型 | ✅ 完成 |
| [03_N-API](03_N-API.md) | Cangjie API 清单与调用链 | ✅ 完成 |
| [04_Inner_API](04_Inner_API.md) | 模块接口与依赖方向 | ✅ 完成 |
| [05_Build](05_Build.md) | GN Targets 与编译产物 | ✅ 完成 |
| [06_Security](06_Security.md) | 安全风险评审 | ✅ 完成 |
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 | ✅ 完成 |

## 更新方式

当代码发生以下变更时，需同步更新 Wiki：

| 变更类型 | 影响文档 |
|---------|---------|
| 新增/删除/修改 Cangjie API | `03_N-API.md`, `appendix/Callgraphs.md` |
| 新增/删除 BUILD.gn target | `05_Build.md` |
| 新增/修改模块依赖 | `04_Inner_API.md`, `02_Architecture.md` |
| 新增安全风险或修复 | `06_Security.md` |

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: `bundle.json:version = 6.1`
- **API Level**: 22
- **SysCapability**: `SystemCapability.Graphic.Graphic2D.ColorManager.Core`

## 相关链接

- [OpenHarmony Graphic Subsystem](https://gitcode.com/openharmony/graphic_graphic_2d)
- [Cangjie Ark Interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- [色彩管理开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/graphics/cj-color-manager-development-guide.md)
