# 架构说明

## 组件图

### 整体架构

```mermaid
graph TB
    subgraph "应用层 (本仓库)"
        CAM[cameraApp<br/>相机应用]
        GAL[gallery<br/>图库应用]
        LAU[launcher<br/>桌面启动器]
        SET[setting<br/>设置应用]
        MED[media<br/>媒体示例]
    end
    
    subgraph "Ability 框架层"
        ABL[ability_lite<br/>Ability 生命周期管理]
        BND[bundle_lite<br/>包管理]
    end
    
    subgraph "UI 框架层"
        UI[ui_lite<br/>图形界面框架]
        SUR[surface_lite<br/>图形缓冲区]
    end
    
    subgraph "媒体服务层"
        CAM_S[camera_lite<br/>相机服务]
        REC[recorder_lite<br/>录像服务]
        PLY[player_lite<br/>播放服务]
        AUD[audio_lite<br/>音频服务]
    end
    
    subgraph "系统服务层"
        SAM[samgr_lite<br/>系统服务管理]
        PER[permission_lite<br/>权限管理]
        KV[kv_store<br/>键值存储]
    end
    
    CAM --> UI
    CAM --> SUR
    CAM --> CAM_S
    CAM --> REC
    CAM --> ABL
    CAM --> SAM
    
    GAL --> UI
    GAL --> SUR
    GAL --> PLY
    GAL --> REC
    GAL --> ABL
    GAL --> SAM
    
    LAU --> UI
    LAU --> SUR
    LAU --> BND
    LAU --> ABL
    LAU --> SAM
    LAU --> KV
    
    SET --> UI
    SET --> SUR
    SET --> PER
    SET --> ABL
    SET --> SAM
    SET --> KV
    
    MED --> CAM_S
    MED --> REC
    MED --> PLY
    MED --> AUD
```

---

## 数据流

### 相机拍照数据流

```mermaid
sequenceDiagram
    participant User as 用户
    participant UI as CameraAbilitySlice
    participant CM as SampleCameraManager
    participant CK as CameraKit
    participant CS as CameraService
    participant FS as 文件系统
    
    User->>UI: 点击拍照按钮
    UI->>CM: SampleCameraCaptrue(type)
    CM->>CK: CreateCamera()
    CK->>CS: 初始化相机
    CS-->>CM: OnCreated 回调
    CM->>CS: TriggerSingleCapture()
    CS->>CS: 图像处理
    CS->>CM: OnFrameFinished 回调
    CM->>CM: TestFrameStateCallback::OnFrameFinished
    CM->>FS: fwrite() 保存 JPEG
    FS-->>CM: 完成
    CM-->>UI: 返回结果
    UI-->>User: 显示预览/缩略图
```

### 视频录制数据流

```mermaid
sequenceDiagram
    participant User as 用户
    participant UI as CameraAbilitySlice
    participant CM as SampleCameraManager
    participant CK as CameraKit
    participant REC as Recorder
    participant FS as 文件系统
    
    User->>UI: 点击录像按钮
    UI->>CM: SampleCameraStartRecord()
    CM->>CK: CreateCamera()
    CM->>REC: SetVideoSource()
    CM->>REC: SetOutputPath()
    CM->>REC: Prepare()
    CM->>REC: Start()
    
    loop 录制过程中
        CK->>CM: 视频帧回调
        CM->>REC: WriteSample()
        REC->>FS: 写入视频文件
    end
    
    User->>UI: 点击停止
    UI->>CM: SampleCameraStopRecord()
    CM->>REC: Stop()
    CM->>REC: Release()
    REC-->>CM: 完成
```

### 图库图片浏览数据流

```mermaid
sequenceDiagram
    participant User as 用户
    participant GAS as GalleryAbilitySlice
    participant PAS as PictureAbilitySlice
    participant FS as 文件系统
    
    User->>GAS: 进入图库
    GAS->>FS: opendir(/userdata/photo/)
    FS-->>GAS: 文件列表
    GAS->>GAS: 生成缩略图
    GAS-->>User: 显示缩略图网格
    
    User->>GAS: 点击图片
    GAS->>PAS: 跳转（Want 传递文件名）
    PAS->>FS: fopen(图片文件)
    FS-->>PAS: 图片数据
    PAS->>PAS: 解码/显示
    PAS-->>User: 显示大图
```

---

## 线程模型

### cameraApp 线程模型

```
主线程 (UI 线程)
├── Ability/AbilitySlice 生命周期回调
├── 用户事件处理（点击、滑动）
└── UI 渲染

工作线程 1 (EventHandler)
├── CameraStateCallback 回调
│   ├── OnCreated
│   ├── OnCreateFailed
│   └── OnReleased
├── FrameStateCallback 回调
│   └── OnFrameFinished (拍照完成)
└── Recorder 状态回调

工作线程 2 (录制线程，由 Recorder 内部创建)
├── 视频编码
├── 音频采集（录音时）
└── 文件写入
```

**关键代码** (`camera_manager.h:70`):
```cpp
SampleCameraStateMng(EventHandler &eventHdlr) : eventHdlr_(eventHdlr)
```

**设计说明**:
- 所有相机回调均在 EventHandler 线程执行，避免阻塞 UI
- 视频编码在独立线程，由 media_lite 库内部管理

---

### gallery 线程模型

```
主线程 (UI 线程)
├── AbilitySlice 生命周期
├── 用户交互处理
└── 图片/视频解码渲染

后台线程 (媒体播放)
├── 视频解码
├── 音频播放
└── 进度更新回调
```

**关键代码** (`player_ability_slice.cpp`):
```cpp
// 播放进度回调（在播放器内部线程）
void PlayerAbilitySlice::UpdatePlayTime()
```

---

### launcher 线程模型

```
主线程 (UI 线程)
├── 桌面渲染
├── 应用图标点击处理
├── 时间/日期更新（定时器）
└── 应用启动

后台线程 (BundleManager)
├── 应用信息查询
├── 安装/卸载监听
└── 应用列表更新
```

---

## 关键时序

### 相机应用生命周期

```mermaid
sequenceDiagram
    participant AM as AbilityManager
    participant CA as CameraAbility
    participant CAS as CameraAbilitySlice
    participant CM as SampleCameraManager
    
    AM->>CA: OnStart()
    CA->>CAS: SetMainRoute()
    CAS->>CAS: OnStart()
    CAS->>CM: new SampleCameraManager()
    CAS->>CM: SampleCameraCreate()
    CAS-->>AM: 启动完成
    
    AM->>CAS: OnActive()
    CAS->>CM: SampleCameraStart()
    CAS->>CM: SampleCameraStartRecord() [录像模式]
    
    Note over CAS: 运行中...
    
    AM->>CAS: OnBackground()
    CAS->>CM: SampleCameraStop()
    CAS->>CM: SampleCameraStopRecord()
    
    AM->>CAS: OnStop()
    CAS->>CM: delete
    CAS-->>AM: 销毁完成
```

### 权限管理时序

```mermaid
sequenceDiagram
    participant UI as AppInfoAbilitySlice
    participant PM as PermissionManager
    participant U as 用户
    
    UI->>PM: QueryPermission(bundleName, &permissions, &permNum)
    PM-->>UI: 权限列表
    
    UI->>UI: 显示权限开关列表
    
    alt 用户开启权限
        U->>UI: 点击开关
        UI->>PM: GrantPermission(bundleName, permissionName)
        PM-->>UI: 结果
    else 用户关闭权限
        U->>UI: 点击开关
        UI->>PM: RevokePermission(bundleName, permissionName)
        PM-->>UI: 结果
    end
```

---

## 模块依赖关系

### 依赖图（无环验证）

```mermaid
graph LR
    CAM[cameraApp] --> UI[ui_lite]
    CAM --> CAM_L[camera_lite]
    CAM --> REC[recorder_lite]
    
    GAL[gallery] --> UI
    GAL --> PLY[player_lite]
    GAL --> REC
    
    LAU[launcher] --> UI
    LAU --> BND[bundle_lite]
    
    SET[setting] --> UI
    SET --> PER[permission_lite]
    
    UI --> SUR[surface_lite]
    UI --> G_UT[graphic_utils_lite]
    
    CAM_L --> SUR
    REC --> SUR
    PLY --> SUR
```

**结论**: 依赖关系为 DAG（有向无环图），无循环依赖。

---

## 关键设计模式

### 1. 回调模式 (Callback)
用于异步事件处理：
```cpp
class SampleCameraStateMng : public CameraStateCallback {
    void OnCreated(Camera &c) override;
    void OnCreateFailed(const std::string cameraId, int32_t errorCode) override;
    void OnReleased(Camera &c) override;
};
```

### 2. 单例模式 (Singleton)
全局唯一实例管理：
```cpp
// CameraKit 为单例
CameraKit *camKit = CameraKit::GetInstance();
```

### 3. 监听器模式 (Listener)
UI 事件处理：
```cpp
class EventListener : public UIView::OnClickListener {
    bool OnClick(UIView &view, const Event &event) override;
};
```

### 4. Ability 框架模式
OpenHarmony 应用标准架构：
- Ability: 应用入口，管理生命周期
- AbilitySlice: 页面管理，处理 UI
- Want: 组件间通信

---

## 相关链接

- [目录结构](./Structure.md) - 代码文件组织
- [对外接口](./External_API.md) - Ability 和权限接口
- [内部接口](./Internal_API.md) - 模块间接口
- [GN 构建系统](./Build_System.md) - 构建配置
