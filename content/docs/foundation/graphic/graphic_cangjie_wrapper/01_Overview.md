# 项目概览 - graphic_cangjie_wrapper

## 项目定位

**graphic_cangjie_wrapper** 是 OpenHarmony Graphics 子系统的 Cangjie（仓颉）语言封装层，为应用开发者提供色彩管理（Color Management）能力 API。

> **证据**: `README.md:5` - "The graphic_cangjie_wrapper is a Cangjie API encapsulated on OpenHarmony for application developers to provide Cangjie color management capability API."

## 核心能力

| 能力 | 描述 | 证据来源 |
|-----|-----|---------|
| 标准色域创建 | 支持 32 种预设色域类型 | `color_space_primaries.cj:33-323` |
| 自定义色域创建 | 支持通过 ColorSpacePrimaries 自定义色域 | `color_space_primaries.cj:403-502` |
| 色域信息获取 | 获取色域类型、白点坐标、伽马值 | `color_space_manager.cj:116-165` |

## 系统定位

```
用户应用层 (ArkTS/Cangjie)
         ↓
图形图像仓颉接口 (kit.ArkGraphics2D)
         ↓
色彩管理封装 (color_space_manager)
         ↓
graphic_2d (native FFI)
```

> **证据**: `README.md:22` - "Color Management Wrapper: Provide gamut-dependent configuration capabilities."

## 运行环境

| 属性 | 值 | 证据来源 |
|-----|-----|---------|
| 适应系统 | standard（标准设备） | `bundle.json:17` |
| API Level | 22 | `color_space_primaries.cj:30,44,70` 等 |
| System Capability | SystemCapability.Graphic.Graphic2D.ColorManager.Core | `color_space_primaries.cj:31,45,71` 等 |

## 关键概念

### ColorSpace（色域枚举）

定义设备支持的色彩空间标准，包含：
- **消费级标准**: sRGB, Display P3, Adobe RGB
- **广播标准**: BT.709, BT.601, BT.2020
- **HDR 标准**: HLG, PQ (Perceptual Quantizer)
- **线性空间**: Linear sRGB, Linear P3, Linear BT.2020
- **Limited 变体**: 限制范围版本

### ColorSpacePrimaries（色域基色）

自定义色域时需指定的 8 个坐标参数：
- 红、绿、蓝三原色坐标 (redX/Y, greenX/Y, blueX/Y)
- 白点坐标 (whitePointX, whitePointY)

### ColorSpaceManager（色域管理器）

核心 API 类，提供：
- 工厂方法创建色域实例
- 查询色域属性（类型、白点、伽马值）
- 内部 FFI 调用 graphic_2d 实现

## 约束与限制

与 ArkTS 提供的 API 能力相比，暂不支持：

| 不支持功能 | 链接 |
|-----------|-----|
| 图像效果（EffectKit） | [js-apis-effectKit](https://gitcode.com/openharmony/docs/blob/master/en/application-dev/reference/apis-arkgraphics2d/js-apis-effectKit.md) |
| 可变帧率（DisplaySync） | [js-apis-graphics-displaySync](https://gitcode.com/openharmony/docs/blob/master/en/application-dev/reference/apis-arkgraphics2d/js-apis-graphics-displaySync.md) |
| HDR 能力 | [js-apis-hdrCapability](https://gitcode.com/openharmony/docs/blob/master/en/application-dev/reference/apis-arkgraphics2d/js-apis-hdrCapability.md) |
| 文本布局 | [js-apis-graphics-text](https://gitcode.com/openharmony/docs/blob/master/en/application-dev/reference/apis-arkgraphics2d/js-apis-graphics-text.md) |
| 效果级联（UIEffect） | [js-apis-uiEffect](https://gitcode.com/openharmony/docs/blob/master/en/application-dev/reference/apis-arkgraphics2d/js-apis-uiEffect.md) |

> **证据**: `README.md:54-60`

## 相关文档

- [色彩管理开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/graphics/cj-color-manager-development-guide.md)
- [Architecture](02_Architecture.md)
- [N-API](03_N-API.md)
