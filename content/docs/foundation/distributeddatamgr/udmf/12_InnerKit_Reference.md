# InnerKit 接口规范

## 接口概述

InnerKit 是 OpenHarmony 为系统组件提供的 C++ 编程接口，位于 `interfaces/innerkits/` 目录。与 N-API 和 NDK 相比，InnerKit 接口面向系统级组件，提供更完整的类型系统和更灵活的扩展能力，是 OpenHarmony 系统内部组件访问 UDMF 功能的主要方式。

InnerKit 接口编译生成两个共享库：`libudmf_client.so` 和 `libutd_client.so`（`interfaces/innerkits/BUILD.gn`）。接口设计遵循 C++ 规范，使用异常处理和智能指针，通过返回值（Status 枚举）报告执行状态。

根据源码证据（`interfaces/innerkits/client/udmf_client.h`），InnerKit 提供两个核心单例类：`UdmfClient`（数据操作客户端）和 `UtdClient`（类型管理客户端）。系统组件通过这两个单例访问 UDMF 的全部功能。

## 头文件清单

| 头文件 | 用途 | 主要内容 |
|--------|------|----------|
| `client/udmf_client.h` | UDMF 客户端 | 数据 CRUD、权限、同步 |
| `client/utd_client.h` | UTD 客户端 | 类型查询、注册管理 |
| `client/udmf_async_client.h` | 异步客户端 | 延迟数据加载 |
| `data/unified_data.h` | 统一数据 | UnifiedData 类定义 |
| `data/unified_record.h` | 统一记录 | UnifiedRecord 类定义 |
| `data/type_descriptor.h` | 类型描述符 | TypeDescriptor 类定义 |
| `data/unified_data_properties.h` | 数据属性 | 属性结构定义 |
| `common/unified_meta.h` | 元数据 | UDType、Intention 枚举 |
| `common/unified_types.h` | 类型定义 | Summary、Runtime 等 |
| `common/error_code.h` | 错误码 | Status 枚举 |
| `common/visibility.h` | 可见性 | API_EXPORT 宏 |

## UdmfClient 接口

### 单例获取

```cpp
// 获取 UdmfClient 单例实例
// 返回值：UdmfClient 引用
// 线程安全：首次访问时自动初始化
// @since 12
static UdmfClient API_EXPORT &GetInstance();
```

### 数据操作 API

**插入数据**

```cpp
// 设置数据
// 参数：option - 自定义选项（包含 intention、key 等）
//        unifiedData - 要存储的统一数据
//        key - 输出参数，返回存储 key
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT SetData(CustomOption &option, UnifiedData &unifiedData,
                          std::string &key);
```

**查询数据**

```cpp
// 查询单条数据
// 参数：query - 查询选项
//        unifiedData - 输出参数，查询结果
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetData(const QueryOption &query, UnifiedData &unifiedData);

// 批量查询数据
// 参数：query - 查询选项
//        unifiedDataSet - 输出参数，数据集合
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetBatchData(const QueryOption &query,
                                std::vector<UnifiedData> &unifiedDataSet);
```

**更新数据**

```cpp
// 更新数据
// 参数：query - 查询选项
//        unifiedData - 新的统一数据
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT UpdateData(const QueryOption &query, UnifiedData &unifiedData);
```

**删除数据**

```cpp
// 删除数据
// 参数：query - 查询选项
//        unifiedDataSet - 输出参数，被删除的数据集合
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT DeleteData(const QueryOption &query,
                             std::vector<UnifiedData> &unifiedDataSet);
```

### 权限与同步 API

**获取摘要**

```cpp
// 获取数据摘要
// 参数：query - 查询选项
//        summary - 输出参数，数据摘要
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetSummary(const QueryOption &query, Summary& summary);
```

**添加权限**

```cpp
// 添加访问权限
// 参数：query - 查询选项
//        privilege - 权限信息
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT AddPrivilege(const QueryOption &query, Privilege &privilege);
```

**同步数据**

```cpp
// 同步数据到设备
// 参数：query - 查询选项
//        devices - 目标设备列表
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT Sync(const QueryOption &query,
                       const std::vector<std::string> &devices);
```

**远程数据检查**

```cpp
// 检查是否为远程数据
// 参数：query - 查询选项
//        result - 输出参数，true 表示远程数据
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT IsRemoteData(const QueryOption &query, bool &result);
```

### 共享选项 API

```cpp
// 设置应用共享选项
// 参数：intention - 使用意图
//        shareOption - 共享选项
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT SetAppShareOption(const std::string &intention,
                                    enum ShareOptions shareOption);

// 移除应用共享选项
// 参数：intention - 使用意图
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT RemoveAppShareOption(const std::string &intention);

// 获取应用共享选项
// 参数：intention - 使用意图
//        shareOption - 输出参数，共享选项
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetAppShareOption(const std::string &intention,
                                    enum ShareOptions &shareOption);
```

### 延迟数据加载 API

```cpp
// 设置延迟加载信息
// 参数：dataLoadParams - 延迟加载参数
//        key - 输出参数，返回加载 key
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT SetDelayInfo(const DataLoadParams &dataLoadParams,
                                std::string &key);

// 推送延迟数据
// 参数：key - 加载 key
//        unifiedData - 统一数据
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT PushDelayData(const std::string &key,
                                UnifiedData &unifiedData);

// 检查数据是否可用（用于延迟加载场景）
// 参数：key - 加载 key
//        dataLoadInfo - 数据加载信息
//        iUdmfNotifier - 远程通知回调
//        unifiedData - 输出参数，统一数据
// 返回值：Status 执行状态
// @since 12
Status GetDataIfAvailable(const std::string &key,
                           const DataLoadInfo &dataLoadInfo,
                           sptr<IRemoteObject> iUdmfNotifier,
                           std::shared_ptr<UnifiedData> unifiedData);
```

### 工具 API

```cpp
// 根据 key 获取来源包名
// 参数：key - 数据 key
// 返回值：来源包名
// @since 12
std::string API_EXPORT GetBundleNameByUdKey(const std::string &key);

// 检查类型是否匹配
// 参数：summary - 数据摘要
//        allowTypes - 允许的类型列表
// 返回值：true 匹配，false 不匹配
// @since 12
bool API_EXPORT IsAppropriateType(const Summary &summary,
                                   const std::vector<std::string> &allowTypes);

// 获取父类型
// 参数：oldSummary - 原始摘要
//        newSummary - 输出参数，父类型摘要
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetParentType(Summary &oldSummary, Summary &newSummary);
```

## UtdClient 接口

### 单例获取

```cpp
// 获取 UtdClient 单例实例
// 返回值：UtdClient 引用
// 线程安全：使用 std::call_once 保证初始化安全
// @since 12
static UtdClient API_EXPORT &GetInstance();
```

### 类型描述符 API

```cpp
// 获取类型描述符
// 参数：typeId - 类型标识符
//        descriptor - 输出参数，类型描述符
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetTypeDescriptor(const std::string &typeId,
                                     std::shared_ptr<TypeDescriptor> &descriptor);
```

### 类型识别 API

```cpp
// 根据文件扩展名获取类型
// 参数：fileExtension - 文件扩展名（如 ".png"）
//        typeId - 输出参数，类型标识符
//        belongsTo - 归属类型（可选）
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetUniformDataTypeByFilenameExtension(
    const std::string &fileExtension,
    std::string &typeId,
    std::string belongsTo = DEFAULT_TYPE_ID);

// 根据文件扩展名获取多个类型
// 参数：fileExtension - 文件扩展名
//        typeIds - 输出参数，类型标识符数组
//        belongsTo - 归属类型（可选）
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetUniformDataTypesByFilenameExtension(
    const std::string &fileExtension,
    std::vector<std::string> &typeIds,
    const std::string &belongsTo = DEFAULT_TYPE_ID);

// 根据 MIME 类型获取类型
// 参数：mimeType - MIME 类型（如 "image/png"）
//        typeId - 输出参数，类型标识符
//        belongsTo - 归属类型（可选）
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetUniformDataTypeByMIMEType(
    const std::string &mimeType,
    std::string &typeId,
    std::string belongsTo = DEFAULT_TYPE_ID);

// 根据 MIME 类型获取多个类型
// 参数：mimeType - MIME 类型
//        typeIds - 输出参数，类型标识符数组
//        belongsTo - 归属类型（可选）
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT GetUniformDataTypesByMIMEType(
    const std::string &mimeType,
    std::vector<std::string> &typeIds,
    const std::string &belongsTo = DEFAULT_TYPE_ID);
```

### 类型验证 API

```cpp
// 验证是否为有效 UTD 类型
// 参数：typeId - 类型标识符
//        result - 输出参数，true 表示有效
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT IsUtd(std::string typeId, bool &result);
```

### 自定义类型管理 API

```cpp
// 安装自定义 UTD 配置
// 参数：bundleName - 包名
//        jsonStr - UTD 配置 JSON 字符串
//        user - 用户 ID
// @since 12
void API_EXPORT InstallCustomUtds(const std::string &bundleName,
                                 const std::string &jsonStr, int32_t user);

// 卸载自定义 UTD
// 参数：bundleName - 包名
//        user - 用户 ID
// @since 12
void API_EXPORT UninstallCustomUtds(const std::string &bundleName, int32_t user);

// 注册自定义类型描述符
// 参数：descriptors - 类型描述符配置数组
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT RegisterTypeDescriptors(
    const std::vector<TypeDescriptorCfg> &descriptors);

// 注销自定义类型
// 参数：typeIds - 类型标识符数组
// 返回值：Status 执行状态
// @since 12
Status API_EXPORT UnregisterTypeDescriptors(
    const std::vector<std::string> &typeIds);
```

## UnifiedData 接口

### 构造函数

```cpp
// 默认构造函数
API_EXPORT UnifiedData();

// 带属性的构造函数
// 参数：properties - 统一数据属性
explicit API_EXPORT UnifiedData(std::shared_ptr<UnifiedDataProperties> properties);
```

### 数据管理

```cpp
// 获取数据大小
int64_t API_EXPORT GetSize();

// 获取组 ID
std::string GetGroupId() const;

// 添加记录
void API_EXPORT AddRecord(const std::shared_ptr<UnifiedRecord> &record);
void API_EXPORT AddRecords(const std::vector<std::shared_ptr<UnifiedRecord>> &records);

// 获取记录
std::shared_ptr<UnifiedRecord> API_EXPORT GetRecordAt(std::size_t index) const;
void API_EXPORT SetRecords(std::vector<std::shared_ptr<UnifiedRecord>> records);
std::vector<std::shared_ptr<UnifiedRecord>> API_EXPORT GetRecords() const;
```

### 类型查询

```cpp
// 获取类型标签
std::vector<std::string> API_EXPORT GetTypesLabels() const;

// 检查是否包含类型
bool API_EXPORT HasType(const std::string &type) const;
bool API_EXPORT HasHigherFileType(const std::string &type) const;
std::vector<std::string> API_EXPORT GetEntriesTypes() const;
bool API_EXPORT HasTypeInEntries(const std::string &type) const;
```

### 验证

```cpp
// 数据校验
bool API_EXPORT IsEmpty() const;
bool API_EXPORT IsValid();
bool API_EXPORT IsComplete();
```

### 属性管理

```cpp
// 获取运行时信息
std::shared_ptr<Runtime> API_EXPORT GetRuntime() const;
void API_EXPORT SetRuntime(Runtime &runtime);

// 属性管理
void API_EXPORT SetProperties(std::shared_ptr<UnifiedDataProperties> properties);
std::shared_ptr<UnifiedDataProperties> API_EXPORT GetProperties() const;

// 最大数据大小限制
static constexpr int64_t MAX_DATA_SIZE = 200 * 1024 * 1024;  // 200MB
```

## UnifiedRecord 接口

### 构造函数

```cpp
// 默认构造函数
API_EXPORT UnifiedRecord();

// 带类型的构造函数
explicit API_EXPORT UnifiedRecord(UDType type);

// 带类型和值的构造函数
API_EXPORT UnifiedRecord(UDType type, ValueType value);
```

### 类型管理

```cpp
// 获取类型
UDType API_EXPORT GetType() const;
std::vector<std::string> API_EXPORT GetTypes() const;
void API_EXPORT SetType(const UDType &type);
```

### 唯一标识

```cpp
// UID 管理
std::string API_EXPORT GetUid() const;
void API_EXPORT SetUid(const std::string &id);
```

### 值管理

```cpp
// 值操作
ValueType API_EXPORT GetValue();
void SetValue(const ValueType &value);
ValueType API_EXPORT GetOriginValue() const;
```

### UTD 管理

```cpp
// UTD ID 管理
void API_EXPORT SetUtdId(const std::string &utdId);
std::set<std::string> API_EXPORT GetUtdIds() const;
std::string API_EXPORT GetUtdId() const;
void API_EXPORT SetUtdId2(const std::string &utdId);
std::string API_EXPORT GetUtdId2() const;
```

### Entry 管理

```cpp
// Entry 查询
bool API_EXPORT HasType(const std::string &utdId) const;

// Entry 操作
void API_EXPORT AddEntry(const std::string &utdId, ValueType &&value);
ValueType API_EXPORT GetEntry(const std::string &utdId);
std::shared_ptr<std::map<std::string, ValueType>> API_EXPORT GetEntries();

// Entry Getter（懒加载）
void API_EXPORT SetEntryGetter(const std::vector<std::string> &utdIds,
                               const std::shared_ptr<EntryGetter> &entryGetter);
std::shared_ptr<EntryGetter> API_EXPORT GetEntryGetter();
```

## TypeDescriptor 接口

### 构造函数

```cpp
// 全参数构造函数
API_EXPORT TypeDescriptor(const std::string &typeId,
                           const std::vector<std::string> &belongingToTypes,
                           const std::vector<std::string> &filenameExtensions,
                           const std::vector<std::string> &mimeTypes,
                           const std::string &description,
                           const std::string &referenceURL,
                           const std::string &iconFile);

// 配置构造函数
API_EXPORT TypeDescriptor(const TypeDescriptorCfg& typeDescriptorCfg);

// 默认构造函数
API_EXPORT TypeDescriptor();
```

### 层次查询

```cpp
// 类型归属判断
Status API_EXPORT BelongsTo(const std::string &typeId, bool &checkResult);
Status API_EXPORT IsLowerLevelType(const std::string &typeId, bool &checkResult);
Status API_EXPORT IsHigherLevelType(const std::string &typeId, bool &checkResult);

// 类型比较
bool API_EXPORT Equals(std::shared_ptr<TypeDescriptor> descriptor);
```

### 元数据获取

```cpp
// 基础属性
const std::string& API_EXPORT GetTypeId() const;
std::vector<std::string> API_EXPORT GetBelongingToTypes();
std::string API_EXPORT GetIconFile();
std::string API_EXPORT GetDescription();
std::string API_EXPORT GetReferenceURL();
std::vector<std::string> API_EXPORT GetFilenameExtensions();
std::vector<std::string> API_EXPORT GetMimeTypes();
```

### 元数据设置

```cpp
// 设置属性
void API_EXPORT SetTypeId(const std::string &typeId);
void API_EXPORT SetBelongingToTypes(const std::vector<std::string> &belongingToTypes);
void API_EXPORT SetFilenameExtensions(const std::vector<std::string> &filenameExtensions);
void API_EXPORT SetMimeTypes(const std::vector<std::string> &mimeTypes);
void API_EXPORT SetDescription(const std::string &description);
void API_EXPORT SetReferenceURL(const std::string &referenceURL);
void API_EXPORT SetIconFile(const std::string &iconFile);
```

## 错误码定义

### Status 枚举

```cpp
// 成功状态
E_OK = ERR_OK,                    // 0

// 通用错误
E_ERROR,                          // 通用错误
E_WRITE_PARCEL_ERROR,             // 序列化错误
E_READ_PARCEL_ERROR,              // 反序列化错误
E_IPC,                            // IPC 通信错误
E_NO_PERMISSION,                  // 无权限
E_INVALID_PARAMETERS,            // 参数无效
E_DB_ERROR,                       // 数据库错误
E_FS_ERROR,                       // 文件系统错误
E_NOT_FOUND,                      // 未找到
E_SETTINGS_EXISTED,               // 设置已存在
E_NO_SYSTEM_PERMISSION,           // 无系统权限
E_SYNC_FAILED,                    // 同步失败
E_COPY_FILE_FAILED,               // 文件复制失败
E_IDEMPOTENT_ERROR,              // 幂等性错误
E_COPY_CANCELED,                 // 复制取消
E_DB_CORRUPTED,                  // 数据库损坏
E_JSON_CONVERT_FAILED,           // JSON 转换失败
E_FORMAT_ERROR,                  // 格式错误
E_CONTENT_ERROR,                 // 内容错误
E_INVALID_TYPE_ID                // 无效类型 ID
```

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [02_Data_Types.md](./02_Data_Types.md)：数据类型体系
- [10_NAPI_Reference.md](./10_NAPI_Reference.md)：N-API 接口
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口
- [20_Service_Layer.md](./20_Service_Layer.md)：服务层架构
- [21_Client_Layer.md](./21_Client_Layer.md)：客户端层架构
