# 目录导航

## 快速入口

- [首页](/README.md)
- [概览与架构](/wiki/00_Overview.md)
- [API 参考](/wiki/01_API_Reference.md)
- [构建系统](/wiki/02_BUILD.md)
- [内部架构](/wiki/03_Architecture.md)
- [安全评审](/wiki/04_Security.md)

## 文档结构

### 核心文档

| 文档 | 内容 | 目标读者 |
|------|------|----------|
| [README](/wiki/README.md) | 项目说明、更新方式、依赖 | 所有开发者 |
| [概览与架构](/wiki/00_Overview.md) | 项目定位、系统架构、目录结构 | 新人入门 |
| [API 参考](/wiki/01_API_Reference.md) | 仓颉 API 清单、参数、错误码 | 应用开发者 |
| [构建系统](/wiki/02_BUILD.md) | GN targets、编译产物、依赖关系 | 构建/集成开发者 |
| [内部架构](/wiki/03_Architecture.md) | 模块边界、FFI 互操作、线程模型 | 贡献者/维护者 |
| [安全评审](/wiki/04_Security.md) | 攻击面、威胁建模、修复建议 | 安全审计 |

### 附录

| 文档 | 内容 |
|------|------|
| [关键调用链](/wiki/appendix/Callgraphs.md) | JS/仓颉 → FFI → 原生调用链 |

## 新人阅读顺序

```
1. README.md          → 项目概述与快速开始
2. 00_Overview.md     → 理解架构与目录结构
3. 01_API_Reference.md → 了解可用 API
4. 02_BUILD.md        → 理解构建配置（可选）
```

---

## 安全研究员阅读路线

> 本路线专为安全审计、渗透测试研究人员设计，快速定位攻击面与风险点。

```
1. 04_Security.md     → 威胁模型与信任边界
2. 00_Overview.md     → 架构与外部依赖分析
3. 04_Security.md     → 风险点详细分析
4. 03_Architecture.md → FFI 边界与数据流
5. NOTES.md           → 代码证据速查
```

### 安全研究员重点关注

| 章节 | 内容 | 优先级 |
|------|------|--------|
| `04_Security.md:11-37` | 信任边界图 | P0 |
| `04_Security.md:39-47` | 数据流与敏感度 | P0 |
| `04_Security.md:64-95` | 电话号码未校验风险 | P0 |
| `04_Security.md:99-131` | 紧急号码滥用风险 | P1 |
| `04_Security.md:133-169` | 错误信息泄露风险 | P1 |
| `03_Architecture.md:46-63` | FFI 声明与边界 | P1 |
| `01_API_Reference.md:265-276` | API 清单表 | P1 |

### 快速风险定位

| 风险类型 | 位置 | 严重度 |
|---------|------|--------|
| 输入验证缺陷 | `call.cj:57` | 中 |
| 资源滥用 | `call.cj:162` | 低 |
| 信息泄露 | `number_format_options.cj:140` | 低 |
| 参数范围 | `number_format_options.cj:177` | 低 |

**证据来源**：`wiki/_work/NOTES.md` 完整代码证据记录

---

## 相关链接

- **主仓库**: https://gitcode.com/openharmony-sig/telephony_telephony_cangjie_wrapper
- **API 文档**: https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop
- **通话管理原仓**: https://gitcode.com/openharmony/telephony_call_manager
- **代码证据库**: [NOTES.md](/wiki/_work/NOTES.md)
- **项目评估**: [ASSESSMENT.md](/wiki/_work/ASSESSMENT.md)
