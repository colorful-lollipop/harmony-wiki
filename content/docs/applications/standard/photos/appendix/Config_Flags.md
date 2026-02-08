# 附录：配置参数表

## 1. MainAbility 参数

### 1.1 Want 参数 (外部传入)

| 参数名 | 类型 | 可选 | 默认值 | 用途 | 证据 |
|--------|------|------|--------|------|------|
| `uri` | string | 必填 | - | 路由类型 | `MainAbility.ts:99` |
| `callerBundleName` | string | 可选 | - | 调用方包名 | `MainAbility.ts:114` |
| `maxSelectCount` | number | 可选 | 9 | 最大选择数 | `MainAbility.ts:125` |
| `filterMediaType` | string | 可选 | "all" | 过滤类型 | `MainAbility.ts:118` |
| `preselectedUris` | string[] | 可选 | [] | 预选 URI | `MainAbility.ts:127` |
| `isPhotoTakingSupported` | boolean | 可选 | true | 允许拍照 | `MainAbility.ts:120` |
| `isEditSupported` | boolean | 可选 | true | 允许编辑 | `MainAbility.ts:121` |
| `albumUri` | string | 可选 | - | 相册 URI | `MainAbility.ts:135` |
| `currentUri` | string | 可选 | - | 当前 URI | `MainAbility.ts:136` |
| `currentIndex` | number | 可选 | - | 当前索引 | `MainAbility.ts:137` |
| `displayName` | string | 可选 | - | 显示名称 | `MainAbility.ts:138` |
| `formId` | string | 可选 | - | FA 卡片 ID | `MainAbility.ts:142` |
| `isShowMenu` | boolean | 可选 | true | 显示菜单 | `MainAbility.ts:145` |
| `viewDataAlbum` | string | 可选 | - | 查看相册 | `MainAbility.ts:153` |
| `viewDataIndex` | number | 可选 | - | 查看索引 | `MainAbility.ts:158` |

### 1.2 URI 值定义

| URI 值 | 用途 | 路由目标 |
|--------|------|----------|
| `"photodetail"` | 相机回调 | `PhotoBrowser` |
| `"singleselect"` | 单选模式 | `ThirdSelectPhotoGridPage` |
| `"multipleselect"` | 多选模式 | `ThirdSelectPhotoGridPage` |
| `"form"` | FA 浏览 | `PhotoBrowser` / `DefaultPhotoPage` |
| `"formNone"` | FA 默认 | `DefaultPhotoPage` |

---

## 2. 常量定义

### 2.1 Constants

| 常量名 | 值 | 用途 |
|--------|-----|------|
| `WANT_PARAM_URI_DETAIL` | `"photodetail"` | 相机浏览 |
| `WANT_PARAM_URI_SELECT_SINGLE` | `"singleselect"` | 单选 |
| `WANT_PARAM_URI_SELECT_MULTIPLE` | `"multipleselect"` | 多选 |
| `WANT_PARAM_URI_FORM` | `"form"` | FA 浏览 |
| `WANT_PARAM_URI_FORM_NONE` | `"formNone"` | FA 默认 |
| `NUMBER_1` | `1` | 数字 1 |
| `NUMBER_9` | `9` | 最大选择数 |
| `KEY_WANT_PARAMETERS_CALLER_BUNDLE_NAME` | `"callerBundleName"` | 调用方包名 Key |

> **证据**: `MainAbility.ts:24` - `import { Constants, ... } from '@ohos/common'`

### 2.2 AlbumDefine

| 常量名 | 值 | 用途 |
|--------|-----|------|
| `FILTER_MEDIA_TYPE_ALL` | `"all"` | 全部 |
| `FILTER_MEDIA_TYPE_IMAGE` | `"image"` | 仅图片 |
| `FILTER_MEDIA_TYPE_VIDEO` | `"video"` | 仅视频 |

### 2.3 EntryFrom 定义

| 常量名 | 值 | 入口类型 |
|--------|-----|----------|
| `ENTRY_FROM_NONE` | `"entryFromNone"` | 无 |
| `ENTRY_FROM_CAMERA` | `"entryFromCamera"` | 相机 |
| `ENTRY_FROM_SINGLE_SELECT` | `"entryFromSingleSelect"` | 单选 |
| `ENTRY_FROM_MULTIPLE_SELECT` | `"entryFromMultipleSelect"` | 多选 |
| `ENTRY_FROM_FORM_ABILITY` | `"entryFromFormAbility"` | FA |
| `ENTRY_FROM_FORM_DEFAULT_ABILITY` | `"entryFromFormDefaultAbility"` | FA 默认 |
| `ENTRY_FROM_FORM_FORM_EDITOR` | `"entryFromFormFormEditor"` | FA 编辑 |
| `ENTRY_FROM_VIEW_DATA` | `"entryFromViewData"` | 查看数据 |

---

## 3. AppStorage 键

| 键名 | 类型 | 用途 | 证据 |
|------|------|------|------|
| `photosAbilityContext` | UIAbilityContext | 能力上下文 | `MainAbility.ts:61` |
| `formContext` | UIAbilityContext | FA 上下文 | `MainAbility.ts:62` |
| `placeholderIndex` | number | 占位索引 | `MainAbility.ts:94` |
| `entryFromHap` | string | 入口来源 | `MainAbility.ts:111` |
| `photosWindowStage` | WindowStage | 窗口舞台 | `MainAbility.ts:184` |
| `deviceType` | string | 设备类型 | `MainAbility.ts:185` |
| `mainWindow` | Window | 主窗口 | `MainAbility.ts:198` |
| `leftBlank` | number | 左侧空白 | `MainAbility.ts:189` |
| `isSplitMode` | boolean | 分屏模式 | `MainAbility.ts:194` |
| `viewDataUri` | string | 查看数据 URI | `MainAbility.ts:148` |
| `form_albumUri` | string | FA 相册 URI | `MainAbility.ts:135` |
| `form_currentUri` | string | FA 当前 URI | `MainAbility.ts:136` |
| `form_currentIndex` | number | FA 当前索引 | `MainAbility.ts:137` |
| `form_displayName` | string | FA 显示名 | `MainAbility.ts:138` |

---

## 4. 页面路由配置

### 4.1 页面清单

| 路由路径 | 页面文件 | 用途 |
|----------|----------|------|
| `pages/index` | index.ets | 首页 |
| `pages/PhotoBrowser` | PhotoBrowser.ets | 照片浏览 |
| `pages/PhotoGridPage` | PhotoGridPage.ets | 宫格视图 |
| `pages/NewAlbumPage` | NewAlbumPage.ets | 新建相册 |
| `pages/ThirdSelectPhotoGridPage` | ThirdSelectPhotoGridPage.ets | 第三方选择 |
| `pages/ThirdSelectAlbumSetPage` | ThirdSelectAlbumSetPage.ets | 第三方相册 |
| `pages/ThirdSelectPhotoBrowser` | ThirdSelectPhotoBrowser.ets | 第三方浏览 |
| `pages/DefaultPhotoPage` | DefaultPhotoPage.ets | FA 默认页 |
| `pages/FormEditorPage` | FormEditorPage.ets | FA 编辑 |
| `pages/VideoBrowser` | VideoBrowser.ets | 视频浏览 |
| `pages/EditMain` | EditMain.ets | 编辑主页 |
| `pages/DeleteUIExtensionPage` | DeleteUIExtensionPage.ets | 删除对话框 |
| `pages/SaveUIExtensionPage` | SaveUIExtensionPage.ets | 保存对话框 |

> **证据**: `MainAbility.ts:203, 229, 243, 256, 280, 305, 321, 326`

---

## 5. BigData 常量

| 常量名 | 值 | 用途 |
|--------|-----|------|
| `SPLIT_SCREEN_ID` | - | 分屏上报 |
| `SELECT_PICKER_ID` | - | 选择器上报 |

> **证据**: `MainAbility.ts:21, 193`

---

## 6. BroadCast 事件

| 事件名 | 用途 | 注册位置 |
|--------|------|----------|
| `BroadCastConstants.THIRD_ROUTE_PAGE` | 第三方路由 | `MainAbility.ts:75` |

---

## 7. 构建配置参数

### 7.1 build-profile.json5

| 参数 | 值 | 说明 |
|------|-----|------|
| `compileSdkVersion` | `23` | 编译 SDK 版本 |
| `compatibleSdkVersion` | `23` | 兼容 SDK 版本 |
| `signAlg` | `SHA256withECDSA` | 签名算法 |

### 7.2 模块类型

| 类型 | 说明 |
|------|------|
| `entry` | 入口模块，可安装 |
| `har` | 静态共享包 |

---

## 8. 权限配置

| 权限名 | 敏感度 | 用途 |
|--------|--------|------|
| `ohos.permission.PROXY_AUTHORIZATION_URI` | 低 | 代理授权 |
| `ohos.permission.READ_IMAGEVIDEO` | 高 | 读媒体 |
| `ohos.permission.WRITE_IMAGEVIDEO` | 高 | 写媒体 |
| `ohos.permission.MEDIA_LOCATION` | 中 | 位置 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 中 | 后台启动 |
| `ohos.permission.GET_BUNDLE_INFO` | 低 | 包信息 |

> **证据**: `module.json5:21-55`
