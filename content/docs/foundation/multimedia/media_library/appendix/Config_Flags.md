# MediaLibrary 关键配置项

## GN 配置项 (media_library.gni)

### 基础配置

| 配置项 | 类型 | 默认值 | 说明 |
|-------|------|-------|------|
| `media_library_link_opt` | bool | false | 是否启用链接优化 |
| `media_library_feature_mtp` | bool | true | MTP 功能开关 |
| `media_library_feature_back_up` | bool | true | 备份功能开关 |
| `media_library_feature_cloud_enhancement` | bool | true | 云增强功能 |

### 云同步配置

| 配置项 | 类型 | 默认值 | 说明 |
|-------|------|-------|------|
| `media_library_feature_cloud_download` | bool | true | 云下载功能 |
| `media_library_cloud_sync_enable` | bool | true | 云同步总开关 |
| `media_library_facard_enable` | bool | true | FACard 功能 |

### 分析功能配置

| 配置项 | 类型 | 默认值 | 说明 |
|-------|------|-------|------|
| `media_library_analysis_data_enable` | bool | true | 分析数据功能 |
| `media_library_feature_custom_restore` | bool | true | 自定义恢复功能 |

### 可选功能配置

| 配置项 | 类型 | 默认值 | 说明 |
|-------|------|-------|------|
| `media_library_feature_device_manager` | bool | false | 设备管理功能 |
| `media_library_feature_hicollie_enable` | bool | false | 性能监控功能 |

---

## Feature 使用示例

### 条件编译

```gn
# BUILD.gn 中使用
if (media_library_cloud_sync_enable) {
  sources += [
    "src/cloud_sync.cpp",
    "src/cloud_download.cpp",
  ]
  
  deps += [
    "//foundation/distributed/device_manager/interfaces/...",
  ]
}
```

### 运行时开关

```cpp
// C++ 中检查
#if defined(MEDIA_LIBRARY_CLOUD_SYNC_ENABLE)
    if (MediaLibraryCloudSync::IsEnabled()) {
        CloudSyncService::Start();
    }
#endif
```

---

## Bundle 配置 (bundle.json)

### 组件配置

```json
{
  "name": "@ohos/media_library",
  "version": "4.0",
  "component": {
    "name": "media_library",
    "subsystem": "multimedia",
    "features": [
      "media_library_link_opt",
      "media_library_feature_mtp",
      "media_library_feature_cloud_enhancement"
    ]
  }
}
```

### 系统能力 (syscap)

| 能力名称 | 用途 |
|---------|------|
| `SystemCapability.FileManagement.UserFileManager.Core` | 用户文件管理核心能力 |
| `SystemCapability.FileManagement.UserFileManager.DistributedCore` | 分布式文件管理能力 |
| `SystemCapability.FileManagement.PhotoAccessHelper.Core` | 照片访问助手核心能力 |

---

## 权限配置

### 声明权限

```json
// module.json5
"requestPermissions": [
  {
    "name": "ohos.permission.READ_IMAGEVIDEO",
    "reason": "Need to read media files",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inuse"
    }
  },
  {
    "name": "ohos.permission.WRITE_IMAGEVIDEO",
    "reason": "Need to write media files"
  }
]
```

### 权限列表

| 权限名称 | 保护级别 | 用途 |
|---------|---------|------|
| `ohos.permission.READ_IMAGEVIDEO` | normal | 读取图片和视频 |
| `ohos.permission.WRITE_IMAGEVIDEO` | normal | 写入图片和视频 |
| `ohos.permission.READ_MEDIA` | normal | 读取媒体文件 |
| `ohos.permission.WRITE_MEDIA` | normal | 写入媒体文件 |
| `ohos.permission.FILE_ACCESS_MANAGER` | system_basic | 文件访问管理 |

---

## 构建配置示例

### 完整 Feature 启用

```bash
# 构建时启用所有功能
hb set
hb build
# 在菜单中选择 media_library 的所有 feature
```

### 最小化构建

```gn
# media_library.gni 自定义
declare_args() {
  media_library_feature_mtp = false
  media_library_feature_cloud_enhancement = false
  media_library_feature_cloud_download = false
  media_library_cloud_sync_enable = false
  media_library_facard_enable = false
  media_library_analysis_data_enable = false
  media_library_feature_custom_restore = false
}
```

---

## 配置验证

### GN 配置检查

```bash
# 生成配置并检查
gn gen out/default --check
gn args out/default --list | grep media_library
```

### Bundle 配置验证

```bash
# 检查 bundle.json 语法
cat bundle.json | python3 -m json.tool
```

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [05_Build_System](05_Build_System.md) | 构建系统 |
| [07_Troubleshooting](07_Troubleshooting.md) | 问题排查 |
