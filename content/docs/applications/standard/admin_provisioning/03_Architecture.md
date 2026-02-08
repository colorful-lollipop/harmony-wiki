# 03_架构设计

> 本文档基于代码证据编写，证据来源见各章节引用。

## 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                    AdminProvisioning 应用层                       │
│  ┌─────────────────┐  ┌──────────────────┐  ┌─────────────────┐ │
│  │ MainAbility     │  │ AutoManagerAbility│  │UIExtensionAbility│ │
│  │ (DA 模式)       │  │ (SDA 模式)        │  │ (定制化发放 UI)  │ │
│  └────────┬────────┘  └────────┬─────────┘  └────────┬────────┘ │
│           │                   │                     │          │
│           └───────────────────┼─────────────────────┘          │
│                               ↓                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    ArkUI 页面层                          │   │
│  │  applicationInfo │ autoManager │ byod │ custProvisioning │   │
│  └─────────────────────────────────────────────────────────┘   │
│                               ↓                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    公共组件/数据层                         │   │
│  │  logger │ utils │ baseData │ accountManager │ appDetailData │ │
│  └─────────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统 API 层                        │
│  ┌──────────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│  │@ohos.enterprise. │  │ @ohos.bundle.   │  │ @ohos.account. │  │
│  │ adminManager     │  │ bundleManager   │  │ osAccount      │  │
│  └──────────────────┘  └─────────────────┘  └────────────────┘  │
│  ┌──────────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│  │ @ohos.update     │  │ @ohos.config    │  │ @ohos.file.fs  │  │
│  │                  │  │ Policy          │  │                │  │
│  └──────────────────┘  └─────────────────┘  └────────────────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统服务层                        │
│  Enterprise Device Management Service │ Bundle Manager Service  │
│  Account Management Service │ Update Service │ Config Service  │
└─────────────────────────────────────────────────────────────────┘
```

## 组件说明

### Ability 组件矩阵

| 组件 | 类型 | 可见性 | 权限 | 主页 | 职责 |
|-----|-----|-------|-----|-----|-----|
| `MainAbility` | UIAbility | 隐藏 | - | `pages/applicationInfo` | DA 入口，处理设备管理员激活 |
| `AutoManagerAbility` | UIAbility | 显示 | `PROVISIONING_MESSAGE` | `pages/autoManager/managerStart` | SDA 入口，超级管理员激活 |
| `MDMUIExtensionAbility` | UIExtensionAbility | - | - | `pages/custProvisioning` | 定制化发放 UI |

### 代码证据

**MainAbility 生命周期** (`MainAbility.ts:21-53`):
```typescript
export default class MainAbility extends UIAbility {
  onCreate(want, launchParam): void {
    this.localStorage.setOrCreate('adminProvisioningWant', want);
  }
  onWindowStageCreate(windowStage): void {
    windowStage.setUIContent(this.context, 'pages/applicationInfo', this.localStorage);
  }
  // ... 其他生命周期方法
}
```

**AutoManagerAbility 生命周期** (`AutoManagerAbility.ts:24-56`):
```typescript
export default class AutoManagerAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    this.localStorage.setOrCreate('autoManagerAbilityWant', want);
  }
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/autoManager/managerStart', this.localStorage);
  }
}
```

## 数据流设计

### 管理员激活数据流

```
用户点击激活按钮
    ↓
applicationInfo.ets:activateAdmin()
    ↓
edmEnterpriseDeviceManager.enableAdmin(want, enterpriseInfo, adminType)
    ↓
@ohos.enterprise.adminManager API
    ↓
企业设备管理服务
    ↓
返回结果 (成功/失败)
    ↓
terminateAbilityPage()
    ↓
结束
```

**证据** (`applicationInfo.ets:285-321`):
```typescript
async activateAdmin(adminType: adminManager.AdminType) {
  let wantTemp: Want = {
    bundleName: elementNameVal.bundleName,
    abilityName: elementNameVal.abilityName,
  };
  await edmEnterpriseDeviceManager.enableAdmin(wantTemp,
    { name: enterInfo.name, description: enterInfo.description },
    edmEnterpriseDeviceManager.AdminType.ADMIN_TYPE_NORMAL)
    .catch((error: BusinessError) => {
      ret = false;
    });
}
```

### 配置文件读取数据流

```
MDMUIExtensionAbility.onSessionCreate()
    ↓
configPolicy.getOneCfgFile('etc/edm/edm_provision_config.json')
    ↓
fs.readTextSync(filePath)
    ↓
JSON.parse(configStr)
    ↓
提取 adminInfo, enterpriseInfo, adminType
    ↓
存储到 LocalStorage
    ↓
加载页面 UI
```

**证据** (`UIExtensionAbility.ets:30-69`):
```typescript
async onSessionCreate(want: Want, session: UIExtensionContentSession): Promise<void> {
  let realpath = 'etc/edm/edm_provision_config.json';
  await configPolicy.getOneCfgFile(realpath).then((value: string) => {
    let configStr = fs.readTextSync(value);
    if (configStr) {
      let jsonArray = JSON.parse(configStr);
      admin.abilityName = jsonArray.admin_info[0].admin.abilityName;
      admin.bundleName = jsonArray.admin_info[0].admin.bundleName;
    }
  });
  session.loadContent('pages/custProvisioning/custProvisioning', this.localStorage);
}
```

## 线程模型

### ArkUI 线程模型

```
Main Thread (UI 线程)
    │
    ├── UI 渲染 (ArkUI)
    ├── 页面逻辑 (aboutToAppear, onClick 等)
    └── 系统 API 调用 (@ohos.*)
    │
    └── Promise/Callback 异步回调
```

### 关键约束

1. **UI 操作必须在主线程**: 所有 UI 更新必须在 `build()` 或生命周期方法中
2. **异步操作使用 Promise**: 如 `await edmEnterpriseDeviceManager.enableAdmin()`
3. **不支持多线程**: ArkTS 不支持创建新线程，所有并发通过异步实现

**证据** (`applicationInfo.ets:212-227`):
```typescript
async aboutToAppear(): Promise<void> {
  await this.getCheckAbilityList(this.applicationInfo, elementNameVal, isAdminTypeVal);
}

async isAdminActive() {
  retAppState = await edmEnterpriseDeviceManager.isAdminEnabled(wantTemp);
  retSuperState = await edmEnterpriseDeviceManager.isSuperAdmin(name);
}
```

## 页面路由

### 路由矩阵

| 入口 | 页面 | 用途 |
|-----|-----|-----|
| MainAbility | `pages/applicationInfo` | 应用信息展示和管理员状态 |
| AutoManagerAbility | `pages/autoManager/managerStart` | SDA 设置开始页 |
| AutoManagerAbility | `pages/autoManager/loadingInfo` | SDA 设置加载页 |
| AutoManagerAbility | `pages/autoManager/termsShowPage` | 条款展示页 |
| AutoManagerAbility | `pages/autoManager/unitManagerShowPage` | 设备管理页 |
| AutoManagerAbility | `pages/autoManager/setFinishSuccess` | 设置成功页 |
| AutoManagerAbility | `pages/autoManager/setFinishFail` | 设置失败页 |
| MDMUIExtensionAbility | `pages/custProvisioning` | 定制化发放页 |
| 通用 | `pages/byod/*` | BYOD 相关页面 |

### 路由跳转方式

**证据** (`managerStart.ets:80-97`):
```typescript
.onClick(() => {
  router.pushUrl({ url: 'pages/autoManager/termsShowPage' })
})

.onClick(() => {
  router.pushUrl({ url: 'pages/autoManager/unitManagerShowPage' })
})
```

---

*文档版本: 1.0*
