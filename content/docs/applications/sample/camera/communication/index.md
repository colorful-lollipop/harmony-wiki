# 项目概览

> Communication Sample - OpenHarmony WiFi 通信子系统示例

---

## 目的

本文档提供 `@ohos/camera_sample_communication` 项目的快速概览，帮助读者理解项目的整体定位和目标。

## 适用范围

本文档适用于：
- 想要快速了解项目的开发者
- 准备深入阅读架构文档的读者
- 需要 WiFi 通信示例的 OpenHarmony 开发者

## 关键结论

- **项目类型**：OpenHarmony 示例代码，展示 wpa_supplicant 的使用方法
- **核心功能**：提供 hostapd（接入点）、wpa_supplicant（客户端）、wpa_cli（控制客户端）三个示例程序
- **技术栈**：C 语言 + wpa_supplicant 2.9 + 动态库加载
- **目标系统**：OpenHarmony mini / small 系统
- **不包含**：N-API、JavaScript API、IPC/ServiceAbility 机制

## 相关跳转

- [项目定位与边界](01_Project_Position.md) - 详细的项目定位和边界说明
- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [架构说明](03_Architecture.md) - 组件交互和数据流

---

## 项目简介

`@ohos/camera_sample_communication` 是 OpenHarmony 系统的 WiFi 通信子系统示例代码，展示了如何在 OpenHarmony 平台上使用 wpa_supplicant 库实现 WiFi 功能。

### 项目特点

1. **示例性质**：项目主要用于展示 wpa_supplicant 的集成和使用方法
2. **轻量级**：代码简洁，便于理解和学习
3. **动态加载**：使用 dlopen 动态加载 libwpa.so 库
4. **多组件**：包含接入点、客户端、控制客户端三个组件

### 核心组件

| 组件 | 可执行文件 | 功能 | 入口文件 |
|------|-----------|------|---------|
| hostapd | hostapd | WiFi 接入点（AP 模式） | hostapd/src/hostapd_sample.c |
| wpa_supplicant | wpa_supplicant | WiFi 客户端（STA 模式） | wpa_supplicant/src/wpa_sample.c |
| wpa_cli | wpa_cli | WiFi 控制客户端 | wpa_cli/src/wpa_cli_sample.c |

### 技术架构

```
┌─────────────────────────────────────────┐
│         应用层（示例程序）             │
├─────────────────────────────────────────┤
│  hostapd │ wpa_supplicant │ wpa_cli   │
└─────────────────────────────────────────┘
         │              │           │
         │ dlopen       │           │
         ▼              │           │
┌─────────────────────────────────────────┐
│         libwpa.so (动态库)             │
└─────────────────────────────────────────┘
         │
         │ wpa_ctrl 接口（UDP）
         ▼
┌─────────────────────────────────────────┐
│      WiFi 驱动 / 硬件接口层            │
└─────────────────────────────────────────┘
```

---

## 快速开始

### 构建项目

```bash
# 编译 hostapd
hb build -f //applications/sample/camera/communication/hostapd

# 编译 wpa_supplicant
hb build -f //applications/sample/camera/communication/wpa_supplicant

# 编译 wpa_cli
hb build -f //applications/sample/camera/communication/wpa_cli
```

### 运行示例

```bash
# 启动 wpa_supplicant（客户端模式）
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf

# 运行 wpa_cli 控制程序
wpa_cli

# 启动 hostapd（接入点模式）
hostapd /etc/hostapd.conf
```

---

## 文档导航

### 按主题阅读

| 主题 | 文档 |
|------|------|
| 理解项目定位 | [项目定位与边界](01_Project_Position.md) |
| 了解代码组织 | [目录结构](02_Directory_Structure.md) |
| 掌握架构设计 | [架构说明](03_Architecture.md) |
| 查看构建配置 | [GN Targets](06_GN_Targets.md) |
| 了解编译产物 | [编译产物](07_Build_Artifacts.md) |
| 安全风险评估 | [安全风险评审](08_Security_Audit.md) |

### 按角色阅读

| 角色 | 推荐阅读顺序 |
|------|-------------|
| 新手开发者 | 概览 → 项目定位 → 目录结构 → 架构说明 |
| 构建工程师 | GN Targets → 编译产物 → 目录结构 |
| 安全审计人员 | 项目定位 → 架构说明 → 安全风险评审 |
| 集成开发者 | 架构说明 → GN Targets → 编译产物 |

---

## 版本信息

| 项目 | 值 |
|------|-----|
| 项目版本 | 3.1 |
| OpenHarmony 兼容 | mini, small 系统 |
| 第三方依赖 | wpa_supplicant 2.9 |

---

## 限制说明

### 不支持的功能

- ❌ N-API / JavaScript API 绑定
- ❌ IPC / ServiceAbility 跨进程通信
- ❌ 权限检查机制
- ❌ 安全验证逻辑

### 设计考虑

本项目作为示例代码， intentionally 简化了部分实现：
- 无复杂的权限管理
- 无输入验证和参数检查
- 无错误恢复机制
- 直接使用动态库加载

生产环境使用时需要根据实际需求完善相关功能。
