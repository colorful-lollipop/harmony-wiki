# 项目概览

## 项目定位

`miscdevice` 是 OpenHarmony sensors 子系统的杂项设备服务模块，提供触觉反馈（振动器）和视觉反馈（LED 灯光）的统一管理能力。

### 核心能力

| 能力 | 说明 | 支持设备 |
|------|------|----------|
| 预设振动效果 | 播放预定义的触觉反馈模式 | 振动器 |
| 自定义振动 | 支持用户自定义振动参数 | 振动器 |
| 振动优先级 | 多应用振动场景的优先级管理 | 振动器 |
| LED 控制 | 控制设备 LED 灯光 | LED 指示灯 |
| 灯光动画 | 支持灯光动画效果 | LED 指示灯 |

### 系统能力

```json
{
  "SystemCapability.Sensors.MiscDevice": "标准设备完整支持",
  "SystemCapability.Sensors.MiscDevice.Lite": "轻量设备支持"
}
```

## 目录结构

```
/base/sensors/miscdevice
├── interfaces/              # 对外接口定义 (6 files)
│   ├── inner_api/          # 内部 C++ API
│   │   ├── vibrator/      # 振动器: vibrator_agent.h, vibrator_agent_type.h
│   │   └── light/         # 灯光: light_agent.h, light_agent_type.h
│   └── kits/c/             # 公开 C API
│       ├── vibrator.h     # C 接口头文件
│       └── vibrator_type.h # C 接口类型定义
├── frameworks/             # 框架层实现 (33 files)
│   ├── native/            # Native C++ 客户端
│   │   ├── vibrator/     # 振动器 native 客户端
│   │   └── light/        # 灯光 native 客户端
│   ├── js/napi/           # JavaScript N-API 绑定
│   ├── capi/              # C API 实现
│   ├── ets/taihe/         # ArkTS/Taihe 框架
│   └── cj/                # Cangjie FFI 支持
├── services/              # 服务层 (31 files)
│   └── miscdevice_service/# 核心 SA 服务
│       ├── src/          # 服务主逻辑
│       ├── hdi_connection/# HDI 连接层
│       └── haptic_matcher/# 触觉模式匹配
├── utils/                 # 工具库 (43 files)
│   ├── common/           # 通用工具
│   └── haptic_decoder/   # 触觉数据解码器
├── sa_profile/           # SA 配置
│   └── 3602.json         # MiscDevice SA 配置
└── miscdevice.gni        # 根构建配置
```

## 关键概念

### System Ability (SA)
MiscDevice 作为系统能力运行，SA ID 为 **3602**，运行在 `sensors` 进程。

### 权限模型
- `ohos.permission.VIBRATE` - 振动器权限 (system_grant)
- `ohos.permission.SYSTEM_LIGHT_CONTROL` - 灯光控制权限

### 错误码
| 错误码 | 含义 |
|--------|------|
| 201 | PERMISSION_DENIED - 权限拒绝 |
| ... | 其他错误码定义见 `utils/common/include/sensors_errors.h` |

## 运行环境

### 依赖组件
- `ability_base` - 基础能力框架
- `ability_runtime` - 运行时能力
- `access_token` - 访问令牌管理
- `hilog` - 日志系统
- `ipc` - IPC/Binder 通信
- `safwk` - System Ability Framework
- `samgr` - Service Manager

### 硬件依赖
- 振动器驱动 (`libvibrator_proxy_2.0.z.so`)
- LED 灯光驱动 (`liblight_proxy_1.0.z.so`)

## 相关资源

- **源码仓库**: `gitee.com/openharmony/sensors_miscdevice`
- **API 文档**: `@ohos.vibrator` (JS), `ohvibrator.h` (C)
- **系统能力**: `SystemCapability.Sensors.MiscDevice`
