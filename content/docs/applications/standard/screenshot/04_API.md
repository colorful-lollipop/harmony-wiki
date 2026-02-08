# API 分析

## 概述

本项目**不包含自定义 N-API 模块**，所有功能通过调用 OpenHarmony 系统框架 API 实现。本章节分析项目调用的系统 API。

## 系统 API 调用清单

### 1. @ohos.screenshot

**用途**: 截屏操作

| API | 功能 | 调用位置 | 行号 |
|-----|------|---------|------|
| `ScreenshotManager.save()` | 执行截屏并返回图片 | screenShotModel.ets | :44 |

**调用示例**:
```typescript
import ScreenshotManager from '@ohos.screenshot';

ScreenshotManager.save().then(async (data) => {
    if (!!data) {
        this.captureImage = data;
        this.saveImage(data, { format: 'image/jpeg', quality: 100 });
    }
});
```

**证据**: `features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:17,44`

**参数与返回值**:

| 参数 | 类型 | 说明 |
|-----|------|------|
| 无 | - | - |

| 返回值 | 类型 | 说明 |
|-------|------|------|
| PixelMap | `ImageMar.PixelMap` | 截屏图片数据 |

---

### 2. @ohos.window

**用途**: 窗口管理

| API | 功能 | 调用位置 | 行号 |
|-----|------|---------|------|
| `windowManager.createWindow()` | 创建窗口 | ServiceExtAbility.ets | :39 |
| `windowManager.find()` | 查找窗口 | screenShotModel.ets | :102 |
| `window.moveWindowTo()` | 移动窗口 | ServiceExtAbility.ets | :41 |
| `window.resize()` | 调整大小 | ServiceExtAbility.ets | :46 |
| `window.setUIContent()` | 设置 UI 内容 | ServiceExtAbility.ets | :48 |
| `window.show()` | 显示窗口 | screenShotModel.ets | :103 |
| `window.destroy()` | 销毁窗口 | screenShotModel.ets | :115 |
| `window.notifyScreenshotEvent()` | 通知截屏事件 | screenShotModel.ets | :42,54,57 |

**调用示例**:
```typescript
import windowManager from '@ohos.window';

const windowConfig: windowManager.Configuration = {
    name: Constants.WIN_NAME,
    windowType: windowManager.WindowType.TYPE_SCREENSHOT,
    ctx: this.context
};

windowManager.createWindow(windowConfig).then((win) => {
    win.moveWindowTo(0, WINDOW_Y);
    win.resize(dis.width * ZOOM_RATIO, dis.height * ZOOM_RATIO);
    win.setUIContent(INDEX_PAGE);
});
```

**证据**: `product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ets:17,34-50`

**窗口类型**:
```typescript
windowManager.WindowType.TYPE_SCREENSHOT
```

---

### 3. @ohos.multimedia.image

**用途**: 图片处理

| API | 功能 | 调用位置 | 行号 |
|-----|------|---------|------|
| `ImageMar.createImagePacker()` | 创建图片打包器 | screenShotModel.ets | :72 |
| `packer.packing()` | 打包像素图 | screenShotModel.ets | :75 |

**调用示例**:
```typescript
import ImageMar from '@ohos.multimedia.image';

const packer = ImageMar.createImagePacker();
const packedImg = await packer.packing(pixelMap, {
    format: 'image/jpeg',
    quality: 100
});
```

**证据**: `features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:19,72-75`

---

### 4. @ohos.filemanagement.userFileManager

**用途**: 文件管理

| API | 功能 | 调用位置 | 行号 |
|-----|------|---------|------|
| `userFileManager.getUserFileMgr()` | 获取用户文件管理器 | screenShotModel.ets | :68 |
| `userFileMgr.createPhotoAsset()` | 创建相册资源 | screenShotModel.ets | :80 |
| `fileAsset.open()` | 打开文件 | screenShotModel.ets | :85 |
| `file.write()` | 写入数据 | screenShotModel.ets | :86 |
| `file.fsync()` | 同步数据 | screenShotModel.ets | :87 |
| `fileAsset.close()` | 关闭文件 | screenShotModel.ets | :94 |

**调用示例**:
```typescript
import userFileManager from '@ohos.filemanagement.userFileManager';
import file from '@ohos.file.fs';

const userFileMgr = userFileManager.getUserFileMgr(context);
const fileAsset = await userFileMgr.createPhotoAsset(
    this.imageFileName,
    { subType: userFileManager.PhotoSubType.SCREENSHOT }
);

const fd = await fileAsset.open('w');
await file.write(fd, packedImg);
await file.fsync(fd);
await fileAsset.close(fd);
```

**证据**: `features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:23,68-94`

---

### 5. @ohos.app.ability.common

**用途**: 应用能力上下文

| 类型 | 用途 | 调用位置 |
|-----|------|---------|
| `common.ServiceExtensionContext` | 服务扩展上下文 | screenShotModel.ets |

**获取上下文**:
```typescript
const context = globalThis.shotScreenContext as common.ServiceExtensionContext;
```

**证据**: `features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:24,66`

---

### 6. @ohos.app.ability.Want

**用途**: 能力启动参数

| 用途 | 调用位置 | 行号 |
|-----|---------|------|
| 启动其他应用 | screenShotModel.ets | :122-124 |

**Want 结构**:
```typescript
const wantData: Want = {
    bundleName: 'com.ohos.photos',
    abilityName: 'com.ohos.photos.MainAbility',
    parameters: { uri: imageFileName }
};

globalThis.shotScreenContext.startAbility(wantData);
```

---

### 7. @ohos.hilog

**用途**: 日志输出

| API | 功能 | 调用位置 |
|-----|------|---------|
| `hiLog.info()` | Info 日志 | Log.ts |
| `hiLog.debug()` | Debug 日志 | Log.ts |
| `hiLog.error()` | Error 日志 | Log.ts |

**日志配置**:
- Domain: `0x55EE`
- Prefix: `[Screenshot]`

---

### 8. @ohos.display

**用途**: 显示信息

| API | 功能 | 调用位置 | 行号 |
|-----|------|---------|------|
| `display.getDefaultDisplaySync()` | 获取默认显示信息 | ServiceExtAbility.ets | :44 |

---

### 9. @ohos.multimedia.media

**用途**: 媒体上报

| API | 功能 | 调用位置 | 行号 |
|-----|------|---------|------|
| `media.reportAVScreenCaptureUserChoice()` | 上报用户选择 | DialogAbility.ets | :52 |

---

## 权限配置

### 声明的权限

**文件**: `product/phone/src/main/module.json5:50-77`

| 权限名称 | 用途 | 使用场景 |
|---------|------|---------|
| `ohos.permission.CAPTURE_SCREEN` | 截屏权限 | ServiceExtAbility 声明 |
| `ohos.permission.MEDIA_LOCATION` | 媒体位置 | 保存图片 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 获取包信息特权 | - |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 后台启动能力 | - |
| `ohos.permission.WRITE_IMAGEVIDEO` | 写入图片视频 | 保存截图 |

### 权限使用位置

| 权限 | 声明位置 | 使用位置 |
|-----|---------|---------|
| CAPTURE_SCREEN | module.json5:27 | 系统服务 |
| WRITE_IMAGEVIDEO | module.json5:70 | screenShotModel.ets |

---

## 能力注册

### ServiceExtAbility

**配置位置**: `product/phone/src/main/module.json5:16-41`

| 配置项 | 值 |
|-------|------|
| name | `com.ohos.screenshot.ServiceExtAbility` |
| type | service |
| visible | true |
| skills | `com.ohos.systemui.action.TOGGLE` |
| permissions | `ohos.permission.CAPTURE_SCREEN` |
| srcEntrance | `./ets/ServiceExtAbility/ServiceExtAbility.ets` |

### DialogAbility

**配置位置**: `product/phone/src/main/module.json5:43-48`

| 配置项 | 值 |
|-------|------|
| name | `com.ohos.screenshot.DialogAbility` |
| type | sys/commonUI |
| visible | true |
| srcEntry | `./ets/PrivacyDialog/DialogAbility.ets` |

---

## API 调用链

### 截屏操作

```
用户触发
    ↓
ServiceExtAbility.onCreate()
    ↓
windowManager.createWindow(TYPE_SCREENSHOT)
    ↓
window.setUIContent(pages/index)
    ↓
Index.aboutToAppear() → shotScreen()
    ↓
ScreenshotManager.save()
    ↓
saveImage() → createImagePacker → packing
    ↓
createPhotoAsset → file.write → fsync
    ↓
AppStorage.setOrCreate('captureImage')
    ↓
UI 更新显示
```

### 打开相册

```
用户点击预览图片
    ↓
ViewModel.StartPhotosAbility()
    ↓
ShotScreenModel.openAbility()
    ↓
startAbility(com.ohos.photos.MainAbility)
    ↓
关闭截屏窗口
```
