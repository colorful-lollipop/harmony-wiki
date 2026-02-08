# NDK 接口规范

## 接口概述

NDK（Native Development Kit）接口是 UDMF 为 Native 应用提供的 C 语言编程接口。与 N-API 相比，NDK 接口具有更高的执行效率和更小的运行时开销，适合对性能有严格要求的底层应用开发。

UDMF NDK 接口位于 `interfaces/ndk/` 目录，编译产物为 `libudmf.so`（安装路径为 `system/lib/ndk/udmf.so`）。接口设计遵循 C 语言规范，不使用 C++ 异常和 RTTI，通过返回值和错误码传递执行状态。

根据 `interfaces/ndk/BUILD.gn` 配置，NDK 库依赖 InnerKit 层（`udmf_client`、`utd_client`），通过封装层实现 C++ 到 C 的桥接。Native 应用通过链接 `libudmf.so` 并包含相应头文件即可使用 UDMF 能力。

## 头文件清单

| 头文件 | 用途 | 主要内容 |
|--------|------|----------|
| `udmf.h` | 主 UDMF API | 数据操作、属性管理 |
| `utd.h` | UTD 类型 API | 类型查询、类型注册 |
| `uds.h` | UDS 数据结构 | PlainText、Hyperlink、HTML 等结构 |
| `udmf_meta.h` | 类型常量 | 200+ 预定义类型标识符 |
| `udmf_err_code.h` | 错误码 | 执行状态与监听状态 |

## 核心数据类型

### Intention（使用意图）

`udmf.h:62-95` 定义了数据的使用意图枚举：

```c
typedef enum Udmf_Intention {
    UDMF_INTENTION_DRAG = 0,           // 拖拽场景
    UDMF_INTENTION_PASTEBOARD,           // 剪贴板场景
    UDMF_INTENTION_DATA_HUB,            // 数据中心场景
    UDMF_INTENTION_SYSTEM_SHARE,         // 系统分享场景
    UDMF_INTENTION_PICKER,              // 选择器场景
    UDMF_INTENTION_MENU,                // 菜单场景
} Udmf_Intention;
```

### ShareOption（共享选项）

`udmf.h:102-115` 定义了数据共享选项：

```c
typedef enum Udmf_ShareOption {
    SHARE_OPTIONS_INVALID = 0,          // 无效选项
    SHARE_OPTIONS_IN_APP,               // 仅应用内可用
    SHARE_OPTIONS_CROSS_APP             // 允许跨应用使用
} Udmf_ShareOption;
```

### FileConflictOptions（文件冲突处理）

`udmf.h:122-131` 定义了文件冲突处理策略：

```c
typedef enum Udmf_FileConflictOptions {
    UDMF_OVERWRITE = 0,                 // 覆盖已存在文件
    UDMF_SKIP = 1                        // 跳过已存在文件
} Udmf_FileConflictOptions;
```

### ProgressIndicator（进度指示器）

`udmf.h:138-147` 定义了进度显示选项：

```c
typedef enum Udmf_ProgressIndicator {
    UDMF_NONE = 0,                      // 不显示进度
    UDMF_DEFAULT = 1                     // 使用系统默认进度
} Udmf_ProgressIndicator;
```

### Visibility（可见性）

`udmf.h:154-164` 定义了数据可见性级别：

```c
typedef enum Udmf_Visibility {
    UDMF_ALL = 0,                        // 任意 Hap/Native 可获取
    UDMF_OWN_PROCESS = 1                 // 仅提供方可获取
} Udmf_Visibility;
```

## 错误码定义

### 执行错误码

`udmf_err_code.h:54-67` 定义了执行错误码：

```c
typedef enum Udmf_ErrCode {
    UDMF_E_OK = 0,                       // 成功
    UDMF_ERR = 20400000,                 // 通用错误基值
    UDMF_E_INVALID_PARAM = (UDMF_ERR + 1) // 参数无效
} Udmf_ErrCode;
```

### 监听状态码

`udmf_err_code.h:74-103` 定义了异步操作监听状态：

```c
typedef enum Udmf_ListenerStatus {
    UDMF_FINISHED,                       // 处理完成
    UDMF_PROCESSING,                     // 处理中
    UDMF_INNER_ERROR,                   // 内部错误
    UDMF_INVALID_PARAMETERS,             // 参数无效
    UDMF_DATA_NOT_FOUND,                // 数据未找到
    UDMF_SYNC_FAILED,                   // 同步失败
    UDMF_COPY_FILE_FAILED               // 文件复制失败
} Udmf_ListenerStatus;
```

## UnifiedData API

### 数据生命周期管理

**创建数据对象**

```c
// 创建 OH_UdmfData 实例
// 返回值：成功返回实例指针，失败返回 nullptr
// @since 12
OH_UdmfData* OH_UdmfData_Create();

// 销毁 OH_UdmfData 实例
// 参数：pThis - OH_UdmfData 实例指针
// @since 12
void OH_UdmfData_Destroy(OH_UdmfData* pThis);
```

**添加记录**

```c
// 向 UnifiedData 添加记录
// 参数：pThis - OH_UdmfData 实例指针
//        record - OH_UdmfRecord 实例指针
// 返回值：UDMF_E_OK 成功，UDMF_E_INVALID_PARAM 参数无效
// @since 12
int OH_UdmfData_AddRecord(OH_UdmfData* pThis, OH_UdmfRecord* record);
```

**类型查询**

```c
// 检查数据是否包含指定类型
// 参数：pThis - OH_UdmfData 实例指针
//        type - 类型标识符字符串
// 返回值：true 包含，false 不包含
// @since 12
bool OH_UdmfData_HasType(OH_UdmfData* pThis, const char* type);

// 获取所有类型列表
// 参数：pThis - OH_UdmfData 实例指针
//        count - 输出参数，返回类型数量
// 返回值：类型字符串数组，失败返回 nullptr
// @since 12
char** OH_UdmfData_GetTypes(OH_UdmfData* pThis, unsigned int* count);
```

**记录查询**

```c
// 获取所有记录
// 参数：pThis - OH_UdmfData 实例指针
//        count - 输出参数，返回记录数量
// 返回值：OH_UdmfRecord 指针数组，失败返回 nullptr
// @since 12
OH_UdmfRecord** OH_UdmfData_GetRecords(OH_UdmfData* pThis, unsigned int* count);

// 获取记录数量
// 参数：data - OH_UdmfData 实例指针
// 返回值：记录数量
// @since 13
int OH_UdmfData_GetRecordCount(OH_UdmfData* data);

// 获取指定索引的记录
// 参数：data - OH_UdmfData 实例指针
//        index - 记录索引（从 0 开始）
// 返回值：OH_UdmfRecord 指针，失败返回 nullptr
// @since 13
OH_UdmfRecord* OH_UdmfData_GetRecord(OH_UdmfData* data, unsigned int index);
```

**数据源检查**

```c
// 检查数据是否来自本地设备
// 参数：data - OH_UdmfData 实例指针
// 返回值：true 来自本地，false 来自远程
// @since 13
bool OH_UdmfData_IsLocal(OH_UdmfData* data);
```

## UnifiedRecord API

### 记录创建与销毁

```c
// 创建 OH_UdmfRecord 实例
// 返回值：成功返回实例指针，失败返回 nullptr
// @since 12
OH_UdmfRecord* OH_UdmfRecord_Create();

// 销毁 OH_UdmfRecord 实例
// 参数：pThis - OH_UdmfRecord 实例指针
// @since 12
void OH_UdmfRecord_Destroy(OH_UdmfRecord* pThis);
```

### 通用条目操作

```c
// 添加通用条目
// 参数：record - OH_UdmfRecord 实例指针
//        typeId - 类型标识符
//        entry - 条目数据
//        count - 数据长度
// 返回值：UDMF_E_OK 成功，UDMF_E_INVALID_PARAM 参数无效
// @since 12
int OH_UdmfRecord_AddGeneralEntry(OH_UdmfRecord* record, const char* typeId,
                                   const unsigned char* entry, unsigned int count);

// 获取通用条目
// 参数：pThis - OH_UdmfRecord 实例指针
//        typeId - 类型标识符
//        entry - 输出参数，条目数据指针
//        count - 输出参数，数据长度
// 返回值：UDMF_E_OK 成功，UDMF_E_INVALID_PARAM 参数无效
// @since 12
int OH_UdmfRecord_GetGeneralEntry(OH_UdmfRecord* pThis, const char* typeId,
                                   unsigned char** entry, unsigned int* count);

// 获取记录的所有类型
// 参数：pThis - OH_UdmfRecord 实例指针
//        count - 输出参数，类型数量
// 返回值：类型字符串数组，失败返回 nullptr
// @since 12
char** OH_UdmfRecord_GetTypes(OH_UdmfRecord* pThis, unsigned int* count);
```

### 结构化数据操作

UDMF 提供多种预定义数据结构，以下是主要类型的操作 API：

**PlainText（纯文本）**

```c
// 创建 PlainText 实例
OH_UdsPlainText* OH_UdsPlainText_Create();
void OH_UdsPlainText_Destroy(OH_UdsPlainText* pThis);

// 获取属性
const char* OH_UdsPlainText_GetType(OH_UdsPlainText* pThis);
const char* OH_UdsPlainText_GetContent(OH_UdsPlainText* pThis);
const char* OH_UdsPlainText_GetAbstract(OH_UdsPlainText* pThis);

// 设置属性
int OH_UdsPlainText_SetContent(OH_UdsPlainText* pThis, const char* content);
int OH_UdsPlainText_SetAbstract(OH_UdsPlainText* pThis, const char* abstract);

// 添加到记录
int OH_UdmfRecord_AddPlainText(OH_UdmfRecord* pThis, OH_UdsPlainText* plainText);
int OH_UdmfRecord_GetPlainText(OH_UdmfRecord* pThis, OH_UdsPlainText* plainText);
```

**Hyperlink（超链接）**

```c
// 创建 Hyperlink 实例
OH_UdsHyperlink* OH_UdsHyperlink_Create();
void OH_UdsHyperlink_Destroy(OH_UdsHyperlink* pThis);

// 获取属性
const char* OH_UdsHyperlink_GetUrl(OH_UdsHyperlink* pThis);
const char* OH_UdsHyperlink_GetDescription(OH_UdsHyperlink* pThis);

// 设置属性
int OH_UdsHyperlink_SetUrl(OH_UdsHyperlink* pThis, const char* url);
int OH_UdsHyperlink_SetDescription(OH_UdsHyperlink* pThis, const char* description);

// 记录操作
int OH_UdmfRecord_AddHyperlink(OH_UdmfRecord* pThis, OH_UdsHyperlink* hyperlink);
int OH_UdmfRecord_GetHyperlink(OH_UdmfRecord* pThis, OH_UdsHyperlink* hyperlink);
```

**HTML 内容**

```c
// 创建 HTML 实例
OH_UdsHtml* OH_UdsHtml_Create();
void OH_UdsHtml_Destroy(OH_UdsHtml* pThis);

// 获取属性
const char* OH_UdsHtml_GetContent(OH_UdsHtml* pThis);
const char* OH_UdsHtml_GetPlainContent(OH_UdsHtml* pThis);

// 设置属性
int OH_UdsHtml_SetContent(OH_UdsHtml* pThis, const char* content);
int OH_UdsHtml_SetPlainContent(OH_UdsHtml* pThis, const char* plainContent);

// 记录操作
int OH_UdmfRecord_AddHtml(OH_UdmfRecord* pThis, OH_UdsHtml* html);
int OH_UdmfRecord_GetHtml(OH_UdmfRecord* pThis, OH_UdsHtml* html);
```

**AppItem（应用信息）**

```c
// 创建 AppItem 实例
OH_UdsAppItem* OH_UdsAppItem_Create();
void OH_UdsAppItem_Destroy(OH_UdsAppItem* pThis);

// 获取属性
const char* OH_UdsAppItem_GetId(OH_UdsAppItem* pThis);
const char* OH_UdsAppItem_GetName(OH_UdsAppItem* pThis);
const char* OH_UdsAppItem_GetBundleName(OH_UdsAppItem* pThis);
const char* OH_UdsAppItem_GetAbilityName(OH_UdsAppItem* pThis);

// 设置属性
int OH_UdsAppItem_SetId(OH_UdsAppItem* pThis, const char* appId);
int OH_UdsAppItem_SetName(OH_UdsAppItem* pThis, const char* appName);
int OH_UdsAppItem_SetBundleName(OH_UdsAppItem* pThis, const char* bundleName);
int OH_UdsAppItem_SetAbilityName(OH_UdsAppItem* pThis, const char* abilityName);

// 记录操作
int OH_UdmfRecord_AddAppItem(OH_UdmfRecord* pThis, OH_UdsAppItem* appItem);
int OH_UdmfRecord_GetAppItem(OH_UdmfRecord* pThis, OH_UdsAppItem* appItem);
```

**FileUri（文件 URI）**

```c
// 创建 FileUri 实例
OH_UdsFileUri* OH_UdsFileUri_Create();
void OH_UdsFileUri_Destroy(OH_UdsFileUri* pThis);

// 获取属性
const char* OH_UdsFileUri_GetFileUri(OH_UdsFileUri* pThis);
const char* OH_UdsFileUri_GetFileType(OH_UdsFileUri* pThis);

// 设置属性
int OH_UdsFileUri_SetFileUri(OH_UdsFileUri* pThis, const char* fileUri);
int OH_UdsFileUri_SetFileType(OH_UdsFileUri* pThis, const char* fileType);

// 记录操作
int OH_UdmfRecord_AddFileUri(OH_UdmfRecord* pThis, OH_UdsFileUri* fileUri);
int OH_UdmfRecord_GetFileUri(OH_UdmfRecord* pThis, OH_UdsFileUri* fileUri);
```

**PixelMap（像素图）**

```c
// 创建 PixelMap 实例
OH_UdsPixelMap* OH_UdsPixelMap_Create();
void OH_UdsPixelMap_Destroy(OH_UdsPixelMap* pThis);

// 获取像素图
void OH_UdsPixelMap_GetPixelMap(OH_UdsPixelMap* pThis, OH_PixelmapNative* pixelmapNative);

// 设置像素图
int OH_UdsPixelMap_SetPixelMap(OH_UdsPixelMap* pThis, OH_PixelmapNative* pixelmapNative);

// 记录操作
int OH_UdmfRecord_AddPixelMap(OH_UdmfRecord* pThis, OH_UdsPixelMap* pixelMap);
int OH_UdmfRecord_GetPixelMap(OH_UdmfRecord* pThis, OH_UdsPixelMap* pixelMap);
```

**ArrayBuffer（数组缓冲区）**

```c
// 创建 ArrayBuffer 实例
OH_UdsArrayBuffer* OH_UdsArrayBuffer_Create();
int OH_UdsArrayBuffer_Destroy(OH_UdsArrayBuffer* buffer);

// 设置数据
int OH_UdsArrayBuffer_SetData(OH_UdsArrayBuffer* buffer,
                               unsigned char* data, unsigned int len);

// 获取数据
int OH_UdsArrayBuffer_GetData(OH_UdsArrayBuffer* buffer,
                                unsigned char** data, unsigned int* len);

// 记录操作
int OH_UdmfRecord_AddArrayBuffer(OH_UdmfRecord* record, const char* type,
                                   OH_UdsArrayBuffer* buffer);
int OH_UdmfRecord_GetArrayBuffer(OH_UdmfRecord* record, const char* type,
                                   OH_UdsArrayBuffer* buffer);
```

**ContentForm（内容卡片）**

```c
// 创建 ContentForm 实例
OH_UdsContentForm* OH_UdsContentForm_Create();
void OH_UdsContentForm_Destroy(OH_UdsContentForm* pThis);

// 获取属性
const char* OH_UdsContentForm_GetDescription(OH_UdsContentForm* pThis);
const char* OH_UdsContentForm_GetTitle(OH_UdsContentForm* pThis);
const char* OH_UdsContentForm_GetAppName(OH_UdsContentForm* pThis);
const char* OH_UdsContentForm_GetLinkUri(OH_UdsContentForm* pThis);

// 获取二进制数据
int OH_UdsContentForm_GetThumbData(OH_UdsContentForm* pThis,
                                   unsigned char** thumbData, unsigned int* len);
int OH_UdsContentForm_GetAppIcon(OH_UdsContentForm* pThis,
                                   unsigned char** appIcon, unsigned int* len);

// 设置属性
int OH_UdsContentForm_SetDescription(OH_UdsContentForm* pThis, const char* description);
int OH_UdsContentForm_SetTitle(OH_UdsContentForm* pThis, const char* title);
int OH_UdsContentForm_SetAppName(OH_UdsContentForm* pThis, const char* appName);
int OH_UdsContentForm_SetLinkUri(OH_UdsContentForm* pThis, const char* linkUri);

// 设置二进制数据
int OH_UdsContentForm_SetThumbData(OH_UdsContentForm* pThis,
                                    const unsigned char* thumbData, unsigned int len);
int OH_UdsContentForm_SetAppIcon(OH_UdsContentForm* pThis,
                                   const unsigned char* appIcon, unsigned int len);

// 记录操作
int OH_UdmfRecord_AddContentForm(OH_UdmfRecord* pThis, OH_UdsContentForm* contentForm);
int OH_UdmfRecord_GetContentForm(OH_UdmfRecord* pThis, OH_UdsContentForm* contentForm);
```

## UnifiedData 存取 API

### 存储数据

```c
// 存储 UnifiedData
// 参数：intention - 使用意图
//        unifiedData - UnifiedData 实例指针
//        key - 输出参数，返回存储 key
//        keyLen - key 缓冲区长度（应不小于 UDMF_KEY_BUFFER_LEN）
// 返回值：UDMF_E_OK 成功，其他值失败
// @since 12
int OH_Udmf_SetUnifiedData(Udmf_Intention intention, OH_UdmfData* unifiedData,
                           char* key, unsigned int keyLen);

// 带选项的存储
// @since 20
int OH_Udmf_SetUnifiedDataByOptions(OH_UdmfOptions* options,
                                    OH_UdmfData *unifiedData,
                                    char *key, unsigned int keyLen);
```

### 检索数据

```c
// 根据 key 检索数据
// 参数：key - 存储时返回的 key
//        intention - 使用意图
//        unifiedData - 输出参数，UnifiedData 实例指针
// 返回值：UDMF_E_OK 成功，其他值失败
// @since 12
int OH_Udmf_GetUnifiedData(const char* key, Udmf_Intention intention,
                          OH_UdmfData* unifiedData);

// 根据选项检索数据
// 参数：options - 查询选项
//        dataArray - 输出参数，UnifiedData 数组指针
//        dataSize - 输出参数，数据数量
// 返回值：UDMF_E_OK 成功，其他值失败
// @since 20
int OH_Udmf_GetUnifiedDataByOptions(OH_UdmfOptions* options,
                                    OH_UdmfData** dataArray, unsigned int* dataSize);
```

### 更新数据

```c
// 更新数据
// 参数：options - 查询选项
//        unifiedData - 新的 UnifiedData
// 返回值：UDMF_E_OK 成功，其他值失败
// @since 20
int OH_Udmf_UpdateUnifiedData(OH_UdmfOptions* options, OH_UdmfData* unifiedData);
```

### 删除数据

```c
// 删除数据
// 参数：options - 查询选项
//        dataArray - 输出参数，被删除的 UnifiedData 数组
//        dataSize - 输出参数，数据数量
// 返回值：UDMF_E_OK 成功，其他值失败
// @since 20
int OH_Udmf_DeleteUnifiedData(OH_UdmfOptions* options,
                              OH_UdmfData** dataArray, unsigned int* dataSize);

// 释放数据数组
void OH_Udmf_DestroyDataArray(OH_UdmfData** dataArray, unsigned int dataSize);
```

## Property API

### 创建与销毁

```c
// 从 UnifiedData 创建 Property
// 参数：unifiedData - UnifiedData 实例指针
// 返回值：Property 实例指针，失败返回 nullptr
// @since 12
OH_UdmfProperty* OH_UdmfProperty_Create(OH_UdmfData* unifiedData);

// 销毁 Property 实例
// 参数：pThis - Property 实例指针
// @since 12
void OH_UdmfProperty_Destroy(OH_UdmfProperty* pThis);
```

### 获取属性

```c
// 获取标签
const char* OH_UdmfProperty_GetTag(OH_UdmfProperty* pThis);

// 获取时间戳
int64_t OH_UdmfProperty_GetTimestamp(OH_UdmfProperty* pThis);

// 获取共享选项
Udmf_ShareOption OH_UdmfProperty_GetShareOption(OH_UdmfProperty* pThis);

// 获取扩展参数
int OH_UdmfProperty_GetExtrasIntParam(OH_UdmfProperty* pThis,
                                      const char* key, int defaultValue);
const char* OH_UdmfProperty_GetExtrasStringParam(OH_UdmfProperty* pThis,
                                                   const char* key);
```

### 设置属性

```c
// 设置标签
int OH_UdmfProperty_SetTag(OH_UdmfProperty* pThis, const char* tag);

// 设置共享选项
int OH_UdmfProperty_SetShareOption(OH_UdmfProperty* pThis,
                                    Udmf_ShareOption option);

// 设置扩展参数
int OH_UdmfProperty_SetExtrasIntParam(OH_UdmfProperty* pThis,
                                       const char* key, int param);
int OH_UdmfProperty_SetExtrasStringParam(OH_UdmfProperty* pThis,
                                          const char* key, const char* param);
```

## UTD API

### 类型描述符管理

```c
// 创建类型描述符
// 参数：typeId - 类型标识符
// 返回值：类型描述符指针，失败返回 nullptr
// @since 12
OH_Utd* OH_Utd_Create(const char* typeId);

// 销毁类型描述符
// 参数：pThis - 类型描述符指针
// @since 12
void OH_Utd_Destroy(OH_Utd* pThis);
```

### 类型元数据获取

```c
// 获取类型标识符
const char* OH_Utd_GetTypeId(OH_Utd* pThis);

// 获取描述
const char* OH_Utd_GetDescription(OH_Utd* pThis);

// 获取参考 URL
const char* OH_Utd_GetReferenceUrl(OH_Utd* pThis);

// 获取图标文件
const char* OH_Utd_GetIconFile(OH_Utd* pThis);

// 获取归属类型列表
const char** OH_Utd_GetBelongingToTypes(OH_Utd* pThis, unsigned int* count);

// 获取文件扩展名列表
const char** OH_Utd_GetFilenameExtensions(OH_Utd* pThis, unsigned int* count);

// 获取 MIME 类型列表
const char** OH_Utd_GetMimeTypes(OH_Utd* pThis, unsigned int* count);
```

### 类型识别

```c
// 根据文件扩展名获取类型
const char** OH_Utd_GetTypesByFilenameExtension(const char* extension,
                                                 unsigned int* count);

// 根据 MIME 类型获取类型
const char** OH_Utd_GetTypesByMimeType(const char* mimeType,
                                         unsigned int* count);
```

### 类型关系判断

```c
// 判断类型归属
// 参数：srcTypeId - 源类型
//        destTypeId - 目标类型
// 返回值：true 归属，false 不归属
// @since 12
bool OH_Utd_BelongsTo(const char* srcTypeId, const char* destTypeId);

// 判断是否下级类型
bool OH_Utd_IsLower(const char* srcTypeId, const char* destTypeId);

// 判断是否上级类型
bool OH_Utd_IsHigher(const char* srcTypeId, const char* destTypeId);

// 判断类型相等
bool OH_Utd_Equals(OH_Utd* utd1, OH_Utd* utd2);
```

### 资源释放

```c
// 释放字符串列表
// 参数：list - 字符串列表指针
//        count - 字符串数量
// @since 12
void OH_Utd_DestroyStringList(const char** list, unsigned int count);
```

## 数据提供者 API

### 创建提供者

```c
// 创建数据提供者实例
// 返回值：提供者实例指针，失败返回 nullptr
// @since 13
OH_UdmfRecordProvider* OH_UdmfRecordProvider_Create();

// 销毁数据提供者
// 参数：provider - 提供者实例指针
// 返回值：UDMF_E_OK 成功，UDMF_E_INVALID_PARAM 参数无效
// @since 13
int OH_UdmfRecordProvider_Destroy(OH_UdmfRecordProvider* provider);
```

### 设置数据回调

```c
// 数据回调函数类型
typedef void* (*OH_UdmfRecordProvider_GetData)(void* context, const char* type);

// 资源释放回调类型
typedef void (*UdmfData_Finalize)(void* context);

// 设置数据获取回调
// 参数：provider - 提供者实例指针
//        context - 上下文指针
//        callback - 数据获取回调函数
//        finalize - 资源释放回调（可选）
// 返回值：UDMF_E_OK 成功，UDMF_E_INVALID_PARAM 参数无效
// @since 13
int OH_UdmfRecordProvider_SetData(OH_UdmfRecordProvider* provider, void* context,
                                   const OH_UdmfRecordProvider_GetData callback,
                                   const UdmfData_Finalize finalize);

// 设置提供者到记录
int OH_UdmfRecord_SetProvider(OH_UdmfRecord* pThis, const char* const* types,
                               unsigned int count,
                               OH_UdmfRecordProvider* provider);
```

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [02_Data_Types.md](./02_Data_Types.md)：数据类型体系
- [10_NAPI_Reference.md](./10_NAPI_Reference.md)：N-API 接口
- [12_InnerKit_Reference.md](./12_InnerKit_Reference.md)：InnerKit 接口
