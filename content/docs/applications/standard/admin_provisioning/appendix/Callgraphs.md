# 附录：关键调用链

> 本文档基于代码证据编写，证据来源见各章节引用。

## 1. 管理员激活调用链

### 1.1 DA 管理员激活

```
用户点击激活按钮
    │
    ├── 入口: pages/applicationInfo.ets:168
    │   └── Button onClick()
    │
    ├── 触发: activateAdmin(adminType)
    │   └── 位置: applicationInfo.ets:285
    │
    ├── 构建 Want 参数
    │   └── applicationInfo.ets:287-290
    │       bundleName: elementNameVal.bundleName
    │       abilityName: elementNameVal.abilityName
    │
    ├── 调用 API: edmEnterpriseDeviceManager.enableAdmin()
    │   └── 位置: applicationInfo.ets:295
    │       权限: MANAGE_ENTERPRISE_DEVICE_ADMIN
    │       参数:
    │       ├── want: Want
    │       ├── enterpriseInfo: { name, description }
    │       └── adminType: ADMIN_TYPE_NORMAL
    │
    ├── 等待结果
    │   └── Promise 异步回调
    │
    ├── 成功 → terminateAbilityPage()
    │   └── 位置: applicationInfo.ets:319
    │       调用: appDetailData.terminateAbilityPage()
    │       位置: appDetailData.ets:145
    │       API: (getContext(this) as common.UIAbilityContext).terminateSelf()
    │
    └── 失败 → 记录错误日志
        └── applicationInfo.ets:298-301
```

**证据** (`applicationInfo.ets:285-321`):

```typescript
async activateAdmin(adminType: adminManager.AdminType) {
  let wantTemp: Want = {
    bundleName: elementNameVal.bundleName,
    abilityName: elementNameVal.abilityName,
  };
  await this.isAdminActive();
  if (!this.isShowActive) {
    if (adminType === edmEnterpriseDeviceManager.AdminType.ADMIN_TYPE_NORMAL) {
      await edmEnterpriseDeviceManager.enableAdmin(wantTemp,
        { name: enterInfo.name, description: enterInfo.description },
        edmEnterpriseDeviceManager.AdminType.ADMIN_TYPE_NORMAL)
        .catch((error: BusinessError) => {
          ret = false;
        });
    }
  } else {
    if (this.isEnableButton) {
      await edmEnterpriseDeviceManager.disableAdmin(wantTemp)
        .catch((error: BusinessError) => {
          ret = false;
        });
    }
  }
  if (ret) {
    appDetailData.terminateAbilityPage();
  }
}
```

---

### 1.2 SDA 管理员激活

```
用户点击"接受并继续"按钮
    │
    ├── 入口: pages/autoManager/managerStart.ets:135
    │   └── Button onClick()
    │
    ├── 跳转: router.pushUrl()
    │   └── 页面: pages/autoManager/loadingInfo
    │
    ├── 加载信息: loadingInfo.ets
    │   └── aboutToAppear()
    │
    ├── 解析参数: parseManageParameter()
    │   └── managerStart.ets:179-208
    │       存储到 AppStorage:
    │       ├── manageAbilityName
    │       ├── manageBundleName
    │       ├── manageUrl
    │       ├── manageTermsName
    │       ├── manageTermsContent
    │       ├── manageEnterpriseName
    │       └── manageEnterpriseDescription
    │
    ├── 加载配置: UIExtensionAbility
    │   └── 位置: UIExtensionAbility.ets:30-69
    │
    ├── 读取配置文件
    │   ├── API: configPolicy.getOneCfgFile()
    │   └── 文件: etc/edm/edm_provision_config.json
    │
    └── 展示定制化发放 UI
        └── pages/custProvisioning/custProvisioning
```

---

## 2. 管理员状态查询调用链

```
页面显示: applicationInfo.ets
    │
    ├── 入口: onPageShow()
    │   └── 位置: applicationInfo.ets:233
    │
    ├── 触发: isAdminActive()
    │   └── 位置: applicationInfo.els:253
    │
    ├── 查询 DA 状态
    │   └── API: edmEnterpriseDeviceManager.isAdminEnabled(wantTemp)
    │       位置: applicationInfo.ets:267
    │       返回: boolean
    │
    ├── 查询 SDA 状态
    │   └── API: edmEnterpriseDeviceManager.isSuperAdmin(name)
    │       位置: applicationInfo.ets:269
    │       返回: boolean
    │
    └── 更新 UI
        ├── isShowActive: boolean
        └── isEnableButton: boolean
```

**证据** (`applicationInfo.ets:253-283`):

```typescript
async isAdminActive() {
  let wantTemp: Want = {
    bundleName: elementNameVal.bundleName,
    abilityName: elementNameVal.abilityName,
  };
  retAppState = await edmEnterpriseDeviceManager.isAdminEnabled(wantTemp);
  retSuperState = await edmEnterpriseDeviceManager.isSuperAdmin(name);
  if (!retAppState) {
    this.isShowActive = false;
    this.isEnableButton = true;
  } else {
    if (retSuperState) {
      this.isEnableButton = false;
    } else {
      this.isEnableButton = true;
    }
    this.isShowActive = true;
  }
}
```

---

## 3. 应用信息查询调用链

```
页面加载: applicationInfo.ets
    │
    ├── 入口: aboutToAppear()
    │   └── 位置: applicationInfo.ets:212
    │
    ├── 调用: getCheckAbilityList()
    │   └── 位置: applicationInfo.ets:362
    │
    ├── 步骤1: 获取 Want 参数
    │   └── getAbilityWantVal()
    │       位置: applicationInfo.ets:334
    │       来源: LocalStorage.get('adminProvisioningWant')
    │       验证: utils.checkObjPropertyValid()
    │
    ├── 步骤2: 检查应用是否有效
    │   └── appDetailData.checkAppItem()
    │       位置: appDetailData.ets:40
    │       API: bundle.queryExtensionAbilityInfo()
    │       参数:
    │       ├── want: ElementName
    │       ├── type: ENTERPRISE_ADMIN
    │       └── userId: localId
    │
    ├── 步骤3: 获取 Bundle 信息
    │   └── appDetailData.getBundleInfoItem()
    │       位置: appDetailData.ets:72
    │       API: bundle.getBundleInfo()
    │       参数:
    │       ├── bundleName
    │       ├── bundleFlag: GET_BUNDLE_INFO_WITH_REQUESTED_PERMISSION
    │       └── userId
    │
    ├── 步骤4: 获取资源信息
    │   └── appDetailData.getResourceItem()
    │       位置: appDetailData.ets:109
    │       操作:
    │       ├── 获取应用图标
    │       └── 获取应用名称
    │
    ├── 步骤5: 获取权限列表
    │   └── appDetailData.getPermissionList()
    │       位置: appDetailData.ets:159
    │       来源: data.reqPermissionDetails
    │
    └── 步骤6: 更新 UI
        └── applicationInfo.appIcon, appTitle, appPermissionList
```

**证据** (`appDetailData.ets:56-62`):

```typescript
try {
  await bundle.queryExtensionAbilityInfo(want, bundle.ExtensionAbilityType.ENTERPRISE_ADMIN,
  bundle.ExtensionAbilityFlag.GET_EXTENSION_ABILITY_INFO_WITH_APPLICATION, userId.localId)
    .then((result) => {
      data = result;
    })
} catch (e) {
  logger.error(TAG, 'checkAppItem queryExtensionAbilityInfo try fail! ' + JSON.stringify(e));
}
```

---

## 4. 账户信息查询调用链

```
查询当前用户 ID
    │
    ├── 入口: accountManager.getAccountUserId()
    │   位置: accountManager.ets:27
    │
    ├── API: account_osAccount.getAccountManager()
    │   位置: accountManager.ets:28
    │
    └── API: queryCurrentOsAccount()
        位置: accountManager.ets:28
        返回: OsAccountInfo
        └── localId: number
```

**证据** (`accountManager.ets:27-35`):

```typescript
async getAccountUserId(userId: UserId): Promise<boolean> {
  let accountInfo = await account_osAccount.getAccountManager().queryCurrentOsAccount();
  if (!utils.isValid(accountInfo)) {
    logger.warn(TAG, 'getAccountUserId queryCurrentOsAccount fail! userId is null');
    return false;
  }
  userId.localId = accountInfo.localId;
  return true;
}
```

---

## 5. 恢复出厂设置调用链

```
触发恢复出厂设置
    │
    ├── 入口: resetFactory.rebootAndCleanUserData()
    │   位置: resetFactory.ets:23
    │
    ├── API: update.getRestorer()
    │   位置: resetFactory.ets:24
    │
    └── API: factoryReset()
        位置: resetFactory.ets:25
        权限: FACTORY_RESET
        异步: Promise<void>
```

**证据** (`resetFactory.ets:23-30`):

```typescript
rebootAndCleanUserData() {
  let restorer = update.getRestorer();
  restorer.factoryReset().then(() => {
    logger.info(TAG, 'rebootAndCleanUserData factoryReset success')
  }).catch((err: BusinessError) => {
    logger.error(TAG, 'rebootAndCleanUserData err=' + JSON.stringify(err))
  })
}
```

---

## 6. 配置文件读取调用链

```
UIExtensionAbility 启动
    │
    ├── 入口: onSessionCreate()
    │   位置: UIExtensionAbility.ets:30
    │
    ├── 构建配置文件路径
    │   └── realpath: 'etc/edm/edm_provision_config.json'
    │
    ├── 调用 API: configPolicy.getOneCfgFile()
    │   位置: UIExtensionAbility.ets:43
    │   异步: Promise<string>
    │
    ├── 读取文件内容
    │   └── fs.readTextSync(value)
    │       位置: UIExtensionAbility.ets:45
    │
    ├── 解析 JSON
    │   └── JSON.parse(configStr)
    │       位置: UIExtensionAbility.ets:48
    │
    └── 提取配置信息
        ├── adminInfo:
        │   ├── abilityName
        │   └── bundleName
        ├── enterpriseInfo:
        │   ├── name
        │   └── description
        └── adminType: ADMIN_TYPE_SUPER | ADMIN_TYPE_NORMAL
```

**证据** (`UIExtensionAbility.ets:42-68`):

```typescript
async onSessionCreate(want: Want, session: UIExtensionContentSession): Promise<void> {
  let realpath = 'etc/edm/edm_provision_config.json';
  await configPolicy.getOneCfgFile(realpath).then((value: string) => {
    logger.info(TAG, 'getOneCfgFile value is : ' + value);
    let configStr = fs.readTextSync(value);
    logger.info(TAG, 'getOneCfgFile configStr is ' + JSON.stringify(configStr));
    if (configStr) {
      let jsonArray = JSON.parse(configStr);
      admin.abilityName = jsonArray.admin_info[0].admin.abilityName;
      admin.bundleName = jsonArray.admin_info[0].admin.bundleName;
      enterpriseInfo.name = jsonArray.enterprise_info.organization_name;
      enterpriseInfo.description = jsonArray.enterprise_info.description;
    }
  });
}
```

---

## 7. 完整数据流图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           用户交互层                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │ 点击激活按钮 │    │ 点击继续按钮 │    │ 恢复出厂设置 │              │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘              │
└─────────┼────────────────────┼────────────────────┼─────────────────────┘
          │                    │                    │
          ↓                    ↓                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                          页面逻辑层                                      │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    applicationInfo.ets                          │    │
│  │  ├── activateAdmin() → enableAdmin/disableAdmin                │    │
│  │  ├── isAdminActive() → isAdminEnabled/isSuperAdmin             │    │
│  │  └── getCheckAbilityList() → bundle query                      │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│          │                    │                    │                    │
│          ↓                    ↓                    ↓                    │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    appDetailData.ets                            │    │
│  │  ├── checkAppItem() → queryExtensionAbilityInfo                 │    │
│  │  ├── getBundleInfoItem() → getBundleInfo                       │    │
│  │  ├── getResourceItem() → resourceManager                        │    │
│  │  └── getPermissionList() → reqPermissionDetails                │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│          │                    │                    │                    │
│          ↓                    ↓                    ↓                    │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    accountManager.ets                           │    │
│  │  └── getAccountUserId() → queryCurrentOsAccount                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────┬───────────────────────────────────────────────────────────────┘
          │
          ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                          系统 API 层                                    │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐            │
│  │adminManager    │  │bundleManager   │  │osAccount      │            │
│  │@ohos.enterprise│  │@ohos.bundle   │  │@ohos.account  │            │
│  └────────────────┘  └────────────────┘  └────────────────┘            │
│  ┌────────────────┐  ┌────────────────┐                                 │
│  │update          │  │configPolicy   │                                 │
│  │@ohos.update    │  │@ohos.config   │                                 │
│  └────────────────┘  └────────────────┘                                 │
└─────────────────────────────────────────────────────────────────────────┘
```

---

*文档版本: 1.0*
