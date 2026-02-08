# 对外 API

## 目的

本文档列出 FilePicker 应用的所有对外接口，包括 API 清单、参数说明、错误码和调用示例。

## 适用范围

本文档适用于：
- 需要集成 FilePicker 功能的第三方应用开发者
- 需要调用 FilePicker 接口的开发者

## 关键结论

1. **通过 startAbilityForResult 拉起**：使用 Want 对象传递参数和接收结果
2. **支持多种启动模式**：choose（选择文件）、save（保存文件）、download（下载授权）
3. **完善的参数体系**：支持文件过滤、数量限制、默认路径等
4. **URI 权限授权**：返回的 URI 自动授权给调用者

## API 清单表

### 1. 文件选择 API

| API 名称 | 类型 | 功能 | 证据 |
|---------|------|------|-------|
| FilePicker.chooseFile | 同步 | 选择文件 | `README.md:14-33` |

#### Want 参数

| 参数名 | 类型 | 必填 | 说明 | 证据 |
|--------|------|-------|------|-------|
| bundleName | string | 是 | 目标应用包名，固定为 "com.ohos.filepicker" | `AppScope/app.json5:3` |
| abilityName | string | 是 | 目标 Ability 名称，固定为 "com.ohos.filepicker.MainAbility" | `entry/src/main/module.json5:17` |
| parameters.startMode | string | 否 | 启动模式，"choose" 或 "save"，默认 "choose" | `FilePickerUtil.ets:70` |
| parameters.key_pick_num | number | 否 | 最大选择数量，默认 1 | `FilePickerUtil.ets:95` |
| parameters.key_select_mode | number | 否 | 选择模式：0=FILE, 1=FOLDER, 2=MIX，默认 0 | `FilePickerUtil.ets:96` |
| parameters.key_file_suffix_filter | string[] | 否 | 文件后缀过滤列表，如 [".jpg", ".png"] | `FilePickerUtil.ets:94` |
| parameters.pickerType | string | 否 | picker 类型，如 "image/*", "video/*" | `FilePickerUtil.ets:82` |
| parameters.key_pick_dir_path | string | 否 | 默认选择路径（URI 格式） | `FilePickerUtil.ets:80` |

#### 返回结果

| 参数名 | 类型 | 说明 | 证据 |
|--------|------|------|-------|
| resultCode | number | 结果码：0=成功, -1=取消 | `FilePickerUtil.ets:204` |
| want.parameters.startMode | string | 返回的启动模式 | `FilePickerUtil.ets:209-227` |
| want.parameters.result | string[] | 选择的 URI 列表 | `FilePickerUtil.ets:223` |
| want.parameters.ability.params.stream | string[] | URI 列表（stream 格式） | `FilePickerUtil.ets:223` |
| want.parameters.ability.params.udkey | string | UDMF 密钥（批量模式） | `FilePickerUtil.ets:212` |

#### 调用示例

```typescript
import { common } from '@kit.AbilityKit';
import { Want } from '@ohos.app.ability.Want';
import { wantConstant } from '@kit.AbilityKit';

// 基本文件选择
let want: Want = {
    bundleName: 'com.ohos.filepicker',
    abilityName: 'com.ohos.filepicker.MainAbility',
    parameters: {
        'startMode': 'choose',
        'key_pick_num': 5,
        'key_select_mode': 0,  // 0: FILE, 1: FOLDER, 2: MIX
        'key_file_suffix_filter': ['.jpg', '.png'],
        'pickerType': 'image/*'
    }
};

let context = getContext(this) as common.UIAbilityContext;
context.startAbilityForResult(
    want,
    { windowMode: 102 }  // 弹窗模式
).then((result) => {
    if (result.resultCode === 0) {
        let uris = result.want?.parameters?.['ability.params.stream'] as string[];
        console.log('Selected files:', uris);
    }
});
```

**证据**：`README.md:14-33`

### 2. 文件保存 API

| API 名称 | 类型 | 功能 | 证据 |
|---------|------|------|-------|
| FilePicker.saveFile | 同步 | 保存文件 | `README.md:35-54` |

#### Want 参数

| 参数名 | 类型 | 必填 | 说明 | 证据 |
|--------|------|-------|------|-------|
| bundleName | string | 是 | 目标应用包名，固定为 "com.ohos.filepicker" | `AppScope/app.json5:3` |
| abilityName | string | 是 | 目标 Ability 名称，固定为 "com.ohos.filepicker.MainAbility" | `entry/src/main/module.json5:17` |
| parameters.startMode | string | 是 | 启动模式，必须为 "save" | `FilePickerUtil.ets:70` |
| parameters.key_pick_file_name | string[] | 否 | 保存文件名列表，默认 ["untitled"] | `FilePickerUtil.ets:100` |
| parameters.key_file_suffix_choices | string[] | 否 | 文件后缀选项，如 [".jpg", ".png"] | `FilePickerUtil.ets:101` |

#### 返回结果

| 参数名 | 类型 | 说明 | 证据 |
|--------|------|------|-------|
| resultCode | number | 结果码：0=成功, 1001=文件名冲突, 1002=文件名非法, 9001=其他错误 | `PathPicker.ets:64-79` |
| want.parameters.startMode | string | 返回的启动模式 | `PathPicker.ets:89-98` |
| want.parameters.result | string[] | 创建文件的 URI 列表 | `PathPicker.ets:57-58` |

#### 调用示例

```typescript
// 保存单个文件
let want: Want = {
    bundleName: 'com.ohos.filepicker',
    abilityName: 'com.ohos.filepicker.MainAbility',
    parameters: {
        'startMode': 'save',
        'key_pick_file_name': ['test.jpg'],
        'key_file_suffix_choices': ['.jpg', '.png']
    }
};

context.startAbilityForResult(
    want,
    { windowMode: 102 }
).then((result) => {
    if (result.resultCode === 0) {
        let uris = result.want?.parameters?.['result'] as string[];
        console.log('Created files:', uris);
    } else if (result.resultCode === 1001) {
        console.error('File name already exists');
    }
});

// 批量保存多个文件
let want: Want = {
    bundleName: 'com.ohos.filepicker',
    abilityName: 'com.ohos.filepicker.MainAbility',
    parameters: {
        'startMode': 'save',
        'key_pick_file_name': ['photo1.jpg', 'photo2.jpg', 'photo3.jpg']
    }
};
```

**证据**：`README.md:35-54`

### 3. 下载授权 API

**说明**：用于浏览器下载功能，为下载提供安全的保存位置。

#### Want 参数

| 参数名 | 类型 | 必填 | 说明 | 证据 |
|--------|------|-------|------|-------|
| bundleName | string | 是 | 目标应用包名，固定为 "com.ohos.filepicker" | `AppScope/app.json5:3` |
| abilityName | string | 是 | 目标 ExtensionAbility，"com.ohos.filepicker.FilePickerUIExtAbility" | `entry/src/main/module.json5:37` |
| parameters.action | string | 是 | 操作类型，"download" | `FilePickerUIExtAbility.ets:46` |
| ohos.aafwk.param.callerBundleName | string | 是 | 调用者 Bundle 名称 | `FilePickerUtil.ets:78` |

#### 返回结果

| 参数名 | 类型 | 说明 | 证据 |
|--------|------|------|-------|
| resultCode | number | 结果码：0=成功, -1=取消 | `DownloadAuth.ets:90` |
| want.parameters.downloadNewUri | string | 下载保存的 URI | `DownloadAuth.ets:93` |

#### 调用示例

```typescript
// 浏览器下载场景
let want: Want = {
    bundleName: 'com.ohos.filepicker',
    abilityName: 'com.ohos.filepicker.FilePickerUIExtAbility',
    action: 'download',
    parameters: {
        'ohos.aafwk.param.callerBundleName': 'com.example.browser'
    }
};

context.startAbilityForResult(
    want,
    { windowMode: 102 }
).then((result) => {
    if (result.resultCode === 0) {
        let downloadUri = result.want?.parameters?.['downloadNewUri'] as string;
        console.log('Download URI:', downloadUri);
        // 使用此 URI 保存下载文件
    }
});
```

**证据**：`DownloadAuth.ets:73-100`

## 错误码

### 应用层错误码

| 错误码 | 名称 | 说明 | 触发条件 | 证据 |
|--------|------|------|----------|-------|
| 1000 | PICKER.NORMAL | 正常 | 操作成功 | `ErrorCodeConst.ts:70` |
| 1001 | PICKER.FILE_NAME_EXIST | 文件名已存在 | 创建文件时文件名重复 | `ErrorCodeConst.ts:54` |
| 1002 | PICKER.FILE_NAME_INVALID | 文件名非法 | 文件名包含非法字符或过长 | `ErrorCodeConst.ts:58` |
| 2001 | PICKER.GRANT_URI_PERMISSION_FAIL | URI 授权失败 | URI 权限授权失败 | `ErrorCodeConst.ts:62` |
| 9001 | PICKER.OTHER_ERROR | 其他未知错误 | 其他未分类错误 | `ErrorCodeConst.ts:66` |

**证据**：`ErrorCodeConst.ts:50-71`

### 系统层错误码

| 错误码 | 名称 | 说明 | 触发条件 | 证据 |
|--------|------|------|----------|-------|
| 13900015 | FILE_NAME_EXIST | 文件名已存在 | FileAccess API 返回 | `ErrorCodeConst.ts:32` |
| 14000001 | FILE_NAME_INVALID | 文件名非法 | FileAccess API 返回 | `ErrorCodeConst.ts:36` |
| -102825984 | IPC_ERROR | IPC 异常 | 跨进程通信异常 | `ErrorCodeConst.ts:40` |
| '3' | GET_MEDIAFILE_NULL | 媒体库查询为空 | PhotoAccessHelper 返回 | `ErrorCodeConst.ts:44` |
| 2002 | ACCOUNT_NOT_LOGIN | 账号未登录 | 使用静默登录接口时账号未登录 | `ErrorCodeConst.ts:23` |

**证据**：`ErrorCodeConst.ts:19-46`

### 错误码映射

系统层错误到应用层错误的映射（`PathPicker.ets:64-79`）：

```
FILE_NAME_EXIST (13900015) → PICKER.FILE_NAME_EXIST (1001)
FILE_NAME_INVALID (14000001) → PICKER.FILE_NAME_INVALID (1002)
其他系统错误 → PICKER.OTHER_ERROR (9001)
```

**证据**：`PathPicker.ets:64-79`

## 权限要求

### 必需权限（调用者）

FilePicker 预置在系统中，第三方应用无需额外权限即可调用。

**说明**：
- ❌ 不需要声明 `ohos.permission.FILE_ACCESS_MANAGER`
- ❌ 不需要声明任何媒体库权限
- ✅ 系统自动处理权限授权

**证据**：`entry/src/main/module.json5:44-82` - 权限仅为 FilePicker 自身声明

### URI 权限机制

返回的 URI 自动授权给调用者：

| 权限标志 | 说明 | 证据 |
|----------|------|-------|
| FLAG_AUTH_READ_URI_PERMISSION | 读权限 | `FilePickerUtil.ets:135-137` |
| FLAG_AUTH_WRITE_URI_PERMISSION | 写权限 | `FilePickerUtil.ets:135-137` |
| FLAG_AUTH_PERSISTABLE_URI_PERMISSION | 持久化权限 | `FilePickerUtil.ets:135-137` |

**证据**：`FilePickerUtil.ets:135-137`

## 参数校验规则

### 文件名校验

**规则**（`Constant.ts:32`）：
- 正则：`/^[^\\/:*?<>\"|]+$/`
- 最大长度：225 个字符

**非法字符**：`\ / : * ? < > " |`

**证据**：`Constant.ts:32-34`

### 文件后缀过滤

**过滤逻辑**（`FilePickerUtil.ets:290-301`）：
```typescript
// 后缀匹配：必须完全匹配（区分大小写）
function checkFileSuffix(fileName: string, filters: string[]): boolean {
    const suffix = '.' + getFileSuffix(fileName);
    return filters.includes(suffix);
}

// MIME 匹配：支持通配符
function checkFileMimetype(fileName: string, pickerTypeList: string[]): boolean {
    const mimeType = getMimeType(fileName);
    return pickerTypeList.includes(mimeType) ||
           pickerTypeList.includes(getCategory(mimeType) + '/*');
}
```

**证据**：`FilePickerUtil.ets:290-346`

### 选择数量限制

| 模式 | 默认最大值 | 可调整 | 证据 |
|--------|------------|-------|-------|
| 文件选择 | 50 | 通过 `key_pick_num` 调整 | `FilePickerUtil.ets:95` |
| 批量授权 | 无限制 | 无 | `FilePickerUtil.ets:209` |

**证据**：`FilePickerUtil.ets:207-228`

## 相关跳转

- [概览](00_Overview.md) - 了解项目定位
- [架构设计](02_Architecture.md) - 理解数据流
- [权限机制](06_Permissions.md) - 了解授权流程
