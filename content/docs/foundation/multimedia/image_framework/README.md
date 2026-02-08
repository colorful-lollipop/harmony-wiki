# Image Framework Wiki

## 简介

本文档是 OpenHarmony **Image Framework**（图像框架）的工程 Wiki，旨在帮助开发者快速理解项目结构、API 接口、架构设计和安全考量。

## 生成信息

- **生成时间**: 2025-02-06
- **版本**: 3.1
- **仓库**: foundation/multimedia/image_framework
- **支持格式**: JPEG, PNG, BMP, GIF, TIFF, RAW, HEIF, WebP, SVG, ASTC

## 文档导航

### 新人必读
1. [项目概览](00_Overview.md) - 项目定位、核心能力、运行环境
2. [目录结构](01_Directory_Structure.md) - 模块职责与代码组织
3. [架构说明](02_Architecture.md) - 组件关系、数据流、线程模型

### API 参考
4. [N-API 接口文档](03_NAPI_Reference.md) - JS/TS 层 API 完整参考
5. [内部 API 文档](04_Inner_API.md) - Native 层接口说明

### 构建与产物
6. [GN 构建目标](05_GN_Targets.md) - 构建系统与目标详解
7. [编译产物](06_Build_Artifacts.md) - 输出文件与安装路径

### 安全与问题
8. [安全风险分析](07_Security_Risks.md) - 攻击面与漏洞分析
9. [常见问题](08_FAQ.md) - 构建/运行/调试问题

### 附录
10. [调用链附录](appendix/Callgraphs.md) - 关键调用链
11. [配置标志附录](appendix/Config_Flags.md) - 编译配置选项

## 快速链接

### 核心入口文件
- [主模块注册](https://gitee.com/openharmony/multimedia_image_framework/blob/master/frameworks/kits/js/common/native_module_ohos_image.cpp) - `multimedia.image` 模块
- [Bundle 配置](https://gitee.com/openharmony/multimedia_image_framework/blob/master/bundle.json) - 组件元数据
- [根 BUILD.gn](https://gitee.com/openharmony/multimedia_image_framework/blob/master/BUILD.gn) - 构建入口

### 关键头文件
- [ImageSource](https://gitee.com/openharmony/multimedia_image_framework/blob/master/interfaces/innerkits/include/image_source.h)
- [ImagePacker](https://gitee.com/openharmony/multimedia_image_framework/blob/master/interfaces/innerkits/include/image_packer.h)
- [PixelMap](https://gitee.com/openharmony/multimedia_image_framework/blob/master/interfaces/innerkits/include/pixel_map.h)
- [Picture](https://gitee.com/openharmony/multimedia_image_framework/blob/master/interfaces/innerkits/include/picture.h)

### 系统能力
- `SystemCapability.Multimedia.Image.Core`
- `SystemCapability.Multimedia.Image.ImageSource`
- `SystemCapability.Multimedia.Image.ImagePacker`
- `SystemCapability.Multimedia.Image.ImageReceiver`
- `SystemCapability.Multimedia.Image.ImageCreator`

## 更新说明

本文档基于代码证据自动生成，关键结论均可追溯至代码中的：
- 文件路径与行号
- 函数/类/宏定义
- 代码片段与调用链

如需更新文档，请遵循 [Phase 工作流](_work/PLAN.md)。
