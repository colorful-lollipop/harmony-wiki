# CalendarData 项目概览

## 目的

本文档介绍 OpenHarmony CalendarData 组件的项目定位、核心能力、运行环境和关键概念，帮助新人快速理解项目。

## 适用范围

- 目标读者：所有开发者
- 知识级别：入门到中级
- 前置知识：了解 OpenHarmony 基础架构

## 项目定位

CalendarData 是 OpenHarmony 系统中的预置应用，作为**数据提供者（Data Provider）**，为系统和其他应用提供日历数据的增删改查服务。

### 系统角色

| 角色 | 说明 |
|------|------|
| Data Provider | 通过 DataShare Extension Ability 提供跨应用数据共享 |
| N-API 模块 | 提供 C++ 到 JavaScript/ArkTS 的绑定接口 |
| CJ (FFI) 模块 | 提供ArkTS到C++的FFI绑定接口 |

### 与其他组件的关系

```
┌─────────────────┐
│  第三方应用     │
└────────┬────────┘
         │ N-API 调用
         ↓
┌─────────────────┐
│ calendarmanager  │ (libcalendarmanager.z.so)
│  N-API/CJ     │
└────────┬────────┘
         │ DataShare 调用
         ↓
┌─────────────────┐
│  entry         │ (DataShareExtAbility)
└────────┬────────┘
         │ 权限验证 + 数据库操作
         ↓
┌─────────────────┐
│  RDB 数据库     │
└─────────────────┘
```

## 核心能力

### 1. 日历账户管理

创建、查询、删除日历账户。

**关键类**:
- `CalendarAccount` - 日历账户信息
- `Calendar` - 日历对象

### 2. 日程事件管理

增删改查日程事件，包括重复规则（Recurrence Rule）和实例展开。

**关键类**:
- `Event` - 日程事件
- `EventFilter` - 事件筛选器
- `RecurrenceRule` - 重复规则
- `RecurrenceFrequency` - 重复频率（每日/每周/每月/每年）

### 3. 数据访问控制

两级权限模型，保护用户数据安全。

**权限级别**:
- **HIGH 权限**: `*_WHOLE_CALENDAR` - 可访问所有日历数据
- **LOW 权限**: `*_CALENDAR` - 仅可访问用户创建的日历数据

## 运行环境

| 配置项 | 值 |
|--------|-----|
| 适用系统 | OpenHarmony 标准系统 |
| API 级别 | API 10 (SDK 23) |
| 系统能力 | SystemCapability.Applications.CalendarData |
| 进程模型 | Extension Ability + Service Ability |
| 存储方式 | RDB (Relational Database) |

## 关键概念

### 日历账户 (Calendar Account)

日历账户代表一个日历数据源，可以是本地账户、Exchange 账户、Google 账户等。

**证据**: `datastructure/src/main/ets/calendars/Calendars.ets:63-102`

### 日程事件 (Event)

日程事件包含时间、地点、描述、重复规则等信息。

**证据**: `datastructure/src/main/ets/events/Events.ets:97-193`

### 事件实例 (Event Instance)

对于重复事件，系统会自动展开生成多个实例（例如每周一重复的事件会展开为具体日期的多个实例）。

**证据**: `datamanager/src/main/ets/processor/instances/InstancesProcessor.ets`

### 权限级别 (Permission Level)

两级权限模型用于控制数据访问范围：

- **HIGH**: 可读写任何日历账户的数据
- **LOW**: 仅可读写属于当前应用的数据（通过 URI 中的 tokenId 识别）

**证据**:
- `dataprovider/src/main/ets/DataShareAbilityAuthenticateProxy.ets:46-49`
- `calendarmanager/native/src/data_share_helper_manager.cpp:28-29, 73-86`

### Data Share Provider

OpenHarmony 的跨应用数据共享机制，CalendarData 作为 Provider 提供数据。

**证据**:
- `entry/src/main/ets/DataAbility/DataShareExtAbility.ets`

### N-API (Node-API)

Node.js/JavaScript 与 C++ 之间的绑定机制，CalendarData 通过 N-API 暴露 C++ API 给 JavaScript 层。

**证据**:
- `calendarmanager/napi/src/module_register.cpp:45-52`

### CJ (FFI) Binding

OpenHarmony 的 C-JavaScript Foreign Function Interface，提供 ArkTS 到 C++ 的绑定。

**证据**:
- `calendarmanager/cj/BUILD.gn:17-69`

## 技术栈

| 层次 | 技术 |
|------|------|
| 数据库 | OpenHarmony RDB |
| IPC 通信 | DataShare 框架 |
| 语言绑定 | N-API, CJ (FFI) |
| 开发语言 | ArkTS, C++ |
| 构建系统 | GN |
| 包管理 | oh-package.json5 |

## 边界与限制

### 功能边界

**提供**:
- ✅ 日历账户 CRUD
- ✅ 日程事件 CRUD
- ✅ 重复规则支持
- ✅ 事件实例查询
- ✅ 提醒管理
- ✅ 数据访问控制

**不提供**:
- ❌ 日历 UI（由 Calendar 应用提供）
- ❌ 网络同步（由 sync 模块提供）
- ❌ 告警提醒调度（由系统闹钟服务提供）

### 技术限制

| 限制项 | 说明 |
|--------|------|
| 系统类型 | 仅支持标准系统，不支持轻量系统 |
| API 级别 | 最低 API 10 |
| 数据库 | 仅使用 RDB，不支持其他存储方式 |

## 相关文档

- [目录结构](01_Directory_Structure.md) - 代码组织方式
- [架构说明](02_Architecture.md) - 详细架构图和数据流
- [对外 API](03_External_API.md) - N-API 完整清单
- [安全评审](07_Security_Review.md) - 权限机制和安全风险

---

返回 [目录](SUMMARY.md) | [首页](README.md)
