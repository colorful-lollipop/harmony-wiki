# AdminProvisioning 项目概览

> 本文档基于代码证据编写，证据来源见各章节引用。

## 项目定位

**AdminProvisioning** 是 OpenHarmony 预置的系统应用，用于企业环境下在设备上发放 Mobile Device Management (MDM) 业务。

### 核心能力

| 能力类型 | 说明 | 对应组件 |
|---------|-----|---------|
| **SDA (Super Device Admin)** | 系统开机导航提供超级管理员业务开放接口，调用设备管理权限接口实现系统深度定制 | `AutoManagerAbility` |
| **DA (Device Admin)** | 为三方 MDM 客户端应用提供设备管理员业务开放接口，实现系统有限定制 | `MainAbility` |
| **定制化发放** | 支持企业定制化配置下发 | `MDMUIExtensionAbility` |

### 代码证据

- **项目定义**: `README.md:5`
  > "AdminProvisioning应用是OpenHarmony中预置的系统应用，用于企业环境下在设备上发放Mobile Device Management(MDM)业务"

- **架构说明**: `README_zh.md:11-14`
  ```
  - AutoManagerAbility为系统开机导航提供SDA业务开放接口
  - MainAbility为三方MDM客户端应用提供DA业务开放接口
  ```

## 运行环境

| 环境项 | 要求 |
|-------|-----|
| **OpenHarmony 版本** | API 9+ (minAPIVersion: 9, targetAPIVersion: 9) |
| **设备类型** | default, tablet, 2in1 (`module.json5:8-12`) |
| **构建工具** | hvigor 1.0.6 (`package.json:13-15`) |
| **应用类型** | 系统应用 (安装于 `/system/app`) |

## 关键特性

### 1. 双管理员模式

- **超级管理员 (SDA)**: 拥有系统级权限，可深度定制设备
- **普通管理员 (DA)**: 权限受限，仅能进行有限定制

### 2. 权限管理

应用声明了以下系统权限 (`module.json5:63-79`)：
- `ohos.permission.MANAGE_LOCAL_ACCOUNTS`
- `ohos.permission.GET_BUNDLE_INFO`
- `ohos.permission.MANAGE_ENTERPRISE_DEVICE_ADMIN`
- `ohos.permission.FACTORY_RESET`
- `ohos.permission.INTERNET`
- `ohos.permission.PROVISIONING_MESSAGE` (AutoManagerAbility 专属)

### 3. 无 N-API 绑定

> **重要**: 本项目是纯 ArkTS HAP 前端应用，**不包含任何 N-API (Native API) 绑定**。
>
> 所有系统能力调用均通过 OpenHarmony 提供的 ArkTS/JS API 实现。

## 与其他模块的关系

```
AdminProvisioning
    ↓ 调用
@ohos.enterprise.adminManager (企业设备管理)
@ohos.bundle (Bundle 管理)
@ohos.account.osAccount (账户管理)
@ohos.update (系统更新)
    ↓ 依赖
customization_enterprise_device_management (MDM 策略实现)
customization_config_policy (配置策略)
```

## 快速开始

### 开发环境
1. 安装 DevEco Studio
2. 配置 OpenHarmony SDK
3. 导入本项目

### 构建与安装
```bash
# Debug 构建
hvigor --mode module -p product=phone assembleHapDebug

# Release 构建 (需要签名)
hvigor --mode module -p product=phone assembleHapRelease

# 安装到设备
hdc file send adminprovisioning.hap /system/app/
hdc shell reboot
```

详细步骤请参阅 [使用说明](../doc/Instructions.md)。

## 相关文档

| 主题 | 链接 |
|-----|-----|
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| API 参考 | [04_API_Reference.md](./04_API_Reference.md) |
| 构建系统 | [05_Build_System.md](./05_Build_System.md) |
| 安全评审 | [07_Security_Review.md](./07_Security_Review.md) |

---

*文档版本: 1.0*
