# 项目概览

> Wallpaper Mgr 子系统定位与核心能力

## 项目定位

### 基本信息
| 属性 | 值 |
|------|-----|
| **子系统名称** | theme |
| **组件名称** | wallpaper_mgr |
| **版本** | 3.1 |
| **License** | Apache License 2.0 |
| **代码路径** | /base/theme/wallpaper_mgr |

### 系统能力
```
SystemCapability.MiscServices.Wallpaper
```

### ROM/RAM 占用
| 指标 | 大小 |
|------|------|
| ROM | 930 KB |
| RAM | 2895 KB |

## 核心能力

### 1. 壁纸生命周期管理
- **设置壁纸**: 支持 JPEG/PNG 图片、MP4 视频
- **获取壁纸**: 获取壁纸 PixelMap、颜色值、文件描述符
- **重置壁纸**: 恢复默认壁纸
- **切换壁纸**: 支持系统壁纸与锁屏壁纸独立管理

### 2. 多态壁纸支持
| 类型 | 说明 | 支持状态 |
|------|------|----------|
| 静态图片 | JPEG/PNG 格式 | ✅ 支持 |
| 动态视频 | MP4 格式 | ✅ 支持 |
| 动画壁纸 | Live Wallpaper | ✅ 支持 |
| 折叠屏多态 | 不同折叠状态显示不同壁纸 | ✅ 支持 |

### 3. 折叠屏适配
- 支持 NORMAL（折叠态）
- 支持 UNFOLD_1（一次展开态）
- 支持 UNFOLD_2（二次展开态）
- 支持 PORT（竖屏）/ LAND（横屏）切换

### 4. 颜色主题提取
- 自动提取壁纸主色调
- 支持颜色变更事件订阅
- 提供 `on('colorChange')` 回调机制

## 运行环境

### 系统依赖
```
graphic_2d, samgr, common_event_service, hiview, ipc, hitrace, hisysevent
ability_runtime, safwk, access_token, napi, ability_base, hilog, c_utils
bundle_framework, os_account, window_manager, image_framework, player_framework
eventhandler, runtime_core, init, memmgr, config_policy, cJSON, selinux_adapter
```

### 目标设备
- OpenHarmony 标准系统设备
- 支持折叠屏设备（可选功能）

## 关键概念

### WallpaperType
| 常量 | 值 | 说明 |
|------|-----|------|
| `WALLPAPER_SYSTEM` | 0 | 系统壁纸（主屏幕） |
| `WALLPAPER_LOCKSCREEN` | 1 | 锁屏壁纸 |

### WallpaperResourceType
| 常量 | 值 | 说明 |
|------|-----|------|
| `DEFAULT` | 0 | 默认壁纸 |
| `PICTURE` | 1 | 图片壁纸 |
| `VIDEO` | 2 | 视频壁纸 |
| `PACKAGE` | 3 | 壁纸包 |

### 错误码定义
详见 [06_Security.md](06_Security.md) 错误码章节

## 相关文档

- N-API 接口: [03_API.md](03_API.md)
- 架构设计: [04_Architecture.md](04_Architecture.md)
- 安全评估: [06_Security.md](06_Security.md)
