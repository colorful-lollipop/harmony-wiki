# N-API 接口规范

## 接口概述

N-API 是 OpenHarmony 提供的原生 Node.js API 接口层，UDMF 通过 N-API 向 JS/ArkTS 应用提供数据管理能力。N-API 具有 ABI 稳定性保证，编译一次即可在不同 Node.js 版本间复用，是 OpenHarmony 应用开发的主要接口形式。

UDMF N-API 层包含三个模块：`data.uniformTypeDescriptor`（统一类型描述符）、`data.unifiedDataChannel`（统一数据通道）和 `data.intelligence`（智能数据处理）。三个模块分别提供类型管理、数据操作和 AI 能力，形成完整的应用开发接口体系。

根据源码证据（`interfaces/jskits/module/*.cpp`），N-API 模块遵循标准注册模式：首先定义 `napi_module` 结构体，指定模块名、注册函数等元信息；然后通过 `napi_module_register()` 在共享库加载时注册模块；最后在注册函数中通过 `napi_define_properties()` 导出类、方法和枚举。

## 模块注册机制

### 注册入口点

三个 N-API 模块的注册文件如下：

| 模块名 | 注册文件 | 注册行号 |
|--------|----------|----------|
| data.uniformTypeDescriptor | `interfaces/jskits/module/uniform_type_descriptor_napi_module.cpp` | 40-44 |
| data.unifiedDataChannel | `interfaces/jskits/module/unified_data_channel_napi_module.cpp` | 97-101 |
| data.intelligence | `framework/jskitsimpl/intelligence/native_module_intelligence.cpp` | 43-46 |

### 注册代码模式

以 `uniform_type_descriptor_napi_module.cpp` 为例（`interfaces/jskits/module/uniform_type_descriptor_napi_module.cpp:32-44`）：

```cpp
// 定义 napi_module 结构体
static napi_module _module = {
    .nm_version = 1,                           // 模块版本
    .nm_flags = 0,                              // 预留标志
    .nm_filename = nullptr,                     // 模块文件名
    .nm_register_func = Init,                    // 初始化函数
    .nm_modname = "data.uniformTypeDescriptor",  // 模块名（JS 中访问名）
    .nm_priv = ((void *)0),                     // 私有数据
    .reserved = { 0 }                            // 保留字段
};

// 构造函数属性确保模块自动注册
extern "C" __attribute__((constructor)) void RegisterUDMFUnifiedDataModule(void)
{
    napi_module_register(&_module);  // 注册模块
}
```

### 属性定义模式

在模块初始化函数中，通过 `napi_define_properties` 导出类、方法和枚举（`framework/jskitsimpl/data/uniform_type_descriptor_napi.cpp:27-38`）：

```cpp
// 属性描述符数组
napi_property_descriptor desc[] = {
    // 导出枚举对象
    DECLARE_NAPI_PROPERTY("UniformDataType", uniformDataType),
    // 导出静态函数
    DECLARE_NAPI_FUNCTION("getTypeDescriptor", GetTypeDescriptor),
    DECLARE_NAPI_FUNCTION("getUniformDataTypeByFilenameExtension", GetUniformDataTypeByFilenameExtension),
    DECLARE_NAPI_FUNCTION("getUniformDataTypeByMIMEType", GetUniformDataTypeByMIMEType),
    // 注册/注销类型
    DECLARE_NAPI_FUNCTION("registerTypeDescriptors", RegisterTypeDescriptors),
    DECLARE_NAPI_FUNCTION("unregisterTypeDescriptors", UnregisterTypeDescriptors),
};

// 定义属性
NAPI_CALL(env, napi_define_properties(env, exports, 
    sizeof(desc) / sizeof(desc[0]), desc));
```

## data.uniformTypeDescriptor 模块

### 模块概述

`data.uniformTypeDescriptor` 模块提供统一类型描述符（UTD）的查询和管理功能，是 UDMF 类型系统的 JS 接口层。应用可以通过该模块查询类型信息、判断类型归属关系、注册自定义类型。

### 导出 API 清单

| JS API | 参数类型 | 返回类型 | C++ 实现位置 |
|--------|----------|----------|--------------|
| getTypeDescriptor(typeId: string): TypeDescriptor | 字符串 | TypeDescriptor 对象 | `uniform_type_descriptor_napi.cpp` |
| getUniformDataTypeByFilenameExtension(ext: string, belongsTo?: string): string | 字符串，可选字符串 | UTD 类型标识符 | `uniform_type_descriptor_napi.cpp` |
| getUniformDataTypeByMIMEType(mime: string, belongsTo?: string): string | 字符串，可选字符串 | UTD 类型标识符 | `uniform_type_descriptor_napi.cpp` |
| getUniformDataTypesByFilenameExtension(ext: string, belongsTo?: string): string[] | 字符串，可选字符串 | UTD 类型标识符数组 | `uniform_type_descriptor_napi.cpp` |
| getUniformDataTypesByMIMEType(mime: string, belongsTo?: string): string[] | 字符串，可选字符串 | UTD 类型标识符数组 | `uniform_type_descriptor_napi.cpp` |
| registerTypeDescriptors(descriptors: TypeDescriptor[]): Promise\<void\> | TypeDescriptor 数组 | Promise | `uniform_type_descriptor_napi.cpp` |
| unregisterTypeDescriptors(typeIds: string[]): Promise\<void\> | 字符串数组 | Promise | `uniform_type_descriptor_napi.cpp` |

### 导出类

**TypeDescriptor 类**

| 实例方法 | 参数类型 | 返回类型 | 说明 |
|----------|----------|----------|------|
| getTypeId(): string | 无 | 字符串 | 获取类型标识符 |
| getDescription(): string | 无 | 字符串 | 获取类型描述 |
| getReferenceURL(): string | 无 | 字符串 | 获取参考链接 |
| getIconFile(): string | 无 | 字符串 | 获取图标路径 |
| getBelongingToTypes(): string[] | 无 | 字符串数组 | 获取归属类型列表 |
| getFilenameExtensions(): string[] | 无 | 字符串数组 | 获取文件扩展名列表 |
| getMimeTypes(): string[] | 无 | 字符串数组 | 获取 MIME 类型列表 |
| belongsTo(typeId: string): boolean | 字符串 | 布尔值 | 判断是否属于目标类型 |
| equals(other: TypeDescriptor): boolean | TypeDescriptor | 布尔值 | 判断类型是否相等 |

### 导出枚举

**UniformDataType 枚举**

`UniformDataType` 是一个冻结对象（Frozen Object），包含所有预定义 UTD 类型标识符常量。部分示例：

```javascript
// 文本类型
UniformDataType.PLAIN_TEXT = 'general.plain-text';
UniformDataType.HTML = 'general.html';
UniformDataType.HYPERLINK = 'general.hyperlink';

// 图像类型
UniformDataType.IMAGE = 'general.image';
UniformDataType.JPEG = 'general.jpeg';
UniformDataType.PNG = 'general.png';

// 视频类型
UniformDataType.VIDEO = 'general.video';
UniformDataType.MPEG4 = 'general.mpeg-4';

// OpenHarmony 类型
UniformDataType.FORM = 'openharmony.form';
UniformDataType.APP_ITEM = 'openharmony.app-item';
UniformDataType.PIXEL_MAP = 'openharmony.pixel-map';
```

## data.unifiedDataChannel 模块

### 模块概述

`data.unifiedDataChannel` 模块是 UDMF 的核心数据操作模块，提供统一数据的创建、存储、查询、更新和删除等 CRUD 操作。该模块是应用进行跨应用、跨设备数据交互的主要入口。

### 导出 API 清单

| JS API | 参数类型 | 返回类型 | 同步/异步 | 说明 |
|--------|----------|----------|-----------|------|
| insertData(options: CustomOption, unifiedData: UnifiedData): Promise\<string\> | CustomOption, UnifiedData | Promise\<string\> | 异步 | 插入数据，返回 key |
| updateData(options: CustomOption, unifiedData: UnifiedData): Promise\<void\> | CustomOption, UnifiedData | Promise | 异步 | 更新数据 |
| queryData(options: QueryOption): Promise\<UnifiedData[]\> | QueryOption | Promise\<UnifiedData[]\> | 异步 | 查询数据 |
| deleteData(options: QueryOption): Promise\<UnifiedData[]\> | QueryOption | Promise\<UnifiedData[]\> | 异步 | 删除数据 |
| convertRecordsToEntries(unifiedData: UnifiedData): void | UnifiedData | void | 同步 | 记录转条目 |
| setAppShareOptions(intention: string, shareOption: ShareOptions): void | 枚举 | void | 同步 | 设置共享选项 |
| removeAppShareOptions(intention: string): void | 字符串 | void | 同步 | 移除共享选项 |

### 导出枚举

**Intention 枚举**

`Intention` 枚举定义数据的使用意图（`unified_data_channel_napi.cpp:33-42`）：

```javascript
Intention.DATA_HUB = 'dataHub';         // 数据中心
Intention.DRAG = 'drag';                 // 拖拽
Intention.PICKER = 'picker';             // 选择器
Intention.SYSTEM_SHARE = 'systemShare';  // 系统分享
Intention.MENU = 'menu';                 // 菜单
```

**ShareOptions 枚举**

```javascript
ShareOptions.IN_APP = 0;         // 仅应用内可用
ShareOptions.CROSS_APP = 1;      // 允许跨应用
```

**FileConflictOptions 枚举**

```javascript
FileConflictOptions.OVERWRITE = 0;  // 覆盖已存在文件
FileConflictOptions.SKIP = 1;      // 跳过已存在文件
```

**ProgressIndicator 枚举**

```javascript
ProgressIndicator.NONE = 0;     // 不显示进度
ProgressIndicator.DEFAULT = 1;   // 使用系统默认进度
```

**ListenerStatus 枚举**

```javascript
ListenerStatus.FINISHED = 0;         // 处理完成
ListenerStatus.PROCESSING = 1;        // 处理中
ListenerStatus.INNER_ERROR = 2;      // 内部错误
ListenerStatus.INVALID_PARAMETERS = 3;  // 参数无效
ListenerStatus.SYNC_FAILED = 4;      // 同步失败
ListenerStatus.COPY_FILE_FAILED = 5;  // 文件复制失败
```

**Visibility 枚举**

```javascript
Visibility.ALL = 0;           // 任意 Hap 或 Native 可获取
Visibility.OWN_PROCESS = 1;    // 仅数据提供方可获取
```

### 导出类（18 个）

**UnifiedData 类**

| 实例方法/属性 | 类型 | 说明 |
|---------------|------|------|
| constructor() | 构造函数 | 创建空 UnifiedData |
| addRecord(record: UnifiedRecord): void | 方法 | 添加记录 |
| getRecords(): UnifiedRecord[] | 方法 | 获取所有记录 |
| hasType(type: string): boolean | 方法 | 判断是否包含类型 |
| getTypes(): string[] | 方法 | 获取所有类型 |
| properties: UnifiedDataProperties | 属性 | 数据属性 |

**UnifiedRecord 类**

| 实例方法/属性 | 类型 | 说明 |
|---------------|------|------|
| constructor(type: string, value?: any) | 构造函数 | 创建记录 |
| addEntry(type: string, value: any): void | 方法 | 添加条目 |
| getEntry(type: string): any | 方法 | 获取条目 |
| getTypes(): string[] | 方法 | 获取所有类型 |

**数据记录类型类**

UnifiedDataChannel 模块导出以下具体数据类型类：

| 类名 | 用途 | C++ 实现位置 |
|------|------|--------------|
| Text | 文本基类 | `text_napi.h/cpp` |
| PlainText | 纯文本 | `plain_text_napi.h/cpp` |
| Hyperlink | 超链接 | `link_napi.h/cpp` |
| HTML | HTML 内容 | `html_napi.h/cpp` |
| File | 文件引用 | `file_napi.h/cpp` |
| Image | 图像数据 | `image_napi.h/cpp` |
| Video | 视频数据 | `video_napi.h/cpp` |
| Audio | 音频数据 | `audio_napi.h/cpp` |
| Folder | 文件夹 | `folder_napi.h/cpp` |

**系统定义记录类型**

| 类名 | 用途 |
|------|------|
| SystemDefinedRecord | 系统记录基类 |
| SystemDefinedForm | UI 卡片信息 |
| SystemDefinedAppItem | 应用描述信息 |
| SystemDefinedPixelMap | 缩略图数据 |

**应用定义记录类型**

| 类名 | 用途 |
|------|------|
| ApplicationDefinedRecord | 应用自定义记录 |

## data.intelligence 模块

### 模块概述

`data.intelligence` 模块提供 AI/ML 相关的数据处理能力，包括文本嵌入和图像嵌入功能。该模块需要设备具备相应的 AI 计算能力。

### 导出类

**TextEmbedding 类**

| 静态方法 | 参数类型 | 返回类型 | 说明 |
|----------|----------|----------|------|
| getTextEmbeddingModel(config: ModelConfig): Promise\<TextEmbedding\> | ModelConfig | Promise\<TextEmbedding\> | 获取文本嵌入模型 |
| splitText(text: string, config: SplitConfig): Promise\<string[]\> | 字符串, SplitConfig | Promise\<string[]\> | 文本分词 |

| 实例方法 | 参数类型 | 返回类型 | 说明 |
|----------|----------|----------|------|
| loadModel(): Promise\<void\> | 无 | Promise | 加载模型 |
| releaseModel(): Promise\<void\> | 无 | Promise | 释放模型 |
| getEmbedding(text: string \| string[]): Promise\<number[][]\> | 字符串或数组 | Promise\<number[][]\> | 获取文本嵌入 |

**ImageEmbedding 类**

| 静态方法 | 参数类型 | 返回类型 | 说明 |
|----------|----------|----------|------|
| getImageEmbeddingModel(config: ModelConfig): Promise\<ImageEmbedding\> | ModelConfig | Promise\<ImageEmbedding\> | 获取图像嵌入模型 |

| 实例方法 | 参数类型 | 返回类型 | 说明 |
|----------|----------|----------|------|
| loadModel(): Promise\<void\> | 无 | Promise | 加载模型 |
| releaseModel(): Promise\<void\> | 无 | Promise | 释放模型 |
| getEmbedding(image: PixelMap): Promise\<number[][]\> | PixelMap | Promise\<number[][]\> | 获取图像嵌入 |

**ModelVersion 枚举**

```javascript
ModelVersion.BASIC_MODEL = 0;  // 基础模型
```

## 异步回调机制

### Promise 模式

N-API 层使用 Promise 处理异步操作，通过 `NapiQueue::AsyncWork` 框架实现（`interfaces/jskits/common/napi_queue.h`）：

```cpp
// 异步工作定义
static napi_value AsyncWork(
    napi_env env,
    std::shared_ptr<ContextBase> ctxt,      // 上下文
    const std::string &name,                  // 工作名称
    NapiAsyncExecute execute,                 // 执行函数（后台线程）
    NapiAsyncComplete complete                // 完成函数（主线程）
);
```

### Threadsafe Function 模式

对于需要实时回调的场景（如进度监听），使用 `napi_threadsafe_function`（`get_data_params_napi.cpp:95-96`）：

```cpp
// 创建 threadsafe function
napi_create_threadsafe_function(env, callback, nullptr, workName,
    0, 1, nullptr, CallProgressListener, &tsfn);

// 从原生线程调用
napi_call_threadsafe_function(tsfn, listenerArgs, napi_tsfn_blocking);

// 释放资源
napi_release_threadsafe_function(tsfn, napi_tsfn_release);
```

### 进度监听示例

```javascript
// 创建 GetDataParams
let params = new GetDataOptions();
params.intention = 'drag';
params.key = 'data-key';

// 设置进度监听器
params.setProgressListener((progressInfo, data) => {
    console.log(`Status: ${progressInfo.status}`);
    console.log(`Progress: ${progressInfo.progress}`);
    if (data) {
        console.log('Data received');
    }
});
```

## 参数校验与错误处理

### 参数校验点

N-API 层在每个入口函数中进行参数校验（`napi_data_utils.h/cpp`）：

- **类型校验**：使用 `napi_typeof` 检查参数类型
- **空值校验**：检查必填参数是否为 null/undefined
- **范围校验**：检查数值、数组长度是否在有效范围内
- **字符串校验**：检查编码、长度、特殊字符

### 错误码映射

| N-API 错误码 | C++ 错误码 | 含义 |
|--------------|------------|------|
| napi_ok | Status::E_OK | 成功 |
| napi_invalid_arg | Status::E_INVALID_PARAMETERS | 参数无效 |
| napi_generic_failure | Status::E_ERROR | 通用错误 |
| napi_pending_exception | Status::E_IPC | IPC 通信失败 |

### 异常封装

使用 `napi_error_utils.h/cpp` 中的工具函数封装异常：

```cpp
// 抛出 JavaScript 异常
NAPI_THROW_IF_FAILED(env, napi_throw_error(env, ...), ...);

// 设置返回值并抛出异常
NAPI_CALL_RETURN_VOID(env, napi_throw_type_error(env, ...));
```

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [02_Data_Types.md](./02_Data_Types.md)：数据类型体系
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口
- [12_InnerKit_Reference.md](./12_InnerKit_Reference.md)：InnerKit 接口
