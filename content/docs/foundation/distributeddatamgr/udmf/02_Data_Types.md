# 数据类型体系

## 类型系统概述

UDMF 数据类型体系基于 UTD（Uniform Type Descriptor，统一类型描述符）构建，是实现跨应用、跨设备数据流转的基础抽象。UTD 系统定义了约 200 种预统一的数据类型，覆盖文本、图像、音频、视频、文件、应用程序等多种数据格式，为上层应用提供统一的类型识别和处理能力。

从类型来源角度，UDMF 将数据类型分为三大类别。第一类是基础数据类型（Base Types），如 File、Text 等，这些类型能够进行跨应用、跨设备以及跨平台流转，是 UDMF 类型系统的核心。第二类是系统定义类型（System Defined Type，SDT），这类类型与具体的平台或操作系统绑定，例如 Form（UI 卡片信息）、AppItem（应用描述信息）、PixelMap（缩略图格式）等。第三类是应用自定义类型（App Defined Type，ADT），单个应用可以自定义数据结构，该类型仅在应用生态内部实现跨平台与跨设备流转。

根据源码证据（`interfaces/ndk/data/udmf_meta.h`），所有类型通过宏定义声明，命名规范为 `UDMF_META_*` 格式。类型系统支持层次关系查询，允许判断一个类型是否属于另一个类型的子集，这在数据接收和类型匹配场景中尤为重要。

## 预定义类型清单

### 通用类型（General Types）

通用类型覆盖基本数据类型，属于类型层次结构的根节点或中间节点：

```
文本类型（general.text 及其子类）：
├── general.plain-text        # 纯文本
├── general.html              # HTML 格式
├── general.hyperlink         # 超链接
├── general.source-code       # 源代码
├── general.script            # 脚本文件
├── general.markdown         # Markdown 格式
└── [其他文本变体]

图像类型（general.image 及其子类）：
├── general.jpeg             # JPEG 图像
├── general.png              # PNG 图像
├── general.raw-image        # 原始图像
├── general.tiff             # TIFF 图像
├── com.microsoft.bmp        # BMP 图像
└── [其他图像格式]

视频类型（general.video 及其子类）：
├── general.avi              # AVI 视频
├── general.mpeg            # MPEG 视频
├── general.mpeg-4          # MP4 视频
├── general.3gpp            # 3GPP 视频
└── [其他视频格式]

音频类型（general.audio 及其子类）：
├── general.aac              # AAC 音频
├── general.aiff            # AIFF 音频
├── general.flac            # FLAC 音频
├── general.mp3             # MP3 音频
├── general.ogg            # OGG 音频
└── [其他音频格式]

文件类型（general.file 及其子类）：
├── general.directory       # 目录
├── general.folder          # 文件夹
├── general.symlink         # 符号链接
├── general.archive         # 归档文件
├── general.zip-archive     # ZIP 归档
└── [其他文件类型]
```

### OpenHarmony 系统类型

OpenHarmony 系统定义了专有类型，用于系统级数据交换：

```
openharmony.form             # UI 卡片信息
openharmony.app-item         # 应用描述信息
openharmony.pixel-map       # 缩略图格式
openharmony.atomic-service  # 元服务信息
openharmony.package         # 包信息
openharmony.hap            # HAP 安装包
openharmony.hdoc            # HDoc 文档
openharmony.hinote         # HiNote 笔记
openharmony.styled-string  # 样式字符串
openharmony.want           # Want 对象
```

### 办公文档类型

UDMF 支持主流办公文档格式：

```
Microsoft Office：
├── com.microsoft.word.doc      # Word 文档
├── com.microsoft.excel.xls     # Excel 电子表格
├── com.microsoft.powerpoint.ppt # PowerPoint 演示文稿
└── [其他 Office 格式]

OpenDocument：
├── org.oasis.opendocument       # OpenDocument 根类型
├── org.oasis.opendocument.text  # ODT 文档
├── org.oasis.opendocument.spreadsheet  # ODS 电子表格
└── org.oasis.opendocument.presentation   # ODP 演示文稿

PDF 和电子书：
├── com.adobe.pdf               # PDF 文档
├── general.ebook               # 电子书根类型
├── general.epub                # EPUB 电子书
└── [其他电子书格式]
```

### 平台特定类型

UDMF 兼容多种平台的文件格式：

```
Microsoft 格式：
├── com.microsoft.windows-media-*  # Windows Media 系列
├── com.microsoft.bmp              # BMP 图像
├── com.microsoft.word.doc         # Word 文档
└── [其他微软格式]

Adobe 格式：
├── com.adobe.photoshop-image     # Photoshop 图像
├── com.adobe.illustrator.ai-image # Illustrator 图像
├── com.adobe.pdf                  # PDF 文档
└── com.adobe.postscript          # PostScript 文档

Amazon 格式：
├── com.amazon.azw                # Amazon Kindle AZW
├── com.amazon.azw3              # AZW3 格式
├── com.amazon.kfx               # KFX 格式
└── com.amazon.mobi              # MobiPocket 格式
```

## 类型层次关系

UTD 类型系统支持类型间的层次关系查询，这是实现类型匹配和自动转换的基础。

### 层次查询 API

根据 `interfaces/innerkits/data/type_descriptor.h:36-38`，TypeDescriptor 类提供三种层次查询方法：

```cpp
// 判断当前类型是否属于目标类型
Status API_EXPORT BelongsTo(const std::string &typeId, bool &checkResult);

// 判断当前类型是否是目标类型的下级类型
Status API_EXPORT IsLowerLevelType(const std::string &typeId, bool &checkResult);

// 判断当前类型是否是目标类型的高级类型
Status API_EXPORT IsHigherLevelType(const std::string &typeId, bool &checkResult);
```

### 层次关系示例

以图像类型为例，说明类型间的层次关系：

```
general.image
├── general.jpeg
├── general.png
├── general.raw-image
├── general.tiff
├── com.microsoft.bmp
└── [其他具体图像格式]

general.video
├── general.avi
├── general.mpeg
├── general.mpeg-4
└── [其他具体视频格式]

查询语义：
- "general.jpeg" BelongsTo "general.image" → true
- "general.image" IsLowerLevelType "general.media" → true
- "general.image" IsHigherLevelType "general.jpeg" → true
```

NDK 层提供对应 API（`interfaces/ndk/data/utd.h:179-203`）：

```c
// 判断类型归属关系
bool OH_Utd_BelongsTo(const char* srcTypeId, const char* destTypeId);

// 判断是否下级类型
bool OH_Utd_IsLower(const char* srcTypeId, const char* destTypeId);

// 判断是否上级类型
bool OH_Utd_IsHigher(const char* srcTypeId, const char* destTypeId);
```

## 类型元数据

每个 UTD 类型携带丰富的元数据，用于描述类型的特征和用途。

### TypeDescriptor 结构

根据 `interfaces/innerkits/data/type_descriptor.h:28-68`，TypeDescriptor 类定义如下：

```cpp
class TypeDescriptor {
public:
    // 构造函数
    TypeDescriptor(const std::string &typeId,
                  const std::vector<std::string> &belongingToTypes,
                  const std::vector<std::string> &filenameExtensions,
                  const std::vector<std::string> &mimeTypes,
                  const std::string &description,
                  const std::string &referenceURL,
                  const std::string &iconFile);

    // 类型标识符
    const std::string& GetTypeId() const;

    // 归属类型列表
    std::vector<std::string> GetBelongingToTypes();

    // 文件扩展名列表
    std::vector<std::string> GetFilenameExtensions();

    // MIME 类型列表
    std::vector<std::string> GetMimeTypes();

    // 类型描述
    std::string GetDescription();

    // 参考链接
    std::string GetReferenceURL();

    // 图标文件
    std::string GetIconFile();
};
```

### 元数据示例

以 PNG 图像类型为例，其元数据可能包含：

```
类型标识符：general.png
归属类型：["general.image", "general.media"]
文件扩展名：[".png", ".PNG"]
MIME 类型：["image/png"]
描述：Portable Network Graphics (PNG) 图像格式
参考链接：https://www.w3.org/TR/PNG/
图标文件：/data/utd/icons/png.png
```

### 元数据获取 API

NDK 层提供以下 API 获取类型元数据（`interfaces/ndk/data/utd.h`）：

```c
// 获取类型标识符
const char* OH_Utd_GetTypeId(OH_Utd* pThis);

// 获取描述
const char* OH_Utd_GetDescription(OH_Utd* pThis);

// 获取参考 URL
const char* OH_Utd_GetReferenceUrl(OH_Utd* pThis);

// 获取图标文件路径
const char* OH_Utd_GetIconFile(OH_Utd* pThis);

// 获取归属类型列表
const char** OH_Utd_GetBelongingToTypes(OH_Utd* pThis, unsigned int* count);

// 获取文件扩展名列表
const char** OH_Utd_GetFilenameExtensions(OH_Utd* pThis, unsigned int* count);

// 获取 MIME 类型列表
const char** OH_Utd_GetMimeTypes(OH_Utd* pThis, unsigned int* count);
```

## 类型识别

UDMF 支持通过多种方式识别数据类型。

### 通过文件扩展名识别

```c
// 根据文件扩展名获取对应类型
const char** OH_Utd_GetTypesByFilenameExtension(const char* extension,
                                                 unsigned int* count);

// 示例：".png" → ["general.png", "general.image", "general.media"]
```

### 通过 MIME 类型识别

```c
// 根据 MIME 类型获取对应类型
const char** OH_Utd_GetTypesByMimeType(const char* mimeType,
                                         unsigned int* count);

// 示例："image/png" → ["general.png", "general.image", "general.media"]
```

### 类型比较

```c
// 判断两个 UTD 实例是否相等
bool OH_Utd_Equals(OH_Utd* utd1, OH_Utd* utd2);
```

## 自定义类型

除了预定义类型，UDMF 支持应用注册自定义类型（ADT）。

### 自定义类型注册

根据 `interfaces/innerkits/client/utd_client.h`，UtdClient 提供类型注册接口：

```cpp
// 注册自定义类型描述符
Status API_EXPORT RegisterTypeDescriptors(const std::vector<TypeDescriptorCfg> &descriptors);

// 注销自定义类型
Status API_EXPORT UnregisterTypeDescriptors(const std::vector<std::string> &typeIds);

// 安装自定义 UTD 配置
void API_EXPORT InstallCustomUtds(const std::string &bundleName,
                                   const std::string &jsonStr, int32_t user);

// 卸载自定义 UTD
void API_EXPORT UninstallCustomUtds(const std::string &bundleName, int32_t user);
```

### 自定义类型约束

自定义类型需要遵循以下约束：

**命名规范**：建议使用反向域名格式，如 `com.example.myCustomType`，避免与系统类型冲突。

**元数据完整性**：自定义类型应提供完整的元数据，包括文件扩展名、MIME 类型、归属类型等。

**生命周期管理**：自定义类型与应用绑定，应用卸载时自动清理关联的自定义类型。

## UnifiedData 结构

UnifiedData 是 UDMF 的核心数据容器，封装了数据及其元信息。

### 数据结构定义

根据 `interfaces/innerkits/data/unified_data.h:24-78`，UnifiedData 类定义如下：

```cpp
class UnifiedData {
public:
    // 构造函数
    API_EXPORT UnifiedData();
    explicit API_EXPORT UnifiedData(std::shared_ptr<UnifiedDataProperties> properties);

    // 获取数据大小
    int64_t API_EXPORT GetSize();

    // 获取组 ID
    std::string GetGroupId() const;

    // 获取运行时信息
    std::shared_ptr<Runtime> API_EXPORT GetRuntime() const;
    void API_EXPORT SetRuntime(Runtime &runtime);

    // 记录管理
    void API_EXPORT AddRecord(const std::shared_ptr<UnifiedRecord> &record);
    void API_EXPORT AddRecords(const std::vector<std::shared_ptr<UnifiedRecord>> &records);
    std::shared_ptr<UnifiedRecord> API_EXPORT GetRecordAt(std::size_t index) const;
    void API_EXPORT SetRecords(std::vector<std::shared_ptr<UnifiedRecord>> records);
    std::vector<std::shared_ptr<UnifiedRecord>> API_EXPORT GetRecords() const;

    // 类型查询
    std::vector<std::string> API_EXPORT GetTypesLabels() const;
    bool API_EXPORT HasType(const std::string &type) const;
    bool API_EXPORT HasHigherFileType(const std::string &type) const;
    std::vector<std::string> API_EXPORT GetEntriesTypes() const;
    bool API_EXPORT HasTypeInEntries(const std::string &type) const;

    // 数据校验
    bool API_EXPORT IsEmpty() const;
    bool API_EXPORT IsValid();
    bool API_EXPORT IsComplete();

    // 属性管理
    void API_EXPORT SetProperties(std::shared_ptr<UnifiedDataProperties> properties);
    std::shared_ptr<UnifiedDataProperties> API_EXPORT GetProperties() const;

    // 最大数据大小限制
    static constexpr int64_t MAX_DATA_SIZE = 200 * 1024 * 1024;  // 200MB
};
```

### UnifiedRecord 结构

UnifiedRecord 是 UnifiedData 中的单条记录，包含具体的数据内容：

```cpp
class UnifiedRecord {
public:
    // 获取类型
    UDType API_EXPORT GetType() const;
    std::vector<std::string> API_EXPORT GetTypes() const;

    // 唯一标识
    std::string API_EXPORT GetUid() const;
    void API_EXPORT SetUid(const std::string &id);

    // 值管理
    ValueType API_EXPORT GetValue();
    void SetValue(const ValueType &value);

    // UTD 管理
    void API_EXPORT SetUtdId(const std::string &utdId);
    std::string API_EXPORT GetUtdId() const;
    void API_EXPORT SetUtdId2(const std::string &utdId);
    std::string API_EXPORT GetUtdId2() const;

    // Entry 管理（支持懒加载）
    void API_EXPORT AddEntry(const std::string &utdId, ValueType &&value);
    ValueType API_EXPORT GetEntry(const std::string &utdId);
    std::shared_ptr<std::map<std::string, ValueType>> API_EXPORT GetEntries();

    // 懒加载 Getter
    void API_EXPORT SetEntryGetter(const std::vector<std::string> &utdIds,
                                    const std::shared_ptr<EntryGetter> &entryGetter);
};
```

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述与核心能力
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构详解
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK API 参考
- [12_InnerKit_Reference.md](./12_InnerKit_Reference.md)：InnerKit API 参考
- [22_Core_Data_Structures.md](./22_Core_Data_Structures.md)：核心数据结构实现
