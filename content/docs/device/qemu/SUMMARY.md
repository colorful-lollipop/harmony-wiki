# 文档导航

本文档为 OpenHarmony device_qemu 仓库的工程 Wiki，提供新人快速理解项目所需的全套文档。

## 新人阅读路线

建议新人按以下顺序阅读：

```
1. README.md (本文档说明)
2. index.md (首页与快速入门)
3. 01_Project_Overview.md (项目概览)
4. 02_Directory_Structure.md (目录结构)
5. 03_Architecture.md (架构设计)
6. 04_GN_Build.md (构建系统)
7. 07_Platforms.md (支持的平台)
8. 06_Security_Review.md (安全评审)
```

## 完整文档列表

### 核心文档

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [README.md](README.md) | Wiki 说明与更新指南 | 必读 |
| [index.md](index.md) | 首页与快速入门 | 必读 |
| [01_Project_Overview.md](01_Project_Overview.md) | 项目定位、核心能力、运行环境 | 必读 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构与模块职责 | 必读 |
| [03_Architecture.md](03_Architecture.md) | 组件架构、数据流、驱动模型 | 必读 |
| [04_GN_Build.md](04_GN_Build.md) | GN 构建配置与 targets | 推荐 |
| [05_Build_Artifacts.md](05_Build_Artifacts.md) | 编译产物与安装路径 | 推荐 |
| [06_Security_Review.md](06_Security_Review.md) | 安全风险评审 | 推荐 |
| [07_Platforms.md](07_Platforms.md) | 支持的硬件平台 | 推荐 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 常见问题与定位 | 参考 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链图 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键配置标志 |

## 代码证据索引

本 Wiki 中的关键结论均基于以下代码证据：

### 构建配置文件

| 文件路径 | 用途 |
|----------|------|
| `drivers/BUILD.gn` | 驱动模块根构建配置 |
| `drivers/char/BUILD.gn` | 字符设备驱动构建 |
| `drivers/char/mmz/BUILD.gn` | 内存管理区域驱动构建 |
| `drivers/uart/BUILD.gn` | UART 串口驱动构建 |
| `drivers/virtio/BUILD.gn` | VirtIO 虚拟化驱动构建 |
| `drivers/Kconfig` | 内核配置菜单 |
| `drivers/lite.mk` | LiteOS Make 构建兼容 |
| `*/ohos.build` | 各平台 OpenHarmony 构建配置 |

### 驱动源码

| 文件路径 | 功能 |
|----------|------|
| `drivers/char/mmz/mmz.c` | 内存管理区域实现 |
| `drivers/uart/uart.c` | UART 核心驱动 |
| `drivers/uart/uart_pl011.c` | PL011 串口实现 |
| `drivers/virtio/virtblock.c` | 虚拟块设备 |
| `drivers/virtio/virtnet.c` | 虚拟网络设备 |
| `drivers/virtio/virtgpu.c` | 虚拟 GPU 设备 |
| `drivers/virtio/virtinput.c` | 虚拟输入设备 |
| `drivers/virtio/virtrng.c` | 虚拟随机数生成器 |

## 术语表

| 术语 | 说明 |
|------|------|
| **device_qemu** | OpenHarmony QEMU 设备模拟器仓库 |
| **QEMU** | Quick Emulator，快速仿真器 |
| **HDF** | Hardware Driver Foundation，硬件驱动框架 |
| **VirtIO** | 虚拟化 I/O 设备标准接口 |
| **MMZ** | Memory Management Zone，内存管理区域 |
| **LiteOS** | OpenHarmony 微内核/轻量级内核 |
| **GN** | Generate Ninja，构建系统生成工具 |
| **OHOS.build** | OpenHarmony 子系统构建配置 |
