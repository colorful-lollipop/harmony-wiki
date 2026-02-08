# 文档导航

本文档为 OpenHarmony `media_utils_lite` 组件的完整技术文档。

## 新人阅读路线

建议按以下顺序阅读：

1. **[项目概述](01_Overview.md)** → 了解组件定位、核心能力
2. **[目录结构](01_Overview.md#目录结构)** → 掌握模块划分
3. **[架构设计](02_Architecture.md)** → 理解组件关系与数据流
4. **[API 参考](03_API_Reference.md)** → 查阅接口定义
5. **[构建配置](04_Build.md)** → 了解编译方式
6. **[安全评审](05_Security.md)** → 关注安全风险

## 完整目录

### 核心文档

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [README](README.md) | 文档说明 | 覆盖范围、更新方式 |
| [项目概述](01_Overview.md) | 组件定位与结构 | 项目简介、目录结构、能力列表 |
| [架构设计](02_Architecture.md) | 组件架构 | 架构图、HAL 抽象、数据流 |
| [API 参考](03_API_Reference.md) | 接口文档 | 类型定义、错误码、头文件清单 |
| [构建配置](04_Build.md) | 构建与编译 | GN targets、依赖关系、产物说明 |
| [安全评审](05_Security.md) | 安全风险分析 | 攻击面、风险点、修复建议 |

### 附录

| 文档 | 说明 |
|------|------|
| [调用链图谱](appendix/Callgraphs.md) | 关键调用链路 |
| [配置参数](appendix/Config_Flags.md) | 构建开关与宏定义 |

## 快速索引

### 按功能查找

**数据类型**:
- [SourceType - 媒体源类型](03_API_Reference.md#sourceh---媒体源)
- [AudioSourceType - 音频源枚举](03_API_Reference.md#audio-source-type)
- [AudioCodecFormat - 音频编解码](03_API_Reference.md#audio-codec-format)
- [VideoCodecFormat - 视频编解码](03_API_Reference.md#video-codec-format)

**错误码**:
- [错误码定义](03_API_Reference.md#错误码定义)
- [错误码速查表](03_API_Reference.md#错误码速查表)

**核心类**:
- [Source 类](03_API_Reference.md#source-类)
- [StreamSource 类](03_API_Reference.md#streamsource-类)
- [Format 类](03_API_Reference.md#format-类)
- [DataStream 类](03_API_Reference.md#datastream-类)

### 按文件查找

| 文件 | 文档位置 |
|------|----------|
| `media_errors.h` | [错误码定义](03_API_Reference.md#media_errorsh) |
| `source.h` | [媒体源接口](03_API_Reference.md#sourceh) |
| `format.h` | [格式化数据](03_API_Reference.md#formath) |
| `media_info.h` | [媒体信息枚举](03_API_Reference.md#media_infoh) |
| `data_stream.h` | [数据流接口](03_API_Reference.md#data_streamh) |
| `hal_camera.h` | [相机 HAL](03_API_Reference.md#hal_camerah) |
| `hal_display.h` | [显示 HAL](03_API_Reference.md#hal_displayh) |
