# 补丁历史

本文档记录 `kernel_linux_patches` 仓库的重要变更历史，包括新增功能和重大修复。

## 版本历史

### v1.0.0（初始版本）

**发布日期**: 2024-01-01

**包含内容**:
- Linux 4.19.y 内核补丁
- Linux 5.10.y 内核补丁
- Hi3516D V300 板卡支持
- HDF 驱动框架补丁

**主要特性**:
- 初始 HDF 框架支持
- 基础 SOC 驱动适配
- 小型系统和标准系统配置

### v1.1.0

**发布日期**: 2024-03-01

**新增内容**:
- Rockchip RK3568 板卡支持
- NXP i.MX8M Mini 板卡支持
- QEMU 模拟器支持

**主要变更**:
```
+ linux-5.10/rk3568_patch/     # 新增 RK3568 支持
+ linux-5.10/imx8mm_patch/     # 新增 i.MX8M Mini 支持
+ linux-5.10/qemu-arm-linux_patch/      # 新增 QEMU ARM
+ linux-5.10/qemu-x86_64-linux_patch/   # 新增 QEMU x86_64
```

### v1.2.0

**发布日期**: 2024-06-01

**新增内容**:
- Loongson 3A5000 板卡支持
- 开天 3566B 开发套件支持
- UnionPi Tiger 支持

**主要变更**:
```
+ linux-5.10/ls3a5000_patch/   # 新增龙芯支持
+ linux-5.10/khdvk_3566b_patch/ # 新增开天支持
+ linux-5.10/unionpi_tiger_pacth/ # 新增 UnionPi 支持
```

### v1.3.0

**发布日期**: 2024-09-01

**新增内容**:
- 扬帆开发板支持
- 致远开发板支持
- HiHope Phoenix 板卡支持

**主要变更**:
```
+ linux-5.10/yangfan_patch/    # 新增扬帆支持
+ linux-5.10/zhiyuan_patch/    # 新增致远支持
+ linux-5.10/hispark_phoenix_patch/ # 新增 Phoenix 支持
```

### v1.4.0

**发布日期**: 2024-12-01

**新增内容**:
- Linux 6.6.y 预览支持
- 增强安全配置
- HDF 框架更新

**主要变更**:
```
+ linux-6.6/                   # 新增 Linux 6.6 支持
+ linux-6.6/common_patch/hdf.patch
+ linux-6.6/rk3568_patch/
```

## 补丁分类记录

### HDF 框架补丁

| 版本 | 补丁文件 | 主要变更 |
|------|----------|----------|
| v1.0.0 | hdf.patch | 初始 HDF 支持 |
| v1.1.0 | hdf.patch | 新增平台适配 |
| v1.4.0 | hdf.patch | Linux 6.6 适配 |

### Hi3516D V300 补丁

| 版本 | 补丁文件 | 主要变更 |
|------|----------|----------|
| v1.0.0 | hispark_taurus.patch | 初始支持 |
| v1.0.0 | hispark_taurus_small.patch | 小型系统支持 |

### RK3568 补丁

| 版本 | 补丁文件 | 主要变更 |
|------|----------|----------|
| v1.1.0 | kernel.patch | 初始支持 |
| v1.1.0 | hdf.patch | HDF 适配 |
| v1.4.0 | kernel.patch | Linux 6.6 支持 |

## 已知问题

### 待解决

| 问题 | 状态 | 预期版本 |
|------|------|----------|
| Linux 6.6 完整支持 | 进行中 | v1.5.0 |
| 更多板卡支持 | 计划中 | v1.5.0 |

### 已解决

| 问题 | 解决版本 | 解决方案 |
|------|----------|----------|
| 补丁冲突 | v1.1.0 | 优化补丁结构 |
| 配置错误 | v1.2.0 | 更新 defconfig |

## 贡献者

| 版本 | 主要贡献者 |
|------|------------|
| v1.0.0 | OpenHarmony 团队 |
| v1.1.0 | Rockchip, NXP 团队 |
| v1.2.0 | Loongson 团队 |
| v1.3.0 | 社区贡献者 |
| v1.4.0 | OpenHarmony 团队 |

## 相关文档

- [支持的板卡](../03_Supported_Boards.md)
- [补丁分析](../04_Patches_Analysis.md)
- [构建系统](../05_Build_System.md)
