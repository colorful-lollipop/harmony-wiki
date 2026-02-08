# MediaLibrary 内部 API

## 内部 API 概述

MediaLibrary 的内部 API 主要分为以下几类：

| API 类型 | 路径 | 稳定性 | 使用范围 |
|---------|------|--------|---------|
| **Native API** | `interfaces/inner_api/` | 🔴 内部 | 框架内部 |
| **Service API** | `services/` | 🔴 内部 | 服务间通信 |
| **Helper API** | `frameworks/innerkitsimpl/` | 🔴 内部 | 实现细节 |

---

## 1. Native API (inner_api)

### 媒体库助手接口

**头文件路径**: `interfaces/inner_api/media_library_helper/include/`

| 头文件 | 主要类/接口 | 职责 |
|-------|------------|------|
| `media_asset.h` | MediaAsset | 媒体资产核心接口 |
| `album_asset.h` | AlbumAsset | 相册资产接口 |
| `photo_album.h` | PhotoAlbum | 照片相册接口 |
| `file_asset.h` | FileAsset | 文件资产接口 |
| `fetch_result.h` | FetchResult<T> | 查询结果模板 |
| `media_library_manager.h` | MediaLibraryManager | 媒体库管理器 |
| `media_asset_manager.h` | MediaAssetManager | 资产管理器 |

### 数据类型接口

| 头文件 | 数据类型 | 用途 |
|-------|---------|------|
| `media_column.h` | MediaColumn | 媒体数据列定义 |
| `photo_album_column.h` | PhotoAlbumColumn | 相册列定义 |
| `audio_column.h` | AudioColumn | 音频列定义 |
| `medialibrary_db_const.h` | MediaLibraryDbConst | 数据库常量 |

---

## 2. Service API

### System Ability 接口

#### MediaAssetsControllerService

**头文件**: `services/media_assets_manager/include/controller/media_assets_controller_service.h`

```cpp
// 远程请求处理
int32_t OnRemoteRequest(uint32_t code, MessageParcel &data, 
                        MessageParcel &reply, MessageOption &option) override;

// 资产操作
int32_t CreateAsset(const AssetAbilityInfo &info, int &assetId);
int32_t ModifyAsset(const AssetAbilityInfo &info);
int32_t DeleteAsset(const int assetId);
int32_t QueryAssets(const QueryArgs &args, QueryResult &result);
```

#### MediaAlbumsControllerService

**头文件**: `services/media_albums_manager/include/controller/media_albums_controller_service.h`

```cpp
// 远程请求处理
int32_t OnRemoteRequest(uint32_t code, MessageParcel &data,
                        MessageParcel &reply, MessageOption &option) override;

// 相册操作
int32_t CreateAlbum(const AlbumAbilityInfo &info, int &albumId);
int32_t DeleteAlbum(const int albumId);
int32_t ModifyAlbum(const AlbumAbilityInfo &info);
int32_t QueryAlbums(const QueryArgs &args, QueryResult &result);
```

---

## 3. 权限 Helper API

### MediaPermissionHelper

**头文件**: `interfaces/inner_api/media_permission_helper/include/media_permission_helper.h`

```cpp
class MediaPermissionHelper {
public:
    // 检查 URI 权限
    bool CheckPhotoUriPermission(const std::string &uri, 
                                 AccessTokenID tokenId);
    
    // 授予 URI 权限
    bool GrantPhotoUriPermission(const std::string &uri,
                                  AccessTokenID targetTokenId,
                                  int permissionType);
    
    // 取消 URI 权限
    bool CancelPhotoUriPermission(const std::string &uri,
                                   AccessTokenID targetTokenId,
                                   int permissionType);
    
    // 获取权限列表
    std::vector<std::string> GetPermissions(AccessTokenID tokenId);
};
```

**证据来源**:
- `interfaces/inner_api/media_permission_helper/include/media_permission_helper.h`
- `frameworks/innerkitsimpl/media_permission_helper/src/media_permission_helper.cpp`

---

## 4. 模块依赖关系

### 依赖方向图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         接口层 (interfaces)                          │
│  ┌──────────────────────┐  ┌──────────────────────┐               │
│  │   inner_api/         │  │      kits/           │               │
│  │  (内部 Native API)   │  │   (公开 N-API)       │               │
│  └──────────┬───────────┘  └───────────┬──────────┘               │
│             │                          │                           │
│             │ 使用                     │ 使用                      │
│             ▼                          ▼                           │
│  ┌─────────────────────────────────────────────────────────────────┐
│  │                     框架层 (frameworks)                          │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌───────────────┐  │
│  │  │  innerkitsimpl/  │  │     js/         │  │   services/   │  │
│  │  └────────┬────────┘  └────────┬────────┘  └───────┬───────┘  │
│  │           │                   │                    │          │
│  │           └───────────────────┼────────────────────┘          │
│  └──────────────────────────────┼────────────────────────────────┘
│                                 │
│                                 ▼
│  ┌─────────────────────────────────────────────────────────────────┐
│  │                     服务层 (services)                            │
│  │  ┌──────────────────┐  ┌──────────────────┐                  │
│  │  │ MediaAssetsService│  │ MediaAlbumsService│                  │
│  │  └──────────────────┘  └──────────────────┘                  │
│  └─────────────────────────────────────────────────────────────────┘
```

### 关键依赖

| 源模块 | 目标模块 | 依赖类型 | 说明 |
|-------|---------|---------|------|
| `js/` | `innerkitsimpl/` | 接口调用 | N-API → Native |
| `innerkitsimpl/` | `services/` | IPC | Native → SA |
| `services/` | `inner_api/` | 数据类型 | SA → 接口定义 |

---

## 5. 稳定性标注

### 接口稳定性分级

| 标注 | 含义 | 使用建议 |
|-----|------|---------|
| 🟢 **kits/** | 公开 API | 外部应用可安全使用 |
| 🟡 **innerkitsimpl/** | 内部 API | 仅框架内部使用 |
| 🔴 **services/** | 服务实现 | 禁止直接调用 |
| 🔴 **inner_api/** | 内部接口 | 禁止外部使用 |

### 稳定性证据

| 目录 | 证据 | 稳定性 |
|-----|------|--------|
| `interfaces/kits/` | 路径命名 `kits` 表示对外 | 🟢 稳定 |
| `interfaces/inner_api/` | 路径命名 `inner_api` 表示内部 | 🔴 内部 |
| `frameworks/innerkitsimpl/` | `impl` 后缀表示实现细节 | 🔴 内部 |

---

## 6. 可替换点

### 插件化设计

| 替换点 | 接口 | 替换方式 |
|-------|------|---------|
| **存储后端** | `MediaLibraryDataAbility` | RDB 替换 |
| **云同步** | `CloudSyncService` | 同步策略替换 |
| **缩略图生成** | `ThumbnailService` | 算法替换 |
| **权限校验** | `MediaPermissionHelper` | 策略替换 |

### 扩展点

| 扩展点 | 接口 | 说明 |
|-------|------|------|
| **自定义元数据** | `PhotoAssetCustomRecord` | 扩展照片属性 |
| **自定义相册** | `SmartAlbumAsset` | AI 分类相册 |
| **分析数据** | `AnalysisDataManager` | ML 分析结果 |

---

## 7. 内部错误码

### 服务层错误码

| 错误码 | 常量名 | 范围 |
|-------|--------|------|
| 401 | `E_INVALID_ARGUMENTS` | 参数错误 |
| 1001 | `E_SERVICE_ERROR` | 服务内部错误 |
| 1002 | `E_DATABASE_ERROR` | 数据库错误 |
| 1003 | `E_PERMISSION_DENIED` | 权限拒绝 |
| 1004 | `E_FILE_NOT_FOUND` | 文件不存在 |
| 1005 | `E_URI_INVALID` | URI 非法 |

---

## 8. 关键数据结构

### AssetAbilityInfo

```cpp
struct AssetAbilityInfo {
    std::string name;           // 文件名
    std::string uri;            // 文件 URI
    std::string path;           // 文件路径
    int64_t size;              // 文件大小
    MediaType type;             // 媒体类型
    int32_t orientation;       // 方向
    std::string mimeType;      // MIME 类型
    time_t dateAdded;          // 添加时间
    time_t dateModified;       // 修改时间
    std::string albumId;       // 相册 ID
};
```

### AlbumAbilityInfo

```cpp
struct AlbumAbilityInfo {
    std::string albumName;      // 相册名
    std::string path;           // 路径
    AlbumType type;            // 相册类型
    int32_t count;             // 文件数
    time_t dateModified;       // 修改时间
};
```

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构 |
| [02_Architecture](02_Architecture.md) | 架构设计 |
| [03_N-API_Reference](03_N-API_Reference.md) | N-API 接口 |
| [05_Build_System](05_Build_System.md) | 构建系统 |
