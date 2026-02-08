# Data Share 内部 API

## 目的

本文档描述 Data Share 的内部 C++ API，供 Native 开发者使用或理解底层实现。

## 适用范围

- 开发 Native 应用使用 Data Share 的开发者
- 需要理解内部实现的系统开发者
- 维护 Data Share 的工程师

## 接口概述

Data Share 内部 API 按模块分为：

| 模块 | 路径 | 主要职责 |
|------|------|----------|
| Consumer | `interfaces/inner_api/consumer/` | 客户端接口 |
| Provider | `interfaces/inner_api/provider/` | 服务端接口 |
| Common | `interfaces/inner_api/common/` | 公共定义 |
| Permission | `interfaces/inner_api/permission/` | 权限接口 |

## Consumer 模块

### DataShareHelper 类

**头文件**: `interfaces/inner_api/consumer/include/datashare_helper.h:49`

**类定义**:
```cpp
class DataShareHelper : public std::enable_shared_from_this<DataShareHelper> {
public:
    // 工厂方法
    static std::pair<int, std::shared_ptr<DataShareHelper>> Create(
        const sptr<IRemoteObject> &token,
        const std::string &strUri, 
        const std::string &extUri, 
        const int waitTime = 2);
    
    // CRUD 操作
    virtual int Insert(Uri &uri, const DataShareValuesBucket &value) = 0;
    virtual int Update(Uri &uri, const DataSharePredicates &predicates, 
                       const DataShareValuesBucket &value) = 0;
    virtual int Delete(Uri &uri, const DataSharePredicates &predicates) = 0;
    virtual std::shared_ptr<DataShareResultSet> Query(
        Uri &uri, const DataSharePredicates &predicates,
        std::vector<std::string> &columns, 
        DatashareBusinessError *businessError = nullptr) = 0;
    
    // 批量操作
    virtual int BatchInsert(Uri &uri, const std::vector<DataShareValuesBucket> &values) = 0;
    virtual int BatchUpdate(const UpdateOperations &operations, 
                            std::vector<BatchUpdateResult> &results) = 0;
    virtual int ExecuteBatch(const std::vector<OperationStatement> &statements, 
                             ExecResultSet &result) = 0;
    
    // 文件操作
    virtual int OpenFile(Uri &uri, const std::string &mode) = 0;
    virtual int OpenRawFile(Uri &uri, const std::string &mode) = 0;
    virtual std::vector<std::string> GetFileTypes(Uri &uri, 
                                                   const std::string &mimeTypeFilter) = 0;
    
    // 观察者
    virtual int RegisterObserver(const Uri &uri, 
                                 const sptr<AAFwk::IDataAbilityObserver> &dataObserver) = 0;
    virtual int UnregisterObserver(const Uri &uri, 
                                   const sptr<AAFwk::IDataAbilityObserver> &dataObserver) = 0;
    virtual void NotifyChange(const Uri &uri) = 0;
    
    // URI 操作
    virtual Uri NormalizeUri(Uri &uri) = 0;
    virtual Uri DenormalizeUri(Uri &uri) = 0;
    
    // 模板与订阅
    virtual int AddQueryTemplate(const std::string &uri, int64_t subscriberId, Template &tpl) = 0;
    virtual int DelQueryTemplate(const std::string &uri, int64_t subscriberId) = 0;
    virtual std::vector<OperationResult> SubscribeRdbData(
        const std::vector<std::string> &uris,
        const TemplateId &templateId, 
        const std::function<void(const RdbChangeNode &changeNode)> &callback) = 0;
    virtual std::vector<OperationResult> UnsubscribeRdbData(
        const std::vector<std::string> &uris,
        const TemplateId &templateId) = 0;
    
    // 发布数据
    virtual std::vector<OperationResult> Publish(const Data &data, 
                                                     const std::string &bundleName) = 0;
    virtual Data GetPublishedData(const std::string &bundleName, int &resultCode) = 0;
    
    // 资源释放
    virtual bool Release() = 0;
};
```

**使用示例**:
```cpp
#include "datashare_helper.h"

// 创建 Helper
auto [errCode, helper] = OHOS::DataShare::DataShareHelper::Create(
    token, "datashareproxy://com.example.provider/DB00/TBL01?Proxy=true", "", 2);

if (errCode != E_OK || helper == nullptr) {
    // 创建失败处理
    return;
}

// 插入数据
OHOS::DataShare::DataShareValuesBucket values;
values.Put("name", OHOS::DataShare::DataShareValueObject("test"));
values.Put("age", OHOS::DataShare::DataShareValueObject(18));

OHOS::Uri uri("datashareproxy://com.example.provider/DB00/TBL01");
int result = helper->Insert(uri, values);

// 释放资源
helper->Release();
```

### DataShareResultSet 类

**头文件**: `interfaces/inner_api/consumer/include/datashare_result_set.h`

**关键方法**:
```cpp
class DataShareResultSet : public DataShareAbsResultSet {
public:
    // 游标操作
    int GoToRow(int position);
    int GoTo(int offset);
    int GoToFirstRow();
    int GoToLastRow();
    int GoToNextRow();
    int GoToPreviousRow();
    bool IsAtFirstRow();
    bool IsAtLastRow();
    bool IsStarted();
    bool IsEnded();
    int GetRowCount();
    int GetRowIndex();
    
    // 列操作
    int GetColumnCount();
    int GetColumnIndex(const std::string &columnName);
    int GetColumnName(int columnIndex, std::string &columnName);
    std::vector<std::string> GetAllColumnNames();
    
    // 数据获取
    int GetString(int columnIndex, std::string &value);
    int GetInt(int columnIndex, int &value);
    int GetLong(int columnIndex, int64_t &value);
    int GetDouble(int columnIndex, double &value);
    int GetBlob(int columnIndex, std::vector<uint8_t> &value);
    int GetDataType(int columnIndex, DataType &dataType);
    bool IsColumnNull(int columnIndex);
    
    // 共享内存相关
    bool HasSharedMem() const;
};
```

### DataProxyHandle 类

**头文件**: `interfaces/inner_api/consumer/include/dataproxy_handle.h`

**用途**: 用于 DataShareProxy 模式下的数据操作

```cpp
class DataProxyHandle {
public:
    std::vector<OperationResult> Publish(const Data &data, const std::string &bundleName);
    Data GetPublishedData(const std::string &bundleName, int &resultCode);
};
```

## Provider 模块

### ResultSetBridge 类

**头文件**: `interfaces/inner_api/provider/include/result_set_bridge.h:24`

**用途**: 数据库 ResultSet 与 DataShare ResultSet 之间的桥接器

```cpp
class ResultSetBridge {
public:
    // Writer 接口，用于写入数据到共享内存
    class Writer {
    public:
        virtual int AllocRow() = 0;
        virtual int FreeLastRow() = 0;
        virtual int Write(uint32_t column) = 0;  // NULL
        virtual int Write(uint32_t column, int64_t value) = 0;
        virtual int Write(uint32_t column, double value) = 0;
        virtual int Write(uint32_t column, const uint8_t *value, size_t size) = 0;
        virtual int Write(uint32_t column, const char *value, size_t size) = 0;
    };
    
    virtual int GetAllColumnNames(std::vector<std::string> &columnNames) = 0;
    virtual int GetRowCount(int32_t &count) = 0;
    virtual int OnGo(int32_t startRowIndex, int32_t targetRowIndex, 
                     int32_t *cachedIndex = nullptr) = 0;
};
```

## Common 模块

### DataSharePredicates 类

**头文件**: `interfaces/inner_api/common/include/datashare_predicates.h:29`

**比较操作**:
```cpp
class DataSharePredicates {
public:
    DataSharePredicates *EqualTo(const std::string &field, const SingleValue &value);
    DataSharePredicates *NotEqualTo(const std::string &field, const SingleValue &value);
    DataSharePredicates *GreaterThan(const std::string &field, const SingleValue &value);
    DataSharePredicates *LessThan(const std::string &field, const SingleValue &value);
    DataSharePredicates *GreaterThanOrEqualTo(const std::string &field, const SingleValue &value);
    DataSharePredicates *LessThanOrEqualTo(const std::string &field, const SingleValue &value);
    
    DataSharePredicates *In(const std::string &field, const MutliValue &values);
    DataSharePredicates *NotIn(const std::string &field, const MutliValue &values);
    
    DataSharePredicates *Contains(const std::string &field, const std::string &value);
    DataSharePredicates *BeginsWith(const std::string &field, const std::string &value);
    DataSharePredicates *EndsWith(const std::string &field, const std::string &value);
    DataSharePredicates *Like(const std::string &field, const std::string &value);
    DataSharePredicates *Unlike(const std::string &field, const std::string &value);
    DataSharePredicates *Glob(const std::string &field, const std::string &value);
    
    DataSharePredicates *Between(const std::string &field, const std::string &low, 
                                 const std::string &high);
    DataSharePredicates *NotBetween(const std::string &field, const std::string &low, 
                                    const std::string &high);
    
    DataSharePredicates *IsNull(const std::string &field);
    DataSharePredicates *IsNotNull(const std::string &field);
    
    DataSharePredicates *OrderByAsc(const std::string &field);
    DataSharePredicates *OrderByDesc(const std::string &field);
    DataSharePredicates *Distinct();
    DataSharePredicates *Limit(const int number, const int offset);
    DataSharePredicates *GroupBy(const std::vector<std::string> &fields);
    
    DataSharePredicates *BeginWrap();
    DataSharePredicates *EndWrap();
    DataSharePredicates *Or();
    DataSharePredicates *And();
    
    // 连接操作
    DataSharePredicates *CrossJoin(const std::string &tableName);
    DataSharePredicates *InnerJoin(const std::string &tableName);
    DataSharePredicates *LeftOuterJoin(const std::string &tableName);
    DataSharePredicates *Using(const std::vector<std::string> &fields);
    DataSharePredicates *On(const std::vector<std::string> &fields);
    
    // 获取构造的查询条件
    std::string GetWhereClause() const;
    std::vector<std::string> GetWhereArgs() const;
    std::vector<OperationItem> GetOperationList() const;
};
```

### DataShareValuesBucket 类

**头文件**: `interfaces/inner_api/common/include/datashare_values_bucket.h:26`

```cpp
class DataShareValuesBucket {
public:
    void Put(const std::string &columnName, const DataShareValueObject &value);
    void Clear();
    bool IsEmpty() const;
    DataShareValueObject::Type Get(const std::string &columnName, bool &isValid) const;
    std::map<std::string, DataShareValueObject::Type> GetAll() const;
};
```

### DataShareValueObject 类

**头文件**: `interfaces/inner_api/common/include/datashare_value_object.h`

```cpp
class DataShareValueObject {
public:
    enum Type {
        TYPE_NULL,
        TYPE_INT,
        TYPE_DOUBLE,
        TYPE_STRING,
        TYPE_BLOB
    };
    
    DataShareValueObject();
    DataShareValueObject(int value);
    DataShareValueObject(int64_t value);
    DataShareValueObject(double value);
    DataShareValueObject(const std::string &value);
    DataShareValueObject(const std::vector<uint8_t> &value);
    
    Type GetType() const;
    int GetInt(int &value) const;
    int GetLong(int64_t &value) const;
    int GetDouble(double &value) const;
    int GetString(std::string &value) const;
    int GetBlob(std::vector<uint8_t> &value) const;
};
```

### DataShareObserver 类

**头文件**: `interfaces/inner_api/common/include/datashare_observer.h`

```cpp
class DataShareObserver {
public:
    struct ChangeInfo {
        ChangeType changeType_;
        std::vector<Uri> uris_;
        std::vector<std::string> data_;
        std::vector<DataShareValuesBucket> valueBuckets_;
    };
    
    virtual void OnChange(const ChangeInfo &changeInfo) = 0;
};
```

## Permission 模块

### DataSharePermission 类

**头文件**: `interfaces/inner_api/permission/include/data_share_permission.h:30`

```cpp
class DataSharePermission {
public:
    // 静态方法：验证权限
    static int VerifyPermission(Security::AccessToken::AccessTokenID tokenID, 
                                const Uri &uri, bool isRead);
    static bool VerifyPermission(uint32_t tokenID, std::string &permission);
    static bool VerifyPermission(Uri &uri, uint32_t tokenID, std::string &permission, 
                                 bool isSilentUri);
    
    // URI 信任检查
    static int32_t UriIsTrust(Uri &uri);
    static bool IsDataShareUri(const std::string &uri);
    static bool IsSingletonTrustUri(const std::string &uri);
    
    // 实例方法：获取权限
    std::pair<int, std::string> GetUriPermission(Uri &uri, int32_t user, 
                                                   bool isRead, bool isSilent);
    std::pair<int, std::string> GetSilentUriPermission(Uri &uri, int32_t user, 
                                                        bool isRead);
    std::pair<int, std::string> GetExtensionUriPermission(Uri &uri, int32_t user, 
                                                           bool isRead);
    
    // 常量
    static constexpr const char *NO_PERMISSION = "noPermission";
    static constexpr const char *SCHEME_DATASHARE = "datashare";
    static constexpr const char *SCHEME_DATASHARE_PROXY = "datashareproxy";
};
```

## 错误码

**头文件**: `interfaces/inner_api/common/include/datashare_errno.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_OK` | 0 | 成功 |
| `E_ERROR` | 1001 | 通用异常 |
| `E_DATASHARE_PERMISSION_DENIED` | 1080 | 权限被拒绝 |
| `E_VERIFY_FAILED` | 1079 | 验证失败 |
| `E_NOT_IN_TRUSTS` | 1082 | 不在信任列表 |
| `E_EMPTY_URI` | 1081 | URI 为空 |

完整错误码列表请参考 [错误码附录](appendix/Error_Codes.md)。

## 模块依赖关系

```
Consumer (datashare_consumer.so)
    ├── Common (datashare_common.so)
    ├── Permission (datashare_permission.so)
    └── External: ability_runtime, access_token, ipc, kv_store, ...

Provider (datashare_provider.so)
    ├── Common (datashare_common.so)
    ├── Permission (datashare_permission.so)
    ├── ANI (datashare_ani_*.so)
    └── External: ability_runtime, access_token, napi, runtime_core, ...
```

**证据**: `interfaces/inner_api/BUILD.gn` - 依赖关系定义

## 关键结论

1. **接口设计遵循 OpenHarmony 标准** - 使用 sptr、IRemoteObject 等标准类型
2. **纯虚接口便于 Mock 测试** - DataShareHelper 等核心类使用纯虚方法
3. **权限检查独立模块** - DataSharePermission 提供静态权限检查方法
4. **桥接模式解耦数据库** - ResultSetBridge 屏蔽底层数据库差异
5. **值对象支持多类型** - DataShareValueObject 支持 null/int/double/string/blob

## 相关文档

- [N-API 参考](04_NAPI_Reference.md) - JS 层 API
- [架构设计](05_Architecture.md) - 架构图和线程模型
- [构建系统](07_Build_System.md) - 产物和依赖
