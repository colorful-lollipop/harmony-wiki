# 目录结构

## 顶层目录概览

```
device_qemu/
├── README.md / README_zh.md          # 项目说明文档
├── LICENSE                           # Apache-2.0 许可证
├── OAT.xml                           # OpenHarmony 代码审计配置
├── .gitee/                          # Gitee 平台配置
├── .git/                            # Git 版本控制
├── drivers/                          # 【核心】设备驱动源码
├── hardware/                         # 硬件抽象层
├── arm_virt/                        # ARM 虚拟化平台
├── arm_mps2_an386/                  # Cortex-M4 模拟
├── arm_mps3_an547/                  # Cortex-M55 模拟
├── riscv32_virt/                    # RISC-V 32位虚拟平台
├── riscv64_virt/                    # RISC-V 64位虚拟平台
├── x86_64_virt/                     # x86_64 虚拟平台
├── esp32/                            # Xtensa ESP32 模拟
├── SmartL_E802/                      # C-SKY SmartL_E802 模拟
└── wiki/                             # 【本 Wiki】工程文档
```

### 目录职责分类

| 类别 | 目录 | 职责 |
|------|------|------|
| **配置** | `README*.md`, `LICENSE`, `OAT.xml` | 项目元信息 |
| **核心驱动** | `drivers/` | 设备驱动实现 (HDF/VirtIO) |
| **硬件抽象** | `hardware/` | 硬件抽象层代码 |
| **平台适配** | `arm_virt/`, `riscv*/`, `esp32/`, etc. | 各架构平台配置 |
| **文档** | `wiki/` | 工程 Wiki 文档 |

## 驱动目录结构 (drivers/)

### 目录树

```
drivers/
├── BUILD.gn                         # GN 构建入口 (drivers 组)
├── Kconfig                          # Linux 内核配置菜单
├── lite.mk                          # LiteOS Make 构建兼容
├── char/                            # 字符设备驱动
│   ├── BUILD.gn                     # char 模块构建配置
│   ├── Makefile                     # Make 构建配置
│   ├── mmz/                         # 内存管理区域
│   │   ├── BUILD.gn                 # mmz 构建配置
│   │   ├── mmz.c                   # MMZ 实现 (8308 行)
│   │   └── mmz.h                   # MMZ 头文件
│   └── Makefile
├── uart/                            # UART 串口驱动
│   ├── BUILD.gn                     # HDF UART 驱动构建
│   ├── Makefile
│   ├── uart.c                      # UART 核心实现 (14707 行)
│   ├── uart_pl011.c                # PL011 串口驱动 (14203 行)
│   └── uart_pl011.h                # PL011 头文件 (6467 行)
└── virtio/                          # VirtIO 虚拟化驱动
    ├── BUILD.gn                     # VirtIO HDF 驱动构建
    ├── Makefile
    ├── fakesdio.c                  # 伪 SDIO 设备
    ├── virtblock.c                 # 虚拟块设备 (18303 行)
    ├── virtgpu.c                  # 虚拟 GPU 设备 (22963 行)
    ├── virtinput.c                # 虚拟输入设备 (11137 行)
    ├── virtmmio.c                 # VirtIO MMIO 基础 (6455 行)
    ├── virtmmio.h                 # VirtIO MMIO 头文件
    ├── virtnet.c                  # 虚拟网络设备 (23284 行)
    └── virtrng.c                  # 虚拟 RNG (6837 行)
```

### 模块职责

| 模块 | 职责 | 关键文件 |
|------|------|----------|
| **char** | 平台字符设备驱动，支持设备节点创建 | `BUILD.gn`, `mmz.c` |
| **uart** | HDF PL011 串口驱动，提供 tty 设备 | `uart_pl011.c`, `uart.c` |
| **virtio** | VirtIO 虚拟设备驱动集合 | `virtblock.c`, `virtnet.c`, `virtgpu.c`, `virtinput.c`, `virtrng.c` |

### 模块依赖关系

```
drivers/BUILD.gn:32-38
┌─────────────────────────────────────┐
│            group("drivers")          │
│  deps: [ "char", "uart", "virtio" ] │
└─────────────────────────────────────┘
            │            │            │
            ▼            ▼            ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│     char     │ │     uart     │ │    virtio    │
│  (kernel_mod │ │  (hdf_driver │ │  (hdf_driver │
│   ule)       │ │   )          │ │   )          │
└──────────────┘ └──────────────┘ └──────────────┘
         │                               │
         ▼                               ▼
┌──────────────┐                ┌──────────────┐
│   mmz        │                │  HDF Core    │
│ (内存管理)    │                │  Framework   │
└──────────────┘                └──────────────┘
```

## 平台目录结构

### ARM 虚拟化平台 (arm_virt/)

```
arm_virt/
├── ohos.build                       # OpenHarmony 构建配置
├── liteos_a/                       # LiteOS_A 内核支持
│   ├── README.md / README_zh.md    # 平台说明
│   └── ...
└── linux/                          # Linux 内核支持
    ├── README.md / README_zh.md
    └── ...
```

### ARM Cortex-M 模拟 (arm_mps2_an386/, arm_mps3_an547/)

```
arm_mps2_an386/                     # Cortex-M4 模拟
├── ohos.build
├── README.md / README_zh.md        # MPS2-AN386 平台说明
└── liteos_m/                       # LiteOS_M 内核支持

arm_mps3_an547/                     # Cortex-M55 模拟
├── ohos.build
├── README.md / README_zh.md        # MPS3-AN547 平台说明
└── liteos_m/                       # LiteOS_M 内核支持
```

### RISC-V 虚拟平台 (riscv32_virt/, riscv64_virt/)

```
riscv32_virt/
├── ohos.build
├── README.md / README_zh.md        # RISC-V 32位说明
└── liteos_m/                       # LiteOS_M 内核支持

riscv64_virt/
├── ohos.build
├── README.md / README_zh.md        # RISC-V 64位说明
├── liteos_m/                       # LiteOS_M 内核支持
└── linux/                          # Linux 内核支持
```

### 其他架构

```
x86_64_virt/                        # x86_64 虚拟平台
├── ohos.build
├── README.md / README_zh.md
└── linux/                          # Linux 内核支持

esp32/                              # Xtensa ESP32 模拟
├── ohos.build
├── README.md / README_zh.md        # ESP32 平台说明
└── liteos_m/                       # LiteOS_M 内核支持

SmartL_E802/                        # C-SKY SmartL_E802 模拟
├── ohos.build
├── README.md / README_zh.md        # SmartL_E802 说明
└── liteos_m/                       # LiteOS_M 内核支持
```

### ohos.build 配置示例

**证据**: `arm_virt/ohos.build`
```json
{
  "parts": {
    "device_arm_virt": {
      "module_list": [
        "//device/qemu/arm_virt:arm_virt"
      ]
    }
  },
  "subsystem": "device_arm_virt"
}
```

## 硬件抽象目录 (hardware/)

```
hardware/
└── display/                        # 显示硬件抽象
    └── ...
```

## Wiki 目录结构

```
wiki/
├── README.md                       # Wiki 说明与更新指南
├── SUMMARY.md                       # 文档导航
├── index.md                         # 首页
├── 01_Project_Overview.md           # 项目概览
├── 02_Directory_Structure.md        # 本文档
├── 03_Architecture.md              # 架构设计
├── 04_GN_Build.md                  # GN 构建系统
├── 05_Build_Artifacts.md           # 编译产物
├── 06_Security_Review.md           # 安全评审
├── 07_Platforms.md                  # 支持的平台
├── 08_Troubleshooting.md           # 常见问题
├── _work/                          # 工作区
│   ├── NOTES.md                    # 事实记录
│   └── PLAN.md                     # 任务计划
└── appendix/                       # 附录
    ├── Callgraphs.md               # 调用链图
    └── Config_Flags.md             # 配置标志
```

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](01_Project_Overview.md) | 项目定位与核心能力 |
| [架构设计](03_Architecture.md) | 驱动架构与数据流 |
| [GN 构建](04_GN_Build.md) | 构建配置详解 |
| [支持的平台](07_Platforms.md) | 各平台详细配置 |
