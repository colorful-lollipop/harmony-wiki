# HarfBuzz 原始库简介

本文档简要介绍 HarfBuzz 文本塑形引擎的基本信息、功能特性和技术架构，为理解其在 OpenHarmony 中的集成提供基础背景。

## 1 项目概述

### 1.1 基本信息

| 项目 | 内容 |
|-----|------|
| **名称** | HarfBuzz |
| **当前版本** | 11.0.0 |
| **许可证** | MIT License |
| **上游地址** | https://github.com/harfbuzz/harfbuzz |
| **初始发布** | 2012年 |
| **主要维护者** | Behdad Esfahbod 等 |

### 1.2 项目定位

HarfBuzz 是一个跨平台的开源文本塑形（Text Shaping）引擎，其主要职责是将Unicode字符序列转换为正确的字形（Glyph）序列和位置信息。这个过程是现代字体渲染系统的核心环节。

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Unicode   │ ──▶ │  HarfBuzz  │ ──▶ │  FreeType  │ ──▶ │   屏幕/    │
│  字符序列  │     │  文本塑形  │     │  字形渲染  │     │   打印机    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### 1.3 应用领域

HarfBuzz 被广泛应用于各类需要高质量文本渲染的场景：

| 领域 | 代表应用 |
|-----|---------|
| **操作系统** | Android、ChromeOS、GNOME、KDE |
| **浏览器** | Chrome、Firefox、Edge |
| **办公软件** | LibreOffice、OpenJDK |
| **图形系统** | Qt、Skia GTK |
| **排版系统** | XeTeX、LaTeX |
| **游戏主机** | PlayStation |

---

## 2 核心功能

### 2.1 文本塑形（Text Shaping）

文本塑形是 HarfBuzz 的核心功能，其过程包括：

#### 2.1.1 字符分析

- **字符分类**: 识别字母、数字、标点、空白等
- **脚本检测**: 确定文本所属的书写系统（Latin、Cyrillic、Arabic 等）
- **语言识别**: 确定文本语言以应用正确的排版规则

#### 2.1.2 字形查找

- **字体查询**: 在字体文件中查找对应字符的字形索引
- **变体选择**: 处理 OpenType 字形变体（alternates）
- **后备字体**: 当主字体缺少字形时的字体回退

#### 2.1.3 字形定位

- **位置计算**: 确定每个字形在文本中的精确位置
- **字距调整**: 处理字形间的间距（kerning）
- **基线对齐**: 处理不同脚本的基线差异

### 2.2 OpenType 支持

HarfBuzz 完整支持 OpenType 规范的排版特性：

#### 2.2.1 字形替代（GSUB）

| 特性 | 标签 | 说明 |
|-----|------|-----|
| Standard Ligatures | `liga` | 标准连字（如 fi、fl） |
| Contextual Ligatures | `clig` | 上下文连字 |
| Small Caps | `smcp` | 小型大写字母 |
| Oldstyle Figures | `onum` | 旧式（文本）数字 |
| Subscript | `subs` | 下标字符 |
| Superscript | `sups` | 上标字符 |
| Slashed Zero | `zero` | 带斜线的零 |

#### 2.2.2 字形定位（GPOS）

| 特性 | 标签 | 说明 |
|-----|------|-----|
| Kerning | `kern` | 字距调整 |
| Mark to Base | `mark` | 组合标记定位 |
| Mark to Mark | `mkmk` | 标记间定位 |
| Cursive Positioning | `curs` | 阿拉伯文连字定位 |

### 2.3 多脚本支持

HarfBuzz 支持全球主要书写系统：

| 脚本类别 | 示例语言 | 特殊处理 |
|---------|---------|---------|
| **Latin** | 英语、法语、西班牙语 | 连字、字距 |
| **CJK** | 中文、日文、韩文 | 大字符集、字体回退 |
| **Arabic** | 阿拉伯语、波斯语 | RTL、双向文本 |
| **Hebrew** | 希伯来语 | RTL 文本 |
| **Indic** | 印地语、泰米尔语 | 复合元音、形态重排 |
| **Thai/Lao** | 泰语、老挝语 | 前后位置标记 |
| **Ethiopic** | 阿姆哈拉语 | 特殊字形组合 |
| **Mongolian** | 蒙古语 | 垂直排版 |

### 2.4 可变字体支持

HarfBuzz 11.0.0 完整支持 OpenType 1.8+ 的可变字体（Variable Fonts）：

```cpp
// 设置可变字体轴
hb_font_set_variations(font, "wght", 700.0, "wdth", 100.0, nullptr);

// 获取可用轴
hb_ot_font_get_var_axes(face, &axis_count, &axes);
```

### 2.5 彩色字体支持

HarfBuzz 支持 OpenType COLR 彩色字体规范：

```
COLR 表结构:
┌─────────────────────────────────────┐
│           PaintGlyph                │  ──▶ 引用字形
├─────────────────────────────────────┤
│ PaintTranslate / Scale / Rotate      │  ──▶ 变换
├─────────────────────────────────────┤
│ PaintComposite                      │  ──▶ 混合模式
└─────────────────────────────────────┘
```

支持的功能：
- Emoji 彩色渲染（🍎、🚗、😊）
- 彩色图标字体
- 渐变色字形
- 多层颜色叠加

---

## 3 技术架构

### 3.1 模块结构

```
HarfBuzz
├── hb.h              # 核心 API
├── hb-blob.h         # 二进制数据管理
├── hb-buffer.h        # 塑形缓冲区
├── hb-font.h         # 字体度量
├── hb-face.h         # 字体字形数据
├── hb-ft.h           # FreeType 集成
├── hb-ot.h           # OpenType 布局
├── hb-ucd.h          # Unicode 数据
│
├── src/
│   ├── hb.cc         # 核心实现
│   ├── hb-buffer.cc  # 缓冲区实现
│   ├── hb-face.cc    # 字形面实现
│   ├── hb-font.cc    # 字体实现
│   ├── hb-shape.cc   # 塑形主逻辑
│   ├── hb-static.cc  # 静态数据
│   │
│   ├── hb-ot-*.cc    # OpenType 表格处理
│   │   ├── hb-ot-layout.cc
│   │   ├── hb-ot-cmap-table.cc
│   │   ├── hb-ot-font.cc
│   │   └── ...
│   │
│   ├── hb-ot-shaper-*.cc  # 各脚本塑形器
│   │   ├── hb-ot-shaper-arabic.cc
│   │   ├── hb-ot-shaper-indic.cc
│   │   └── ...
│   │
│   └── OT/           # OpenType 表格定义
│       ├── Layout/
│       ├── Color/
│       └── glyf/
```

### 3.2 核心数据类型

#### 3.2.1 hb_buffer_t（塑形缓冲区）

```cpp
// 缓冲区存储:
// 1. 输入: Unicode 字符序列
// 2. 输出: 字形信息和位置

hb_buffer_t* buffer = hb_buffer_create();

hb_buffer_add_utf8(buffer, "Hello", 5, 0, 5);
// 缓冲区内容:
// [0x0048, 0x0065, 0x006C, 0x006C, 0x006F]

hb_shape(font, buffer, nullptr, 0);
// 缓冲区内容:
// [GlyphInfo('H', gid=10, cluster=0), 
//  GlyphInfo('e', gid=32, cluster=1), ...]
```

#### 3.2.2 hb_font_t（字体对象）

```cpp
// 字体对象包含:
// 1. 字体度量信息（em units, ascent, descent）
// 2. 变体轴设置
// 3. 回调函数

hb_font_t* font = hb_ft_font_create(ft_face, nullptr);

// 设置变体
hb_font_set_variations(font, "wght", 700.0, nullptr);
```

#### 3.2.3 hb_face_t（字形面）

```cpp
// 字形面对象包含:
// 1. 指向字体二进制数据的指针
// 2. 字形计数
// 3. 各 OpenType 表的加速器（accelerator）

hb_face_t* face = hb_face_create_from_file("font.ttf", 0);

// 获取字形计数
unsigned int glyph_count = hb_face_get_glyph_count(face);  // 例如: 1234
```

### 3.3 塑形流程

```
┌─────────────────────────────────────────────────────────────────────┐
│                         hb_shape() 流程                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. guess_segment_properties()                                       │
│     ├─ 检测脚本 (hb_ot_tag_from_script)                              │
│     ├─ 检测语言 (hb_ot_tag_from_language)                            │
│     └─ 设置方向 (LTR/RTL/TTB)                                        │
│                                                                      │
│  2. 脚本特定塑形器                                                    │
│     ├─ hb_ot_shaper_collect_features()                               │
│     ├─ hb_ot_shaper_shape()                                         │
│     └─ 各脚本: Arabic, Indic, Thai, Hangul...                        │
│                                                                      │
│  3. 布局应用                                                         │
│     ├─ GSUB: 应用字形替代                                            │
│     └─ GPOS: 应用字形定位                                            │
│                                                                      │
│  4. 后处理                                                           │
│     ├─ 方向镜像 (RTL 字符)                                           │
│     ├─ 组合标记重新排序                                              │
│     └─ 输出格式化                                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4 API 概览

### 4.1 版本检查

```cpp
#include <hb-version.h>

unsigned int major, minor, micro;
hb_version(&major, &minor, &micro);

if (HB_VERSION_CHECK(major, minor, micro)) {
    // HarfBuzz 版本兼容
}
```

### 4.2 缓冲区操作

```cpp
// 创建和销毁
hb_buffer_t* hb_buffer_create();
void hb_buffer_destroy(hb_buffer_t* buffer);

// 添加文本
hb_buffer_add_utf8(hb_buffer_t* buffer, 
                   const char* text, int text_len,
                   int item_offset, int item_length);
hb_buffer_add_utf16(hb_buffer_t* buffer, ...);
hb_buffer_add_utf32(hb_buffer_t* buffer, ...);

// 设置属性
void hb_buffer_guess_segment_properties(hb_buffer_t* buffer);
void hb_buffer_set_direction(hb_buffer_t* buffer, hb_direction_t direction);
void hb_buffer_set_script(hb_buffer_t* buffer, hb_script_t script);
void hb_buffer_set_language(hb_buffer_t* buffer, hb_language_t language);

// 获取结果
hb_glyph_info_t* hb_buffer_get_glyph_infos(hb_buffer_t* buffer, 
                                           unsigned int *length);
hb_glyph_position_t* hb_buffer_get_glyph_positions(hb_buffer_t* buffer,
                                                   unsigned int *length);
```

### 4.3 字体操作

```cpp
// 创建
hb_font_t* hb_ft_font_create(FT_Face ft_face, 
                            hb_destroy_func_t destroy);
hb_font_t* hb_font_create_from_blob(hb_blob_t* blob, 
                                    int length,
                                    hb_destroy_func_t destroy);

// 变体设置
void hb_font_set_variations(hb_font_t* font,
                            const char* first_variation,
                            ...);  // 以 nullptr 结尾
```

### 4.4 核心塑形

```cpp
hb_shape(hb_font_t* font, 
         hb_buffer_t* buffer,
         const hb_feature_t* features,
         int num_features);

// 特性示例
hb_feature_t features[] = {
    { HB_TAG('l','i','g','a'), 1, 0, UINT_MAX },  // 启用连字
    { HB_TAG('k','e','r','n'), 1, 0, UINT_MAX },  // 启用字距
};
hb_shape(font, buffer, features, 2);
```

### 4.5 字形信息

```cpp
// 获取字形索引
hb_codepoint_t hb_ft_face_get_glyph_index(FT_Face ft_face, 
                                          hb_codepoint_t unicode);

// 获取字形边界
hb_bool_t hb_ft_font_get_glyph_extents(hb_font_t* font,
                                       hb_codepoint_t glyph,
                                       hb_glyph_extents_t *extents);

// 获取字形名称
char* hb_ft_font_get_glyph_name(FT_Face ft_face,
                                hb_codepoint_t glyph,
                                char *name, unsigned int size);
```

---

## 5 性能特性

### 5.1 内存模型

HarfBuzz 提供多种内存配置选项：

| 模式 | 宏定义 | 内存占用 | 适用场景 |
|-----|-------|---------|---------|
| **标准** | 默认 | 中等 | 桌面系统 |
| **精简** | HB_TINY | 低 | 移动设备 |
| **微型** | HB_TINY + 定制 | 极低 | 嵌入式 |

### 5.2 缓存机制

```cpp
// HarfBuzz 内部缓存:
// 1. 字形面缓存 (hb_face_t)
// 2. 字体缓存 (hb_font_t)
// 3. 特性查找缓存
// 4. 字形信息缓存

// 用户层缓存建议:
class FontCache {
    std::map<std::string, hb_face_t*> cache_;
};
```

### 5.3 多线程支持

```cpp
// ✅ 线程安全的使用方式:
// - 每个线程创建独立的 hb_buffer_t
// - hb_face_t 可以安全共享（只读）
// - hb_font_t 应在线程间传递或复制

void thread_func(FT_Face shared_face) {
    hb_buffer_t* buffer = hb_buffer_create();  // 线程本地
    hb_font_t* font = hb_ft_font_create(shared_face, nullptr);  // 共享
    
    // ... 使用 ...
    
    hb_buffer_destroy(buffer);
    hb_ft_font_destroy(font);
}
```

---

## 6 与 OpenHarmony 的集成点

### 6.1 OH 特有修改

OpenHarmony 对 HarfBuzz 进行了以下特定修改：

| 修改项 | 目的 | 影响 |
|-------|------|-----|
| ICCARM 适配 | 嵌入式编译器支持 | 条件编译路径 |
| COLR 增强 | 彩色 Emoji 支持 | 新增 Paint 类实现 |
| glyf 增强 | TrueType 字形表优化 | ICCARM 特定实现 |

详见: [Patch 详细分析](02_Patches.md)

### 6.2 在 OH 中的角色

```
OpenHarmony 渲染栈:
┌─────────────────────────────────────────────────────────┐
│  应用层                                                  │
│  (Ace Engine / Native UI)                                │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Rosen 图形引擎                                         │
│  HarfBuzz 集成: font_harfbuzz.cpp                       │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Skia 图形库                                            │
│  HarfBuzz 集成: SkShaper (高级文本塑形)                  │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  HarfBuzz 11.0.0                                        │
│  (OpenType 塑形引擎)                                    │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  FreeType / 字体渲染器                                  │
└─────────────────────────────────────────────────────────┘
```

### 6.3 依赖关系

HarfBuzz 在 OH 中被以下模块使用：

| 模块 | 用途 | 链接方式 |
|-----|------|---------|
| Rosen | 基础文本塑形 | 静态链接 |
| UI Lite | 轻量文本 | 静态链接 |
| Skia | 高级塑形 | 外部依赖 |

详见: [依赖关系与使用](04_Usage_in_OH.md)

---

## 7 学习资源

### 7.1 官方资源

- **GitHub**: https://github.com/harfbuzz/harfbuzz
- **文档**: https://harfbuzz.github.io/
- **API 文档**: https://harfbuzz.github.io/user-manual.html
- **源码**: https://github.com/harfbuzz/harfbuzz/tree/main/src

### 7.2 相关标准

- **OpenType 规范**: https://docs.microsoft.com/en-us/typography/opentype/
- **Unicode 规范**: https://unicode.org/
- **FreeType**: https://www.freetype.org/

### 7.3 社区

- **邮件列表**: harfbuzz@lists.freedesktop.org
- **问题报告**: https://github.com/harfbuzz/harfbuzz/issues

---

## 8 快速参考

### 最小使用示例

```cpp
#include <harfbuzz/hb.h>
#include <harfbuzz/hb-ft.h>

int main() {
    FT_Library ft_library;
    FT_Init_FreeType(&ft_library);
    
    FT_Face ft_face;
    FT_New_Face(ft_library, "font.ttf", 0, &ft_face);
    FT_Set_Char_Size(ft_face, 0, 16*64, 300, 300);
    
    hb_buffer_t* buffer = hb_buffer_create();
    hb_font_t* font = hb_ft_font_create(ft_face, nullptr);
    
    const char* text = "Hello, HarfBuzz!";
    hb_buffer_add_utf8(buffer, text, strlen(text), 0, strlen(text));
    hb_buffer_guess_segment_properties(buffer);
    
    hb_shape(font, buffer, nullptr, 0);
    
    unsigned int count;
    hb_glyph_info_t* glyphs = hb_buffer_get_glyph_infos(buffer, &count);
    hb_glyph_position_t* positions = hb_buffer_get_glyph_positions(buffer, &count);
    
    for (unsigned int i = 0; i < count; i++) {
        printf("Glyph: %u, x: %d, y: %d\n",
               glyphs[i].codepoint,
               positions[i].x_offset,
               positions[i].y_offset);
    }
    
    hb_buffer_destroy(buffer);
    hb_ft_font_destroy(font);
    FT_Done_Face(ft_face);
    FT_Done_FreeType(ft_library);
    
    return 0;
}
```

### 编译命令

```bash
# 使用 FreeType 编译
gcc -o shapetext shapetext.c \
    $(pkg-config --cflags harfbuzz freetype2) \
    $(pkg-config --libs harfbuzz freetype2)
```

---

*本文档最后更新: 2024年*
