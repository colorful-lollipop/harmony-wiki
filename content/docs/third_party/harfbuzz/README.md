# HarfBuzz 在 OpenHarmony 中的集成

## 概述

HarfBuzz 是一个开源的文本塑形（Text Shaping）引擎，在 OpenHarmony 系统中作为核心文本渲染组件，为 UI 框架和图形引擎提供高质量的字符到字形转换能力。

## 在 OpenHarmony 中的定位

```
┌─────────────────────────────────────────────────────────────┐
│                      OpenHarmony 系统                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │  ArkUI      │    │  UI Lite    │    │  第三方应用  │    │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    │
│         │                   │                   │            │
│         ▼                   ▼                   ▼            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Rosen 2D 图形引擎                        │   │
│  │              (HarfBuzz 集成)                         │   │
│  └────────────────────────┬────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Skia 图形库                              │   │
│  │              (HarfBuzz 高级塑形)                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              HarfBuzz 11.0.0                         │   │
│  │              (文本塑形核心)                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 主要功能

### 1. 文本塑形（Text Shaping）

将 Unicode 字符序列转换为正确的字形序列和位置：
- 字符编码转换
- 字形索引查找
- 字形定位计算
- 字距调整（Kerning）

### 2. OpenType 支持

完整支持 OpenType 排版特性：
- 连字（Ligatures）
- 字形替代（Glyph Substitutions）
- 位置修饰（Mark Positioning）
- 锚点定位（Anchor Positioning）

### 3. 多语言支持

支持全球主要语言的文本渲染：
- Latin 脚本（英语、法语、德语等）
- CJK（中文、日文、韩文）
- Arabic/Hebrew（从右到左文本）
- Indic 脚本（Devanagari、Bengali 等）
- Thai/Lao

### 4. 彩色字体（COLR）

支持 OpenType COLR 彩色字体：
- Emoji 彩色渲染
- 彩色图标字体
- 渐变色字形

## OH 特有修改

OpenHarmony 对 HarfBuzz 进行了以下特定修改：

### Patch 概要

| 修改类型 | 修改文件数 | 代码行数 | 主要内容 |
|---------|-----------|---------|---------|
| ICCARM 适配 | 10 | 872 | 嵌入式编译器兼容 |
| COLR 增强 | 1 | 506 | 彩色字体支持 |
| glyf 支持 | 2 | 269 | TrueType 字形表支持 |

### 关键修改

1. **ICCARM 编译器支持**: 针对华为 ICCARM 编译器的特定适配
2. **彩色字体增强**: 新增 COLR Paint 类完整实现
3. **字形处理增强**: 扩展 glyf 字形表的处理能力

详见: [Patch 详细分析](02_Patches.md)

## 构建集成

### 支持的构建配置

| 配置 | 系统类型 | 工具链 | 特点 |
|-----|---------|-------|-----|
| 标准 | 标准系统 | GCC/Clang | 完整功能 |
| Lite | 轻量系统 | GCC/Clang | 精简配置 |
| ICCARM | 嵌入式 | ICCARM | 嵌入式优化 |

### 构建产物

- **静态库**: `libharfbuzz_static.a`
- **链接方式**: 静态链接
- **头文件**: `${gen_dir}/harfbuzz-11.0.0/src/`

详见: [构建适配说明](03_Build_Integration.md)

## 使用方式

### 直接依赖者

| 模块 | 用途 | 依赖类型 |
|-----|------|---------|
| Rosen 2D 图形 | 字体塑形 | 静态链接 |
| UI Lite | 轻量 UI | 静态链接 |
| Skia SkShaper | 高级文本塑形 | 外部依赖 |

详见: [依赖关系与使用](04_Usage_in_OH.md)

## 快速开始

### 基础使用

```cpp
#include <harfbuzz/hb.h>
#include <harfbuzz/hb-ft.h>

void shape_text(FT_Face ft_face, const char* text) {
    // 创建 HarfBuzz 缓冲区
    hb_buffer_t* buffer = hb_buffer_create();
    
    // 设置文本
    hb_buffer_add_utf8(buffer, text, strlen(text), 0, strlen(text));
    hb_buffer_guess_segment_properties(buffer);
    
    // 创建字体
    hb_font_t* hb_font = hb_ft_font_create(ft_face, nullptr);
    
    // 执行塑形
    hb_shape(hb_font, buffer, nullptr, 0);
    
    // 获取结果
    unsigned int glyph_count;
    hb_glyph_info_t* glyphs = hb_buffer_get_glyph_infos(buffer, &glyph_count);
    hb_glyph_position_t* positions = hb_buffer_get_glyph_positions(buffer, &glyph_count);
    
    // 清理
    hb_buffer_destroy(buffer);
    hb_ft_font_destroy(hb_font);
}
```

### 资源管理最佳实践

```cpp
// 使用 RAII 管理生命周期
class HarfBuzzShaper {
public:
    HarfBuzzShaper() {
        buffer_ = hb_buffer_create();
        font_ = nullptr;
    }
    
    ~HarfBuzzShaper() {
        if (buffer_) hb_buffer_destroy(buffer_);
    }
    
    void set_font(FT_Face face) {
        if (font_) hb_ft_font_destroy(font_);
        font_ = hb_ft_font_create(face, nullptr);
    }
    
private:
    hb_buffer_t* buffer_;
    hb_font_t* font_;
};
```

## 版本信息

| 项目 | 版本 |
|-----|------|
| HarfBuzz | 11.0.0 |
| OH 组件 | 3.1 |
| 许可证 | MIT |
| 上游地址 | https://github.com/harfbuzz/harfbuzz |

## 相关文档

1. [概述与导航](SUMMARY.md) - 阅读路线建议
2. [原始库简介](01_Overview.md) - HarfBuzz 基础信息
3. [Patch 详细分析](02_Patches.md) - OH 特有修改详解
4. [构建适配说明](03_Build_Integration.md) - 构建系统集成
5. [依赖关系与使用](04_Usage_in_OH.md) - 使用场景和集成方式

## 常见问题

### Q: 如何排查文本显示问题？

A: 检查以下几点：
1. 字体文件是否正确加载
2. 字符编码是否正确（UTF-8）
3. 字体是否包含所需字形
4. 启用 HarfBuzz 调试输出

### Q: ICCARM 编译失败怎么办？

A: 确保在 BUILD.gn 中定义 `ENABLE_ICCARM` 宏，并使用正确的工具链配置。

### Q: 如何支持彩色 Emoji？

A: 需要启用 COLR 支持，并使用支持彩色字体的字体文件（如 Noto Color Emoji）。

## 贡献者

- **维护者**: zhangyifan39@huawei.com
- **Patch 作者**: Zacoh (kouzhenrong@h-partners.com)

---

*本文档最后更新: 2024年*
