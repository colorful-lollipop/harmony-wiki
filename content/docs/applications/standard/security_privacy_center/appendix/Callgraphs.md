# 关键调用链

## 概述

本文档描述安全隐私中心模块的关键调用链，帮助开发者理解从用户操作到系统调用的完整流程。

## 入口调用链

### 1. 应用启动流程

```
用户点击"设置" → "隐私"菜单
        │
        ▼
系统启动安全隐私中心应用
        │
        ├──▶ 加载 entry-default-signed.hap
        │       │
        │       └──▶ 初始化 ArkTS 运行时
        │
        ├──▶ 实例化 EntryAbility
        │       │
        │       ├──▶ onCreate(want, launchParam)
        │       │       │ [EntryAbility.ets:25-27]
        │       │       └── Logger.info(TAG, 'Ability onCreate....');
        │       │
        │       └──▶ onWindowStageCreate(windowStage)
        │               │ [EntryAbility.ets:33-43]
        │               │
        │               └──▶ windowStage.loadContent('pages/Index')
        │                       │
        │                       └──▶ 渲染 Index 页面
        │                               │ [Index.ets]
        │                               │
        │                               ├──▶ onPageShow()
        │                               │       │ [Index.ets:56-60]
        │                               │       │
        │                               │       ├──▶ AutoMenuRefreshIntent
        │                               │       │       │
        │                               │       │       └──▶ autoMenuViewModel.processIntent()
        │                               │       │               │ [AutoMenuViewModel.ets]
        │                               │       │               │
        │                               │       │               └──▶ AutoMenuModel.getMenuInfoListFromBms()
        │                               │       │                       │ [AutoMenuModel.ets:86-104]
        │                               │       │                       │
        │                               │       │                       ├──▶ _getUserId()
        │                               │       │                       │       └──▶ osAccount.getAccountManager()
        │                               │       │                       │
        │                               │       │                       ├──▶ AutoMenuManager.getMenuConfigFromBms()
        │                               │       │                       │       └──▶ bundleManager 查询
        │                               │       │                       │
        │                               │       │                       └──▶ _refreshDb()
        │                               │       │                               └──▶ RDB 写入
        │                               │       │
        │                               │       └──▶ BundleInfoModel.getAllBundleInfoByFunctionAccess()
        │                               │               │ [BundleInfoModel.ets]
        │                               │               └──▶ bundleManager 查询
        │                               │
        │                               └──▶ 初始化 LocationService
        │                                       │ [LocationViewModel.ets:26-35]
        │                                       │
        │                                       └──▶ LocationService.startService()
        │                                               │ [LocationService.ets:28-38]
        │                                               │
        │                                               └──▶ geolocation.on('locationEnabledChange')
        │                                                       └── 注册状态变化监听
```

### 2. 位置开关操作流程

```
用户点击位置服务入口
        │
        ▼
Index.ets:139-150 (onClick)
        │
        ├──▶ HiSysEventUtil.reportLocationClick('LOCATION')
        │       │ [HiSysEventUtil.ets]
        │       └──▶ 上报系统事件
        │
        └──▶ router.pushUrl({ url: 'pages/locationServices' })
                │
                └──▶ 跳转到位置服务页面
                        │ [locationServices.ets]
                        │
                        ├──▶ onPageShow()
                        │       │
                        │       └──▶ locationVM.initViewModel()
                        │               │ [LocationViewModel.ets:26-35]
                        │               │
                        │               ├──▶ LocationService.registerListener()
                        │               │       └──▶ 注册状态回调
                        │               │
                        │               ├──▶ LocationService.startService()
                        │               │       └──▶ geolocation.on('locationEnabledChange')
                        │               │
                        │               └──▶ LocationService.getServiceState()
                        │                       │ [LocationService.ets:45-55]
                        │                       │
                        │                       └──▶ geolocation.isLocationEnabled()
                        │                               │
                        │                               └──▶ 返回布尔值
                        │                                       │
                        │                                       └──▶ mListener.updateServiceState(state)
                        │                                               │
                        │                                               └──▶ AppStorage.setOrCreate()
                        │
用户点击开关按钮
        │
        ▼
locationServices.ets (开关组件 onClick)
        │
        ├──▶ if (currentState) → locationVM.disableLocation()
        │       │ [LocationViewModel.ets:47-50]
        │       │
        │       └──▶ LocationService.disableLocation()
        │               │ [LocationService.ets:69-76]
        │               │
        │               └──▶ geolocation.disableLocation()
        │                       │
        │                       └──▶ 关闭位置服务
        │
        └──▶ else → locationVM.enableLocation()
                │ [LocationViewModel.ets:42-45]
                │
                └──▶ LocationService.enableLocation()
                        │ [LocationService.ets:57-67]
                        │
                        └──▶ geolocation.enableLocation()
                                │
                                └──▶ 开启位置服务
```

### 3. 菜单点击操作流程

```
用户点击菜单项
        │
        ▼
PrivacyProtectionListView (itemClickEvent)
        │
        └──▶ AutoMenuClickIntent
                │
                └──▶ autoMenuViewModel.processIntent()
                        │ [AutoMenuViewModel.ets:57-60]
                        │
                        └──▶ AutoMenuModel.handleMenuClick(menuInfo)
                                │ [AutoMenuModel.ets:107-145]
                                │
                                ├──▶ 判断 dstAbilityMode
                                │       │
                                │       ├──▶ if (0) DST_UIABILITY_MODE
                                │       │       │ [AutoMenuModel.ets:109-119]
                                │       │       │
                                │       │       └──▶ context.startAbility()
                                │       │               │
                                │       │               └──▶ 启动目标 UIAbility
                                │       │
                                │       ├──▶ if (1) DST_ABILITY_MODE
                                │       │       │ [AutoMenuModel.ets:132-143]
                                │       │       │
                                │       │       └──▶ router.pushUrl({
                                │       │               url: 'pages/UiExtensionPage',
                                │       │               params: { dstBundleName, dstAbilityName }
                                │       │       })
                                │       │               │
                                │       │               └──▶ 跳转到 UiExtensionPage
                                │       │                       │ [UiExtensionPage.ets]
                                │       │                       │
                                │       │                       └──▶ UIExtensionComponent
                                │       │                               │
                                │       │                               └──▶ 加载第三方 UI
                                │       │
                                │       └──▶ if (2) DST_PAGE_MODE
                                │               │ [AutoMenuModel.ets:121-129]
                                │               │
                                │               └──▶ router.pushUrl({
                                │                       url: menuInfo.dstAbilityName
                                │                       })
                                │                       │
                                │                       └──▶ 跳转到本地页面
                                │
                                └──▶ HiSysEventUtil.reportAccessClick()
                                        └──▶ 上报访问事件
```

## 系统 API 调用链

### bundleManager 调用

```
Index.ets:64
    │
    └──▶ BundleInfoModel.getAllBundleInfoByFunctionAccess()
            │
            └──▶ bundleManager.getAllBundleInfoByFunctionAccess(bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION)
                    │
                    └──▶ IPC 调用 BMS
                            │
                            └──▶ 返回 BundleInfo[]
```

### geoLocationManager 调用

```
LocationService.ets:48
    │
    └──▶ geolocation.isLocationEnabled()
            │
            └──▶ IPC 调用位置服务
                    │
                    └──▶ 返回 boolean

LocationService.ets:60
    │
    └──▶ geolocation.enableLocation()
            │
            └──▶ IPC 调用位置服务
                    │
                    └──▶ 开启位置服务（需 CONTROL_LOCATION_SWITCH 权限）

LocationService.ets:72
    │
    └──▶ geolocation.disableLocation()
            │
            └──▶ IPC 调用位置服务
                    │
                    └──▶ 关闭位置服务（需 CONTROL_LOCATION_SWITCH 权限）
```

### relationalStore 调用

```
AutoMenuModel.ets:44
    │
    └──▶ RdbManager.getInstance().query()
            │
            └──▶ relationalStore.RdbPredicates
                    │
                    └──▶ RDB 查询（ANTO_MENU_TABLE_V2）
                            │
                            └──▶ 返回 ResultSet

AutoMenuModel.ets:154
    │
    └──▶ RdbManager.getInstance().insert()
            │
            └──▶ RDB 插入
                    │
                    └──▶ 缓存菜单数据

AutoMenuModel.ets:149
    │
            └──▶ RdbManager.getInstance().delete()
                    │
                    └──▶ RDB 删除
                            │
                            └──▶ 清除旧缓存
```

## 返回导航

- [SUMMARY.md](../SUMMARY.md) → 文档导航
- [03_Architecture.md](../03_Architecture.md) → 架构设计
- [05_Inner_API.md](../05_Inner_API.md) → 内部 API
