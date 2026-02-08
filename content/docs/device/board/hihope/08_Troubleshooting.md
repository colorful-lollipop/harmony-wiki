# 常见问题与排查

## 构建问题

### Q1: 编译 RK3568 失败，提示 `is_support_graphic` 未定义

**错误信息**:
```
error: undefined variable is_support_graphic
```

**原因**: `device.gni` 中未正确声明 Feature Flags

**解决**:
```bash
# 检查 device.gni 是否包含 declare_args
cat /rk3568/device.gni | grep -A5 "declare_args"
```

**证据**: `/rk3568/device.gni:24-28`

```gn
declare_args() {
  is_support_boot_animation = true
  is_support_graphic = true
  is_support_codec = true
}
```

**临时解决**: 在命令行定义
```bash
hb set --enable-graphic
hb build -f
```

---

### Q2: Neptune100 编译提示 `LOSCFG_BOARD_NEPTUNE100` 未定义

**错误信息**:
```
error: variable LOSCFG_BOARD_NEPTUNE100 not defined
```

**原因**: 内核配置未启用对应 Board

**解决**: 确保在 `neptune100_defconfig` 中启用
```bash
# 检查配置
cat neptune100_defconfig | grep NEPTUNE100
```

**证据**: `/neptune100/neptune100_defconfig`

**解决**: 使用正确的产品配置
```bash
hb set -> winner_micro -> neptune100
hb build -f
```

---

### Q3: DAYU210 内核编译失败

**错误信息**:
```
build_kernel.sh: command not found
```

**原因**: 内核构建脚本路径错误

**证据**: `/dayu210/kernel/BUILD.gn`

**解决**: 确保在正确的目录执行
```bash
cd dayu210
hb build -f
```

---

## 运行问题

### Q4: 音频无声

**现象**: 播放音频无声音输出

**排查步骤**:

1. **检查音频驱动加载**
```bash
hdc shell
hilog | grep audio
```

2. **检查 ALSA 配置**
```bash
hdc shell
cat /vendor/etc/audio_config.json
```

3. **检查 Codec 状态**
```bash
hdc shell
cat /sys/class/audio/codec/0/status
```

**证据**: `/rk3568/audio_drivers/codec/rk809_codec/`

---

### Q5: 相机无法启动

**现象**: 打开相机应用黑屏或崩溃

**排查步骤**:

1. **检查 V4L2 设备**
```bash
hdc shell
ls /dev/video*
```

2. **检查 ISP 服务**
```bash
hdc shell
hilog | grep camera
```

3. **检查权限配置**
```bash
hdc shell
cat /etc/permissions/camera_permission.xml
```

**证据**: `/rk3568/camera/vdi_impl/v4l2/`

---

### Q6: WiFi 连接失败

**现象**: WiFi 无法扫描或连接

**排查步骤**:

1. **检查 WiFi 驱动**
```bash
hdc shell
hilog | grep bdh
cat /sys/class/net/wlan0/operstate
```

2. **检查 BCM 固件**
```bash
hdc shell
ls /vendor/firmware/bcm/
```

3. **检查网络配置**
```bash
hdc shell
ifconfig wlan0
```

**证据**: `/rk3568/wifi/bcmdhd_wifi6/hdfadapt/`

---

## 调试方法

### 日志查看

```bash
# 查看所有日志
hilog

# 过滤特定模块
hilog | grep -E "audio|camera|wifi"

# 实时日志
hilog -c &
```

### 设备节点

```bash
# 音频设备
ls /dev/audio*

# 相机设备
ls /dev/video*

# 网络设备
ls /sys/class/net/
```

### 驱动状态

```bash
# HDF 驱动列表
hdc shell
hdf devguest

# 特定驱动状态
cat /sys/bus/platform/devices/*/status
```

---

## 定位路径速查

| 问题 | 日志标签 | 配置文件 | 证据路径 |
|------|----------|----------|----------|
| 音频 | `audio` | init.rk3568.cfg | `/rk3568/audio_drivers/` |
| 相机 | `camera` | init.rk3568.cfg | `/rk3568/camera/` |
| WiFi | `bdh` / `wifi` | init.rk3568.cfg | `/rk3568/wifi/` |
| 启动 | `init` | init.*.cfg | `/rk3568/cfg/` |
| 分布式 | `distributed` | distributed_hardware_*.json | `/rk3568/distributedhardware/` |

---

## 工具使用

### hdc (HarmonyOS Device Connector)

```bash
# 连接设备
hdc conn 192.168.1.100

# 无线连接
hdc tconn 192.168.1.100

# 查看设备列表
hdc list targets

# 执行 shell 命令
hdc shell <command>

# 文件传输
hdc file send local_path remote_path
```

### hilog (日志工具)

```bash
# 查看日志
hilog

# 过滤错误
hilog | grep -i error

# 清除日志
hilog -c
```

---

## 相关文档

- [05_Board_Configurations](05_Board_Configurations.md) - 开发板配置
- [06_Hardware_Drivers](06_Hardware_Drivers.md) - 硬件驱动
- [07_Security_Review](07_Security_Review.md) - 安全评审
