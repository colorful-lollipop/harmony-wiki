# powermgr_cangjie_wrapper 工程文档

> **生成时间**: 2025-02-06
> **覆盖范围**: 电池信息 Cangjie API 封装层
> **版本**: 6.1

## 文档说明

本文档集为 `powermgr_cangjie_wrapper` 组件的工程 Wiki，面向新人快速理解项目架构、API 设计、构建系统和安全风险。

### 更新方式

文档应随代码变更同步更新：
1. 新增/修改 API → 更新 `03_Public_API.md` 和相关章节
2. 架构调整 → 更新 `02_Architecture.md` 和 `04_Internal_API.md`
3. 构建配置变更 → 更新 `05_GN_Targets.md` 和 `06_Build_Artifacts.md`
4. 发现安全问题 → 更新 `07_Security_Review.md`

### 覆盖与未覆盖范围

**已覆盖**:
- ✅ 对外 Cangjie API 完整清单
- ✅ 内部架构和模块职责
- ✅ GN 构建目标和依赖
- ✅ 安全风险分析
- ✅ 调用链和数据流

**未覆盖**:
- ❌ 外部依赖 `battery_manager:cj_battery_info_ffi` 的实现细节
- ❌ 测试相关内容（按要求排除）
- ❌ 性能数据和基准测试
- ❌ 开发环境搭建指南（见 OpenHarmony 官方文档）

### 证据说明

本文档所有关键结论都包含代码证据：
- 文件路径（必要时含行号，格式: `path:line`）
- 关键符号名（函数/类/枚举）
- 最小必要代码片段或调用链描述

无法确认的内容标注为 `TODO(需确认)`。

---

## 快速导航

- [项目概览](SUMMARY.md) - 从这里开始阅读
- [API 清单](03_Public_API.md) - 所有公开接口的详细说明
- [安全评审](07_Security_Review.md) - 已知风险和修复建议

---

## 项目定位

**powermgr_cangjie_wrapper** 是 OpenHarmony 上基于 Cangjie（仓颉）编程语言的电池信息 API 封装层。

**核心能力**:
- 电池电量查询（SoC）
- 充电状态查询
- 电池健康状态
- 充电器类型识别
- 电压、温度、电流等硬件参数
- 电池容量等级

**技术特点**:
- 使用 Cangjie FFI 调用底层 C 接口
- 静态属性 API 设计（无实例化）
- 同步调用模式（无异步/Promise）
- Beta 特性，功能有限制

**目标系统**: 标准设备 (standard)

---

## 文档结构

详见 [SUMMARY.md](SUMMARY.md)
