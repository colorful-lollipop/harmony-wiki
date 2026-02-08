# 附录A: 调用链

> 目的: 详细记录关键功能的调用链  
> 适用范围: 开发者、架构师

---

## 1. 锁屏启动调用链

### 1.1 系统启动流程

```
系统启动
    │
    ▼
System Server初始化
    │
    ▼
Ability Manager Service启动
    │
    ▼
启动com.ohos.systemui ServiceExtAbility
    │
    ├──► ServiceExtAbility.onCreate() 
    │         │
    │         ├──► Log.showInfo(TAG, 'onCreate')
    │         │
    │         ├──► AbilityManager.setContext(ABILITY_NAME_SCREEN_LOCK, this.context)
    │         │         │
    │         │         └──► 缓存Ability上下文
    │         │
    │         ├──► TimeManager.init(this.context)
    │         │         │
    │         │         ├──► dataShare.createDataShareHelper()
    │         │         └──► 订阅时间变化事件
    │         │
    │         ├──► ServiceExtAbility.statusBarWindow()
    │         │         │
    │         │         ├──► display.getDefaultDisplay()
    │         │         │         └──► 获取屏幕宽高
    │         │         │
    │         │         └──► AbilityManager.setAbilityData(ABILITY_NAME_STATUS_BAR, "rect", rect)
    │         │                   └──► 存储状态栏尺寸
    │         │
    │         └──► ServiceExtAbility.createWindow(Constants.WIN_NAME)
    │                   │
    │                   ├──► windowManager.create(context, name, TYPE_KEYGUARD)
    │                   │         │
    │                   │         ├──► Window.create()
    │                   │         └──► 返回win实例
    │                   │
    │                   ├──► win.loadContent("pages/index")
    │                   │         │
    │                   │         ├──► 加载index.ets
    │                   │         ├──► 初始化ViewModel
    │                   │         └──► 渲染UI
    │                   │
    │                   ├──► win.getUIContext()
    │                   │         └──► AppStorage.SetOrCreate('UIContext', UIContext)
    │                   │
    │                   └──► win.show()
    │                             └──► 显示锁屏窗口
    │
    └──► ServiceExtAbility.onDestroy() (应用退出时)
              │
              └──► TimeManager.release()
```

---

## 2. 解锁流程调用链

### 2.1 滑动解锁

```
用户滑动解锁
    │
    ▼
SlideScreenLock.onTouchEvent()
    │
    ▼
slideScreenLockViewModel.unlockScreen()
    │
    ▼
ScreenLockService.unlockScreen()
    │
    ├──► checkPinAuthProperty()
    │         │
    │         ├──► AccountsModel.getAuthProperty()
    │         │         │
    │         │         └──► userAuthManager.getProperty()
    │         │                   └──► 返回认证类型
    │         │
    │         └──► 设置mRouterPath
    │               ("pages/digitalPassword" | "pages/mixedPassword" | ...)
    │
    └──► Router.pushUrl({ url: mRouterPath })
              └──► 跳转到密码页面
```

### 2.2 数字密码验证

```
用户点击数字键盘
    │
    ▼
DigitalPSD.onKeyPress(value)
    │
    ▼
DigitalPSDViewModel.onKeyPress(value)
    │
    ├──► 收集6位密码
    │
    └──► 密码完整后:
              │
              ▼
         DigitalPSDViewModel.verifyPassword(password)
              │
              ▼
         ScreenLockService.authUser(password)
              │
              ▼
         AccountsModel.authUser(challenge, AuthType.PIN, authLevel)
              │
              ├──► registerInputer(password)
              │         │
              │         └──► pinAuthManager.registerInputer({ onGetData })
              │
              └──► userAuthManager.authUser(userId, challenge, PIN, authLevel, callback)
                        │
                        ├──► 系统触发PIN输入请求
                        │
                        ├──► onGetData(passType, inputData) 被调用
                        │         │
                        │         ├──► textEncoder.encode(password)
                        │         └──► inputData.onSetData(passType, uint8PW)
                        │
                        ├──► 系统验证密码
                        │
                        └──► callback.onResult(result, extraInfo)
                                  │
                                  ├──► result === AUTH_SUCCESS:
                                  │         │
                                  │         ▼
                                  │     ScreenLockService.unlocking()
                                  │         │
                                  │         ├──► ScreenLockModel.hiddenScreenLockWindow()
                                  │         │         │
                                  │         │         ├──► windowManager.find(Constants.WIN_NAME)
                                  │         │         └──► win.hide()
                                  │         │
                                  │         └──► ScreenLockMar.sendScreenLockEvent(EVENT_UNLOCK_SCREEN, ...)
                                  │                   └──► 通知系统解锁成功
                                  │
                                  └──► result !== AUTH_SUCCESS:
                                            │
                                            ▼
                                        BaseViewModel.showErrorTip("密码错误")
```

---

## 3. 通知显示调用链

### 3.1 通知订阅与显示

```
系统启动/通知服务初始化
    │
    ▼
NotificationManager.subscribe(tag, subscriber, callback)
    │
    ▼
Notification.subscribe(subscriber, asyncCallback)
    │
    ▼
系统通知服务
    │
    └──► 当有新通知时:
              │
              ▼
         subscriber.onConsume(data)
              │
              ▼
         NotificationService.onConsume(data)
              │
              ├──► 解析通知数据
              │         │
              │         ├──► ParseDataUtil.parseNotification(data)
              │         │         └──► 提取标题、内容、图标等
              │         │
              │         └──► 创建NotificationItem
              │
              ├──► NotificationManager.notify(notificationItem)
              │         │
              │         └──► 更新通知列表
              │
              └──► EventManager.publish(NOTIFICATION_CHANGE_EVENT)
                        │
                        ▼
                    NotificationList.onNotificationChange()
                              │
                              ├──► 刷新通知列表UI
                              └──► 显示新通知
```

### 3.2 通知点击处理

```
用户点击通知项
    │
    ▼
NotificationItem.onClick()
    │
    ▼
NotificationManager.handleNotificationClick(notification)
    │
    ├──► 检查是否需要解锁
    │         │
    │         ├──► 已解锁:
    │         │         │
    │         │         ▼
    │         │     WantAgent.trigger(wantAgentInfo)
    │         │             └──► 启动对应应用
    │         │
    │         └──► 未解锁:
    │                   │
    │                   ▼
    │               提示用户先解锁
    │
    └──► Notification.remove(hashCode, ...)
              └──► 从通知列表移除
```

---

## 4. 事件传播调用链

### 4.1 屏幕开关事件

```
系统检测到屏幕状态变化
    │
    ▼
系统发送公共事件
    │
    ├──► COMMON_EVENT_SCREEN_OFF
    │
    └──► COMMON_EVENT_SCREEN_ON
              │
              ▼
ScreenLockManager.mSubscriber 收到事件
    │
    ▼
ScreenLockManager.回调函数(err, data)
    │
    ├──► 检查err.code === 0
    │
    ├──► 根据data.event判断:
    │         │
    │         ├──► COMMON_EVENT_SCREEN_OFF:
    │         │         │
    │         │         ▼
    │         │     ScreenLockManager.notifyScreenEvent(false)
    │         │             │
    │         │             └──► sEventManager.publish(SCREEN_CHANGE_EVENT, false)
    │         │                       │
    │         │                       └──► EventBus.emit(SCREEN_CHANGE_EVENT, false)
    │         │                                 │
    │         │                                 ├──► ScreenLockService.onScreenChange(false)
    │         │                                 │         │
    │         │                                 │         └──► ScreenLockService.lockScreen()
    │         │                                 │
    │         │                                 └──► 其他订阅者处理
    │         │
    │         └──► COMMON_EVENT_SCREEN_ON:
    │                   │
    │                   ▼
    │               ScreenLockManager.notifyScreenEvent(true)
    │                       │
    │                       └──► sEventManager.publish(SCREEN_CHANGE_EVENT, true)
    │
    └──► (防抖处理: debounceTimeout = 500ms)
```

### 4.2 用户切换事件

```
系统用户切换
    │
    ▼
osAccount.on('activate') 触发
    │
    ▼
SwitchUserManager.回调函数
    │
    ▼
SwitchUserManager.updateCurrentUser(userId)
    │
    ├──► 更新当前用户ID
    │
    ├──► EventManager.publish(USER_SWITCH_EVENT, userId)
    │         │
    │         └──► EventBus.emit(USER_SWITCH_EVENT, userId)
    │                   │
    │                   ├──► AccountsModel.onUserSwitch(userId)
    │                   │         │
    │                   │         └──► 更新当前用户信息
    │                   │
    │                   ├──► ScreenLockService.onUserSwitch(userId)
    │                   │         │
    │                   │         └──► 刷新锁屏状态
    │                   │
    │                   └──► 其他订阅者处理
    │
    └──► 更新UI显示
```

---

## 5. 窗口管理调用链

### 5.1 窗口创建

```
调用WindowManager.createWindow(context, name, rect, loadContent)
    │
    ▼
WindowManager.createWindow()
    │
    ├──► Log.showInfo(TAG, `createWindow name: ${name}...`)
    │
    ├──► Window.create(context, name, SYSTEM_WINDOW_TYPE_MAP[name])
    │         │
    │         └──► 系统创建窗口
    │
    ├──► win.moveTo(rect.left, rect.top)
    │
    ├──► win.resetSize(rect.width, rect.height)
    │
    ├──► win.loadContent(loadContent)
    │         │
    │         └──► 加载页面内容
    │
    ├──► this.mWindowInfos.set(name, { visibility: false, rect })
    │         └──► 缓存窗口信息
    │
    └──► Log.showInfo(TAG, `create window[${name}] success.`)
```

### 5.2 窗口显示

```
调用WindowManager.showWindow(name)
    │
    ▼
WindowManager.showWindow()
    │
    ├──► Log.showInfo(TAG, `showWindow name: ${name}`)
    │
    ├──► Window.find(name)
    │         └──► 查找窗口实例
    │
    ├──► window.show()
    │         └──► 系统显示窗口
    │
    ├──► this.mWindowInfos.set(name, { ..., visibility: true })
    │         └──► 更新窗口状态
    │
    ├──► sEventManager.publish(WINDOW_SHOW_HIDE_EVENT, { windowName: name, isShow: true })
    │         │
    │         └──► 通知订阅者窗口显示
    │
    └──► Log.showInfo(TAG, `show window[${name}] success.`)
```

---

## 6. 状态管理调用链

### 6.1 AppStorage状态更新

```
状态变更
    │
    ▼
AppStorage.SetOrCreate(key, value)
    │
    ▼
ArkUI框架
    │
    ├──► 存储状态值
    │
    ├──► 通知所有订阅该状态的组件
    │         │
    │         └──► @StorageLink(key) 装饰的变量自动更新
    │
    └──► 触发UI重新渲染
              │
              └──► 组件build()方法重新执行
```

### 6.2 组件@State更新

```
用户交互或事件触发
    │
    ▼
修改@State变量
    │
    ▼
ArkUI状态管理
    │
    ├──► 标记组件为脏状态
    │
    ├──► 调度UI更新（异步）
    │
    └──► 重新渲染组件
              │
              └──► 只更新变化的部分（差异化渲染）
```

---

## 7. 认证流程调用链

### 7.1 人脸认证

```
ServiceExtAbility初始化或用户选择人脸解锁
    │
    ▼
AccountsModel.checkFaceAuthAvailable()
    │
    ▼
userAuthManager.getProperty({ authType: AuthType.FACE })
    │
    ▼
如果支持人脸:
    │
    ▼
用户触发人脸解锁
    │
    ▼
ScreenLockService.authUser()
    │
    ▼
AccountsModel.authUser(challenge, AuthType.FACE, authLevel)
    │
    ▼
userAuthManager.authUser(userId, challenge, FACE, authLevel, callback)
    │
    ├──► 系统调起人脸认证界面
    │
    ├──► 用户面对摄像头
    │
    ├──► 系统识别人脸
    │
    └──► callback.onResult(result, extraInfo)
              │
              ├──► 识别成功: 解锁
              │
              └──► 识别失败: 提示错误
```

---

*说明: 调用链中的"►"表示调用关系，缩进表示层级。实际调用可能涉及异步操作（Promise/回调）。*
