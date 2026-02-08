# 常见问题与调试

## 目的

本文档汇总 OpenHarmony Hisilicon 板卡仓库的常见构建、运行和调试问题，提供快速定位路径。

## 适用范围

本文档适用于：
- 开发者快速定位问题
- 系统工程师排查构建失败
- 运维人员解决运行时问题

---

## 构建相关问题

### Q1: U-Boot 编译失败 - 工具链路径错误

**症状**:
```
error: arm-linux-gcc: command not found
```

**原因**: `uboot/Makefile` 中 `OSDRV_CROSS` 工具链路径配置不正确

**定位路径**:
1. 检查 `hispark_aries/uboot/Makefile` 中的 `OSDRV_CROSS` 定义
2. 确认工具链版本是否更新

**解决方案**:
```makefile
# 修改 uboot/Makefile
OSDRV_CROSS = /path/to/your/toolchain/arm-linux-gnueabihf-
```

**证据**:
- `hispark_aries/README_zh.md:50` - "修改该目录下的Makefile中的OSDRV_CROSS所定义的工具链的路径"

---

### Q2: GN 构建报错 - 缺少 SoC 依赖

**症状**:
```
error: //device/board/hisilicon/hispark_taurus:... depends on //device/soc/hisilicon/..., which is not defined
```

**原因**: 未拉取 `device_soc_hisilicon` 仓库或路径错误

**定位路径**:
1. 检查 `hispark_taurus/BUILD.gn` 中的 `deps`
2. 确认 SoC 仓库是否存在

**解决方案**:
```bash
# 拉取完整源码
repo init -u https://gitee.com/openharmony/manifest.git -b master
repo sync -c
```

**证据**:
- `hispark_taurus/BUILD.gn:10` - `"//device/soc/hisilicon/hi3516dv300/sdk_linux:hispark_taurus_sdk"`

---

### Q3: HDF 配置编译失败 - HCS 文件未找到

**症状**:
```
error: $product_config_path/hdf_config/uhdf/camera/... not found
```

**原因**: Vendor 层 HDF 配置文件缺失

**定位路径**:
1. 检查 `hispark_taurus/camera/BUILD.gn` 中的 `hcs_file_prefix`
2. 确认 `product_config_path` 指向正确路径

**解决方案**:
```bash
# 检查 vendor 仓库
ls -la vendor/hisilicon/hispark_taurus_standard/hdf_config/uhdf/camera/
```

**证据**:
- `hispark_taurus/camera/BUILD.gn:12-16` - `hcs_file_prefix = "$product_config_path/hdf_config/uhdf/camera"`

---

## 运行时相关问题

### Q4: 设备启动失败 - Secure Boot 验证失败

**症状**:
```
Secure Boot: Signature verification failed!
System halted.
```

**原因**: 内核镜像签名不匹配或被篡改

**定位路径**:
1. 检查 U-Boot 日志（UART 输出）
2. 确认编译的内核镜像签名状态

**解决方案**:
```bash
# 重新编译并签名
hb build -f hispark_taurus --ccache
```

**证据**:
- `hispark_aries/uboot/secureboot_release/` - Secure Boot 实现

---

### Q5: 相机 HAL 加载失败

**症状**:
```
Failed to load camera HAL: /vendor/lib/libcamera_host.so
```

**原因**: MPP 库缺失或版本不匹配

**定位路径**:
1. 检查 `/vendor/lib/` 目录中的库文件
2. 查看 `dmesg` 或 `logcat` 日志

**解决方案**:
```bash
# 检查 MPP 库
ls -la /vendor/lib/libmpp.so
ls -la /vendor/lib/libmpi.so

# 重新编译
hb build -f hispark_taurus --ccache
```

**证据**:
- `hispark_taurus/device.gni:28` - `is_support_mpi = true`
- `hispark_taurus/camera/driver_adapter/include/mpi_adapter.h` - MPI 接口

---

### Q6: 音频无输出

**症状**: 音频播放或录制无声音

**原因**: Codec 驱动未加载或配置错误

**定位路径**:
1. 检查 `/sys/class/sound/` 设备节点
2. 查看 HAL 日志

**解决方案**:
```bash
# 检查音频设备
ls /sys/class/sound/
cat /proc/asound/cards

# 检查 HAL
ls -la /vendor/lib/libaudio_hal.so
```

**证据**:
- `hispark_taurus/audio_drivers/codec/hi3516/src/hi3516_codec_impl.c` - Codec 实现

---

## 调试技巧

### UART 串口调试

**连接方式**:
1. 硬件连接 UART RX/TX/GND
2. 使用串口工具（minicom, screen 等）

**参数**:
- 波特率: 115200
- 数据位: 8
- 停止位: 1
- 校验位: 无

**证据**:
- `hispark_taurus/liteos_a/board/include/hisoc/uart.h` - UART 接口定义

---

### HDF 配置调试

**查看运行时配置**:
```bash
# HDF 配置文件位置
cat /vendor/chipsets/.../hdfconfig/camera_host_config.hcb
```

**重新生成配置**:
```bash
# 删除编译输出
rm -rf out/hispark_taurus/

# 重新编译
hb build -f hispark_taurus --ccache
```

**证据**:
- `hispark_taurus/camera/BUILD.gn:40-44` - `.hcb` 文件生成

---

### GN 构建调试

**查看依赖图**:
```bash
# 生成依赖图
gn gen out/hispark_taurus --all
```

**查看目标列表**:
```bash
# 列出所有 targets
gn desc out/hispark_taurus //:*
```

---

## 日志查看

### LiteOS-A 日志

**位置**: UART 串口输出

**关键日志**:
```
[INFO] Board initialization complete
[INFO] HDF framework initialized
[INFO] Camera HAL loaded
```

---

### Linux 日志

**位置**: `dmesg` 或 `/var/log/messages`

**关键命令**:
```bash
# 查看内核日志
dmesg | grep camera

# 查看 OpenHarmony 日志
hilog | grep camera

# 实时跟踪
hilog -T | grep camera
```

---

## 常见错误码

### U-Boot 错误码

| 错误信息 | 原因 | 解决方案 |
|---------|------|---------|
| `Signature verification failed` | 签名不匹配 | 重新编译镜像 |
| `DDR init failed` | DDR 初始化失败 | 检查 DDR 配置 |
| `Flash read error` | Flash 读取失败 | 检查 Flash 连接 |

### HDF 错误码

| 错误信息 | 原因 | 解决方案 |
|---------|------|---------|
| `Failed to load HCS config` | 配置文件缺失 | 检查 `.hcb` 文件 |
| `Device not found` | 驱动未加载 | 检查驱动依赖 |
| `Failed to get service` | 服务未注册 | 检查 HDF 服务 |

---

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 详细的目录组织
- [GN Targets](04_GN_Targets.md) - 构建系统和目标依赖
- [编译产物](05_Build_Artifacts.md) - 产物和安装路径
- [安全评审](06_Security_Review.md) - Secure Boot 和安全边界
