# 项目概览

## 目的

本文档为企业设备管理(EDM - Enterprise Device Management)组件提供快速入门指南，帮助新成员理解项目定位、核心能力和整体架构。

## 适用范围

- 目标读者：EDM组件开发者、MDM应用开发者、系统集成工程师
- 覆盖内容：项目概述、核心能力、架构概览
- 不包含：详细API参考、插件实现细节

## 关键结论

- EDM是OpenHarmony的系统级组件，提供企业设备管理能力
- 采用插件化架构，支持126+个策略插件
- 提供JS/N-API接口供应用调用，通过IPC与系统服务通信
- 支持超级管理员、普通管理员和BYOD管理员三种权限模型

## 相关跳转

- [01_Project_Positioning.md](01_Project_Positioning.md) - 详细的项目定位和边界
- [02_Directory_Structure.md](02_Directory_Structure.md) - 完整的代码组织
- [03_Architecture.md](03_Architecture.md) - 详细架构设计
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - N-API接口文档

---

## 什么是EDM？

EDM (Enterprise Device Management) 是OpenHarmony的企业设备管理组件，为MDM（Mobile Device Management）应用提供系统级的管理能力。

### 核心价值

- **企业管控**: 允许企业对设备进行统一管控和策略配置
- **多级权限**: 支持超级管理员、普通管理员和BYOD管理员分级管理
- **插件扩展**: 通过插件机制动态加载和执行设备管理策略
- **安全合规**: 提供设备信息获取、应用管控、安全策略设置等能力

---

## 核心能力

### 1. 设备管理员管理

| 能力 | 说明 | 权限要求 |
|------|------|----------|
| 启用管理员 | EnableAdmin、EnableDeviceAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN |
| 禁用管理员 | DisableAdmin、DisableDeviceAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN |
| 查询管理员 | GetEnabledAdmin、GetAdmins | - |
| 授权管理员 | AuthorizeAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN |
| 替换超级管理员 | ReplaceSuperAdmin | MANAGE_ENTERPRISE_DEVICE_ADMIN |

证据：`interfaces/kits/admin_manager/src/admin_manager_addon.cpp`

### 2. 设备策略管理

涵盖以下能力分类：

- **系统管理**: 重启、关机、恢复出厂、锁屏
- **应用管控**: 安装/卸载限制、应用黑名单/白名单、Kiosk模式
- **网络管控**: WiFi、蓝牙、移动数据、VPN、防火墙
- **安全管控**: 密码策略、指纹认证、相机禁用、USB控制
- **系统设置**: 日期时间、壁纸、输入法、浏览器策略

证据：`interfaces/inner_api/common/include/edm_ipc_interface_code.h`

### 3. 信息查询

- 设备信息：序列号、版本号、MAC地址、IP地址
- 应用信息：已安装应用列表、应用详情
- 网络信息：网络接口、IP配置
- 蓝牙信息：蓝牙状态、配对设备

---

## 架构概览

### 组件层级

```
┌─────────────────────────────────────────────────────────────┐
│                  MDM应用层                            │
│          (JS/TS N-API调用)                          │
└────────────────────┬──────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                MDM Kit层                              │
│         17个Manager NAPI模块                         │
└────────────────────┬──────────────────────────────────────┘
                     │ IPC
                     ▼
┌─────────────────────────────────────────────────────────────┐
│          EnterpriseDeviceManagerService                   │
│           (SA 1601 SystemAbility)                      │
│  ┌────────┐  ┌────────┐  ┌──────────────┐ │
│  │AdminMgr │  │PolicyMgr│  │ PluginManager │ │
│  └────────┘  └────────┘  └──────────────┘ │
│       │            │              │              │
│       ▼            ▼              ▼              │
│  ┌────────────────────────────────────────────────────┐ │
│  │         126+个策略插件                    │ │
│  │  (device_core/communication/sys_service)   │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 关键模块

| 模块 | 文件位置 | 职责 |
|------|----------|------|
| AdminManager | services/edm/include/admin_manager.h | 管理员生命周期 |
| PolicyManager | services/edm/include/policy_manager.h | 策略持久化和查询 |
| PluginManager | services/edm/include/plugin_manager.h | 插件加载和执行 |
| PermissionChecker | services/edm/include/permission_checker.h | 权限校验 |

---

## 管理员类型

### 三级权限模型

```
Super Device Admin (超级管理员)
       │
       │ 激活/授权
       ▼
Normal Device Admin (普通管理员)
       │
       │ 执行策略
       ▼
BYOD Device Admin (BYOD管理员)
```

| 类型 | AdminType | 权限级别 | 激活方式 |
|-----|----------|----------|----------|
| Super Admin | ENT (0) | 最高 | 超级管理员激活或授权 |
| Normal Admin | NORMAL (1) | 中等 | 由超级管理员激活 |
| BYOD Admin | BYOD (2) | 中等 | BYOD流程激活 |

证据：`interfaces/inner_api/plugin_kits/include/iplugin.h:41-46`

---

## 系统能力

### SA ID和进程

- **SystemAbility ID**: 1601
- **进程名**: edm
- **库文件**: libedmservice.z.so
- **启动方式**: 按需启动（binder call或参数persist.edm.edm_enable=true）

证据：`sa_profile/1601.json:5`

### 依赖的系统能力

来自`bundle.json`的40+个依赖：
- ability_runtime、bundle_framework、ipc、safwk（基础框架）
- access_token、permission（权限系统）
- wifi、bluetooth、location（网络/位置）
- storage_service、relational_store（存储）
- ...等

---

## 运行环境

### 平台支持

- **适配系统**: Standard（标准系统）
- **架构支持**: ARM64、X86_64
- **ROM占用**: 1800KB
- **RAM占用**: 6293KB

证据：`bundle.json:40-110`

### 编译特性开关

30+个feature flags控制不同能力模块的启用：
- `enterprise_device_management_support_all` (true) - 全功能支持
- `wifi_edm_enable`、`bluetooth_edm_enable`、`location_edm_enable` 等
- 根据依赖的系统组件自动启用/禁用

证据：`common/config/common.gni:16-162`

---

## 关键概念

### 1. 设备管理员 (Device Administrator)

拥有设备管理权限的应用，可以设置和执行企业管控策略。

### 2. 策略 (Policy)

企业对设备设置的行为约束或配置，如：
- 禁止某些应用运行
- 禁用WiFi
- 设置屏幕超时
- 密码强度要求

### 3. 插件 (Plugin)

实现特定策略的可加载模块，每个插件：
- 实现`IPlugin`接口
- 对应一个或多个`funcCode`
- 支持策略设置和查询

### 4. 委托策略 (Delegated Policy)

超级管理员可以授权普通管理员管理某些策略，实现权限委托。

### 5. 管理事件 (Managed Event)

系统事件通知机制，管理员可以订阅：
- 应用安装/卸载
- 应用启动/停止
- 系统更新
- 账号切换

---

## 快速开始

### 开发者

1. **了解架构**: 阅读 [03_Architecture.md](03_Architecture.md)
2. **学习API**: 参考 [04_External_API_NAPI.md](04_External_API_NAPI.md)
3. **查看示例**: 参考 [admin_provisioning](https://gitcode.com/openharmony/applications_admin_provisioning) 示例应用

### MDM应用开发者

1. **申请权限**: 在module.json中声明需要的EDM权限
2. **激活管理员**: 调用`adminManager.enableAdmin()`
3. **设置策略**: 使用各Manager设置设备策略
4. **监听事件**: 订阅MANAGED_EVENT获取通知

---

## 相关资源

- [官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/enterprise-device-management-V5)
- [示例仓库](https://gitcode.com/openharmony/applications_admin_provisioning)
- [相关组件](https://gitcode.com/openharmony/customization_config_policy)
