# SUMMARY - 全站导航

本文档提供 OpenHarmony ArkUI 高级 UI 组件库的完整导航。

## 新人阅读路线

```
推荐阅读顺序:
  1. README.md          → 项目概览
  2. 01_Overview.md    → 项目定位与核心能力
  3. 02_Architecture.md → 架构设计
  4. 03_Components.md  → 组件清单与 API
  5. [选择] 04_Build.md → 构建配置 (开发者)
  6. [选择] 05_Usage.md → 使用说明 (使用者)
  7. 06_Security.md    → 安全评估 (安全审查者)
  8. 07_Troubleshooting.md → 问题定位 (调试时)
```

---

## 文档索引

### 入门文档

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [README](README.md) | 项目整体说明 | 所有读者 |
| [01_Overview](01_Overview.md) | 项目定位与核心能力 | 所有读者 |
| [05_Usage](05_Usage.md) | 组件使用示例 | 应用开发者 |

### 架构文档

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [02_Architecture](02_Architecture.md) | 整体架构设计 | 架构师、开发者 |
| [03_Components](03_Components.md) | 组件详细说明 | 开发者 |

### 开发文档

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [04_Build](04_Build.md) | GN 构建配置 | 构建工程师 |
| [07_Troubleshooting](07_Troubleshooting.md) | 常见问题 | 开发者 |

### 安全文档

| 文档 | 说明 | 目标读者 |
|------|------|----------|
| [06_Security](06_Security.md) | 安全风险评审 | 安全工程师 |

---

## 组件快速跳转

### 原子化服务组件

| 组件 | N-API 模块 | 主要功能 |
|------|------------|----------|
| [AtomServiceNavigation](03_Components.md#atomservicenavigation) | `atomicservice.AtomicServiceNavigation` | 导航组件 |
| [AtomServiceSearch](03_Components.md#atomicservicesearch) | `atomicservice.AtomicServiceSearch` | 搜索组件 |
| [AtomServiceTabs](03_Components.md#atomservicetabs) | `atomicservice.AtomicServiceTabs` | 标签页组件 |
| [AtomServiceWeb](03_Components.md#atomicserviceweb) | `atomicservice.AtomicServiceWeb` | Web 组件 |
| [CustomAppBar](03_Components.md#customappbar) | `atomicservice.CustomAppBar` | 应用栏组件 |
| [CustomAppBarMenuBar](03_Components.md#customappbarmenubar) | `atomicservice.AtomServiceMenuBar` | 菜单栏组件 |

### 启动组件

| 组件 | N-API 模块 | 主要功能 |
|------|------------|----------|
| [FullScreenLaunchComponent](03_Components.md#fullscreenlaunchcomponent) | - | 全屏启动组件 |
| [HalfScreenLaunchComponent](03_Components.md#halfscreenlaunchcomponent) | - | 半屏启动组件 |
| [InnerFullScreenLaunchComponent](03_Components.md#innerfullscreenlaunchcomponent) | - | 内部全屏启动组件 |

### 弹窗与导航组件

| 组件 | N-API 模块 | 主要功能 |
|------|------------|----------|
| [InterstitialDialogAction](03_Components.md#interstitialdialogaction) | - | 插屏弹窗 |
| [NavPushPathHelper](03_Components.md#navpushpathhelper) | `atomicservice.NavPushPathHelper` | HSP 路径管理 |

---

## API 快速参考

### 导出 JS API 的组件

| 组件 | JS API | C++ 实现 | 说明 |
|------|--------|----------|------|
| AtomServiceWeb | `checkUrl(bundleName, domainType, url)` | `CheckUrl()` | URL 策略校验 |
| NavPushPathHelper | `silentInstall()` | `HspSilentInstallNapi::SilentInstall` | HSP 静默安装 |
| NavPushPathHelper | `isHspExist()` | `HspSilentInstallNapi::IsHspExist` | 检查 HSP 存在 |
| NavPushPathHelper | `updateRouteMap()` | `HspSilentInstallNapi::UpdateRouteMap` | 更新路由表 |
| AtomServiceMenuBar | `setMenubarVisible()` | `MenubarAPIImplement::setMenubarVisible` | 菜单栏可见性 |

---

## 术语表

| 术语 | 说明 |
|------|------|
| ArkTS | OpenHarmony 声明式 UI 开发语言 |
| ABC | ArkTS 字节码格式 |
| N-API | Node.js API, OpenHarmony 原生模块接口 |
| HSP | Hot Stream Package, 热更包 |
| atomic service | 原子化服务, 即开即用的服务 |

---

## 相关资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [ArkUI 组件参考](https://docs.openharmony.cn/pages/master/zh-cn/application-dev/reference/apis-arkui/Readme-CN.md)
- [N-API 开发指南](https://docs.openharmony.cn/pages/master/zh-cn/application-dev/reference/native-lib/Readme-CN.md)