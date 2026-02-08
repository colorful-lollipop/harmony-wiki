# User File Service - N-API 参考

## 概述

N-API (Native API) 是 OpenHarmony 提供的原生模块接口，允许 C/C++ 模块与 ArkTS/JS 运行时交互。user_file_service 通过 N-API 向上层应用提供文件访问能力。

---

## N-API 模块清单

### 1. file.fileAccess

**模块名**：`@ohos.file.fileAccess`

**注册位置**：`frameworks/js/napi/file_access_module/native_fileaccess_module.cpp:76-84`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_modname = "file.fileAccess",
    .nm_register_func = Init,
};
```

**安装路径**：`/system/lib/module/file/libfileaccess.z.so`

### 2. file.fileExtensionInfo

**模块名**：`@ohos.file.fileExtensionInfo`

**注册位置**：`frameworks/js/napi/file_extension_info_module/module_export_napi.cpp:44-52`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_modname = "file.fileExtensionInfo",
    .nm_register_func = FileExtensionInfoExport,
};
```

### 3. file.picker

**模块名**：`@ohos.file.picker`

**注册位置**：`interfaces/kits/picker/native_module_ohos_picker.cpp:76-84`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_modname = "file.picker",
    .nm_register_func = Export,
};
```

---

## FileAccess 模块 API

### FileAccessHelper 类

**JS 构造**：`new FileAccessHelper(context)`

**权限要求**：`ohos.permission.FILE_ACCESS_MANAGER`

#### 静态方法

| 方法 | 参数 | 返回值 | 同步/异步 | 说明 |
|------|------|--------|----------|------|
| `getRoot()` | context | Promise&lt;RootInfo[]&gt; | Async | 获取根路径列表 |
| `scanFile()` | context, uri | Promise&lt;FileInfo[]&gt; | Async | 扫描文件 |

#### 实例方法

| 方法 | 参数 | 返回值 | 同步/异步 | 说明 |
|------|------|--------|----------|------|
| `listFile(uri)` | string | Promise&lt;FileInfo[]&gt; | Async | 列出目录 |
| `createFile(uri, name)` | string, string | Promise&lt;FileInfo&gt; | Async | 创建文件 |
| `mkdir(uri, name)` | string, string | Promise&lt;FileInfo&gt; | Async | 创建目录 |
| `delete(uri)` | string | Promise&lt;number&gt; | Async | 删除 |
| `move(srcUri, destUri)` | string, string | Promise&lt;FileInfo&gt; | Async | 移动 |
| `rename(uri, newName)` | string, string | Promise&lt;FileInfo&gt; | Async | 重命名 |
| `open(uri, flags)` | string, number | Promise&lt;number&gt; | Async | 打开文件 |
| `access(uri)` | string | Promise&lt;boolean&gt; | Async | 检查存在 |
| `getFileInfo(uri)` | string | Promise&lt;FileInfo&gt; | Async | 获取信息 |
| `getFileInfoFromUri(uri)` | string | Promise&lt;FileInfo&gt; | Async | 从 URI 获取 |

**证据**：`native_fileaccess_module.cpp:36-49`

---

### FileInfo 类

**导出位置**：`frameworks/js/napi/file_access_module/file_info/napi_file_info_exporter.cpp`

#### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `uri` | string | 文件 URI |
| `path` | string | 文件路径 |
| `name` | string | 文件名 |
| `size` | number | 文件大小 |
| `mtime` | number | 修改时间 |
| `ctime` | number | 创建时间 |
| `mode` | number | 权限模式 |
| `type` | number | 文件类型 |

#### 方法

| 方法 | 说明 |
|------|------|
| `iterator()` | 获取文件迭代器 |

---

### RootInfo 类

**导出位置**：`frameworks/js/napi/file_access_module/root_info/napi_root_info_exporter.cpp`

#### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `uri` | string | 根路径 URI |
| `path` | string | 根路径 |
| `deviceType` | DeviceType | 设备类型 |
| `displayName` | string | 显示名称 |

---

### Iterator 类

| 类型 | 说明 |
|------|------|
| `FileIterator` | 文件迭代器 |
| `RootIterator` | 根路径迭代器 |

**导出位置**：
- `napi_file_iterator_exporter.cpp`
- `napi_root_iterator_exporter.cpp`

#### 迭代器方法

| 方法 | 说明 |
|------|------|
| `next()` | 获取下一个元素 |
| `getURI()` | 获取 URI |
| `isEnded()` | 是否结束 |

---

## FileExtensionInfo 模块 API

**模块名**：`@ohos.file.fileExtensionInfo`

**注册位置**：`module_export_napi.cpp:29-37`

### DeviceFlag 枚举

| 值 | 说明 |
|---|------|
| `DEVICE_LOCAL` | 本地存储 |
| `DEVICE_SHARED` | 共享存储 |
| `CLOUD` | 云存储 |

### DocumentFlag 枚举

| 值 | 说明 |
|---|------|
| `NO_QUERY` | 不查询文件 |
| `GET_FILE_INFO` | 获取文件信息 |
| `GET_DIR_INFO` | 获取目录信息 |
| `FOR_ALL` | 所有文件 |

### DeviceType 枚举

| 值 | 说明 |
|---|------|
| `LOCAL_DISK` | 本地磁盘 |
| `USB` | USB 设备 |
| `NETWORK` | 网络存储 |
| `CLOUD` | 云盘 |

**证据**：`file_extension_info_napi.cpp:29-37`

---

## Picker 模块 API

**模块名**：`@ohos.file.picker`

**注册位置**：`native_module_ohos_picker.cpp:31-46`

### DocumentViewPicker 类

**构造**：`new DocumentViewPicker()`

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `select()` | options | Promise&lt;string[]&gt; | 选择文档 |
| `save()` | options | Promise&lt;string[]&gt; | 保存文档 |

### PhotoViewPicker 类

**构造**：`new PhotoViewPicker()`

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `select()` | options | Promise&lt;string[]&gt; | 选择图片 |

### AudioViewPicker 类

**构造**：`new AudioViewPicker()`

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `select()` | options | Promise&lt;string[]&gt; | 选择音频 |

### VideoViewPicker 类

**构造**：`new VideoViewPicker()`

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `select()` | options | Promise&lt;string[]&gt; | 选择视频 |

---

## 权限配置

### 权限清单

| 权限名 | 用途 | 敏感级别 | 申请方式 |
|--------|------|----------|----------|
| `ohos.permission.FILE_ACCESS_MANAGER` | 文件访问管理 | system_grant | 预置权限 |
| `ohos.permission.FILE_ACCESS_AS_USER` | 用户级文件访问 | user_grant | 动态申请 |

### 权限申请示例

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

async function requestFileAccessPermission() {
    let atManager = abilityAccessCtrl.createAtManager();
    let bundleInfo = await bundleManager.getBundleInfoForSelf(
        bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    );
    
    let tokenId = bundleInfo.appInfo.accessTokenId;
    atManager.grantUserGrantedPermission(tokenId, 
        'ohos.permission.FILE_ACCESS_MANAGER', 
        0);
}
```

---

## 错误码

### 通用错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | `E_OK` | 成功 |
| -1 | `E_PERMISSION` | 权限不足 |
| -2 | `E_URI` | URI 非法 |
| -3 | `E_IO` | IO 错误 |
| -4 | `E_NOENT` | 文件不存在 |
| -5 | `E_EXIST` | 文件已存在 |
| -6 | `E_ISDIR` | 是目录 |
| -7 | `E_NOTDIR` | 不是目录 |
| -8 | `E_INVAL` | 参数无效 |
| -9 | `E_NAMETOOLONG` | 名称过长 |
| -10 | `E_NOSPC` | 空间不足 |

**证据**：`file_access_framework_errno.h`

### 扩展错误码

| 错误码 | 说明 |
|--------|------|
| `E_CALLBACK_AND_URI_HAS_NOT_RELATIONS` | 回调与 URI 无关联 |
| `E_BUNDLE_NAME_NOT_FOUND` | 包名未找到 |
| `E_LOAD_SA_FAILED` | 加载系统服务失败 |
| `E_EXT_CONNECT_FAILED` | 扩展连接失败 |

---

## API 调用链示例

### JS → N-API → Service 调用链

```
┌─────────────────────────────────────────────────────────────────┐
│  应用层 (ArkTS/JS)                                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  N-API 胶水层                                                     │
│  文件: native_fileaccess_module.cpp                              │
│  函数: Init(env, exports)                                       │
│  - FileAccessHelperInit()                                        │
│  - napi_define_properties() 导出 JS 类                           │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  N-API 实现层                                                     │
│  文件: napi_fileaccess_helper.cpp                                │
│  函数: ListFile(env, info)                                       │
│  - napi_get_cb_info() 解析参数                                   │
│  - uv_queue_work() 异步任务                                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  IPC Client                                                      │
│  文件: file_access_service_client.cpp                            │
│  函数: ListFile(uri)                                             │
│  - GetSystemAbility(5010)                                        │
│  - SendRequest(CMD_LISTFILE, data)                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  FileAccessService (SA 5010)                                     │
│  文件: file_access_service.cpp                                   │
│  函数: OnRequest(code, data, reply)                              │
│  - CMD_LISTFILE 处理                                             │
│  - CheckCallingPermission()                                      │
│  - ConnectExtension() → IFileAccessExtBase                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  底层服务 (扩展 Ability)                                          │
│  - medialibrary (媒体库)                                          │
│  - externalFileManager (外置存储)                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 使用示例

### 文件列表查询

```typescript
import fileAccess from '@ohos.file.fileAccess';

async function listPublicFiles() {
    let faHelper = await fileAccess.getFileAccessHelper();
    
    // 获取根路径
    let roots = await faHelper.getRoot();
    
    // 遍历根路径，列出文件
    for (let root of roots) {
        console.log(`Root: ${root.uri}`);
        let files = await faHelper.listFile(root.uri);
        for (let file of files) {
            console.log(`  - ${file.name} (${file.size} bytes)`);
        }
    }
}
```

### 文件创建

```typescript
async function createDocument() {
    let faHelper = await fileAccess.getFileAccessHelper();
    
    // 创建文档
    let file = await faHelper.createFile(
        'datashare:///document/primary:Documents',
        'my_document.txt'
    );
    
    console.log(`Created: ${file.uri}`);
    return file;
}
```

### 文件选择器

```typescript
import picker from '@ohos.file.picker';

async function selectImage() {
    let photoPicker = new picker.PhotoViewPicker();
    
    let result = await photoPicker.select({
        maxSelectNumber: 9,
        MIMEType: picker.PhotoViewMIMEType.IMAGE_TYPE
    });
    
    console.log(`Selected: ${result.photoUris}`);
    return result;
}
```
