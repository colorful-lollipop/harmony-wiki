# applications_cangjie_wrapper Wiki

本文档是 OpenHarmony `applications_cangjie_wrapper` 项目的工程 Wiki，为开发者提供完整的项目架构、API 参考、构建系统和安全分析信息。

## 文档覆盖范围

- **项目概述**：定位、功能边界、运行环境
- **架构说明**：组件关系、数据流、线程模型
- **API 参考**：Cangjie API 接口清单、参数、错误码
- **构建系统**：GN 目标、编译产物、依赖关系
- **安全分析**：攻击面、信任边界、可被利用点评估

## 文档更新方式

本文档基于代码证据（文件路径+符号名+代码片段）手工维护。当代码发生以下变更时，应同步更新对应章节：

- 新增/修改 `@!APILevel` 注解的公开接口
- 新增/修改 `BUILD.gn` 构建目标
- FFI 调用签名变更
- 错误码定义变更
- 安全相关参数校验逻辑变更

## 文档结构

```
wiki/
├── README.md                 # 本文档
├── SUMMARY.md               # 全站导航与阅读路线
├── 00_Overview.md           # 项目定位与核心能力
├── 10_Architecture.md       # 架构设计与数据流
├── 20_API_Reference.md      # Cangjie API 参考
├── 30_GN_Build.md          # 构建系统与编译产物
├── 40_Security_Analysis.md  # 安全风险评审
└── appendix/
    └── Callgraphs.md        # 关键调用链
```

## 生成时间

- **生成日期**: 2026-02-05
- **基于代码版本**: Git HEAD ( applications_cangjie_wrapper @ 6.1 )
- **覆盖文件**: 6 个核心源文件 (kit + ohos/settings，排除 test/mock)

## 术语约定

| 术语 | 说明 |
|------|------|
| Cangjie | 仓颉编程语言，华为自研编程语言 |
| N-API | OpenHarmony 原生 API 绑定机制 |
| FFI | Foreign Function Interface，外部函数接口 |
| GN | Generate Ninja，构建系统 |
| Syscap | System Capability，系统能力标识 |
| UIAbility | OpenHarmony 应用能力组件 |

## 相关文档

- [OpenHarmony Settings 应用 API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/BasicServicesKit/cj-apis-settings.md)
- [cangjie_ark_interop 仓库](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- [ability_cangjie_wrapper 仓库](https://gitcode.com/openharmony-sig/ability_ability_cangjie_wrapper)
- [hiviewdfx_cangjie_wrapper 仓库](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper)
