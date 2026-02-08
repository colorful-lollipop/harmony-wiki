# 架构说明

## 整体架构

```mermaid
graph TD
    subgraph 用户层
        User[用户触发截屏]
        Gallery[相册应用]
    end

    subgraph 应用层
        ServiceExtAbility[ServiceExtAbility<br/>截屏服务入口]
        DialogAbility[DialogAbility<br/>隐私对话框]
        Index[Index 页面<br/>预览]
        ViewModel[ViewModel<br/>页面逻辑]
        ScreenShotModel[ScreenShotModel<br/>核心逻辑]
    end

    subgraph 系统框架层
        ScreenshotAPI[@ohos.screenshot]
        WindowAPI[@ohos.window]
        MediaAPI[@ohos.multimedia.image]
        FileAPI[@ohos.filemanagement]
        UIExtAPI[@ohos.app.ability]
    end

    subgraph Native层
        ScreenshotService[截屏服务]
        WindowManager[窗口管理]
        FileSystem[文件系统]
    end

    User -->|"TOGGLE Action"| ServiceExtAbility
    ServiceExtAbility --> WindowAPI
    ServiceExtAbility --> Index
    Index --> ViewModel
    ViewModel --> ScreenShotModel
    ScreenShotModel --> ScreenshotAPI
    ScreenShotModel --> WindowAPI
    ScreenShotModel --> MediaAPI
    ScreenShotModel --> FileAPI
    ScreenShotModel --> Gallery
    DialogAbility --> UIExtAPI
    ScreenshotAPI --> ScreenshotService
    WindowAPI --> WindowManager
    FileAPI --> FileSystem
```

## 模块依赖关系

```mermaid
graph LR
    subgraph product/phone [entry 模块]
        A[ServiceExtAbility]
        B[DialogAbility]
        C[Index]
        D[ViewModel]
        E[AbilityStage]
    end

    subgraph features/screenshot [har 模块]
        F[ScreenShotModel]
        G[Constants]
    end

    subgraph common [har 模块]
        H[Log]
    end

    A --> F
    A --> G
    A --> H
    B --> H
    C --> D
    C --> H
    D --> F
    D --> H
    F --> H
    E --> H
```

**证据**: 代码导入语句分析
- `product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ets:21`
- `product/phone/src/main/ets/vm/ViewModel.ets:17`

## 启动流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant SystemUI as System UI
    participant Service as ServiceExtAbility
    participant Window as 窗口管理
    participant Index as Index 页面
    participant Model as ScreenShotModel
    participant Screenshot as @ohos.screenshot

    User->>SystemUI: 触发截屏
    SystemUI->>Service: startAbility(TOGGLE)

    Note over Service: onCreate()
    Service->>Window: createWindow(TYPE_SCREENSHOT)
    Window-->>Service: Window 实例

    Service->>Window: moveWindowTo(0, 300)
    Service->>Window: resize(40% 尺寸)
    Service->>Window: setUIContent(pages/index)

    Note over Index: 页面加载完成
    Index->>Model: shotScreen()

    Model->>Screenshot: save()
    Screenshot-->>Model: PixelMap

    Model->>Model: saveImage(PixelMap)
    Model->>Model: 显示预览窗口

    Note over Index: 5秒后自动关闭
    Model->>Window: destroy()
```

## 核心数据流

### 截图保存流程

```mermaid
flowchart TD
    A[shotScreen 调用] --> B[ScreenshotManager.save]
    B --> C[获取 PixelMap]
    C --> D{成功?}
    D -->|是| E[保存到 AppStorage]
    D -->|否| F[通知截屏中止]
    E --> G[createImagePacker]
    G --> H[packing 打包 JPEG]
    H --> I[createPhotoAsset]
    I --> J[打开文件描述符]
    J --> K[写入图片数据]
    K --> L[fsync 同步]
    L --> M[关闭文件]
    M --> N[设置图片 URI]
```

**证据**: `features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:40-98`

## 线程模型

### 线程分布

| 组件 | 运行线程 | 说明 |
|-----|---------|------|
| ServiceExtensionAbility | 主线程 | UI 线程，处理生命周期 |
| UIExtensionAbility | 主线程 | 对话框 UI 线程 |
| Index 页面 | UI 线程 | ArkUI 渲染 |
| ScreenShotModel | UI 线程 | 业务逻辑（异步） |

### 异步操作

所有系统 API 调用均使用 Promise/async-await 模式：

```typescript
// 文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:44
ScreenshotManager.save().then(async (data) => {
    // 异步回调
})
```

## 生命周期

### ServiceExtAbility 生命周期

```mermaid
stateDiagram-v2
    [*] --> onCreate: 系统启动
    onCreate --> [*]: 能力销毁

    state onCreate {
        创建窗口
        配置窗口位置/大小
        加载 UI 内容
        触发截屏
    }
```

**证据**: `product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ets:31-60`

### 隐私对话框生命周期

```mermaid
stateDiagram-v2
    [*] --> onCreate
    onCreate --> onForeground
    onForeground --> onBackground
    onBackground --> onDestroy
    onDestroy --> [*]

    onCreate --> onSessionCreate: UIExtension 创建
    onSessionCreate --> onSessionDestroy
    onSessionDestroy --> onDestroy
```

**证据**: `product/phone/src/main/ets/PrivacyDialog/DialogAbility.ets:31-75`
