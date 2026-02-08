# libexif in OpenHarmony

本文档详细说明了 OpenHarmony (OH) 对第三方库 **libexif** 的集成、适配和使用方式。

## 库概述

**libexif** 是一个用于解析、编辑和保存 EXIF 数据的 C 语言库。EXIF (Exchangeable Image File Format) 是一种文件格式，用于在 JPEG、TIFF 等图像格式中嵌入元数据（如相机型号、拍摄时间、GPS 位置等）。

### 在 OpenHarmony 中的定位

- **子系统**: thirdparty
- **组件名称**: @ohos/libexif
- **版本**: v0.6.25 (上游), 3.1 (OH 组件版本)
- **许可证**: LGPL-2.1+
- **资源占用**: ROM 196KB, RAM 392KB

libexif 是 OH **多媒体子系统** 的基础组件，主要用于：
- 图像编解码插件提取 JPEG 图像的 EXIF 信息
- 摄像头驱动处理拍摄图像的元数据
- 华为相机特色功能的 Maker Note 解析（XMAGE、XTStyle 等）

## OH 适配概述

OpenHarmony 对 libexif 的适配采用**扩展式设计**，而非传统的 Patch 修改方式：

### 核心适配方式

1. **无 Patch 文件**
   - OH 不使用 `.patch` 文件修改上游代码
   - 便于上游版本升级和合并

2. **华为 Maker Note 扩展**
   - 添加 `libexif/huawei/` 目录（~31KB）
   - 完整实现华为相机的 EXIF Maker Note 解析
   - 支持 XMAGE、XTStyle、云增强、AI 编辑等华为特色功能

3. **BUILD.gn 集成**
   - 支持 OH Lite（轻量级系统）和标准系统
   - 提供 `exif_static`（静态库）和 `libexif`（共享库）两个构建目标
   - 集成安全加固（bounds_checking_function、分支保护）

4. **安全加固**
   - 集成 `bounds_checking_function` 边界检查库
   - 启用 `branch_protector_ret = "pac_ret"` 分支保护
   - 严格编译选项（`-Werror`）

### OH 价值

| 方面 | OH 价值 |
|------|---------|
| **华为相机支持** | 完整解析华为 Maker Note，支持 XMAGE、XTStyle 等特色功能 |
| **多媒体框架** | 图像编解码插件的基础依赖 |
| **摄像头驱动** | 支持 USB 摄像头、V4L2 管线的 EXIF 处理 |
| **安全性** | 边界检查、分支保护、大量 fuzzer 测试 |
| **可升级性** | 无 Patch，便于跟踪上游更新 |

## 文档导航

### 快速开始

1. **[01_Overview.md](01_Overview.md)** - libexif 原始库简介及在 OH 中的作用
2. **[02_Patches.md](02_Patches.md)** - OH 扩展实现分析（华为 Maker Note）
3. **[03_Build_Integration.md](03_Build_Integration.md)** - BUILD.gn 构建系统适配
4. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - OH 依赖关系与使用模式
5. **[05_API_Differences.md](05_API_Differences.md)** - API 差异与 OH 特定功能
6. **[06_Security.md](06_Security.md)** - 安全风险分析、CVE 修复状态

### 工作文档

- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 项目评估结果（Phase 0 完成报告）
- **[_work/NOTES.md](_work/NOTES.md)** - 分析过程记录
- **[_work/PLAN.md](_work/PLAN.md)** - 任务进度跟踪

### 阅读路线建议

参见 **[SUMMARY.md](SUMMARY.md)**。

## 核心特性

### 1. 华为 Maker Note 支持

libexif 在 OH 中新增了完整的华为 Maker Note 实现，包括：

- **拍摄信息**: 捕获模式、连拍数量、摄像头姿态（翻滚/俯仰角）、物理光圈
- **XMAGE 功能**: 华为影像技术，支持模式选择和裁剪区域
- **云增强与 AI**: 云增强模式、AI 编辑、风抓拍
- **运动照片**: 运动照片版本、微视频时间戳、运动照片 ID
- **XTStyle 滤镜**: 多种滤镜效果（明暗、饱和度、色调、暗角、噪点）
- **场景识别**: 12 种场景（美食、舞台、蓝天、绿植、海滩、雪、日落、花朵、夜景、文字）
- **人脸识别**: 人脸数量、置信度、微笑评分、人脸矩形、五官位置

详细标签定义参见 **[05_API_Differences.md](05_API_Differences.md)**。

### 2. 多厂商 Maker Note 支持

除华为外，OH 的 libexif 还包含以下厂商的 Maker Note 实现：
- Apple（iOS 相机）
- Canon（佳能）
- Fuji（富士）
- Olympus（奥林巴斯）
- Pentax（宾得）

### 3. 安全加固

- **边界检查**: 集成 `bounds_checking_function`，防止缓冲区溢出
- **分支保护**: `branch_protector_ret = "pac_ret"` 保护返回地址
- **严格编译**: `-Werror` 将警告视为错误
- **模糊测试**: 14+ 个 fuzzer 测试确保对恶意 EXIF 数据的安全性

## 依赖关系

### 直接依赖者（21+ 个）

- **图像框架**: libjpegplugin、libextplugin、librawplugin
- **摄像头驱动**: usb_camera、v4l2 pipeline
- **开发板**: Dayu210、RK3568 摄像头实现
- **测试**: 14+ 个 fuzzer 测试

详细依赖分析参见 **[04_Usage_in_OH.md](04_Usage_in_OH.md)**。

## 安全性

libexif 是一个处理不可信数据的库，主要威胁面包括：
- 内存破坏（缓冲区溢出、越读越写）
- 拒绝服务（无限循环、整数溢出导致的计算时间过长）
- 除零错误

### 历史 CVE 修复

上游从 2006 年至 2020 年修复了 17 个 CVE，涵盖：
- 缓冲区溢出
- 整数溢出
- 未初始化内存使用
- DoS 攻击
- 除零错误

详细 CVE 列表和 OH 修复状态参见 **[06_Security.md](06_Security.md)**。

## 上游信息

- **仓库**: https://github.com/libexif/libexif
- **版本**: v0.6.25 (2025-01-08 发布)
- **许可证**: LGPL-2.1+
- **文档**: https://libexif.sourceforge.io/

## 贡献与维护

- **华为适配代码**: Copyright (C) 2024 Huawei Device Co., Ltd.
- **许可证**: Apache License 2.0（华为扩展代码）
- **上游代码**: LGPL-2.1+

## 快速链接

- [原始库简介](01_Overview.md)
- [OH 扩展实现](02_Patches.md)
- [构建系统适配](03_Build_Integration.md)
- [依赖关系分析](04_Usage_in_OH.md)
- [API 差异](05_API_Differences.md)
- [安全分析](06_Security.md)
- [项目评估](_work/ASSESSMENT.md)
