# 项目概览

## 项目定位

**System Ability Manager (Samgr)** 是 OpenHarmony 的核心系统组件，负责管理系统能力（System Ability，也称系统服务）的全生命周期，包括：

- **注册管理**：系统能力的注册与注销
- **服务发现**：本地和跨设备的服务查询
- **动态加载**：按需启动系统能力（On-Demand Loading）
- **状态管理**：系统能力的生命周期状态机
- **进程管理**：系统能力进程的启动、停止和监控

## 核心能力

| 能力 | 说明 | 关键接口 |
|------|------|----------|
| **服务注册** | SA 向 Samgr 注册，保存到 abilityMap | `AddSystemAbility()` |
| **服务查询** | 根据 SA ID 获取远程对象 | `CheckSystemAbility()`, `GetSystemAbility()` |
| **动态加载** | 按需启动 SA 进程 | `LoadSystemAbility()` |
| **订阅通知** | SA 状态变化订阅/取消订阅 | `SubscribeSystemAbility()` |
| **跨设备访问** | 分布式场景下的 SA 发现 | `GetSystemAbility(deviceId)` |
| **进程管理** | SA 进程的启动、停止、查询 | `AddSystemProcess()`, `UnloadSystemAbility()` |

## 运行环境

| 属性 | 说明 |
|------|------|
| **系统类型** | Standard 系统（非 Lite） |
| **进程名** | `samgr` |
| **运行用户** | `system` |
| **启动方式** | 系统启动时由 init 拉起 |
| **关键依赖** | IPC、FFRT、Hilog、Access Token、Selinux |

## 关键概念

### System Ability (SA)
系统能力/系统服务，是 OpenHarmony 中提供系统级功能的组件，每个 SA 有唯一的 SA ID。

### SA ID
系统能力标识符，32位整数，范围定义：
- **范围**: `0x00000001` - `0x00ffffff`
- **厂商区间**: `0x00010000` - `0x00020000`
- **定义位置**: `interfaces/innerkits/samgr_proxy/include/system_ability_definition.h`

### On-Demand Loading
按需加载机制，SA 进程不需要在系统启动时就运行，而是在第一次被访问时动态启动。

## 架构位置

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Applications)                    │
├─────────────────────────────────────────────────────────────┤
│                   应用框架层 (Application Framework)          │
├─────────────────────────────────────────────────────────────┤
│                    系统服务层 (System Services)               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ Bundle Manager│  │  AbilityMgr  │  │   PowerMgr   │       │
│  │   (SA:401)   │  │   (SA:180)   │  │   (SA:3301)  │       │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                 │                 │               │
│  ┌──────▼─────────────────▼─────────────────▼───────┐       │
│  │         System Ability Manager (Samgr)           │       │
│  │            - 服务注册与发现                       │       │
│  │            - 动态加载管理                         │       │
│  │            - 进程生命周期                         │       │
│  └──────────────────────────────────────────────────┘       │
├─────────────────────────────────────────────────────────────┤
│                    IPC / DBinder 层                          │
├─────────────────────────────────────────────────────────────┤
│                    内核层 (Kernel)                           │
└─────────────────────────────────────────────────────────────┘
```

## 代码统计

| 类型 | 数量 |
|------|------|
| 生产环境 C++ 源文件 | 41 个 .cpp |
| 生产环境头文件 | 57 个 .h |
| 接口定义文件 | 17 个 .h |
| BUILD.gn 构建文件 | 8 个 |
| Rust 绑定文件 | 6 个 .rs |

## 相关仓库

- [systemabilitymgr_safwk](https://gitee.com/openharmony/systemabilitymgr_safwk) - SA 框架
- **systemabilitymgr_samgr** (本仓库) - SA 管理器
- [systemabilitymgr_safwk_lite](https://gitee.com/openharmony/systemabilitymgr_safwk_lite) - 轻量级版本
- [systemabilitymgr_samgr_lite](https://gitee.com/openharmony/systemabilitymgr_samgr_lite) - 轻量级版本
