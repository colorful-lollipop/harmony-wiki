# 附录：关键调用链

> **目的**: 记录PermissionManager关键功能的完整调用链  
> **适用范围**: 开发者、架构师  
> **最后更新**: 2026-02-05

---

## 调用链 1: 权限请求完整流程

### 触发条件
应用调用 `abilityAccessCtrl.requestPermissionsFromUser()`

### 调用链

```
[系统服务层]
    │
    ├── AT服务验证请求
    │
    ▼
[PermissionManager应用]
    │
    ├── ServiceExtAbility.onRequest(want, startId)
    │   ├── deviceInfo.deviceType检查 (wearable跳过)
    │   ├── display.getDefaultDisplaySync() 获取屏幕信息
    │   └── GrantDialogModel.createWindow(context, name, rect, want)
    │       │
    │       ├── window.createWindow(config) 创建窗口
    │       │
    │       ├── getCallerAppInfo(want)
    │       │   ├── 解析want.parameters:
    │       │   │   ├── 'ohos.aafwk.param.callerBundleName'
    │       │   │   ├── 'ohos.aafwk.param.callerToken'
    │       │   │   ├── 'ohos.user.grant.permission'
    │       │   │   ├── 'ohos.user.grant.permission.state'
    │       │   │   └── 'ohos.ability.params.callback'
    │       │   ├── getUserId() 获取用户ID
    │       │   ├── bundleManager.getBundleInfoSync() 获取应用信息
    │       │   ├── initGrantResult() 初始化授权状态
    │       │   └── preProcessPermission() 预处理权限
    │       │       └── PermissionGroupManager.preProcessPermission()
    │       │           └── 具体策略.preProcessingPermission()
    │       │
    │       ├── getGroupWithPermission(reqPerms, reqPermsState)
    │       │   └── PermissionGroupManager.getGroupNameByPermission()
    │       │       └── 遍历所有策略.getPermissionGroupConfig()
    │       │
    │       ├── BindDialogTarget(win, want)
    │       │   ├── 解析callback和token
    │       │   └── win.bindDialogTarget() 绑定对话框
    │       │
    │       └── win.loadContent('pages/dialogPlus', storage)
    │
    ▼
[UI层]
    │
    ├── dialogPlus.ets (页面加载)
    │   ├── @LocalStorageProp('callerAppInfo') 获取数据
    │   ├── GrantDialogViewModel 初始化
    │   └── 显示权限请求对话框
    │
    ▼
[用户交互]
    │
    ├── 用户点击"允许"/"拒绝"
    │
    ▼
[业务逻辑层]
    │
    ├── GrantDialogModel.clickHandle(groupConfig, callerAppInfo, locationFlag, buttonStatus)
    │   ├── PermissionGroupManager.getPermissionOption()
    │   │   ├── PermissionGroupManager.getClickSituation() 转换按钮状态
    │   │   └── 具体策略.grantHandle() / denyHandle()
    │   │       └── 返回 PermissionOption (GRANT/REVOKE/SKIP)
    │   │
    │   ├── grantPermissionWithResult() / revokePermissionWithResult()
    │   │   ├── atManager.grantUserGrantedPermission() / revokeUserGrantedPermission()
    │   │   └── 返回操作结果
    │   │
    │   ├── writeToGrantResult() 更新授权结果
    │   │
    │   └── terminateWithResult(context, win, callerAppInfo)
    │       ├── data.writeInterfaceToken()
    │       ├── data.writeStringArray(callerAppInfo.reqPerms)
    │       ├── data.writeIntArray(callerAppInfo.grantResult)
    │       ├── callerAppInfo.proxy.sendMessageRequest(RESULT_CODE, ...)
    │       ├── data.reclaim() / reply.reclaim() 释放资源
    │       ├── win.destroyWindow() 销毁窗口
    │       └── context.terminateSelf() 结束Ability
    │
    ▼
[系统服务层]
    │
    ├── AT服务接收IPC回调
    ├── 更新权限状态
    └── 返回结果给调用方应用
```

**入口**: `ServiceExtAbility.ets:38`  
**出口**: `GrantDialogModel.ets:509`  
**关键文件**:
- ServiceExtAbility.ets
- GrantDialogModel.ets
- dialogPlus.ets
- PermissionGroupManager.ets

---

## 调用链 2: 权限管理设置启动流程

### 触发条件
用户通过 Settings → Privacy → Permission Manager 进入

### 调用链

```
[系统设置]
    │
    ├── 启动MainAbility (action: 'action.access.privacy.center')
    │
    ▼
[PermissionManager]
    │
    ├── MainAbility.onCreate(want)
    │   └── GlobalContext.store('bundleName', callerBundleName)
    │
    ├── MainAbility.onWindowStageCreate(windowStage)
    │   ├── permissionCheck() 检查权限
    │   │   ├── bundleManager.getBundleInfoForSelfSync()
    │   │   ├── atManager.verifyAccessTokenSync()
    │   │   └── 返回权限检查结果
    │   │
    │   ├── 如果指定了bundleName:
    │   │   └── getSperifiedApplication(bundleName)
    │   │       ├── bundleManager.getBundleInfo()
    │   │       ├── 构建应用信息对象
    │   │       ├── GlobalContext.store('applicationInfo', info)
    │   │       └── windowStage.setUIContent('pages/application-secondary')
    │   │
    │   └── 否则:
    │       └── getAllApplications()
    │           ├── account_osAccount.getAccountManager()
    │           ├── accountManager.getActivatedOsAccountLocalIds()
    │           ├── bundleManager.getAllBundleInfo(flag, userId)
    │           ├── 遍历应用列表:
    │           │   └── bundleManager.queryAbilityInfo() 过滤有效应用
    │           ├── 构建应用列表
    │           ├── windowStage.loadContent('pages/authority-management')
    │           │
    │           └── bundleMonitor.on('add/remove/update') 注册监听
    │               └── 应用变更时刷新列表
    │
    ▼
[UI层]
    │
    ├── authority-management.ets
    │   ├── 显示应用列表
    │   ├── 支持搜索和排序
    │   └── 点击进入应用详情
    │
    ├── application-secondary.ets (应用详情)
    │   └── 显示应用信息和权限列表
    │
    ├── authority-tertiary.ets (权限详情)
    │   └── 显示权限说明和应用列表
    │
    └── permission-access-record.ets (访问记录)
        └── 显示权限使用历史
```

**入口**: `MainAbility.ts:34`  
**出口**: `MainAbility.ts:79`  
**关键文件**:
- MainAbility.ts
- authority-management.ets
- application-secondary.ets

---

## 调用链 3: 权限策略选择流程

### 触发条件
权限请求时需要确定使用哪个策略处理

### 调用链

```
[权限请求处理]
    │
    ├── PermissionGroupManager.getGroupConfigs(callerAppInfo, context, appName, locationFlag, pasteBoardName)
    │   │
    │   ├── 遍历callerAppInfo.groupWithPermission
    │   │   │
    │   │   ├── groupHandle = this.permissionStrategies.get(permissionGroup)
    │   │   │   └── 从Map获取策略实例
    │   │   │       ├── LocationStrategy
    │   │   │       ├── CameraStrategy
    │   │   │       ├── MicrophoneStrategy
    │   │   │       └── ... (20个策略)
    │   │   │
    │   │   ├── groupConfig = groupHandle.getPermissionGroupConfig()
    │   │   │   └── 返回权限组配置（权限列表、图标、标题等）
    │   │   │
    │   │   ├── title = groupHandle.getGroupTitle(appName, locationFlag)
    │   │   │   └── 根据权限组返回不同标题格式
    │   │   │
    │   │   ├── readAndWrite = groupHandle.getReadAndWrite(permissions)
    │   │   │   └── 部分策略覆盖此方法（如文件权限）
    │   │   │
    │   │   ├── reason = groupHandle.getReasonByPermission(...)
    │   │   │   ├── 从callerAppInfo.reqPermsDetails查找权限详情
    │   │   │   ├── context.createModuleContext() 创建模块上下文
    │   │   │   ├── moduleContext.resourceManager.getStringSync() 获取理由文本
    │   │   │   └── 返回授权理由
    │   │   │
    │   │   ├── buttonList = groupHandle.getButtonList()
    │   │   │   └── 默认返回 [DENY, ALLOW]
    │   │   │       └── 部分策略返回更多选项
    │   │   │           └── LocationStrategy: [DENY, ALLOW_ONLY_DURING_USE, ALLOW]
    │   │   │
    │   │   └── 构建PermissionGroupConfig对象
    │   │
    │   └── 返回groupConfigList[]
    │
    ▼
[UI展示]
    │
    └── dialogPlus.ets 根据groupConfigList渲染对话框
```

**入口**: `PermissionGroupManager.ets:152`  
**出口**: `PermissionGroupManager.ets:182`  
**关键文件**:
- PermissionGroupManager.ets
- BasePermissionStrategy.ets
- 各具体策略类

---

## 调用链 4: IPC结果返回流程

### 触发条件
用户完成权限选择，需要返回结果给调用方

### 调用链

```
[用户确认]
    │
    ├── dialogPlus.ets 用户点击"确定"
    │
    ▼
[业务逻辑层]
    │
    ├── GrantDialogModel.terminateWithResult(context, win, callerAppInfo)
    │   │
    │   ├── 准备IPC数据
    │   │   ├── data.writeInterfaceToken(Constants.ACCESS_TOKEN)
    │   │   ├── data.writeStringArray(callerAppInfo.reqPerms)
    │   │   └── data.writeIntArray(callerAppInfo.grantResult)
    │   │
    │   ├── 发送结果
    │   │   └── callerAppInfo.proxy.sendMessageRequest(Constants.RESULT_CODE, data, reply, option)
    │   │       └── IPC通信到AT服务
    │   │
    │   ├── 发送完成信号
    │   │   └── callerAppInfo.proxy.sendMessageRequest(Constants.RESULT_CODE_1, setDialogData, reply, option)
    │   │
    │   ├── 异常处理 (try-catch-finally)
    │   │   ├── catch: Log.error() 记录错误
    │   │   └── finally:
    │   │       ├── data.reclaim() 释放MessageSequence
    │   │       ├── reply.reclaim()
    │   │       └── setDialogData.reclaim()
    │   │
    │   ├── 清理资源
    │   │   ├── GlobalContext.getContext().decreaseAndGetWindowNum()
    │   │   ├── win.destroyWindow() 销毁窗口
    │   │   └── windowNum <= 0 ? context.terminateSelf() : null
    │   │
    │   └── 流程结束
    │
    ▼
[系统服务层]
    │
    ├── AT服务接收IPC消息
    ├── 解析授权结果
    ├── 更新权限数据库
    └── 回调给调用方应用
```

**入口**: `GrantDialogModel.ets:496`  
**出口**: `GrantDialogModel.ets:522`  
**关键文件**:
- GrantDialogModel.ets
- @ohos.rpc (系统模块)

---

## 调用链 5: 安全组件对话框流程

### 触发条件
应用使用安全组件（如CameraButton）

### 调用链

```
[应用使用安全组件]
    │
    ├── 系统启动SecurityExtAbility
    │
    ▼
[PermissionManager]
    │
    ├── SecurityExtAbility.onCreate(want)
    │   └── globalThis.windowNum = 0
    │
    ├── SecurityExtAbility.onRequest(want, startId)
    │   ├── 解析want.parameters:
    │   │   ├── 'ohos.display.width'
    │   │   ├── 'ohos.display.height'
    │   │   ├── 'ohos.caller.uid'
    │   │   └── 'ohos.ability.params.windowId'
    │   │
    │   └── createWindow(name, windowType, rect, want)
    │       │
    │       ├── GlobalContext.load('dialogSet') 获取对话框集合
    │       ├── 构建token = callerToken + '_' + windId
    │       ├── 检查dialogSet.has(token) 防止重复
    │       │
    │       ├── window.createWindow(config) 创建窗口
    │       ├── win.bindDialogTarget(tokenValue, callback)
    │       │   └── 设置窗口关闭回调:
    │       │       ├── win.destroyWindow()
    │       │       ├── dialogSet.delete(token)
    │       │       └── dialogSet.size === 0 ? terminateSelf() : null
    │       │
    │       ├── win.setFollowParentWindowLayoutEnabled(true)
    │       ├── win.loadContent('pages/securityDialog', storage)
    │       ├── win.setWindowBackgroundColor('#00000000')
    │       └── win.showWindow()
    │
    ▼
[UI层]
    │
    ├── securityDialog.ets
    │   └── 显示安全组件授权对话框
    │
    ▼
[用户交互]
    │
    └── 用户授权后关闭对话框
```

**入口**: `SecurityExtAbility.ets:39`  
**出口**: `SecurityExtAbility.ets:118`  
**关键文件**:
- SecurityExtAbility.ets
- securityDialog.ets

---

## 调用链总结表

| 调用链 | 入口文件 | 入口函数 | 核心文件数 | 复杂度 |
|--------|----------|----------|------------|--------|
| 权限请求流程 | ServiceExtAbility.ets | onRequest | 10+ | 高 |
| 权限管理设置 | MainAbility.ts | onWindowStageCreate | 8+ | 中 |
| 策略选择 | PermissionGroupManager.ets | getGroupConfigs | 22+ | 中 |
| IPC结果返回 | GrantDialogModel.ets | terminateWithResult | 3 | 低 |
| 安全组件对话框 | SecurityExtAbility.ets | onRequest | 5 | 中 |

---

## 关键函数索引

### Ability生命周期

| 函数 | 文件 | 行号 | 说明 |
|------|------|------|------|
| onCreate | ServiceExtAbility.ets | 30 | ServiceExtAbility创建 |
| onRequest | ServiceExtAbility.ets | 38 | 处理权限请求 |
| onCreate | MainAbility.ts | 27 | MainAbility创建 |
| onWindowStageCreate | MainAbility.ts | 34 | 窗口创建 |
| onCreate | SecurityExtAbility.ets | 31 | SecurityExtAbility创建 |
| onRequest | SecurityExtAbility.ets | 39 | 处理安全组件请求 |

### 核心业务流程

| 函数 | 文件 | 行号 | 说明 |
|------|------|------|------|
| getCallerAppInfo | GrantDialogModel.ets | 56 | 解析调用方信息 |
| createWindow | GrantDialogModel.ets | 225 | 创建权限对话框 |
| clickHandle | GrantDialogModel.ets | 382 | 处理用户点击 |
| terminateWithResult | GrantDialogModel.ets | 496 | 返回授权结果 |
| getGroupConfigs | PermissionGroupManager.ets | 152 | 获取权限组配置 |
| getPermissionOption | PermissionGroupManager.ets | 193 | 获取权限操作选项 |
| getAllApplications | MainAbility.ts | 159 | 获取所有应用 |

---

*返回 [README](README.md) | [常见问题排查](07_Troubleshooting.md) | [配置标志](Config_Flags.md)*
