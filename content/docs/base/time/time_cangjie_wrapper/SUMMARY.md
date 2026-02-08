# Wiki 导航

**项目**: time_cangjie_wrapper  
**用途**: 双路线导航（新人学习 / 安全研究）

---

## 🚀 新人学习路线

> 目标：快速理解项目、掌握API使用

### 第一阶段：建立全局认知（15分钟）

1. **[项目概览](01_Overview.md)** ⭐ 必读
   - 项目定位和能力边界
   - 快速开始示例
   - 与ArkTS API对比

2. **[架构分析](02_Architecture.md)** ⭐ 必读
   - FFI架构设计原理
   - 组件依赖关系
   - 数据流和调用链

### 第二阶段：掌握API使用（15分钟）

3. **[接口文档](04_Interface.md)** ⭐ 必读
   - 3个API详细说明
   - 使用示例和最佳实践
   - 错误处理指南

### 第三阶段：深入了解（可选）

4. **[代码地图](03_CodeMap.md)** 📍 查阅
   - 快速定位代码位置
   - 功能到代码映射
   - 文件依赖关系

5. **[构建与产物](07_Build.md)** 📍 查阅
   - GN构建配置
   - 编译产物说明
   - 集成指南

---

## 🔒 安全研究路线

> 目标：识别攻击面、评估安全风险

### 第一阶段：理解架构（10分钟）

1. **[架构分析](02_Architecture.md)** ⭐ 必读
   - 信任边界分析
   - FFI机制详解
   - 数据流向图

### 第二阶段：攻击面分析（15分钟）

2. **[攻击面分析](05_AttackSurface.md)** ⭐ 必读
   - 外部输入清单
   - 敏感操作识别
   - 信任边界跨越点
   - 攻击向量分析

### 第三阶段：深度风险评估（15分钟）

3. **[安全风险评估](06_SecurityReview.md)** ⭐ 必读
   - 5类安全风险分析
   - 可利用性评估
   - 修复建议
   - 审计检查清单

### 第四阶段：代码审计（可选）

4. **[代码地图](03_CodeMap.md)** 📍 查阅
   - 快速定位关键代码
   - 安全相关代码位置
   - FFI调用点

---

## 📚 文档索引

### 按类型索引

| 类型 | 文档 |
|------|------|
| **入门** | [01_Overview.md](01_Overview.md) |
| **架构** | [02_Architecture.md](02_Architecture.md) |
| **导航** | [03_CodeMap.md](03_CodeMap.md) |
| **参考** | [04_Interface.md](04_Interface.md) |
| **安全** | [05_AttackSurface.md](05_AttackSurface.md), [06_SecurityReview.md](06_SecurityReview.md) |
| **工程** | [07_Build.md](07_Build.md) |

### 按主题索引

| 主题 | 相关文档 |
|------|----------|
| **API使用** | [01_Overview.md](01_Overview.md) → [04_Interface.md](04_Interface.md) |
| **架构理解** | [02_Architecture.md](02_Architecture.md) |
| **代码定位** | [03_CodeMap.md](03_CodeMap.md) |
| **安全审计** | [05_AttackSurface.md](05_AttackSurface.md) → [06_SecurityReview.md](06_SecurityReview.md) |
| **构建集成** | [07_Build.md](07_Build.md) |

---

## 🎯 快速入口

### 常见问题

| 问题 | 答案位置 |
|------|----------|
| 这个项目是做什么的？ | [01_Overview.md#一句话定义](01_Overview.md#一句话定义) |
| 如何快速使用API？ | [01_Overview.md#快速开始](01_Overview.md#快速开始) |
| 支持哪些功能？ | [01_Overview.md#能力边界](01_Overview.md#能力边界) |
| 与ArkTS API有什么区别？ | [01_Overview.md#与arkts-api的能力对比](01_Overview.md#与arkts-api的能力对比) |
| FFI是如何工作的？ | [02_Architecture.md#ffi机制详解](02_Architecture.md#ffi机制详解) |
| 安全风险高吗？ | [06_SecurityReview.md#执行摘要](06_SecurityReview.md#执行摘要) |
| 如何构建？ | [07_Build.md#构建流程](07_Build.md#构建流程) |

### 关键代码位置

| 功能 | 文件 | 行号 |
|------|------|------|
| `getTime()` | `ohos/system_date_time/system_date_time.cj` | 61 |
| `getUptime()` | `ohos/system_date_time/system_date_time.cj` | 78 |
| `getTimezone()` | `ohos/system_date_time/system_date_time.cj` | 95 |
| `TimeType` 枚举 | `ohos/system_date_time/cj_date_time_common.cj` | 30 |
| 错误处理 | `ohos/system_date_time/cj_date_time_error.cj` | 34 |
| FFI声明 | `ohos/system_date_time/system_date_time.cj` | 23 |

---

## 📊 阅读时间估算

### 新人学习路线

| 阶段 | 文档 | 预计时间 |
|------|------|----------|
| 1 | 01_Overview.md | 5分钟 |
| 1 | 02_Architecture.md | 10分钟 |
| 2 | 04_Interface.md | 15分钟 |
| **总计** | | **30分钟** |

### 安全研究路线

| 阶段 | 文档 | 预计时间 |
|------|------|----------|
| 1 | 02_Architecture.md | 10分钟 |
| 2 | 05_AttackSurface.md | 15分钟 |
| 3 | 06_SecurityReview.md | 15分钟 |
| **总计** | | **40分钟** |

---

## 🔗 外部链接

### 官方文档

- [仓颉时间时区API参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-system_date_time.md)
- [仓颉时间时区开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/system_date_time/cj-system_data_time.md)

### 相关仓库

- [time_service](https://gitcode.com/openharmony/time_time_service)
- [cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- [hiviewdfx_cangjie_wrapper](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper)

---

## 📝 工作文档

Wiki生成过程中的工作文档：

- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估报告
- [_work/PLAN.md](_work/PLAN.md) - 任务计划
- [_work/NOTES.md](_work/NOTES.md) - 代码证据汇总

---

## 🏠 关于本Wiki

- **[README.md](README.md)** - 项目说明、维护信息、质量保证

---

**生成时间**: 2026-02-07  
**版本**: v1.0
