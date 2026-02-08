# SUMMARY - Wiki 导航

> 目的：提供全站导航 + 新人阅读顺序
> 适用范围：所有 Wiki 读者
> 最后更新：2026-02-06

## 新人推荐阅读路线

### 快速入门路线（1-2 小时）

```
[00_Overview.md] (15 分钟)
    ↓ 了解项目定位与核心能力
[01_Directories_and_Modules.md] (20 分钟)
    ↓ 熟悉代码组织结构
[02_Architecture.md] (30 分钟)
    ↓ 理解架构与数据流
[04_Public_API.md] (45 分钟)
    ↓ 学习对外的 API 使用方法
```

**完成目标**：能够使用 Cangjie 文件管理 API 开发应用

---

### 深入理解路线（4-8 小时）

```
[00_Overview.md]
    ↓
[01_Directories_and_Modules.md]
    ↓
[02_Architecture.md]
    ↓
[03_NAPI_FFI_Bindings.md] (如需集成)
    ↓ 理解 FFI 接口与外部依赖
[04_Public_API.md]
    ↓ 掌握对外的 API
[05_Internal_API.md]
    ↓ 理解内部接口设计
[06_GN_Targets_and_Build.md]
    ↓ 掌握构建系统
[07_Security_Review.md]
    ↓ 了解安全风险与最佳实践
```

**完成目标**：能够深度理解系统架构、参与代码贡献或集成开发

---

## 全部 Wiki 索引

### 概览与入门

| 文档 | 章节 | 阅读时间 | 适用人群 |
|------|------|----------|----------|
| **[00_Overview.md](00_Overview.md)** | 项目定位、边界、核心能力、运行环境 | 15 分钟 | 所有读者 |
| **[01_Directories_and_Modules.md](01_Directories_and_Modules.md)** | 目录结构、模块职责、依赖关系 | 20 分钟 | 新人、架构师 |
| **[02_Architecture.md](02_Architecture.md)** | 架构图、数据流、线程模型、时序图 | 30 分钟 | 架构师、开发者 |

### 技术深度文档

| 文档 | 章节 | 阅读时间 | 适用人群 |
|------|------|----------|----------|
| **[03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md)** | FFI 函数清单（70+）、参数校验、错误处理 | 45 分钟 | 集成开发者、底层开发 |
| **[04_Public_API.md](04_Public_API.md)** | Cangjie API 列表、参数/返回值、使用示例 | 60 分钟 | 应用开发者 |
| **[05_Internal_API.md](05_Internal_API.md)** | 内部接口、依赖方向、稳定性标注 | 30 分钟 | 框架开发者、贡献者 |
| **[06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md)** | GN 目标、编译产物、依赖关系 | 25 分钟 | 构建工程师 |

### 安全与最佳实践

| 文档 | 章节 | 阅读时间 | 适用人群 |
|------|------|----------|----------|
| **[07_Security_Review.md](07_Security_Review.md)** | 攻击面、信任边界、可被利用点、修复建议 | 40 分钟 | 安全审计、架构师 |

---

## 按角色查找文档

### 仓颉开发者

**入门路径**：00 → 01 → 04

| 主题 | 推荐章节 |
|------|----------|
| **API 调用** | [04_Public_API.md](04_Public_API.md) |
| **错误处理** | [00_Overview.md](00_Overview.md) → [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) |
| **异步 I/O** | [02_Architecture.md](02_Architecture.md) |
| **URI 操作** | [00_Overview.md](00_Overview.md) → [04_Public_API.md](04_Public_API.md) |

### 系统架构师

**入门路径**：00 → 02 → 05 → 03

| 主题 | 推荐章节 |
|------|----------|
| **分层架构** | [00_Overview.md](00_Overview.md) → [02_Architecture.md](02_Architecture.md) |
| **模块依赖** | [01_Directories_and_Modules.md](01_Directories_and_Modules.md) |
| **接口设计** | [05_Internal_API.md](05_Internal_API.md) |
| **FFI 集成** | [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) |

### 安全审计人员

**入门路径**：00 → 02 → 07

| 主题 | 推荐章节 |
|------|----------|
| **系统边界** | [00_Overview.md](00_Overview.md) → [02_Architecture.md](02_Architecture.md) |
| **攻击面分析** | [07_Security_Review.md](07_Security_Review.md) |
| **权限机制** | [00_Overview.md](00_Overview.md) |
| **数据流安全** | [02_Architecture.md](02_Architecture.md) → [07_Security_Review.md](07_Security_Review.md) |

### 构建工程师

**入门路径**：00 → 01 → 06

| 主题 | 推荐章节 |
|------|----------|
| **构建目标** | [06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md) |
| **依赖管理** | [01_Directories_and_Modules.md](01_Directories_and_Modules.md) |
| **产物定位** | [06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md) |

---

## 文档地图

### 架构视图

```
┌─────────────────────────────────────────────────────────────┐
│                   00_Overview.md                        │
│               (项目定位与核心能力）                      │
└─────────────────────────────────────────────────────────────┘
                              │
                ┌───────────┴───────────┐
                │                    │
        ┌───────▼───────┐  ┌───▼────────┐
        │                │  │            │
┌───────▼───────┐  ┌───▼────────┐  │  ┌─────▼───────┐
│ 01_Directories   │  │ 02_Architecture  │  │  │ 04_Public    │
│  _and_Modules    │  │                │  │  │  _API          │
└─────────────────┘  └─────────────────┘  │  └──────────────┘
                                       │
                            ┌───────────▼───────────┐
                            │                      │
                ┌─────────────▼─────────┐  ┌─────▼────────┐
                │                       │  │              │
        ┌───────▼───────┐  ┌─────▼───────┐  │  ┌─────▼───────┐
        │               │  │              │  │  │               │
│  03_NAPI_FFI   │  │ 05_Internal  │  │  │ 06_GN        │
│  _Bindings      │  │  _API        │  │  │  Targets       │
└─────────────────┘  └──────────────┘  │  └──────────────┘
                                       │
                            ┌───────────▼───────────┐
                            │                      │
                        ┌─────────────▼─────────┐
                        │                       │
                ┌───────────▼─────────┐
                │                       │
        │  07_Security_Review      │
        │                       │
        └───────────────────────┘
```

---

## 快速查找

### 按功能主题

| 功能 | 相关章节 |
|------|----------|
| **文件操作** | [00_Overview.md](00_Overview.md) → [04_Public_API.md](04_Public_API.md) |
| **流 I/O** | [00_Overview.md](00_Overview.md) → [02_Architecture.md](02_Architecture.md) |
| **URI 处理** | [00_Overview.md](00_Overview.md) → [01_Directories_and_Modules.md](01_Directories_and_Modules.md) |
| **目录列表** | [04_Public_API.md](04_Public_API.md) |
| **错误处理** | [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) |

### 按技术主题

| 技术 | 相关章节 |
|------|----------|
| **FFI 机制** | [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) |
| **WorkerThread** | [02_Architecture.md](02_Architecture.md) → [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) |
| **RemoteDataLite** | [02_Architecture.md](02_Architecture.md) |
| **错误码** | [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) |
| **GN 构建** | [06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md) |

---

## 更新日志

| 版本 | 日期 | 变更章节 | 变更内容 |
|------|------|----------|----------|
| 1.0 | 2026-02-06 | 全部 | 初始版本，创建所有章节 |

---

## 常见问题

### Q: 如何快速上手？

**A**: 按照"快速入门路线"阅读：[00_Overview.md](00_Overview.md) → [01_Directories_and_Modules.md](01_Directories_and_Modules.md) → [04_Public_API.md](04_Public_API.md)，预计 1-2 小时。

### Q: 如何理解 FFI 调用？

**A**: 阅读 [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) 了解所有 70+ FFI 函数的定义和参数映射。

### Q: 构建产物在哪里？

**A**: 阅读 [06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md) 了解 GN 目标和预计输出位置。

### Q: 如何参与代码贡献？

**A**:
1. 阅读源码理解模块职责（[01_Directories_and_Modules.md](01_Directories_and_Modules.md)）
2. 理解内部接口设计（[05_Internal_API.md](05_Internal_API.md)）
3. 参考 [README.md](../README.md) 中的"代码贡献"章节

### Q: 安全风险在哪里？

**A**: 阅读 [07_Security_Review.md](07_Security_Review.md) 了解攻击面、信任边界和修复建议。

---

## 反馈与贡献

如发现问题或有改进建议：

1. 在对应章节文件中直接编辑
2. 更新 `wiki/_work/PLAN.md` 记录变更
3. 确保所有修改有代码证据支撑

---

> **提示**：本导航文档基于当前 Wiki 结构自动生成，如有章节缺失，请先创建对应文件。
