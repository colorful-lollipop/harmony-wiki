# 附录：关键调用链图谱

## 1. 应用启动调用链

```
用户点击图标
    │
    ▼
系统启动 Ability
    │
    ▼
AbilityStage.onCreate()
    │
    ▼
MainAbility.onCreate(want, param)
    │   ├── initPhotosPref()
    │   ├── parseWantParameter() ──────────┐
    │   ├── UserFileManagerAccess.onCreate() ──┐
    │   ├── MediaObserver.registerForAllPhotos()   │
    │   ├── MediaObserver.registerForAllAlbums()   │
    │   ├── TimelineDataSourceManager.getInstance()
    │   └── BroadCast 事件注册
    │                                   │
    ▼                                   ▼
onWindowStageCreate()              ├─────────────────────┐
    │                               │                     │
    ├── ScreenManager 初始化        │                     │
    ├── WindowStage 设置            │                     │
    └── setUIContent('pages/index') │                     │
                                    │                     │
                                    ▼                     ▼
                              单选入口                多选入口
                              router.replaceUrl()    router.replaceUrl()
                                    │                     │
                                    ▼                     ▼
                              ThirdSelectPhotoGridPage
```

---

## 2. 外部调用路由链

```
startAbility/sendResult
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│                 MainAbility.parseWantParameter()            │
│                                                             │
│  根据 want.parameters.uri 路由:                            │
│                                                             │
│  ├── "photodetail" ────► PhotoBrowser (相机回调)           │
│  ├── "singleselect" ──► ThirdSelectPhotoGridPage (单选)    │
│  ├── "multipleselect" ─► ThirdSelectPhotoGridPage (多选)    │
│  ├── "form" ───────────► PhotoBrowser (FA 浏览)            │
│  ├── "formNone" ───────► DefaultPhotoPage (FA 默认)        │
│  ├── "formId" ─────────► FormEditorPage (FA 编辑)          │
│  └── "action.viewData" ► PhotoBrowser (外部查看)           │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 媒体数据流

```
MediaLibrary (系统服务)
        │
        │ 1. 查询请求
        ◄───────────────────────── MediaDataSource
        │
        │ 2. 返回结果集
        ─────────────────────────►
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                   MediaObserver                             │
│  监听 MediaLibrary 数据变化:                                 │
│                                                             │
│  ├── registerForAllPhotos() ── 照片变化监听                 │
│  └── registerForAllAlbums() ── 相册变化监听                 │
└─────────────────────────────────────────────────────────────┘
        │
        │ 3. 数据变化通知
        ◄──────────────────────────────
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                  DataSource (AbsDataSource)                 │
│                                                             │
│  ├── notifyDataChange() ── 通知 LazyForEach 刷新            │
│  └── getRawData(index) ──── 获取指定索引数据                │
└─────────────────────────────────────────────────────────────┘
        │
        │ 4. UI 刷新
        ◄──────────────────────────────────
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                    LazyForEach                             │
│                                                             │
│  ├── onAreaChange() ── 区域变化监听                         │
│  └── DataChangeListener ── 数据变化监听                    │
└─────────────────────────────────────────────────────────────┘
        │
        │ 5. 组件重新渲染
        ───────────────────────────────►
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                   Image/Video Component                      │
│                                                             │
│  ├── src: "file://..." 或 URI                               │
│  ├── objectFit: cover/contain/fill                          │
│  └── autoResize: true                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. 页面跳转链

```
index.ets (首页)
    │
    ├── router.pushUrl('pages/PhotoBrowser')
    │       │
    │       ▼
    │   ┌─────────────────────────────────────┐
    │   │         PhotoBrowser                  │
    │   │  ├── onPageShow()                    │
    │   │  ├── DataSource 初始化                │
    │   │  └── swipeToIndex()                  │
    │   └─────────────────────────────────────┘
    │
    ├── router.pushUrl('pages/PhotoGridPage')
    │       │
    │       ▼
    │   ┌─────────────────────────────────────┐
    │   │         PhotoGridPage                │
    │   │  ├── LazyForEach 渲染               │
    │   │  ├── onSelect()                     │
    │   │  └── navigateToBrowser()             │
    │   └─────────────────────────────────────┘
    │
    └── router.pushUrl('pages/NewAlbumPage')
            │
            ▼
        ┌─────────────────────────────────────┐
        │         NewAlbumPage                │
        │  ├── createAlbum()                  │
        │  └── addToAlbum()                   │
        └─────────────────────────────────────┘
```

---

## 5. 事件广播链

```
用户操作 (选择/删除/移动)
        │
        ▼
BroadCast.emit(event, data)
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                 BroadCastManager                            │
│                                                             │
│  ├── getInstance() ── 获取单例                              │
│  ├── getBroadCast() ── 获取广播实例                         │
│  ├── on(event, callback) ── 注册监听                        │
│  └── emit(event, data) ── 发送事件                          │
└─────────────────────────────────────────────────────────────┘
        │
        ▼
事件监听器回调
        │
        ├── BroadCastManager.on()
        ├── SelectManager.on()
        └── AlbumDataImpl.on()
```

---

## 6. 第三方选择回调链

```
第三方应用
    │
    ├── startAbilityForResult(uri=singleselect)
    │       │
    │       ▼
    │   ┌─────────────────────────────────────┐
    │   │         MainAbility                  │
    │   │  ├── parseWantParameter()           │
    │   │  └── thirdRouterPage()               │
    │   └─────────────────────────────────────┘
    │       │
    │       ▼
    │   ┌─────────────────────────────────────┐
    │   │    ThirdSelectPhotoGridPage         │
    │   │  ├── 加载媒体数据                    │
    │   │  ├── 用户选择图片                    │
    │   │  └── setResult()                    │
    │   └─────────────────────────────────────┘
    │       │
    ◄──────┘
返回结果
    │
    ▼
{ resultCode: 0, want: { parameters: { "select-item-list": [...] } } }
```

---

## 7. FA 卡片调用链

```
桌面系统
    │
    ├── 点击 FA 卡片 ──► FormAbility
    │       │
    │       ▼
    │   ┌─────────────────────────────────────┐
    │   │         FormAbility                  │
    │   │  ├── onCreate()                      │
    │   │  ├── onUpdate()                     │
    │   │  └── onVisibilityChange()           │
    │   └─────────────────────────────────────┘
    │       │
    │       ▼
    │   router.replaceUrl('pages/PhotoBrowser' 或 'FormEditorPage')
    │
    └── 长按 FA ──► FormEditorPage
            │
            ▼
        ┌─────────────────────────────────────┐
        │         FormEditorPage               │
        │  ├── onFormEvent()                   │
        │  └── 调用 editor 模块                │
        └─────────────────────────────────────┘
```
