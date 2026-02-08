# Wiki 导航与阅读指南

本文档提供完整的 Wiki 站点导航，建议按以下顺序阅读以快速理解项目。

---

## 推荐阅读顺序

### 🚀 快速入门路线（30 分钟）

适合：需要快速了解项目定位和核心能力的开发者

1. **[项目概述 (00_Overview.md)](./00_Overview.md)**
   - 了解项目定位和核心能力
   - 理解功能边界和限制

2. **[API 参考 (20_API_Reference.md)](./20_API_Reference.md)**
   - 浏览公开的 Cangjie API
   - 查看使用示例和错误码

3. **[架构概览 (10_Architecture.md)](./10_Architecture.md)**
   - 理解组件关系
   - 查看数据流图

---

### 🔧 深度开发路线（2 小时）

适合：需要进行二次开发或集成的开发者

1. **[项目概述 (00_Overview.md)](./00_Overview.md)**
2. **[架构说明 (10_Architecture.md)](./10_Architecture.md)**
   - 深入理解组件图和数据流
   - 理解线程模型
3. **[API 参考 (20_API_Reference.md)](./20_API_Reference.md)**
   - 掌握所有公开 API 的详细参数
   - 理解 FFI 绑定机制
4. **[构建系统 (30_GN_Build.md)](./30_GN_Build.md)**
   - 理解 GN 目标结构
   - 掌握产物路径和依赖关系
5. **[安全分析 (40_Security_Analysis.md)](./40_Security_Analysis.md)**
   - 理解攻击面和信任边界
   - 掌握安全使用注意事项

---

### 🔍 问题排查路线

适合：遇到具体问题需要定位的开发者

- **构建问题** → [构建系统 (30_GN_Build.md)](./30_GN_Build.md) + [附录: 调用链](./appendix/Callgraphs.md)
- **API 调用失败** → [API 参考 (20_API_Reference.md)](./20_API_Reference.md) 错误码章节
- **崩溃/异常** → [安全分析 (40_Security_Analysis.md)](./40_Security_Analysis.md) + [架构说明 (10_Architecture.md)](./10_Architecture.md) 错误处理章节
- **性能问题** → [架构说明 (10_Architecture.md)](./10_Architecture.md) 线程模型章节

---

## 文档索引

### 核心文档

| 文档 | 内容 | 目标读者 |
|------|------|----------|
| [00_Overview.md](./00_Overview.md) | 项目定位、功能边界、核心概念 | 所有开发者 |
| [10_Architecture.md](./10_Architecture.md) | 架构图、数据流、线程模型 | 架构师、高级开发者 |
| [20_API_Reference.md](./20_API_Reference.md) | Cangjie API 清单、参数、错误码 | 应用开发者 |
| [30_GN_Build.md](./30_GN_Build.md) | GN 目标、编译产物、依赖 | 构建工程师 |
| [40_Security_Analysis.md](./40_Security_Analysis.md) | 攻击面、风险点、修复建议 | 安全工程师 |

### 附录文档

| 文档 | 内容 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链 (JS → Cangjie → FFI → Native) |

---

## 快速查找

### 按主题查找

- **API 列表** → [API 参考 - 公开 API 清单](./20_API_Reference.md#公开-api-清单)
- **错误码** → [API 参考 - 错误码定义](./20_API_Reference.md#错误码定义)
- **构建产物** → [构建系统 - 编译产物清单](./30_GN_Build.md#编译产物清单)
- **安全风险** → [安全分析 - 可被利用点清单](./40_Security_Analysis.md#可被利用点清单)

### 按符号名查找

| 符号 | 位置 |
|------|------|
| `getValue` | [API 参考 - 核心 API](./20_API_Reference.md#getvalue) |
| `DomainName` | [API 参考 - 枚举定义](./20_API_Reference.md#domainname-枚举) |
| `Date` | [API 参考 - 枚举定义](./20_API_Reference.md#date-枚举) |
| `Display` | [API 参考 - 枚举定义](./20_API_Reference.md#display-枚举) |
| `FfiSettingsGetValue` | [架构说明 - FFI 绑定层](./10_Architecture.md#ffi-绑定层) |
| `ohos.settings` | [构建系统 - GN Targets](./30_GN_Build.md#ohossettings) |
| `kit.BasicServicesKit` | [构建系统 - GN Targets](./30_GN_Build.md#kitbasicserviceskit) |

---

## 外部链接

### 上游依赖

- [cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) - Cangjie API 注解和异常类
- [ability_cangjie_wrapper](https://gitcode.com/openharmony-sig/ability_ability_cangjie_wrapper) - Ability 上下文能力
- [hiviewdfx_cangjie_wrapper](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper) - 日志接口

### 下游应用

- [applications_settings](https://gitcode.com/openharmony/applications_settings) - Settings 应用

### 官方文档

- [Cangjie Settings API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/BasicServicesKit/cj-apis-settings.md)
- [OpenHarmony 贡献指南](https://gitcode.com/openharmony/docs/blob/master/en/contribute/how-to-contribute.md)
