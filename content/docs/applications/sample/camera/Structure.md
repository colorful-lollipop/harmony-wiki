# 目录结构与模块职责

## 顶层目录概览

```
applications/sample/camera/
├── LICENSE                 # Apache 2.0 许可证
├── OAT.xml                 # OpenAtom 合规检查配置
├── README.md               # 英文项目说明
├── README_zh.md            # 中文项目说明
├── bundle.json             # OpenHarmony 组件配置
├── cameraApp/              # 相机应用
├── communication/          # WiFi 通信工具（子模块）
├── figures/                # 文档图片资源
├── gallery/                # 图库应用
├── launcher/               # 桌面启动器
├── media/                  # 媒体示例程序
├── screensaver/            # 屏保应用（子模块，独立仓库）
├── setting/                # 设置应用
└── wiki/                   # 本 Wiki 文档
```

---

## 各模块详细结构

### 1. cameraApp - 相机应用

**路径**: `cameraApp/`  
**类型**: HAP 应用（Native）  
**核心功能**: 相机预览、拍照、录像

```
cameraApp/
├── BUILD.gn                          # GN 构建配置
├── cameraApp/
│   └── src/main/
│       ├── cpp/
│       │   ├── camera_ability.cpp        # Ability 入口 (line 19: REGISTER_AA)
│       │   ├── camera_ability.h          # Ability 声明
│       │   ├── camera_ability_slice.cpp  # 主页面切片
│       │   ├── camera_ability_slice.h    # 页面切片声明
│       │   ├── camera_manager.cpp        # 相机管理器实现
│       │   ├── camera_manager.h          # 相机管理器接口
│       │   ├── event_listener.h          # 事件监听
│       │   └── ui_config.h               # UI 配置/常量
│       ├── config.json               # 应用配置（权限、Ability）
│       └── resources/                # 资源文件（图片、字体）
└── cert/                             # 签名证书
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `CameraAbility` | camera_ability.cpp | Ability 生命周期管理 |
| `CameraAbilitySlice` | camera_ability_slice.cpp/h | UI 主页面、事件处理 |
| `SampleCameraManager` | camera_manager.cpp/h | 相机控制 API 封装 |
| `SampleCameraStateMng` | camera_manager.cpp/h | 相机状态回调处理 |
| `TestFrameStateCallback` | camera_manager.cpp/h | 帧捕获回调 |

**权限声明** (`config.json:46-97`):
- `ohos.permission.CAMERA` - 相机访问
- `ohos.permission.MICROPHONE` - 录音
- `ohos.permission.READ_MEDIA` - 读取媒体
- `ohos.permission.WRITE_MEDIA` - 写入媒体
- `ohos.permission.MODIFY_AUDIO_SETTINGS` - 修改音频设置

---

### 2. gallery - 图库应用

**路径**: `gallery/`  
**类型**: HAP 应用（Native）  
**核心功能**: 图片浏览、缩略图展示、视频播放

```
gallery/
├── BUILD.gn                    # GN 构建配置
├── config.json                 # 应用配置
├── include/                    # 头文件目录
│   ├── event_listener.h
│   ├── gallery_ability.h
│   ├── gallery_ability_slice.h
│   ├── gallery_config.h
│   ├── picture_ability_slice.h
│   └── player_ability_slice.h
├── resources/                  # 资源文件
└── src/                        # 源文件
    ├── gallery_ability.cpp
    ├── gallery_ability_slice.cpp
    ├── picture_ability_slice.cpp
    └── player_ability_slice.cpp
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `GalleryAbility` | gallery_ability.cpp | Ability 入口 |
| `GalleryAbilitySlice` | gallery_ability_slice.cpp | 图库主页面 |
| `PictureAbilitySlice` | picture_ability_slice.cpp | 图片查看页面 |
| `PlayerAbilitySlice` | player_ability_slice.cpp | 视频播放页面 |

**关键路径** (`gallery_config.h`):
- 缩略图目录: `/userdata/photo/`
- 视频目录: `/userdata/video/`

---

### 3. launcher - 桌面启动器

**路径**: `launcher/`  
**类型**: HAP 应用（Native）  
**核心功能**: 桌面展示、应用启动、应用管理

```
launcher/
├── BUILD.gn
├── cert/                       # 签名证书
└── launcher/src/main/
    ├── cpp/                    # 源文件
    │   ├── app_info.cpp        # 应用信息结构
    │   ├── app_info.h
    │   ├── app_manage.cpp      # 应用管理（安装/卸载）
    │   ├── app_manage.h
    │   ├── event_listener.h    # 事件监听
    │   ├── long_press_view.cpp # 长按菜单
    │   ├── long_press_view.h
    │   ├── main_ability.cpp    # Ability 入口
    │   ├── main_ability.h
    │   ├── main_ability_slice.cpp  # 主页面
    │   ├── main_ability_slice.h
    │   ├── native_base.h
    │   ├── swipe_view.cpp      # 滑动视图
    │   ├── swipe_view.h
    │   ├── time_weather_view.cpp   # 时间天气组件
    │   ├── time_weather_view.h
    │   ├── ui_config.h
    │   ├── view_group_page.cpp
    │   └── view_group_page.h
    ├── config.json             # 应用配置
    └── resources/              # 资源
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `MainAbility` | main_ability.cpp | Ability 入口 |
| `MainAbilitySlice` | main_ability_slice.cpp | 桌面主页面 |
| `AppManage` | app_manage.cpp | 应用安装/卸载/查询 |
| `SwipeView` | swipe_view.cpp | 滑动翻页视图 |

---

### 4. setting - 设置应用

**路径**: `setting/`  
**类型**: HAP 应用（Native）  
**核心功能**: 系统设置、权限管理、WiFi设置、显示设置

```
setting/
├── BUILD.gn
├── cert/                       # 签名证书
└── setting/src/main/
    ├── cpp/                    # 源文件
    │   ├── app_ability_slice.cpp   # 应用列表页面
    │   ├── app_ability_slice.h
    │   ├── app_info_ability_slice.cpp  # 应用详情/权限管理
    │   ├── app_info_ability_slice.h
    │   ├── event_listener.h
    │   ├── main_ability_slice.cpp  # 设置主页面
    │   ├── main_ability_slice.h
    │   ├── setting_about_ability_slice.cpp # 关于页面
    │   ├── setting_about_ability_slice.h
    │   ├── setting_display_ability_slice.cpp   # 显示设置
    │   ├── setting_display_ability_slice.h
    │   ├── setting_main_ability.cpp    # Ability 入口
    │   ├── setting_main_ability.h
    │   ├── setting_utils.cpp       # 工具函数
    │   ├── setting_utils.h
    │   ├── setting_wifi_ability_slice.cpp  # WiFi 列表
    │   ├── setting_wifi_ability_slice.h
    │   ├── setting_wifi_input_password_ability_slice.cpp   # WiFi 密码输入
    │   ├── setting_wifi_input_password_ability_slice.h
    │   ├── ui_config.h
    │   ├── wpa_work.c              # WPA 客户端实现
    │   └── wpa_work.h
    ├── config.json
    └── resources/
```

**关键类**:
| 类名 | 文件 | 职责 |
|------|------|------|
| `SettingMainAbility` | setting_main_ability.cpp | Ability 入口 |
| `MainAbilitySlice` | main_ability_slice.cpp | 设置主页面 |
| `AppInfoAbilitySlice` | app_info_ability_slice.cpp | **权限管理** |
| `SettingWifiAbilitySlice` | setting_wifi_ability_slice.cpp | WiFi 设置 |

**安全关键函数** (`app_info_ability_slice.cpp:127`):
```cpp
int ret = QueryPermission(bundleName_, &permissions_, &permNum);
// GrantPermission(bundleName_, name_);
// RevokePermission(bundleName_, name_);
```

---

### 5. media - 媒体示例程序

**路径**: `media/`  
**类型**: 独立可执行文件（非 HAP）  
**核心功能**: 独立的相机、播放器、录音机示例

```
media/
├── BUILD.gn                # GN 构建配置
├── audio_capture_sample.cpp    # 音频录制示例
├── camera_sample.cpp       # 相机示例
└── player_sample.cpp       # 播放器示例
```

**构建目标** (`BUILD.gn`):
| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| camera_sample | executable | camera_sample | `$root_out_dir/dev_tools/` |
| player_sample | executable | player_sample | `$root_out_dir/dev_tools/` |
| audio_capture_sample | executable | audio_capture_sample | `$root_out_dir/dev_tools/` |

---

### 6. communication - WiFi 通信工具

**路径**: `communication/`  
**类型**: 子模块（独立仓库）  
**核心功能**: WiFi 相关工具（hostapd, wpa_supplicant, wpa_cli）

```
communication/
├── hostapd/            # AP 模式工具
│   ├── config/
│   └── src/
├── wpa_cli/            # WPA 客户端
│   └── src/
└── wpa_supplicant/     # WPA 协议实现
    ├── config/
    └── src/
```

**注意**: 此目录为 Git 子模块，代码不纳入本仓库版本管理。

---

### 7. screensaver - 屏保应用

**路径**: `screensaver/`  
**类型**: 子模块（独立仓库）+ HAP 应用  
**核心功能**: 屏幕保护程序

```
screensaver/
├── cert/               # 签名证书
├── screensaver/        # 应用代码
│   └── src/main/
│       ├── cpp/        # 源文件
│       ├── config.json
│       └── resources/  # 资源
└── wiki/               # 独立 Wiki（子模块自带）
```

**注意**: 此模块自带独立的 Wiki，位于 `screensaver/wiki/`。

---

## 代码统计（排除测试代码）

| 模块 | 源文件数 | 头文件数 | 主要语言 |
|------|----------|----------|----------|
| cameraApp | 3 | 5 | C++ |
| gallery | 4 | 6 | C++ |
| launcher | 8 | 10 | C++ |
| setting | 11 | 12 | C/C++ |
| media | 3 | 0 | C++ |

---

## 相关链接

- [架构说明](./Architecture.md) - 模块间关系与数据流
- [GN 构建系统](./Build_System.md) - 如何构建各模块
- [安全风险分析](./Security.md) - 各模块安全风险点
