# 关键宏与 Feature Flags

## GNI 参数 (distributedcamera.gni)

### 全局开关

| 宏 | 文件 | 默认值 | 用途 |
|----|------|--------|------|
| `distributed_camera_common` | distributedcamera.gni:45 | `true` | 控制公共代码路径（标准 vs 通用） |
| `device_security_level_camera` | distributedcamera.gni:46 | `true` | 启用设备安全级别检查 |
| `distributed_camera_filter_front` | distributedcamera.gni:47 | `false` | 启用前置摄像头过滤 |
| `distributed_camera_wakeup_enabled` | distributedcamera.gni:48 | `false` | 启用唤醒功能 |
| `distributed_camera_open_stabile` | distributedcamera.gni:49 | `false` | 启用相机打开稳定性优化 |

### OS 账户支持

```gn
if (!defined(global_parts_info) ||
    defined(global_parts_info.account_os_account)) {
    os_account_camera = true
} else {
    os_account_camera = false
}
```

---

## 条件编译宏

### 调试相关

| 宏 | 触发条件 | 用途 | 风险 |
|----|----------|------|------|
| `DUMP_DCAMERA_FILE` | `build_variant == "root"` | 调试文件转储 | 仅调试版本 |
| `DCAMERA_YUV` | demo 版本 | YUV 格式支持 | 仅 demo |

### 安全相关

| 宏 | 触发条件 | 用途 |
|----|----------|------|
| `SECURITY_LEVEL_CHECK_ENABLE` | `!distributed_camera_common` | 启用安全级别检查 |
| `DEVICE_SECURITY_LEVEL_ENABLE` | `device_security_level_camera` | 设备安全级别使能 |

### 功能相关

| 宏 | 触发条件 | 用途 |
|----|----------|------|
| `DCAMERA_SUPPORT_FFMPEG` | `distributed_camera_common` | FFmpeg 编解码支持 |
| `DCAMERA_MMAP_RESERVE` | `!distributed_camera_common` | MMAP 内存保留 |
| `DCAMERA_WAKEUP` | `distributed_camera_wakeup_enabled` | 唤醒功能 |
| `DCAMERA_OPEN_STABILE` | `distributed_camera_open_stabile` | 相机打开稳定性 |
| `DCAMERA_FRONT` | `distributed_camera_filter_front` | 前置摄像头过滤 |
| `OS_ACCOUNT_ENABLE` | `os_account_camera` | OS 账户功能 |

### 账户相关

```gn
# BUILD.gn 中的条件编译
if (os_account_camera) {
    defines += [ "OS_ACCOUNT_ENABLE" ]
    external_deps += [
        "os_account:libaccountkits",
        "os_account:os_account_innerkits",
    ]
}
```

---

## 日志相关

### 日志宏定义

| 宏 | 定义位置 | 用途 |
|----|----------|------|
| `DH_LOG_TAG` | 各 BUILD.gn | 日志标签 (如 `dcamerasourcesvr`) |
| `LOG_DOMAIN` | 各 BUILD.gn | 日志域 (`0xD004150`) |
| `HI_LOG_ENABLE` | 各 BUILD.gn | 启用 HiLog |

### 日志级别

| 级别 | 宏 | 说明 |
|------|-----|------|
| DEBUG | `DHLOGD` | 调试日志 |
| INFO | `DHLOGI` | 信息日志 |
| WARN | `DHLOGW` | 警告日志 |
| ERROR | `DHLOGE` | 错误日志 |
| FATAL | `DHLOGF` | 致命错误 |

**定义位置**: `common/include/utils/dh_log.h`

---

## 错误码基址

| 常量 | 值 | 说明 |
|------|-----|------|
| `ERR_DH_CAMERA_BASE` | `0x05C20000` | 错误码基址 |

---

## 常量定义

### 长度限制

| 常量 | 值 | 说明 | 文件 |
|------|-----|------|------|
| `DID_MAX_SIZE` | 256 | 设备 ID 最大长度 | distributed_camera_constants.h |
| `PARAM_MAX_SIZE` | 4096 | 参数最大长度 | distributed_camera_constants.h |
| `DCAMERA_MAX_RECV_DATA_LEN` | - | 接收数据最大长度 | dcamera_softbus_session.cpp |

### 缓冲区限制

| 常量 | 值 | 说明 |
|------|-----|------|
| `MAX_YUV420_BUFFER_SIZE` | - | YUV420 缓冲区最大大小 |
| `MAX_IMG_SIZE` | 46340 | 图像尺寸最大值 |

---

## 安全编译选项

### sanitizer 配置

```gn
sanitize = {
  cfi = true                    # 控制流完整性
  cfi_cross_dso = true         # 跨 DSO CFI
  boundary_sanitize = true      # 边界检查
  integer_overflow = true       # 整数溢出检查
  ubsan = true                  # 未定义行为检查
}
stack_protector_ret = true       # 栈保护
branch_protector_ret = "pac_ret" # ARM PAC 返回地址保护
```

### 链接器选项

```gn
ldflags = [
  "-fpie",               # 位置无关可执行文件
  "-Wl,-z,relro",       # 只读重定位
  "-Wl,-z,now",         # 立即绑定
]
```

---

## Feature Flag 使用场景

### 标准版 (distributed_camera_common = true)

```gn
# 使用 FFmpeg 编解码
DCAMERA_SUPPORT_FFMPEG = true

# 不使用 MMAP 保留
DCAMERA_MMAP_RESERVE = false
```

### 通用版 (distributed_camera_common = false)

```gn
# 不使用 FFmpeg
DCAMERA_SUPPORT_FFMPEG = false

# 使用 MMAP 保留
DCAMERA_MMAP_RESERVE = true

# 启用安全级别检查
SECURITY_LEVEL_CHECK_ENABLE = true
```

---

## 配置示例

### 最小权限配置

```gn
distributed_camera_common = true
device_security_level_camera = false
distributed_camera_filter_front = false
distributed_camera_wakeup_enabled = false
distributed_camera_open_stabile = false
```

### 安全增强配置

```gn
distributed_camera_common = false
device_security_level_camera = true
distributed_camera_filter_front = true
distributed_camera_wakeup_enabled = true
distributed_camera_open_stabile = true
```
