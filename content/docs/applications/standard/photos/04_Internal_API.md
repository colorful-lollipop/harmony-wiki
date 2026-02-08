# 内部 API 文档

## 1. 概述

本章节说明图库应用**内部模块的导出 API**，即 HAR 模块对外提供的接口，供 `phone_photos` 主模块及其他模块调用。

> **注意**: 本应用为**纯 ArkTS 应用**，无原生 N-API 接口。所有 API 均为 TypeScript/ArkTS 接口。

---

## 2. photos_common 导出 API

### 2.1 模块引用

```typescript
import { ... } from '@ohos/common';
```

### 2.2 导出常量

#### 2.2.1 Constants - 全局常量

| 常量名 | 类型 | 值 | 说明 |
|--------|------|-----|------|
| `WANT_PARAM_URI_DETAIL` | string | `"photodetail"` | 相机浏览 |
| `WANT_PARAM_URI_SELECT_SINGLE` | string | `"singleselect"` | 单选 |
| `WANT_PARAM_URI_SELECT_MULTIPLE` | string | `"multipleselect"` | 多选 |
| `WANT_PARAM_URI_FORM` | string | `"form"` | FA 浏览 |
| `WANT_PARAM_URI_FORM_NONE` | string | `"formNone"` | FA 默认 |
| `KEY_WANT_PARAMETERS_CALLER_BUNDLE_NAME` | string | `"callerBundleName"` | 调用方包名 |
| `NUMBER_1` | number | `1` | 数字 1 |
| `PHOTOS_STORE_KEY` | string | `"photos_preferences"` | 存储 Key |
| `DEFAULT_SLIDING_WIN_SIZE` | number | - | 默认滑动窗口 |
| `ENTRY_FROM_NONE` | string | `"entryFromNone"` | 无入口 |
| `ENTRY_FROM_CAMERA` | string | `"entryFromCamera"` | 相机 |
| `ENTRY_FROM_SINGLE_SELECT` | string | `"entryFromSingleSelect"` | 单选 |
| `ENTRY_FROM_MULTIPLE_SELECT` | string | `"entryFromMultipleSelect"` | 多选 |
| `ENTRY_FROM_FORM_ABILITY` | string | `"entryFromFormAbility"` | FA |
| `ENTRY_FROM_FORM_DEFAULT_ABILITY` | string | `"entryFromFormDefaultAbility"` | FA 默认 |
| `ENTRY_FROM_FORM_FORM_EDITOR` | string | `"entryFromFormFormEditor"` | FA 编辑 |
| `ENTRY_FROM_VIEW_DATA` | string | `"entryFromViewData"` | 查看数据 |
| `APP_KEY_PHOTO_BROWSER` | string | `"photoBrowser"` | PhotoBrowser Key |
| `IS_FIRST_TIME_DELETE` | string | `"isFirstTimeDelete"` | 首次删除标记 |
| `DEFAULT_DEVICE_TYPE` | string | `"default"` | 默认设备 |
| `PAD_DEVICE_TYPE` | string | `"pad"` | 平板设备 |

> **证据**: `MainAbility.ts:24` - `import { Constants, ... } from '@ohos/common'`

---

#### 2.2.2 AlbumDefine - 相册定义

| 常量名 | 类型 | 值 | 说明 |
|--------|------|-----|------|
| `FILTER_MEDIA_TYPE_ALL` | string | `"all"` | 全部媒体 |
| `FILTER_MEDIA_TYPE_IMAGE` | string | `"image"` | 仅图片 |
| `FILTER_MEDIA_TYPE_VIDEO` | string | `"video"` | 仅视频 |

> **证据**: `MainAbility.ts:20` - `import { AlbumDefine, ... } from '@ohos/common'`

---

#### 2.2.3 BigDataConstants - 大数据常量

| 常量名 | 类型 | 值 | 说明 |
|--------|------|-----|------|
| `SPLIT_SCREEN_ID` | string | - | 分屏上报 ID |
| `SELECT_PICKER_ID` | string | - | 选择器上报 ID |

> **证据**: `MainAbility.ts:21` - `import { BigDataConstants, ... } from '@ohos/common'`

---

#### 2.2.4 BroadCastConstants - 广播常量

| 常量名 | 类型 | 值 | 说明 |
|--------|------|-----|------|
| `THIRD_ROUTE_PAGE` | string | - | 第三方路由事件 |

> **证据**: `MainAbility.ts:22` - `import { BroadCastConstants, ... } from '@ohos/common'`

---

### 2.3 导出类/函数

#### 2.3.1 Log - 日志工具

```typescript
import { Log } from '@ohos/common';

// 使用方式
Log.info(TAG, `message: ${data}`);
Log.error(TAG, `error: ${message}`);
```

| 方法 | 参数 | 说明 |
|------|------|------|
| `info(tag: string, msg: string)` | 标签、消息 | Info 级别日志 |
| `error(tag: string, msg: string)` | 标签、消息 | Error 级别日志 |
| `warn(tag: string, msg: string)` | 标签、消息 | Warn 级别日志 |
| `debug(tag: string, msg: string)` | 标签、消息 | Debug 级别日志 |

> **证据**: `MainAbility.ts:25` - `import { Log, ... } from '@ohos/common'`

---

#### 2.3.2 UserFileManagerAccess - 媒体文件访问

```typescript
import { UserFileManagerAccess } from '@ohos/common';

// 获取单例
let access = UserFileManagerAccess.getInstance();

// 初始化
access.onCreate(context);

// 销毁
access.onDestroy();

// 准备系统相册
access.prepareSystemAlbums();
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `getInstance()` | - | `UserFileManagerAccess` | 获取单例 |
| `onCreate(context)` | UIAbilityContext | void | 初始化 |
| `onDestroy()` | - | void | 销毁 |
| `prepareSystemAlbums()` | - | Promise | 准备系统相册 |

> **证据**: `MainAbility.ts:31, 68, 78, 178`

---

#### 2.3.3 MediaObserver - 媒体观察者

```typescript
import { MediaObserver } from '@ohos/common';

let observer = MediaObserver.getInstance();

// 注册观察
observer.registerForAllPhotos();
observer.registerForAllAlbums();

// 取消注册
observer.unregisterForAllPhotos();
observer.unregisterForAllAlbums();
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `getInstance()` | - | `MediaObserver` | 获取单例 |
| `registerForAllPhotos()` | - | void | 注册照片观察 |
| `registerForAllAlbums()` | - | void | 注册相册观察 |
| `unregisterForAllPhotos()` | - | void | 取消照片观察 |
| `unregisterForAllAlbums()` | - | void | 取消相册观察 |

> **证据**: `MainAbility.ts:27, 69-70, 176-177`

---

#### 2.3.4 MediaDataSource - 媒体数据源

```typescript
import { MediaDataSource } from '@ohos/common';

let dataSource = new MediaDataSource(windowSize);
dataSource.setAlbumUri(albumUri);
dataSource.initialize();
dataSource.getRawData(index);
```

| 构造函数 | 参数 | 说明 |
|-----------|------|------|
| `new MediaDataSource(size)` | number | 创建数据源 |

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `setAlbumUri(uri)` | string | void | 设置相册 URI |
| `initialize()` | - | Promise | 初始化 |
| `getRawData(index)` | number | Object | 获取数据 |

> **证据**: `MainAbility.ts:26, 288-297`

---

#### 2.3.5 BroadCastManager - 广播管理器

```typescript
import { BroadCastManager } from '@ohos/common';

let manager = BroadCastManager.getInstance();
let broadcast = manager.getBroadCast();

// 发送事件
broadcast.emit(event, data);

// 监听事件
broadcast.on(event, callback);
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `getInstance()` | - | `BroadCastManager` | 获取单例 |
| `getBroadCast()` | - | `BroadCast` | 获取广播实例 |

> **证据**: `MainAbility.ts:23-24, 48, 75`

---

#### 2.3.6 ScreenManager - 屏幕管理器

```typescript
import { ScreenManager } from '@ohos/common';

let screenMgr = ScreenManager.getInstance();

// 初始化
screenMgr.initializationSize(win);
screenMgr.initWindowMode();

// 监听事件
screenMgr.on(ON_LEFT_BLANK_CHANGED, callback);
screenMgr.on(ON_SPLIT_MODE_CHANGED, callback);

// 销毁
screenMgr.destroyWindowMode();
screenMgr.release();
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `getInstance()` | - | `ScreenManager` | 获取单例 |
| `initializationSize(win)` | Window | Promise | 初始化尺寸 |
| `initWindowMode()` | - | void | 初始化窗口模式 |
| `destroyWindowMode()` | - | void | 销毁窗口模式 |
| `release()` | - | void | 释放资源 |

> **证据**: `MainAbility.ts:29, 187-194, 212`

---

#### 2.3.7 StatusBarColorController - 状态栏控制器

```typescript
import { StatusBarColorController } from '@ohos/common';

let controller = StatusBarColorController.getInstance();
controller.release();
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `getInstance()` | - | `StatusBarColorController` | 获取单例 |
| `release()` | - | void | 释放资源 |

> **证据**: `MainAbility.ts:30, 173-174`

---

#### 2.3.8 ReportToBigDataUtil - 大数据上报

```typescript
import { ReportToBigDataUtil } from '@ohos/common';

ReportToBigDataUtil.report(eventId, params);
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `report(eventId, params)` | string, Object | void | 上报数据 |

> **证据**: `MainAbility.ts:28, 237-240`

---

---

## 3. photos_timeline 导出 API

### 3.1 模块引用

```typescript
import { TimelineDataSourceManager } from '@ohos/timeline';
```

### 3.2 TimelineDataSourceManager

```typescript
let manager = TimelineDataSourceManager.getInstance();
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `getInstance()` | - | `TimelineDataSourceManager` | 获取单例 |

> **证据**: `MainAbility.ts:33, 72`

---

## 4. photos_thirdselect 导出 API

### 4.1 模块引用

```typescript
import { SmartPickerUtils } from '@ohos/thirdselect/src/main/ets/default/utils/SmartPickerUtils';
```

### 4.2 SmartPickerUtils

```typescript
SmartPickerUtils.initIfNeeded(context, want, localStorage);
```

| 方法 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `initIfNeeded(context, want, localStorage)` | Context, Want, LocalStorage | void | 初始化选择器 |

> **证据**: `MainAbility.ts:40, 122, 131`

---

## 5. 内部 API 稳定性标注

### 5.1 稳定性等级说明

| 等级 | 标记 | 说明 |
|------|------|------|
| **稳定** | 无特殊标记 | 公开 API，可安全使用 |
| **内部** | 无 `@ohos` 导出 | 仅模块内部使用 |
| **实验性** | 无 | 可能在未来版本变更 |

### 5.2 稳定性标注

| API | 稳定性 | 说明 |
|-----|--------|------|
| `@ohos/common` 常量 | ✅ 稳定 | 公开导出 |
| `UserFileManagerAccess` | ✅ 稳定 | 公开类 |
| `MediaObserver` | ✅ 稳定 | 公开类 |
| `MediaDataSource` | ✅ 稳定 | 公开类 |
| `BroadCastManager` | ✅ 稳定 | 公开类 |
| `ScreenManager` | ✅ 稳定 | 公开类 |
| `TimelineDataSourceManager` | ✅ 稳定 | 公开类 |
| `SmartPickerUtils` | ✅ 稳定 | 公开类 |

---

## 6. 内部调用关系

```
phone_photos (entry)
    │
    ├── @ohos/common
    │   ├── UserFileManagerAccess ───┐
    │   ├── MediaObserver ──────────┤── 共享依赖
    │   ├── MediaDataSource ────────┤
    │   ├── BroadCastManager ───────┤
    │   ├── ScreenManager ──────────┤
    │   └── ... (其他工具类) ────────┘
    │
    ├── @ohos/timeline
    │   └── TimelineDataSourceManager
    │
    └── @ohos/thirdselect
        └── SmartPickerUtils
```

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [02_Module_Structure.md](02_Module_Structure.md) | 模块结构 |
| [01_Architecture.md](01_Architecture.md) | 架构说明 |
| [05_Build_System.md](05_Build_System.md) | 构建配置 |
