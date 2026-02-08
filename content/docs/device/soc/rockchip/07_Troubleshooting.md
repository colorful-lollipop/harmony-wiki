# 常见问题与故障排除

## 文档信息

- **目的**: 汇总 Rockchip OpenHarmony 仓库的常见构建、运行、调试问题及解决方法
- **适用范围**: 所有芯片平台
- **关键结论**: 问题主要集中在构建配置、驱动加载和 HDI 接口调用

## 构建问题

### 1. GN 构建失败

**问题现象**:
```
ERROR at //device/soc/rockchip/rk3568/hardware/display/BUILD.gn:xx:xx
Unable to find target: libdrm
```

**原因分析**:
- 缺少依赖库
- 路径配置错误

**解决方法**:
```bash
# 1. 确认 libdrm 已同步
cd /path/to/openharmony/third_party/libdrm
git status

# 2. 检查 gn 配置
gn args out/ --list | grep libdrm

# 3. 重新生成构建配置
gn gen out/ --args="target_cpu=\"arm64\""
```

**代码证据**: `rk3568/hardware/display/BUILD.gn:88`
```gn
deps = [
  "${root_path}/third_party/libdrm:libdrm",
]
```

### 2. 编译器版本不匹配

**问题现象**:
```
error: 'xxx' was not declared in this scope
```

**原因分析**:
- GCC/Clang 版本过低
- C++ 标准不匹配

**解决方法**:
```bash
# 检查编译器版本
gcc --version
clang --version

# 设置正确的编译器
export CC=/path/to/clang
export CXX=/path/to/clang++
```

### 3. 头文件找不到

**问题现象**:
```
fatal error: 'xxx.h' file not found
```

**原因分析**:
- include_dirs 配置不完整
- 依赖模块未编译

**解决方法**:
```bash
# 1. 检查头文件路径
find /path/to/openharmony -name "xxx.h" 2>/dev/null

# 2. 更新 BUILD.gn 添加 include_dirs
include_dirs = [
  "${root_path}/path/to/header",
]
```

## 运行问题

### 1. VDI 库加载失败

**问题现象**:
```
E/HDF: Failed to load libdisplay_composer_vdi_impl.so
```

**原因分析**:
- 库文件未安装到正确位置
- 依赖库缺失

**解决方法**:
```bash
# 1. 检查库文件是否存在
ls -la /vendor/lib64/libdisplay_composer_vdi_impl.so

# 2. 检查依赖库
ldd /vendor/lib64/libdisplay_composer_vdi_impl.so

# 3. 检查日志
dmesg | grep -i display
logcat | grep -i display
```

**代码证据**: `rk3568/hardware/display/BUILD.gn:155-156`
```gn
install_enable = true
install_images = [ chipset_base_dir ]
```

### 2. DRM 设备打开失败

**问题现象**:
```
E/Display: Failed to open DRM device: Permission denied
```

**原因分析**:
- 权限不足
- 设备节点不存在

**解决方法**:
```bash
# 1. 检查设备节点
ls -la /dev/dri/card0

# 2. 检查权限
getfacl /dev/dri/card0

# 3. 修改权限（临时）
chmod 666 /dev/dri/card0

# 4. 永久解决：修改 ueventd.rc
# /dev/dri/card* 0666 root root
```

**代码证据**: `common/hardware/display/src/display_device/drm_device.cpp`

### 3. MPP 初始化失败

**问题现象**:
```
E/MPP: mpp_init failed, ret = -9
```

**原因分析**:
- 内核驱动未加载
- 编解码格式不支持

**解决方法**:
```bash
# 1. 检查内核模块
lsmod | grep mpp

# 2. 加载驱动
insmod /vendor/lib/modules/mpp_service.ko

# 3. 检查支持的格式
cat /sys/class/mpp_service/mpp_service/support_formats

# 4. 查看详细日志
echo 1 > /sys/class/mpp_service/mpp_service/debug
```

**代码证据**: `common/hardware/mpp/include/mpp_err.h`
```c
#define MPP_ERR_OPEN_CODEC  -4  /* 打开编解码器失败 */
```

### 4. HDMI 无输出

**问题现象**:
屏幕黑屏，无 HDMI 信号

**原因分析**:
- HDMI 未连接
- 显示模式不匹配
- 驱动问题

**解决方法**:
```bash
# 1. 检查 HDMI 连接状态
cat /sys/class/drm/card0-HDMI-A-1/status

# 2. 查看显示模式
cat /sys/class/drm/card0-HDMI-A-1/modes

# 3. 设置显示模式
echo "1920x1080" > /sys/class/drm/card0-HDMI-A-1/mode

# 4. 查看 DRM 状态
drm_info
```

## 调试方法

### 1. HDF 日志调试 (RK2206)

**启用日志**:
```c
// 在驱动代码中添加
#include "hilog/log.h"

#define LOG_TAG "GPIO_DRIVER"
#define LOGD(...) HILOG_DEBUG(LOG_CORE, __VA_ARGS__)
#define LOGE(...) HILOG_ERROR(LOG_CORE, __VA_ARGS__)

// 使用
LOGD("GPIO init, pin = %d", pin);
```

**查看日志**:
```bash
# 通过串口查看
hilog

# 过滤特定标签
hilog | grep GPIO_DRIVER
```

**代码证据**: `rk2206/hdf_driver/gpio/gpio_driver.c`

### 2. Display 调试

**启用调试日志**:
```bash
# 设置日志级别
setprop persist.sys.hilog.debug 1

# 查看显示日志
logcat -s DisplayComposerVdiImpl:*
logcat -s HdiSession:*
```

**DRM 调试**:
```bash
# 查看 DRM 信息
drm_info

# 查看连接器状态
cat /sys/kernel/debug/dri/0/state

# 查看帧缓冲
cat /sys/kernel/debug/dri/0/framebuffer
```

**代码证据**: `common/hardware/display/src/display_device/hdi_session.cpp`

### 3. MPP 调试

**启用 MPP 日志**:
```bash
# 设置日志级别
export mpp_debug=1

# 查看 MPP 日志
logcat -s mpp:*
```

**MPP 性能分析**:
```bash
# 查看 MPP 统计信息
cat /sys/class/mpp_service/mpp_service/stats

# 查看解码器状态
cat /sys/class/mpp_service/mpp_service/decoder/state
```

**代码证据**: `common/hardware/mpp/mpp/hdi_mpp/hdi_mpp_mpi.cpp`

### 4. 内核调试

**内核日志**:
```bash
# 查看内核日志
dmesg

# 实时查看
adb shell dmesg -w

# 清除日志
dmesg -c
```

**调试接口**:
```bash
# 查看设备树
cat /proc/device-tree/compatible

# 查看内存映射
cat /proc/iomem

# 查看中断
cat /proc/interrupts
```

## 性能优化

### 1. 显示性能

**问题**: 帧率低、卡顿

**优化方法**:
```bash
# 1. 检查 VSync 状态
cat /sys/class/drm/card0/vblank_event

# 2. 启用硬件合成
setprop debug.hwui.disable_hwcomposer 0

# 3. 调整缓冲区数量
setprop ro.hardware.gralloc.min_buffers 3
```

### 2. 编解码性能

**问题**: 解码卡顿、延迟高

**优化方法**:
```c
// 设置低延迟模式
MppParam param;
int low_delay = 1;
param = &low_delay;
mpi->control(ctx, MPP_DEC_SET_PARSER_FAST_MODE, param);

// 设置输出格式
MppFrameFormat fmt = MPP_FMT_YUV420SP;
param = &fmt;
mpi->control(ctx, MPP_DEC_SET_OUTPUT_FORMAT, param);
```

**代码证据**: `common/hardware/mpp/include/rk_mpi_cmd.h`

## 常见问题速查表

| 问题 | 检查点 | 命令 |
|------|--------|------|
| 构建失败 | 依赖库 | `gn args out/ --list` |
| 库加载失败 | 库路径 | `ldd /vendor/lib64/xxx.so` |
| DRM 失败 | 设备节点 | `ls -la /dev/dri/` |
| MPP 失败 | 内核模块 | `lsmod \| grep mpp` |
| 无显示 | HDMI 状态 | `cat /sys/class/drm/card0-HDMI-A-1/status` |
| 性能问题 | CPU 占用 | `top` |
| 内存问题 | 内存使用 | `free -h` |

## 调试工具

### 1. 日志工具

| 工具 | 用途 | 示例 |
|------|------|------|
| hilog | HDF 日志 | `hilog \| grep GPIO` |
| logcat | 系统日志 | `logcat -s Display:*` |
| dmesg | 内核日志 | `dmesg \| grep -i error` |

### 2. 分析工具

| 工具 | 用途 | 示例 |
|------|------|------|
| strace | 系统调用跟踪 | `strace -p $(pidof display_service)` |
| ltrace | 库调用跟踪 | `ltrace -p $(pidof display_service)` |
| perf | 性能分析 | `perf top` |

### 3. 硬件工具

| 工具 | 用途 | 示例 |
|------|------|------|
| drm_info | DRM 信息 | `drm_info` |
| modetest | DRM 测试 | `modetest -M rockchip` |
| v4l2-ctl | 视频测试 | `v4l2-ctl -d /dev/video0 --all` |

## 相关链接

- [项目概览](00_Overview.md) - 项目定位
- [架构说明](01_Architecture.md) - 系统架构
- [HDI/VDI 接口](03_HDI_Interfaces.md) - 接口文档
- [GN 构建系统](04_GN_Build.md) - 构建配置
