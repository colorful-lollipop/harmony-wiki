# MediaLibrary N-API 接口参考

## 模块概述

MediaLibrary 提供三个主要的 N-API 模块，分别面向不同的使用场景：

| 模块命名空间 | 用途 | 主要类 | 注册文件 |
|------------|------|-------|---------|
| `multimedia.mediaLibrary` | 主媒体库 API | MediaLibraryNapi | `native_module_ohos_medialibrary.cpp` |
| `userfileManager` | 用户文件管理 | UserFileMgrNapi | `native_module_ohos_userfile_manager.cpp` |
| `photoAccessHelper` | 照片访问助手 | PhotoAccessHelperNapi | `native_module_ohos_photoaccess_helper.cpp` |

---

## 1. multimedia.mediaLibrary 模块

### 模块注册

**文件**: `frameworks/js/src/native_module_ohos_medialibrary.cpp`

```cpp
// 行 64-72: 模块定义
static napi_module g_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Export,           // 注册函数
    .nm_modname = "multimedia.mediaLibrary", // 模块名
    .nm_priv = reinterpret_cast<void *>(0),
    .reserved = {0}
};

// 行 77-80: 模块注册
extern "C" __attribute__((constructor)) void RegisterModule(void) {
    napi_module_register(&g_module);
}
```

**导出函数**: `frameworks/js/src/native_module_ohos_medialibrary.cpp:27-35`

```cpp
static napi_value Export(napi_env env, napi_value exports) {
    FileAssetNapi::Init(env, exports);
    FetchFileResultNapi::Init(env, exports);
    AlbumNapi::Init(env, exports);
    SmartAlbumNapi::Init(env, exports);
    MediaLibraryNapi::Init(env, exports);
    return exports;
}
```

### API 清单

#### MediaLibrary 类

| JS 方法 | 参数 | 返回值 | C++ 实现 | 说明 |
|--------|------|-------|---------|------|
| `getMediaAssets()` | options | FetchFileResult | `media_library_napi.cpp` | 获取媒体资产 |
| `getAudioAssets()` | options | FetchFileResult | `media_library_napi.cpp` | 获取音频资产 |
| `getVideoAssets()` | options | FetchFileResult | `media_library_napi.cpp` | 获取视频资产 |
| `getImageAssets()` | options | FetchFileResult | `media_library_napi.cpp` | 获取图片资产 |
| `createAlbum()` | name | Album | `media_library_napi.cpp` | 创建相册 |

#### FileAsset 类

| JS 方法 | 参数 | 返回值 | C++ 实现 | 说明 |
|--------|------|-------|---------|------|
| `getThumbnail()` | options | PixelMap | `file_asset_napi.cpp` | 获取缩略图 |
| `getFilePath()` | - | string | `file_asset_napi.cpp` | 获取文件路径 |
| `commitModify()` | - | boolean | `file_asset_napi.cpp` | 提交修改 |
| `open()` | mode | number | `file_asset_napi.cpp` | 打开文件 |
| `close()` | fd | boolean | `file_asset_napi.cpp` | 关闭文件 |

#### FetchFileResult 类

| JS 方法 | 参数 | 返回值 | C++ 实现 | 说明 |
|--------|------|-------|---------|------|
| `getFirstObject()` | - | FileAsset | `fetch_file_result_napi.cpp` | 获取第一个对象 |
| `getNextObject()` | - | FileAsset | `fetch_file_result_napi.cpp` | 获取下一个对象 |
| `getCount()` | - | number | `fetch_file_result_napi.cpp` | 获取总数 |
| `isAfterLast()` | - | boolean | `fetch_file_result_napi.cpp` | 是否遍历完毕 |
| `close()` | - | void | `fetch_file_result_napi.cpp` | 关闭结果集 |

#### Album 类

| JS 方法 | 参数 | 返回值 | C++ 实现 | 说明 |
|--------|------|-------|---------|------|
| `createAlbum()` | name | Album | `album_napi.cpp` | 创建相册 |
| `deleteAlbum()` | - | boolean | `album_napi.cpp` | 删除相册 |
| `getPhotoAssets()` | options | FetchFileResult | `album_napi.cpp` | 获取相册内照片 |
| `commitModify()` | - | boolean | `album_napi.cpp` | 提交修改 |

---

## 2. userfileManager 模块

### 模块注册

**文件**: `frameworks/js/src/native_module_ohos_userfile_manager.cpp`

```cpp
// 行 51: 模块注册
napi_module_register(&g_userFileManagerModule);
```

**模块名**: `userfileManager`

### API 清单

| JS 方法 | 参数 | 返回值 | 说明 |
|--------|------|-------|------|
| `getMediaAssets()` | options | FetchFileResult | 获取媒体资产 |
| `createAsset()` | uri | FileAsset | 创建资产 |
| `deleteAssets()` | uris | boolean | 删除资产 |
| `moveAssets()` | sourceUri, destUri | FetchFileResult | 移动资产 |
| `copyAssets()` | sourceUri, destUri | FetchFileResult | 复制资产 |

---

## 3. photoAccessHelper 模块

### 模块注册

**文件**: `frameworks/js/src/native_module_ohos_photoaccess_helper.cpp`

```cpp
// 行 87: 模块注册
napi_module_register(&g_photoAccessHelperModule);
```

**模块名**: `photoAccessHelper`

### API 清单

| JS 方法 | 参数 | 返回值 | 权限要求 | 说明 |
|--------|------|-------|---------|------|
| `getPhotoAssets()` | options | FetchFileResult | READ_IMAGEVIDEO | 获取照片 |
| `createPhotoAsset()` | uri | PhotoAsset | WRITE_IMAGEVIDEO | 创建照片 |
| `deletePhotoAssets()` | uris | boolean | WRITE_IMAGEVIDEO | 删除照片 |
| `modifyPhotoAsset()` | uri, data | boolean | WRITE_IMAGEVIDEO | 修改照片 |
| `openPhoto()` | uri, mode | number | READ_IMAGEVIDEO | 打开照片 |

---

## 4. 权限检查机制

### 权限检查点

**文件**: `frameworks/js/src/moving_photo_napi.cpp:619`

```cpp
static bool result = (AccessTokenKit::VerifyAccessToken(
    IPCSkeleton::GetSelfTokenID(), 
    PERM_READ_IMAGEVIDEO
) == PERMISSION_GRANTED);
```

**文件**: `interfaces/kits/cj/src/media_asset_manager_ffi.cpp:98-99`

```cpp
AccessTokenID tokenCaller = IPCSkeleton::GetSelfTokenID();
int result = AccessTokenKit::VerifyAccessToken(tokenCaller, PERM_READ_IMAGEVIDEO);
```

### 权限清单

| 权限名称 | 用途 | 保护级别 | 使用 API |
|---------|------|---------|---------|
| `ohos.permission.READ_IMAGEVIDEO` | 读取图片和视频 | normal | PhotoAccessHelper |
| `ohos.permission.WRITE_IMAGEVIDEO` | 写入图片和视频 | normal | PhotoAccessHelper |
| `ohos.permission.READ_MEDIA` | 读取媒体文件 | normal | UserFileManager |
| `ohos.permission.WRITE_MEDIA` | 写入媒体文件 | normal | UserFileManager |

---

## 5. 错误码参考

### 通用错误码

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| 401 | `JS_ERR_PARAMETER_INVALID` | 参数无效 |
| 14000001 | `EC_INVALID_ARGUMENT` | 非法参数 |
| 14000002 | `EC_OPERATION_NOT_SUPPORTED` | 操作不支持 |
| 14000003 | `EC_PERMISSION_DENIED` | 权限被拒绝 |
| 14000004 | `EC_FILE_NOT_FOUND` | 文件不存在 |
| 14000005 | `EC_DEVICE_NOT_FOUND` | 设备不存在 |
| 14000011 | `EC_ALLOC_FAILED` | 内存分配失败 |

### 权限错误

| 错误码 | 说明 |
|-------|------|
| 201 | 权限检查失败 |
| 202 | 非系统应用调用系统 API |

---

## 6. 参数校验模式

### N-API 参数校验宏

**文件**: `interfaces/kits/js/include/napi/medialibrary_napi_utils.h`

```cpp
// 行 39-45: 参数校验宏
#define CHECK_ARGS_WITH_MESSAGE(env, cond, msg)                 \
    do {                                                            \
        if (!(cond)) {                                    \
            NapiError::ThrowError(env, JS_ERR_PARAMETER_INVALID, __FUNCTION__, __LINE__, msg); \
            return nullptr;                                          \
        }                                                           \
    } while (0)

// 行 47-53: 条件校验宏
#define CHECK_COND_WITH_MESSAGE(env, cond, msg)                 \
    do {                                                            \
        if (!(cond)) {                                    \
            NapiError::ThrowError(env, OHOS_INVALID_PARAM_CODE, __FUNCTION__, __LINE__, msg); \
            return nullptr;                                          \
        }                                                           \
    } while (0)
```

### 典型校验流程

```cpp
napi_value MediaLibraryNapi::GetMediaAssets(napi_env env, napi_callback_info info) {
    // 1. 获取参数
    size_t argc = 1;
    napi_value argv[1];
    napi_value thisVar;
    napi_get_cb_info(env, info, &argc, argv, &thisVar, nullptr);

    // 2. 参数类型校验
    napi_valuetype valueType;
    napi_typeof(env, argv[0], &valueType);
    if (valueType != napi_object) {
        // 抛出参数错误
        napi_throw_type_error(env, nullptr, "Expected object as first argument");
        return nullptr;
    }

    // 3. 可选参数处理
    FetchOptions options;
    if (argc > 0) {
        // 解析 options 对象
        ParseFetchOptions(env, argv[0], options);
    }

    // 4. 返回结果
    return CreateFetchResult(env, options);
}
```

---

## 7. 异步模式

### Promise 模式

```cpp
napi_value AsyncQuery(napi_env env, napi_callback_info info) {
    // 1. 创建异步上下文
    auto asyncContext = std::make_unique<AsyncContext>();
    
    // 2. 创建 Promise
    napi_value promise;
    napi_create_promise(env, &asyncContext->deferred, &promise);
    
    // 3. 创建异步工作
    napi_create_async_work(
        env, 
        nullptr,  // executor
        [](napi_env env, void* data) {
            // 执行耗时操作
            auto* ctx = static_cast<AsyncContext*>(data);
            ctx->result = DoQuery();
        },
        [](napi_env env, napi_status status, void* data) {
            // 完成回调
            auto* ctx = static_cast<AsyncContext*>(data);
            napi_resolve_promise(env, ctx->deferred, ctx->result);
            delete ctx;
        },
        asyncContext.get(),
        &asyncContext->work
    );
    
    // 4. 队列执行
    napi_queue_async_work(env, asyncContext->work);
    asyncContext.release(); // 所有权转移
    
    return promise;
}
```

### Callback 模式

```cpp
napi_value AsyncWithCallback(napi_env env, napi_callback_info info) {
    size_t argc = 2;
    napi_value argv[2];
    napi_value thisVar;
    napi_get_cb_info(env, info, &argc, argv, &thisVar, nullptr);

    // 1. 提取 callback
    napi_valuetype type;
    napi_typeof(env, argv[1], &type);
    if (type != napi_function) {
        napi_throw_type_error(env, nullptr, "Second argument must be a function");
        return nullptr;
    }

    napi_ref callbackRef;
    napi_create_reference(env, argv[1], 1, &callbackRef);

    // 2. 创建异步工作（带 callback）
    napi_create_async_work(
        env, 
        argv[1],  // callback 作为最后一个参数
        [](napi_env env, void* data) {
            // 执行操作
        },
        [](napi_env env, napi_status status, void* data) {
            auto* ctx = static_cast<AsyncContext*>(data);
            napi_call_function(env, nullptr, ctx->callback, 1, &ctx->result, nullptr);
            napi_delete_reference(env, ctx->callback);
        },
        asyncContext.get(),
        &asyncContext->work
    );

    napi_queue_async_work(env, asyncContext->work);
    return nullptr;
}
```

---

## 8. 接口头文件索引

### 核心头文件

| 头文件 | 用途 | 关键类 |
|-------|------|-------|
| `file_asset_napi.h` | FileAsset N-API | FileAssetNapi |
| `fetch_file_result_napi.h` | FetchResult N-API | FetchFileResultNapi |
| `album_napi.h` | Album N-API | AlbumNapi |
| `smart_album_napi.h` | SmartAlbum N-API | SmartAlbumNapi |
| `media_library_napi.h` | MediaLibrary N-API | MediaLibraryNapi |
| `photo_album_napi.h` | PhotoAlbum N-API | PhotoAlbumNapi |
| `media_asset_manager_napi.h` | AssetManager N-API | MediaAssetManagerNapi |
| `moving_photo_napi.h` | MovingPhoto N-API | MovingPhotoNapi |

### 工具头文件

| 头文件 | 用途 |
|-------|------|
| `napi/medialibrary_napi_utils.h` | N-API 工具宏 |
| `napi_error.h` | 错误处理 |
| `napi/medialibrary_napi_log.h` | 日志宏 |

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构 |
| [02_Architecture](02_Architecture.md) | 架构设计 |
| [04_Inner_API](04_Inner_API.md) | 内部 API |
| [06_Security_Review](06_Security_Review.md) | 安全评审 |
