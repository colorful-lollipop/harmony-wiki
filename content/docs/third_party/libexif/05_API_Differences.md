# API/接口差异

## 概述

OpenHarmony 对 libexif 的 API 扩展主要通过**华为 Maker Note 模块**提供。本文档记录 OH 新增的 API、行为变更的 API 以及废弃/禁用的功能。

### API 扩展方式

| 方式 | 说明 |
|------|------|
| **新增华为 Maker Note API** | 添加独立的 Huawei 命名空间和函数 |
| **保留上游 API** | 完全兼容上游 libexif API |
| **无修改上游 API** | 不修改现有函数签名或行为 |

## 华为 Maker Note API

### 1. 核心数据结构

#### ExifMnoteDataHuawei

```c
struct _ExifMnoteDataHuawei {
    ExifMnoteData parent;  // ← 继承自基础 MnoteData

    MnoteHuaweiEntry *entries;      // 华为标签条目数组
    unsigned int count;              // 条目数量

    ExifByteOrder order;           // 字节序
    unsigned int offset;            // 数据偏移
    unsigned int ifd_tag;          // IFD 标签
    unsigned int ifd_size;          // IFD 大小
    unsigned int is_loaded;         // 是否已加载
};
typedef struct _ExifMnoteDataHuawei ExifMnoteDataHuawei;
```

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 32-43 行)

#### MnoteHuaweiEntry

```c
typedef struct _MnoteHuaweiEntry MnoteHuaweiEntry;
```

详细定义见 `libexif/huawei/mnote-huawei-data-type.h`

### 2. 识别与创建

#### exif_mnote_data_huawei_identify()

```c
int exif_mnote_data_huawei_identify(
    const ExifData *ed,    // 图像 EXIF 数据
    const ExifEntry *e        // Maker Note 条目
);
```

**功能**: 识别 EXIF 数据中的 Maker Note 是否为华为格式

**返回值**:
- `0`: 不是华为 Maker Note
- `非 0`: 是华为 Maker Note，可能返回子类型标识

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 61 行)

**OH 需求**: 图像框架需要识别华为相机拍摄的图片，以正确解析华为 Maker Note。

#### exif_mnote_data_huawei_new()

```c
ExifMnoteData *exif_mnote_data_huawei_new(
    ExifMem *mem  // 内存分配器
);
```

**功能**: 创建新的华为 Maker Note 数据对象

**返回值**: 新的 Huawei MnoteData 对象指针

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 78 行)

**OH 需求**: 摄像头驱动生成华为 Maker Note 时使用。

### 3. 清理与销毁

#### exif_mnote_data_huawei_clear()

```c
void exif_mnote_data_huawei_clear(
    ExifMnoteDataHuawei *n  // Huawei 数据对象
);
```

**功能**: 清理华为 Maker Note 数据对象（递归清理子树）

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 73 行)

#### mnote_huawei_free_entry_count()

```c
void mnote_huawei_free_entry_count(
    MnoteHuaweiEntryCount *ec  // 条目计数对象
);
```

**功能**: 释放华为条目计数对象

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 71 行)

### 4. 条目管理

#### exif_mnote_data_huawei_get_entry_by_tag()

```c
MnoteHuaweiEntry* exif_mnote_data_huawei_get_entry_by_tag(
    ExifMnoteDataHuawei *n,  // Huawei 数据对象
    const MnoteHuaweiTag tag  // 标签枚举值
);
```

**功能**: 按标签枚举查询华为条目

**返回值**: 匹配的华为条目指针，未找到返回 NULL

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 76 行)

**OH 需求**: 应用层查询华为 Maker Note 中的特定标签（如场景、人脸）。

#### exif_mnote_data_huawei_get_entry_by_index()

```c
MnoteHuaweiEntry* exif_mnote_data_huawei_get_entry_by_index(
    ExifMnoteDataHuawei *n,  // Huawei 数据对象
    const int dest_idx          // 条目索引
);
```

**功能**: 按索引查询华为条目

**返回值**: 指定索引的华为条目指针，索引越界返回 NULL

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 77 行)

#### exif_mnote_data_add_entry()

```c
int exif_mnote_data_add_entry(
    ExifMnoteData *ne,    // MnoteData 基类
    MnoteHuaweiEntry *e      // 要添加的华为条目
);
```

**功能**: 添加华为条目到 Maker Note

**返回值**: 成功返回 0，失败返回非 0

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 74 行)

#### exif_mnote_data_remove_entry()

```c
void exif_mnote_data_remove_entry(
    ExifMnoteData *ne,    // MnoteData 基类
    MnoteHuaweiEntry *e      // 要移除的华为条目
);
```

**功能**: 从 Maker Note 中移除华为条目

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 75 行)

### 5. 条目计数

#### mnote_huawei_get_entry_count()

```c
void mnote_huawei_get_entry_count(
    const ExifMnoteDataHuawei *n,        // Huawei 数据对象
    MnoteHuaweiEntryCount **entry_count  // 输出：条目计数对象
);
```

**功能**: 获取华为 Maker Note 的条目计数（包括子树）

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 72 行)

### 6. 工具函数

#### is_huawei_md()

```c
int is_huawei_md(
    ExifMnoteData* md  // MnoteData 基类
);
```

**功能**: 检查 MnoteData 对象是否为华为类型

**返回值**: 是华为类型返回非 0，否则返回 0

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 68 行)

#### print_huawei_md()

```c
void print_huawei_md(
    const ExifMnoteDataHuawei* n  // Huawei 数据对象
);
```

**功能**: 打印华为 Maker Note 数据（调试用）

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 69 行)

#### exif_mnote_data_huawei_get_byte_order()

```c
ExifByteOrder exif_mnote_data_huawei_get_byte_order(
    ExifMnoteData *ne  // MnoteData 基类
);
```

**功能**: 获取华为 Maker Note 的字节序

**返回值**: EXIF_BYTE_ORDER_MOTOROLA (大端) 或 EXIF_BYTE_ORDER_INTEL (小端)

**文件**: `libexif/huawei/exif-mnote-data-huawei.h` (第 70 行)

## 华为标签 API

### 1. 标签查询

#### mnote_huawei_tag_get_name()

```c
MnoteHuaweiTag mnote_huawei_tag_from_name(
    const char *name  // 标签名称字符串
);
```

**功能**: 从标签名称字符串解析为标签枚举

**返回值**: 匹配的 Huawei 标签枚举，未找到返回 MNOTE_HUAWEI_INFO

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 91 行)

#### mnote_huawei_tag_get_name()

```c
const char *mnote_huawei_tag_get_name(
    MnoteHuaweiTag  // 标签枚举
);
```

**功能**: 获取标签的名称字符串

**返回值**: 标签名称（如 "XMAGE_MODE"），未找到返回 NULL

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 92 行)

#### mnote_huawei_tag_get_title()

```c
const char *mnote_huawei_tag_get_title(
    MnoteHuaweiTag  // 标签枚举
);
```

**功能**: 获取标签的标题（用户友好名称）

**返回值**: 标签标题（中文或英文），未找到返回 NULL

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 93 行)

**示例**:
- MNOTE_HUAWEI_CAPTURE_MODE → "Capture Mode" / "捕获模式"
- MNOTE_HUAWEI_XMAGE_MODE → "XMAGE Mode" / "XMAGE 模式"

#### mnote_huawei_tag_get_description()

```c
const char *mnote_huawei_tag_get_description(
    MnoteHuaweiTag  // 标签枚举
);
```

**功能**: 获取标签的详细描述

**返回值**: 标签描述（中文或英文），未找到返回 NULL

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 94 行)

**示例**:
- MNOTE_HUAWEI_SCENE_FOOD_CONF → "Confidence that the image is food" / "图像为美食场景的置信度"

### 2. 标签类型查询

#### mnote_huawei_tag_type()

```c
MnoteHuaweiTagType mnote_huawei_tag_type(
    MnoteHuaweiTag  // 标签枚举
);
```

**功能**: 获取标签的类型（普通标签/IFD 子树/结构化数据）

**返回值**:
- MNOTE_HUAWEI_TAG_TYPE_TAG: 普通标签
- MNOTE_HUAWEI_TAG_TYPE_IFD: IFD 子树（如场景、人脸）
- MNOTE_HUAWEI_TAG_TYPE_STRUCT: 结构化数据

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 95 行)

#### get_tag_owner_tag()

```c
MnoteHuaweiTag get_tag_owner_tag(
    MnoteHuaweiTag  // 标签枚举
);
```

**功能**: 获取子树标签的父级标签

**返回值**: 父级标签，如果没有则返回标签本身

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 96 行)

**示例**:
- MNOTE_HUAWEI_SCENE_FOOD_CONF → MNOTE_HUAWEI_SCENE_INFO (0x0000)
- MNOTE_HUAWEI_FACE_COUNT → MNOTE_HUAWEI_FACE_INFO (0x0100)

#### is_ifd_tag()

```c
int is_ifd_tag(
    MnoteHuaweiTag  // 标签枚举
);
```

**功能**: 判断标签是否为 IFD 子树

**返回值**: 是 IFD 子树返回非 0，否则返回 0

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 97 行)

## 华为标签枚举

### 完整标签列表

参见 **[02_Patches.md](02_Patches.md)** 的"华为标签分类与功能"章节。

### 关键标签枚举

```c
enum _MnoteHuaweiTag {
    MNOTE_HUAWEI_INFO                  = 0xFFFF,

    // 拍摄信息
    MNOTE_HUAWEI_CAPTURE_MODE        = 0x0200,
    MNOTE_HUAWEI_BURST_NUMBER        = 0x0201,
    MNOTE_HUAWEI_FRONT_CAMERA        = 0x0202,
    MNOTE_HUAWEI_ROLL_ANGLE          = 0x0203,
    MNOTE_HUAWEI_PITCH_ANGLE         = 0x0204,
    MNOTE_HUAWEI_PHYSICAL_APERTURE   = 0x0205,

    // XMAGE 功能
    MNOTE_HUAWEI_IS_XMAGE_SUPPORTED  = 0x0206,
    MNOTE_HUAWEI_XMAGE_MODE          = 0x0207,
    MNOTE_HUAWEI_XMAGE_LEFT          = 0x0208,
    MNOTE_HUAWEI_XMAGE_TOP           = 0x0209,
    MNOTE_HUAWEI_XMAGE_RIGHT         = 0x020a,
    MNOTE_HUAWEI_XMAGE_BOTTOM        = 0x020b,

    // 云增强与 AI
    MNOTE_HUAWEI_CLOUD_ENHANCEMENT_MODE = 0x020c,
    MNOTE_HUAWEI_WIND_SNAPSHOT_MODE    = 0x020e,
    MNOTE_MOVING_PHOTO_VERSION         = 0x020f,
    MNOTE_MICRO_VIDEO_PRESENTATION_TIMESTAMP_US = 0x0210,
    MNOTE_MOVING_PHOTO_ID             = 0x0211,
    MNOTE_HUAWEI_AI_EDIT              = 0x0212,
    MNOTE_HUAWEI_ANNOTATION_EDIT       = 0x0214,

    // XTStyle 滤镜
    MNOTE_HUAWEI_XTSTYLE_TEMPLATE_NAME        = 0x0300,
    MNOTE_HUAWEI_XTSTYLE_CUSTOM_LIGHT_SHADOW = 0x0301,
    MNOTE_HUAWEI_XTSTYLE_CUSTOM_SATURATION  = 0x0302,
    MNOTE_HUAWEI_XTSTYLE_CUSTOM_HUE         = 0x0303,
    MNOTE_HUAWEI_XTSTYLE_EXPOSUREPARAM_PARAM = 0x0304,
    MNOTE_HUAWEI_XTSTYLE_ALGO_VERSION       = 0x0307,
    MNOTE_HUAWEI_XTSTYLE_ALGO_VIDEO_ENABLE  = 0x0308,
    MNOTE_HUAWEI_XTSTYLE_VIGNETTING         = 0x0309,
    MNOTE_HUAWEI_XTSTYLE_NOISE               = 0x0310,

    // 场景信息子树
    MNOTE_HUAWEI_SCENE_INFO    = 0x0000,  // 子树根
    MNOTE_HUAWEI_SCENE_VERSION  = 0x0001,
    MNOTE_HUAWEI_SCENE_FOOD_CONF     = 0x0002,
    MNOTE_HUAWEI_SCENE_STAGE_CONF    = 0x0003,
    MNOTE_HUAWEI_SCENE_BLUESKY_CONF  = 0x0004,
    // ... (更多场景）

    // 人脸信息子树
    MNOTE_HUAWEI_FACE_INFO    = 0x0100,  // 子树根
    MNOTE_HUAWEI_FACE_VERSION = 0x0101,
    MNOTE_HUAWEI_FACE_COUNT   = 0x0102,
    MNOTE_HUAWEI_FACE_CONF    = 0x0103,
    MNOTE_HUAWEI_FACE_SMILE_SCORE = 0x0104,
    MNOTE_HUAWEI_FACE_RECT  = 0x0105,
    // ... (更多人脸属性）
};
typedef enum _MnoteHuaweiTag MnoteHuaweiTag;
```

**文件**: `libexif/huawei/mnote-huawei-tag.h` (第 25-80 行)

## 上游 API 兼容性

### 完全兼容的 API

libexif 的上游核心 API 在 OH 中**完全兼容**，包括：

| API 类别 | 示例函数 | 兼容性 |
|---------|-----------|--------|
| **数据加载** | `exif_data_load_data()`, `exif_loader_new()` | ✅ 完全兼容 |
| **数据保存** | `exif_data_save_data()` | ✅ 完全兼容 |
| **标签查询** | `exif_tag_get_name()`, `exif_tag_get_title()` | ✅ 完全兼容 |
| **条目查询** | `exif_entry_get_value()`, `exif_content_get_entry()` | ✅ 完全兼容 |
| **内存管理** | `exif_data_new()`, `exif_data_free()` | ✅ 完全兼容 |
| **字节序** | `exif_data_get_byte_order()` | ✅ 完全兼容 |

### 厂商 Maker Note API

除了华为，以下厂商的 Maker Note API 也完全保留：

| 厂商 | API 前缀 | 兼容性 |
|------|-----------|--------|
| Apple | `exif_mnote_data_apple_*` | ✅ 完全兼容 |
| Canon | `exif_mnote_data_canon_*` | ⚠️ 上游 0.6.24 禁用 |
| Fuji | `exif_mnote_data_fuji_*` | ✅ 完全兼容 |
| Olympus | `exif_mnote_data_olympus_*` | ✅ 完全兼容 |
| Pentax | `exif_mnote_data_pentax_*` | ✅ 完全兼容 |

## 行为变更

### 华为 Maker Note 识别优先级

**上游行为**: 按厂商 Maker Note 模块注册顺序识别

**OH 行为**: 华为 Maker Note 优先识别（Huawei 模块最后注册，但头部检查优先）

**影响**: 混合使用多个厂商相机的场景，华为相机的 Maker Note 优先被识别。

### 错误处理

**上游行为**: 未知 Maker Note 类型返回 NULL

**OH 行为**: 华为 Maker Note 识别失败返回 0，但继续尝试其他厂商

**代码证据**:
```c
// exif-mnote-data-huawei.c
int exif_mnote_data_huawei_identify(...) {
    // 检查华为头部
    if (memcmp(data, HUAWEI_HEADER, sizeof(HUAWEI_HEADER)) != 0) {
        return 0;  // ← 不是华为 Maker Note
    }
    return 1;  // ← 是华为 Maker Note
}
```

## 废弃/禁用的功能

### Apple Maker Note（上游 0.6.24 禁用）

**上游 NEWS (0.6.24)**:
```
* Disabled Apple Makernote support, as its not complete
```

**OH 行为**: 仍包含 Apple Maker Note 实现

**配置**: BUILD.gn 仍然包含 `apple/` 目录的源文件

**影响**:
- ⚠️ 可能存在未完成的 Apple Maker Note 功能
- ⚠️ 上游已禁用，OH 仍在使用

**建议**: TODO(需确认) 是否需要在 OH 中也禁用 Apple Maker Note

### 其他禁用功能

libexif 在 OH 中没有禁用其他上游功能。

## API 使用示例

### 示例 1：识别华为 Maker Note

```c
#include <libexif/exif-data.h>
#include <libexif/exif-mnote-data.h>
#include <libexif/huawei/exif-mnote-data-huawei.h>

void check_huawei_makernote(const char *jpeg_file) {
    ExifData *ed = exif_data_new_from_file(jpeg_file);

    // 获取 Maker Note 条目
    ExifEntry *e = exif_content_get_entry(ed->ifd[0],
                                         EXIF_TAG_MAKER_NOTE);

    if (e) {
        // 识别是否为华为 Maker Note
        int is_huawei = exif_mnote_data_huawei_identify(ed, e);

        if (is_huawei) {
            printf("This is a Huawei camera photo\n");

            // 加载华为 Maker Note
            exif_mnote_data_load(e->data, e->size, ed);

            // 获取 XMAGE 模式
            ExifMnoteData *md = ed->priv;
            MnoteHuaweiEntry *entry =
                exif_mnote_data_huawei_get_entry_by_tag(md, MNOTE_HUAWEI_XMAGE_MODE);

            if (entry) {
                printf("XMAGE Mode: %s\n",
                       mnote_huawei_tag_get_title(entry->tag));
            }
        }
    }

    exif_data_unref(ed);
}
```

### 示例 2：生成华为 Maker Note

```c
#include <libexif/exif-data.h>
#include <libexif/huawei/exif-mnote-data-huawei.h>

void create_huawei_photo_with_xmage(const char *output_file) {
    ExifData *ed = exif_data_new();

    // 设置标准 EXIF 标签
    exif_data_set_option(ed, EXIF_DATA_OPTION_DONT_CHANGE_MAKER_NOTE, 1);

    // 创建华为 Maker Note
    ExifMnoteData *huawei_md =
        (ExifMnoteData*)exif_mnote_data_huawei_new(ed->mem);

    // 添加 XMAGE 模式标签
    MnoteHuaweiEntry *xmage_entry = /* ... 创建条目 ... */;
    xmage_entry->tag = MNOTE_HUAWEI_XMAGE_MODE;
    /* 设置 XMAGE 模式值 ... */

    // 添加到华为 Maker Note
    exif_mnote_data_add_entry((ExifMnoteData*)huawei_md, xmage_entry);

    // 关联到 EXIF 数据
    ed->priv = huawei_md;

    // 保存为 JPEG
    // ... (与 libjpeg-turbo 配合使用）

    exif_data_unref(ed);
}
```

### 示例 3：解析华为场景识别

```c
#include <libexif/exif-data.h>
#include <libexif/huawei/exif-mnote-data-huawei.h>
#include <libexif/huawei/mnote-huawei-tag.h>

void parse_huawei_scene_info(const char *jpeg_file) {
    ExifData *ed = exif_data_new_from_file(jpeg_file);
    ExifMnoteDataHuawei *huawei = (ExifMnoteDataHuawei*)ed->priv;

    if (!is_huawei_md((ExifMnoteData*)huawei)) {
        printf("Not a Huawei photo\n");
        return;
    }

    // 遍历华为标签，查找场景信息
    for (unsigned int i = 0; i < huawei->count; i++) {
        MnoteHuaweiEntry *entry = &huawei->entries[i];

        if (entry->tag >= MNOTE_HUAWEI_SCENE_FOOD_CONF &&
            entry->tag <= MNOTE_HUAWEI_SCENE_TEXT_CONF) {

            // 获取场景名称和置信度
            const char *scene_name = mnote_huawei_tag_get_name(entry->tag);
            const char *scene_title = mnote_huawei_tag_get_title(entry->tag);

            printf("Scene: %s (%s), Confidence: %d\n",
                   scene_name, scene_title, entry->value);
        }
    }

    exif_data_unref(ed);
}
```

## 总结

OpenHarmony 对 libexif 的 API 扩展主要通过华为 Maker Note 模块实现：

### API 扩展特点

✅ **独立命名空间**: 华为 API 使用 `huawei` 前缀，不影响上游 API
✅ **完全兼容**: 保留所有上游 API 和厂商 Maker Note API
✅ **新增 40+ 个华为标签**: 涵盖 XMAGE、XTStyle、场景、人脸
✅ **递归支持**: 支持 IFD 子树（场景、人脸信息）
✅ **中文支持**: 标签标题和描述支持中文

### 关键 API 类别

| 类别 | 函数数量 | 示例 |
|------|----------|------|
| **识别与创建** | 2 | `exif_mnote_data_huawei_identify()`, `exif_mnote_data_huawei_new()` |
| **清理与销毁** | 2 | `exif_mnote_data_huawei_clear()`, `mnote_huawei_free_entry_count()` |
| **条目管理** | 4 | `exif_mnote_data_huawei_get_entry_by_tag()`, `exif_mnote_data_add_entry()` |
| **条目计数** | 1 | `mnote_huawei_get_entry_count()` |
| **工具函数** | 3 | `is_huawei_md()`, `print_huawei_md()`, `exif_mnote_data_huawei_get_byte_order()` |
| **标签查询** | 7 | `mnote_huawei_tag_get_name()`, `mnote_huawei_tag_get_title()` 等 |

### 上游兼容性

✅ **核心 API**: 完全兼容
✅ **厂商 API**: 保留 Apple、Canon、Fuji、Olympus、Pentax
⚠️ **Apple**: 上游已禁用，OH 仍包含（需确认）

### 使用建议

1. **识别华为照片**: 使用 `exif_mnote_data_huawei_identify()`
2. **查询华为标签**: 使用 `exif_mnote_data_huawei_get_entry_by_tag()`
3. **获取标签描述**: 使用 `mnote_huawei_tag_get_title()` 和 `mnote_huawei_tag_get_description()`
4. **生成华为 Maker Note**: 使用 `exif_mnote_data_huawei_new()` 和 `exif_mnote_data_add_entry()`

## 证据文件

- **华为数据结构**: `libexif/huawei/exif-mnote-data-huawei.h`
- **华为标签定义**: `libexif/huawei/mnote-huawei-tag.h`
- **华为条目类型**: `libexif/huawei/mnote-huawei-data-type.h`

## 参考资料

- **libexif API 文档**: https://libexif.sourceforge.io/api/
- **OH 华为 Maker Note**: `third_party/libexif/libexif/huawei/`
- **华为 XMAGE 介绍**: 华为官网
