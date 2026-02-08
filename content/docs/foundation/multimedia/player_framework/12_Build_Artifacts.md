# 编译产物说明

## 产物清单

### 系统服务产物

| 产物 | 路径 | 说明 |
|------|-----|------|
| libmedia_service.z.so | /system/lib64/media_service/ | 媒体服务主进程 |
| libplayer_service.z.so | /system/lib64/media_service/ | 播放服务 |
| librecorder_service.z.so | /system/lib64/media_service/ | 录制服务 |
| libscreen_capture_service.z.so | /system/lib64/media_service/ | 屏幕捕获服务 |
| libavmetadatahelper_service.z.so | /system/lib64/media_service/ | 元数据服务 |
| libmonitor_service.z.so | /system/lib64/media_service/ | 监控服务 |

### 客户端库产物

| 产物 | 路径 | 说明 |
|------|-----|------|
| libmedia_client.z.so | /system/lib64/ | 媒体客户端库 |
| libavplayer_napi.z.so | /system/lib64/ | AVPlayer N-API |
| libsoundpool_napi.z.so | /system/lib64/ | SoundPool N-API |
| libaudio_haptic_napi.z.so | /system/lib64/ | AudioHaptic N-API |

### 引擎产物

| 产物 | 路径 | 说明 |
|------|-----|------|
| libhistreamer.z.so | /system/lib64/media_service/ | HiStreamer 引擎 |
| liblpp_engine.z.so | /system/lib64/media_service/ | LPP 低功耗引擎 |

## 运行时加载关系

```
┌─────────────────────────────────────────────────────────────┐
│                     应用进程                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  libmedia_client.z.so                                      │
│      │                                                     │
│      ▼                                                     │
│  libavplayer_napi.z.so ──────────────┐                     │
│      │                               │                     │
│      ▼                               ▼                     │
│  Media Service SA ◄───────────────────┘                     │
│      │                                                     │
│      ▼                                                     │
│  libmedia_service.z.so                                     │
│      │                                                     │
│      ├──▶ libplayer_service.z.so                          │
│      ├──▶ librecorder_service.z.so                         │
│      ├──▶ libhistreamer.z.so                               │
│      └──▶ liblpp_engine.z.so                               │
└─────────────────────────────────────────────────────────────┘
```

## 安装路径

| 产物类型 | 系统路径 |
|---------|---------|
| 系统服务 | /system/lib64/media_service/ |
| N-API 库 | /system/lib64/ |
| 头文件 | /include/multimedia/ |
| 配置 | /etc/media/ |

## 产物版本

- **命名格式**: `{module}.{version}.so` 或 `{module}.z.so`
- **版本号**: 基于 OpenHarmony 版本
- **符号链接**: 指向最新版本

## 调试符号

- **位置**: /symbols/system/lib64/media_service/
- **用途**: Crash 分析、性能调试

## 相关文档

- [GN 构建系统](11_GN_Build.md)
