# 配置开关与宏定义

## 编译期宏定义

### 全局宏定义

| 宏 | 定义位置 | 值 | 说明 |
|----|----------|-----|------|
| `ENABLE_WINDOW` | 所有 BUILD.gn | `1` | 启用窗口系统支持 |
| `ABILITY_WINDOW_SUPPORT` | 所有 BUILD.gn | - | Ability 窗口支持 |
| `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` | gallery, launcher, setting | - | 包管理器支持 |
| `ENABLE_PASSTHROUGH_SAMPLE` | media/player_sample | 条件定义 | 透传模式示例 |

### 代码中使用

```cpp
// cameraApp/cameraApp/src/main/cpp/camera_ability_slice.cpp
#ifdef ENABLE_WINDOW
    // 窗口相关代码
#endif
```

---

## 路径配置

### 存储路径常量

**文件**: `gallery/include/gallery_config.h`

| 常量 | 值 | 说明 |
|------|-----|------|
| `PHOTO_DIRECTORY` | `"/userdata/photo/"` | 照片存储目录 |
| `THUMBNAIL_DIRECTORY` | `"/userdata/photo/thumb/"` | 缩略图存储目录 |
| `VIDEO_SOURCE_DIRECTORY` | `"/userdata/video/"` | 视频存储目录 |

**文件**: `cameraApp/cameraApp/src/main/cpp/ui_config.h`

| 常量 | 值 | 说明 |
|------|-----|------|
| `PHOTO_PATH` | `"/userdata/photo/"` | 照片路径 |
| `THUMB_PATH` | `"/userdata/photo/thumb/"` | 缩略图路径 |
| `VIDEO_PATH` | `"/userdata/video/"` | 视频路径 |

---

## UI 配置常量

### 字体与尺寸

**文件**: `cameraApp/cameraApp/src/main/cpp/ui_config.h`

```cpp
#define UI_FONT_SIZE        28
#define UI_BUTTON_WIDTH     120
#define UI_BUTTON_HEIGHT    60
#define UI_IMAGE_PATH       "/storage/app/run/com.huawei.camera/cameraApp/assets/..."
```

### 颜色定义

```cpp
#define UI_COLOR_BACKGROUND 0xFF000000  // 黑色背景
#define UI_COLOR_TEXT       0xFFFFFFFF  // 白色文字
```

---

## 运行时配置

### 权限配置

**文件**: `cameraApp/cameraApp/src/main/config.json`

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `launchType` | `"standard"` | Ability 启动模式 |
| `type` | `"page"` | Ability 类型 |
| `visible` | `true` | 是否可见 |

### 设备支持

**文件**: `cameraApp/cameraApp/src/main/config.json`

```json
"deviceType": [
    "phone",
    "tv",
    "tablet",
    "pc",
    "car",
    "smartWatch",
    "sportsWatch",
    "smartVision"
]
```

---

## 构建配置

### GN 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `enable_media_passthrough_mode` | `false` | 媒体透传模式 |
| `ohos_root_path` | - | OpenHarmony 源码根目录 |
| `root_out_dir` | `out` | 输出目录 |

### 条件编译示例

```gn
# media/BUILD.gn
executable("player_sample") {
  if (enable_media_passthrough_mode == true) {
    defines = [ "ENABLE_PASSTHROUGH_SAMPLE" ]
  }
  # ...
}
```

---

## 相关链接

- [GN 构建系统](../Build_System.md) - 完整构建配置
- [编译产物](../Build_Outputs.md) - 输出说明
