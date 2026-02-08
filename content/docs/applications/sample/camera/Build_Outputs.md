# 编译产物

## 产物类型

本项目编译产生以下类型的产物：

| 类型 | 扩展名 | 用途 | 示例 |
|------|--------|------|------|
| **HAP 包** | `.hap` | OpenHarmony 应用包 | `cameraApp.hap` |
| **共享库** | `.so` | Native 动态库 | `libcameraApp.so` |
| **可执行文件** | 无扩展名 | 独立程序 | `camera_sample` |

---

## HAP 应用包

### 产物清单

| HAP 包 | 源模块 | BUILD.gn target | 签名证书 |
|--------|--------|-----------------|----------|
| `cameraApp.hap` | cameraApp | `cameraApp_hap` | `cert/camera_AppProvision_Release.p7b` |
| `gallery.hap` | gallery | `gallery_hap` | `cert/gallery_AppProvision_Release.p7b` |
| `launcher.hap` | launcher | `launcher_hap` | `cert/com.huawei.launcher_AppProvision_release.p7b` |
| `setting.hap` | setting | `setting_hap` | `cert/com.huawei.setting_AppProvision_release.p7b` |

### HAP 包结构

```
cameraApp.hap (ZIP 格式)
├── libcameraApp.so          # Native 共享库
├── config.json              # 应用配置
├── resources/               # 资源文件
│   ├── base/media/          # 图片资源
│   └── rawfile/             # 原始文件
└── signature/               # 签名信息
```

---

## Native 共享库 (.so)

### 产物清单

| 库文件 | 源模块 | 类型 | 依赖库 |
|--------|--------|------|--------|
| `libcameraApp.so` | cameraApp | shared_library | camera_lite, recorder_lite, surface, ui_lite |
| `libgallery.so` | gallery | shared_library | player_lite, recorder_lite, surface, ui_lite |
| `liblauncher.so` | launcher | shared_library | bundle, surface, ui_lite |
| `libsetting.so` | setting | shared_library | wpa_client, begetutil, pms_client, surface, ui_lite |

---

## 独立可执行文件

### 产物清单

| 可执行文件 | 源文件 | BUILD.gn target | 输出目录 | 依赖 |
|------------|--------|-----------------|----------|------|
| `camera_sample` | `media/camera_sample.cpp` | `camera_sample` | `$root_out_dir/dev_tools/` | camera_lite, recorder_lite |
| `player_sample` | `media/player_sample.cpp` | `player_sample` | `$root_out_dir/dev_tools/` | player_lite |
| `audio_capture_sample` | `media/audio_capture_sample.cpp` | `audio_capture_sample` | `$root_out_dir/dev_tools/` | audio_capturer_lite |

### 用途说明

| 可执行文件 | 用途 | 运行方式 |
|------------|------|----------|
| `camera_sample` | 独立相机示例 | `./camera_sample` |
| `player_sample` | 独立播放器示例 | `./player_sample <filepath>` |
| `audio_capture_sample` | 独立录音示例 | `./audio_capture_sample` |

**注意**: 这些可执行文件不打包为 HAP，需通过命令行在设备上运行。

---

## 安装路径

### HAP 安装路径

```
/system/app/{bundleName}/
├── {bundleName}.hap         # HAP 包
├── lib/                     # Native 库目录
│   └── lib{bundleName}.so
└── ...

# 示例
/system/app/com.huawei.camera/
├── com.huawei.camera.hap
├── lib/
│   └── libcameraApp.so
```

### 媒体文件路径

```
/userdata/photo/               # 照片存储
/userdata/video/               # 视频存储
```

**代码引用** (`gallery/include/gallery_config.h`):
```cpp
#define PHOTO_DIRECTORY "/userdata/photo/"
#define VIDEO_SOURCE_DIRECTORY "/userdata/video/"
```

---

## 运行时加载关系

### 库加载顺序

```
libcameraApp.so (应用层)
    ├── libcamera_lite.so (相机服务)
    │   └── libsurface.so (图形缓冲区)
    ├── librecorder_lite.so (录像服务)
    │   └── libsurface.so
    ├── libui_lite.so (UI 框架)
    │   └── libgraphic_utils_lite.so
    └── libability_lite.so (Ability 框架)
        └── libsamgr_lite.so (服务管理)
```

---

## 产物大小估算

| 产物 | 预计大小 | 说明 |
|------|----------|------|
| `cameraApp.hap` | ~500KB-1MB | 含资源文件 |
| `libcameraApp.so` | ~100-200KB | stripped |
| `gallery.hap` | ~500KB-1MB | 含资源文件 |
| `libgallery.so` | ~100-200KB | stripped |
| `launcher.hap` | ~500KB-1MB | 含资源文件 |
| `liblauncher.so` | ~100-200KB | stripped |
| `setting.hap` | ~500KB-1MB | 含资源文件 |
| `libsetting.so` | ~150-250KB | 含 WiFi 代码 |
| `camera_sample` | ~50KB | 独立可执行文件 |
| `player_sample` | ~50KB | 独立可执行文件 |
| `audio_capture_sample` | ~50KB | 独立可执行文件 |

---

## 相关链接

- [GN 构建系统](./Build_System.md) - 构建配置详情
- [安全风险分析](./Security.md) - 产物安全考虑
