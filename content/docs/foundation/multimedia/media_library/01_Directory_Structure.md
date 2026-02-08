# MediaLibrary 目录结构与模块职责

## 根目录结构

```
/foundation/multimedia/media_library/
├── .gitignore                          # Git 忽略配置
├── BUILD.gn                            # GN 构建入口
├── LICENSE                             # Apache 2.0 许可证
├── OAT.xml                            # 源码扫描配置
├── README.md / README_zh.md            # 项目文档
├── bundle.json                         # Bundle 配置
├── hisysevent.yaml                     # HiSysEvent 事件配置
├── media_library.gni                   # GN 通用配置
├── publicity.xml                       # 公开配置
│
├── common/                            # 公共代码模块
├── figures/                           # 架构图等文档资源
├── MediaLibraryExt/                   # 扩展模块
│
├── frameworks/                        # 框架实现层
├── interfaces/                         # 接口定义层
├── sa_profile/                        # System Ability 配置
├── services/                          # 服务实现层
└── tools/                             # 工具集
```

---

## 目录职责说明

### 📁 common/ - 公共代码模块

**职责**: 提供跨模块复用的公共功能

| 子目录 | 职责 | 关键内容 |
|-------|------|---------|
| `media_ipc_common/` | IPC 公共模块 | IPC 客户端、服务端通用代码 |
| `utils/` | 公共工具 | 字符串处理、URI 处理、路径处理 |

**证据来源**:
- `common/media_ipc_common/src/`
- `common/utils/src/`

---

### 📁 frameworks/ - 框架实现层

**职责**: 实现 N-API、Native API 和服务框架

#### frameworks/ani/
**职责**: ANI (ArkNative Interface) 接口实现

#### frameworks/client/
**职责**: 客户端封装实现

#### frameworks/innerimpl/
**职责**: 内部实现细节

#### frameworks/innerkitsimpl/
**职责**: 内部 Native API 实现

```
innerkitsimpl/
└── media_library/
    ├── media_library_manager/       # 媒体库管理器
    ├── media_permission_helper/     # 权限助手
    ├── medialibrary_data_extension/  # 数据扩展
    └── ...
```

#### frameworks/js/
**职责**: JS N-API 实现

```
js/src/
├── native_module_ohos_medialibrary.cpp    # 主模块注册 ⭐
├── native_module_ohos_userfile_manager.cpp # 用户文件管理
├── native_module_ohos_photoaccess_helper.cpp # 照片访问
├── media_library_napi.cpp                  # N-API 实现
├── file_asset_napi.cpp                     # FileAsset 封装
├── album_napi.cpp                          # Album 封装
├── photopickercomponent.cpp                # 照片选择器
└── sendable/                              # 可分享模块
```

**证据来源**:
- `frameworks/js/src/native_module_ohos_medialibrary.cpp:79` - N-API 注册

#### frameworks/native/
**职责**: Native 接口实现

#### frameworks/services/
**职责**: 服务框架实现

```
services/
├── media_albums_manager/        # 相册管理服务
├── media_assets_manager/       # 资产管理服务
├── media_cloud_sync/           # 云同步服务
├── media_permission/           # 权限服务
├── media_thumbnail/            # 缩略图服务
├── media_notification/          # 通知服务
├── media_analysis_data_manager/# 分析数据服务
└── ...
```

#### frameworks/utils/
**职责**: 框架工具类

---

### 📁 interfaces/ - 接口定义层

**职责**: 定义对外和内部的 API 接口

#### interfaces/inner_api/
**职责**: 内部 Native API 接口定义

```
inner_api/
├── media_library_helper/       # 媒体库助手接口
│   └── include/
│       ├── media_asset.h       # 资产接口
│       ├── album_asset.h       # 相册接口
│       ├── photo_album.h       # 照片相册接口
│       └── ...
├── analysis_data_kits/         # 分析数据接口
└── media_permission_helper/    # 权限助手接口
```

**稳定性标注**:
- 🟡 **内部接口**: 仅限框架内部使用
- 路径: `interfaces/inner_api/`

#### interfaces/kits/
**职责**: 外部 API 接口定义

```
kits/
├── js/                         # JS 接口
│   └── include/
│       ├── napi/               # N-API 工具类
│       ├── file_asset_napi.h  # FileAsset 导出
│       ├── album_napi.h        # Album 导出
│       ├── media_library_napi.h # 媒体库导出
│       └── ...
└── c/                          # C 语言接口
    ├── media_asset_capi.h      # C API
    ├── media_asset_manager_capi.h
    └── ...
```

**稳定性标注**:
- 🟢 **公开 API**: 外部应用可用
- 路径: `interfaces/kits/`

#### interfaces/kits/cj/
**职责**: CJ (Cross-platform JavaScript) 接口

---

### 📁 services/ - 服务实现层

**职责**: System Ability 服务实现

| 服务模块 | 职责 | 关键文件 |
|---------|------|---------|
| `media_assets_manager/` | 媒体资产 CRUD | controller/ |
| `media_albums_manager/` | 相册管理 | controller/ |
| `media_cloud_sync_service/` | 云同步 | controller/ |
| `media_analysis_data_manager/` | 数据分析 | controller/ |
| `media_thumbnail/` | 缩略图生成 | - |
| `media_permission/` | 权限管理 | - |
| `media_scanner/` | 媒体扫描 | - |
| `media_mtp/` | MTP 协议支持 | - |

**证据来源**:
- `services/media_assets_manager/include/controller/media_assets_controller_service.h`
- `services/media_albums_manager/include/controller/media_albums_controller_service.h`

---

### 📁 sa_profile/ - SA 配置

**职责**: System Ability 配置文件

---

### 📁 tools/ - 工具集

**职责**: 开发、调试工具

| 工具 | 职责 |
|-----|------|
| `medialibrary_tool/` | 媒体库命令行工具 |
| `medialibrary_scanner/` | 媒体扫描工具 |

---

## 模块依赖关系

```
应用层
  │
  ▼
┌──────────────────────────────────────────────────────────────┐
│                    interfaces/kits/                          │
│              (JS N-API, C API - 公开接口)                    │
└──────────────────────────────────────────────────────────────┘
  │
  │ 调用
  ▼
┌──────────────────────────────────────────────────────────────┐
│                  frameworks/js/                              │
│                  (N-API 实现层)                               │
└──────────────────────────────────────────────────────────────┘
  │
  │ 调用
  ▼
┌──────────────────────────────────────────────────────────────┐
│              frameworks/native/ + innerkitsimpl/             │
│                  (Native API 实现层)                         │
└──────────────────────────────────────────────────────────────┘
  │
  │ IPC 调用
  ▼
┌──────────────────────────────────────────────────────────────┐
│                      services/                               │
│              (System Ability 服务)                           │
└──────────────────────────────────────────────────────────────┘
  │
  │ RDB 操作
  ▼
┌──────────────────────────────────────────────────────────────┐
│                 relational_store                             │
│                    (数据持久化)                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 模块职责总结

| 模块层级 | 稳定性 | 职责 |
|---------|--------|------|
| `interfaces/kits/` | 🟢 稳定 | 对外 JS/C API |
| `frameworks/js/` | 🟢 稳定 | N-API 胶水层 |
| `frameworks/native/` | 🟡 内部 | Native 接口 |
| `frameworks/innerkitsimpl/` | 🔴 内部 | 内部实现 |
| `services/` | 🔴 内部 | System Ability |
| `interfaces/inner_api/` | 🔴 内部 | 内部接口定义 |

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [02_Architecture](02_Architecture.md) | 架构设计 |
| [03_N-API_Reference](03_N-API_Reference.md) | N-API 接口 |
| [04_Inner_API](04_Inner_API.md) | 内部 API |
