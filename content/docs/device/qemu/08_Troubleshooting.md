# 常见问题

## 构建问题

### 问题 1: GN 编译失败

**错误信息**:
```
ERROR: Can't find source for: drivers/virtio/virtblock.c
```

**原因**: 源文件路径配置错误或文件不存在

**解决方法**:
1. 检查文件是否存在:
   ```bash
   ls -la drivers/virtio/virtblock.c
   ```

2. 检查 BUILD.gn 配置:
   ```gn
   # drivers/virtio/BUILD.gn
   sources = [
     "virtblock.c",  # 确保文件名正确
   ]
   ```

3. 检查相对路径:
   ```bash
   pwd  # 确认当前目录
   ```

### 问题 2: HDF 驱动模块未编译

**错误信息**:
```
module_switch is false: hdf_uart
```

**原因**: HDF 驱动编译开关未启用

**解决方法**:
1. 检查 Kconfig 配置:
   ```bash
   # 在内核配置中启用
   make menuconfig
   # 路径: Drivers → HDF → Platform → UART
   ```

2. 检查编译参数:
   ```bash
   # 确保启用了配置
   hb set -p arm_virt --product OHOS
   ```

3. 验证配置:
   ```bash
   # 检查生成的 .config
   grep DRIVERS_HDF_PLATFORM_UART out/arm_virt/.config
   ```

### 问题 3: 依赖库缺失

**错误信息**:
```
ninja: error: dependency '//drivers/hdf_core/adapter/khdf/liteos:hdf' not found
```

**原因**: HDF 框架未正确初始化

**解决方法**:
1. 初始化子模块:
   ```bash
   git submodule update --init --recursive
   ```

2. 检查依赖仓库:
   ```bash
   # 确保 drivers_hdf_core 仓库已克隆
   ls -la drivers/hdf_core/
   ```

3. 同步构建配置:
   ```bash
   hb clean
   hb set
   hb build -f
   ```

## 运行问题

### 问题 4: QEMU 未找到

**错误信息**:
```
qemu-system-aarch64: command not found
```

**原因**: QEMU 未安装或未添加到 PATH

**解决方法**:
1. 安装 QEMU:
   ```bash
   # Ubuntu/Debian
   sudo apt install qemu-system-arm qemu-system-riscv64

   # macOS
   brew install qemu
   ```

2. 验证安装:
   ```bash
   which qemu-system-aarch64
   qemu-system-aarch64 --version
   ```

3. 添加到 PATH:
   ```bash
   export PATH=$PATH:/usr/bin/qemu-system-aarch64
   ```

### 问题 5: 内核启动失败

**错误信息**:
```
Uncompress OK
Booting Kernel...
...
Kernel panic - not syncing: VFS: Unable to mount root fs
```

**原因**: 根文件系统未正确配置

**解决方法**:
1. 检查内核命令行:
   ```bash
   -append "root=devvdb rw console=ttyAMA0"
   ```

2. 确认镜像路径:
   ```bash
   ls -la out/arm_virt/
   ```

3. 检查设备树:
   ```bash
   -dtb out/arm_virt/OHOS.dtb
   ```

### 问题 6: VirtIO 设备未识别

**错误信息**:
```
virtio0: failed to recognize transport!
```

**原因**: VirtIO 驱动未加载或配置错误

**解决方法**:
1. 检查内核配置:
   ```bash
   # 确保 VirtIO 支持已启用
   grep VIRTIO out/arm_virt/.config
   ```

2. 验证 QEMU 命令:
   ```bash
   # 检查 VirtIO 设备是否添加
   -device virtio-blk-device,drive=hd0
   -device virtio-net-device,netdev=net0
   ```

3. 检查驱动日志:
   ```bash
   # 启用详细日志
   -serial stdio -d guest_errors
   ```

## 调试问题

### 问题 7: GDB 调试连接失败

**错误信息**:
```
Remote 'g' packet reply is too long
```

**原因**: GDB 与 QEMU 版本不兼容

**解决方法**:
1. 使用匹配版本的 GDB:
   ```bash
   # Ubuntu
   sudo apt install gdb-multiarch

   # macOS
   brew install armmbedtls
   ```

2. QEMU GDB 配置:
   ```bash
   # 启动 QEMU 并等待 GDB
   qemu-system-aarch64 -M virt -s -S
   ```

3. GDB 连接:
   ```bash
   gdb-multiarch out/arm_virt/OHOS.elf
   (gdb) target remote localhost:1234
   ```

### 问题 8: UART 输出乱码

**错误信息**:
```
[0;31m[0m[0;31m[0m[0;31m[0m[1;31m[1;31m
```

**原因**: 串口波特率不匹配

**解决方法**:
1. 检查 QEMU 串口配置:
   ```bash
   -serial stdio
   # 默认波特率: 115200
   ```

2. 检查内核配置:
   ```bash
   # 确保 CONFIG_SERIAL_8250=y
   ```

3. 使用正确终端:
   ```bash
   # minicom 或 screen
   minicom -D /dev/ttyUSB0 -b 115200
   ```

### 问题 9: 网络连接失败

**错误信息**:
```
net0: NIC not connected
```

**原因**: 虚拟网络未正确配置

**解决方法**:
1. 检查 TAP 设备:
   ```bash
   # 创建 TAP 设备 (需要 root)
   ip tuntap add dev tap0 mode tap
   ip link set tap0 up
   ```

2. QEMU 网络配置:
   ```bash
   -netdev tap,id=net0,ifname=tap0,script=no,downscript=no
   -device virtio-net-device,netdev=net0
   ```

3. 检查防火墙:
   ```bash
   # 允许桥接流量
   sudo iptables -A FORWARD -i tap0 -j ACCEPT
   ```

## 性能问题

### 问题 10: 运行缓慢

**症状**:
- 启动时间过长
- 交互响应延迟

**优化建议**:

1. 分配更多内存:
   ```bash
   -m 2G  # 增加内存
   ```

2. 使用 KVM 加速 (Linux):
   ```bash
   -accel kvm  # 硬件加速
   ```

3. 分配更多 CPU:
   ```bash
   -smp 4  # 多核加速
   ```

4. 禁用不必要设备:
   ```bash
   -device virtio-gpu-device,disable-modern=true
   ```

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](01_Project_Overview.md) | 环境准备与依赖 |
| [架构设计](03_Architecture.md) | 设备架构 |
| [支持的平台](07_Platforms.md) | 平台特定配置 |
| [GN 构建](04_GN_Build.md) | 构建问题排查 |
| [编译产物](05_Build_Artifacts.md) | 构建产物验证 |
