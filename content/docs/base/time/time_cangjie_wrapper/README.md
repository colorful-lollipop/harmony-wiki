# time_cangjie_wrapper Wiki

**项目**: time_cangjie_wrapper (时间时区仓颉封装)  
**版本**: 6.1  
**生成日期**: 2026-02-07  
**维护者**: OpenHarmony Wiki Agent

---

## 关于本Wiki

本Wiki是 **time_cangjie_wrapper** 项目的综合技术文档，面向两类受众：

1. **新人学习者** - 快速理解项目、上手使用API
2. **安全研究员** - 深入分析攻击面、评估安全风险

### 文档特点

- ✅ **证据驱动**: 每个技术结论都有代码位置支撑
- ✅ **双路线设计**: 新人路线 + 安全研究路线
- ✅ **简洁实用**: 聚焦核心内容，避免过度文档化
- ✅ **中文为主**: 专业术语保留英文

---

## 快速导航

### 如果你是新人学习者 👋

**推荐阅读顺序**（预计30分钟）：

1. **[01_Overview.md](./01_Overview.md)** - 5分钟
   - 了解项目定位和能做什么
   - 查看快速开始示例

2. **[02_Architecture.md](./02_Architecture.md)** - 10分钟
   - 理解FFI架构设计
   - 了解组件依赖关系

3. **[04_Interface.md](./04_Interface.md)** - 15分钟
   - 学习3个API的详细用法
   - 掌握错误处理

4. **[03_CodeMap.md](./03_CodeMap.md)** - 随时查阅
   - 需要看代码时使用
   - 快速定位功能实现

### 如果你是安全研究员 🔒

**推荐阅读顺序**（预计40分钟）：

1. **[02_Architecture.md](./02_Architecture.md)** - 10分钟
   - 了解架构和信任边界
   - 识别数据流

2. **[05_AttackSurface.md](./05_AttackSurface.md)** - 15分钟
   - 分析所有输入点
   - 识别信任边界跨越点

3. **[06_SecurityReview.md](./06_SecurityReview.md)** - 15分钟
   - 深度安全风险评估
   - 查看可利用性分析

4. **[03_CodeMap.md](./03_CodeMap.md)** - 随时查阅
   - 快速定位关键代码

---

## 文档目录

### 核心文档

| 文档 | 类型 | 说明 |
|------|------|------|
| [01_Overview.md](./01_Overview.md) | 入门 | 项目概览、快速开始、能力边界 |
| [02_Architecture.md](./02_Architecture.md) | 架构 | FFI架构、组件依赖、数据流图 |
| [03_CodeMap.md](./03_CodeMap.md) | 导航 | 目录结构、代码位置、快速导航 |
| [04_Interface.md](./04_Interface.md) | 参考 | API详细文档、使用示例、错误码 |
| [05_AttackSurface.md](./05_AttackSurface.md) | 安全 | 攻击面分析、输入点、信任边界 |
| [06_SecurityReview.md](./06_SecurityReview.md) | 安全 | 深度安全评估、风险分析、修复建议 |
| [07_Build.md](./07_Build.md) | 工程 | 构建配置、产物说明、集成指南 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/PLAN.md](./_work/PLAN.md) | Wiki生成任务计划 |
| [_work/NOTES.md](./_work/NOTES.md) | 代码证据汇总 |

---

## 项目速览

### 一句话定义

**time_cangjie_wrapper** 是 OpenHarmony 上为仓颉(Cangjie)语言应用提供系统时间和时区访问能力的官方SDK封装层。

### 核心能力

| 功能 | API | 说明 |
|------|-----|------|
| ✅ 获取系统时间 | `SystemDateTime.getTime()` | Unix时间戳 |
| ✅ 获取运行时间 | `SystemDateTime.getUptime()` | 从启动至今 |
| ✅ 获取系统时区 | `SystemDateTime.getTimezone()` | IANA时区标识符 |
| ❌ 设置系统时间 | N/A | 不支持 |
| ❌ 设置系统时区 | N/A | 不支持 |
| ❌ 定时器操作 | N/A | 不支持 |

### 技术特征

- **编程语言**: Cangjie (仓颉)
- **互操作机制**: FFI (Foreign Function Interface)
- **架构定位**: Framework层封装
- **API级别**: 22+
- **系统能力**: SystemCapability.MiscServices.Time
- **Beta阶段**: 功能稳定，API可能微调

---

## 安全评估摘要

### 风险评级: 🟢 低风险

| 评估维度 | 结果 |
|----------|------|
| **攻击面** | 极小（仅3个API） |
| **用户输入** | 无复杂输入（仅布尔/枚举） |
| **敏感操作** | 仅读取，无写入 |
| **内存安全** | 规范（复制后释放） |
| **依赖风险** | 依赖官方框架组件 |

### 关键安全结论

1. ✅ 本项目仅提供**读取操作**，无权限提升风险
2. ✅ 输入参数**类型简单**，无注入攻击面
3. ✅ 内存管理**规范**，无Use-After-Free风险
4. ⚠️ FFI边界需**关注底层** time_service 安全性

详细分析请参见 [05_AttackSurface.md](./05_AttackSurface.md) 和 [06_SecurityReview.md](./06_SecurityReview.md)。

---

## 代码证据说明

本Wiki所有技术结论都基于代码证据，引用格式如下：

```markdown
**证据**: `文件路径:行号`

```cangjie
// 关键代码片段
```
```

主要代码位置：
- **核心API**: `ohos/system_date_time/system_date_time.cj`
- **枚举定义**: `ohos/system_date_time/cj_date_time_common.cj`
- **错误处理**: `ohos/system_date_time/cj_date_time_error.cj`
- **构建配置**: `ohos/system_date_time/BUILD.gn`

---

## 适用范围

### 适用场景

- 学习仓颉语言时间API的使用
- 理解OpenHarmony FFI封装架构
- 评估time_cangjie_wrapper的安全风险
- 了解OpenHarmony框架层组件设计模式

### 不适用场景

- 学习底层 `time_service` C++实现
- 了解OpenHarmony内核时间子系统
- 仓颉语言基础语法学习

---

## 相关资源

### 官方文档

- [仓颉时间时区API参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-system_date_time.md)
- [仓颉时间时区开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/system_date_time/cj-system_data_time.md)

### 相关仓库

- [time_service](https://gitcode.com/openharmony/time_time_service) - 底层时间服务
- [cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) - 仓颉互操作框架
- [hiviewdfx_cangjie_wrapper](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper) - 日志封装

### 项目源码

- 组件路径: `base/time/time_cangjie_wrapper`
- 核心源码: `ohos/system_date_time/`

---

## 维护说明

### 更新策略

本Wiki基于代码静态分析生成，应在以下情况更新：

1. **代码变更**: 当核心源文件修改时
2. **版本发布**: 新版本发布时
3. **安全问题**: 发现新的安全风险时
4. **架构调整**: 项目架构发生变化时

### 更新流程

1. 重新执行代码扫描
2. 更新 `_work/NOTES.md` 证据汇总
3. 更新相关Wiki文档
4. 更新本文档的生成日期

### 局限性说明

1. **静态分析**: 基于代码静态分析，未包含运行时行为
2. **版本锁定**: 基于特定版本（v6.1）分析
3. **底层假设**: 假设底层 `time_service` 实现正确
4. **测试代码**: 已排除测试代码，不包含测试分析

---

## 质量保证

### 已完成检查

- [x] 每个技术结论都有代码证据支撑
- [x] 所有文件路径和行号已验证
- [x] 新人路线和安全路线完整
- [x] 术语使用统一
- [x] 内部链接有效
- [x] 无测试代码引用

### 证据来源

| 来源类型 | 文件 |
|----------|------|
| 核心源码 | `ohos/system_date_time/*.cj` |
| 构建配置 | `BUILD.gn`, `ohos/system_date_time/BUILD.gn` |
| 组件配置 | `bundle.json` |
| 项目说明 | `README.md`, `README_zh.md` |

---

## 反馈与贡献

如发现Wiki中的错误或需要补充内容，请：

1. 确认问题并收集代码证据
2. 更新相关Wiki文档
3. 更新 `_work/NOTES.md` 证据汇总
4. 在本文档更新日志中记录

---

## 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-02-07 | v1.0 | 初始版本，创建完整Wiki文档集 |

---

**生成时间**: 2026-02-07  
**文档版本**: v1.0  
**证据版本**: time_cangjie_wrapper v6.1
