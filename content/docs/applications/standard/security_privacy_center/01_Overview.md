# 项目概览

## 项目定位

安全隐私中心（Security Privacy Center）是 OpenHarmony 系统中的安全设置入口模块，向用户提供位置服务总开关管理功能以及隐私权限相关应用的入口列表。

**核心定位**：

- **位置开关管理**：用户可在设备上统一管理位置服务的开启与关闭
- **隐私应用入口**：为已注册隐私相关功能的应用提供统一的入口展示
- **系统设置集成**：页面嵌入"设置"应用的"隐私"菜单中

**证据来源**：`README.md:11-16`

## 核心能力

### 1. 位置总开关管理

| 功能 | 描述 | 实现位置 |
|-----|------|---------|
| 查看位置开关状态 | 显示当前设备位置服务的开启状态 | `LocationService.ets:45-54` |
| 开启位置服务 | 用户点击开启位置服务 | `LocationService.ets:57-66` |
| 关闭位置服务 | 用户点击关闭位置服务 | `LocationService.ets:69-76` |
| 状态监听 | 监听系统位置开关变化 | `LocationService.ets:28-37` |

### 2. 隐私权限应用入口

| 功能 | 描述 | 实现位置 |
|-----|------|---------|
| 菜单列表展示 | 展示已注册的隐私功能入口 | `PrivacyProtectionListView.ets` |
| 菜单点击跳转 | 根据配置跳转到目标应用 | `AutoMenuModel.ets:107-145` |
| 三种跳转模式 | UIAbility / UIExtensionAbility / 页面跳转 | `AutoMenuModel.ets:33-37` |
| 动态刷新 | 从 BMS 实时获取配置并刷新 | `AutoMenuModel.ets:86-104` |

### 3. 菜单接入能力

外部应用可通过以下方式接入安全隐私中心：

1. **注册菜单配置**：在应用的 `module.json5` 中声明隐私菜单配置
2. **配置解析**：系统 BMS（Bundle Manager Service）解析配置
3. **菜单展示**：安全隐私中心从 BMS 获取配置并展示

**参考文档**：[应用接入指导说明](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/SecurityPrivacyCenter/auto-menu-guidelines.md)

## 运行环境

### 系统要求

| 要求 | 说明 |
|-----|------|
| 操作系统 | OpenHarmony |
| SDK 版本 | compileSdkVersion 23 |
| 运行时 | ArkTS/ETS 运行时 |
| 设备类型 | 支持标准设备（default） |

**证据来源**：`build-profile.json5:36-38`

### 依赖的系统能力

| 能力 | 用途 | 必需权限 |
|-----|------|---------|
| 位置服务（geoLocationManager） | 位置开关控制 | `ohos.permission.CONTROL_LOCATION_SWITCH` |
| 包管理（bundleManager） | 应用信息查询 | `ohos.permission.GET_BUNDLE_INFO` |
| 账户管理（osAccount） | 用户 ID 获取 | 无（系统 API） |
| 数据存储（relationalStore） | 菜单数据缓存 | 无（沙箱内） |

## 关键概念

### MVI 架构

本项目采用 **MVI（Model-View-Intent）** 架构模式：

| 概念 | 说明 | 示例文件 |
|-----|------|---------|
| Model | 数据模型与业务逻辑 | `AutoMenuModel.ets`、`LocationService.ets` |
| View | UI 组件与页面 | `Index.ets`、`PrivacyProtectionListView.ets` |
| Intent | 用户意图/页面事件 | `AutoMenuIntent.ets` |
| ViewState | 视图状态 | `AutoMenuViewState.ets` |
| ViewModel | 协调 Model 与 View | `AutoMenuViewModel.ets`、`LocationViewModel.ets` |

### 数据流

```
User Action → Intent → ViewModel → Model → Data Source
                ↓
            ViewState → View (UI Update)
```

### 页面结构

| 页面 | 路径 | 功能 |
|-----|------|------|
| 首页 | `pages/Index.ets` | 展示菜单列表 + 位置入口 |
| 位置服务页 | `pages/locationServices.ets` | 位置开关管理 |
| UIExtension 页 | `pages/UiExtensionPage.ets` | 承载第三方 UI |

**证据来源**：`module.json5:26`（pages 配置）

## 应用信息

| 属性 | 值 |
|-----|---|
| 应用名称 | 安全隐私中心 |
| 包名 | `com.ohos.security.privacycenter` |
| 版本 | 1.0.0 |
| 版本代码 | 1000000 |
| 许可证 | Apache License 2.0 |

**证据来源**：`AppScope/app.json5:18-22`

## 相关模块

| 模块 | 关系 | 说明 |
|-----|------|------|
| security_certificate_manager | 相关仓 | 证书管理模块 |

**证据来源**：`README.md:89-90`

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [02_Directory_Structure.md](./02_Directory_Structure.md) → 目录结构
- [03_Architecture.md](./03_Architecture.md) → 架构设计
