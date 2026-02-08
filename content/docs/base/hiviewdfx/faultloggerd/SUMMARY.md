# FaultLoggerd Wiki 导航

本导航帮助新人按顺序阅读文档，快速理解 faultloggerd 组件。

## 推荐阅读顺序

### 新手入门
1. **[项目定位与核心能力](00_Overview.md)** - 了解项目是什么、解决什么问题
2. **[目录结构与模块职责](01_Directory_Structure.md)** - 熟悉代码组织
3. **[架构设计](02_Architecture.md)** - 理解组件如何协同工作

### 开发者深入
4. **[对外 API](03_External_API.md)** - 学习如何调用 faultloggerd 能力
5. **[内部 API](04_Inner_API.md)** - 了解内部模块接口（二次开发需要）
6. **[GN 构建系统](05_GN_Targets.md)** - 理解编译流程和目标

### 系统集成与安全
7. **[编译产物](06_Build_Artifacts.md)** - 了解最终产物和安装位置

### 安全研究路线
8. **[攻击面分析](05_AttackSurface.md)** - 外部输入入口、敏感操作、信任边界
9. **[安全风险评审](07_Security_Review.md)** - 风险分析、可被利用点、安全建议

### 进阶参考
9. **[附录 - 关键调用链](appendix/Appendix.md)** - 完整调用链和时序
10. **[附录 - 配置标志](appendix/Appendix.md)** - 编译选项和运行时配置详解
11. **[附录 - 错误码](appendix/Appendix.md)** - 完整错误码列表
12. **[附录 - 常见问题](appendix/Appendix.md)** - 问题定位和解决方法

## 快速索引

### 按主题查找

| 主题 | 相关文档 |
|------|----------|
| 架构总览 | [架构设计](02_Architecture.md) |
| 崩溃日志生成 | [项目定位与核心能力](00_Overview.md#进程崩溃日志生成) |
| 主动抓栈 API | [对外 API](03_External_API.md#dumpcatcher-api) |
| 本地回栈能力 | [对外 API](03_External_API.md#backtrace-api) |
| 信号处理 | [对外 API](03_External_API.md#signalhandler-api) |
| 客户端通信 | [架构设计](02_Architecture.md#socket-通信机制) |
| 权限控制 | [安全风险评审](07_Security_Review.md#权限控制机制) |
| 编译配置 | [GN 构建系统](05_GN_Targets.md#关键编译选项) |
| 安全风险 | [攻击面分析](05_AttackSurface.md), [安全风险评审](07_Security_Review.md#可被利用点总结) |
| 错误码 | [附录 - 错误码](appendix/Appendix.md#附录-c错误码完整列表) |
| 常见问题 | [附录 - 常见问题](appendix/Appendix.md) |

### 按角色查找

#### 应用开发者
- [对外 API](03_External_API.md) - 如何在应用中使用 DumpCatcher、Backtrace
- [项目定位与核心能力](00_Overview.md#核心能力) - 故障日志位置和格式

#### 系统开发者
- [目录结构与模块职责](01_Directory_Structure.md) - 了解模块划分
- [内部 API](04_Inner_API.md) - 模块间接口
- [GN 构建系统](05_GN_Targets.md) - 构建配置
- [编译产物](06_Build_Artifacts.md) - 产物和安装路径

#### 安全审计员
- [攻击面分析](05_AttackSurface.md) - 外部输入入口、信任边界
- [安全风险评审](07_Security_Review.md) - 详细风险分析和可被利用点
- [架构设计](02_Architecture.md#权限控制与隔离) - 安全边界
- [对外 API](03_External_API.md) - API 权限要求

#### 故障诊断工程师
- [项目定位与核心能力](00_Overview.md) - 故障日志位置和格式
- [对外 API](03_External_API.md) - 抓栈工具使用
- [架构设计](02_Architecture.md#处理流程) - 完整处理流程
- [附录 - 常见问题](appendix/Appendix.md) - 问题定位和解决方法

## 文档说明

### 符号标记
- 🔍 TODO: 待确认内容
- 📋 Note: 重要说明
- ⚠️ Warning: 警告信息
- ✅ Verified: 已代码验证
- ⚠️ Medium Risk: 中风险
- 🔴 High Risk: 高风险

### 代码引用格式
本文档中的代码引用遵循以下格式：

```
文件路径:行号
```

例如：`services/fault_logger_service.cpp:70-86` 表示位于 fault_logger_service.cpp 第 70-86 行的代码。

### 测试代码排除说明
本文档不引用 `test/` 目录下的任何代码作为业务逻辑证据。

### 证据等级

| 证据类型 | 说明 |
|----------|------|
| **代码引用** | 直接在源代码中可验证 |
| **架构图** | 基于代码结构推导的组件关系 |
| **时序图** | 基于代码执行流程推导 |
| **配置文档** | 来自实际配置文件 |

### N-API 说明

**重要声明**：faultloggerd **不提供** JavaScript/N-API 接口。所有 API 均为 Native C/C++ 接口。

**接口类型**：
- DumpCatcher API - C/C++ 原生 API
- Backtrace API - C/C++ 原生 API  
- SignalHandler API - C 原生 API
- FaultloggerdClient API - Socket 客户端 API
- Rust PanicHandler - Rust 原生 API

## 文档统计

- **Wiki 文档总数**：14 篇（主文档 + 附录）
- **API 文档数量**：5 篇
- **架构文档数量**：1 篇
- **构建文档数量**：2 篇
- **安全文档数量**：2 篇（攻击面分析 + 安全风险评审）
- **附录文档数量**：1 篇（包含 4 个部分）
- **代码证据条目**：60+ 处（路径:行号引用）
- **图表数量**：5 个（架构图 + 信任边界图 + 3 个时序图）

## 更新记录

- **生成时间**：2026-02-06
- **基于代码版本**：main 分支
- **覆盖范围**：完整生产代码（不含测试）
- **文档语言**：中文
- **文档结构**：
  - 1 个概览页
  - 1 个目录结构页
  - 1 个架构设计页
  - 1 个对外 API 页
  - 1 个内部 API 页
  - 1 个 GN 构建系统页
  - 1 个编译产物页
  - 1 个攻击面分析页
  - 1 个安全风险评审页
  - 1 个附录页（包含 4 个部分）

## 使用指南

### 快速入门（10 分钟阅读）

1. 阅读 [项目定位与核心能力](00_Overview.md) 了解项目定位
2. 浏览 [目录结构与模块职责](01_Directory_Structure.md) 熟悉代码组织
3. 查阅 [对外 API](03_External_API.md) 了解可用的 API
4. 参考 [安全风险评审](07_Security_Review.md) 理解权限要求

### 深入开发（30 分钟阅读）

1. 完整阅读 [架构设计](02_Architecture.md) 理解组件交互
2. 精读 [对外 API](03_External_API.md) 的 API 详细文档
3. 参考 [内部 API](04_Inner_API.md) 进行模块间交互
4. 研究 [GN 构建系统](05_GN_Targets.md) 了解编译选项
5. 使用 [编译产物](06_Build_Artifacts.md) 了解运行时依赖

### 故障诊断（按需阅读）

1. 参考 [附录 - 常见问题](appendix/Appendix.md) 的崩溃日志分析
2. 使用 [附录 - 关键调用链](appendix/Appendix.md) 理解完整处理流程
3. 参考 [附录 - 错误码](appendix/Appendix.md) 理解错误码含义

## 技术支持

如需技术支持或有疑问，请参考：

- OpenHarmony 文档门户：https://docs.openharmony.cn/
- OpenHarmony Gitee 文档：https://github.com/openharmony/docs
- FaultLoggerd 源码：https://github.com/openharmony/hiviewdfx_faultloggerd
