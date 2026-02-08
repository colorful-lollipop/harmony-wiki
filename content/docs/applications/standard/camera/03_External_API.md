# 对外 API

## 说明

Camera 应用作为 ArkTS 纯应用，没有直接实现 N-API。所有"对外 API"指的是应用调用的 OpenHarmony 系统 SDK API。本文档梳理应用使用的系统能力、调用链路和权限要求。

## 系统能力清单

### 1. 多媒体能力

#### @ohos.multimedia.camera (相机核心)

| API/类 | 用途 | 代码位置 |
|--------|------|----------|
| `camera.getCameraManager()` | 获取相机管理器 | `CameraService.ts:75` |
| `CameraManager.getSupportedCameras()` | 获取可用相机 | `CameraService.ts:200` |
| `CameraManager.createCameraInput()` | 创建相机输入 | `CameraService.ts:210` |
| `CameraManager.createCaptureSession()` | 创建捕获会话 | `CameraService.ts:220` |
| `CameraManager.createPreviewOutput()` | 创建预览输出 | `CameraService.ts:230` |
| `CameraManager.createPhotoOutput()` | 创建拍照输出 | `CameraService.ts:240` |
| `CameraManager.createVideoOutput()` | 创建录像输出 | `CameraService.ts:250` |
| `CameraInput.open()` | 打开相机 | `CameraService.ts:260` |
| `CameraInput.close()` | 关闭相机 | `CameraService.ts:270` |
| `CaptureSession.beginConfig()` | 开始配置 | `CameraService.ts:280` |
| `CaptureSession.addInput()` | 添加输入 | `CameraService.ts:285` |
| `CaptureSession.addOutput()` | 添加输出 | `CameraService.ts:290` |
| `CaptureSession.commitConfig()` | 提交配置 | `CameraService.ts:295` |
| `CaptureSession.start()` | 开始捕获 | `CameraService.ts:300` |
| `CaptureSession.stop()` | 停止捕获 | `CameraService.ts:310` |
| `PhotoOutput.capture()` | 拍照 | `CameraService.ts:350` |
| `VideoOutput.start()` | 开始录像 | `CameraService.ts:400` |
| `VideoOutput.stop()` | 停止录像 | `CameraService.ts:410` |

**证据**: `common/src/main/ets/default/camera/CameraService.ts:17`
```typescript
import camera from '@ohos.multimedia.camera';
```

#### @ohos.multimedia.image (图片处理)

| API/类 | 用途 | 代码位置 |
|--------|------|----------|
| `image.createImageReceiver()` | 创建图片接收器 | `CameraService.ts:85` |
| `ImageReceiver.on('imageArrival')` | 监听图片到达 | `CameraService.ts:360` |
| `ImageReceiver.readNextImage()` | 读取下一张图片 | `CameraService.ts:365` |
| `Image.getComponent()` | 获取图像数据 | `CameraService.ts:370` |
| `image.createPixelMap()` | 创建像素图 | `ThumbnailGetter.ts` |

**证据**: `common/src/main/ets/default/camera/CameraService.ts:18`
```typescript
import image from '@ohos.multimedia.image';
```

#### @ohos.multimedia.media (媒体录制)

| API/类 | 用途 | 代码位置 |
|--------|------|----------|
| `media.createAVRecorder()` | 创建录像器 | `CameraService.ts:87` |
| `AVRecorder.prepare()` | 准备录制 | `CameraService.ts:390` |
| `AVRecorder.start()` | 开始录制 | `CameraService.ts:400` |
| `AVRecorder.stop()` | 停止录制 | `CameraService.ts:410` |
| `AVRecorder.release()` | 释放资源 | `CameraService.ts:420` |

**证据**: `common/src/main/ets/default/camera/CameraService.ts:19`
```typescript
import media from '@ohos.multimedia.media';
```

### 2. 应用框架能力

#### @ohos.app.ability.UIAbility

| 方法/事件 | 用途 | 代码位置 |
|-----------|------|----------|
| `onCreate()` | Ability 创建 | `MainAbility.ts:30` |
| `onDestroy()` | Ability 销毁 | `MainAbility.ts:54` |
| `onWindowStageCreate()` | 窗口创建 | `MainAbility.ts:63` |
| `onForeground()` | 进入前台 | `MainAbility.ts` |
| `onBackground()` | 进入后台 | `MainAbility.ts` |

**证据**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:27`
```typescript
export default class MainAbility extends Ability {
```

#### @ohos.window (窗口管理)

| API | 用途 | 代码位置 |
|-----|------|----------|
| `windowStage.getMainWindow()` | 获取主窗口 | `MainAbility.ts:78` |
| `Window.setLayoutFullScreen()` | 全屏显示 | `MainAbility.ts:80` |
| `Window.setSystemBarEnable()` | 系统栏控制 | `MainAbility.ts:82` |
| `Window.setSystemBarProperties()` | 系统栏属性 | `MainAbility.ts:83` |
| `windowStage.on('windowStageEvent')` | 窗口事件监听 | `MainAbility.ts:67` |

**证据**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:17`
```typescript
import window from '@ohos.window';
```

### 3. 系统信息能力

#### @ohos.deviceInfo

| API | 用途 | 代码位置 |
|-----|------|----------|
| `deviceInfo.deviceType` | 获取设备类型 | `CameraPlatformCapability.ts`, `FootBar.ets` |
| `deviceInfo.displayVersion` | 获取系统版本 | 多处使用 |

**证据**: `common/src/main/ets/default/camera/CameraService.ts:20`
```typescript
import deviceInfo from '@ohos.deviceInfo';
```

#### @ohos.display

| API | 用途 | 代码位置 |
|-----|------|----------|
| `display.getDefaultDisplay()` | 获取默认显示 | `index.ets` |
| `Display.getWidth()` | 屏幕宽度 | 布局计算 |
| `Display.getHeight()` | 屏幕高度 | 布局计算 |

### 4. 权限管理能力

#### @ohos.abilityAccessCtrl

| API | 用途 | 代码位置 |
|-----|------|----------|
| `abilityAccessCtrl.createAtManager()` | 创建权限管理器 | `index.ets:37` |
| `AtManager.requestPermissionsFromUser()` | 请求权限 | `index.ets:178` |
| `AtManager.checkAccessToken()` | 检查权限 | 权限检查 |

**证据**: `product/phone/src/main/ets/pages/index.ets:37`
```typescript
import abilityAccessCtrl, { Permissions } from '@ohos.abilityAccessCtrl';
```

### 5. 文件与存储能力

#### @ohos.fileshare

| API | 用途 | 代码位置 |
|-----|------|----------|
| `fileshare.share()` | 分享文件 | `ThirdPreviewView.ets:17` |

#### @ohos.data.preferences (轻量级存储)

| API | 用途 | 代码位置 |
|-----|------|----------|
| `preferences.getPreferences()` | 获取偏好存储 | `PreferencesService.ts` |
| `Preferences.put()` | 存储键值 | 设置保存 |
| `Preferences.get()` | 读取键值 | 设置读取 |
| `Preferences.flush()` | 持久化 | `MainAbility.ts:59` |

#### @ohos.data.relationalStore (关系型数据库)

| API | 用途 | 代码位置 |
|-----|------|----------|
| `relationalStore.getRdbStore()` | 获取 RDB 存储 | `RdbStoreManager.ts` |

### 6. 位置能力

#### @ohos.geoLocationManager

| API | 用途 | 代码位置 |
|-----|------|----------|
| `geoLocationManager.getCurrentLocation()` | 获取当前位置 | `GeoLocation.ts` |
| `Location.longitude` | 经度 | 照片地理标签 |
| `Location.latitude` | 纬度 | 照片地理标签 |

### 7. 路由与导航

#### @system.router / @ohos.router

| API | 用途 | 代码位置 |
|-----|------|----------|
| `router.push()` | 页面跳转 | `PreviewArea.ets:16` |
| `router.back()` | 返回上一页 | 多处使用 |

#### @ohos.ability.wantConstant

| 常量 | 用途 | 代码位置 |
|------|------|----------|
| `wantConstant.Action.ACTION_IMAGE_CAPTURE` | 拍照 Action | `MainAbility.ts:97` |
| `wantConstant.Action.ACTION_VIDEO_CAPTURE` | 录像 Action | `MainAbility.ts` |

### 8. 提示与交互

#### @ohos.promptAction

| API | 用途 | 代码位置 |
|-----|------|----------|
| `promptAction.showToast()` | 显示 Toast | `PreviewArea.ets:35` |

## 权限清单

### 已申请权限 (module.json5)

**完整权限列表** (13 项):

| 权限 | 权限级别 | 用途 | 代码位置 |
|------|----------|------|----------|
| `ohos.permission.CAMERA` | normal | 相机访问 | `module.json5:73-79` |
| `ohos.permission.MICROPHONE` | normal | 录音/录像 | `module.json5:82-88` |
| `ohos.permission.READ_IMAGEVIDEO` | user_grant | 读取照片/视频 | `module.json5:52-58` |
| `ohos.permission.WRITE_IMAGEVIDEO` | user_grant | 保存照片/视频 | `module.json5:61-67` |
| `ohos.permission.LOCATION` | user_grant | 精确定位(地理标签) | `module.json5:100-106` |
| `ohos.permission.LOCATION_IN_BACKGROUND` | user_grant | 后台定位 | `module.json5:109-115` |
| `ohos.permission.APPROXIMATELY_LOCATION` | user_grant | 模糊定位 | `module.json5:118-124` |
| `ohos.permission.MEDIA_LOCATION` | user_grant | 媒体位置信息 | `module.json5:36-43` |
| `ohos.permission.DISTRIBUTED_DATASYNC` | user_grant | 分布式数据同步 | `module.json5:91-97` |
| `ohos.permission.ACCESS_SERVICE_DM` | system_grant | 设备管理服务 | `module.json5:127` |
| `ohos.permission.PROXY_AUTHORIZATION_URI` | system_grant | URI 代理授权 | `module.json5:130` |
| `ohos.permission.INTERNET` | normal | 网络访问 | `module.json5:46` |
| `ohos.permission.MODIFY_AUDIO_SETTINGS` | normal | 修改音频设置 | `module.json5:49` |

**证据**: `product/phone/src/main/module.json5:35-131`

### 权限分类

#### 用户授权权限 (user_grant) - 需要弹窗申请
1. `ohos.permission.READ_IMAGEVIDEO`
2. `ohos.permission.WRITE_IMAGEVIDEO`
3. `ohos.permission.LOCATION`
4. `ohos.permission.LOCATION_IN_BACKGROUND`
5. `ohos.permission.APPROXIMATELY_LOCATION`
6. `ohos.permission.MEDIA_LOCATION`
7. `ohos.permission.DISTRIBUTED_DATASYNC`

#### 系统授权权限 (system_grant) - 安装时自动授予
1. `ohos.permission.ACCESS_SERVICE_DM`
2. `ohos.permission.PROXY_AUTHORIZATION_URI`

#### 普通权限 (normal) - 安装时自动授予
1. `ohos.permission.CAMERA`
2. `ohos.permission.MICROPHONE`
3. `ohos.permission.INTERNET`
4. `ohos.permission.MODIFY_AUDIO_SETTINGS`

### 权限申请时序

```
页面显示 (onPageShow)
    │
    ▼
检查权限状态
    │
    ├─ 已授权 ──> 初始化相机
    │
    └─ 未授权 ──> 请求权限
                      │
                      ▼
                 显示系统弹窗
                      │
            ┌────────┴────────┐
            ▼                 ▼
        用户允许           用户拒绝
            │                 │
            ▼                 ▼
        初始化相机         显示错误提示
```

**代码证据**: `product/phone/src/main/ets/pages/index.ets:170-200`

## 错误码与异常处理

### 系统 API 错误码

#### 相机 API 错误码

| 错误码 | 说明 | 处理策略 |
|--------|------|----------|
| 7400101 | 相机不存在 | 提示用户相机不可用 |
| 7400102 | 相机已占用 | 等待或提示关闭其他应用 |
| 7400103 | 相机设备错误 | 重试或退出 |
| 7400104 | 会话未配置 | 检查配置流程 |
| 7400105 | 会话配置失败 | 检查参数合法性 |
| 7400201 | 参数错误 | 检查输入参数 |

#### 权限错误码

| 错误码 | 说明 | 处理策略 |
|--------|------|----------|
| 201 | 权限拒绝 | 引导用户开启权限 |
| 202 | 权限未申请 | 申请权限 |

### 应用层错误处理

**证据**: `common/src/main/ets/default/camera/CameraService.ts`

```typescript
try {
  await this.mCaptureSession.commitConfig();
} catch (err) {
  Log.error(`${TAG} commitConfig error: ${JSON.stringify(err)}`);
  // 错误处理逻辑
}
```

**证据**: `product/phone/src/main/ets/pages/PreviewArea.ets:38`

```typescript
import { BusinessError } from '@ohos.base';

// 使用 BusinessError 类型捕获系统错误
catch (error: BusinessError) {
  Log.error(`${this.TAG} error: ${JSON.stringify(error)}`);
}
```

### 全局错误处理策略

1. **相机操作错误**: 记录日志 + 回调通知 UI
2. **权限错误**: 提示用户 + 引导设置
3. **存储错误**: 重试机制 + 错误提示
4. **网络错误**: 忽略或延迟重试

## API 调用链示例

### 拍照完整调用链

```
1. UI 层: FootBar.ets
   └─ ShutterButton.onClick()

2. 功能层: CaptureFunction.ts
   └─ capture()

3. 服务层: CameraService.ts
   ├─ capturePhoto()
   ├─ mPhotoOutPut.capture()
   ├─ mImageReceiver.on('imageArrival', callback)
   └─ onCaptureSuccess()

4. 保存层: SaveCameraAsset.ts
   └─ saveImage()

5. 系统 API: @ohos.multimedia.camera
   ├─ PhotoOutput.capture()
   └─ PhotoOutput.off('captureStart')

6. 系统 API: @ohos.multimedia.image
   ├─ ImageReceiver.readNextImage()
   └─ Image.getComponent()
```

### 预览启动调用链

```
1. UI 层: PreviewArea.ets
   └─ XComponent.onLoad()

2. 服务层: CameraService.ts
   ├─ initCamera()
   ├─ createCameraManager()
   ├─ createCaptureSession()
   ├─ createPreviewOutput()
   ├─ session.addInput()
   ├─ session.addOutput()
   ├─ session.commitConfig()
   ├─ cameraInput.open()
   └─ session.start()

3. 系统 API: @ohos.multimedia.camera
   ├─ camera.getCameraManager()
   ├─ CameraManager.createCaptureSession()
   ├─ CameraManager.createPreviewOutput()
   ├─ CaptureSession.addInput()
   ├─ CaptureSession.addOutput()
   ├─ CaptureSession.commitConfig()
   ├─ CameraInput.open()
   └─ CaptureSession.start()
```

## 外部调用入口

### Ability 入口

| Ability | 类型 | 入口 Action | 用途 |
|---------|------|-------------|------|
| MainAbility | UIAbility | action.system.home | 主入口 |
| MainAbility | UIAbility | ohos.want.action.imageCapture | 第三方拍照 |
| MainAbility | UIAbility | ohos.want.action.videoCapture | 第三方录像 |
| ExtensionPickerAbility | ExtensionAbility | sysPicker/camera | 系统选择器 |
| FormAbility | FormAbility | form | 桌面卡片 |

**证据**: `product/phone/src/main/module.json5:133-218`

### URI Scheme

| Scheme | 用途 |
|--------|------|
| `file://` | 文件处理 |
| `ability://com.ohos.camera.MainAbility` | 卡片跳转 |

## 兼容性说明

### API 版本
- **compileSdkVersion**: 23
- **compatibleSdkVersion**: 23
- **最低系统版本**: OpenHarmony 4.1+

### 设备适配
- **手机**: 竖屏布局，底部控制栏
- **平板**: 横屏布局，侧边控制栏

**证据**: `build-profile.json5:22-24`
