# 全站导航

## 新人阅读顺序

建议按以下顺序阅读，逐步深入理解项目：

1. **[README](README.md)** - 了解 Wiki 覆盖范围和更新方式
2. **[项目概览](00_Overview.md)** - 理解项目定位、核心能力和运行环境
3. **[目录结构](02_Directory_Structure.md)** - 熟悉代码组织方式
4. **[架构说明](01_Architecture.md)** - 理解系统架构和数据流
5. **[HDI/VDI 接口](03_HDI_Interfaces.md)** - 了解对外接口定义
6. **[GN 构建系统](04_GN_Build.md)** - 掌握构建配置
7. **[编译产物](05_Compilation_Products.md)** - 了解输出文件
8. **[安全风险评审](06_Security_Analysis.md)** - 了解安全注意事项
9. **[常见问题](07_Troubleshooting.md)** - 掌握调试技巧

## 按主题导航

### 入门必读
- [项目概览](00_Overview.md) - 项目定位、边界、核心能力
- [目录结构](02_Directory_Structure.md) - 模块职责与文件组织

### 架构设计
- [架构说明](01_Architecture.md) - 组件图、数据流、线程模型
- [附录：调用链](appendix/Callgraphs.md) - 关键调用链分析

### 接口文档
- [HDI/VDI 接口](03_HDI_Interfaces.md) - 对外接口清单
  - Display Composer VDI
  - Display Buffer VDI
  - Codec VDI
  - MPP 接口

### 构建与产物
- [GN 构建系统](04_GN_Build.md) - Targets、依赖、配置
- [编译产物](05_Compilation_Products.md) - 产物清单与加载关系

### 安全与调试
- [安全风险评审](06_Security_Analysis.md) - 攻击面与修复建议
- [常见问题](07_Troubleshooting.md) - 构建/运行/调试问题

## 芯片平台速查

| 芯片 | 概览 | 硬件模块 | 内核驱动 |
|------|------|----------|----------|
| RK2206 | [详情](00_Overview.md#rk2206) | HDF驱动 | LiteOS-M |
| RK3399 | [详情](00_Overview.md#rk3399) | display, mpp, rga | Linux |
| RK3566 | [详情](00_Overview.md#rk3566) | display, gpu, isp, mpp, rga | Linux |
| RK3568 | [详情](00_Overview.md#rk3568) | display, gpu, isp, mpp, rga, wifi, codec | Linux |
| RK3588 | [详情](00_Overview.md#rk3588) | display, gpu, isp, mpp, rga, wifi, codec | Linux |

## 硬件模块速查

| 模块 | 说明 | 位置 |
|------|------|------|
| Display | 显示输出 (DRM/GBM) | `*/hardware/display/` |
| GPU | Mali GPU 驱动 | `*/hardware/gpu/`, `common/kernel/drivers/gpu/` |
| ISP | 图像信号处理 | `*/hardware/isp/` |
| MPP | 媒体处理平台 | `*/hardware/mpp/`, `common/hardware/mpp/` |
| RGA | 2D 图形加速 | `*/hardware/rga/`, `common/hardware/rga/` |
| WiFi | 无线网卡驱动 | `*/hardware/wifi/`, `common/kernel/drivers/net/wireless/` |
| Codec | 音视频编解码 | `*/hardware/codec/`, `*/hardware/omx_il/` |

## 快速链接

### 关键文件
- [根 BUILD.gn](../BUILD.gn) - 根构建配置
- [RK3568 soc.gni](../rk3568/soc.gni) - RK3568 构建配置
- [RK2206 board.gni](../rk2206/board.gni) - RK2206 构建配置

### 关键代码
- [Display HDI Session](../common/hardware/display/src/display_device/hdi_session.cpp) - 显示会话管理
- [MPP MPI 接口](../common/hardware/mpp/include/rk_mpi.h) - MPP 主接口
- [DRM 认证](../common/sdk_linux/drivers/gpu/drm/drm_auth.c) - DRM 权限控制

## 外部参考

- [OpenHarmony HDI 框架](https://gitee.com/openharmony/drivers_hdf_core)
- [Rockchip MPP 文档](https://github.com/rockchip-linux/mpp)
- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
