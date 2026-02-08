# MediaLibrary 项目概览

## 项目定位

**MediaLibrary** (medialibrary_standard) 是 OpenHarmony 系统的核心多媒体媒体库服务，提供媒体文件元数据查询和管理能力。

### 核心能力

| 能力 | 描述 | 典型场景 |
|-----|------|---------|
| **媒体查询** | 查询音频、视频、图片文件元数据 | 获取歌曲信息、照片属性 |
| **相册管理** | 图片/视频相册的 CRUD 操作 | 创建相册、整理照片 |
| **文件操作** | 媒体文件的创建、重命名、复制、删除 | 文件整理、备份 |
| **云同步** | 云端媒体文件同步 | 跨设备媒体共享 |
| **智能相册** | 基于 AI 的智能分类 | 人脸相册、地点相册 |

### 系统能力依赖

```json
// bundle.json
"syscap": [
  "SystemCapability.FileManagement.UserFileManager.Core",
  "SystemCapability.FileManagement.UserFileManager.DistributedCore",
  "SystemCapability.FileManagement.PhotoAccessHelper.Core"
]
```

---

## 项目边界

### 对外接口

| 接口类型 | 命名空间 | 说明 |
|---------|---------|------|
| **JS N-API** | `multimedia.mediaLibrary` | 主媒体库 API |
| **JS N-API** | `userfileManager` | 用户文件管理 API |
| **JS N-API** | `photoAccessHelper` | 照片访问助手 API |
| **C API** | `@ohos.mediaLibrary` | C 语言接口 |

### 内部接口

| 接口类型 | 路径 | 说明 |
|---------|------|------|
| **Native API** | `interfaces/inner_api/` | 内部 Native 接口 |
| **Service API** | `services/` | 系统服务接口 |

### 不对外暴露

- 数据库直接访问
- 内部实现细节
- 测试框架代码

---

## 运行环境

### 硬件要求

| 组件 | 要求 |
|-----|------|
| **ROM** | 10444 KB |
| **RAM** | 35093 KB |

### 系统要求

- **适配系统类型**: small, standard
- **依赖子系统**: multimedia (子系统)

### 关键运行时依赖

| 依赖组件 | 用途 | 重要性 |
|---------|------|-------|
| **napi** | Native API 框架 | ⭐ 必需 |
| **ipc** | IPC 通信 | ⭐ 必需 |
| **safwk** | System Ability Framework | ⭐ 必需 |
| **relational_store** | 关系型数据库 | ⭐ 必需 |
| **data_share** | 数据共享 | ⭐ 必需 |
| **access_token** | 权限管理 | ⭐ 必需 |

---

## 关键概念

### 核心数据类型

| 类型 | 说明 | 所在文件 |
|-----|------|---------|
| **FileAsset** | 媒体文件资产 | `file_asset.h` |
| **PhotoAsset** | 照片资产 | `photo_asset.h` |
| **AudioAsset** | 音频资产 | `audio_asset.h` |
| **VideoAsset** | 视频资产 | `video_asset.h` |
| **AlbumAsset** | 相册资产 | `album_asset.h` |
| **PhotoAlbum** | 照片相册 | `photo_album.h` |
| **SmartAlbumAsset** | 智能相册资产 | `smart_album_asset.h` |

### 媒体类型枚举

```cpp
// medialibrary_type_const.h
enum MediaType {
    MEDIA_TYPE_IMAGE = 1,      // 图片
    MEDIA_TYPE_VIDEO = 2,      // 视频
    MEDIA_TYPE_AUDIO = 3,      // 音频
    // ...
};
```

### URI 格式

```
datashare:///media/{type}/{id}
```

---

## 系统集成

### System Ability

| SA 名称 | 功能 | 代码位置 |
|--------|------|---------|
| **MediaAssetsManager** | 媒体资产管理 | `services/media_assets_manager/` |
| **MediaAlbumsManager** | 相册管理 | `services/media_albums_manager/` |

### 权限模型

| 权限 | 保护级别 | 使用场景 |
|-----|---------|---------|
| `ohos.permission.READ_IMAGEVIDEO` | normal | 读取媒体文件 |
| `ohos.permission.WRITE_IMAGEVIDEO` | normal | 写入媒体文件 |
| `ohos.permission.FILE_ACCESS_MANAGER` | system_basic | 文件访问管理 |

---

## 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Application)                    │
├─────────────────────────────────────────────────────────────┤
│  JS N-API                                                   │
│  ┌─────────────────┐ ┌─────────────────┐ ┌───────────────┐ │
│  │multimedia.      │ │userfileManager  │ │photoAccess-   │ │
│  │mediaLibrary     │ │                 │ │Helper         │ │
│  └─────────────────┘ └─────────────────┘ └───────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  Native/C API                                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                  @ohos.mediaLibrary                  │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  服务层 (Services)                                          │
│  ┌──────────────────┐ ┌──────────────────┐               │
│  │MediaAssetsManager│ │MediaAlbumsManager│               │
│  │     Service      │ │     Service      │               │
│  └──────────────────┘ └──────────────────┘               │
├─────────────────────────────────────────────────────────────┤
│  数据层 (Data)                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            Relational Store (RDB)                   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [01_Directory_Structure](01_Directory_Structure.md) | 详细目录结构 |
| [02_Architecture](02_Architecture.md) | 架构设计 |
| [03_N-API_Reference](03_N-API_Reference.md) | N-API 接口参考 |
| [README](README.md) | Wiki 使用指南 |

---

## 参考资源

- **官方文档**: OpenHarmony MediaLibrary 标准库
- **源码仓库**: gitee.com/openharmony/multimedia_medialibrary_standard
- **Issue 反馈**: 官方 Issue 跟踪系统
