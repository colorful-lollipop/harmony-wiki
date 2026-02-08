# 编译产物

## 目的

本文档介绍 media_lite 项目的编译产物，包括产物清单、安装路径和运行时加载关系。

## 适用范围

- 适用于构建工程师
- 适用于系统集成人员
- 适用于需要理解部署的开发者

---

## 产物清单

### 可执行文件

| 产物名 | 来源 Target | 大小估计 | 安装路径 | 说明 |
|--------|------------|----------|----------|------|
| `media_server` | `media_server` executable | ~500KB | `/usr/bin/media_server` | 媒体服务主进程 |
| `test_play_file_h265` | `test_play_file_h265` executable | ~200KB | `$root_out_dir/` | H265 解码测试程序 |

### 共享库（.so）

| 产物名 | 来源 Target | 大小估计 | 安装路径 | 加载方式 | 说明 |
|--------|------------|----------|----------|------|
| `libplayer_impl.so` | `player_impl` shared_library | ~2MB | `/usr/lib/libplayer_impl.so` | 被 player_server 动态加载 | 播放器核心实现 |
| `libplayer_server.so` | `player_server` shared_library | ~1MB | `/usr/lib/libplayer_server.so` | 被 media_server 动态加载 | 播放器服务 |
| `librecorder_impl.so` | `recorder_impl` shared_library | ~3MB | `/usr/lib/librecorder_impl.so` | 被 recorder_server 动态加载 | 录音器实现 |
| `libplayer_lite.so` | `player_lite` shared_library | ~100KB | `/usr/lib/libplayer_lite.so` | 被应用动态链接 | 播放器框架 |
| `librecorder_lite.so` | `recorder_lite` shared_library | ~100KB | `/usr/lib/librecorder_lite.so` | 被应用动态链接 | 录音器框架 |
| `libaudio_lite_api.so` | `audio_lite_api` lite_library | ~50KB | `/usr/lib/libaudio_lite_api.so` | 被 JS 引擎动态链接 | JSI 音频 API |
| `libmedia_ndk.so` | `media_ndk` ndk_lib | ~100KB | `/usr/lib/libmedia_ndk.so` | 被应用动态链接 | NDK 库 |

### 静态库（.a）

| 产物名 | 来源 Target | 大小估计 | 用途 | 说明 |
|--------|------------|----------|------|
| `librecorder_server.a` | `recorder_server` static_library | ~500KB | 静态链接到 media_server | 录音器服务静态库 |
| `libplayer_lite.a` | `player_lite` static_library | ~800KB | 静态链接（LiteOS-M） | LiteOS-M 播放器静态库 |
| `libaudio_lite_api.a` | `audio_lite_api` static_library | ~40KB | 静态链接（LiteOS-M） | LiteOS-M JSI 音频 API |

---

## 安装路径

### 标准路径（OpenHarmony Lite 系统）

| 产物类型 | 安装路径 | 说明 |
|---------|----------|------|
| 可执行文件 | `/usr/bin/` 或 `/system/bin/` | 系统可执行文件目录 |
| 共享库 | `/usr/lib/` 或 `/system/lib/` | 系统库目录 |
| NDK 库 | `/usr/lib/` 或 `/system/lib/` | 开发库目录 |
| 测试程序 | `$root_out_dir/` | 构建输出目录 |

### 动态库搜索路径

应用启动时，系统会按以下顺序搜索共享库：
1. `/usr/lib/`
2. `/system/lib/`
3. `/vendor/lib/`（如果存在）

---

## 运行时加载关系

### 启动流程

```
systemd/init
    ↓
media_server (进程启动)
    ↓
SAMGR_Bootstrap()
    ↓
注册 RecorderServer 服务
    ↓
注册 PlayerServer 服务
    ↓
[如果非 Passthrough 模式]
    ↓
初始化 AudioCapturerServer 服务
    ↓
[如果支持相机]
    ↓
初始化 CameraServer 服务
    ↓
进入信号等待循环
```

### 进程间通信

**Binder 模式**：
```
应用进程                media_server 进程
    ↓
Player/Recorder 框架   IPC 层 (IpcIo)
    ↓
PlayerServer/RecorderServer 服务
```

**Passthrough 模式**：
```
应用进程
    ↓
Player/Recorder 框架  直接调用（dlopen）
    ↓
player_impl/recorder_impl
```

### 库依赖

| 库 | 被加载者 | 依赖的库 |
|------|----------|----------|
| `libplayer_impl.so` | 被 `libplayer_server.so` 加载 | `libmedia_common.so`, `libsurface_lite.so` |
| `librecorder_impl.so` | 被 `librecorder_server.a` 链接 | `libmedia_common.so`, `libsurface_lite.so`, `libpms_client.so`, `libipc_single.so` |
| `libplayer_lite.so` | 被应用动态链接 | `libaudio_lite_api.so`（JSI） |
| `librecorder_lite.so` | 被应用动态链接 | 无 |

---

## 目标设备内存占用

### media_server 进程

| 组件 | 估计内存占用 | 说明 |
|------|----------|------|
| media_server 主程序 | ~1MB | 服务框架和主循环 |
| player_server 服务 | ~2MB | 播放器服务端 + player_impl |
| recorder_server 服务 | ~3MB | 录音器服务端 + recorder_impl |
| samgr 管理层 | ~500KB | 服务管理和发现 |
| 总计 | **~6.5MB** | 不包括动态库 |

### 动态库

| 库 | 估计内存占用 | 说明 |
|------|----------|------|
| libplayer_impl.so | ~2MB | 播放器实现 |
| librecorder_impl.so | ~3MB | 录音器实现 |
| libplayer_lite.so | ~500KB | 播放器框架 |
| librecorder_lite.so | ~500KB | 录音器框架 |
| libaudio_lite_api.so | ~200KB | JSI 音频 API |
| libmedia_ndk.so | ~300KB | NDK 接口 |
| 总计 | **~6.5MB** | 动态库（所有加载时） |

---

## Map 文件

| 文件 | 说明 |
|------|------|
| `media_server.map` | media_server 进程的内存布局和符号表（用于调试） |

---

## 构建输出

### 输出目录结构

```
$out_dir/
├── bin/
│   └── media_server
├── lib/
│   ├── libplayer_impl.so
│   ├── libplayer_server.so
│   ├── librecorder_impl.so
│   ├── libplayer_lite.so
│   ├── librecorder_lite.so
│   ├── libaudio_lite_api.so
│   └── libmedia_ndk.so
├── test/
│   ├── unittest/
│   │   ├── playerlite/
│   │   │   └── lite_player_unittest.bin
│   │   └── recorder/
│   │       └── lite_recorder_unittest.bin
│   └── test_play_file_h265
└── media_server.map
```

---

## 配置文件

### 运行时配置

media_server 在启动时可能读取以下配置：
- 相机配置：`/data/cameradev*.ini`（已注释，当前未使用）
- 权限配置：由 permission_lite 服务管理
- 日志配置：由 hilog_lite 服务管理

### 编译时配置

- `config.gni`：全局编译选项
- `enable_media_passthrough_mode`：运行模式选择
- `enable_multimedia_camera_lite`：相机支持开关

---

## 调试与测试产物

### 单元测试

| 测试程序 | 覆盖范围 | 运行方式 |
|----------|----------|------|
| `lite_player_unittest.bin` | Player 框架单元测试 | 直接运行或通过测试框架 |
| `lite_recorder_unittest.bin` | Recorder 框架单元测试 | 直接运行或通过测试框架 |

### 功能测试

| 测试程序 | 测试内容 | 运行方式 |
|----------|----------|------|
| `test_play_file_h265` | H265 视频解码 | 直接运行，验证播放器解码能力 |

---

## 部署建议

### 镜像集成

将以下产物集成到系统镜像：
1. 复制共享库到 `/usr/lib/`
2. 复制 media_server 到 `/usr/bin/`
3. 复制 NDK 库到 `/usr/lib/`
4. 配置 systemd 启动服务（如果需要）

### 应用开发

应用开发者需要：
1. 链接 `libplayer_lite.so` 使用 Player 接口
2. 链接 `librecorder_lite.so` 使用 Recorder 接口
3. 使用 JS 引擎加载 `libaudio_lite_api.so` 调用音频播放功能

---

## 相关跳转

- [项目概览](00_Overview.md)
- [目录结构与模块职责](01_Directory_Structure.md)
- [GN Targets](05_GN_Targets.md)
- [对外 JSI 接口](03_JSI_Interfaces.md)
- [内部 API](04_Inner_API.md)
