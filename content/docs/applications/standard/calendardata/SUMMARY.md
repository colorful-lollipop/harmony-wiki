# CalendarData Wiki 目录

本文档为 OpenHarmony CalendarData 组件的完整技术文档。

## 新人快速入门

如果你是第一次接触 CalendarData 组件，建议按以下顺序阅读：

1. 📖 [项目概览](00_Overview.md) - 项目定位、边界、核心能力
2. 📁 [目录结构](01_Directory_Structure.md) - 代码组织方式和模块职责
3. 🏗️ [架构说明](02_Architecture.md) - 组件图、数据流、时序
4. 🔌 [对外 API](03_External_API.md) - N-API 清单和使用方法
5. ⚙️ [内部 API](04_Internal_API.md) - 模块接口和依赖关系

## 深入理解

### 构建与部署

5. 🛠️ [GN 构建](05_GN_Build.md) - 构建系统详解
6. 📦 [编译产物](06_Build_Artifacts.md) - 产物清单和运行时加载
7. ❓ [FAQ](08_FAQ.md) - 常见构建/运行/调试问题

### 安全与质量

7. 🔒 [安全评审](07_Security_Review.md) - 攻击面、信任边界、风险分析

### 附录

- [调用链图](appendix/Callgraphs.md) - 关键 API 调用链
- [配置标志](appendix/Config_Flags.md) - 关键宏和 feature flags

## 文档总览

| 文档 | 说明 | 目标读者 |
|------|------|---------|
| [项目概览](00_Overview.md) | 项目定位、运行环境、关键概念 | 所有开发者 |
| [目录结构](01_Directory_Structure.md) | 目录组织、模块职责、代码入口 | 所有开发者 |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型、时序 | 架构师/高级开发 |
| [对外 API](03_External_API.md) | N-API 清单、参数、错误码、权限 | 应用开发 |
| [内部 API](04_Internal_API.md) | 模块接口、依赖方向、稳定性 | 组件开发 |
| [GN 构建](05_GN_Build.md) | targets、依赖、开关、产物 | 构建工程师 |
| [编译产物](06_Build_Artifacts.md) | .so/.a/.hap、安装路径、加载关系 | 运维/部署 |
| [安全评审](07_Security_Review.md) | 威胁模型、攻击面、可被利用点 | 安全审计 |
| [FAQ](08_FAQ.md) | 构建问题、运行问题、调试技巧 | 所有问题排查者 |

## 关键概念速查

### 权限模型

| 权限 | 级别 | 说明 |
|------|------|------|
| `ohos.permission.READ_WHOLE_CALENDAR` | HIGH | 读取所有日历数据 |
| `ohos.permission.WRITE_WHOLE_CALENDAR` | HIGH | 写入所有日历数据 |
| `ohos.permission.READ_CALENDAR` | LOW | 读取日历数据（受限） |
| `ohos.permission.WRITE_CALENDAR` | LOW | 写入日历数据（受限） |

### 核心模块

| 模块 | 职责 | 语言 |
|------|------|------|
| calendarmanager | N-API/CJ 绑定、核心逻辑 | C++ |
| datamanager | 数据库操作、数据转换 | ArkTS |
| dataprovider | DataShare Ability、权限验证 | ArkTS |
| datastructure | 表结构定义 | ArkTS |
| entry | Extension Ability 入口 | ArkTS |
| common | 工具类 | ArkTS |
| rrule | 重复规则处理 | ArkTS |

### 数据流

```
外部应用/JS
    ↓ N-API/CJ
calendarmanager
    ↓ DataShare Helper
entry/DataShareExtAbility
    ↓ 权限验证
dataprovider/AuthenticateProxy
    ↓ 权限级别
dataprovider/Delegate
    ↓ 工厂模式
datamanager/ProcessorFactory
    ↓ 表处理器
datamanager/[Table]Processor
    ↓
RDB 数据库
```

## 快速链接

### 开发相关

- [对外 API - CalendarManager](03_External_API.md#calendarmanager)
- [对外 API - Calendar](03_External_API.md#calendar)
- [对外 API - EventFilter](03_External_API.md#eventfilter)
- [对外 API - 枚举类型](03_External_API.md#枚举类型)

### 构建相关

- [GN Targets](05_GN_Build.md#targets-清单)
- [构建依赖](05_GN_Build.md#依赖关系)
- [编译产物](06_Build_Artifacts.md#产物清单)

### 安全相关

- [权限检查机制](07_Security_Review.md#权限检查)
- [可被利用点](07_Security_Review.md#可被利用点)
- [修复建议](07_Security_Review.md#修复建议)

## 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|---------|
| 2026-02-05 | 1.0.0 | 初始版本 |

---

返回 [首页](README.md)
