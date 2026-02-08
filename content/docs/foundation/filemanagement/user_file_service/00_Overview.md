# User File Service - 项目概览

## 项目定位

**user_file_service**（公共文件访问框架，FileAccessFramework）是 OpenHarmony 文件管理子系统的核心组件。

### 核心职责

```
┌─────────────────────────────────────────────────────────────────┐
│                     user_file_service                           │
├─────────────────────────────────────────────────────────────────┤
│  上层应用 ←──────→ FileAccessFramework ←──────→ 底层文件服务      │
│  (Stage模型)       (统一抽象接口)            medialibrary         │
│                                                    externalFileManager │
└─────────────────────────────────────────────────────────────────┘
```

**证据**：`README_zh.md:5-7`

### 能力边界

| 能力 | 媒体文件 | 文档文件 | 共享盘 | 外置存储 |
|------|----------|----------|--------|----------|
| 查询 | ✅ | ✅ | ✅ | ✅ |
| 创建 | ✅ | ✅ | ✅ | ✅ |
| 删除 | ✅ | ✅ | ✅ | ✅ |
| 打开 | ✅ | ✅ | ✅ | ✅ |
| 移动 | ✅ | ✅ | ✅ | ✅ |
| 重命名 | ✅ | ✅ | ✅ | ✅ |

**说明**：
- 媒体文件：图片、音频、视频（以相册方式呈现）
- 文档文件：以目录树方式呈现
- 共享盘：局域网共享存储设备
- 外置存储：USB、SD卡等外部存储设备

---

## 系统能力 (SysCap)

| SysCap | 用途 | 必需 |
|--------|------|------|
| `SystemCapability.FileManagement.UserFileService` | 公共文件访问基础能力 | ✅ |
| `SystemCapability.FileManagement.UserFileService.FolderSelection` | 文件夹选择能力 | ⭕ |
| `SystemCapability.FileManagement.CloudDiskManager` | 云盘管理能力 | ⭕ |

**证据**：`bundle.json:15-19`

---

## 运行环境

| 要求 | 说明 |
|------|------|
| 系统类型 | OpenHarmony Standard |
| 架构 | ARM64 (armeabi-v7a 已移除) |
| 模型限制 | 仅支持 Stage 模型 |
| 调用方限制 | 目前仅限文件管理器和文件选择器 |

**证据**：`README_zh.md:38`

---

## 目录结构

```
user_file_service/
├── figures/                      # 架构图等资源
├── frameworks/                   # N-API 实现
│   └── js/napi/
│       ├── file_access_module/   # 文件访问 N-API (file.fileAccess)
│       ├── file_extension_info_module/  # 文件扩展信息
│       └── file_access_ext_ability/     # 文件访问扩展 Ability
├── interfaces/                   # 接口定义
│   ├── inner_api/               # 内部件间接口
│   │   ├── file_access/        # 文件访问内部 API
│   │   └── cloud_disk_kit_inner/  # 云盘管理内部 API
│   └── kits/                   # 应用接口
│       ├── picker/              # 文件选择器
│       ├── taihe/              # ArkUI 组件
│       └── native/             # Native API
│           ├── recent/          # 最近文件
│           ├── trash/          # 回收站
│           └── clouddiskmanager/ # 云盘管理
├── services/                    # 服务实现
│   ├── native/
│   │   ├── file_access_service/ # FileAccess 系统服务 (SA 5010)
│   │   ├── cloud_disk_service/  # 云盘服务
│   │   └── notify_event/       # 通知事件服务
│   ├── rdb_adapter/            # RDB 数据库适配器
│   └── file_extension_hap/     # 外部文件管理器 HAP
├── utils/                       # 工具类
│   ├── file_uri_check.h        # URI 检查
│   ├── file_access_check_util.h # 访问检查
│   └── hilog_wrapper.h         # 日志封装
├── BUILD.gn                     # 构建入口
├── bundle.json                  # 部件描述
└── filemanagement_aafwk.gni    # GN 特性开关
```

**证据**：`README_zh.md:21-35`, `bundle.json`

---

## 模块职责

| 模块 | 职责 | 稳定性 |
|------|------|--------|
| `frameworks/js/napi/*` | N-API 胶水层，JS ↔ C++ 绑定 | 稳定 |
| `interfaces/kits/*` | 应用级 API 导出 | 稳定 |
| `interfaces/inner_api/*` | 内部件间接口 | 半稳定 |
| `services/*` | 核心业务逻辑 | 稳定 |
| `utils/*` | 公共工具 | 稳定 |

---

## 关键依赖

### 外部依赖

| 依赖组件 | 用途 | 来源 |
|----------|------|------|
| `ability_runtime` | Ability 框架 | 系统组件 |
| `bundle_framework` | 包管理 | 系统组件 |
| `ipc` | 进程间通信 | 系统组件 |
| `safwk` | System Ability 框架 | 系统组件 |
| `access_token` | 权限管理 | 系统组件 |
| `os_account` | 多用户管理 | 系统组件 |
| `file_api` | 文件操作基础 API | filemanagement 子系统 |
| `relational_store` | RDB 关系数据库 | 系统组件 |

### 内部依赖

| 被依赖模块 | 依赖模块 | 说明 |
|------------|----------|------|
| `services:file_access_service` | `frameworks/*` | N-API 依赖服务 |
| `interfaces/kits/*` | `services:*` | 上层依赖下层 |

---

## 特性开关

| 开关 | 默认值 | 条件 | 作用 |
|------|--------|------|------|
| `picker_udmf_enabled` | true | - | Picker 使用统一数据管理 (UDMF) |
| `user_file_service_cloud_disk_enable` | false | - | 启用云盘管理功能 |
| `ufs_sandbox_manarer` | false | accesscontrol_sandbox_manager | 启用沙箱管理器 |

**证据**：`filemanagement_aafwk.gni:19-29`

---

## 相关仓库

| 仓库 | 用途 |
|------|------|
| [multimedia_medialibrary_standard](https://gitee.com/openharmony/multimedia_medialibrary_standard) | 媒体库服务 |
| [filemanagement_storage_service](https://gitee.com/openharmony/filemanagement_storage_service) | 存储管理服务 |
| [filemanagement_file_api](https://gitee.com/openharmony/filemanagement_file_api) | 文件访问接口 |
| [account_os_account](https://gitee.com/openharmony/account_os_account) | 多用户管理 |

**证据**：`README_zh.md:41-45`

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 3.1 | - | 当前版本 |
