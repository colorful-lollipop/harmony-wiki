# 01_项目定位与核心能力

> 本文档基于代码证据编写，证据来源见各章节引用。

## 项目身份

| 属性 | 值 |
|-----|-----|
| **项目名称** | admin_provisioning |
| **应用名称** | AdminProvisioning |
| **包名** | com.ohos.adminprovisioning (`AppScope/app.json:3`) |
| **版本** | 1.0.1 (`AppScope/app.json:6`) |
| **许可证** | Apache-2.0 |

## 项目定位

AdminProvisioning 是 OpenHarmony 系统预置的企业设备管理应用，主要功能是在企业环境下为设备发放 MDM (Mobile Device Management) 业务。

### 业务价值

```
企业管理员
    ↓ 配置 MDM 策略
AdminProvisioning (SDA/DA)
    ↓ 调用系统 API
OpenHarmony 系统
    ↓ 执行策略
设备管理策略生效
```

### 代码证据

- **README 定义** (`README.md:5`):
  > "As a system application preset in OpenHarmony, AdminProvisioning is used to provision MDM services on devices in enterprise environments."

- **中文说明** (`README_zh.md:5`):
  > "用于企业环境下在设备上发放Mobile Device Management(MDM)业务"

## 核心能力详解

### 1. SDA (Super Device Admin) 超级管理员

**组件**: `AutoManagerAbility`

**能力**:
- 系统开机导航提供 SDA 业务开放接口
- 调用设备管理权限接口实现对系统的深度定制
- 适用于需要全面控制设备的场景

**证据** (`README_zh.md:13`):
  > "AutoManagerAbility为系统开机导航提供SDA(Super Device Admin)业务开放接口, 调用设备管理权限接口，实现对系统的定制"

**入口**:
```
ability: com.ohos.automanager.AutoManagerAbility
srcEntrance: ./ets/MainAbility/AutoManagerAbility.ts
launchType: singleton
visible: true
权限: ohos.permission.PROVISIONING_MESSAGE
```

### 2. DA (Device Admin) 设备管理员

**组件**: `MainAbility`

**能力**:
- 为三方 MDM 客户端应用提供 DA 业务开放接口
- 权限范围小于 SDA，实现对系统的有限定制
- 适用于第三方 MDM 应用集成

**证据** (`README_zh.md:14`):
  > "MainAbility为三方MDM客户端应用提供DA(Device Admin)业务开放接口，其调用接口与SDA调用的设备管理权限接口存在差异"

**入口**:
```
ability: com.ohos.adminprovisioning.MainAbility
srcEntrance: ./ets/MainAbility/MainAbility.ts
launchType: singleton
visible: false
```

### 3. 定制化业务发放 UI

**组件**: `MDMUIExtensionAbility`

**能力**:
- 提供 UI 扩展能力用于定制化业务发放界面
- 读取企业配置文件 (`edm_provision_config.json`)
- 展示企业信息和条款

**证据** (`UIExtensionAbility.ets:27-69`):
  ```typescript
  // 读取配置文件
  let realpath = 'etc/edm/edm_provision_config.json';
  await configPolicy.getOneCfgFile(realpath).then((value: string) => {
    let configStr = fs.readTextSync(value);
    let jsonArray = JSON.parse(configStr);
    admin.abilityName = jsonArray.admin_info[0].admin.abilityName;
    admin.bundleName = jsonArray.admin_info[0].admin.bundleName;
  })
  ```

## 技术特征

### 架构类型

| 特征 | 说明 |
|-----|-----|
| **应用类型** | ArkTS HAP (前端应用) |
| **Native 代码** | 无 (纯 ArkTS) |
| **N-API 绑定** | 无 |
| **Ability 组件** | 3 个 (UIAbility × 2, UIExtensionAbility × 1) |

### 运行时依赖

```
ArkUI 框架
    ↓
ArkTS 应用层 (Ability + Page + Component)
    ↓
OpenHarmony 系统 API (@ohos.*)
    ↓
系统服务层
```

### 系统 API 调用清单

| API 模块 | 用途 | 权限要求 |
|---------|-----|---------|
| `@ohos.enterprise.adminManager` | 管理员管理、企业信息 | MANAGE_ENTERPRISE_DEVICE_ADMIN |
| `@ohos.bundle.bundleManager` | Bundle 查询 | GET_BUNDLE_INFO |
| `@ohos.account.osAccount` | 账户管理 | MANAGE_LOCAL_ACCOUNTS |
| `@ohos.update` | 恢复出厂设置 | FACTORY_RESET |
| `@ohos.configPolicy` | 配置文件读取 | 无 |
| `@ohos.file.fs` | 文件系统操作 | 无 |

## 与其他模块的关系

### 上游依赖

| 模块 | 关系 | 说明 |
|-----|-----|-----|
| OpenHarmony 框架 | 依赖 | 提供 ArkUI、系统 API |
| customization_enterprise_device_management | 调用 | MDM 策略实现 |
| customization_config_policy | 调用 | 配置策略解析 |

### 下游调用

| 调用方 | 接口 | 说明 |
|-------|-----|-----|
| 系统开机流程 | AutoManagerAbility | SDA 激活 |
| 三方 MDM 应用 | MainAbility | DA 激活 |
| 系统设置 | (潜在) | 管理员管理 |

## 能力边界

### 可以在本应用做

- ✅ 激活/停用设备管理员
- ✅ 获取管理员状态
- ✅ 获取企业信息
- ✅ 展示管理员应用信息
- ✅ 恢复出厂设置（需权限）
- ✅ 读取配置策略文件

### 不能在本应用做

- ❌ 直接访问硬件（需通过系统 API）
- ❌ 修改系统底层配置（需 SDA 权限）
- ❌ 绕过权限验证调用敏感 API
- ❌ 访问其他应用的私有数据

---

*文档版本: 1.0*
