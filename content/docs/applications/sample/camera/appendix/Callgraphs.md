# 关键调用链

## 相机拍照调用链

### 入口 -> 拍照完成

```
CameraAbilitySlice::OnStart()
├── SampleCameraManager* cam_manager = new SampleCameraManager()
└── cam_manager->SampleCameraCreate()
    ├── CameraKit* camKit = CameraKit::GetInstance()
    ├── camKit->GetCameraIds()                    [获取相机列表]
    ├── new SampleCameraStateMng(eventHdlr_)
    └── camKit->CreateCamera(camId, *CamStateMng, eventHdlr_)
        └── SampleCameraStateMng::OnCreated()     [相机创建回调]
            └── StartPreview(surface)             [启动预览]

[用户点击拍照按钮]
CameraImageButtonOnClickListener::OnClick()
└── cManager_->SampleCameraCaptrue(0)
    └── SampleCameraStateMng::Capture(0)
        ├── cam_->TriggerSingleCapture(*fc_)     [触发拍照]
        └── [异步] TestFrameStateCallback::OnFrameFinished()
            ├── fwrite() -> /userdata/photo/photo{timestamp}.jpg  [保存照片]
            └── GenerateThumb() -> /userdata/photo/thumb/...       [生成缩略图]
```

**代码位置**: `cameraApp/cameraApp/src/main/cpp/camera_manager.cpp:521-547`

---

## 视频录制调用链

```
[用户点击录像按钮]
CameraImageButtonOnClickListener::OnClick()
└── cManager_->SampleCameraStartRecord(surface)
    └── SampleCameraStateMng::StartRecord(surface)
        ├── recorder_ = new Recorder()
        ├── recorder_->SetVideoSource(VideoSourceType::CAMERA)
        ├── recorder_->SetOutputPath(videoName_)
        ├── recorder_->Prepare()
        ├── recorder_->Start()                     [开始录制]
        └── gRecordSta_ = 1

[录制过程中]
SampleCameraStateMng::OnFrameFinished()
└── [录制线程] -> 写入视频文件

[用户点击停止]
CameraImageButtonOnClickListener::OnClick()
└── cManager_->SampleCameraStopRecord()
    ├── recorder_->Stop()
    └── recorder_->Release()
```

**代码位置**: `cameraApp/cameraApp/src/main/cpp/camera_manager.cpp:434-459`

---

## 权限管理调用链

```
[setting 应用]
MainAbilitySlice::OnClick() [应用列表点击]
└── StartAbility(want) with bundleName
    └── AppInfoAbilitySlice::OnStart(want)
        ├── memcpy_s(bundleName_, want.data, want.dataLength)
        └── PermissionInfoList()
            ├── QueryPermission(bundleName_, &permissions_, &permNum)
            │   └── [IPC] -> permission_lite 服务
            └── for each permission:
                └── SetAppPermissionInfo(i, permissions_[i])
                    └── 显示权限开关

[用户切换权限开关]
ToggBtnOnListener::OnClick()
├── if (togglebutton_->GetState()):
│   └── RevokePermission(bundleName_, name_)    [关闭权限]
└── else:
    └── GrantPermission(bundleName_, name_)     [开启权限]
        └── [IPC] -> permission_lite 服务
```

**代码位置**: `setting/setting/src/main/cpp/app_info_ability_slice.cpp:118-156`

---

## 图库浏览调用链

```
GalleryAbilitySlice::OnStart()
└── AddAllPictures()
    ├── opendir(PHOTO_DIRECTORY)                [/userdata/photo/]
    ├── readdir() for each file
    └── AddPictureItem() for each image
        ├── 创建 UIImageView
        ├── 加载缩略图
        └── 绑定点击事件

[用户点击图片]
GalleryAbilitySlice::OnPictureClick()
├── want.data = imageName
└── StartAbility(want)
    └── PictureAbilitySlice::OnStart(want)
        ├── sprintf_s(imagePath, "%s/%s", PHOTO_DIRECTORY, want.data)
        └── 显示大图

[用户点击视频]
GalleryAbilitySlice::OnVideoClick()
└── StartAbility(want)
    └── PlayerAbilitySlice::OnStart(want)
        ├── videoPlayer_ = new Player()
        ├── videoPlayer_->SetSource(videoPath)
        ├── videoPlayer_->Prepare()
        └── videoPlayer_->Start()
```

**代码位置**: `gallery/src/gallery_ability_slice.cpp:200-230`

---

## WiFi 配置调用链

```
SettingWifiAbilitySlice::OnStart()
└── 扫描 WiFi 网络
    └── 显示 SSID 列表

[用户选择 WiFi]
SettingWifiAbilitySlice::OnWifiClick()
├── if (加密网络):
│   └── StartAbility(want) -> SettingWifiInputPasswordAbilitySlice
│       └── [用户输入密码]
│           └── ConnectWifi(ssid, password)
└── else:
    └── ConnectWifi(ssid, nullptr)

ConnectWifi()
└── WpaWork::ConnectNetwork()
    ├── sprintf_s(cmd, "SET_NETWORK %s ssid \"%s\"", networkId, ssid)
    ├── sprintf_s(cmd, "SET_NETWORK %s psk \"%s\"", networkId, password)
    └── 发送命令到 wpa_supplicant
```

**代码位置**: `setting/setting/src/main/cpp/wpa_work.c:381-397`

---

## 应用启动调用链

```
Launcher::MainAbilitySlice::OnStart()
└── LoadAppList()
    ├── GetBundleManager()->QueryBundleInfos()
    │   └── [IPC] -> bundle_lite 服务
    └── for each bundleInfo:
        └── AddAppIcon()
            ├── 加载应用图标
        └── 绑定点击事件

[用户点击应用图标]
AppIconClickListener::OnClick()
└── AppManage::StartAbility()
    ├── Want want
    ├── want.SetBundleName(appInfo.bundleName_)
    ├── want.SetAbilityName(appInfo.abilityName_)
    └── AbilityManager::StartAbility(want)
        └── [IPC] -> ability_lite 服务 -> 启动目标应用
```

**代码位置**: `launcher/launcher/src/main/cpp/main_ability_slice.cpp`

---

## 关键回调链

### 相机状态回调

```
SampleCameraStateMng (CameraStateCallback)
├── OnCreated(Camera &c)        [相机就绪]
├── OnCreateFailed(...)         [相机创建失败]
└── OnReleased(Camera &c)       [相机关闭]
```

### 帧捕获回调

```
TestFrameStateCallback (FrameStateCallback)
└── OnFrameFinished(Camera&, FrameConfig&, FrameResult&)
    ├── if (PHOTO_TYPE_NORMAL):
    │   └── 保存 JPEG
    ├── if (PHOTO_TYPE_VIDEO):
    │   └── 保存视频帧
    └── if (PHOTO_TYPE_TMP):
        └── 保存临时帧
```

### UI 事件回调

```
EventListener (OnClickListener)
├── CameraImageButtonOnClickListener  [相机按钮]
├── ToggBtnOnListener                 [权限开关]
└── AppIconClickListener              [应用图标]
```

---

## 相关链接

- [架构说明](./Architecture.md) - 整体架构
- [内部接口](./Internal_API.md) - 接口详情
