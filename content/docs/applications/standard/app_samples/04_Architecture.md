# 架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────┐
│                   应用层 (ArkTS)                      │
│  ┌─────────────────────────────────────────────┐   │
│  │              页面组件 (Pages)                │   │
│  │  ├── Index.ets (首页)                       │   │
│  │  ├── FeaturePage/                          │   │
│  │  │   ├── *.ets (主页面)                    │   │
│  │  │   └── components/ (子组件)              │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│               入口层 (EntryAbility)                  │
│  • 应用生命周期管理                                 │
│  • 全局上下文 (Context)                            │
└─────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│              OpenHarmony SDK 层                      │
│  • @kit.AbilityKit (能力)                          │
│  • @kit.ArkUI (UI组件)                             │
│  • @kit.BasicServicesKit (基础服务)                 │
│  • @kit.CoreFileKit (文件操作)                      │
└─────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│              系统能力 (SA / Native)                  │
│  • 文件系统                                         │
│  • 相机能力                                         │
│  • 网络能力                                         │
└─────────────────────────────────────────────────────┘
```

## Stage 模型架构

Stage 模型是 OpenHarmony 主流的应用模型，核心组件包括：

### 1. Ability

```
Ability
├── UIAbility (界面Ability)
│   ├── onCreate()      - 创建时调用
│   ├── onWindowStageCreate() - 窗口创建
│   ├── onForeground()  - 切换到前台
│   └── onBackground()  - 切换到后台
└── ServiceExtensionAbility (服务扩展)
```

### 2. 组件结构

```
┌─────────────────────────────────────────┐
│            Page (页面)                   │
│  ├── @Component                         │
│  ├── @Entry                             │
│  └── @Builder                           │
├─────────────────────────────────────────┤
│            自定义组件                    │
│  ├── @Component                         │
│  ├── @Prop / @Link / @State             │
│  └── build()                            │
└─────────────────────────────────────────┘
```

## 组件通信模式

### 1. 页面间跳转

```typescript
// 使用 router 进行页面跳转
import { router } from '@kit.ArkUI';

router.pushUrl({
  url: 'pages/FeaturePage/FeaturePage'
});
```

### 2. 数据传递

```typescript
// 通过参数传递
router.pushUrl({
  url: 'pages/Detail/Detail',
  params: {
    id: 123,
    name: 'example'
  }
});

// 在目标页面获取
const params = router.getParams();
```

### 3. 组件间数据同步

```typescript
// 使用 @Link 实现双向绑定
@Component
struct ChildComponent {
  @Link parentValue: string;

  build() {
    Button('Click')
      .onClick(() => {
        this.parentValue = 'updated';
      })
  }
}
```

## 常见架构模式

### 1. MVVM 模式

```
┌─────────────────────────────────────────┐
│              View (UI)                  │
│  • .ets 页面文件                        │
│  • 接收数据渲染                          │
└────────────────┬────────────────────────┘
                 │ 数据绑定
                 ▼
┌─────────────────────────────────────────┐
│           ViewModel                      │
│  • 状态管理 (@State, @Link)             │
│  • 业务逻辑处理                          │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│              Model (数据)                │
│  • 数据定义                              │
│  • 本地存储                              │
│  • 网络请求                              │
└─────────────────────────────────────────┘
```

### 2. 组件化模式

```
┌─────────────────────────────────────────┐
│            页面组件                      │
│  ├── 自定义子组件 (components/)         │
│  │   ├── ButtonComponent.ets            │
│  │   └── ListComponent.ets              │
│  └── 视图组件 (view/)                   │
│      ├── HeaderView.ets                 │
│      └── ContentView.ets                │
└─────────────────────────────────────────┘
```

## 资源管理架构

```
resources/
├── base/                    # 默认资源
│   ├── element/            # 元素资源
│   │   ├── color.json      # 颜色
│   │   ├── string.json     # 字符串
│   │   ├── float.json      # 尺寸
│   │   └── integer.json    # 整数
│   ├── media/              # 图片资源
│   └── profile/            # 配置
│       ├── main_pages.json # 页面路由
│       └── backup_config.json
├── [locale]/               # 国际化资源
│   └── element/
└── dark/                   # 深色主题
    └── element/
        └── color.json
```

## 生命周期管理

### 应用生命周期

```
onCreate()           ──────────────►  应用创建
    │
    ▼
onForeground()       ──────────────►  切换到前台
    │
    ▼
onBackground()       ──────────────►  切换到后台
    │
    ▼
onTerminate()        ──────────────►  应用终止
```

### 组件生命周期

```
aboutToAppear()      ──────────────►  组件即将显示
    │
    ▼
build()              ──────────────►  构建UI
    │
    ▼
aboutToDisappear()   ──────────────►  组件即将销毁
```
