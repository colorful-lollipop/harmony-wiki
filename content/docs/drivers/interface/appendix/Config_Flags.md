# 关键配置标志

## 概述

本文档描述 HDI 构建和运行时的关键配置标志，包括 GN 构建参数、HCS 配置选项和特性开关。

---

## 构建配置标志

### 系统类型标志

| 标志 | 含义 | 适用场景 |
|-----|------|---------|
| `is_standard_system` | 标准系统 | 手机、平板 |
| `is_small_system` | 小型系统 | 手表、电视 |
| `is_mini_system` | 迷你系统 | IoT 设备 |

**证据**: `interface.gni:15-33`

### 语言选择

| 标志 | 含义 | 说明 |
|-----|------|------|
| `language = "c"` | C 语言 | 轻量级、兼容性好 |
| `language = "cpp"` | C++ 语言 | 支持面向对象、复杂接口 |

### 生成模式

| 标志 | 含义 | 开销 | 适用场景 |
|-----|------|------|---------|
| `mode = "ipc"` | 进程间通信 | 较高 | 需要进程隔离 |
| `mode = "passthrough"` | 直通模式 | 极低 | 延迟敏感 |

**证据**: `input/v1_0/BUILD.gn:26`

---

## HDI 模板参数

### 必需参数

| 参数 | 类型 | 说明 |
|-----|------|------|
| `module_name` | string | 驱动模块名称 |
| `sources` | list | IDL 文件列表 |
| `language` | string | 生成语言 |

### 可选参数

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| `subsystem_name` | string | `"hdf"` | 子系统名称 |
| `part_name` | string | - | 部件名称 |
| `mode` | string | `"ipc"` | 生成模式 |
| `innerapi_tags` | list | `[]` | API 标签 |
| `install_images` | list | `["system"]` | 安装分区 |
| `branch_protector_ret` | string | - | 分支保护 |

---

## 特性开关 (Features)

### Audio 模块

```json
{
  "features": [
    "drivers_interface_audio_feature_alsa_lib",
    "drivers_interface_audio_community",
    "drivers_interface_audio_feature_offload"
  ]
}
```

| 特性 | 说明 |
|-----|------|
| `feature_alsa_lib` | ALSA 库支持 |
| `community` | 社区版本特性 |
| `feature_offload` | 音频卸载支持 |

### 其他模块特性

根据 `bundle.json` 分析，各模块特性开关不固定，需查看具体模块配置。

---

## HCS 配置标志

### Host 配置

```hcs
hostName = "audioHost";       // Host 名称
priority = 50;               // 调度优先级
hostUID = "1000";            // 用户 ID
hostGID = "1000";            // 组 ID
hostCaps = "...";            // 进程能力
```

### Device 配置

```hcs
policy = 2;                  // 驱动策略
priority = 100;              // 设备优先级
preload = 2;                 // 预加载策略
moduleName = "libxxx.so";    // 模块名称
serviceName = "xxx_service"; // 服务名称
```

| 策略值 | 含义 |
|--------|------|
| 0 | 不发布 |
| 1 | 仅内核态 |
| 2 | 发布到用户态 |
| 3 | 内核+用户态 |

---

## 安全相关标志

### 分支保护

```gni
branch_protector_ret = "pac_ret";  // ARM PAC 分支保护
```

### SELinux

```hcs
secon = "hdf_service:type:audio_service";  // 安全上下文
```

### 沙箱配置

```hcs
sandBox = 1;  // 启用沙箱
```

---

## API 标签

### innerapi_tags

| 标签 | 含义 |
|-----|------|
| `chipsetsdk` | 芯片 SDK 级别 |
| `platformsdk_indirect` | 间接平台 SDK |

**证据**: `sensor/v3_0/BUILD.gn:30-33`

```gni
innerapi_tags = [
  "chipsetsdk",
  "platformsdk_indirect",
]
```

---

## 安装目标

### install_images

| 目标 | 含义 |
|-----|------|
| `system` | 系统分区 |
| `updater` | 更新分区 |
| `vendor` | 厂商分区 |

**证据**: `input/v1_0/BUILD.gn:21-23`

```gni
install_images = [
  "system",
  "updater"
]
```
