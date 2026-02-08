# 目录结构

## 整体架构

```
kernel_linux_patches/
├── linux-4.19/              # Linux 4.19 内核补丁
│   ├── common_patch/        # 通用补丁
│   │   └── hdf.patch        # HDF 框架补丁
│   ├── hispark_taurus_patch/ # Hi3516D V300 专用补丁
│   │   ├── hispark_taurus.patch
│   │   └── hispark_taurus_small.patch
│   └── prebuilts/           # 预编译文件
│       └── usr/include/     # 头文件
├── linux-5.10/              # Linux 5.10 内核补丁
│   ├── common_patch/        # 通用补丁
│   │   └── hdf.patch
│   ├── hispark_taurus_patch/
│   ├── hispark_phoenix_patch/
│   ├── imx8mm_patch/
│   ├── khdvk_3566b_patch/
│   ├── ls3a5000_patch/
│   ├── qemu-arm-linux_patch/
│   ├── qemu-x86_64-linux_patch/
│   ├── rk3568_patch/
│   ├── unionpi_tiger_pacth/
│   ├── yangfan_patch/
│   ├── zhiyuan_patch/
│   └── prebuilts/
├── linux-6.6/               # Linux 6.6 内核补丁
│   ├── common_patch/
│   ├── rk3568_patch/
│   └── prebuilts/
├── wiki/                    # 文档目录
│   ├── _work/              # 工作目录
│   ├── README.md           # Wiki 说明
│   ├── SUMMARY.md          # 导航索引
│   ├── 01_Overview.md      # 项目概览
│   ├── 02_Directory_Structure.md  # 本文档
│   ├── 03_Supported_Boards.md    # 支持的板卡
│   ├── 04_Patches_Analysis.md    # 补丁分析
│   ├── 05_Build_System.md        # 构建系统
│   ├── 06_Security_Review.md     # 安全评审
│   └── appendix/           # 附录
├── LICENSE                 # GPL 2.0 许可证
├── README.md               # 项目说明
├── README_zh.md            # 中文说明
└── OAT.xml                # OpenHarmony 审计配置
```

## 目录职责说明

### 根目录文件

| 文件 | 职责 |
|------|------|
| `README.md` | 项目英文说明文档 |
| `README_zh.md` | 项目中文说明文档 |
| `LICENSE` | GPL 2.0 许可证文本 |
| `OAT.xml` | OpenHarmony 源码审计配置文件 |

### 内核版本目录

每个内核版本目录包含该版本的全部补丁：

```
linux-{version}/
├── common_patch/           # 通用补丁（所有板卡共享）
│   └── hdf.patch          # HDF 驱动框架补丁
├── {board}_patch/         # 板卡特定补丁
│   ├── kernel.patch       # 内核功能补丁
│   ├── hdf.patch          # HDF 适配补丁
│   └── device_tree.patch  # 设备树补丁（可选）
└── prebuilts/             # 预编译产物
    └── usr/include/       # 交叉编译头文件
```

### Wiki 目录结构

```
wiki/
├── _work/                  # 生成过程工作目录
├── 01_Overview.md         # 项目概览
├── 02_Directory_Structure.md  # 目录结构（本文档）
├── 03_Supported_Boards.md    # 支持的板卡
├── 04_Patches_Analysis.md    # 补丁分析
├── 05_Build_System.md        # 构建系统
├── 06_Security_Review.md     # 安全评审
└── appendix/
    ├── Config_Details.md     # 配置详情
    └── Patch_History.md      # 补丁历史
```

## 模块职责

### common_patch（通用补丁）

包含所有板卡共享的基础补丁：

- **hdf.patch**: OpenHarmony HDF 驱动框架核心补丁
- **security_patch**: 安全加固补丁
- **core_patch**: 内核核心功能补丁

### {board}_patch（板卡补丁）

包含特定板卡的适配补丁：

| 补丁类型 | 说明 |
|----------|------|
| kernel.patch | 芯片特定内核功能 |
| hdf.patch | 板卡 HDF 驱动适配 |
| defconfig | 内核配置文件 |

### prebuilts（预编译产物）

预编译的头文件和库：

- 交叉编译工具链头文件
- 架构特定的头文件（asm-*）
- 编译好的工具脚本

## 命名规范

### 补丁文件命名

- `hdf.patch`: HDF 框架补丁
- `kernel.patch`: 内核功能补丁
- `{board}.patch`: 板卡主补丁
- `{board}_small.patch`: 小型系统专用补丁

### 配置文件命名

- `{board}_small_defconfig`: 小型系统配置
- `{board}_standard_defconfig`: 标准系统配置
- `small_common_defconfig`: 通用小型系统配置
- `standard_common_defconfig`: 通用标准系统配置

## 相关文档

- [支持的板卡](./03_Supported_Boards.md)
- [补丁分析](./04_Patches_Analysis.md)
- [构建系统](./05_Build_System.md)
